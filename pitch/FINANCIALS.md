# Proyecciones financieras — Workforce Manager

> Todas las cifras son **conservadoras**. No incluyen expansion revenue agresiva ni cross-sell optimista. Supuestos documentados abajo.

---

## 1. Resumen 5 años

| Año | Clientes fin | ARR | Costes | Margen bruto | EBITDA |
|---|---|---|---|---|---|
| **1** | 50 | 25.000 € | 15.000 € | 40 % | -5.000 € |
| **2** | 200 | 110.000 € | 33.000 € | 70 % | +15.000 € |
| **3** | 500 | 300.000 € | 75.000 € | 75 % | +80.000 € |
| **4** | 1.100 | 650.000 € | 160.000 € | 75 % | +230.000 € |
| **5** | 2.000 | 1.200.000 € | 280.000 € | 77 % | +520.000 € |

**Break-even operacional**: año 2 (bootstrap) o año 3 (con ronda y equipo).

---

## 2. Supuestos

### 2.1 Precios efectivos (tras cupón introductorio 3 meses)

| Plan | Precio efectivo mes 4+ | % mix clientes |
|---|---|---|
| Autónomo | 14 € | 15 % |
| Workforce Subcontratas | 48,80 € | 55 % |
| Empresa Principal | 39 € | 20 % |
| Pro Constructora | 89 € | 10 % |

**ARPU medio ponderado**: 46 €/mes = **552 €/año**.

Durante los 3 primeros meses de cada cliente el ARPU efectivo es 9,90 € (cupón). El modelo asume que el 60 % de los que entran al cupón se quedan tras los 3 meses, dando un ARPU blended año 1 de ~35 €/mes.

### 2.2 Churn

- **Mes 1-3 (periodo cupón)**: 10 % mensual (self-select natural).
- **Mes 4-12**: 5 % mensual.
- **Año 2+**: 3 % mensual (3 %/mes = ~30 %/año logo churn).
- **Objetivo año 3**: < 2 %/mes.

### 2.3 Growth

- **Año 1**: +10 clientes/mes a partir del mes 4.
- **Año 2**: +15 clientes/mes.
- **Año 3**: +25-30 clientes/mes.
- **Año 4**: +50 clientes/mes.
- **Año 5**: +75 clientes/mes.

### 2.4 Costes

| Partida | Año 1 | Año 2 | Año 3 | Año 5 |
|---|---|---|---|---|
| VPS + dominio + certificados | 1.200 € | 2.400 € | 6.000 € | 20.000 € |
| Stripe fees (~2 % ARR) | 500 € | 2.200 € | 6.000 € | 24.000 € |
| Email transaccional (Postmark/SES) | 300 € | 800 € | 2.000 € | 8.000 € |
| Backup externo cifrado | 240 € | 600 € | 1.800 € | 6.000 € |
| Infra total COGS | ~2.240 € | ~6.000 € | ~15.800 € | ~58.000 € |
| Salarios (ver sección 3) | 10.000 € | 24.000 € | 55.000 € | 220.000 € |
| Legal + contabilidad + Delaware fees | 2.500 € | 2.500 € | 3.500 € | 5.000 € |
| Marketing (SEO + ads + eventos) | 1.500 € | 10.000 € | 25.000 € | 60.000 € |
| **Total costes** | **~15.000 €** | **~33.000 €** | **~75.000 €** | **~280.000 €** |

Nota: salarios año 1 = 10 K€ asume fundador sin salario (bootstrap) o salario simbólico. Si hay ronda, salario fundador asciende a 36 K€ año 1 y los costes totales suben a ~45 K€.

---

## 3. Equipo

### Escenario bootstrap (sin ronda)

| Año | Fundador | Developer | Comercial | Soporte |
|---|---|---|---|---|
| 1 | Full-time, sin salario | — | — | — |
| 2 | Full-time, 12 K€ | Freelance 0,5 FTE | — | — |
| 3 | Full-time, 30 K€ | Full-time, 35 K€ | Part-time | — |
| 4 | Full-time, 45 K€ | Full-time, 42 K€ | Full-time, 36 K€ + comisiones | Part-time |
| 5 | Full-time, 55 K€ | 2× Full-time, 85 K€ | 2× Full-time, 70 K€ | Full-time, 28 K€ |

### Escenario con ronda semilla 250 K€

| Año | Fundador | Developer | Comercial | Soporte |
|---|---|---|---|---|
| 1 | FT, 36 K€ | FT, 42 K€ | FT, 32 K€ | — |
| 2 | FT, 42 K€ | FT, 45 K€ | FT, 38 K€ | PT |
| 3 | FT, 48 K€ | FT, 48 K€ + senior 55 K€ | FT, 40 K€ + 1 nuevo | FT |

En ambos escenarios, el salario fundador se mantiene por debajo de mercado hasta año 3-4 para extender runway.

---

## 4. Proyección mensual año 1 (detalle)

Supuestos: launch comercial efectivo en mes 2. Datos cumulativos de clientes activos.

| Mes | Nuevos | Churn | Clientes | MRR | ARR |
|---|---|---|---|---|---|
| 1 | 2 | 0 | 2 | 20 € | 240 € |
| 2 | 5 | 1 | 6 | 60 € | 720 € |
| 3 | 8 | 1 | 13 | 130 € | 1.560 € |
| 4 | 10 | 2 | 21 | 420 € | 5.040 € |
| 5 | 10 | 2 | 29 | 820 € | 9.840 € |
| 6 | 10 | 2 | 37 | 1.300 € | 15.600 € |
| 7 | 10 | 2 | 45 | 1.700 € | 20.400 € |
| 8 | 10 | 2 | 53 | 2.000 € | 24.000 € |
| 9 | 10 | 3 | 60 | 2.300 € | 27.600 € |
| 10 | 10 | 3 | 67 | 2.550 € | 30.600 € |
| 11 | 10 | 3 | 74 | 2.800 € | 33.600 € |
| 12 | 10 | 3 | 81 | 3.100 € | 37.200 € |

El año 1 cierra entre **50-80 clientes activos**. ARR proyectado 25-37 K€ (rango por volatilidad del churn).

---

## 5. Sensibilidad: ¿qué pasa si...?

### Escenario pesimista (churn 2x, growth 0,7x)

| Año | Clientes | ARR |
|---|---|---|
| 1 | 25 | 10 K€ |
| 2 | 90 | 48 K€ |
| 3 | 220 | 125 K€ |

Resultado: break-even año 4, requiere runway de 24 meses mínimo.

### Escenario optimista (growth 1,5x, churn constante)

| Año | Clientes | ARR |
|---|---|---|
| 1 | 80 | 42 K€ |
| 2 | 350 | 190 K€ |
| 3 | 900 | 540 K€ |

Resultado: break-even año 2, ronda Serie A viable año 3.

El **escenario base** proyectado en la tabla principal está en el 60 % entre pesimista y optimista.

---

## 6. Uso de fondos (si ronda 250 K€)

| Partida | % | € | Qué compra |
|---|---|---|---|
| Developer senior full-time (12 m) | 25 % | 62.500 € | Acelerar roadmap, descargar a fundador |
| Comercial B2B full-time (12 m) | 25 % | 62.500 € | 50-100 clientes/año conseguidos por outbound |
| Marketing + SEO + ads | 15 % | 37.500 € | Google Ads nicho, SEO content, 2 eventos sectoriales |
| Salario fundador 12 m | 15 % | 37.500 € | Sostenibilidad personal del fundador |
| Infra + VPS + servicios | 5 % | 12.500 € | Hetzner + backup + monitoring + Postmark |
| Legal + RGPD + contratos | 5 % | 12.500 € | Revisión abogado, actualización LOPD, certificados |
| Reserva operativa | 10 % | 25.000 € | Buffer para imprevistos y oportunidades |

Hito objetivo tras 12 meses: **200 clientes activos**, **ARR 110 K€**, equipo de 3 personas, runway reservado 6 meses adicionales.

---

## 7. Métricas de salud del negocio (dashboard semanal)

- MRR
- ARR
- New customers / semana
- Churn rate / mes
- CAC blended
- LTV blended
- LTV/CAC
- CAC payback (meses)
- NPS (encuesta trimestral)
- Tickets soporte / cliente / mes (idealmente < 0,5)
- % clientes con proforma generada mes anterior (health score)
- % clientes con incidencia abierta > 7 días (red flag)

---

## 8. Opciones de salida (horizonte 5-7 años)

Escenarios realistas:

- **Lifestyle business**: 1.000-2.000 clientes, equipo 5-8 personas, dividendos al fundador. Realista sin ronda.
- **Adquisición estratégica**: comprador potencial = ERPs sectoriales (Sage, Odoo, Factorial) o grupos construcción (asesorías laborales que quieran digitalizar). Múltiplo 3-5x ARR = 3-6 M€ a 2.000 clientes.
- **Private equity**: 5.000+ clientes, ARR 3 M€+. Múltiplo 4-6x = 12-18 M€. Requiere crecimiento > 50 % anual.

El plan no asume salida. Prioridad: sostenibilidad y margen operativo.
