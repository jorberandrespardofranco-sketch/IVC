# Construcción del Índice de Vulnerabilidad Climática-Urbana integrando variables ambientales y socioeconómicas mediante ciencia de datos

Repositorio con el desarrollo replicable del modelamiento de *machine learning* de la tesis de **Maestría en Ciencia de Datos** de la Pontificia Universidad Javeriana Cali: un modelo **LightGBM** que estima el **Índice de Vulnerabilidad Climática-Urbana (IVC)** a nivel de **manzana urbana** en Bogotá D.C. a partir del Censo Nacional de Población y Vivienda 2018 (DANE) y variables climáticas, y su **transferencia espacial** a Medellín y Cali. El flujo se desarrolló en **Google Colab** y **AWS (SageMaker, Glue, S3)**.

| | |
|---|---|
| **Autores** | Jorber Andrés Pardo Franco · Alexandra Ortiz Rodríguez · Andrés Felipe Ospina Nagles |
| **Directora** | Yady Tatiana Solano-Correa |
| **Programa** | Maestría en Ciencia de Datos · Facultad de Ingeniería y Ciencias |
| **Institución** | Pontificia Universidad Javeriana Cali · 2026 |
| **Lenguaje** | Python |

## Resultados principales

| Evaluación | R² | MAE | RMSE |
|---|---|---|---|
| LightGBM, hold-out 90/10 (test) | 0.8778 | 0.0170 | 0.0276 |
| LightGBM, validación cruzada 10-fold (test) | 0.8757 ± 0.0058 | 0.0170 ± 0.0004 | 0.0271 ± 0.0008 |
| LightGBM, train (diagnóstico de sobreajuste) | 0.8900 | 0.0164 | 0.0255 |

Comparación de modelos con PyCaret en SageMaker (CV, ordenado por R²): **LightGBM 0.8856**, Extra Trees 0.8795, Random Forest 0.8794, Gradient Boosting 0.8566, Decision Tree 0.7423.

La brecha train–test (ΔR² ≈ 0.012) indica sobreajuste bajo. Detalle en `docs/metodologia/SELECCION_EVALUACION_MODELO_IVC.pdf`.

## Datos

- **Unidad de análisis:** manzanas urbanas de Bogotá (n = 43,408).
- **Fuentes:** Censo Nacional de Población y Vivienda 2018 (DANE), precipitación media satelital 2000–2018 y temperatura, unificadas con geometría catastral/DANE mediante *spatial join* punto-en-polígono (ver `docs/metodologia/Informe_Tecnico_Spatial_Join.pdf`).
- **Variable objetivo:** `ivc_n`.
- **Fuga de datos (data leakage):** se eliminaron `ivc_formula`, `categoria_ivc` y `perfil_vulnerabilidad`, derivadas directamente del objetivo (62 → 58 columnas).

Descripción de cada archivo en [`data/README.md`](data/README.md).

## Estructura del repositorio

```
.
├── data/
│   ├── raw/              # base consolidada original (43,408 × 67)
│   ├── processed/        # Bogotá listo para modelar; Medellín y Cali filtrados
│   └── synthetic/        # dataset sintético para probar el despliegue
├── notebooks/
│   ├── 01_eda_bogota.ipynb
│   ├── 02_pipeline_aws_sagemaker_pycaret.ipynb
│   ├── 03_modelado_final_prediccion_aws.ipynb
│   └── 04_seleccion_evaluacion_modelo_colab.ipynb
├── models/               # modelo serializado (.joblib)
├── outputs/
│   ├── bogota/           # gráficos y dashboard de métricas
│   └── medellin_cali/    # predicciones consolidadas, análisis y mapas HTML
├── docs/                 # documentación técnica y capítulos (aws, colab, metodología, despliegue)
├── requirements.txt
├── requirements-aws.txt
└── LICENSE
```

## Flujo metodológico

1. **Integración espacial** (`docs/metodologia/`): cruce punto-en-polígono entre mediciones y manzanas DANE (EPSG:4326).
2. **EDA** (`01_eda_bogota.ipynb`): limpieza, selección de variables, correlación, multicolinealidad (VIF), outliers y segmentación geográfica.
3. **Modelado en AWS** (`02_` y `03_`): ETL con AWS Glue/S3, comparación de algoritmos con PyCaret en SageMaker, ajuste de hiperparámetros y predicción sobre manzanas de Bogotá.
4. **Selección y evaluación en Colab** (`04_`): LightGBM, partición estratificada, K-Fold (k=10), análisis de sobreajuste e importancia de variables, y exportación del modelo.
5. **Transferencia espacial a Medellín y Cali** (`docs/metodologia/Protocolo_Transferencia_Espacial_IVC.pdf`): entrenamiento sin `LATITUD`/`LONGITUD` para evitar deriva espacial, inferencia sobre las bases locales y re-acoplamiento geográfico para mapas.
6. **Despliegue** (`docs/despliegue/Arquitectura_Despliegue_MLOps_IVC.pdf`): modelo serializado con `joblib` (preprocesador + LightGBM).

## Cómo replicar

### Google Colab (notebooks 01 y 04)
1. Abre el notebook en Colab y sube a `/content/` los archivos que usa (por ejemplo `data/processed/bogota_ivc_thesis_ready.csv`; el notebook 01 usa `data/raw/base_final.csv`, que en el código aparece como `base final final.csv`).
2. Instala dependencias: `pip install -r requirements.txt`.
3. Ejecuta las celdas en orden.

> Los notebooks conservan las rutas originales de Colab (`/content/...`). Si los ejecutas en local, ajusta esas rutas a `data/...`.

### AWS SageMaker (notebooks 02 y 03)
1. Usa un kernel con **Python 3.10** (PyCaret no es compatible con 3.12).
2. Instala: `pip install -r requirements-aws.txt`.
3. Sustituye los *buckets* de S3 de los notebooks (`s3://databogota/...`, `s3://bogotamanzanas/...`) por los tuyos y sube los CSV de `data/processed/`.

### Usar el modelo serializado

```python
import joblib, pandas as pd

modelo = joblib.load("models/modelo_ivc_despliegue_top20.joblib")
X = pd.read_csv("data/synthetic/dataset_sintetico_despliegue_ivc.csv")
# Usa las mismas columnas/orden con las que se entrenó el modelo
# predicciones = modelo.predict(X[columnas_del_modelo])
```

> Seguridad: nunca cargues archivos `.joblib`/`.pkl` de fuentes que no sean de confianza.

## Limitaciones conocidas

- El MAPE reportado en el notebook 04 no es interpretable (valores del IVC cercanos a cero inflan el error porcentual); se recomienda reportar MAE, RMSE y R².
- Las predicciones para Medellín y Cali no tienen verdad terreno en este repositorio; son una transferencia del modelo entrenado en Bogotá.

## Licencia

Código bajo licencia MIT (ver `LICENSE`). Los datos derivados del DANE y de las fuentes climáticas están sujetos a los términos de uso de sus proveedores.

## Cómo citar

> Pardo Franco, J. A., Ortiz Rodríguez, A., & Ospina Nagles, A. F. (2026). *Construcción del índice de vulnerabilidad climática-urbana integrando variables ambientales y socioeconómicas mediante ciencia de datos* [Tesis de maestría, Pontificia Universidad Javeriana Cali]. Código disponible en GitHub.
