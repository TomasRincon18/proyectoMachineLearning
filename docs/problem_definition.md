# Definición analítica del proyecto

Este documento corresponde a la Fase 1 del proyecto. Su objetivo es fijar una
primera versión defendible del problema antes de descargar datos de forma masiva
o construir el dashboard.

Las decisiones de este documento son preliminares: deben validarse con muestras
reales en la Fase 2.

## Pregunta de trabajo

¿Podemos clasificar el nivel de escalada de una ventana región-día combinando
eventos estructurados, cobertura noticiosa y señales contextuales abiertas sobre
el conflicto Irán-Israel-EE. UU.?

## Versión operativa de la pregunta

Dada una región y una fecha, el sistema intentará estimar si esa ventana presenta
un nivel de escalada `bajo`, `medio` o `alto`, usando variables derivadas de:

- eventos estructurados de conflicto;
- volumen y tono de noticias;
- señales complementarias de contexto o movilidad.

La salida no debe interpretarse como una predicción militar ni como una alerta en
tiempo real. Es una clasificación analítica construida con fuentes abiertas, útil
para explorar patrones y límites de los datos.

## Unidad de análisis

La unidad de análisis principal será:

```text
region-dia
```

Cada fila del dataset final representará una región observada durante un día.

Esta unidad se elige porque:

- permite integrar fuentes con granularidades distintas;
- reduce ruido frente a registros individuales de noticias o eventos;
- facilita construir agregados interpretables;
- permite usar modelos de clasificación vistos en ML1;
- puede representarse de forma clara en un dashboard con filtros de fecha y
  región.

## Regiones iniciales de interés

La selección inicial busca cubrir el núcleo del conflicto y zonas regionales
relevantes sin hacer el problema inmanejable.

| Región | Justificación inicial |
| --- | --- |
| Irán | Actor central del conflicto y posible origen o destino de eventos. |
| Israel / Palestina | Actor central y zona de alta cobertura noticiosa. |
| Irak | Zona regional conectada con dinámicas de seguridad y actores armados. |
| Siria | Zona de actividad regional y presencia de actores vinculados al conflicto. |
| Líbano | Relevante por su proximidad y por dinámicas asociadas al norte de Israel. |
| Yemen | Relevante por incidentes regionales y dimensión marítima. |
| Golfo Pérsico / Estrecho de Ormuz | Zona operativa crítica para señales marítimas y contexto regional. |

En la Fase 2 se validará si las fuentes permiten representar estas regiones con
calidad suficiente. Si una región no tiene datos consistentes, se podrá eliminar
o agrupar.

## Período inicial de estudio

El período preliminar será:

```text
2024-01-01 a 2026-05-08
```

Razones:

- cubre un período suficientemente amplio para observar variaciones temporales;
- permite construir ventanas diarias con más ejemplos para entrenamiento;
- incluye meses recientes respecto al enunciado del proyecto;
- mantiene un tamaño manejable para validación, EDA y modelado en el curso.

Este período podrá ajustarse si:

- una fuente no ofrece datos históricos suficientes;
- el volumen de datos es demasiado bajo o demasiado alto;
- la cobertura temporal entre fuentes queda muy desbalanceada.

## Tarea principal de machine learning

La tarea principal propuesta es:

```text
clasificación supervisada multiclase
```

La variable objetivo será `escalation_level`, con tres clases:

- `low`;
- `medium`;
- `high`.

Se usarán nombres de clase en inglés dentro del dataset para mantener consistencia
técnica, pero el dashboard podrá mostrarlos en español.

## Qué entendemos por nivel de escalada

En este proyecto, el nivel de escalada no se entiende como una verdad absoluta
sobre el conflicto ni como una predicción operacional. Será una categoría
analítica construida para resumir la intensidad relativa de señales observables
en una región durante un día.

La escalada se interpretará como un aumento en la tensión observable a partir de
fuentes abiertas. Ese aumento puede aparecer en varias dimensiones:

- **dimensión factual**: más eventos registrados, incidentes violentos, alertas o
  cambios en el tipo de evento reportado;
- **dimensión territorial**: concentración de eventos en una región, aparición de
  eventos en zonas sensibles o expansión hacia regiones vecinas;
- **dimensión actoral**: participación de más actores, actores estatales,
  milicias, fuerzas aliadas u organizaciones asociadas;
- **dimensión mediática**: aumento del volumen de noticias, menciones,
  intensidad narrativa o tono negativo en fuentes textuales;
- **dimensión contextual**: señales complementarias como actividad de movilidad,
  anomalías térmicas, incidentes marítimos o cambios en patrones externos;
- **dimensión temporal**: incremento respecto a días anteriores, persistencia de
  actividad o aparición repentina de picos.

El nivel de escalada será relativo al dataset y al período analizado. Un día
clasificado como `high` no significa necesariamente guerra abierta; significa que,
dentro de la evidencia disponible, esa ventana concentra señales más intensas o
anómalas que otras ventanas comparables.

### Interpretación de las clases

| Clase técnica | Etiqueta en dashboard | Interpretación esperada |
| --- | --- | --- |
| `low` | Bajo | Ventana con pocas señales relevantes, actividad cercana al comportamiento base, ausencia de incidentes fuertes o cobertura mediática limitada. |
| `medium` | Medio | Ventana con señales moderadas: aumento de eventos, mayor volumen noticioso, aparición de incidentes relevantes o cambios visibles frente al comportamiento reciente. |
| `high` | Alto | Ventana con concentración fuerte de señales: varios eventos, eventos de mayor severidad, alta cobertura mediática, participación de actores relevantes o anomalías contextuales coincidentes. |

### Lo que no significa el nivel de escalada

Para evitar una interpretación exagerada, el target no debe leerse como:

- probabilidad real de guerra;
- alerta militar;
- confirmación causal de un ataque;
- medición completa de violencia regional;
- evaluación geopolítica definitiva.

Es una variable de trabajo para comparar ventanas `region-dia` y evaluar si las
fuentes abiertas permiten construir una señal analítica razonable.

## Construcción preliminar del target

El target se construirá mediante reglas transparentes a partir de variables
observables en la ventana `region-dia`.

Variables candidatas para construir el score de escalada:

- número de eventos estructurados;
- presencia de eventos con mayor severidad aproximada;
- número de actores involucrados;
- volumen de noticias relacionadas;
- tono o intensidad mediática si la fuente lo permite;
- señales contextuales anómalas, si aportan información real.

Propuesta inicial:

1. Calcular un `escalation_score` por región y día.
2. Normalizar los componentes para que ninguna fuente domine por escala.
3. Asignar clases con umbrales documentados:
   - `low`: score bajo o ausencia de señales relevantes;
   - `medium`: actividad moderada en una o más fuentes;
   - `high`: concentración alta de eventos, noticias o señales contextuales.

Los umbrales exactos no se fijan todavía. Se decidirán después del EDA para evitar
clases vacías o extremadamente desbalanceadas.

## Features candidatas

### Eventos estructurados

- conteo de eventos por región-día;
- conteo por tipo de evento;
- número de actores distintos;
- coordenadas agregadas o presencia de eventos geolocalizados;
- proxy de severidad si la fuente lo permite.

Nota de implementación actual:

- la primera ruta pensada para esta capa era `ACLED`;
- por restricciones de acceso con credenciales, ACLED no se usará como ruta
  principal en esta fase;
- la alternativa prioritaria pasa a ser `UCDP GED + UCDP Candidate`, porque
  ofrece eventos georreferenciados descargables sin credenciales y permite
  cubrir `2024` con GED y `2025-2026` con Candidate.

### Texto y noticias

- volumen de artículos o menciones;
- términos frecuentes;
- variables TF-IDF agregadas;
- tono o sentimiento si está disponible;
- entidades o países mencionados, si se implementa extracción básica.

### Contexto o movilidad

- conteos agregados de actividad aérea, marítima o señales contextuales;
- presencia de anomalías;
- cambios respecto a promedios recientes.

## Modelos candidatos

La comparación debe incluir una línea base y al menos tres modelos.

Primera lista para clasificación:

- baseline por clase mayoritaria;
- Naive Bayes;
- KNN;
- Logistic Regression;
- Decision Tree o Random Forest como modelo adicional explicable.

Si el target queda como valor continuo, la tarea podrá cambiar a regresión y se
considerarán:

- baseline por media o mediana;
- Linear Regression;
- Ridge o Lasso;
- KNN Regressor;
- Random Forest Regressor como modelo adicional.

## Métricas candidatas

Para clasificación:

- accuracy como referencia general;
- macro F1 como métrica principal si las clases quedan desbalanceadas;
- precision y recall por clase;
- matriz de confusión;
- análisis de errores por región y fecha.

Para regresión, si se cambia la tarea:

- MAE;
- RMSE;
- R2;
- análisis de residuos por región y fecha.

## Supuestos iniciales

- Las fuentes abiertas tendrán cobertura desigual por país, idioma y tipo de
  evento.
- El volumen de noticias no equivale automáticamente a escalada real.
- El target será una construcción analítica, no una verdad absoluta.
- La unidad región-día puede ocultar eventos intra-día, pero mejora la integración
  multifuente.
- El modelo debe ser explicable para que el dashboard no sea una caja negra.

## Criterios para validar o ajustar esta definición

Durante la Fase 2 se revisará:

- si existen datos suficientes por región y fecha;
- si las fuentes pueden conectarse a la unidad `region-dia`;
- si el target produce clases con ejemplos suficientes;
- si los textos tienen volumen y calidad para usar NLP básico;
- si la fuente contextual aporta información o solo ruido;
- si el alcance sigue siendo manejable para el curso.

## Decisión de avance

La Fase 1 queda definida con una propuesta inicial de clasificación por nivel de
escalada en ventanas `region-dia`.

La siguiente fase debe validar esta propuesta con muestras reales de fuentes antes
de confirmar definitivamente:

- fuentes finales;
- período exacto;
- regiones finales;
- reglas del target;
- métrica principal.
