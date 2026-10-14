---
# Metadatos de la ficha de modelo (estilo Hugging Face).
# Especificación: https://github.com/huggingface/hub-docs/blob/main/modelcard.md
language:
  - en
license: apache-2.0
library_name: pytorch
pipeline_tag: image-segmentation
tags:
  - remote-sensing
  - geospatial
  - diffusion-models
  - self-supervised-learning
  - foundation-models
  - image-classification
  - semantic-segmentation
  - geo-bench
base_model: DiffusionSat
---

# Ficha de Modelo para SatDiFuser

<!-- Resumen rápido de qué es y qué hace el modelo. -->

**SatDiFuser** es un marco de aprendizaje autosupervisado (*self-supervised learning*, SSL) que reutiliza un modelo generativo de difusión para imágenes satelitales (**DiffusionSat**) como extractor de representaciones **discriminativas** para teledetección. Mediante la fusión de características de difusión en múltiples etapas y niveles de ruido, logra resultados estado del arte en clasificación y segmentación semántica sobre el *benchmark* **GEO-Bench**.

## Detalles del modelo

### Descripción del modelo

<!-- Resumen más extenso de qué es el modelo. -->

SatDiFuser parte de la hipótesis de que los modelos generativos de difusión, además de sintetizar imágenes, aprenden representaciones semánticas ricas durante su proceso de *denoising*. El modelo analiza las características del proceso de difusión —mapas de salida de bloques **ResNet**, **self-attention (SA)** y **cross-attention (CA)** del U-Net— que varían según la escala y el *timestep* de ruido, y propone tres estrategias de fusión para agregarlas de forma efectiva:

1. **Global Weighted Fusion (gw):** pesos escalares aprendibles por cada par bloque–timestep (agregación global amplia).
2. **Localized Weighted Fusion (lw):** pesos a nivel de píxel generados por una función de *gating*, que enfatizan selectivamente diferentes características en cada posición espacial.
3. **MoE Joint Fusion (moe):** una arquitectura *mixture-of-experts* que modela conjuntamente las interacciones entre múltiples *timesteps* y bloques, con enrutamiento *top-k*.

El *backbone* generativo (DiffusionSat) permanece **congelado**; solo se entrenan los módulos de fusión y un decodificador específico de la tarea (un clasificador lineal para clasificación y un decodificador **UPerNet** para segmentación). Esto convierte a SatDiFuser en un marco **training-free** respecto al *backbone* y reutilizable en distintas tareas de teledetección.

- **Desarrollado por:** Yuru Jia (KU Leuven / KTH), Valerio Marsocci (Agencia Espacial Europea), Ziyang Gong (Shanghai AI Lab), Xue Yang (SJTU), Maarten Vergauwen (KU Leuven) y Andrea Nascetti (KTH).
- **Financiado por:** Parcialmente apoyado por la *National Academic Infrastructure for Supercomputing in Sweden* (NAISS, Grant No. 2022-06725).
- **Compartido por [opcional]:** Rousmith Ladino (Universidad Distrital Francisco José de Caldas).
- **Tipo de modelo:** Modelo fundacional geoespacial basado en difusión (*diffusion-based geospatial foundation model*); extractor de representaciones con fusión de características multi-etapa.
- **Idioma(s):** El texto de condicionamiento (*prompts*) está en inglés; las tareas de visión no dependen del lenguaje natural.
- **Licencia:** Apache-2.0 (misma que el repositorio original).
- **Ajustado a partir del modelo [opcional]:** DiffusionSat (modelo de difusión latente adaptado a imágenes satelitales, basado en Stable Diffusion v2-1).

### Fuentes del modelo [opcional]

- **Repositorio:** <https://github.com/yurujaja/SatDiFuser>
- **Artículo [opcional]:** <https://arxiv.org/abs/2503.07890>
- **Demo [opcional]:** [Información adicional necesaria]

## Usos

<!-- Usuarios previstos y usos del modelo. -->

### Uso directo

<!-- Uso sin ajuste fino ni integración en aplicaciones mayores. -->

SatDiFuser se emplea como **extractor de representaciones** de imágenes satelitales (RGB) para tareas discriminativas. Su uso directo consiste en:

- Extraer características de un *backbone* de difusión preentrenado (DiffusionSat a 256 px o 512 px) y fusionarlas con una de las tres estrategias disponibles (`gw`, `lw`, `moe`).
- Entrenar un decodificador ligero (*head*) específico de la tarea sobre esas características congeladas:
  - **Clasificación:** clasificador lineal (mapa de clases de cobertura del suelo, uso del suelo, detección de objetos como paneles solares o hornos de ladrillo).
  - **Segmentación semántica:** decodificador UPerNet (delimitación de cultivos, ganado, cobertura terrestre, etc.).

### Uso posterior (downstream) [opcional]

<!-- Uso cuando el modelo se ajusta para una tarea o se integra en un ecosistema mayor. -->

- Adaptación a nuevos *datasets* de teledetección mediante el ajuste de los módulos de fusión y un *head* de tarea, sin reentrenar el *backbone* generativo.
- Integración dentro de *pipelines* de monitoreo ambiental, agrícola, forestal o urbano, siempre que se documenten los sesgos geográficos y de sensores (ver más abajo).
- Sustitución del *backbone* por otro modelo de difusión, ya que el marco es, en principio, extensible a otras arquitecturas de difusión.

### Usos fuera de alcance

<!-- Usos indebidos, maliciosos o para los que el modelo no funciona bien. -->

- **Síntesis/generación de imágenes:** aunque su *backbone* es generativo, SatDiFuser se documenta aquí para uso **discriminativo**. No se evaluó como generador de imágenes.
- **Tareas de seguridad crítica** (p. ej. respuesta a desastres, verificación de cumplimiento o decisiones legales) sin supervisión humana y validación local.
- **Entradas fuera de RGB:** los experimentos usan únicamente las bandas **RGB** del *backbone*, aunque los *datasets* tengan más bandas espectrales (las faltantes se rellenan con ceros). No debe asumirse buen desempeño con imágenes multiespectrales crudas sin adaptación.
- **Localizaciones no vistas:** no existe garantía de generalización a regiones geográficas, sensores o condiciones atmosféricas no representadas en los datos de preentrenamiento y evaluación.

## Sesgos, riesgos y limitaciones

<!-- Limitaciones técnicas y sociotécnicas. -->

- **Cobertura geográfica y de sensores limitada:** el preentrenamiento de DiffusionSat y la evaluación en GEO-Bench no cubren uniformemente todas las regiones del planeta; el desempeño puede degradarse en zonas subrepresentadas (p. ej. alta latitud, trópicos densos, zonas áridas).
- **Solo RGB:** al usar únicamente bandas RGB, se descarta información espectral (infrarrojo, etc.) relevante para cobertura vegetal y análisis de agua; esto puede introducir sesgos en la detección de ciertas clases.
- **Dependencia del *backbone*:** la calidad de las representaciones está acotada por la del modelo de difusión base (DiffusionSat / Stable Diffusion v2-1), incluidos sus sesgos de dominio.
- **Sensibilidad espacial de la fusión localizada:** la fusión *localized-weighted* puede responder a variaciones locales como sombras o diferencias de iluminación, generando artefactos en la segmentación.
- **No determinismo del muestreo difusivo:** los resultados dependen de los *timesteps* seleccionados; los autores recomiendan muestrear en el primer ~20 % de los pasos de difusión, ya que pasos tardíos pierden detalle fino.
- **Etiquetas de referencia:** los *datasets* de evaluación pueden contener errores de etiquetado propios de la anotación manual o de fuentes satelitales; esto afecta las métricas reportadas.

### Recomendaciones

- Documentar siempre el dominio de aplicación, la región geográfica y el sensor de origen de los datos de entrada.
- Validar con datos locales antes de desplegar en un dominio nuevo, especialmente si difiere del de GEO-Bench.
- Reportar las métricas desagregadas por clase y por región, no únicamente el promedio global.
- Reconocer que se trata de un modelo de investigación académica y que requiere supervisión humana en contextos sensibles.

## Cómo empezar a usar el modelo

El siguiente flujo reproduce el uso del repositorio original. Requiere **Python 3.11.5** y **CUDA 12.4.0**.

```bash
# 1. Clonar el repositorio
git clone https://github.com/yurujaja/SatDiFuser.git
cd SatDiFuser

# 2. Instalar dependencias
pip install -r requirements.txt
pip install --no-deps geobench

# 3. Descargar los checkpoints de DiffusionSat (512x512 y 256x256)
#    desde https://zenodo.org/records/13751498 y https://zenodo.org/records/13756199
#    y colocarlos en:
#      diffusionsat/pretrained/resolution512
#      diffusionsat/pretrained/resolution256
#    Luego apuntar 'pretrained_model_name_or_path' en
#    configs/_base_/diffusionsat.yaml a la carpeta raíz de checkpoints.

# 4. Definir la carpeta de GEO-Bench y ejecutar un experimento
export GEO_BENCH_DIR=/path/to/geobench
python run.py --config configs/eurosat_exp.yaml
```

**Selección de la variante de fusión** (en `configs/_base_/satdifuser.yaml`):

- `fuser: gw`  → Global-Weighted
- `fuser: lw`  → Localized-Weighted
- `fuser: moe` → Mixture-of-Experts

## Detalles de entrenamiento

### Datos de entrenamiento

<!-- Enlazar a la ficha del conjunto de datos si es posible. -->

SatDiFuser **no entrena el *backbone* de difusión**; hereda las representaciones preentrenadas de **DiffusionSat**, entrenado a gran escala sobre imágenes satelitales (dataset *fMoW*, Functional Map of the World, en resoluciones de 256 px y 512 px). Los componentes que sí se entrenan (módulos de fusión + decodificador de tarea) se ajustan sobre el *benchmark* **GEO-Bench**.

Ver la **[Ficha del conjunto de datos → `DATASET_CARD.md`](./DATASET_CARD.md)** para más detalle.

### Procedimiento de entrenamiento

- **Backbone:** congelado. Se extraen características de salidas **ResNet** y **self-attention** de bloques del decodificador del U-Net de DiffusionSat, en los *timesteps* `{1, 100, 200}`.
- **Optimizador:** AdamW con *learning rate* inicial de `0.01`, decaimiento coseno y *warm-up* de 5 épocas.
- **Batch size:** 32.
- **Decodificadores:** clasificador lineal (clasificación) y UPerNet (segmentación).
- **Condicionamiento:** se omite la metadata y el condicionamiento por clase para evitar filtración de información; se usa el *prompt* genérico *"A satellite image"*.

#### Preprocesamiento [opcional]

- Las imágenes se recortan o redimensionan (interpolación bilineal) a la resolución de preentrenamiento del *backbone* (512 px o 256 px).
- Cada banda espectral se normaliza individualmente con los valores mínimo y máximo calculados sobre todo el dataset.
- Se usan únicamente las bandas **RGB**; las bandas faltantes se rellenan con ceros.

#### Hiperparámetros de entrenamiento

- **Régimen de entrenamiento:** precisión mixta (según la implementación original de DiffusionSat/PANGAEA). Verificar el script de entrenamiento para el detalle exacto.

#### Velocidades, tamaños y tiempos [opcional]

[Información adicional necesaria] — no reportado en el artículo.

## Evaluación

### Datos, factores y métricas de prueba

#### Datos de prueba

Se evalúa sobre **GEO-Bench**, un *benchmark* con tareas de clasificación y segmentación semántica en dominios y regiones geográficas diversos. Ver la **[Ficha del conjunto de datos → `DATASET_CARD.md`](./DATASET_CARD.md)**.

#### Factores

- **Tipo de tarea:** clasificación vs. segmentación semántica.
- **Escenario de datos:** 100 % vs. 10 % de datos etiquetados (*data scarcity*).
- **Tipo de escenario de clase:** binario vs. multiclase.
- **Estrategia de fusión:** `gw`, `lw`, `moe`.

#### Métricas

- **Clasificación:** *top-1 accuracy* (y **F1-score** para `m-bigearthnet`, por su desbalance de clases).
- **Segmentación semántica:** **mIoU** (*mean Intersection over Union*).

Se usan estas métricas por ser las estándar del *benchmark* GEO-Bench, lo que permite comparar de forma justa con otros GFMs.

### Resultados

**Segmentación semántica (mIoU ↑, %)** — con decodificador UPerNet:

| Método | pv-s | nz-c | neon | cashew | sa-c | ches |
| --- | --- | --- | --- | --- | --- | --- |
| Mejor GFM previo | 94.8 | 84.1 | 58.1 | 62.2 | 34.4 | 65.7 |
| **SatDiFuser (MoE)** | **95.3** | **83.7** | **63.4** | **66.1** | 31.9 | **71.6** |
| **Δ vs. SOTA** | +0.5 | −0.4 | **+5.7** | **+4.3** | +0.3 | **+5.9** |

**Clasificación (top-1 acc / F1 ↑, %)** — con clasificador lineal:

| Método | ben (F1) | bk | es | fn | pv | s2s |
| --- | --- | --- | --- | --- | --- | --- |
| Mejor GFM previo | 58.3 | 95.8 | 93.8 | 52.5 | 97.3 | 55.4 |
| **SatDiFuser (MoE)** | 65.8 | 98.3 | 97.7 | **58.4** | **97.8** | 58.8 |
| **Δ vs. SOTA** | **+7.9** | +2.5 | +3.9 | +5.9 | +0.5 | +3.9 |

*Nota:* se reportan los mejores resultados de SatDiFuser por tarea; `pv-s`/`nz-c`/`neon`/`cashew`/`sa-c`/`ches` = `m-pv4ger-seg`, `m-nz-cattle`, `m-NeonTree`, `m-cashew-plantation`, `m-SA-crop-type`, `m-chesapeake-landcover`; `ben`/`bk`/`es`/`fn`/`pv`/`s2s` = `m-bigearthnet`, `m-brick-kiln`, `m-eurosat`, `m-forestnet`, `m-pv4ger`, `m-so2sat`.

#### Resumen

SatDiFuser obtiene el mejor desempeño en 5 de 6 tareas de segmentación y de forma consistente supera a los GFMs del estado del arte en clasificación, con ganancias de hasta **+5.7 % mIoU** y **+7.9 % F1**. Los mayores avances ocurren en *datasets* no saturados y bajo escasez de datos (10 % de etiquetas). El marco demuestra que los modelos generativos de difusión pueden rivalizar con los modelos fundacionales discriminativos en teledetección.

## Examen del modelo [opcional]

<!-- Trabajo de interpretabilidad. -->

Los autores analizan cualitativamente los mapas de características fusionadas y observan que cada experto del MoE se especializa en distintos patrones espaciales y de textura. También muestran que los mapas de *self-attention* del modelo de difusión atienden objetos semánticamente similares a diferentes escalas. Se trata, sin embargo, de un análisis cualitativo; no se incluye un estudio cuantitativo formal de interpretabilidad.

## Impacto ambiental

Las emisiones de carbono pueden estimarse con la [calculadora de impacto de ML](https://mlco2.github.io/impact#compute) de [Lacoste et al. (2019)](https://arxiv.org/abs/1910.09700).

- **Tipo de hardware:** [Información adicional necesaria] (el artículo no reporta el cómputo exacto del preentrenamiento de DiffusionSat).
- **Horas de uso:** [Información adicional necesaria]
- **Proveedor de nube:** [Información adicional necesaria]
- **Región de cómputo:** Se menciona apoyo de la infraestructura NAISS (Suecia): <https://www.naiss.se/>
- **Carbono emitido:** [Información adicional necesaria]

## Especificaciones técnicas [opcional]

### Arquitectura y objetivo del modelo

- **Backbone:** U-Net de un modelo de difusión latente (DiffusionSat, derivado de Stable Diffusion v2-1). Genera características en `S = 4` escalas, con bloques residuales (ResNet), *self-attention* (SA) y *cross-attention* (CA).
- **Extracción de características:** inversión con DDIM (*DDIM inversion*) para obtener latentes ruidosos y luego extracción multi-escala y multi-*timestep* de las salidas de bloques.
- **Fusión:** tres variantes (Global-Weighted, Localized-Weighted, Mixture-of-Experts con `E = 8` expertos y `top-k = 2`).
- **Decodificadores:** lineal (clasificación) y UPerNet (segmentación).
- **Objetivo:** aprendizaje autosupervisado de representaciones mediante el proceso generativo; el *backbone* no se ajusta.

### Infraestructura de cómputo

[Información adicional necesaria] — ver el repositorio original para el detalle de GPU utilizadas.

#### Hardware

Se ofrecen checkpoints entrenados a 256 px y 512 px; se recomienda GPU con soporte CUDA.

#### Software

- Python 3.11.5
- CUDA 12.4.0
- Dependencias en `requirements.txt` del repositorio (PyTorch, `diffusers`, PANGAEA, GEO-Bench, etc.)

## Cita [opcional]

**BibTeX:**

```bibtex
@inproceedings{jia2025satdifuser,
  title={Can Generative Geospatial Diffusion Models Excel as Discriminative Geospatial Foundation Models?},
  author={Jia, Yuru and Marsocci, Valerio and Gong, Ziyang and Yang, Xue and Vergauwen, Maarten and Nascetti, Andrea},
  booktitle={International Conference on Computer Vision (ICCV)},
  year={2025}
}
```

**APA:**

Jia, Y., Marsocci, V., Gong, Z., Yang, X., Vergauwen, M., & Nascetti, A. (2025). *Can Generative Geospatial Diffusion Models Excel as Discriminative Geospatial Foundation Models?*. En Proceedings of the International Conference on Computer Vision (ICCV).

## Glosario [opcional]

- **SSL (*Self-Supervised Learning*):** aprendizaje sin etiquetas, usando la propia estructura de los datos.
- **GFM (*Geospatial Foundation Model*):** modelo fundacional preentrenado sobre grandes volúmenes de datos geoespaciales y adaptado a múltiples tareas.
- **LDM (*Latent Diffusion Model*):** modelo de difusión que opera en un espacio latente comprimido.
- **mIoU:** *mean Intersection over Union*, métrica estándar de segmentación semántica.
- **F1-score:** media armónica de precisión y exhaustividad (*recall*).
- **MoE (*Mixture of Experts*):** arquitectura que reparte el cómputo entre subredes especializadas seleccionadas por un *router*.
- **DDIM inversion:** proceso que invierte el muestreo determinista de DDIM para obtener latentes ruidosos a partir de una imagen.
- **GEO-Bench:** *benchmark* de tareas de teledetección para evaluar modelos fundacionales.

## Más información [opcional]

- Repositorio original y código: <https://github.com/yurujaja/SatDiFuser>
- Artículo (arXiv): <https://arxiv.org/abs/2503.07890>
- DiffusionSat: <https://arxiv.org/abs/2312.03606>
- GEO-Bench: <https://arxiv.org/abs/2306.03831>

## Autores de la ficha de modelo [opcional]

- **Rousmith Ladino** — Universidad Distrital Francisco José de Caldas — GitHub: [@rousmith](https://github.com/rousmith)

## Contacto de la ficha de modelo

- Rousmith Ladino — GitHub: [@rousmith](https://github.com/rousmith) — Correo: rvladinog@udistrital.edu.co
