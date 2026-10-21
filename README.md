# Workshop RAG · SI3003 Inteligencia Artificial

- **Nombre:** …
- **Código estudiantil:** …
- **Usuario de GitHub:** …
- **Fecha:** 21 de octubre de 2026
- **Entrega a:** jipadillab@eafit.edu.co

## Progreso

Reemplaza esta tabla con la que genera el botón «Copiar mi progreso en Markdown» de la página de Tareas, o llénala a mano.

| ID | Tarea | Puntos | Hecha | Evidencia |
|---|---|---:|:---:|---|
| T1 | Cuántos nodos viven en cada capa | 2 | ⬜ | |
| T2 | ef, k y el tamaño de la cola | 2 | ⬜ | |
| T3 | Inserción frente a búsqueda | 2 | ⬜ | |
| T4 | El invariante de FAISSVectorStore | 2 | ⬜ | |
| T5 | El costo de la fuerza bruta | 2 | ⬜ | |
| A1 | Ejecuta la app RAG de punta a punta | 4 | ⬜ | |
| A2 | Tu propio corpus de documentos | 6 | ⬜ | |
| A3 | Ingeniería del prompt (antes y después) | 5 | ⬜ | |
| A4 | Muestra el score de similitud en las fuentes | 5 | ⬜ | |
| B1 | Ejecuta el RAG de papers y evalúa con el ground truth | 6 | ⬜ | |
| B2 | Experimentos con chunk_size, k y score_threshold | 8 | ⬜ | |
| B3 | Preguntas trampa: ¿el RAG admite que no sabe? | 6 | ⬜ | |
| B4 | Preguntas en español sobre papers en inglés | 5 | ⬜ | |
| B5 | Escribe tu propio quiz con ground truth | 5 | ⬜ | |
| R1 | Reflexión final con tus datos | 10 | ⬜ | |
| C1 | HNSW contra fuerza bruta con FAISS (reto) | 15 | ⬜ | |
| C2 | Un tercer paper: el RAG que explica HNSW (reto) | 12 | ⬜ | |
| C3 | App Gradio para los papers con filtro por paper (reto) | 15 | ⬜ | |
| C4 | Evaluación automática de la recuperación (reto) | 12 | ⬜ | |
| C5 | MMR frente a similitud pura (reto) | 8 | ⬜ | |
| C6 | Persistencia del índice en disco (reto) | 8 | ⬜ | |

**Autoevaluación:** … / 100 puntos.

## Cómo ejecutar mis notebooks

1. Abre cada notebook de `notebooks/` en Google Colab.
2. Crea el secreto `NVIDIA_API_KEY` en Colab (panel Secrets) con acceso para el notebook.
3. Ejecuta las celdas en orden.

> No incluyas tu API key en ningún archivo de este repositorio.

## Decisiones y aprendizajes

Escribe 5 a 8 líneas sobre lo más importante que descubriste hoy.

## Parte A: App RAG empresarial (Notebook 02b)

### Datos de ejecución:
- **Número de documentos cargados:** 4
- **Número de chunks generados (N):** 4

---

### Tarea A1: Prueba con preguntas fuera del contexto y observaciones

**Pregunta fuera de contexto realizada:** `¿Cómo pido vacaciones?`  
**Respuesta del modelo:** *"No dispongo de esa información en el contexto proporcionado."*

**Análisis y Conclusión:**
1. **Prevención de Alucinaciones:** El pipeline RAG funcionó de manera óptima y esperada. A pesar de que el modelo de lenguaje posee conocimientos previos generales sobre solicitudes de vacaciones, las instrucciones del *System Prompt* restringieron la generación únicamente al contexto provisto.
2. **Efectividad del Retriever y Filtros:** La búsqueda en Qdrant (con umbral de similitud `score_threshold: 0.3`) no devolvió ningún fragmento relevante, obligando al modelo a responder de forma concisa que no posee esa información.
3. **Confiabilidad Empresarial:** El sistema demuestra ser seguro para un entorno empresarial, ya que previene respuestas falsas o no verificadas (alucinaciones) ante consultas sobre temas no documentados.

## Parte B: RAG sobre papers académicos (Notebook 03)

### Evaluaciones del Quiz (Parte 5 y Bonus)

1. **Pregunta 1 (BLEU Score):**
   - **¿Es correcta?**: Sí. Respondió exactamente 28.4 BLEU.
   - **¿Viene del paper correcto?**: Sí, identificó *Attention Is All You Need*.
   - **¿Es completa?**: Sí.

2. **Pregunta 2 (Memoria Paramétrica y No-Paramétrica):**
   - **¿Es correcta?**: Sí. Explicó que la memoria paramétrica es BART y la no-paramétrica es el índice denso de Wikipedia.
   - **¿Viene del paper correcto?**: Sí, citó *Retrieval-Augmented Generation*.
   - **¿Es completa?**: Sí, detalló la aproximación top-K y el ajuste fino (joint fine-tuning).

3. **Pregunta 3 (Factor de escala sqrt(dk)):**
   - **¿Es correcta?**: Sí. Explicó que evita el crecimiento de las magnitudes del producto punto y previene gradientes extremadamente pequeños en la función softmax.
   - **¿Viene del paper correcto?**: Sí, de *Attention Is All You Need*.
   - **¿Es completa?**: Sí.

4. **Pregunta 4 (Variantes RAG-Sequence y RAG-Token):**
   - **¿Es correcta?**: Sí. Explicó la diferencia entre usar el mismo documento para toda la secuencia vs. diferentes documentos por token.
   - **¿Viene del paper correcto?**: Sí, del RAG paper.
   - **¿Es completa?**: Sí.

5. **Pregunta 5 (Attention Heads y dimensiones):**
   - **¿Es correcta?**: Sí. Indicó 8 cabezas con d_k = d_v = 64.
   - **¿Viene del paper correcto?**: Sí, de *Attention Is All You Need*.
   - **¿Es completa?**: Sí.

6. **Pregunta Bonus (Modelo Seq2Seq en RAG y arquitectura Transformer):**
   - **¿Es correcta?**: Sí. Confirmó que usa BART y que está basado en la arquitectura Transformer.
   - **¿Recuperó fragmentos de ambos papers?**: Sí, recuperó 4 fragmentos de `rag_paper.pdf` y 1 fragmento de `attention_paper.pdf`.

---

### Reflexión final (Tarea R1)

1. **¿Qué ocurrió cuando el contexto no contenía información suficiente?**
   El modelo reconoció la limitación y abstuvo de inventar información gracias a las restricciones del *System Prompt*, previniendo alucinaciones.

2. **¿Qué fragmentos recuperó para la pregunta bonus? ¿Aparecieron ambos papers?**
   Sí, recuperó fragmentos de ambos artículos (`rag_paper.pdf` y `attention_paper.pdf`), lo que permitió conectar información entre dos fuentes distintas para construir una respuesta completa.

3. **¿Qué efecto tendría modificar `chunk_size`, `chunk_overlap`, `k` o `score_threshold`?**
   - Reducir `chunk_size` o aumentar `k` otorga contextos más específicos pero puede fragmentar ideas complejas.
   - Aumentar `score_threshold` hace la búsqueda más estricta (evitando ruido), pero si es muy alto podría omitir información relevante.

4. **¿Por qué conviene separar el modelo de embeddings del modelo generativo?**
   Permite optimizar costos y rendimiento: los embeddings requieren modelos livianos y rápidos optimizados para comparación vectorial, mientras que la generación requiere un LLM con gran capacidad de razonamiento.

5. **¿Qué cambiarías para llevar este prototipo a producción?**
   - **Persistencia:** Reemplazar Qdrant en memoria por una base de datos vectorial persistente (Qdrant Cloud/Cluster).
   - **Observabilidad y Evaluación:** Integrar herramientas como LangSmith o Ragas para monitorear latencia y calidad de respuestas.
   - **Escalabilidad y Seguridad:** Implementar control de acceso por roles (RBAC) para los documentos y mecanismos de caché para consultas frecuentes.
