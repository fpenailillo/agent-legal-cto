# Decisiones de Diseño
## Agente de Inteligencia Contractual para CTOs

---

## 1. Fine-Tuning sobre RAG

**Decisión:** Utilizar fine-tuning supervisado (SFT) en lugar de Retrieval-Augmented Generation (RAG).

**Justificación:**
- **Privacidad:** El modelo fine-tuneado se ejecuta localmente en Google Colab. Los contratos analizados nunca salen del entorno controlado del usuario. Con RAG, los documentos deberían almacenarse en una base de datos vectorial externa, aumentando el riesgo de exposición.
- **Latencia:** No se requiere búsqueda vectorial en tiempo de inferencia; el modelo responde directamente.
- **Especialización profunda:** El modelo internaliza patrones de análisis contractual durante el entrenamiento, produciendo respuestas más estructuradas y consistentes.

**Trade-off aceptado:** El modelo no puede actualizar su conocimiento sin re-entrenamiento, a diferencia de RAG que puede incorporar nuevos documentos dinámicamente.

---

## 2. Qwen2.5-7B-Instruct sobre Llama 3.1 8B

**Decisión:** Usar Qwen2.5-7B-Instruct como modelo base para fine-tuning.

**Justificación:**
- Mejores resultados en benchmarks de razonamiento (MMLU, GSM8K, HumanEval)
- Modelo más reciente (septiembre 2024) con mejores capacidades de seguimiento de instrucciones
- Buen soporte para español en el pre-entrenamiento
- Compatible con cuantización 4-bit y QLoRA en GPU T4

**Alternativas descartadas:**
- Llama 3.1 8B: benchmarks inferiores en razonamiento
- Mistral 7B v0.3: menor soporte nativo para español
- Phi-3 Medium 14B: demasiado grande para GPU T4 con QLoRA

---

## 3. QLoRA sobre Full Fine-Tuning

**Decisión:** Utilizar QLoRA (cuantización 4-bit + LoRA) en lugar de full fine-tuning.

**Justificación:**
- **Restricción de hardware:** La GPU T4 tiene 12.7 GB de VRAM. Un modelo de 7B parámetros en FP16 requiere ~14 GB solo para los pesos, haciendo imposible el full fine-tuning.
- **Eficiencia:** QLoRA entrena solo ~0.5% de los parámetros (matrices LoRA en q/v/k/o_proj), reduciendo drásticamente el consumo de memoria y tiempo.
- **Calidad comparable:** La literatura demuestra que QLoRA logra resultados cercanos al full fine-tuning para tareas de instrucción.

**Hiperparámetros seleccionados:**
- r=16: rango suficiente para capturar la complejidad de análisis contractual
- alpha=32: ratio alpha/r de 2x, estándar en la literatura
- Módulos target: q_proj, v_proj, k_proj, o_proj (todas las proyecciones de atención)
- Dropout 0.05: regularización ligera para evitar overfitting en 1000 ejemplos

---

## 4. Datos Sintéticos sobre Datos Reales

**Decisión:** Generar datos de entrenamiento sintéticos usando un modelo teacher (Llama 3.3 70B).

**Justificación:**
- **Privacidad:** No se dispone de contratos reales etiquetados, y obtenerlos requeriría acuerdos de confidencialidad complejos.
- **Escalabilidad:** Se pueden generar 1000+ ejemplos diversos en ~30 minutos.
- **Control:** Se garantiza distribución balanceada entre tipos de tarea (250 por tipo).
- **Contexto regional:** Los prompts especifican contexto Chile/Latam con vendors reales.

**Trade-off aceptado:** Los datos sintéticos pueden no capturar la complejidad total de contratos reales. En producción se recomendaría complementar con datos reales anonimizados.

---

## 5. Google Drive para Almacenamiento del Adapter

**Decisión:** Almacenar el adapter LoRA, dataset y métricas en Google Drive.

**Justificación:**
- **Reproducibilidad:** El profesor puede ejecutar la demo completa compartiendo solo la carpeta de Drive y el notebook.
- **Costo cero:** Google Drive gratuito ofrece 15 GB, suficiente para el adapter (~100 MB), dataset (~50 MB) y métricas.
- **Resiliencia:** Si la sesión de Colab se desconecta, los checkpoints persisten en Drive.
- **Simplicidad:** No requiere infraestructura adicional (S3, GCS, etc.).

---

## 6. Databricks Free Edition para Generación de Datos

**Decisión:** Usar el AI Gateway de Databricks con Llama 3.3 70B para generar el dataset sintético.

**Justificación:**
- **Costo controlado:** Pay-per-token con costo estimado de $15-20 USD para 1000 ejemplos.
- **Calidad:** Llama 3.3 70B (dic 2024) produce respuestas de alta calidad como modelo teacher, con mejor seguimiento de instrucciones que la versión 3.1.
- **Compatibilidad:** API compatible con OpenAI SDK, facilitando la integración.

**Alternativas descartadas:**
- OpenAI GPT-4: costo significativamente mayor (~$100+ para 1000 ejemplos)
- Modelo local: Llama 70B no cabe en GPU T4
- APIs gratuitas: límites de rate y calidad inferior

---

## 7. LangChain ReAct para el Agente

**Decisión:** Implementar el agente usando LangChain con patrón ReAct.

**Justificación:**
- **Transparencia:** El patrón ReAct muestra el razonamiento del agente (Pensamiento → Acción → Observación), fundamental para un caso de estudio académico.
- **Modularidad:** Las 4 herramientas son independientes y fácilmente extensibles.
- **Ecosistema:** LangChain ofrece integración nativa con HuggingFace pipelines.

---

## 8. Estructura de 4 Notebooks Separados

**Decisión:** Dividir el pipeline en 4 notebooks independientes.

**Justificación:**
- **Ejecución independiente:** Cada notebook puede ejecutarse sin depender de los otros (usa Drive como intermediario).
- **Gestión de sesión:** Colab gratuito desconecta sesiones tras ~90 minutos de inactividad. Notebooks separados permiten retomar sin re-ejecutar todo.
- **Claridad pedagógica:** Cada notebook aborda una fase del pipeline, facilitando la comprensión del profesor.
- **Debugging:** Errores en una fase no afectan a las demás.
