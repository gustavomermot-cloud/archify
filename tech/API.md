# API REST — Workforce Manager

> Catálogo de endpoints. Base URL: `https://workforce.workflow-cloud.com/api` (dev: `https://workforce.test/api`).

## Convenciones

- Formato: **JSON** request/response.
- Autenticación: **cookie de sesión** (`wfm_sid`) tras `/auth/login`.
- CSRF: header `X-CSRF-Token` **obligatorio en POST/PUT/PATCH/DELETE**. Token obtenido en respuesta de `/auth/login` o `/auth/me`.
- CORS: estricta (same-origin). Para la app móvil futura: JWT separado en `/mobile/*`.
- Paginación: query params `?page=1&per_page=50`. Max per_page 200.
- Filtros: query params nombrados (`?status=active&project_id=5`).
- Errores: HTTP status estándar + body `{ "error": "code_slug", "message": "...", "details": {...} }`.
- Timestamps: ISO 8601 UTC (`2026-10-05T14:30:00Z`).

## Códigos HTTP

| Código | Significado |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content (DELETE) |
| 400 | Validation error |
| 401 | No autenticado |
| 403 | Autenticado pero sin permiso (hard-deny) |
| 404 | No encontrado |
| 409 | Conflicto estado (ej. proforma ya aprobada) |
| 422 | Validación lógica (ej. observaciones abiertas bloquean submit) |
| 423 | Locked (rate limit, tenant suspendido) |
| 500 | Error servidor |

---

## 1. Auth

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| POST | `/auth/login` | público | Body: `{email, password}`. Devuelve user + csrf_token + Set-Cookie. |
| POST | `/auth/logout` | authenticated | Invalida sesión. |
| GET | `/auth/me` | authenticated | Datos user actual + tenant activo + roles. |
| POST | `/auth/forgot-password` | público | Body: `{email}`. Siempre 200 (anti-enumeración). |
| POST | `/auth/reset-password` | público (con token) | Body: `{token, new_password}`. |
| POST | `/auth/activate` | público (con código) | Body: `{code, email, password, full_name, accepted_terms, accepted_privacy}`. |
| POST | `/auth/switch-tenant` | authenticated | Body: `{tenant_id}` para usuarios con múltiples tenants. |

## 2. Users (self-service RGPD)

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/users/me` | authenticated | Perfil completo. |
| PATCH | `/users/me` | authenticated | Cambio nombre, email, password actual. |
| DELETE | `/users/me` | authenticated | **Borrado RGPD**: anonymizes + revoca accesos + log. |
| GET | `/users/me/export` | authenticated | Portabilidad: ZIP con todos sus datos. |

## 3. Projects (obras)

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/projects` | project.read | Lista obras del tenant. Filtros: `?status=active`. |
| GET | `/projects/{id}` | project.read | Detalle. |
| POST | `/projects` | project.create | Body: `{name, code, address, start_date, end_date, notes}`. |
| PATCH | `/projects/{id}` | project.update | Modifica campos. |
| DELETE | `/projects/{id}` | project.delete | Soft delete (status='archived'). |
| POST | `/projects/{id}/assign` | project.assign | Body: `{subcontractor_id}`. Asigna sub a obra. |
| POST | `/projects/{id}/unassign` | project.assign | Body: `{subcontractor_id}`. Desasigna. |

## 4. Subcontractors

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/subcontractors` | subcontractor.read | Lista subs de la empresa. |
| GET | `/subcontractors/{id}` | subcontractor.read | Detalle (hard-deny si sub != self para rol sub). |
| POST | `/subcontractors` | subcontractor.create | Body: `{name, slug, color, default_jornal_price, ...}`. Provision BD + usuario MySQL. |
| PATCH | `/subcontractors/{id}` | subcontractor.update | |
| DELETE | `/subcontractors/{id}` | subcontractor.delete | Soft delete + archiva BD tenant. |
| POST | `/subcontractors/{id}/activation-codes` | subcontractor.create | Genera código único de 7 días. |
| GET | `/subcontractors/{id}/activation-codes` | subcontractor.read | Lista códigos emitidos. |
| GET | `/subcontractors/{id}/access` | subcontractor.read | Usuarios con acceso + roles. |
| POST | `/subcontractors/{id}/access` | subcontractor.update | Invita user existente. |
| DELETE | `/subcontractors/{id}/access/{user_id}` | subcontractor.update | Revoca acceso. |

## 5. Workers (operarios)

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/workers` | worker.read | Lista operarios del tenant. Filtros: `?status=active&project_id=5`. |
| GET | `/workers/{id}` | worker.read | Detalle. |
| POST | `/workers` | worker.create | Body: `{display_name, document_number, category, phone}`. |
| PATCH | `/workers/{id}` | worker.update | |
| DELETE | `/workers/{id}` | worker.delete | Soft delete. |
| POST | `/workers/{id}/assign` | worker.assign | Body: `{project_id}`. |
| POST | `/workers/{id}/unassign` | worker.assign | Body: `{project_id}`. |
| POST | `/workers/import` | worker.create | multipart CSV + `?project_id=` opcional. |

## 6. Jornales

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/jornales` | jornal.read | Query: `?project_id=5&year=2026&month=10`. Devuelve matriz [worker × date]. |
| GET | `/jornales/month` | jornal.read | Vista mes agregada. |
| POST | `/jornales` | jornal.create | Body: `{project_id, worker_id, date, quantity, notes}`. UPSERT. |
| POST | `/jornales/bulk` | jornal.create | Body: `{items: [...]}`. Transaccional. |
| PATCH | `/jornales/{id}` | jornal.update | **DENY para rol company_supervisor** (hard-deny). |
| DELETE | `/jornales/{id}` | jornal.delete | Si jornal no referenciado por proforma. |
| POST | `/jornales/import` | jornal.create | CSV upload con detect encoding. |

## 7. Observations

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/observations` | observation.read | Filtros: `?status=open&project_id=5`. |
| GET | `/observations/{id}` | observation.read | Detalle + timeline. |
| POST | `/observations` | observation.create | Body: `{subcontractor_id, project_id, worker_id, date, type, message}`. |
| POST | `/observations/{id}/respond` | observation.respond | Body: `{response_message}`. Solo sub. status open→reviewing. |
| POST | `/observations/{id}/accept` | observation.transition | Solo empresa. corrected→accepted. |
| POST | `/observations/{id}/close` | observation.transition | Solo empresa. accepted→closed. |
| POST | `/observations/{id}/correct` | observation.transition | Auto desde JornalService al corregir. |

## 8. Proformas

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/proformas` | proforma.read | Filtros: `?status=submitted&period_year=2026&period_month=10`. Multi-tenant si rol empresa. |
| GET | `/proformas/{id}` | proforma.read | Detalle + lines[]. Requiere `?subcontractor_id=X` si rol empresa. |
| POST | `/proformas` | proforma.create | Body: `{period_year, period_month}`. Solo sub. Genera draft. 422 si observations open. |
| PATCH | `/proformas/{id}` | proforma.update | Edita header draft. |
| DELETE | `/proformas/{id}` | proforma.delete | Solo si status=draft. |
| POST | `/proformas/{id}/submit` | proforma.submit | draft→submitted. |
| POST | `/proformas/{id}/review` | proforma.transition | submitted→under_review. Solo empresa. |
| POST | `/proformas/{id}/approve` | proforma.approve | under_review→approved. **DENY si rol sub**. |
| POST | `/proformas/{id}/reject` | proforma.transition | Body: `{reason}`. under_review→rejected. |
| POST | `/proformas/{id}/open-claim` | proforma.transition | approved→claim_period. |
| POST | `/proformas/{id}/close` | proforma.transition | approved/claim_period→closed. |
| POST | `/proformas/{id}/invoice` | proforma.transition | closed→invoiced. |
| GET | `/proformas/{id}/lines` | proforma.read | Líneas detalladas. |
| POST | `/proformas/{id}/lines` | proforma.update | Añade línea manual. Solo draft. |
| PATCH | `/proformas/{id}/lines/{line_id}` | proforma.update | Edita línea. Solo draft. |
| DELETE | `/proformas/{id}/lines/{line_id}` | proforma.update | Solo draft. |
| GET | `/proformas/{id}/export` | proforma.read | `?format=pdf` o `?format=xlsx`. Descarga. |

## 9. Supervision

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/supervision/overview` | supervision.read | Agregado: KPIs todas las subs. Solo empresa. |
| GET | `/supervision/calendar` | supervision.read | Query: `?subcontractor_id=X&project_id=Y&month=10&year=2026`. Read-only. |
| GET | `/supervision/subs` | supervision.read | Lista subs con status (al día / con incidencias / sin partes). |

## 10. Reports

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/reports/monthly-hours` | report.read | Horas por sub × mes. CSV o JSON. |
| GET | `/reports/by-project` | report.read | Horas por obra. |
| GET | `/reports/by-category` | report.read | Horas por categoría (oficial/peón/cintero). |
| GET | `/reports/overtime` | report.read | Jornales > 8h. |

## 11. Billing (Stripe)

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/billing/subscription` | authenticated | Estado suscripción (status, trial_end, days_until_trial_end). |
| POST | `/billing/portal` | authenticated | Crea Billing Portal session. Devuelve URL. |
| POST | `/billing/checkout` | authenticated | Body: `{plan_code}`. Crea Checkout session. |
| POST | `/billing/webhook` | público (verificación firma) | Receptor eventos Stripe. Verifica `Stripe-Signature`. |

Eventos Stripe procesados:

- `checkout.session.completed`
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.paid`
- `invoice.payment_failed`

## 12. Activation (público)

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/activation/{code}` | público | Verifica código válido. 404 si expirado o usado. |
| POST | `/activation` | público | Body: `{code, email, password, full_name, accepted_*}`. Crea user + user_tenant. |

## 13. CSRF

| Método | Path | Permiso | Descripción |
|---|---|---|---|
| GET | `/csrf-token` | authenticated | Devuelve token rotativo para la sesión actual. |

---

## Ejemplo request

```http
POST /api/jornales HTTP/1.1
Host: workforce.workflow-cloud.com
Content-Type: application/json
Cookie: wfm_sid=abc123
X-CSRF-Token: xyz789

{
  "project_id": 42,
  "worker_id": 7,
  "date": "2026-10-05",
  "quantity": 8.0,
  "notes": "turno completo"
}
```

Respuesta:

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "data": {
    "id": 1583,
    "project_id": 42,
    "worker_id": 7,
    "date": "2026-10-05",
    "quantity": 8.0,
    "notes": "turno completo",
    "status": "confirmed",
    "created_at": "2026-10-05T14:32:00Z"
  }
}
```

---

## Rate limiting

| Endpoint | Límite |
|---|---|
| `/auth/login` | 5 fallos / 15 min / (ip, email) |
| `/auth/forgot-password` | 3 / hora / email |
| `/auth/activate` | 10 / hora / IP |
| `/jornales` (POST) | 300 / hora / usuario |
| General | 1000 / hora / usuario |

Exceder → 423 Locked con header `Retry-After`.

---

## Versionado

Sin versión en URL hoy (`/api/`). Si se introduce v2 breaking: `/api/v2/`. Mantenemos v1 1 año tras publicación de v2.
