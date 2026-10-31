# Datos

| Archivo | Filas × Columnas | Descripción |
|---|---|---|
| `raw/base_final.csv` | 43,408 × 67 | Base consolidada de Bogotá: variables censales DANE 2018 (`TP*`), geometría (`AREA`, `Shape_*`), coordenadas y variables climáticas, antes de la limpieza. |
| `processed/bogota_ivc_thesis_ready.csv` | 43,408 × 62 | Base lista para modelar (incluye `ivc_n`, variable objetivo). Contiene 3 columnas que generan fuga de datos y se eliminan antes de entrenar: `ivc_formula`, `categoria_ivc`, `perfil_vulnerabilidad`. |
| `processed/df_medellin_ivc_filtrado.csv` | 14,328 × 21 | Variables de entrada para inferencia en Medellín. |
| `processed/df_cali_ivc_filtrado.csv` | 15,658 × 21 | Variables de entrada para inferencia en Cali. |
| `synthetic/dataset_sintetico_despliegue_ivc.csv` | 500 × 21 | Datos sintéticos para probar el flujo de despliegue. No corresponden a manzanas reales. |

Las predicciones para Medellín y Cali están en `../outputs/medellin_cali/predicciones_ivc_medellin_cali_consolidado.csv` (`ID_CATASTRAL`, `LATITUD`, `LONGITUD`, `IVC_PREDICHO`, `CIUDAD`).

## Diccionario de variables (parcial)

| Variable | Significado |
|---|---|
| `COD_DANE_A` / `CD_LC_CM` | Identificador de la manzana |
| `LATITUD`, `LONGITUD` | Coordenadas (WGS84, EPSG:4326) |
| `AREA` | Área de la manzana |
| `TVIVIENDA` | Total de viviendas |
| `ppt_mean_2000_2018` | Precipitación media 2000–2018 |
| `ivc_n` | Variable objetivo: IVC |

Las variables `TP*` provienen del Censo Nacional de Población y Vivienda 2018 del DANE; su definición exacta está en la documentación oficial del DANE. **Pendiente:** completar este diccionario con la definición de cada `TP*` usada en el modelo.
