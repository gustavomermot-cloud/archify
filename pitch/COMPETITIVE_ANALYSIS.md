# Análisis competitivo — Workforce Manager

## 1. Tabla comparativa

| Dimensión | **Workforce Manager** | Excel compartido | Factorial / Sesame | Odoo Construction | Sage 50 construcción | Zoho People |
|---|---|---|---|---|---|---|
| Precio €/mes (empresa pequeña) | **48,80** | 0 | 70-150 | 200-800 + implantación | 100-300 + licencia | 2,50/user |
| Aislamiento BD físico | **Sí (1 BD/sub)** | — | No (lógico) | No (lógico) | No (lógico) | No (lógico) |
| Multi-sub nativo | **Sí** | No (hojas separadas) | No (addon manual) | Parcial | No | No |
| Idioma español nativo | **Sí** | Sí | Sí | Parcial | Sí | Parcial |
| Categorías obrero construcción (oficial/peón/cintero) | **Sí** | Manual | No | No | Parcial | No |
| Tiempo setup | **15 min** | 0 | 2-5 días | 2-6 semanas | 1-3 semanas | 1-3 días |
| Proformas automáticas desde jornales | **Sí** | No | No | Sí (con configuración) | Sí (con configuración) | No |
| Export PDF proforma profesional | **Sí (wkhtmltopdf)** | No | No | Sí | Sí | No |
| API REST | **Sí (55+ endpoints)** | — | Sí | Sí | Limitada | Sí |
| White-label | **Sí (Pro)** | — | No | Addon pago | No | Addon |
| Pensado para construcción española | **Sí** | — | No | Parcial | Parcial | No |
| Supervisión read-only cross-tenant | **Sí** | No | No | Parcial | No | No |
| Trial sin tarjeta | **Sí (30 d)** | — | Sí | Variable | No | Sí |
| Target empresa | **PYME construcción (10-100)** | Todas | PYME HR | Mid-market multisector | PYME construcción | SMB general |

---

## 2. Análisis competidor por competidor

### 2.1 Excel compartido (status quo)

**Es el competidor real al que vencer.** 70-85 % de las PYMES de construcción españolas siguen aquí.

**Fortalezas**:
- Coste 0 €.
- Flexibilidad total.
- Todos los empleados saben usarlo.

**Debilidades**:
- Sin trazabilidad: nadie recuerda quién editó qué celda.
- Discrepancias de versión cuando se envía por WhatsApp.
- Imposible gestionar 5+ subs sin colapso.
- Cero firma, cero auditoría, cero validación.

**Cómo le ganamos**:
- Oferta 9,90 €/mes × 3 meses → el coste de un café al día durante 3 meses, con ROI inmediato en tiempo recuperado.
- Mensaje: "Si tu Excel ya no te vale, no necesitas un ERP de 30 K€. Prueba esto 30 días gratis."

---

### 2.2 Factorial / Sesame HR / Personio

**Son HR genéricos adaptados a España**. Enfoque: nómina, bajas, vacaciones, control horario de oficina.

**Fortalezas**:
- Marca conocida, 50 M€+ facturados cada uno.
- Integraciones con bancos y gestorías.
- UX pulida.

**Debilidades**:
- No entienden "proforma mensual de subcontrata".
- No tienen multi-sub nativo (requiere crear "empresa" por cada sub → precio multiplicado).
- No tienen categorías construcción (oficial/peón/cintero).
- Fichaje por geolocalización está pensado para oficina, no obra.
- Precio: a 70-150 €/mes por cuenta, una constructora con 5 subs = 500-900 €/mes solo HR.

**Cómo les ganamos**:
- Vertical. Específico. Construcción.
- Precio: 48,80 €/mes por sub vs 70-150 €/mes por "cuenta HR".
- Mensaje: "Factorial es un buen HR genérico. Esto es software de construcción."

---

### 2.3 Odoo Construction (y módulos sectoriales Odoo)

**Es el ERP opensource más usado en España**. Mucho integrador local pero coste real alto.

**Fortalezas**:
- Potente, modular, multi-empresa nativo.
- Integra ventas, compras, nómina, inventario, proyectos.
- Comunidad grande.

**Debilidades**:
- Implantación 2-6 semanas con consultora (**8.000-25.000 €**).
- UX densa, requiere formación.
- Overkill para PYMES de construcción que sólo quieren jornales + proformas.
- No tiene aislamiento físico multi-tenant (usa `company_id` lógico).
- Hosting Odoo online a 24,90 €/usuario/mes se dispara rápido.

**Cómo les ganamos**:
- Setup 15 min vs 6 semanas.
- 48,80 €/mes vs 500+ €/mes efectivos en Odoo + consultoría.
- Mensaje: "¿Vas a pagar 15.000 € para implementar un ERP cuando lo que necesitas es un calendario y una proforma?"

---

### 2.4 Sage 50 construcción (y A3 Construcción)

**ERPs tradicionales con módulos específicos para el sector**. Instalados en gestorías.

**Fortalezas**:
- Reconocidos por la asesoría laboral.
- Integración con Hacienda, Seg. Social, SILCON.

**Debilidades**:
- Software de escritorio con addon web; UX de los 2000.
- Precio 100-300 €/mes + licencias usuario + mantenimiento.
- Pensado para la gestoría, no para el jefe de obra.
- Multi-tenant inexistente: cada empresa es un "fichero" aparte.
- Instalar y configurar lleva días.

**Cómo les ganamos**:
- UX moderna, calendario interactivo, auto-save.
- Precio: 50-80 % más barato.
- API REST real vs exportación CSV mensual.
- Mensaje: "Sage es excelente para la gestoría. Esto es excelente para la obra."

**Integración futura**: exportación CSV compatible con Sage/A3 para que ambos convivan. No competimos con la gestoría, le damos los datos limpios.

---

### 2.5 Zoho People / BambooHR

**HR genéricos internacionales**. Baratos pero descontextualizados.

**Fortalezas**:
- Precio por usuario bajo.
- Suite completa (CRM, People, Books).

**Debilidades**:
- Castellano traducido, no nativo.
- Nada específico de construcción española.
- No multi-sub nativo.
- Soporte en inglés/india, con horario fuera de EU.

**Cómo les ganamos**:
- Soporte en español, mismo huso horario.
- Vocabulario sectorial correcto.
- Datos en servidor EU/España (RGPD sin fricción).
- Mensaje: "No es un software global adaptado. Está hecho aquí."

---

### 2.6 Soluciones sectoriales específicas españolas

Hay 4-5 soluciones pequeñas (típicamente startup catalana o valenciana, 1-5 empleados) que atacan el mismo nicho:

- **Obralia** (gestión documental obra, no jornales).
- **Nalanda** (control acceso + jornada, orientado grandes obras).
- **CoordiPlan** (seguridad y salud, no gestión).
- Otras tipo "WorkerApp" / "SmartSite" sin masa crítica.

**Cómo les ganamos**:
- Ninguna tiene multi-tenant físico (verificable pidiendo documentación técnica).
- Precio comparable o inferior.
- Producto más terminado (148 tests, 20 pantallas, Stripe LIVE operativo).

---

## 3. Diferenciadores clave (resumen)

1. **Único con aislamiento físico multi-tenant real**. Cada subcontrata tiene su BD y usuario MySQL. Esto no es marketing; es demostrable con un `GRANTS` listado.
2. **Diseñado específicamente para construcción española**. Categorías oficial/peón/cintero/encargado no son un campo "custom field": son parte del modelo nativo.
3. **Precio imbatible en su segmento**. 9,90 €/mes introductorio, 48,80 €/mes recurrente. Un ERP sectorial pequeño empieza en 150-200 €/mes.
4. **Setup 15 minutos vía `install.sh`**. Un VPS, un comando, Let's Encrypt automático, fail2ban preconfigurado. No hay implantador, no hay consultora.
5. **Supervisión cross-tenant real**. La empresa principal ve el calendario agregado de todas sus subs en una pantalla, con hard-denies que impiden que una sub vea datos de otra.
6. **Proformas automáticas con exportación PDF profesional**. 10 páginas estilo Blueprint, listas para firma. Factorial/Zoho no lo hacen; Odoo lo hace pero tras 6 semanas de configuración.

---

## 4. ¿Qué podría pasar si un competidor copiara?

El escenario que más preocuparía:

- **Factorial o similar** decide atacar construcción con un vertical dedicado.
  - Les llevaría 12-18 meses tener el vocabulario sectorial correcto.
  - No pueden reescribir su arquitectura a multi-tenant físico (lock-in irreversible).
  - Su precio base ya es 2-3x el nuestro; difícilmente bajarían.
  - Reacción esperada: subirían un addon "construcción" caro.

- **Un ERP sectorial pequeño** copia la UX y el pricing.
  - Riesgo real, mitigación: velocidad de iteración, boca-oreja, red Bloc Creatiu.
  - Diferenciador: aislamiento físico sigue siendo defendible técnicamente.

- **Un desarrollador freelance** vende "una cosa parecida" por 25 €/mes.
  - Pasa en cualquier mercado SaaS. Nuestra diferencia: 148 tests passing, Stripe LIVE, cumplimiento RGPD documentado, soporte real. Un freelance solo no sostiene eso.

---

## 5. Nuestra tesis competitiva en una frase

> **"Factorial para HR, Sage para gestoría, Odoo para ERP. Workforce Manager para la obra."**
