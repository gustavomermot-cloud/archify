# Arquitectura detallada — Workforce Manager

## Capas lógicas

```mermaid
flowchart TB
  subgraph UI_LAYER["🖼️ UI LAYER (vanilla JS)"]
    LOGIN[login.html<br/>+ selector Empresa/Sub]
    DASHE[dashboard empresa<br/>+ supervisión + obras + proformas]
    DASHS[sub/dashboard<br/>+ calendario editable]
  end

  subgraph HTTP_LAYER["🌐 HTTP LAYER (public/index.php)"]
    FC[Front Controller<br/>Routing + CORS + Error handling]
    ROUTES[Routes.php<br/>55+ endpoints declarativos]
  end

  subgraph MW_LAYER["🛡️ MIDDLEWARE LAYER"]
    AUTH[AuthMiddleware]
    TEN[TenantMiddleware]
    CSRF[CsrfMiddleware]
    PERM[PermissionMiddleware]
    PA[ProjectAssignmentMiddleware]
  end

  subgraph CTRL_LAYER["🎮 CONTROLLER LAYER"]
    AC[AuthController]
    PC[ProjectController]
    SC[SubcontractorController]
    WC[WorkerController]
    JC[JornalController]
    OC[ObservationController]
    PFC[ProformaController]
    BC[BillingController]
    SUP[SupervisionController]
    AKC[ActivationController]
    RC[ReportController]
    UC[UserController]
  end

  subgraph SVC_LAYER["⚙️ SERVICE LAYER"]
    AS[AuthService]
    PS[ProjectService]
    SS[SubcontractorService]
    WS[WorkerService]
    JS[JornalService]
    OS[ObservationService]
    PFS[ProformaService]
    SBS[SubscriptionService]
    ACS[ActivationService]
    RS[ReportService]
    MS[MailService]
    PE[ProformaExporter]
    JCI[JornalCsvImporter]
    AudS[AuditService]
  end

  subgraph DATA_LAYER["🗄️ DATA LAYER"]
    TR[TenantResolver]
    TC[TenantConnection]
    TPR[PdoTenantRegistry]
    MPDO[(master PDO<br/>wf_master_runtime)]
    T1[(tenant PDO<br/>wf_tenant_bloc)]
    T2[(tenant PDO<br/>wf_tenant_punjab701)]
    TN[(tenant PDO<br/>wf_tenant_...)]
  end

  LOGIN --> FC
  DASHE --> FC
  DASHS --> FC
  FC --> ROUTES
  ROUTES --> AUTH
  AUTH --> TEN
  TEN --> CSRF
  CSRF --> PERM
  PERM --> PA
  PA --> AC & PC & SC & WC & JC & OC & PFC & BC & SUP & AKC & RC & UC
  AC --> AS
  PC --> PS
  SC --> SS & ACS
  WC --> WS
  JC --> JS & JCI
  OC --> OS
  PFC --> PFS & PE
  BC --> SBS
  AKC --> ACS
  RC --> RS
  AS & PS & SS & WS & JS & OS & PFS & SBS & ACS & RS & AudS --> TR
  TR --> TPR --> TC
  TC --> MPDO & T1 & T2 & TN
```

---

## Resolución tenant (fase crítica de seguridad)

```mermaid
sequenceDiagram
  participant R as Request HTTP
  participant AM as AuthMiddleware
  participant TM as TenantMiddleware
  participant TR as TenantResolver
  participant TPR as PdoTenantRegistry
  participant FS as /etc/workforce/tenants/<slug>.env
  participant SQL as MariaDB

  R->>AM: cookie wfm_sid
  AM->>AM: lee $_SESSION
  AM-->>TM: user autenticado
  TM->>TR: fromAuthenticatedUser(user)
  TR->>TR: lee user.subcontractor_id<br/>(NUNCA de HTTP)
  TR->>TPR: findBySubcontractorId(9)
  TPR->>SQL: SELECT * FROM master.tenant_databases WHERE subcontractor_id=9
  SQL-->>TPR: {database_name, host, port, username}
  TPR->>FS: lee /etc/workforce/tenants/punjab701.env
  FS-->>TPR: TENANT_DB_PASSWORD
  TPR-->>TR: Tenant{slug,dbName,user,pass,...}
  TR-->>TM: Tenant
  TM->>SQL: new PDO(mysql:host=X;dbname=workforce_bloc_punjab701, wf_tenant_punjab701, pass)
  TM->>SQL: SELECT * FROM tenant_meta → verifica fingerprint
  SQL-->>TM: master_subcontractor_id=9 ✓
  TM-->>R: $container->instance('tenant.pdo', $pdo)
  Note over R,SQL: Request continúa con PDO<br/>que solo puede acceder a su BD
```

Si cualquier eslabón falla (sesión inválida, tenant_meta no coincide, fichero .env no legible), la petición aborta con 403/423/404.

---

## Flujo autenticación

```mermaid
sequenceDiagram
  actor U as Usuario
  participant F as Form login
  participant AC as AuthController
  participant AS as AuthService
  participant UR as UserRepository
  participant PP as PasswordPolicy
  participant RL as RateLimiter
  participant S as Session
  participant M as master DB

  U->>F: credenciales
  F->>AC: POST /api/auth/login
  AC->>RL: tooManyAttempts(ip,user)?
  RL->>M: SELECT COUNT login_attempts
  M-->>RL: n
  RL-->>AC: ok
  AC->>UR: findByLogin(user_or_email)
  UR->>M: SELECT * FROM users WHERE ...
  M-->>UR: user o null
  alt user existe
    AC->>PP: verify(password, hash)
  else user no existe
    AC->>PP: verifyDummy() ← mismo coste bcrypt
    Note over AC,PP: anti-timing attack
  end
  PP-->>AC: match true/false
  alt match + active
    AC->>S: session_regenerate_id(true)
    AC->>S: $_SESSION = {user_id, role, company_id, subcontractor_id, tenant, csrf_token}
    AC->>M: INSERT login_attempts (success=1)
    AC-->>U: 200 {user, csrf_token} + Set-Cookie wfm_sid
  else no match
    AC->>M: INSERT login_attempts (success=0)
    AC-->>U: 401 credenciales_invalidas
  end
```

---

## Flujo activación por código

```mermaid
sequenceDiagram
  actor E as Empresa
  actor S as Subcontrata (nueva)
  participant W as Workforce
  participant M as master DB
  participant ST as workforce_bloc_<slug>

  E->>W: POST /api/subcontractors/{id}/activation-codes
  W->>W: Genera código tipo PUNJAB701-A3F7K2
  W->>M: INSERT activation_codes (code, sub_id, expires_at=+7d)
  W-->>E: {code, activation_url}
  E->>S: Envía URL por WhatsApp
  S->>W: GET /activar.html?codigo=PUNJAB701-A3F7K2
  W-->>S: HTML con form (email, pass, nombre, checkboxes RGPD)
  S->>W: POST /api/activate {code, email, pass, full_name, accepted_*}
  W->>M: SELECT activation_codes WHERE code=X AND used_at IS NULL AND expires_at > NOW()
  M-->>W: fila
  W->>M: BEGIN TX
  W->>M: INSERT users (bcrypt cost 12, consent_ip, consent_ua, consent_at)
  W->>M: INSERT user_tenants (role='subcontractor_admin', is_primary=1)
  W->>M: UPDATE activation_codes SET used_at=NOW
  W->>M: COMMIT
  W-->>S: 201 {next: '/login.html?email=...'}
  Note over S: Sub puede loguear y entrar a /sub/dashboard.html
```

---

## Modelo billing — ciclo de vida

```mermaid
stateDiagram-v2
  [*] --> incomplete: Checkout iniciado
  incomplete --> trialing: Método pago aceptado
  trialing --> active: Trial terminó, cobro OK
  trialing --> canceled: Usuario canceló durante trial
  active --> past_due: Pago falló
  past_due --> active: Reintento OK
  past_due --> canceled: Reintentos agotados
  active --> canceled: Usuario cancela
  canceled --> [*]
```

El banner UI reacciona a cada estado:
- `trialing + days_until_trial_end ≤ 7` → banner naranja "Tu prueba acaba en N días"
- `past_due` → banner rojo "Pago fallido"
- `canceled` → banner neutro
- `active` → sin banner

---

## Modelo proforma — máquina de estados

```mermaid
stateDiagram-v2
  [*] --> draft: Sub genera proforma
  draft --> submitted: Sub envía
  submitted --> under_review: Empresa toma
  under_review --> approved: Empresa aprueba
  under_review --> rejected: Empresa rechaza (con motivo)
  rejected --> draft: Sub corrige y regenera
  approved --> claim_period: Empresa abre reclamación (7d)
  approved --> closed: Empresa cierra directo
  claim_period --> closed: Vence periodo
  closed --> invoiced: Empresa factura
  invoiced --> [*]
```

Reglas de negocio:
- Si hay observaciones abiertas del mes → `draft→submitted` bloqueado (422)
- Sub NUNCA puede aprobar su propia proforma (hard-deny en PermissionService)
- Las líneas de proforma referencian `source_type='jornal'` + `source_id` para trazabilidad total

---

## Despliegue y entorno

```mermaid
flowchart LR
  subgraph DEV["🧑‍💻 Desarrollo local"]
    DEVPC[gtm-MP100<br/>Apache local<br/>MariaDB local<br/>Vhost workforce.test]
  end

  subgraph GH["💾 GitHub"]
    REPO[Repo privado<br/>workforce-manager]
    REPO2[Repo público<br/>archify docs]
  end

  subgraph HOST["🏢 Hostinger shared"]
    LANDING[work-flow.solutions/workforce/<br/>Landing comercial]
    OFER[/oferta-subcontratas.php]
    PANEL[/workforce/panel/ → redirect 302]
  end

  subgraph PROD["🚀 Producción (VPS futuro)"]
    VPS[Ubuntu 22<br/>Apache + PHP 8.3 + MariaDB<br/>+ wkhtmltopdf + fail2ban + ufw]
  end

  subgraph TUNNEL["☁️ Cloudflare"]
    CT[Cloudflare Tunnel<br/>quick o named]
  end

  DEVPC -->|git push| REPO
  REPO -->|docs| REPO2
  DEVPC -->|rsync deploy.sh| LANDING
  PANEL -.302.-> CT
  CT -.-> DEVPC
  DEVPC -.futuro.-> VPS
  LANDING --> OFER
```

---

## Decisiones arquitectónicas clave

| Decisión | Razón |
|---|---|
| **PHP vanilla sin framework** | Minimizar deuda técnica, control total, menos CVEs, deploy trivial |
| **Multi-tenant físico (BD por sub)** | Aislamiento real: Punjab y Dicotec no se tocan ni por bug SQL |
| **PDO::ATTR_EMULATE_PREPARES=false** | Prepared statements reales, no simulados → inmune a SQL injection |
| **Sin JavaScript framework** | SPAs vanilla, sin node_modules, sin build pipeline |
| **wkhtmltopdf para PDFs** | Resultado profesional vs dompdf; aceptamos binario a cambio |
| **Cloudflare Tunnel para dev público** | Exponer app local a clientes sin VPS temporalmente |
| **Stripe Payment Links primero, Checkout API después** | Validar demanda antes de invertir en integración backend |
| **Blueprint Industrial design system** | Diferenciación visual vs SaaS genéricos azul+verde |

---

## Costes operativos estimados

| Escenario | Mensual | Anual |
|---|---|---|
| Dev local + Cloudflare Tunnel | 0 € | 0 € |
| VPS Hetzner CX22 + dominio | 5,83 € + 1 € | ~82 € |
| VPS Hostinger KVM 2 | 5,99 € | 72 € |
| Fly.io free tier (hasta 2 vms) | 0 € | 0 € |
| Hostinger shared (ya pagado) | 0 € adicional | — |
| Stripe fees | 1,5% + 0,25€ por transacción | — |

Primera suscripción Workforce Subcontratas (48,80 €) paga 8 meses de VPS Hetzner.
