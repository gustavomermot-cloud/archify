# Archify — Workforce Manager

Documentación arquitectónica y de producto de **Workforce Manager**, el SaaS multi-tenant de **Workflow Cloud** para que empresas constructoras supervisen jornales, incidencias y proformas de sus subcontratas.

- Landing comercial: [work-flow.solutions/workforce](https://work-flow.solutions/workforce/)
- Oferta subcontratas: [work-flow.solutions/workforce/oferta-subcontratas.php](https://work-flow.solutions/workforce/oferta-subcontratas.php)
- Panel app (vía Cloudflare Tunnel): [work-flow.solutions/workforce/panel/](https://work-flow.solutions/workforce/panel/)
- Repo de código: privado

---

## Resumen 60 segundos

**Workforce Manager** separa radicalmente **Empresa Principal** (constructora) y sus **Subcontratas** (equipos de obra). Cada subcontrata trabaja con sus propios operarios y jornales en una **base de datos aislada físicamente** (1 BD MariaDB + 1 usuario MySQL por sub); la empresa supervisa en modo lectura, levanta incidencias trazables y aprueba proformas mensuales.

**En producción hoy** (octubre 2026):

- 14 subcontratas reales activas en Bloc Creatiu SL (Barcelona).
- 222 obras, 50+ operarios, 1.750 jornales históricos.
- Stripe LIVE integrado (Workflow Cloud LLC Delaware).
- 148 tests PHPUnit passing · 55+ endpoints API · 15K líneas PHP.

Stack: **PHP 8.3 + MariaDB 10.11 + Apache + vanilla JS, sin frameworks**.

---

## Navegación de la documentación

### Pitch e inversión

- [`pitch/EXECUTIVE_SUMMARY.md`](pitch/EXECUTIVE_SUMMARY.md) — Resumen 2 páginas (problema, solución, mercado, tracción, ask).
- [`pitch/PITCH_DECK.md`](pitch/PITCH_DECK.md) — Pitch estilo "slides en Markdown", 16 slides.
- [`pitch/MARKET_ANALYSIS.md`](pitch/MARKET_ANALYSIS.md) — TAM/SAM/SOM, ICP, pain points cuantificados.
- [`pitch/BUSINESS_MODEL.md`](pitch/BUSINESS_MODEL.md) — Precios, unit economics, upsells, dunning.
- [`pitch/COMPETITIVE_ANALYSIS.md`](pitch/COMPETITIVE_ANALYSIS.md) — Comparativa vs Excel, Factorial, Odoo, Sage, Zoho.
- [`pitch/FINANCIALS.md`](pitch/FINANCIALS.md) — Proyecciones conservadoras 5 años, uso de fondos.
- [`pitch/ROADMAP.md`](pitch/ROADMAP.md) — Hitos Q1-Q4 2027.

### Técnico

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — Capas lógicas, resolución tenant, flujos core.
- [`DATA_FLOW.md`](DATA_FLOW.md) — 7 secuencias detalladas end-to-end.
- [`tech/DATABASE.md`](tech/DATABASE.md) — Esquema master + tenant, 14 migraciones, backups.
- [`tech/API.md`](tech/API.md) — Catálogo de los 55+ endpoints agrupados.
- [`tech/SECURITY.md`](tech/SECURITY.md) — Modelo multi-tenant físico, 6 capas middleware, RGPD, GRANTS.
- [`tech/DEPLOYMENT.md`](tech/DEPLOYMENT.md) — `install.sh` VPS, Docker, Cloudflare Tunnel, CI/CD.

### Producto

- [`product/FEATURES.md`](product/FEATURES.md) — Catálogo completo con estado (producción / beta / roadmap).
- [`product/USE_CASES.md`](product/USE_CASES.md) — 4 casos de uso narrativos con datos reales.
- [`product/SCREENS.md`](product/SCREENS.md) — 20 pantallas con rol y descripción.

---

## Diferenciador clave

**Aislamiento físico multi-tenant real**: cada subcontrata tiene su propia base de datos MariaDB con su propio usuario MySQL y sus propios GRANTS. 14 bases de datos distintas en producción hoy. Si un bug SQL intenta leer datos ajenos, MariaDB lo rechaza a nivel de permisos antes de ejecutar la query.

```
mysql> SELECT * FROM workforce_bloc_bloc.workers;
ERROR 1142 (42000): SELECT command denied to user 'wf_tenant_punjab701'@'127.0.0.1'
mysql> SHOW DATABASES;
+--------------------------+
| Database                 |
+--------------------------+
| information_schema       |
| workforce_bloc_punjab701 |
+--------------------------+
```

Para el usuario de Punjab, el resto del sistema no existe. Esta decisión arquitectónica es imposible de replicar sin reescribir el producto, y es auditable por cualquier cliente que lo pida formalmente.

---

## Stack técnico

| Capa | Tecnología | Versión |
|---|---|---|
| Backend | PHP | 8.3 |
| Base de datos | MariaDB | 10.11 |
| Web server | Apache 2.4 + mod_rewrite + mod_ssl | 2.4.58 |
| Frontend | HTML5 + CSS vanilla + JS vanilla (ES2020) | — |
| PDF | wkhtmltopdf | 0.12.6 |
| Testing | PHPUnit | 10.5 |
| Pagos | Stripe PHP SDK | 16.2 |
| Tunnel dev | Cloudflare Tunnel | 2026.3 |

**Sin frameworks** (ni Laravel ni Symfony). PHP vanilla con PSR-4 autoload y PDO prepared statements reales (`EMULATE_PREPARES=false`).

---

## Métricas producto (octubre 2026)

| Métrica | Valor |
|---|---|
| Empresas (companies) | 2 |
| Subcontratas activas | 14 |
| Operarios | 50+ |
| Obras (projects) | 222 |
| Jornales históricos | 1.750 |
| Tests PHPUnit | 148 passing |
| Líneas de código PHP | ~15.000 |
| Páginas UI | 20 |
| Endpoints API | 55+ |
| Migraciones SQL | 14 |

---

## Modelo de negocio (resumen)

4 planes mensuales:

| Plan | Precio |
|---|---|
| Autónomo | 14 €/mes |
| Workforce Subcontratas | 48,80 €/mes |
| Empresa Principal | 39 €/mes |
| Pro Constructora | 89 €/mes |

Oferta introductoria: **30 días trial gratis** + **9,90 €/mes × 3 meses** (cupones `SUBCONTRATA3M` / `AUTONOMO3M`).

Detalle: ver [`pitch/BUSINESS_MODEL.md`](pitch/BUSINESS_MODEL.md) y [`pitch/FINANCIALS.md`](pitch/FINANCIALS.md).

---

## Despliegue rápido

```bash
# VPS Ubuntu 22+
git clone <repo> workforce-manager
cd workforce-manager/deploy/vps
./install.sh --domain workforce.midominio.com --email admin@midominio.com
```

**10-15 minutos**: Apache + MariaDB + PHP + wkhtmltopdf + Let's Encrypt + ufw + fail2ban + backup diario.

Detalle completo en [`tech/DEPLOYMENT.md`](tech/DEPLOYMENT.md).

---

## Contacto

**Gustavo Mermot** — Fundador
Workflow Cloud LLC · Delaware
gustavo@bloccreatiu.com · +34 654 162 515

---

## Licencia

Workforce Manager (código) es **software privado propiedad de Workflow Cloud LLC** (Delaware, USA · 2026).

Esta documentación arquitectónica (repo `archify`) se publica bajo [Creative Commons BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Ver [`LICENSE`](LICENSE).
