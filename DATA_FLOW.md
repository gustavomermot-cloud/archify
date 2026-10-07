# Flujos de datos — Workforce Manager

## 1. Flujo "declarar jornales día a día"

```mermaid
sequenceDiagram
  autonumber
  actor S as Subcontrata
  participant UI as /sub/dashboard.html
  participant API as /api/jornales
  participant MW as Middleware chain
  participant JS as JornalService
  participant T as workforce_bloc_<sub>
  participant AL as activity_log

  S->>UI: Click celda vacía día 15, operario Salim
  UI->>UI: Input numérico inline (8)
  UI->>API: POST /api/jornales<br/>{project_id, worker_id, date, quantity}<br/>Cookie wfm_sid + X-CSRF-Token
  API->>MW: AuthMiddleware (sesión OK)
  MW->>MW: TenantMiddleware (resuelve tenant.pdo = Punjab)
  MW->>MW: CsrfMiddleware (token OK)
  MW->>MW: PermissionMiddleware (jornal.create → sub ✓)
  MW->>MW: ProjectAssignmentMiddleware (project asignado a Punjab ✓)
  MW->>JS: createOrUpdate(...)
  JS->>JS: Validar worker pertenece a Punjab
  JS->>JS: Validar fecha razonable
  JS->>T: BEGIN
  JS->>T: INSERT jornales ON DUPLICATE KEY UPDATE quantity, notes
  JS->>AL: INSERT activity_log (action='jornal_created')
  JS->>T: Buscar observations open del mismo (project,date)
  alt hay observations missing_worker/wrong_quantity
    JS->>T: UPDATE observations SET status='corrected'
    JS->>AL: INSERT activity_log (action='observation_corrected')
  end
  JS->>T: COMMIT
  JS-->>API: {id, quantity}
  API-->>UI: 201 {data: {...}}
  UI->>UI: Debounce 800ms, auto-save siguiente celda
```

---

## 2. Flujo "generar proforma mensual"

```mermaid
sequenceDiagram
  autonumber
  actor S as Subcontrata
  participant UI as /sub/proformas.html
  participant API as /api/proformas
  participant PFS as ProformaService
  participant OS as ObservationService
  participant PRC as PriceService
  participant T as workforce_bloc_<sub>

  S->>UI: Click "+ Generar proforma octubre 2026"
  UI->>API: POST /api/proformas<br/>{period_year: 2026, period_month: 10}
  API->>PFS: generate(10, 2026, user)
  PFS->>T: SELECT jornales WHERE date BETWEEN '2026-10-01' AND '2026-10-31'
  T-->>PFS: 142 jornales
  PFS->>OS: hasOpenObservationsForMonth(2026,10)
  OS->>T: SELECT jornal_observations WHERE status IN (open,reviewing) AND date in month
  T-->>OS: [...] (si hay)
  alt hay incidencias abiertas
    OS-->>PFS: throw ValidationException
    PFS-->>UI: 422 {error: 'open_observations_blocking', ids: [34,35]}
    UI->>UI: Muestra banner rojo con enlaces a observations
  else sin incidencias
    PFS->>T: BEGIN TX
    PFS->>T: INSERT proformas (period_year, period_month, status='draft')
    loop por cada jornal del mes agrupado por project+worker
      PFS->>PRC: priceFor(worker_id, date, project_id)
      PRC-->>PFS: unit_price (project_subcontractor_prices o default 120)
      PFS->>T: INSERT proforma_lines (proforma_id, project_id, worker_id, quantity, unit_price, total, source_type='jornal', source_id=jornal.id)
    end
    PFS->>T: UPDATE proformas SET subtotal, tax=subtotal*0.21, total
    PFS->>T: COMMIT
    PFS-->>UI: 201 {id, subtotal, tax, total, lines_count: 142}
    UI->>UI: Mostrar modal detalle
  end
  S->>UI: Click "Enviar"
  UI->>API: POST /api/proformas/{id}/submit
  API->>PFS: submit(id, user)
  PFS->>T: UPDATE proformas SET status='submitted', submitted_at=NOW
  PFS-->>UI: 200 {status: 'submitted'}
```

---

## 3. Flujo "empresa aprueba proforma"

```mermaid
sequenceDiagram
  actor E as Empresa
  participant UI as /proformas.html
  participant API as /api/proformas
  participant PFS as ProformaService
  participant TMW as TenantMiddleware
  participant T as workforce_bloc_<sub>
  participant M as workforce_master
  participant PE as ProformaExporter
  participant WKH as wkhtmltopdf
  participant FS as /var/lib/wfm/exports

  E->>UI: Lista proformas filtradas por status=submitted
  UI->>API: GET /api/proformas (sin filtro = agregado master)
  API->>PFS: list cross-tenant iterando subs activas
  PFS->>M: SELECT subcontractors WHERE company_id=1 AND status='active'
  loop por cada sub
    PFS->>TMW: resolveTargetTenant(user, sub.id)
    TMW->>T: conectar
    PFS->>T: SELECT proformas filtered
  end
  PFS-->>API: lista agregada con _meta.subcontractor_name
  API-->>UI: 200 [...]
  E->>UI: Click en proforma Punjab → modal detalle
  UI->>API: GET /api/proformas/5?subcontractor_id=9
  API->>PFS: get(5, user)
  PFS-->>UI: {proforma, lines[...]}
  E->>UI: Click "Pasar a revisión"
  UI->>API: POST /api/proformas/5/review {subcontractor_id: 9}
  API->>PFS: review(5, user)
  PFS->>T: UPDATE status='under_review'
  E->>UI: Click "Aprobar"
  UI->>API: POST /api/proformas/5/approve
  API->>PFS: approve(5, user)
  PFS->>PFS: assertNotSubcontractor() ← hard-deny si llega una sub
  PFS->>T: UPDATE status='approved', approved_at=NOW
  E->>UI: Click "Export PDF"
  UI->>API: GET /api/proformas/5/export?format=pdf&subcontractor_id=9
  API->>PE: exportPdf(5, user)
  PE->>T: fetch proforma + lines + sub + obras
  PE->>PE: renderHtml con Blueprint template
  PE->>WKH: proc_open('wkhtmltopdf --disable-local-file-access - -')
  WKH-->>PE: PDF bytes
  PE->>FS: write /var/lib/wfm/exports/proformas/<slug>/proforma-5-2026-10.pdf
  PE-->>API: $path
  API->>API: readfile + Content-Type: application/pdf + Content-Disposition: attachment
  API-->>E: download .pdf (10 páginas, 103 KB)
```

---

## 4. Flujo "observación cross-tenant"

```mermaid
sequenceDiagram
  actor E as Empresa
  actor S as Subcontrata
  participant UI_E as /supervision.html
  participant UI_S as /sub/dashboard.html
  participant API as /api/observations
  participant OS as ObservationService
  participant TMW as TenantMiddleware
  participant T as workforce_bloc_punjab701
  participant M as workforce_master
  participant MAIL as MailService

  E->>UI_E: Ve calendario Punjab · click celda día 5
  UI_E->>UI_E: popover "Nueva observación"<br/>tipo=missing_worker, mensaje
  UI_E->>API: POST /api/observations<br/>{subcontractor_id: 9, project_id, date, type, message}
  API->>OS: create(..., user)
  OS->>TMW: resolveTargetTenant(user, 9) ← pivot a tenant Punjab
  TMW->>T: conecta
  OS->>M: valida project_id pertenece a company_id del user
  OS->>T: INSERT jornal_observations (created_by_role='company_supervisor', status='open')
  OS->>T: INSERT activity_log (observation_created)
  OS->>MAIL: send to subcontractor_admin de Punjab
  MAIL->>MAIL: template observationCreated()
  OS-->>API: {id, status: 'open'}
  API-->>UI_E: 201
  Note over E,S: ---- Más tarde Punjab ----
  S->>UI_S: Ve badge amarillo en celda día 5
  S->>UI_S: Click badge → popover "Responder"
  UI_S->>API: POST /api/observations/{id}/respond {response_message}
  API->>OS: respond(id, msg, user)
  OS->>OS: assertSubcontractorOwner()
  OS->>T: UPDATE status='reviewing', response_message, responded_by, responded_at
  S->>UI_S: Mete el jornal que faltaba
  Note over UI_S,T: Flujo "declarar jornal" (ver 1)
  Note over T: JornalService detecta observación<br/>open sobre ese (project,date)<br/>→ UPDATE status='corrected'
  S->>UI_S: La observación aparece como corrected
  E->>UI_E: Ve notificación "observación corrected"
  E->>API: POST /api/observations/{id}/accept
  API->>OS: accept(id, user)
  OS->>T: UPDATE status='accepted'
  E->>API: POST /api/observations/{id}/close
  API->>OS: close(id, user)
  OS->>T: UPDATE status='closed'
```

---

## 5. Flujo "signup + billing Stripe"

```mermaid
sequenceDiagram
  actor C as Cliente nuevo
  participant L as Landing work-flow.solutions/workforce/
  participant SP as /registro.html
  participant SA as /api/signup.php
  participant SQ as signups.sqlite
  participant ST as Stripe Payment Link
  participant SC as checkout.stripe.com
  participant WH as /api/webhook.php (landing)
  participant WAPP as App Workforce
  participant M as workforce_master.subscriptions

  C->>L: Visita landing
  C->>L: Click "Suscríbete"
  L->>SP: Click "Crear perfil" primero
  C->>SP: Rellena nombre, empresa, email, pass, checkboxes RGPD
  SP->>SA: POST con CSRF
  SA->>SA: Valida consentimientos server-side
  SA->>SQ: INSERT signup (bcrypt cost 12, ip, user_agent, timestamp)
  SA-->>SP: 200 {ok, next: 'inicio.html?email=...'}
  C->>L: Click "Suscríbete por 9,90 €"
  L->>ST: GET Payment Link con prefilled_promo_code
  ST->>SC: Hosted checkout + trial 30d + cupón WORKFORCE_INTRO
  C->>SC: Rellena tarjeta
  SC->>C: Redirect a /gracias.html?session_id=cs_xxx
  SC-->>WH: POST webhook customer.subscription.created
  WH->>WH: Verifica firma Stripe
  WH->>SQ: UPDATE signup WHERE email con stripe_customer_id, subscription_id
  Note over WH,M: Este webhook también puede llamar a la app
  WH->>WAPP: POST /api/billing/webhook (cross-service)
  WAPP->>WAPP: SubscriptionService::syncFromWebhook
  WAPP->>M: INSERT subscriptions (status='trialing', trial_end=+30d)
  Note over C: ---- Día 23: quedan 7 días trial ----
  C->>WAPP: Login en /sub/dashboard
  WAPP->>M: GET subscription
  WAPP-->>C: Response con banner_level='warning'
  C->>C: Banner naranja "Tu prueba acaba en 7 días"
  C->>C: Click "Configurar pago"
  WAPP->>SC: Billing Portal session
  SC-->>C: redirect a portal
```

---

## 6. Flujo "import CSV jornales"

```mermaid
sequenceDiagram
  actor S as Subcontrata
  participant UI as /sub/dashboard.html
  participant API as /api/jornales/import
  participant JCI as JornalCsvImporter
  participant WS as WorkerService
  participant JS as JornalService
  participant T as workforce_bloc_<sub>

  S->>UI: Click [Importar CSV]
  UI->>UI: Modal con descarga plantilla + file input
  S->>UI: Selecciona archivo.csv + project_id
  UI->>API: POST multipart (file + project_id + X-CSRF)
  API->>JCI: import(file, project_id, user)
  JCI->>JCI: Valida tamaño ≤ 2MB
  JCI->>JCI: finfo MIME = text/csv
  JCI->>JCI: Convertir latin1 → UTF-8 si procede
  JCI->>JCI: strip BOM UTF-8
  JCI->>JCI: autodetect separador ; o ,
  JCI->>JCI: Parse header obligatorio
  loop por cada fila (max 1000)
    JCI->>T: SELECT worker WHERE document_number=X AND subcontractor_id=mi_sub
    alt no existe
      JCI->>WS: create(first_name, last_name, document_number, category='oficial')
      WS->>T: INSERT workers
      JCI->>JCI: created_workers++
    end
    JCI->>T: SELECT worker_project_assignments WHERE worker_id=X AND project_id=Y
    alt no asignado
      JCI->>T: INSERT worker_project_assignments
    end
    JCI->>JS: createOrUpdate(project_id, worker_id, date, quantity, notes, user)
    JS->>T: INSERT jornales ON DUPLICATE KEY UPDATE
    alt fila con error
      JCI->>JCI: errors[] += {row, reason}
    else ok
      JCI->>JCI: imported++
    end
  end
  JCI-->>API: {imported, skipped, errors[], created_workers}
  API-->>UI: 200
  UI->>UI: Toast "Importados X, Y errores"
  UI->>UI: Modal muestra lista errores por fila
```

---

## 7. Aislamiento cross-tenant visualizado

```mermaid
flowchart TB
  subgraph MASTER["🗄️ workforce_master (compartida)"]
    T_C[companies]
    T_S[subcontractors]
    T_U[users]
    T_UT[user_tenants]
    T_P[projects]
    T_PSA[project_subcontractor_assignments]
    T_TD[tenant_databases]
    T_SUB[subscriptions]
  end

  subgraph PUNJAB["🔒 workforce_bloc_punjab701<br/>SOLO usuario wf_tenant_punjab701"]
    PW[workers]
    PWA[worker_project_assignments]
    PJ[jornales]
    PO[jornal_observations]
    PP[proformas]
    PPL[proforma_lines]
    PAL[activity_log]
    PTM[tenant_meta<br/>master_subcontractor_id=9]
  end

  subgraph BLOC["🔒 workforce_bloc_bloc<br/>SOLO usuario wf_tenant_bloc"]
    BW[workers]
    BWA[worker_project_assignments]
    BJ[jornales]
    BO[jornal_observations]
    BP[proformas]
    BPL[proforma_lines]
    BAL[activity_log]
    BTM[tenant_meta<br/>master_subcontractor_id=1]
  end

  wf_tenant_punjab701 -.->|GRANT solo aquí| PUNJAB
  wf_tenant_bloc -.->|GRANT solo aquí| BLOC
  wf_master_runtime -.->|GRANT solo aquí| MASTER
  wf_dba -.->|GRANT ALL ← solo scripts| MASTER
  wf_dba -.-> PUNJAB
  wf_dba -.-> BLOC
```

**Clave**: `wf_tenant_punjab701` no existe desde la perspectiva de `BLOC`. Si una query intentara `SELECT * FROM workforce_bloc_bloc.workers` desde el PDO de Punjab, MariaDB respondería `Access denied for user 'wf_tenant_punjab701'@'127.0.0.1' to database 'workforce_bloc_bloc'`.

El fingerprint `tenant_meta.master_subcontractor_id` es una segunda capa: aunque alguien swapeara credenciales por error (ej. apuntar Punjab a la BD de BLOC), la app verifica que la BD contiene el ID esperado y aborta con `TenantMismatchException`.
