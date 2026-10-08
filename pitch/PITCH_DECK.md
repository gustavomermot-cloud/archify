# Workforce Manager — Pitch Deck

> Workflow Cloud LLC · Delaware · 2026
> Documento vivo. Formato "slides en Markdown"; cada `---` separa una diapositiva.

---

# Slide 1 — Portada

**Workforce Manager**
Gestión de jornales multi-subcontrata para la construcción española

- Workflow Cloud LLC · Delaware
- Fundador: Gustavo Mermot
- Barcelona · 2026
- Landing: [work-flow.solutions/workforce](https://work-flow.solutions/workforce/)

---

# Slide 2 — El problema

Las constructoras medianas en España pierden tiempo y dinero supervisando a sus subcontratas.

- **8-12 horas/semana** de un jefe de obra cuadrando partes en Excel o WhatsApp.
- **5-15 %** de discrepancias en proformas mensuales → disputas y pagos retenidos.
- Cero trazabilidad: cuando un operario falta, no queda registro ni responsable.
- Un ERP serio cuesta **15.000-40.000 €** de implantación + licencias; una PYME de construcción no lo compra.
- El resultado: todo el sector sigue usando Excel + WhatsApp + papeles arrugados.

---

# Slide 3 — Nuestra solución

Una pantalla por rol. Un calendario. Un flujo mensual que termina en proforma aprobada.

- **Empresa principal**: ve en modo supervisión el calendario de todas sus subs y aprueba proformas.
- **Subcontrata**: declara jornales día a día en calendario editable con auto-save.
- **Incidencias trazables**: timeline con 6 tipos de observación y 5 estados.
- **Proformas automáticas**: generadas desde los jornales del mes, exportables a PDF profesional.
- **Setup 15 minutos**: no hay consultora, no hay implantador.

---

# Slide 4 — Diferenciador clave: aislamiento físico multi-tenant

Cada subcontrata tiene **su propia base de datos** y **su propio usuario MySQL**.

- 14 bases de datos distintas hoy en producción, una por subcontrata.
- GRANTS MySQL por tenant: `wf_tenant_punjab701` ni siquiera ve que existe la BD de BLOC.
- Imposible leer datos ajenos aunque un bug SQL lo intente — MariaDB lo rechaza antes.
- Fingerprint `tenant_meta.master_subcontractor_id` como segunda capa de verificación.
- Auditable para cualquier cliente que pida explicación formal del aislamiento.

No conocemos ningún otro SaaS español del sector que haga esto. La mayoría son multi-tenant lógico (misma BD + `WHERE tenant_id = ?`).

---

# Slide 5 — Mercado — España

- **140.000 empresas** del sector construcción en España (INE 2024).
- **~20.000** son constructoras medianas (10-250 empleados) que subcontratan de forma habitual.
- Cada constructora trabaja con **3-15 subcontratas** simultáneas.
- Mercado directo accesible: Cataluña, Madrid, Valencia, Andalucía — ~12.000 cuentas.
- Expansión natural: Portugal, México, Argentina (mismo idioma, mismas categorías laborales).

---

# Slide 6 — Modelo de negocio

Suscripción SaaS mensual. 4 planes.

| Plan | Precio | Target | ARR/cliente |
|---|---|---|---|
| Autónomo | 14 €/mes | 1 operario + 1 obra | 168 € |
| Workforce Subcontratas | **48,80 €/mes** | Sub con operarios ilimitados | 586 € |
| Empresa Principal | 39 €/mes | Constructora que supervisa | 468 € |
| Pro Constructora | 89 €/mes | Constructora + reportes + API + white-label | 1.068 € |

**Oferta introductoria**: 30 días gratis + 9,90 €/mes × 3 meses (cupones `SUBCONTRATA3M` / `AUTONOMO3M`).

ARR medio ponderado estimado: **~550 €/cliente/año**.

---

# Slide 7 — Tracción actual (octubre 2026)

Lo que ya funciona en producción real:

- **Bloc Creatiu SL** (Barcelona) usando el sistema desde su cuenta empresa principal.
- **14 subcontratas** activas con sus propias bases de datos aisladas: BLOC, PUNJAB 701, EDUARDO, MALIK, DICOTEC, JAVI, NAIM, FARID, SOHAN PLAC SL, NETGEN PLAC, CASTILLA LEON, ZIA CINTERO, HUSSEIN TMK, amigos naim, MANYOT.
- **222 obras** reales creadas y asignadas.
- **1.750 jornales históricos** importados de producción (datos reales de meses previos).
- Stripe LIVE integrado y testeado end-to-end (Workflow Cloud LLC).
- Landing comercial publicada: `work-flow.solutions/workforce/`.

---

# Slide 8 — Métricas de validación

| Métrica | Valor |
|---|---|
| Subcontratas gestionadas simultáneamente | 14 |
| Operarios declarados | 50+ |
| Obras activas | 222 |
| Jornales en el sistema | 1.750 |
| Tests PHPUnit passing | 148 |
| Páginas UI en producción | 20 |
| Endpoints API | 55+ |
| Líneas de código PHP | ~15.000 |
| Migraciones SQL | 14 |

Honesto: cliente de pago externo = 0. Primer caso de uso real = la empresa del fundador. Siguiente paso = captar 5-10 constructoras externas en Q1.

---

# Slide 9 — Go-to-market

Enfoque directo, sin paid ads hasta validar funnel.

1. **Mes 1-2**: 10 constructoras conocidas del círculo profesional (Barcelona, red Bloc Creatiu). Trial 30 días.
2. **Mes 3-4**: Convertir a 5 de pago (CAC humano, cero ads). Testimonios en vídeo.
3. **Mes 5-6**: SEO + landing optimizada. Primera campaña Google Ads sobre keywords de nicho ("gestión jornales construcción", "proforma subcontrata").
4. **Mes 7-12**: Partnerships con asesorías laborales que atienden PYMES de construcción (comisión 20 % año 1).
5. **Año 2**: Programa partner para gestorías + eventos sectoriales (FEVEC, SEOPAN).

---

# Slide 10 — Equipo

**Gustavo Mermot** — Fundador & desarrollador único hasta hoy.

- Entidad legal: **Workflow Cloud LLC** (Delaware, USA).
- Operación comercial en España vía Bloc Creatiu SL (cliente ancla).
- 15+ años programando: PHP, Python, Android, infraestructura.
- Perfil operacional: conoce el sector construcción desde dentro (jefe de obra en Bloc Creatiu).
- Diseña, programa, despliega y vende el mismo producto. Ventaja: ciclo feedback 24 h; iteración semanal real.

Equipo objetivo con 300 K€ ronda: +1 developer PHP, +1 comercial B2B.

---

# Slide 11 — Competencia

| Producto | Precio/mes | Multi-sub nativo | Aislamiento BD | Setup | Español | Categorías oficial/peón |
|---|---|---|---|---|---|---|
| **Workforce Manager** | **48,80 €** | **Sí** | **Físico** | **15 min** | **Nativo** | **Sí** |
| Excel compartido | 0 € | No | — | 0 | Sí | Manual |
| Factorial / Sesame | 70-150 € | No | Lógico | 2-5 días | Sí | No |
| Odoo Construction | 200-800 € + implantación | Parcial | Lógico | 2-6 semanas | Parcial | No |
| Sage 50 construcción | 100-300 € + licencia | No | Lógico | Semanas | Sí | Parcial |
| Zoho People | 2,50 €/user | No | Lógico | 1-3 días | Parcial | No |

Nadie ocupa el hueco de "multi-sub nativo + aislamiento físico + categorías construcción".

---

# Slide 12 — Ventaja técnica

Por qué el stack permite competir con equipos de 10 personas:

- **PHP 8.3 vanilla** sin frameworks → cero deuda técnica externa, cero `node_modules`, cero cadena de dependencias.
- **PDO prepared statements reales** (`EMULATE_PREPARES=false`) → inmune a SQL injection.
- **Multi-tenant físico**: 1 BD + 1 usuario MySQL por sub. 6 capas de middleware encadenadas.
- **148 tests PHPUnit** passing cubriendo controladores, servicios y hard-denies.
- **Deploy VPS en 10-15 min** vía `install.sh` idempotente (Apache + MariaDB + wkhtmltopdf + Let's Encrypt + ufw + fail2ban + backup).
- **Hostless local dev**: Cloudflare Tunnel permite enseñar a cliente sin VPS mientras validamos.
- **Design system Blueprint Industrial** propio → distinción visual vs genéricos SaaS azul-verde.

---

# Slide 13 — Roadmap 12 meses

| Trimestre | Hitos |
|---|---|
| **Q1 2027** | Migración a VPS Hetzner · onboarding wizard · 10 clientes beta · panel admin interno |
| **Q2 2027** | App móvil operario (declarar horas desde móvil) · integración ERP A3 · 50 clientes |
| **Q3 2027** | Analytics avanzado · remesas SEPA · exportación Contaplús/A3 · 100 clientes |
| **Q4 2027** | Expansión Portugal + México + Argentina · white-label para constructoras grandes · 200 clientes |

Riesgo asumido: priorizar producto sobre marketing hasta Q2. Si tracción Q1 flojea, revisar antes de contratar.

---

# Slide 14 — Financials (proyección conservadora)

Supuestos: ARR medio 550 €/cliente/año, churn 5 %/mes primeros 12m, 3 %/mes después.

| Año | Clientes | ARR | Costes | Margen bruto |
|---|---|---|---|---|
| 1 | 50 | 25.000 € | 15.000 € | 40 % |
| 2 | 200 | 110.000 € | 33.000 € | 70 % |
| 3 | 500 | 300.000 € | 75.000 € | 75 % |
| 4 | 1.100 | 650.000 € | 160.000 € | 75 % |
| 5 | 2.000 | 1.200.000 € | 280.000 € | 77 % |

Break-even operacional alcanzable en año 3 con equipo de 3 personas.

Fuente de costes: VPS, Stripe fees (~2 % sobre revenue), soporte, dominio, infraestructura, salario base equipo mínimo.

---

# Slide 15 — Ask

**Ronda semilla: 150-300 K€** (opcional, sólo si acelera go-to-market).

Uso de los fondos:

- **40 %** desarrollo (1 developer PHP senior, 12 meses).
- **35 %** comercial B2B (1 account executive construcción, 12 meses).
- **15 %** marketing (SEO, landing, 2 eventos sectoriales).
- **10 %** infraestructura + legal (VPS, cumplimiento RGPD, contratos con partners).

Hito a 12 meses tras cierre: **200 clientes de pago**, ARR **110 K€**, equipo de 3.

Alternativa bootstrap: sin ronda, llegar a 50 clientes con cash flow operativo antes de replantear.

---

# Slide 16 — Contacto

**Gustavo Mermot**
Workflow Cloud LLC · Delaware
gustavo@bloccreatiu.com · +34 654 162 515

- Landing: [work-flow.solutions/workforce](https://work-flow.solutions/workforce/)
- Documentación técnica: este repo (`archify`)
- Demo en vivo bajo petición.
