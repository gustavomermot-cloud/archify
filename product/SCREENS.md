# Pantallas — Workforce Manager

> Lista de las 20 pantallas en producción. Rol que la usa y descripción breve.

| # | Pantalla | Path | Rol | Descripción |
|---|---|---|---|---|
| 1 | **Login** | `/login.html` | público | Formulario único con **2 tarjetas visuales**: "Empresa Principal" y "Subcontrata". Email + password. Link a forgot password. |
| 2 | **Activación por código** | `/activar.html?codigo=XXX` | público | Nueva sub entra con código enviado por WhatsApp. Form: email, password, nombre completo, 2 checkboxes RGPD. Al enviar: crea user + user_tenant + redirect login. |
| 3 | **Forgot password** | `/forgot-password.html` | público | Input email. Siempre devuelve "Si existe la cuenta te hemos enviado un correo". |
| 4 | **Reset password** | `/reset-password.html?token=XXX` | público | Formulario para nueva contraseña con validación token. |
| 5 | **Dashboard empresa** | `/dashboard.html` | company_admin / company_supervisor | KPIs (jornales mes, horas, subs activas, observaciones abiertas) + 4 gráficos SVG + lista últimas obras + accesos directos. |
| 6 | **Dashboard subcontrata** | `/sub/dashboard.html` | subcontractor_admin / subcontractor_user | Calendario mensual editable con filtro de obra. Auto-save 800 ms. Totales fila/columna. Botones [Importar CSV] + [Nueva observación] + [Generar proforma]. |
| 7 | **Supervisión global** | `/supervision.html` | company_admin / company_supervisor | **Modo global**: lista de las 14 subs con status (al día / con incidencias / sin partes). **Modo detalle**: click en sub → calendario mes read-only + botón abrir observación. |
| 8 | **Obras CRUD** | `/obras.html` | company_admin | Lista obras + filtro estado + botón "+ Nueva". Ficha obra con detalles + asignación subs + precios acordados. |
| 9 | **Subcontratas CRUD** | `/subcontratas.html` | company_admin | Lista 14 subs con color de tag + status. Click → ficha sub con operarios + obras asignadas + calendario mes + accesos + código activación. |
| 10 | **Ficha subcontrata detalle** | `/subcontrata.html?id=9` | company_admin | Vista completa de la sub: datos fiscales, lista operarios (read-only desde empresa), obras asignadas, calendario mes, usuarios con acceso. Botón "Generar código acceso". |
| 11 | **Operarios CRUD** | `/sub/operarios.html` | subcontractor_admin | Lista operarios de la sub + filtro estado/categoría. Botón "+ Nuevo" + "Importar CSV". Click → ficha con asignaciones a obras. |
| 12 | **Incidencias lista** | `/incidencias.html` | ambos | Lista observaciones filtrables por estado/sub/obra/fecha. Columna "Edad" con días desde apertura. Click → modal detalle con timeline. |
| 13 | **Incidencia detalle (modal)** | — | ambos | Timeline con todas las transiciones (quién, cuándo, qué). Mensaje original + respuesta. Botones de acción según rol y estado. |
| 14 | **Proformas lista** | `/proformas.html` | ambos | Lista proformas filtrables. Columnas: periodo, sub, total, estado. Para empresa: multi-tenant agregado. Para sub: sólo las suyas. |
| 15 | **Proforma detalle (modal)** | — | ambos | Header con totales + tabla 23 líneas desglosadas. Botones según estado: Submit, Review, Approve/Reject, Close, Invoice, Export PDF. |
| 16 | **Informes** | `/informes.html` | company_admin / Pro | 4 reportes mensuales con selector periodo + sub + obra. Export CSV / JSON. |
| 17 | **Panel billing** | `/billing.html` | authenticated | Estado suscripción (status, trial_end, próximo cobro). Banner condicional. Botón "Gestionar suscripción" → Stripe Customer Portal. |
| 18 | **Perfil usuario** | `/perfil.html` | authenticated | Cambio nombre, email, password. Botón "Exportar mis datos" (RGPD portabilidad). Botón "Eliminar mi cuenta" (RGPD olvido). |
| 19 | **Admin tenants (futuro)** | `/admin/tenants.html` | superadmin | **Roadmap Q1**: panel interno para monitorizar tenants, bases de datos, suscripciones, errores por tenant. No visible a clientes. |
| 20 | **Health check** | `/api/health` | público | JSON con estado de servidor, BD master, BDs tenant sample, uptime, disk. Usado por UptimeRobot. |

---

## Pantallas de la landing comercial (Hostinger, no panel app)

Estas viven en `work-flow.solutions/workforce/` como HTML + PHP estático:

| # | Pantalla | Path | Descripción |
|---|---|---|---|
| L1 | **Inicio** | `/workforce/inicio.html` | Landing principal con hero + 3 bloques de beneficios + testimonios + CTA. |
| L2 | **Oferta subcontratas** | `/workforce/oferta-subcontratas.php` | Landing dirigida al segmento sub: precio promocional 9,90 €/mes × 3 meses, botón CTA directo al Payment Link. |
| L3 | **Precios** | `/workforce/precios.html` | Tabla comparativa de los 4 planes con botones a Stripe Payment Links. |
| L4 | **Registro** | `/workforce/registro.html` | Form crear perfil con RGPD completo. Guarda en SQLite local para después asociar al pago Stripe. |
| L5 | **Términos** | `/workforce/terminos.html` | T&C Workflow Cloud LLC Delaware. |
| L6 | **Privacidad** | `/workforce/privacidad.html` | Política privacidad RGPD completa. |
| L7 | **Gracias** | `/workforce/gracias.html` | Post-checkout exitoso. Mensaje: "Revisa tu email, te hemos mandado el acceso". |
| L8 | **Cancelado** | `/workforce/cancelado.html` | Post-checkout cancelado. CTA "Reintentar". |
| L9 | **Panel (redirect)** | `/workforce/panel/` | Redirect 302 al dominio del panel real (hoy Cloudflare Tunnel). |

---

## Design system "Blueprint Industrial" (aplicado en las 20 pantallas)

Características visuales consistentes:

- **Fondo**: gris claro con rejilla sutil tipo plano técnico.
- **Tipografía**: `Inter` para texto largo, `JetBrains Mono` para datos numéricos.
- **Paleta**:
  - Azul profundo `#1E3A5F` (primario).
  - Ámbar industrial `#F59E0B` (acentos).
  - Verde técnico `#10B981` (confirmaciones).
  - Rojo señalización `#EF4444` (errores, incidencias).
  - Gris técnico `#64748B` (secundarios).
- **Componentes**:
  - Botones con sombra diagonal 2px.
  - Cards con doble borde (línea gruesa + línea fina interior).
  - Tablas con rayado horizontal muy sutil (como plano a mano).
  - Modales con header tipo "ficha técnica" (bordes rectos, header en caps).
- **Iconografía**: iconos lineales (feather icons) o inline SVG custom.
- **Microanimación**: solo donde aporta feedback (auto-save flash en celda, transición entre estados).

Diferenciación clara respecto al SaaS genérico "azul claro + verde + ilustración 3D" que domina el mercado.

---

## Rutas protegidas vs públicas

### Públicas (sin sesión requerida)

- `/login.html`, `/forgot-password.html`, `/reset-password.html`
- `/activar.html?codigo=X`
- `/api/auth/login`, `/api/auth/forgot-password`, `/api/auth/reset-password`, `/api/auth/activate`
- `/api/activation/{code}`
- `/api/billing/webhook` (verificación por firma)
- `/api/health`

### Autenticadas (requiere sesión)

Todo el resto. Cada endpoint además tiene su matriz role × action.

### Redirects

- `/` → `/dashboard.html` si logueado, `/login.html` si no.
- `/sub/` → `/sub/dashboard.html` si rol sub, 403 si no.
- `/admin/` → `/admin/tenants.html` si superadmin, 404 si no.
