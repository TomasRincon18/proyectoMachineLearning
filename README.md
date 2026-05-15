# Proyecto Final ML1: Sistema de Inteligencia Multifuente OSINT

Este repositorio documenta el desarrollo del proyecto final de Machine Learning 1:
un sistema de inteligencia multifuente sobre la escalada regional asociada al
conflicto Irán-Israel-EE. UU., construido exclusivamente con fuentes abiertas,
gratuitas y documentadas.

El objetivo no es construir una plataforma militar ni un sistema de monitoreo en
tiempo real. El objetivo es demostrar criterio de ciencia de datos: conseguir
datos reales, integrarlos en un dataset usable, formular una tarea de machine
learning defendible, comparar modelos y comunicar los resultados en un dashboard
web desplegado.

## 1. Entendimiento del problema

El enunciado propone una pregunta orientadora:

> ¿Hasta qué punto un conjunto de fuentes abiertas y gratuitas permite detectar,
> clasificar o modelar episodios de escalada regional en el conflicto
> Irán-Israel-EE. UU.?

Para convertir esa pregunta amplia en un proyecto ejecutable, necesitamos tomar
decisiones concretas:

- qué fuentes usaremos;
- cuál será la unidad de análisis;
- qué variable objetivo o tarea de ML vamos a resolver;
- qué modelos compararemos;
- cómo mostraremos los hallazgos en un dashboard.

La clave del proyecto será construir evidencia analítica, no solo acumular
gráficas. Cada visualización, feature y modelo debe estar conectado con una
pregunta clara.

## 2. Alcance obligatorio del proyecto

Según el enunciado, el proyecto debe cumplir como mínimo con:

- entre 3 y 5 fuentes de datos;
- al menos 1 fuente textual;
- al menos 1 fuente estructurada u operativa;
- al menos 1 fuente adicional de contexto, movilidad o señal social;
- una pregunta propia de machine learning;
- una unidad de análisis justificada;
- mínimo 3 modelos comparados;
- métricas adecuadas al problema;
- un dashboard web desplegado y accesible por URL;
- documentación clara del proceso, decisiones, sesgos y limitaciones.

## 3. Formulación inicial propuesta

Esta es la formulación de partida. Se validará después de revisar muestras reales
de las fuentes.

**Pregunta de trabajo**

¿Podemos clasificar el nivel de escalada de una ventana región-día combinando
eventos estructurados, cobertura noticiosa y señales contextuales abiertas?

**Unidad de análisis propuesta**

Ventana `region-dia`.

Cada fila del dataset integrado representará una región en una fecha específica.
Esta unidad permite combinar fuentes heterogéneas:

- eventos o incidentes agregados por día y región;
- noticias o menciones textuales agregadas por día y región;
- señales de movilidad, contexto o actividad externa agregadas en la misma
  ventana temporal.

**Tarea principal de ML propuesta**

Clasificación supervisada del nivel de escalada:

- `bajo`;
- `medio`;
- `alto`.

En este proyecto, el nivel de escalada será una categoría analítica que resume la
intensidad relativa de señales observables en una ventana `region-dia`. No será
una alerta militar ni una predicción directa del conflicto. Se construirá a partir
de dimensiones como eventos registrados, severidad aproximada, actores
involucrados, volumen mediático, tono textual, señales contextuales y cambios
respecto a días anteriores.

Interpretación inicial:

| Nivel | Significado operativo |
| --- | --- |
| `bajo` | Actividad limitada o cercana al comportamiento base de la región. |
| `medio` | Aumento moderado de eventos, cobertura o señales contextuales. |
| `alto` | Concentración fuerte de señales, eventos relevantes o anomalías coincidentes. |

El target no debe inventarse manualmente sin justificación. La primera opción será
construirlo a partir de reglas transparentes usando eventos estructurados, conteos
de incidentes, severidad aproximada, intensidad noticiosa u otras variables
observables. Si los datos no soportan esta clasificación, se ajustará la tarea a
regresión, por ejemplo estimar número de eventos o intensidad mediática en la
siguiente ventana.

## 4. Fuentes candidatas

La selección final dependerá de disponibilidad, calidad y facilidad de acceso.
Primero trabajaremos con muestras pequeñas.

| Fuente | Tipo | Rol esperado en el proyecto |
| --- | --- | --- |
| ACLED | Estructurada | Eventos de conflicto, actores, fechas, lugares y tipos de evento. Candidata principal para construir labels o targets. |
| GDELT | Textual/API | Noticias, menciones, tono, volumen mediático y posibles variables NLP. |
| BBC RSS o Al Jazeera RSS | Textual/RSS | Corpus noticioso complementario y contraste editorial. |
| OpenSky | Movilidad/API | Actividad aérea agregada por ventana temporal y zona, si la cobertura es suficiente. |
| NASA FIRMS | Contexto/API | Hotspots o señales térmicas como variable contextual, si aporta información en la zona de estudio. |

Plan inicial de fuentes:

- fuente estructurada: `ACLED`;
- fuente textual principal: `GDELT`;
- fuente textual complementaria: `BBC RSS` o `Al Jazeera RSS`;
- fuente contextual o movilidad: `OpenSky` o `NASA FIRMS`.

## 5. Campos mínimos del dataset integrado

Cada fuente se normalizará hacia un esquema común cuando sea posible:

| Campo | Descripción |
| --- | --- |
| `date` | Fecha normalizada de la observación. |
| `region` | Región o país asociado a la observación. |
| `source` | Fuente original del registro. |
| `lat` / `lon` | Coordenadas cuando existan. |
| `text` | Texto disponible: título, descripción, noticia, post o resumen. |
| `event_type` | Tipo de evento o categoría normalizada. |
| `source_count` | Conteo agregado por fuente en la ventana. |
| `text_volume` | Volumen de documentos o menciones textuales. |
| `sentiment_or_tone` | Tono, sentimiento o proxy textual si la fuente lo permite. |
| `target` | Variable objetivo final: clase de escalada o valor continuo. |

El dataset final no tiene que contener exactamente estos campos, pero cualquier
cambio debe quedar documentado.

## 6. Instalación y ejecución inicial

La Fase 0 deja preparado un entorno base de Python. Desde la raíz del repositorio:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Si alguna fuente requiere credenciales, se debe copiar el archivo de ejemplo y
completar solo las variables necesarias:

```bash
cp .env.example .env
```

Reglas iniciales de trabajo:

- los datos crudos van en `data/raw/`;
- los datos limpios por fuente van en `data/processed/`;
- el dataset integrado va en `data/final/`;
- los scripts deben ejecutarse desde la raíz del repositorio;
- las convenciones del proyecto están en `docs/conventions.md`.

## 7. Plan de trabajo paso a paso

### Fase 0 - Preparación del repositorio

Objetivo: dejar una base reproducible para trabajar.

Tareas:

- crear estructura de carpetas del proyecto;
- definir archivo de dependencias (`requirements.txt` o `pyproject.toml`);
- separar datos crudos, datos procesados, notebooks, scripts, modelos y dashboard;
- documentar convenciones de nombres y ejecución.

Resultado esperado:

- repositorio ordenado y ejecutable por otra persona.

Estado actual:

- estructura de carpetas creada;
- dependencias iniciales definidas en `requirements.txt`;
- variables de entorno documentadas en `.env.example`;
- reglas de exclusión definidas en `.gitignore`;
- convenciones iniciales documentadas en `docs/conventions.md`.

### Fase 1 - Definición analítica

Objetivo: cerrar la primera versión defendible del problema.

Tareas:

- confirmar la pregunta de trabajo;
- justificar la unidad de análisis `region-dia`;
- definir regiones de interés;
- definir período de estudio;
- decidir si la tarea principal será clasificación o regresión;
- escribir cómo se construirá el target.

Resultado esperado:

- documento corto con pregunta, unidad de análisis, target preliminar y supuestos.

Estado actual:

- definición analítica inicial documentada en `docs/problem_definition.md`;
- pregunta de trabajo confirmada de forma preliminar;
- unidad de análisis propuesta: `region-dia`;
- tarea principal propuesta: clasificación multiclase del nivel de escalada;
- período inicial propuesto: `2024-01-01` a `2026-05-08`;
- regiones iniciales propuestas: Irán, Israel/Palestina, Irak, Siria, Líbano,
  Yemen y Golfo Pérsico/Estrecho de Ormuz.

Pendiente de validación en Fase 2:

- disponibilidad real de datos por fuente;
- cobertura por región y fecha;
- reglas exactas para construir el target;
- balance de clases;
- métrica principal final.

### Fase 2 - Validación de fuentes

Objetivo: comprobar que las fuentes sirven antes de construir el pipeline completo.

Tareas:

- descargar muestras pequeñas de cada fuente candidata;
- revisar campos disponibles, cobertura temporal y geográfica;
- evaluar ruido, duplicados, valores faltantes y restricciones de acceso;
- descartar fuentes que no aporten evidencia suficiente;
- dejar documentado qué se intentó y por qué se mantiene o se descarta.

Resultado esperado:

- tabla comparativa de fuentes y decisión final de 3 a 5 fuentes.

Estado actual:

- metodología de validación documentada en `docs/source_validation.md`;
- primera fuente validada: `GDELT`;
- script de validación creado: `src/ingestion/validate_gdelt.py`;
- GDELT queda aceptada provisionalmente como fuente textual/noticiosa y señal
  complementaria de eventos derivados de noticias;
- pendientes: ACLED, RSS noticioso, OpenSky y NASA FIRMS.

Hallazgos clave de GDELT:

- la API DOC devuelve artículos para el 8 de mayo de 2026;
- los archivos crudos GDELT 2.0 de eventos, menciones y GKG existen para el
  período preliminar;
- los archivos de 15 minutos tienen esquema consistente;
- en una muestra de 15 minutos se observaron registros para Irán, Israel,
  Líbano, Siria, Irak y Gaza;
- GDELT debe usarse con cautela porque deriva eventos desde noticias y puede
  reflejar sesgos de cobertura mediática.

### Fase 3 - Ingesta y limpieza

Objetivo: construir scripts reproducibles para obtener y limpiar datos.

Tareas:

- crear scripts de ingesta por fuente;
- guardar datos crudos en `data/raw/`;
- limpiar fechas, coordenadas, texto y categorías;
- estandarizar nombres de columnas;
- guardar datos limpios por fuente en `data/processed/`.

Resultado esperado:

- pipeline inicial capaz de regenerar los datos limpios desde las fuentes.

### Fase 4 - Integración del dataset

Objetivo: convertir fuentes heterogéneas en una tabla principal de modelado.

Tareas:

- agregar cada fuente a la unidad `region-dia`;
- construir variables numéricas, categóricas y textuales;
- unir fuentes por fecha y región;
- crear el target con reglas transparentes;
- revisar balance de clases o distribución del target;
- crear diccionario de datos.

Resultado esperado:

- dataset integrado en `data/final/` y diccionario de variables.

### Fase 5 - Análisis exploratorio EDA

Objetivo: entender el comportamiento de los datos antes de modelar.

Tareas:

- revisar distribuciones por fuente, fecha y región;
- analizar valores faltantes y duplicados;
- visualizar volumen de eventos y noticias en el tiempo;
- comparar regiones y fuentes;
- revisar relación preliminar entre features y target;
- identificar sesgos, huecos de cobertura y limitaciones.

Resultado esperado:

- notebook o reporte de EDA con hallazgos accionables para modelado.

### Fase 6 - Modelado

Objetivo: entrenar y comparar modelos alineados con ML1.

Tareas:

- definir partición de entrenamiento y prueba respetando el tiempo si aplica;
- construir una línea base simple;
- entrenar mínimo 3 modelos;
- comparar representaciones textuales simples como TF-IDF si se usa texto;
- evaluar con métricas coherentes;
- guardar resultados, parámetros y modelos entrenados.

Modelos candidatos para clasificación:

- baseline por clase mayoritaria o regla simple;
- Naive Bayes;
- KNN;
- Logistic Regression;
- Decision Tree o Random Forest como modelo adicional justificado.

Métricas candidatas:

- accuracy como referencia general;
- precision, recall y F1 por clase;
- matriz de confusión;
- análisis de errores por región y fecha.

Resultado esperado:

- comparación clara de modelos y selección justificada del modelo final.

### Fase 7 - Interpretación y análisis de errores

Objetivo: explicar qué aprendió el modelo y dónde falla.

Tareas:

- revisar errores frecuentes;
- analizar si el modelo falla más en ciertas regiones, fechas o clases;
- identificar variables más informativas cuando el modelo lo permita;
- comparar resultados contra la línea base;
- documentar limitaciones metodológicas y de datos.

Resultado esperado:

- sección de resultados interpretables, no solo métricas.

### Fase 8 - Dashboard web

Objetivo: comunicar el sistema de inteligencia multifuente de forma navegable.

Contenido mínimo:

- explicación breve del problema;
- fuentes utilizadas;
- filtros por fecha, región, categoría o fuente;
- visualizaciones exploratorias;
- mapa o vista geográfica si los datos lo permiten;
- resultados del modelo;
- métricas principales;
- interpretación de hallazgos;
- limitaciones del sistema.

Herramientas posibles:

- Streamlit;
- Dash;
- Shiny for Python;
- Next.js.

Resultado esperado:

- dashboard desplegado y accesible por URL.

### Fase 9 - Documentación y entrega final

Objetivo: cerrar el proyecto de forma reproducible y evaluable.

Tareas:

- actualizar instrucciones de instalación y ejecución;
- documentar fuentes, transformaciones y target;
- explicar modelos, métricas y resultados;
- incluir limitaciones y consideraciones éticas;
- preparar presentación final;
- verificar que el dashboard funcione desde la URL pública.

Resultado esperado:

- repositorio completo, dashboard desplegado y narrativa final coherente.

## 8. Estructura esperada del repositorio

```bash
.
├── .env.example             # Plantilla de variables de entorno
├── .gitignore               # Archivos locales que no se suben al repositorio
├── data/
│   ├── raw/                 # Datos crudos por fuente
│   ├── processed/           # Datos limpios por fuente
│   └── final/               # Dataset integrado para modelado
├── docs/                    # Convenciones y documentación técnica
├── notebooks/               # EDA y experimentos iniciales
├── src/
│   ├── ingestion/           # Scripts de descarga o lectura por fuente
│   ├── processing/          # Limpieza, normalización e integración
│   ├── features/            # Construcción de variables y target
│   └── modeling/            # Entrenamiento y evaluación
├── models/                  # Modelos exportados
├── dashboard/               # Aplicación web
├── reports/                 # Figuras, tablas y notas de resultados
├── requirements.txt         # Dependencias del proyecto
└── README.md                # Documentación principal
```

## 9. Riesgos y decisiones a vigilar

- No construir el dashboard antes de tener dataset y tarea de ML.
- No usar más fuentes de las que podamos limpiar y explicar bien.
- No formular un target imposible, ambiguo o desconectado de los datos.
- No confundir volumen de noticias con escalada real sin justificarlo.
- No reportar solo accuracy si las clases quedan desbalanceadas.
- No ocultar sesgos de cobertura, errores de geolocalización o limitaciones de
  fuentes gratuitas.
- No usar embeddings o modelos complejos sin compararlos contra una línea base
  simple.

## 10. Bitácora de decisiones

| Fecha | Decisión | Estado |
| --- | --- | --- |
| 2026-04-30 | Se toma el README como centro de control del proyecto. | Hecho |
| 2026-05-14 | Se define plan de trabajo por fases según el enunciado oficial. | Hecho |
| 2026-05-14 | Se completa la Fase 0 con estructura, dependencias y convenciones iniciales. | Hecho |
| 2026-05-14 | Se completa la primera versión de Fase 1 en `docs/problem_definition.md`. | Hecho |
| 2026-05-15 | Se inicia Fase 2 y se valida GDELT como fuente provisional. | Hecho |
| Pendiente | Validar ACLED con muestra pequeña o documentar bloqueo por credenciales. | Pendiente |
| Pendiente | Validar fuente RSS noticiosa complementaria. | Pendiente |
| Pendiente | Validar fuente contextual o movilidad: OpenSky o NASA FIRMS. | Pendiente |
| Pendiente | Confirmar fuentes finales, regiones y período después de validar datos. | Pendiente |
| Pendiente | Definir target final, umbrales y métrica principal. | Pendiente |

## 11. Criterio de éxito

El proyecto será exitoso si logra responder una pregunta acotada con datos reales,
un dataset integrado y documentado, modelos comparados con rigor, y un dashboard
que permita entender tanto los hallazgos como las limitaciones del sistema.
