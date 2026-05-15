# Convenciones del proyecto

Este documento fija reglas mínimas para que el trabajo sea reproducible y fácil de
evaluar.

## Estructura de datos

- `data/raw/`: datos descargados sin modificar. No se editan manualmente.
- `data/processed/`: datos limpios por fuente, con columnas normalizadas.
- `data/final/`: dataset integrado usado para EDA, modelado y dashboard.

Los archivos de datos grandes no deben subirse al repositorio. Si un dataset no se
incluye, debe existir un script o una instrucción clara para reconstruirlo.

## Nombres de archivos

Usaremos nombres descriptivos en minúsculas:

```text
fuente_periodo_estado.ext
```

Ejemplos:

```text
gdelt_2026-04_raw.parquet
acled_2026-04_processed.parquet
region_day_features.parquet
```

## Esquema común mínimo

Cuando una fuente lo permita, se intentará normalizar hacia estas columnas:

- `date`
- `region`
- `source`
- `lat`
- `lon`
- `text`
- `event_type`
- `value`

No todas las fuentes tendrán todas las columnas. Las ausencias deben documentarse
en el diccionario de datos.

## Ejecución

El flujo esperado será:

1. Descargar o cargar datos crudos.
2. Limpiar cada fuente por separado.
3. Integrar a la unidad de análisis.
4. Ejecutar EDA.
5. Entrenar modelos.
6. Levantar dashboard.

Cada script debe poder ejecutarse desde la raíz del repositorio.
