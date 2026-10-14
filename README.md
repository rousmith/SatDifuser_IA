# SatDiFuser — Ficha de Modelo (Model Card)

**Modelo:** SatDiFuser
**Título del artículo:** *Can Generative Geospatial Diffusion Models Excel as Discriminative Geospatial Foundation Models?* (ICCV 2025)
**Repositorio original:** <https://github.com/yurujaja/SatDiFuser>

---

## Descripción general

**SatDiFuser** es un marco (*framework*) de aprendizaje autosupervisado (SSL, *self-supervised learning*) que convierte un modelo generativo de difusión para imágenes satelitales —**DiffusionSat**— en un potente extractor de representaciones **discriminativas** para teledetección (*remote sensing*, RS). A diferencia de los *geospatial foundation models* (GFMs) convencionales, que se entrenan con objetivos contrastivos o de *masked image modeling*, SatDiFuser demuestra que un modelo generativo puede también aprender representaciones útiles para tareas de clasificación y segmentación. Para ello analiza sistemáticamente las características de la difusión —que dependen del nivel de ruido y de la escala— a lo largo de múltiples *timesteps* y bloques de la red, y propone **tres estrategias de fusión** (*Global-Weighted*, *Localized-Weighted* y *Mixture-of-Experts*) para aprovecharlas. El *backbone* generativo se mantiene congelado (*training-free*) y solo se entrenan los componentes de fusión y un decodificador específico de la tarea. Evaluado sobre el *benchmark* **GEO-Bench**, SatDiFuser supera al estado del arte con mejoras de hasta **+5.7 % mIoU** en segmentación semántica y **+7.9 % F1-score** en clasificación.

---

## 📄 Fichas de documentación

- **[Ficha del modelo (Model Card) → `MODEL_CARD.md`](./MODEL_CARD.md)**
- **[Ficha del conjunto de datos (Dataset Card) → `DATASET_CARD.md`](./DATASET_CARD.md)**

---

## 👤 Autor de la ficha

- **Nombre completo:** Rousmith Ladino
- **Usuario de GitHub:** [@rousmith](https://github.com/rousmith)
- **Institución:** Universidad Distrital Francisco José de Caldas
- **Contacto:** _(pendiente)_

> Ficha elaborada como ejercicio académico de documentación de modelos y conjuntos de datos, siguiendo las directrices de **Mitchell et al. (2019), *Model Cards for Model Reporting*** y las plantillas de **Hugging Face** (`modelcard_template.md` y `datasetcard_template.md`).

---

## 📚 Referencias

- Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., Spitzer, E., Raji, I. D., & Gebru, T. (2019). *Model Cards for Model Reporting*. En Proceedings of the Conference on Fairness, Accountability, and Transparency (pp. 220–229). <https://arxiv.org/abs/1810.03993>
- Plantillas de Hugging Face: <https://github.com/huggingface/huggingface_hub/tree/main/src/huggingface_hub/templates>
- Jia, Y., Marsocci, V., Gong, Z., Yang, X., Vergauwen, M., & Nascetti, A. (2025). *Can Generative Geospatial Diffusion Models Excel as Discriminative Geospatial Foundation Models?*. ICCV 2025. <https://arxiv.org/abs/2503.07890>
