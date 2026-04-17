# TP Final · EcoBici CABA 2024

**Análisis de Datos · Carrera de Especialización en Inteligencia Artificial · FIUBA · 1B 2026 · Grupo 7**

Trabajo práctico integrador de la materia *Análisis de Datos*. Se realiza el análisis exploratorio completo del dataset público de EcoBici (sistema de bicicletas compartidas de la Ciudad Autónoma de Buenos Aires) correspondiente al año 2024, se plantea un problema de Machine Learning supervisado, y se deja el dataset preparado para la etapa de modelado.

## Integrantes

- Carmen María Rodríguez Pastrano
- Federico Agustín Fernández
- Claudio Marcelo Imberlina
- María Teresa Mallaupoma León
- Matías Guido Bovio

## Descripción del problema

A partir del dataset de recorridos realizados en EcoBici durante 2024, se plantea un problema de **clasificación multiclase supervisada**: predecir la categoría de duración de un recorrido *antes de que comience*, usando únicamente información disponible al momento de inicio del viaje.

**Variable objetivo** `tipo_viaje`, obtenida discretizando la duración en tres clases con fundamento operativo:

| Clase | Rango | Criterio |
|---|---|---|
| **Corto** | < 10 min | Uso rápido intra-barrial |
| **Mediano** | 10 – 30 min | Dentro del tramo gratuito del sistema |
| **Largo** | > 30 min | Excede el tramo gratuito · recreativo o largo recorrido |

**Valor operativo:** el sistema puede anticipar si un viaje va a exceder el período gratuito, recomendar el tipo de bici adecuado (mecánica vs eléctrica) y distribuir la flota en función de la demanda predicha.

## Dataset

**Fuente:** [Portal de Datos Abiertos de Buenos Aires](https://data.buenosaires.gob.ar/dataset/bicicletas-publicas) — EcoBici Recorridos Realizados 2024

| Métrica | Valor |
|---|---|
| Filas crudas | 3.559.283 |
| Columnas originales | 17 |
| Filas post-filtro | 3.250.461 |
| Rango temporal | Enero – Diciembre 2024 |
| Estaciones únicas | ~395 |

El CSV crudo (~800 MB) no se incluye en el repo. El notebook descarga el archivo automáticamente mediante `gdown` desde un mirror de Google Drive en la primera celda de la Sección 1.

## Estructura del repositorio

```
TP_Final_Grupo7_EcoBici/
├── README.md                                           ← este archivo
├── requirements.txt                                    ← dependencias Python
├── .gitignore                                          ← ignora CSVs y artefactos
├── TP_Grp7_V1_ecobici_presentation_ready.ipynb         ← notebook principal
└── Presentacion_Grupo7_Final.pptx                      ← presentación de defensa (6 slides)
```

## Cómo ejecutar

**Requisitos:** Python 3.10 o superior y `pip` instalado.

```bash
# Clonar el repositorio
git clone <url-del-repo>
cd TP_Final_Grupo7_EcoBici

# Crear entorno virtual (recomendado)
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows

# Instalar dependencias
pip install -r requirements.txt

# Abrir el notebook
jupyter notebook TP_Grp7_V1_ecobici_presentation_ready.ipynb
```

Al correr el notebook completo (de arriba a abajo) se generan los 4 archivos CSV exportados en el mismo directorio: `X_train.csv`, `X_test.csv`, `y_train.csv`, `y_test.csv` (listos para cargar directamente en un notebook de modelado).

Tiempo de ejecución aproximado en una laptop estándar: **5 – 8 minutos** (descarga inicial del dataset incluida).

## Contenido del notebook

El notebook está organizado en 11 secciones que reflejan el pipeline completo de preparación del dataset:

| § | Sección | Cubre |
|---|---|---|
| 1 | Carga del dataset | Descarga con `gdown` · `read_csv` con manejo de CSV malformado |
| 1.1 | Primer vistazo | `head` · `describe` · clasificación de las 17 variables en 4 familias funcionales |
| 2 | Limpieza inicial | Conversión de tipos · filtro de duración (3 métodos evaluados: Dominio / IQR / Winsor) · boxplot antes/después |
| 2.3 | Análisis integral de faltantes | Tipificación MCAR/MAR/MNAR por variable · decisiones de tratamiento |
| 3 | Feature engineering | Encoding cíclico (hora, mes) · distancia Haversine · discretización del target |
| 4 | EDA visual complementario | Mapa de estaciones · distribución de distancias · top 15 estaciones · distribución del target · duración por hora |
| 5 | Selección de features | Mutual Information como filtro · planteo del problema de ML supervisado |
| 6 | Split | 70/30 stratified con `stratify=y` |
| 7 | Distribución del target | Verificación del balance de clases (26 / 56 / 18 %) · decisión razonada de no rebalancear |
| 8 | Encoding de categóricas | Label Encoding · One-Hot Encoding |
| 9 | Escalado | StandardScaler con `fit` solo en train (sin data leakage) |
| 10 | Verificación final | Shapes · tipos · nulos |
| 11 | Reducción de dimensionalidad | Consolidación: MI como filtro + cíclico y Haversine como extracción · evaluación de PCA como alternativa general |
| 12 | Exportación | Generación de los 4 CSV del dataset final |

## Decisiones técnicas destacadas

**Problema de ML.** Se plantea como **clasificación multiclase** en lugar de regresión sobre la duración cruda. La discretización neutraliza los outliers extremos (bicis no devueltas) sin pérdida de información relevante, y produce un modelo más robusto y operativamente útil.

**Tratamiento de outliers.** Se evaluaron tres enfoques (criterio de dominio, IQR de Tukey, Winsor p5/p95) y se aplicó el filtro por dominio (30 s < duración < 24 h) porque los umbrales son físicamente significativos. IQR y Winsor habrían eliminado viajes largos válidos que son justamente la clase minoritaria `Largo`.

**Valores faltantes.** Tipificados según Little & Rubin:
- `genero` (0,34 %) como MNAR → eliminación de filas (imputar introduciría sesgo).
- `fecha_destino_recorrido` (0,10 %) como MAR → no se usa como feature (se calcula después del viaje, sería data leakage).
- `modelo_bicicleta` y coordenadas → MCAR, sin tratamiento especial.

**Feature engineering.** Encoding cíclico sin/cos para `hora` y `mes` (preserva topología circular); distancia Haversine como feature geométrica que condensa 4 coordenadas en 1 escalar interpretable.

**Balance de clases.** Se documenta la decisión de **no aplicar SMOTE ni otros rebalanceos**: el desbalance es moderado (ratio 3:1), `stratify=y` preserva proporciones en el split, y los modelos candidatos downstream (RF, XGBoost, LogReg) soportan `class_weight='balanced'` que es más eficiente que generar datos sintéticos.

**Reducción de dimensionalidad.** Se aplican dos técnicas de extracción de features con justificación de dominio — encoding cíclico (23→2, 11→2) y Haversine (4→1) — más Mutual Information como método de filtro. Se evalúa PCA como alternativa general y se concluye justificadamente no aplicarlo (dimensionalidad final ya baja, features continuas ya ortogonales, binarias violan supuestos de PCA, modelos de árbol downstream no se benefician).

## Resultados del pipeline

**Dataset exportado** listo para modelado:

| Archivo | Contenido | Shape aprox. |
|---|---|---|
| `X_train.csv` | Features de entrenamiento escaladas | 2.275.322 × 16 |
| `X_test.csv` | Features de test escaladas | 975.139 × 16 |
| `y_train.csv` | Target de entrenamiento | 2.275.322 × 1 |
| `y_test.csv` | Target de test | 975.139 × 1 |

Las features exportadas incluyen: encoding cíclico de tiempo (4 columnas), `es_fin_de_semana` (1), distancia Haversine (1), coordenadas de origen y destino (4), `modelo_bicicleta` label-encoded (1), y dummies de género y día de semana (5 columnas post-OHE con `drop_first=True`).

## Próximos pasos (fuera del alcance del TP)

El modelado en sí queda fuera del alcance del TP de Análisis de Datos. La etapa siguiente incluiría:

- Entrenamiento y comparación de modelos base: **Logistic Regression**, **Random Forest** y **XGBoost** con `class_weight='balanced'`.
- Métrica principal: **F1-score macro** para priorizar la clase minoritaria `Largo`.
- Análisis de *feature importance* para interpretabilidad.
- Evaluación del impacto operativo: ¿con qué precisión puede el sistema anticipar viajes que exceden el tramo gratuito?

## Tecnologías

Python 3.10+ · pandas · numpy · matplotlib · seaborn · scikit-learn · scipy · folium · gdown · Jupyter

## Licencia

Uso académico. Dataset bajo licencia del Gobierno de la Ciudad Autónoma de Buenos Aires (Datos Abiertos).
