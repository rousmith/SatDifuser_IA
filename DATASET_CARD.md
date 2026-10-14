---
# Metadatos de la ficha de conjunto de datos (estilo Hugging Face).
# Especificación: https://github.com/huggingface/hub-docs/blob/main/datasetcard.md
language:
  - en
license: other
pretty_name: GEO-Bench (evaluación de SatDiFuser)
tags:
  - remote-sensing
  - geospatial
  - image-classification
  - semantic-segmentation
  - benchmark
---

# Ficha de Conjunto de Datos para GEO-Bench

<!-- Resumen rápido del conjunto de datos. -->

**GEO-Bench** es un *benchmark* de teledetección que reúne conjuntos de datos de clasificación y segmentación semántica en dominios y regiones geográficas diversos. Se utiliza en SatDiFuser como banco de evaluación de las representaciones aprendidas por el modelo de difusión.

## Detalles del conjunto de datos

### Descripción del conjunto de datos

<!-- Resumen más extenso. -->

GEO-Bench (*Geospatial Evaluation Benchmark*) es una colección estandarizada de tareas tipo *few-shot*/*fine-tuning* para evaluar modelos fundacionales geoespaciales (GFMs). Agrupa datasets con distintos sensores (p. ej. Sentinel-2, Landsat), resoluciones, números de bandas y tipos de etiqueta, cubriendo aplicaciones como agricultura, silvicultura, uso del suelo y detección de objetos.

En SatDiFuser se emplean dos familias de tareas:

- **Clasificación (con clasificador lineal):** `m-bigearthnet` (F1), `m-brick-kiln`, `m-eurosat`, `m-forestnet`, `m-pv4ger`, `m-so2sat`.
- **Segmentación semántica (con UPerNet):** `m-pv4ger-seg`, `m-nz-cattle`, `m-NeonTree`, `m-cashew-plantation`, `m-SA-crop-type`, `m-chesapeake-landcover`.

> Además de GEO-Bench, el *backbone* de SatDiFuser (DiffusionSat) fue preentrenado sobre **fMoW** (*Functional Map of the World*), un gran conjunto de imágenes satelitales; ver la sección [Datos de origen](#datos-de-origen).

- **Curado por:** Equipo de GEO-Bench (ServiceNow) — Lacoste et al. (2023).
- **Financiado por [opcional]:** [Información adicional necesaria]
- **Compartido por [opcional]:** Rousmith Ladino (Universidad Distrital Francisco José de Caldas).
- **Idioma(s):** N/A (datos de imagen; anotaciones y nombres de clase en inglés).
- **Licencia:** Varía por subconjunto (ver el repositorio de GEO-Bench). El conjunto como *benchmark* se distribuye según sus propias condiciones; consultar la fuente original.

### Fuentes del conjunto de datos [opcional]

- **Repositorio:** <https://github.com/ServiceNow/geo-bench>
- **Artículo [opcional]:** <https://arxiv.org/abs/2306.03831>
- **Demo [opcional]:** [Información adicional necesaria]

## Usos

### Uso directo

- Evaluación de modelos fundacionales geoespaciales en tareas *downstream* de clasificación y segmentación.
- Comparación *zero-shot*/*few-shot*/*fine-tuning* entre GFMs bajo un protocolo común.
- Estudio de escenarios de escasez de etiquetas (p. ej. 1 %, 10 % de los datos de entrenamiento).

### Usos fuera de alcance

- **Producción o toma de decisiones automáticas** sin validación local: los datos son un *benchmark* de investigación, no un sistema operacional.
- **Garantía de cobertura global o balance geográfico:** las regiones, sensores y distribuciones no están equilibrados entre subconjuntos.
- **Inferencias sobre atributos sensibles de personas:** no corresponde; los datos describen cobertura del suelo y objetos, no individuos.

## Estructura del conjunto de datos

<!-- Descripción de los campos y de cómo se construyen las particiones. -->

- **Modalidad:** imágenes satelitales/aéreas en formato ráster, con etiquetas de clasificación (etiqueta única o multietiqueta) o máscaras de segmentación semántica por píxel.
- **Particiones:** típicamente `train` / `val` / `test`, con una versión `*_mini` (versiones reducidas) para evaluaciones rápidas tipo *few-shot*.
- **Bandas:** número variable por subconjunto (desde RGB hasta multiespectral). SatDiFuser utiliza únicamente **bandas RGB** y rellena con ceros las faltantes.
- **Etiquetas:** varían por tarea; p. ej. clases de cobertura del suelo, presencia de objetos (paneles solares, hornos de ladrillo), delimitación de ganado o cultivos.

## Creación del conjunto de datos

### Justificación de la curación

GEO-Bench nace de la necesidad de un *benchmark* **uniforme y reproducible** para comparar GFMs, que hasta entonces se evaluaban con protocolos heterogéneos que dificultaban la comparación justa. Su diseño busca cubrir dominios, sensores y tipos de tarea suficientemente diversos como para medir la transferibilidad de las representaciones.

### Datos de origen

#### Recopilación y procesamiento de datos

Los subconjuntos provienen de fuentes satelitales/aéreas preexistentes y de programas de anotación; GEO-Bench los armoniza en un formato y protocolo comunes (resolución, normalización, particiones). Cada banda se normaliza individualmente con los valores mínimo y máximo del conjunto. Los detalles de descarga y exportación se describen en el repositorio (`GEO_BENCH_DIR`).

#### ¿Quiénes son los productores de los datos de origen?

Los organismos, agencias espaciales, proyectos académicos y equipos de anotación originales de cada subconjunto de datos del *benchmark*, así como el equipo curador de GEO-Bench (ServiceNow). El preentrenamiento del *backbone* DiffusionSat usa **fMoW** (*Functional Map of the World*), un conjunto de imágenes satelitales con metadatos.

### Anotaciones [opcional]

#### Proceso de anotación

Las anotaciones (etiquetas de clase y máscaras de segmentación) provienen de los subconjuntos originales y sus respectivos procesos de etiquetado; GEO-Bench no las re-anota, sino que las reempaqueta bajo un protocolo común. Consultar la documentación de cada subconjunto para el detalle.

#### ¿Quiénes son los anotadores?

Los anotadores y equipos originales de cada subconjunto de origen; [Información adicional necesaria].

#### Información personal y sensible

No se espera información personal identificable: los datos consisten en imágenes de la superficie terrestre y etiquetas de cobertura/objetos. Aun así, imágenes de alta resolución de zonas pobladas podrían, en principio, revelar detalles físicos de áreas privadas. Verificar las condiciones de las fuentes originales.

## Sesgos, riesgos y limitaciones

- **Cobertura geográfica desigual:** algunos subconjuntos están concentrados en regiones específicas (p. ej. Chesapeake en EE. UU., ganado en Nueva Zelanda), lo que limita la generalización global.
- **Desbalance de clases:** varias tareas tienen clases raras o muy desbalanceadas (de ahí el uso de F1 en `m-bigearthnet`).
- **Diversidad de sensores y resoluciones:** mezclar bandas, sensores y resoluciones puede introducir diferencias de dominio entre subconjuntos.
- **Ruido de etiquetado:** las máscaras y etiquetas heredan errores de los procesos de anotación originales.
- **Sesgo de tamaño:** los subconjuntos tienen tamaños muy distintos, lo que puede sesgar las comparaciones agregadas.

### Recomendaciones

- Reportar resultados **por subconjunto y por clase**, no solo promedios agregados.
- Tener en cuenta el origen geográfico y el sensor al interpretar el desempeño.
- Usar las particiones `mini`/completas de forma consistente para comparaciones reproducibles.
- Estar consciente de las limitaciones de representatividad antes de trasladar conclusiones a producción.

## Cita [opcional]

**BibTeX:**

```bibtex
@inproceedings{lacoste2023geo,
  title={GEO-Bench: Toward Foundation Models for Earth Monitoring},
  author={Lacoste, Alexandre and Lehmann, Nils and Rodriguez, Pau and Sherrick, Evan and Metref, Hamza and Grave, Edouard and others},
  booktitle={Advances in Neural Information Processing Systems (NeurIPS)},
  year={2023}
}
```

**APA:**

Lacoste, A., Lehmann, N., Rodriguez, P., Sherrick, E., Metref, H., Grave, E., et al. (2023). *GEO-Bench: Toward Foundation Models for Earth Monitoring*. En Advances in Neural Information Processing Systems (NeurIPS).

## Glosario [opcional]

- **GEO-Bench:** *benchmark* de teledetección para modelos fundacionales.
- **fMoW:** *Functional Map of the World*, gran conjunto de imágenes satelitales con metadatos.
- **Subconjunto `m-*`:** versiones estandarizadas de las tareas dentro de GEO-Bench (p. ej. `m-eurosat`).
- **Máscara de segmentación:** imagen por píxel con la clase asignada a cada posición.

## Más información [opcional]

- GEO-Bench: <https://github.com/ServiceNow/geo-bench> · <https://arxiv.org/abs/2306.03831>
- DiffusionSat: <https://arxiv.org/abs/2312.03606>
- SatDiFuser: <https://github.com/yurujaja/SatDiFuser> · <https://arxiv.org/abs/2503.07890>
- Guía de descarga de GEO-Bench: ver el README del repositorio de GEO-Bench.

## Autores de la ficha de conjunto de datos [opcional]

- **Rousmith Ladino** — Universidad Distrital Francisco José de Caldas — GitHub: [@rousmith](https://github.com/rousmith)

## Contacto de la ficha de conjunto de datos

- Rousmith Ladino — GitHub: [@rousmith](https://github.com/rousmith) — Correo: _(pendiente)_
