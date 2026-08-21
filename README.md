# Genie Code Workshop — Banco Popular

Workshop práctico de **Databricks Genie Code** adaptado para el equipo técnico de datos de **Banco Popular (Colombia)**. Cubre 4 tracks de ~105 minutos cada uno usando datos sintéticos bancarios modelados sobre **8 regionales de Colombia**, con la **Libranza** como producto insignia.

> Datos 100% sintéticos. Moneda: COP. Regulador de referencia: **Superintendencia Financiera de Colombia (SFC)**, **SARLAFT** y **Ley 1581 (Habeas Data)**.

---

## Tracks disponibles

| Track | Descripción | Duración |
|---|---|---|
| ⚙️ Data Engineering | Pipelines PySpark, ingesta medallion de transacciones, reconciliación core bancario, Jobs nocturnos, Knowledge Assistant de cartera | ~105 min |
| 📊 BI & Analytics | SQL desde lenguaje natural, DAX→SQL, Metric Views (NPL/mora), Genie Spaces para riesgo, dashboards, Genie Supervisor | ~105 min |
| 🧠 Data Science & ML | Scoring crediticio, MLflow, Model Serving, applyInPandas por regional/segmento, detección de sospechosas (SARLAFT) con FMAPI | ~105 min |
| 🛡️ Data Governance | DQ regulatorio, framework configurable, CLS/RLS para datos financieros, Data Academy, Data Dictionary para auditoría | ~105 min |

---

## Tres formas de usar el workshop

Este repositorio soporta **tres modos de entrega**, todos con el mismo contenido:

### 1) Databricks Free Edition (o cualquier workspace)
1. Genera los datos con el notebook (`generate_workshop_data.py`) — ver *Preparación* abajo.
2. Los participantes trabajan los prompts en notebooks con **Genie Code** activo.
3. Muestra las instrucciones abriendo `frontend/index_static.html` en el navegador, o desplegando la app (modo 2).

### 2) Databricks App (FastAPI)
Despliega la app de instrucciones interactivas en **Databricks Apps** (ver *Paso 7*). Backend FastAPI + frontend React single-page servido desde `frontend/`.

### 3) HTML autocontenido (abrir en el explorador)
`frontend/index_static.html` es un **único archivo autocontenido** — datos y logos embebidos (data-URI), sin backend ni conexión. Ábrelo con doble clic en cualquier navegador. Ideal para compartir por correo o proyectar sin infraestructura.

---

## Modelo de datos sintético

Esquema `workshop.gold` con 5 tablas:

| Tabla | Filas aprox. | Descripción |
|---|---|---|
| `dim_clientes` | ~2,400 | Clientes con segmento, regional, credit_score, perfil de riesgo, KYC |
| `dim_sucursales` | ~136 | Sucursales/agencias/corresponsales por regional |
| `fact_transacciones` | ~150K | Transacciones por canal (App, Sucursal, Cajero, Web, Corresponsal) |
| `fact_cartera_creditos` | ~32K | Cartera por producto (Libranza, Consumo, Hipotecario, Vehículo, Comercial, Tarjeta, Microcrédito) con DPD y mora |
| `fact_kpis_diarios` | ~6,800 | KPIs diarios por regional (cartera, NPL, desembolsos, provisiones) |

### Regionales incluidas

| Código | Regional | Clientes | Sucursales |
|---|---|---|---|
| BOG | Bogotá D.C. y Cundinamarca | 500 | 28 |
| ANT | Antioquia | 380 | 22 |
| CAR | Costa Caribe | 320 | 18 |
| VAL | Occidente (Valle del Cauca) | 290 | 16 |
| SAN | Santanderes | 270 | 15 |
| EJE | Eje Cafetero | 250 | 14 |
| CEN | Centro Oriente | 220 | 13 |
| SUR | Sur (Nariño, Cauca, Putumayo) | 170 | 10 |

Las tablas incluyen **~382 defectos de calidad intencionados** para el track de Governance.

---

## Preparación del ambiente (paso a paso)

Estos pasos los ejecuta el facilitador **antes del workshop**. Tiempo estimado: 30-45 minutos.

### Paso 1 — Subir el notebook generador de datos
1. Descarga `generate_workshop_data.py` de este repositorio.
2. En Databricks: **Workspace** → carpeta compartida (ej. `/Shared/workshop/`) → **Import** → sube el archivo (se reconoce como notebook).

### Paso 2 — Ejecutar el notebook generador
1. **Compute** → usa **Serverless** (recomendado en Free Edition) o crea un cluster (Runtime 15.x ML o superior para el track de Data Science).
2. Abre `generate_workshop_data`, asigna el compute y haz **Run all** (▶▶).
3. Al terminar debe mostrar las 5 tablas `workshop.gold.*` creadas.

> El notebook es idempotente — seguro de re-ejecutar.

### Paso 3 — Verificar tablas en Unity Catalog
**Catalog** → `workshop` → `gold` → confirma las 5 tablas con datos.

### Paso 4 — Permisos a los participantes
```sql
GRANT USE CATALOG ON CATALOG workshop TO `workshop_users`;
GRANT USE SCHEMA ON SCHEMA workshop.gold TO `workshop_users`;
GRANT SELECT ON ALL TABLES IN SCHEMA workshop.gold TO `workshop_users`;
```
> En Free Edition (single user) puedes omitir este paso.

### Paso 5 — Habilitar Foundation Model API
Requerido para los steps con FMAPI (Knowledge Assistant, AI Functions, detección SARLAFT). Verifica que el endpoint `databricks-claude-sonnet-4` esté disponible.

### Paso 6 — Verificar que Genie Code está activo
Abre un notebook y confirma el botón ✨ **Genie Code** (arriba a la derecha). Si no aparece: **Settings → Feature Preview → Databricks Assistant**.

### Paso 7 — Desplegar la app de instrucciones (opcional)

**Opción A — Databricks Apps (UI):**
1. **Apps** → **Create app** → **Custom** (FastAPI).
2. Sube este código al workspace y apunta al path.
3. **Deploy** → comparte la URL.

**Opción B — CLI:**
```bash
databricks workspace import_dir . /Workspace/Users/<tu-usuario>/banco-popular-genie-workshop --overwrite
databricks apps deploy banco-popular-genie-workshop \
  --source-code-path /Workspace/Users/<tu-usuario>/banco-popular-genie-workshop
```

**Opción C — Solo HTML:** comparte `frontend/index_static.html` (no requiere despliegue).

### Checklist final
- [ ] Las 5 tablas existen en `workshop.gold` con datos
- [ ] Participantes con permisos `SELECT` (o single-user en Free)
- [ ] Foundation Model API responde
- [ ] Botón ✨ Genie Code visible en notebooks
- [ ] App desplegada **o** `index_static.html` listo para compartir
- [ ] Compute activo (serverless o cluster) para evitar cold start

---

## Correr la app localmente (opcional)

```bash
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
# abre http://localhost:8000
```

---

## Estructura del proyecto

```
banco-popular-genie-workshop/
├── app.yaml                    # Configuración Databricks Apps
├── main.py                     # Backend FastAPI
├── requirements.txt            # Dependencias Python
├── generate_workshop_data.py   # Genera workshop.gold.* (ejecutar una vez)
├── data/
│   └── tracks.json             # Contenido de los 4 tracks (pasos, prompts, FAQs)
└── frontend/
    ├── index.html              # App React (single-page, usa el backend)
    ├── index_static.html       # Versión autocontenida (datos + logos embebidos)
    └── img/                     # Logos e íconos
```

---

## Notas para el facilitador
- Los datos son **100% sintéticos** — no contienen información real de Banco Popular ni de sus clientes.
- La **Libranza** tiene menor mora estructural en los datos (descuento de nómina) — buen contraste vs. Consumo en los análisis.
- El track de Data Science requiere ML Runtime (XGBoost, MLflow) si no usas serverless ML.
- Los steps con FMAPI muestran una advertencia visible — ten un notebook de respaldo con el output esperado.
- `index_static.html` funciona 100% offline: útil si el proyector no tiene acceso al workspace.
