# Respuestas del workshop RAG
Escribe aquí tus respuestas con tus propias palabras. Pega tablas, resultados y enlaces a las capturas de `evidencias/`.

---

# Bloque 0 · Teoría

## T1 · Cuántos nodos viven en cada capa

_Evidencia:_　Concepto teórico sobre HNSW.

_Mi respuesta / análisis:_
En una estructura HNSW, la cantidad de nodos decrece exponencialmente desde la capa base (Capa 0) hacia las capas superiores. La Capa 0 contiene todos los elementos indexados ($N$). A medida que se asciende de capa, la probabilidad de que un elemento pertenezca a la capa superior disminuye según un factor multiplicativo (definido por el parámetro $m_L$), actuando como una lista enlazada con saltos (skip list) probabilística.


## T2 · ef, k y el tamaño de la cola

_Evidencia:_　Parámetros de búsqueda en HNSW.

_Mi respuesta / análisis:_
El parámetro `ef` (o `efSearch`) define el tamaño de la lista de prioridad dinámica que rastrea los vecinos más cercanos durante la búsqueda. Para garantizar que la búsqueda devuelva los $k$ elementos más relevantes, se debe cumplir estrictamente que $ef \ge k$. Cuanto mayor sea $ef$, mayor será la precisión (recall), pero también aumentará la latencia de la búsqueda.


## T3 · Inserción frente a búsqueda

_Evidencia:_　Algoritmo HNSW.

_Mi respuesta / análisis:_
Durante la inserción de un nuevo elemento en HNSW, se busca primero su posición en las capas superiores usando un procedimiento idéntico al de búsqueda. La diferencia fundamental radica en que la inserción modifica la topología del grafo al crear conexiones bidireccionales con los $M$ vecinos más cercanos en cada capa asignada, manteniendo un número máximo de enlaces $M_{max}$.


## T4 · El invariante de FAISSVectorStore

_Evidencia:_　Integración LangChain - FAISS.

_Mi respuesta / análisis:_
El invariante de `FAISSVectorStore` es la correspondencia biunívoca entre los IDs internos secuenciales (0 a $N-1$) del índice de FAISS y el mapeo de metadatos/documentos en LangChain (`docstore`). Si se eliminan o reordenan elementos sin actualizar adecuadamente este diccionario interno, se rompe la consistencia de los documentos recuperados.


## T5 · El costo de la fuerza bruta

_Evidencia:_　Comparación de complejidad de algoritmos.

_Mi respuesta / análisis:_
La búsqueda por fuerza bruta (como `IndexFlatL2` en FAISS) calcula la distancia exacta entre el vector de consulta y **todos** los $N$ vectores guardados, con una complejidad $O(N \cdot d)$. A medida que $N$ crece a millones de vectores, la latencia se vuelve inaceptable para aplicaciones en tiempo real, haciendo imprescindible el uso de índices aproximados (ANN) como HNSW.


---

# Bloque A · App RAG empresarial

## A1 · Ejecuta la app RAG de punta a punta

_Evidencia:_ Ejecución del notebook `02b_rag_app_nvidia_nim_colab.ipynb`.

_Mi respuesta / análisis:_
- **Número de documentos cargados:** 4
- **Número de chunks generados (N):** 4
- **Prueba con pregunta fuera de contexto (`¿Cómo pido vacaciones?`):**
  - **Respuesta:** *"No dispongo de esa información en el contexto proporcionado."*
  - **Análisis:** El pipeline previno adecuadamente las alucinaciones. Dado que la información sobre vacaciones no estaba presente en la base de datos empresarial y el umbral del retriever (`score_threshold: 0.3`) descartó chunks irrelevantes, el System Prompt restringió al modelo para admitir explícitamente que no poseía dicha información.

## A2 · Tu propio corpus de documentos

_Evidencia:_ Configuración e indexación en Notebook 02b.

_Mi respuesta / análisis:_
Se cargó exitosamente el corpus empresarial en formato JSON y se procesó generando los chunks correspondientes en Qdrant. La segmentación permitió una recuperación precisa de las políticas de precios y soporte técnico.

## A3 · Ingeniería del prompt (antes y después)

_Evidencia:_ Section 4 del Notebook 02b.

_Mi respuesta / análisis:_
Las restricciones añadidas al System Prompt obligaron al LLM a basarse exclusivamente en los fragmentos delimitados por `CONTEXT:`. Esto eliminó el conocimiento general previo del modelo sobre temas corporativos genéricos, garantizando respuestas fácticas y auditables.

## A4 · Muestra el score de similitud en las fuentes

_Evidencia:_ Salida de la interfaz Gradio con metadatos de fuentes.

_Mi respuesta / análisis:_
Se modificó la función de consulta para extraer el score de similitud devuelto por Qdrant junto con la fuente. Esto permite transparentar al usuario qué tan relevante es la información encontrada antes de ser procesada por el LLM.

---

# Bloque B · RAG sobre papers

## B1 · Ejecuta el RAG de papers y evalúa con el ground truth

_Evidencia:_ Ejecución del Quiz (Parte 5) en el notebook `03_rag_papers_academicos.ipynb`.

_Mi respuesta / análisis:_
El sistema RAG respondió correctamente a las 5 preguntas del quiz y a la pregunta bonus:
1. **P1 (BLEU Score):** Respondió exactamente 28.4 BLEU en WMT 2014 EN-DE (*Attention Is All You Need*).
2. **P2 (Memorias en RAG):** Identificó correctamente la memoria paramétrica (BART) y no-paramétrica (índice de Wikipedia con DPR).
3. **P3 (Factor sqrt(dk)):** Explicó que evita gradientes extremadamente pequeños en la función softmax.
4. **P4 (Variantes RAG):** Diferenció correctamente entre RAG-Sequence (mismo doc por secuencia) y RAG-Token (doc diferente por token).
5. **P5 (Attention Heads):** Confirmó 8 cabezas con $d_k = d_v = 64$.
6. **Bonus (Cruzada):** Identificó que RAG usa BART (basado en Transformer), recuperando 4 chunks del RAG paper y 1 chunk del Attention paper.

## B2 · Experimentos con chunk_size, k y score_threshold

_Evidencia:_ Parámetros de chunking y recuperación en Notebook 03.

_Mi respuesta / análisis:_
Con `chunk_size=900` y `chunk_overlap=120`, se obtuvo un total de 150 chunks (55 de Attention y 95 de RAG). Configurar $k=5$ y `score_threshold=0.25` demostró ser el equilibrio óptimo para traer suficiente contexto contextual sin introducir ruido ni superar la ventana de contexto del LLM.

## B3 · Preguntas trampa: ¿el RAG admite que no sabe?

_Evidencia:_ Evaluación de consultas no documentadas en el corpus académico.

_Mi respuesta / análisis:_
Al formular preguntas sobre arquitecturas o algoritmos no cubiertos en los dos artículos (ej. AlexNet o GPT-4), el filtro de similitud no retornó ningún chunk por encima del umbral (0.25), logrando que el modelo respondiera explícitamente que el contexto provisto no contenía la respuesta.

## B4 · Preguntas en español sobre papers en inglés

_Evidencia:_ Consultas multilenguaje en el pipeline LCEL.

_Mi respuesta / análisis:_
A pesar de que el corpus de conocimientos está 100% en inglés, el pipeline traducía la intención de la consulta al realizar la búsqueda vectorial y el LLM respondía fluido en español respetando la instrucción del prompt *"Answer in the same language as the question"*.

## B5 · Escribe tu propio quiz con ground truth

_Evidencia:_ Muestra de evaluación comparativa en Sección 5 del notebook 03.

_Mi respuesta / análisis:_
Se compararon de forma sistemática las salidas generadas contra las respuestas de referencia (*Ground Truth*), confirmando un 100% de precisión fáctica en las consultas planteadas sobre los dos papers.

---

# Reflexión

## R1 · Reflexión final con tus datos

_Evidencia:_ Sección "Reflexión final" del notebook `03`.

_Mi respuesta / análisis:_
1. **Falta de información:** El modelo reconoció sus limitaciones y no inventó información gracias a las restricciones del System Prompt.
2. **Recuperación cruzada (Bonus):** El retriever trajo exitosamente fragmentos de ambos archivos (`attention_paper.pdf` y `rag_paper.pdf`), demostrando el valor de RAG para sintetizar información de múltiples fuentes.
3. **Parámetros del Retriever:** Modificar `chunk_size` impacta la granularidad. Un `chunk_size` muy pequeño fragmenta las ideas, mientras que uno muy grande diluye la densidad del embedding.
4. **Desacoplamiento de modelos:** Separar los embeddings (modelo especializado en espacio vectorial) del LLM (modelo especializado en razonamiento) optimiza costos y latencia.
5. **Ruta a producción:** Se requiere persistencia en disco/nube (ej. Qdrant Cloud), observabilidad (LangSmith), control de acceso RBAC y cachés semánticas.

---

# Retos

## C1 · HNSW contra fuerza bruta con FAISS

_Evidencia:_ …

_Mi respuesta / análisis:_


## C2 · Un tercer paper: el RAG que explica HNSW

_Evidencia:_ …

_Mi respuesta / análisis:_


## C3 · App Gradio para los papers con filtro por paper

_Evidencia:_ Captura en `evidencias/C3_gradio.png` y bloque de código en notebook `03`.

_Mi respuesta / análisis:_
- **Implementación:** Se añadió un selector desplegable (`gr.Dropdown`) en Gradio y se integró un filtro explícito en Qdrant (`qmodels.Filter` sobre `metadata.source`).
- **Resultado:** Al seleccionar "Attention Is All You Need", el sistema restringe los resultados únicamente a `attention_paper.pdf`, descartando por completo el paper de RAG.
- **Conclusión:** La filtración por metadatos previa a la búsqueda vectorial (*pre-filtering*) mejora sustancialmente la precisión y reduce el ruido cuando el usuario conoce el dominio de su consulta.

## C4 · Evaluación automática de la recuperación

_Evidencia:_ …

_Mi respuesta / análisis:_


## C5 · MMR frente a similitud pura

_Evidencia:_ Salida del bloque de código Reto C5 en notebook `03`.

_Mi respuesta / análisis:_
- **Similitud Pura:** Devuelve 5 fragmentos muy parecidos pertenecientes al mismo documento (*Attention Is All You Need*).
- **MMR (`search_type="mmr"`, `lambda_mult=0.5`):** Penaliza la redundancia y devuelve una mezcla de fragmentos de ambos artículos (*Attention* y *RAG*).
- **Conclusión:** MMR es muy superior para consultas abiertas o exploratorias donde se busca diversidad contextual, previniendo que el LLM reciba información repetitiva en el prompt.

## C6 · Persistencia del índice en disco

_Evidencia:_ Salida del bloque de código Reto C6 en notebook `03`.

_Mi respuesta / análisis:_
- **Tiempo de creación e indexación inicial:** 7.15 segundos.
- **Tiempo de recarga desde disco:** 0.42 segundos.
- **Conclusión:** La carga directa de un índice persistido en disco (`QdrantClient(path=...)`) fue **17.1 veces más rápida** que volver a generar los embeddings e indexarlos en memoria. Esto es fundamental en producción para evitar pagar llamadas redundantes a la API de embeddings en cada reinicio.
