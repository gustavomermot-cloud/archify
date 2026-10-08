# Casos de uso — Workforce Manager

> 4 narrativas basadas en flujo real de Bloc Creatiu SL con sus subcontratas (Punjab 701, BLOC, Dicotec, etc.).

---

## Caso 1: Lunes por la mañana — Gustavo da de alta una obra nueva

**Contexto**: Bloc Creatiu acaba de firmar una obra nueva "GOODMAN" (nave logística, 1800 m²) en el polígono de Castellbisbal. Punjab 701 se encargará de la estructura de pladur.

**Actor**: Gustavo (jefe de obra, rol `company_admin` en Bloc Creatiu).

**Flujo**:

1. 09:12 — Gustavo entra en `/dashboard.html` desde el PC de obra. El sistema está logueado desde la semana pasada; cookie de sesión aún válida.
2. Navega a `/obras.html` → click "+ Nueva obra".
3. Rellena:
   - Nombre: `GOODMAN`.
   - Código interno: `BLC-2026-14`.
   - Dirección: `Pol. Les Fallulles, Castellbisbal`.
   - Fecha inicio: `2026-10-15`.
   - Fecha fin estimada: `2027-02-28`.
4. Click "Guardar". Request `POST /api/projects` → 201 Created.
5. En la ficha de la obra recién creada, click "Asignar subcontrata" → selector muestra las 14 subs activas → selecciona `Punjab 701`.
6. Request `POST /api/projects/42/assign` con `{subcontractor_id: 9}` → 201.
7. En la misma ficha, click "Precio acordado" → modal con tabla editable:
   - Oficial: 25 €/h.
   - Peón: 18 €/h.
   - Cintero: 22 €/h.
8. Guardar → 3 filas insertadas en `project_subcontractor_prices`.
9. 09:16 — Gustavo copia el enlace de la obra, se lo manda por WhatsApp al responsable de Punjab 701.

**Tiempo total**: 4 minutos.

**Datos generados**:

- 1 fila en `projects` (master).
- 1 fila en `project_subcontractor_assignments`.
- 3 filas en `project_subcontractor_prices`.
- 2 entradas en `activity_log` (master).

---

## Caso 2: Punjab 701 declara jornales de la semana

**Contexto**: Es viernes tarde. Rajesh, el responsable administrativo de Punjab 701, actualiza los jornales de la semana.

**Actor**: Rajesh (rol `subcontractor_admin` en Punjab 701).

**Flujo**:

1. 17:45 — Rajesh entra en `/login.html`, selecciona tarjeta "Subcontrata", mete credenciales.
2. Session regenerate_id, cookie wfm_sid. Redirect a `/sub/dashboard.html`.
3. El calendario mostrado por defecto es el mes actual (octubre 2026) con el filtro de obra "Todas". Rajesh selecciona "GOODMAN" en el filtro.
4. Aparecen 5 operarios asignados a GOODMAN en el eje Y, los 31 días de octubre en el X. Días pasados muestran jornales ya declarados.
5. Rajesh pulsa en la celda `(Salim Khan, lunes 5)` que está vacía → input numérico inline → teclea `8` → tab.
6. Request `POST /api/jornales` con `{project_id: 42, worker_id: 7, date: '2026-10-05', quantity: 8}`.
7. Middleware chain:
   - Auth ✓.
   - Tenant resolves a `workforce_bloc_punjab701`.
   - CSRF ✓.
   - Permission (`jornal.create` para `subcontractor_admin`) ✓.
   - ProjectAssignment (obra 42 asignada a Punjab) ✓.
8. `JornalService.createOrUpdate()`:
   - Valida que worker 7 pertenece a Punjab (query local).
   - Insert con `ON DUPLICATE KEY UPDATE`.
   - Chequea observaciones abiertas en `(42, 2026-10-05)` → ninguna.
   - Insert en `activity_log`.
   - 201 Created.
9. UI: celda queda en verde con "8". Auto-save a los 800 ms.
10. Rajesh continúa con el resto de celdas de los 5 operarios durante la semana. 25 celdas en 90 segundos.
11. Al terminar, click "Guardar borrador" (opcional, redundante porque hay auto-save). UI muestra toast "Guardado 25 jornales".

**Tiempo total**: 2 minutos para la semana completa de 5 operarios.

**Datos generados**:

- 25 filas en `jornales` (tenant Punjab).
- 25 filas en `activity_log` (tenant Punjab).

---

## Caso 3: Fin de mes — Punjab genera proforma, Bloc Creatiu la aprueba

**Contexto**: 1 de noviembre. Punjab 701 cerró el mes de octubre con 142 jornales en 3 obras distintas.

**Actor**: Rajesh (Punjab) + Gustavo (Bloc Creatiu).

**Flujo lado Punjab**:

1. 09:00 — Rajesh entra en `/sub/proformas.html` → click "+ Generar proforma octubre 2026".
2. Request `POST /api/proformas` con `{period_year: 2026, period_month: 10}`.
3. `ProformaService.generate()`:
   - `SELECT jornales WHERE date BETWEEN '2026-10-01' AND '2026-10-31'` → 142 filas.
   - Chequea `jornal_observations` con status `open` o `reviewing` en el mes → 0 incidencias.
   - BEGIN TX.
   - INSERT en `proformas` con `status='draft'`.
   - Agrupa 142 jornales por `(project_id, worker_id, category)` → 23 líneas distintas.
   - Para cada línea, calcula `priceFor(worker_id, date, project_id)`:
     - Si hay precio específico en `project_subcontractor_prices` → usa ese.
     - Si no, usa `subcontractors.default_jornal_price`.
   - INSERT 23 filas en `proforma_lines` con `source_type='jornal'` + array de `source_id`.
   - UPDATE `proformas` con subtotal = 18.750 €, tax = 3.937,50 €, total = 22.687,50 €.
   - COMMIT.
4. Response 201 → UI muestra modal con detalle: 23 líneas, 142 jornales, 1125 horas, total 22.687,50 €.
5. Rajesh revisa, todo correcto, click "Enviar a Bloc Creatiu".
6. Request `POST /api/proformas/5/submit` → `UPDATE status='submitted'`.
7. 09:04 — Email automático sale hacia Gustavo con asunto "Proforma octubre 2026 de Punjab 701 pendiente de revisión".

**Flujo lado Bloc Creatiu**:

8. 10:20 — Gustavo recibe el email, entra en `/proformas.html`.
9. UI muestra la lista de proformas filtradas por `status=submitted` → aparece la de Punjab octubre.
10. Click → modal detalle con las 23 líneas desglosadas.
11. Gustavo cuadra 3 líneas rápido contra sus propios registros, todo OK.
12. Click "Pasar a revisión" → `status='under_review'` (indica a Punjab que la están viendo).
13. 5 min después, click "Aprobar". Request `POST /api/proformas/5/approve`:
    - `ProformaService.assertNotSubcontractor()` → verifica rol = empresa.
    - `UPDATE status='approved', approved_at=NOW()`.
14. Gustavo click "Exportar PDF". `GET /api/proformas/5/export?format=pdf&subcontractor_id=9`:
    - `ProformaExporter.exportPdf()` → renderiza HTML → `wkhtmltopdf` → 10 páginas, 103 KB.
    - Guarda en `/var/lib/wfm/exports/proformas/punjab701/proforma-5-2026-10.pdf`.
    - Respuesta `Content-Disposition: attachment` → descarga.
15. Gustavo envía el PDF por email a la gestoría de Punjab 701 para que facturen a Bloc Creatiu.

**Tiempo total**: 8 minutos lado sub + 12 minutos lado empresa.

**Antes de Workforce Manager**: 2-4 horas de llamadas, Excel comparado celda a celda, discusiones por 50 € de diferencia.

---

## Caso 4: Discrepancia — Bloc marca incidencia, Punjab corrige

**Contexto**: Durante la supervisión de octubre, Gustavo detecta que el día 15 Salim trabajó 8 h pero Rajesh no lo declaró (olvido).

**Actores**: Gustavo + Rajesh.

**Flujo**:

1. 11:30 — Gustavo entra en `/supervision.html` → selecciona Punjab 701 → calendario mes octubre.
2. Ve que la celda `(Salim, miércoles 15)` está vacía, pero él sabe que estuvo porque le firmó el albarán.
3. Click en la celda vacía → aparece popover "Abrir observación".
4. Rellena:
   - Tipo: `missing_worker`.
   - Mensaje: "Salim trabajó 8h el día 15 según albarán firmado, pero no está declarado".
5. Click "Crear". Request `POST /api/observations` con `{subcontractor_id: 9, project_id: 42, worker_id: 7, date: '2026-10-15', type: 'missing_worker', message: '...'}`.
6. `ObservationService.create()`:
   - Resuelve tenant Punjab.
   - Valida obra pertenece a Bloc Creatiu.
   - INSERT en `jornal_observations` con `status='open'`, `created_by_role='company_supervisor'`.
   - MailService envía email a Rajesh.
   - `activity_log` entry.
7. UI muestra confirmación "Observación #34 creada". Celda ahora aparece amarilla con badge.
8. 14:15 — Rajesh recibe email, entra en `/sub/dashboard.html`. Ve badge amarillo en celda día 15 de Salim.
9. Click en el badge → popover muestra la observación "Salim trabajó 8h...". Opción "Responder".
10. Rajesh responde: "Tienes razón, lo había olvidado. Lo meto ahora". `POST /api/observations/34/respond`:
    - `UPDATE status='reviewing', response_message='...', responded_by, responded_at`.
11. Rajesh cierra el popover, mete `8` en la celda vacía.
12. `JornalService.createOrUpdate()` detecta que hay observation open sobre `(42, 2026-10-15, worker=7)`:
    - INSERT jornal.
    - UPDATE observation `status='corrected'`.
    - `activity_log` doble entry.
13. 14:17 — Gustavo recibe push UI (próxima feature) o email "La observación #34 ha sido corregida".
14. Gustavo entra en `/incidencias.html`, ve la observación en estado `corrected`.
15. Click → modal con timeline:
    - 11:30: Abierta por Gustavo.
    - 14:15: Rajesh respondió.
    - 14:16: Rajesh corrigió (metió jornal).
    - Estado actual: `corrected`.
16. Click "Aceptar" → `UPDATE status='accepted'`.
17. Click "Cerrar" → `UPDATE status='closed', closed_at=NOW()`.
18. La observación desaparece de la lista de pendientes.

**Tiempo total**: 3 minutos entre Gustavo abriendo y cerrando.

**Trazabilidad**: el timeline completo queda guardado permanentemente. Si en diciembre la gestoría pregunta "¿por qué cambiasteis el jornal del 15 de octubre?" → se abre la observación #34 y se ve toda la conversación con timestamps.

---

## Patrón común a los 4 casos

- **Latencia**: todas las acciones son < 500 ms en producción real (medido en Hetzner CX22).
- **Fricción**: minimizada por auto-save, popovers inline, selectores pre-filtrados.
- **Transparencia**: todo queda loggeado en `activity_log` + timeline visible al usuario.
- **Rol-correctness**: ningún usuario puede hacer algo que no corresponde a su rol (hard-denies).
- **Aislamiento**: Punjab nunca ve datos de BLOC o Dicotec, aunque todos vayan al mismo panel.
