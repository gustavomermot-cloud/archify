# Executive Summary — Workforce Manager

**Workflow Cloud LLC** · Delaware · Octubre 2026

---

## Problema

Las constructoras medianas españolas pierden entre 8 y 12 horas semanales supervisando a sus subcontratas con Excel, WhatsApp y papel. El resultado: 5-15 % de discrepancias mensuales en proformas, disputas de pago, cero trazabilidad cuando falta un operario. Los ERPs sectoriales (Odoo, Sage, A3) cuestan 15-40 K€ de implantación y están diseñados para empresas grandes; las PYMES de construcción quedan desatendidas.

## Solución

**Workforce Manager** es un SaaS multi-tenant construido específicamente para que una constructora principal supervise a sus subcontratas. Dos pantallas, dos roles:

- La **empresa principal** da de alta obras y subcontratas, supervisa el calendario mensual de cada sub en modo lectura, levanta incidencias trazables y aprueba proformas al final del mes.
- Cada **subcontrata** declara jornales día a día en un calendario editable con auto-save, responde incidencias y genera proformas automáticamente desde los jornales del mes.

Las proformas se exportan a PDF profesional listo para firma. Los datos del operario (nombre, DNI, categoría oficial/peón/cintero, precio hora) están en el núcleo del dominio, no son un afterthought.

## Diferenciador clave

**Aislamiento físico multi-tenant real**: cada subcontrata tiene su propia base de datos MariaDB con su propio usuario MySQL y sus propios GRANTS. 14 bases de datos distintas hoy en producción. Si un bug SQL intenta leer datos ajenos, MariaDB lo rechaza a nivel de permisos antes de ejecutar la query. No conocemos ningún competidor español que lo haga así; todos usan multi-tenant lógico (misma BD + columna `tenant_id`).

Esta decisión arquitectónica es imposible de replicar sin reescribir el producto, y es auditable por cualquier cliente que lo pida formalmente.

## Mercado

- **TAM**: 140.000 empresas de construcción en España (INE 2024).
- **SAM**: ~20.000 constructoras medianas que subcontratan de forma habitual.
- **SOM** año 3: 500 clientes = **2,5 %** del SAM.

ARR medio ponderado estimado 550 €/cliente/año (mix 70 % subcontratas 48,80 €, 20 % empresas 39-89 €, 10 % autónomos 14 €).

## Tracción

En producción real desde septiembre 2026:

- Bloc Creatiu SL (Barcelona) como cliente ancla operativo.
- 14 subcontratas reales conectadas, con bases de datos aisladas.
- 222 obras creadas, 50+ operarios, **1.750 jornales históricos** importados.
- 148 tests PHPUnit passing, 15.000 líneas de código PHP, 55+ endpoints API.
- Stripe LIVE integrado (Workflow Cloud LLC Delaware), landing publicada.

Clientes externos de pago: 0 al momento de este documento. Siguiente hito: 10 constructoras del círculo profesional entrando en trial 30 días durante Q1 2027.

## Modelo de negocio

Suscripción SaaS mensual, 4 planes:

| Plan | Precio |
|---|---|
| Autónomo | 14 €/mes |
| Workforce Subcontratas | 48,80 €/mes |
| Empresa Principal | 39 €/mes |
| Pro Constructora | 89 €/mes |

Oferta introductoria: trial 30 días gratis + 9,90 €/mes × 3 meses (cupones `SUBCONTRATA3M` / `AUTONOMO3M`). Margen bruto SaaS objetivo 75-80 % a partir del año 2.

## Equipo

**Gustavo Mermot** — Fundador, desarrollador y operador. 15+ años programando, 10+ años en el sector construcción (Bloc Creatiu SL como jefe de obra). Diseña, construye y vende el mismo producto; ciclo de feedback 24 h con el cliente ancla.

Entidad legal: Workflow Cloud LLC (Delaware). Operación comercial España vía Bloc Creatiu SL.

## Proyección financiera (conservadora)

| Año | Clientes | ARR | Margen |
|---|---|---|---|
| 1 | 50 | 25 K€ | 40 % |
| 2 | 200 | 110 K€ | 70 % |
| 3 | 500 | 300 K€ | 75 % |
| 5 | 2.000 | 1,2 M€ | 77 % |

Break-even operacional alcanzable en año 3 con equipo de 3 personas.

## Ask

Ronda semilla **150-300 K€**, opcional, sólo si acelera go-to-market. Alternativa bootstrap: cash flow operativo desde cliente 10.

Si ronda: 40 % producto, 35 % comercial, 15 % marketing, 10 % infra y legal. Hito a 12 meses: 200 clientes, ARR 110 K€, equipo de 3.

---

**Contacto**: gustavo@bloccreatiu.com · +34 654 162 515
**Landing**: [work-flow.solutions/workforce](https://work-flow.solutions/workforce/)
