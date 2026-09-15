# parcial1_ing_de_datos
# Juan David Giraldo Benjumea
# Pipeline ETL: Ingesta y Procesamiento de Tasas de Cambio (Frankfurter API)

## 1. Propósito
Diseñar e implementar un pipeline de ingeniería de datos reproducible de extremo a extremo: extracción programática desde una API pública del Banco Central Europeo, persistencia raw, transformación analítica, controles de calidad automatizados y serialización columnar curada.

## 2. Fuente de Datos
* **Servicio:** Frankfurter API (datos oficiales de divisas del BCE).
* **Endpoint:** `https://api.frankfurter.app/`
* **Divisa Base:** USD.
* **Divisas Destino:** EUR, GBP, JPY, CAD, BRL.
* **Rango Temporal:** 01/01/2026 al 31/08/2026.
* **Autenticación:** No requerida (API pública abierta).

## 3. Tecnologías Utilizadas
* **Lenguaje:** Python 3.10+
* **Librerías principales:** `requests`, `pandas`, `pyarrow`, `os`, `json`.

## 4. Estructura y Etapas del Pipeline
1. **Extracción y Persistencia Raw:** Petición HTTP GET parametrizada y almacenamiento íntegro de la respuesta JSON en `data/raw/tasas_raw.json`.
2. **Exploración Inicial:** Carga en DataFrame de Pandas e inspección de dimensiones (`shape`), mapeo de tipos (`dtypes`), visualización preliminar (`head`) y detección de valores nulos (`isnull`).
3. **Transformaciones Significativas:**
   - Desanidación de registros del payload JSON.
   - Conversión de marcas de tiempo e indexación por `DatetimeIndex`.
   - Extracción de dimensiones temporales: año (`year`), mes (`month`) y día (`day_name`).
   - Métrica derivada: variación porcentual diaria de EUR frente a USD (`EUR_daily_pct_change`).
   - Agregación estadística mensual con cálculo de promedio, mínimo y máximo para cada divisa.
4. **Validaciones de Calidad (Data Quality Assertions):**
   - Comprobación de unicidad de la clave temporal (`df.index.is_unique`).
   - Verificación de límites de negocio (todas las tasas cambiarias mayores a cero).
   - Control estricto de completitud (cero valores nulos en el dataset curado).
5. **Almacenamiento y Verificación:** Persistencia en `data/processed/tasas_procesadas.parquet` mediante `pyarrow` y validación de lectura con `pd.read_parquet()`.

## 5. Decisiones Técnicas de Almacenamiento
Se seleccionó el formato **Apache Parquet** sobre CSV por las siguientes razones:
* **Almacenamiento Columnar:** Minimiza las lecturas de disco (*I/O throughput*) al consultar únicamente las columnas requeridas para análisis de series de tiempo.
* **Compresión Eficiente:** Reduce el espacio en disco comparado con texto plano (CSV o JSON).
* **Preservación Estricta de Esquema:** Retiene los tipos de datos nativos (`float64`, `datetime64`) sin necesidad de re-parseos ni riesgo de pérdida de precisión.

## 6. Instrucciones de Ejecución
1. Instalar dependencias:
   ```bash
   pip install requests pandas pyarrow
