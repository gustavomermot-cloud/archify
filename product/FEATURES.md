# Features — Workforce Manager

> Catálogo completo de funcionalidades. Marcadas con ✓ las que ya están en producción, con 🧪 las que están en beta, con 📅 las de roadmap.

---

## 1. Gestión de obras (projects)

- ✓ CRUD completo de obras con nombre, código interno, dirección, fechas inicio/fin, notas.
- ✓ Estados: `active`, `paused`, `closed`, `archived`.
- ✓ Asignación/desasignación de subcontratas a una obra (con fecha).
- ✓ Precio por hora acordado por obra + sub + categoría (overrides del precio default).
- ✓ Filtros por estado, por sub asignada.
- 📅 Dependencias entre obras (fase 1, fase 2).

## 2. Gestión de subcontratas

- ✓ CRUD con nombre, slug, color visual (hex), tax_id, contacto, precio hora default.
- ✓ Estados: `active`, `inactive`, `suspended`.
- ✓ **Provisioning automático de BD y usuario MySQL** por sub (aislamiento físico).
- ✓ **Código de activación** único con TTL 7 días para onboarding sin password.
- ✓ Página detalle con operarios + obras asignadas + calendario mes actual.
- ✓ Lista de accesos (usuarios con rol en la sub) + revocación.
- 📅 Rating interno de la sub (puntualidad, calidad, disputas).

## 3. Gestión de operarios (workers)

- ✓ CRUD con nombre, apellidos, DNI/NIE, teléfono, categoría, fecha alta, notas.
- ✓ Categorías construcción nativas: `oficial`, `peon`, `cintero`, `encargado`.
- ✓ Estados: `active`, `inactive`, `deleted`.
- ✓ Asignación a obras (worker × project assignment history).
- ✓ Import CSV masivo con detección automática encoding (UTF-8, Latin1) y separador (`;` o `,`).
- 📅 Foto operario + validez documentación PRL.
- 📅 Firma digital operario (al confirmar sus horas).

## 4. Calendario mensual editable

- ✓ Vista mensual con **matriz operarios × días**.
- ✓ Click en celda vacía → input numérico inline → auto-save a los 800 ms.
- ✓ Celdas pintadas por color de la sub.
- ✓ Totales por operario (fila) y por día (columna).
- ✓ Navegación mes anterior / siguiente.
- ✓ Filtro por obra (si la sub trabaja en varias).
- ✓ Indicadores visuales por celda:
  - Verde: jornal confirmado.
  - Amarillo: observación abierta en esa celda.
  - Rojo: incidencia sin resolver > 7 días.
- 📅 Vista semanal (compactada).
- 📅 Copiar semana completa a semana siguiente (para subs con jornada estable).

## 5. Supervisión read-only (empresa principal)

- ✓ Pantalla `/supervision.html` con **modo global** (todas las subs) + **modo detalle obra**.
- ✓ Lista de subs con estado: al día (verde), con incidencias (amarillo), sin partes esta semana (gris).
- ✓ Click en sub → calendario mensual de esa sub en modo lectura (sin edición).
- ✓ Posibilidad de abrir observación directamente desde la supervisión.
- ✓ Agregado KPIs: horas totales mes, operarios activos, obras con actividad.

## 6. Sistema de observaciones (incidencias)

- ✓ 6 tipos:
  - `missing_worker` (falta operario que debió trabajar).
  - `wrong_quantity` (horas no cuadran).
  - `overtime` (horas extras no autorizadas).
  - `absence` (ausencia no comunicada).
  - `mismatch_price` (precio hora no cuadra).
  - `other` (campo libre).
- ✓ 5 estados: `open → reviewing → corrected → accepted → closed`.
- ✓ Timeline completo con cada transición (quién, cuándo, por qué).
- ✓ Response message cuando la sub corrige.
- ✓ Email automático a la sub al abrir una observación.
- ✓ Auto-detect: si se mete el jornal faltante, la observación pasa de `open` a `corrected` automáticamente.
- ✓ Las observaciones **bloquean** la generación de proforma del mes hasta que se cierran todas.

## 7. Proformas automáticas

- ✓ Generación de proforma mensual desde los jornales del periodo.
- ✓ Agrupación por obra + operario + categoría.
- ✓ Aplicación del precio correcto en cascada:
  1. `project_subcontractor_prices` (obra+sub+categoría específico).
  2. `project_subcontractor_prices` (obra+sub, categoría=null).
  3. `subcontractors.default_jornal_price`.
- ✓ IVA 21 % calculado y mostrado.
- ✓ 8 estados: `draft → submitted → under_review → approved / rejected → claim_period → closed → invoiced`.
- ✓ Diagrama estados implementado como máquina de estados estricta en `ProformaService`.
- ✓ Trazabilidad total: cada línea de proforma referencia `source_type='jornal'` + `source_id` del jornal origen.
- ✓ Edición de líneas sólo en estado `draft`.
- ✓ Posibilidad de añadir líneas manuales (ajustes, materiales).
- ✓ Hard-deny: una sub **nunca** puede aprobar su propia proforma.
- 📅 Numeración automática formato `PROF-2026-10-001`.
- 📅 Firma digital al aprobar (FirmaProfesional API).

## 8. Export PDF proformas

- ✓ Export vía `wkhtmltopdf 0.12.6` con template HTML+CSS custom.
- ✓ Diseño "Blueprint Industrial": tipografía mono + colores sobrios + bloques geométricos.
- ✓ 10 páginas típicas para una proforma mensual estándar:
  - Portada con logo + datos empresa/sub.
  - Resumen (totales por obra).
  - Detalle líneas (hasta ~80 líneas por página).
  - Totales y firmas.
- ✓ Export también a XLSX (biblioteca `PhpSpreadsheet` futuro, hoy CSV).

## 9. Import CSV jornales

- ✓ Plantilla descargable desde el modal.
- ✓ Detección automática:
  - Encoding (UTF-8 con/sin BOM, Latin1).
  - Separador (`;` o `,`).
- ✓ Validación por fila con lista de errores detallada.
- ✓ Auto-crear operarios si el DNI no existe (categoría default `oficial`).
- ✓ Auto-asignar operario a la obra si no lo estaba.
- ✓ `INSERT ... ON DUPLICATE KEY UPDATE` → re-importar actualiza en vez de duplicar.
- ✓ Máximo 1000 filas por import, 2 MB tamaño archivo.

## 10. Dashboard ejecutivo

- ✓ KPIs principales para la empresa:
  - Jornales totales mes actual.
  - Horas totales mes actual.
  - Subs activas.
  - Observaciones abiertas.
  - Proformas pendientes de revisión.
- ✓ 4 gráficos SVG inline (sin dependencias JS):
  - Horas por sub (barras).
  - Evolución mensual últimos 6 meses (línea).
  - Distribución por categoría operario (donut).
  - Observaciones por estado (barras apiladas).

## 11. Reportes mensuales

- ✓ 4 reportes exportables (CSV o JSON):
  - **Horas mensuales**: horas por sub × mes.
  - **Por obra**: horas por obra × sub × mes.
  - **Por categoría**: horas por oficial/peón/cintero × sub.
  - **Horas extras**: jornales > 8h con detalle operario y día.
- 📅 Reportes personalizados (filtros guardables).
- 📅 Exportación a XLSX con gráficos embebidos.

## 12. Billing (Stripe)

- ✓ Integración Stripe LIVE con `Workflow Cloud LLC` como merchant.
- ✓ 4 planes configurados con precios reales (14 €, 48,80 €, 39 €, 89 €).
- ✓ **Trial 30 días gratis** sin tarjeta (checkout sin método de pago primeros 30 d).
- ✓ Cupón introductorio `WORKFORCE_INTRO` (9,90 €/mes × 3 meses).
- ✓ Payment Links usados en landing comercial.
- ✓ Webhook `/api/billing/webhook` procesa 6 tipos de evento.
- ✓ Customer Portal integrado (botón "Gestionar suscripción" → redirect portal.stripe.com).
- ✓ Banner UI con 4 estados:
  - Verde / sin banner: `active`.
  - Naranja: `trialing` + `days_until_trial_end ≤ 7`.
  - Rojo: `past_due`.
  - Neutro: `canceled`.
- ✓ Suspensión soft tras 14 días `past_due`.
- 📅 Facturación anual con 15 % descuento.
- 📅 Redsys como alternativa a Stripe (opcional).

## 13. Autenticación y onboarding

- ✓ Login con 2 tarjetas visuales (**Empresa** / **Subcontrata**).
- ✓ Login por email + password.
- ✓ Session cookie `wfm_sid` HttpOnly + Secure + SameSite=Lax.
- ✓ Forgot password con token temporal (TTL 60 min).
- ✓ Reset password con validación token hash.
- ✓ **Activación por código único**: nueva sub entra en `/activar.html?codigo=XXX` → crea password → acceso inmediato.
- ✓ RGPD compliant: 2 checkboxes separados (términos + privacidad) con guardado IP+UA+timestamp.
- ✓ Switch entre tenants para usuarios multi-rol (ej. gestor que supervisa 2 empresas).

## 14. Design system "Blueprint Industrial"

- ✓ Paleta propia: azul profundo (#1E3A5F), ámbar industrial (#F59E0B), gris técnico (#64748B).
- ✓ Tipografías: `JetBrains Mono` para datos, `Inter` para texto largo.
- ✓ Componentes custom:
  - Botones con sombras diagonales.
  - Cards con bordes dobles.
  - Tablas con rayado sutil tipo plano de obra.
  - Modales con header tipo "ficha técnica".
- ✓ Diferenciación clara vs SaaS genéricos azul-verde.

## 15. Infraestructura y operación

- ✓ `install.sh` idempotente (10-15 min deploy VPS).
- ✓ Backup automático cifrado diario.
- ✓ Multi-tenant físico (BD + usuario MySQL por sub).
- ✓ 6 capas de middleware encadenadas.
- ✓ 148 tests PHPUnit passing (unit + feature + integration).
- ✓ Rate limiting por endpoint.
- ✓ Logs sanitizados (passwords/tokens masked).
- ✓ fail2ban + ufw en producción.

## 16. Features explícitamente NO incluidas (roadmap intencional)

- Nómina (la hace la gestoría con export CSV nuestro).
- CRM clientes.
- Facturación completa (sólo proforma).
- Gestión materiales / compras.
- BIM / planos.
- Control de accesos físicos a obra (puede llegar vía integración NFC).
- Partes de trabajo con fotos.
- Fichaje por geolocalización (planeado para Q2 2027 vía app móvil, opt-in).
