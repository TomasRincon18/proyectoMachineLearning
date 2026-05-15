# Validación de fuentes

Este documento registra la Fase 2 del proyecto. La validación se hará fuente por
fuente, con muestras pequeñas, antes de construir pipelines completos o descargar
datos masivos.

## Criterios de validación

Cada fuente se evaluará con los mismos criterios:

| Criterio | Pregunta de control |
| --- | --- |
| Acceso | ¿La fuente es pública, gratuita y accesible desde el entorno de trabajo? |
| Reproducibilidad | ¿Se puede consultar con un script o URL documentada? |
| Cobertura temporal | ¿Cubre el período preliminar `2024-01-01` a `2026-05-08` o una parte útil? |
| Cobertura geográfica | ¿Permite filtrar o inferir regiones del proyecto? |
| Campos útiles | ¿Aporta texto, eventos, fecha, región, coordenadas, actores, tono o señales agregables? |
| Calidad | ¿Hay duplicados, ruido, campos faltantes o sesgos claros? |
| Integración | ¿Puede agregarse a la unidad `region-dia`? |
| Riesgo | ¿Tiene límites técnicos, éticos, legales o metodológicos relevantes? |
| Decisión | ¿Se acepta, se deja en observación o se descarta? |

## Flujo de validación por fuente

1. Revisar documentación oficial o primaria.
2. Probar acceso con una consulta pequeña.
3. Guardar una muestra cruda en `data/raw/` cuando sea razonable.
4. Extraer un resumen técnico de campos, filas y cobertura.
5. Evaluar si la fuente aporta a la pregunta de trabajo.
6. Registrar decisión provisional.

## Estado resumen

| Fuente | Tipo esperado | Estado | Decisión provisional |
| --- | --- | --- | --- |
| GDELT | Textual / eventos noticiosos | Validada inicialmente | Aceptada provisionalmente |
| ACLED | Eventos estructurados | Pendiente | Pendiente |
| BBC RSS / Al Jazeera RSS | Texto noticioso | Pendiente | Pendiente |
| OpenSky | Movilidad aérea | Pendiente | Pendiente |
| NASA FIRMS | Contexto satelital | Pendiente | Pendiente |

## Fuente 1: GDELT

### Rol esperado

GDELT puede aportar al proyecto como fuente textual y, potencialmente, como fuente
de eventos noticiosos:

- volumen de noticias relacionadas con regiones y actores;
- títulos, URLs, idiomas y países de origen de medios;
- menciones y tono mediante datasets GDELT 2.0;
- eventos derivados de noticias, útiles como señal complementaria.

### Documentación revisada

- Página oficial de datos GDELT: https://www.gdeltproject.org/data.html
- API DOC 2.0: https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/
- Codebook GDELT Event Database V2.0: https://data.gdeltproject.org/documentation/GDELT-Event_Codebook-V2.0.pdf
- Codebook GDELT Global Knowledge Graph V2.0: https://data.gdeltproject.org/documentation/GDELT-Global_Knowledge_Graph_Codebook-V2.pdf

### Prueba de acceso

Se validarán dos superficies:

1. `GDELT DOC 2.0 API`, para artículos y cobertura textual reciente.
2. Archivos crudos `GDELT 2.0`, para eventos, menciones y GKG por intervalos de
   15 minutos.

Fecha de prueba inicial:

```text
2026-05-08
```

Consulta textual inicial:

```text
(Iran OR Israel)
```

### Hallazgos preliminares

Fecha de ejecución local:

```text
2026-05-15
```

Script usado:

```bash
python3 src/ingestion/validate_gdelt.py
```

Archivos generados localmente:

- `data/raw/gdelt_doc_2026-05-08_sample.json`
- `data/raw/gdelt_events_20260508000000_sample.json`
- `data/raw/gdelt_mentions_20260508000000_sample.json`
- `data/raw/gdelt_gkg_20260508000000_sample.json`
- `reports/tables/gdelt_validation_summary.md`

Estos archivos están en rutas ignoradas por git porque son muestras de datos y
salidas de validación. La evidencia principal queda resumida aquí.

#### Resultado DOC API

Consulta:

```text
https://api.gdeltproject.org/api/v2/doc/doc?query=(Iran OR Israel)&mode=artlist&format=json&maxrecords=10&startdatetime=20260508000000&enddatetime=20260508235959
```

Resultado:

- acceso exitoso para el 8 de mayo de 2026;
- 10 artículos retornados;
- idiomas observados: chino, hebreo, lituano, malayalam, serbio y urdu;
- países de origen de fuentes observados: China, India, Israel, Lituania,
  Pakistán, Serbia y Taiwán;
- dominios observados: `cna.com.tw`, `informer.rs`, `jang.com.pk`, `lrt.lt`,
  `maariv.co.il`, `madhyamam.com`, `military.china.com`, `rtv.rs`, `udn.com`.

Observación técnica:

- la API DOC aplicó límite de tasa durante las pruebas exploratorias;
- el mensaje recibido fue: `Please limit requests to one every 5 seconds`;
- por esta razón, cualquier pipeline futuro debe incluir pausas, reintentos y
  consultas pequeñas.

#### Resultado archivos crudos GDELT 2.0

Se validaron archivos del intervalo:

```text
2026-05-08 00:00:00 UTC
```

| Archivo | URL | Filas leídas | Columnas esperadas | Columnas observadas |
| --- | --- | ---: | ---: | ---: |
| Eventos | `http://data.gdeltproject.org/gdeltv2/20260508000000.export.CSV.zip` | 2742 | 61 | 61 |
| Menciones | `http://data.gdeltproject.org/gdeltv2/20260508000000.mentions.CSV.zip` | 4152 | 16 | 16 |
| GKG | `http://data.gdeltproject.org/gdeltv2/20260508000000.gkg.csv.zip` | 1369 | 27 | 27 |

También se comprobó que existen archivos crudos para el inicio del período
preliminar:

```text
2024-01-01 00:00:00 UTC
```

Esto sugiere que los archivos crudos GDELT 2.0 pueden cubrir el período
`2024-01-01` a `2026-05-08`, aunque descargar todo el histórico completo sería
costoso y debe evitarse sin filtros o estrategia incremental.

#### Cobertura regional en la muestra

En el archivo de eventos de 15 minutos del 8 de mayo de 2026 se encontraron 280
filas con códigos asociados a regiones candidatas del proyecto.

Desglose por código de país/región observado:

| Código | Región aproximada | Conteo |
| --- | --- | ---: |
| `IR` | Irán | 189 |
| `IS` | Israel | 62 |
| `LE` | Líbano | 10 |
| `SY` | Siria | 10 |
| `IZ` | Irak | 7 |
| `GZ` | Gaza | 2 |

No se observaron hits de Yemen en este intervalo específico. Esto no implica
ausencia general de cobertura; solo describe una muestra de 15 minutos.

#### Campos útiles observados

GDELT Events aporta:

- fecha (`SQLDATE`, `DATEADDED`);
- actores (`Actor1Name`, `Actor2Name`, códigos de país y tipo);
- tipo de evento (`EventCode`, `EventBaseCode`, `EventRootCode`, `QuadClass`);
- intensidad aproximada (`GoldsteinScale`);
- menciones y fuentes (`NumMentions`, `NumSources`, `NumArticles`);
- tono promedio (`AvgTone`);
- geografía (`ActionGeo_FullName`, `ActionGeo_CountryCode`, latitud, longitud);
- URL fuente (`SOURCEURL`).

GDELT Mentions aporta:

- evento vinculado (`GLOBALEVENTID`);
- fecha de evento y fecha de mención;
- fuente de mención;
- URL o identificador del documento;
- tono del documento mencionado;
- confianza de extracción.

GDELT GKG aporta:

- identificador de documento;
- fuente común;
- temas;
- ubicaciones;
- personas y organizaciones;
- tono (`V2Tone`);
- imagen compartida y otros metadatos.

#### Riesgos y limitaciones

- GDELT es una fuente derivada de noticias, no un registro directo de hechos.
- Puede duplicar eventos cuando múltiples medios reportan lo mismo.
- El volumen mediático puede reflejar interés editorial, idioma o cobertura, no
  necesariamente escalada real.
- La API DOC tiene límites de tasa y no debe consultarse agresivamente.
- El dominio `https://data.gdeltproject.org` presentó error de certificado en
  esta máquina; el acceso por `http://data.gdeltproject.org` funcionó para
  archivos crudos.
- Los archivos crudos de GDELT 2.0 están particionados cada 15 minutos; descargar
  todo el período sin estrategia generaría demasiado volumen.
- La fecha del archivo, la fecha de mención y `SQLDATE` no siempre representan lo
  mismo. Para `region-dia` se debe definir con cuidado cuál fecha se usará.

### Decisión provisional

GDELT queda **aceptada provisionalmente** como fuente textual/noticiosa y como
señal complementaria de eventos derivados de noticias.

Uso recomendado:

- usar `GDELT DOC API` para validar consultas textuales y ejemplos de artículos,
  respetando límites de tasa;
- usar archivos crudos `GDELT 2.0 Events`, `Mentions` y `GKG` para construir
  variables agregadas por `region-dia`;
- no usar GDELT como única fuente para construir el target de escalada;
- contrastar sus señales con una fuente estructurada independiente como ACLED.

Variables candidatas para Fase 4:

- `gdelt_event_count`;
- `gdelt_num_mentions`;
- `gdelt_num_sources`;
- `gdelt_num_articles`;
- `gdelt_avg_tone`;
- `gdelt_goldstein_mean`;
- `gdelt_negative_event_count`;
- `gdelt_doc_volume`;
- `gdelt_source_country_count`;
- `gdelt_theme_count`.
