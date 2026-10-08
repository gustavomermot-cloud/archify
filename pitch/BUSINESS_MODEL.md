# Modelo de negocio — Workforce Manager

## 1. Modelo

**SaaS puro, suscripción mensual recurrente**, cobro automático con tarjeta vía Stripe. Sin set-up fees. Sin consultoría obligatoria. Sin contratos anuales obligatorios (opcional con descuento 15 %).

Entidad facturadora: **Workflow Cloud LLC** (Delaware, USA). Facturas en EUR, cumple IVA español vía MOSS para residentes UE.

---

## 2. Planes de precios

| Plan | Precio | Operarios | Obras activas | Features clave |
|---|---|---|---|---|
| **Autónomo** | **14 €/mes** | hasta 2 | 1 | Calendario, 1 proforma/mes, soporte email |
| **Workforce Subcontratas** | **48,80 €/mes** | ilimitados | ilimitadas | Calendario multi-obra, proformas automáticas, PDF export, incidencias, soporte email |
| **Empresa Principal** | **39 €/mes** | — (supervisión) | ilimitadas | Supervisión read-only de subs, aprobación proformas, 4 reportes mensuales |
| **Pro Constructora** | **89 €/mes** | ilimitados | ilimitadas | Todo lo anterior + reportes avanzados + API + white-label parcial + soporte prioritario |

**Facturación**: la constructora paga Empresa (39 €) y cada subcontrata paga su propio plan (14 € ó 48,80 €). El escenario típico: una constructora con 5 subs genera **39 + 5 × 48,80 = 283 €/mes** combinados en la plataforma.

---

## 3. Oferta introductoria (lanzamiento 2026-2027)

- **Trial 30 días** gratis sin tarjeta hasta minuto 29.
- Durante el trial se pide tarjeta para activar el cupón introductorio:
  - **SUBCONTRATA3M**: 9,90 €/mes × 3 meses (ahorro 116,70 €).
  - **AUTONOMO3M**: 9,90 €/mes × 3 meses (ahorro 12,30 €).
- Pasados los 3 meses, cobro automático al precio normal del plan.
- Objetivo: reducir fricción adopción. Un propietario de subcontrata prueba 4 meses efectivos por ~30 € totales.

Esta oferta deja de aplicarse automáticamente tras 2027-Q2 cuando tengamos los primeros 100 clientes de pago.

---

## 4. Unit economics

Supuestos conservadores:

| Concepto | Valor |
|---|---|
| ARPU medio ponderado | 46 €/mes = **552 €/año** |
| Churn mensual año 1 | 5 % |
| Churn mensual año 2+ | 3 % |
| LTV medio (año 2+) | 46 / 0,03 = **~1.530 €** |
| CAC humano (red directa) | 50-100 € |
| CAC medio target año 2 (ads + partner) | 150 € |
| **LTV/CAC** | **~10x** |
| Payback período | 3-4 meses |
| Margen bruto SaaS | 75-85 % (restando Stripe fees + VPS + soporte) |

El target LTV/CAC > 3x es ya razonable; a 10x el modelo es muy sostenible siempre que el churn no se desboque.

### Breakdown costes por cliente

| Coste | % de ARPU |
|---|---|
| Stripe fees (1,5 % + 0,25 € tx) | ~3 % |
| Hosting proporcional (VPS compartido 100 clientes/VPS) | ~2 % |
| Soporte email (tier 1) | ~5 % |
| Transaccionales (email, SMS futuro) | ~1 % |
| **Total COGS** | **~11 %** |
| **Margen bruto** | **~89 %** |

Añadiendo salarios base equipo mínimo (3 personas), el margen operacional cae a 20-30 % en año 1-2 y sube a 50 %+ a partir de año 3.

---

## 5. Revenue mix proyectado

Año 2 estimado (200 clientes):

| Segmento | Clientes | % Mix | ARPU | ARR |
|---|---|---|---|---|
| Autónomo | 30 | 15 % | 168 € | 5 K€ |
| Workforce Subcontratas | 110 | 55 % | 586 € | 64 K€ |
| Empresa Principal | 40 | 20 % | 468 € | 19 K€ |
| Pro Constructora | 20 | 10 % | 1.068 € | 21 K€ |
| **Total** | **200** | **100 %** | **546 €** | **110 K€** |

El plan Workforce Subcontratas (48,80 €) es el motor del ARR: 55-60 % del revenue, 90 % de las "activaciones por boca-oreja".

---

## 6. Upsells y expansion revenue

Oportunidades para ARPU > base:

- **Subcontratas adicionales** sobre Empresa Principal: 15 €/mes por sub extra tras la 5ª.
- **Reportes avanzados** (gráficos personalizados, exportación datos): +15 €/mes.
- **Integración ERP** (Contaplús, A3, Sage): +20 €/mes.
- **White-label completo** (logo, dominio propio): +50 €/mes.
- **Soporte prioritario** (SLA 4 h): +25 €/mes.
- **Firma digital integrada en proformas**: +10 €/mes.
- **Backup extendido 7 años**: +10 €/mes (cumplimiento fiscal).

Target expansion revenue: 20 % del ARR en año 3.

---

## 7. Cobro, dunning y churn

- **Stripe Billing** con reintentos automáticos tarjeta rechazada: 3 intentos en 7 días.
- **Smart retries** activados: Stripe reintenta en horarios óptimos según historial.
- Si tras 7 días sigue sin cobrar → `past_due` → banner rojo en UI → email día 5.
- Día 14 sin cobrar → suspensión soft (login OK, panel read-only, no se pueden crear jornales).
- Día 30 sin cobrar → cancelación automática + conservación datos 90 días adicionales para recuperación.

Dunning email flow:

| Día | Acción |
|---|---|
| 0 | Factura emitida |
| +1 (fallo) | Email: "no pudimos cobrar, actualiza tu tarjeta" |
| +3 | Reintento 1 Stripe |
| +5 | Reintento 2 Stripe + email recordatorio |
| +7 | Reintento 3 Stripe + email "último aviso" |
| +14 | Suspensión soft + email |
| +30 | Cancelación + email con enlace recuperación |
| +120 | Borrado datos tras aviso |

---

## 8. Métricas financieras clave (KPIs)

Tracked mensualmente desde día 1:

- **MRR** (Monthly Recurring Revenue)
- **ARR** (Annual Recurring Revenue = MRR × 12)
- **New MRR** (nuevos clientes del mes)
- **Expansion MRR** (upsells)
- **Churn MRR** (bajas)
- **Net MRR change** = New + Expansion - Churn
- **Logo churn %** (clientes dados de baja / total)
- **Gross revenue retention** (% ARR retenido excluyendo expansion)
- **Net revenue retention** (% ARR retenido incluyendo expansion; target > 100 %)
- **CAC payback period** (meses hasta recuperar CAC vía ARPU)
- **Cash burn mensual** (si hay ronda) o **cash flow operativo** (si bootstrap)

---

## 9. Modelo bootstrap vs. financiado

### Escenario A — Bootstrap (preferido si ronda no llega)

- 0 € capital externo.
- Año 1: 20-30 clientes → MRR 1 K€ → cash flow break-even cuando MRR > costes operativos fundador (zero-salario fase 1).
- Equipo: fundador sólo primeros 18 meses. Freelancer PHP a tiempo parcial desde mes 12.
- Objetivo año 3: 300 clientes, MRR 15 K€, salario fundador + 1 persona.

### Escenario B — Ronda semilla 150-300 K€

- Ronda semilla valoración pre-money ~1,5-2 M€ (dilución 10-15 %).
- Equipo mes 1 tras cierre: fundador + 1 developer + 1 comercial B2B.
- Año 1: 100 clientes → MRR 4,5 K€.
- Año 2: 400 clientes → MRR 18 K€.
- Año 3: 1.000 clientes → MRR 45 K€ → siguiente ronda opcional.

El escenario B es 3-4x más rápido pero asume que la ronda cierra; el A es el fallback real.
