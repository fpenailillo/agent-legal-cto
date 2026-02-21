# Informe del Caso de Estudio
## Agente de Inteligencia Contractual para CTOs
### Magíster en Tecnologías de la Información — Universidad Técnica Federico Santa María

---

## 1. Introducción

### 1.1 Problema
Los CTOs de empresas tecnológicas en Chile y Latinoamérica enfrentan la gestión de múltiples contratos con proveedores de tecnología (AWS, Azure, SAP, Oracle, Salesforce, entre otros). La revisión manual de cláusulas, comparación de propuestas, seguimiento de renovaciones y negociación de condiciones consume tiempo valioso y es propensa a errores humanos.

### 1.2 Motivación
Se propone un agente de IA especializado que asista a CTOs en tareas contractuales, operando completamente en un entorno controlado (Google Colab + Google Drive) para garantizar la privacidad de los documentos sensibles. El modelo fine-tuneado se ejecuta localmente, sin enviar datos a APIs externas para inferencia.

### 1.3 Objetivos
- Diseñar e implementar un pipeline completo de fine-tuning con QLoRA sobre un LLM open-source
- Generar un dataset sintético de contratos tecnológicos usando un modelo teacher (Llama 3.1 70B)
- Evaluar cuantitativamente la mejora del fine-tuning frente al modelo base
- Construir un agente LangChain con 4 herramientas especializadas en tareas contractuales
- Lograr reproducibilidad total con un costo inferior a $20 USD

---

## 2. Marco Teórico

### 2.1 Modelos de Lenguaje de Gran Escala (LLMs)
Los LLMs son modelos de aprendizaje profundo basados en la arquitectura Transformer, entrenados sobre grandes corpus de texto para generar lenguaje natural coherente y contextualmente relevante.

### 2.2 Fine-Tuning Supervisado (SFT)
El fine-tuning supervisado adapta un modelo pre-entrenado a una tarea específica mediante pares de instrucción-respuesta. En este proyecto se utiliza SFT para especializar el modelo en tareas de análisis contractual.

### 2.3 QLoRA (Quantized Low-Rank Adaptation)
QLoRA combina cuantización a 4 bits con adaptadores de bajo rango (LoRA), permitiendo fine-tuning de modelos de 7B+ parámetros en GPUs con memoria limitada como la T4 (12.7 GB VRAM). Los adaptadores LoRA agregan matrices de bajo rango a las capas de atención, entrenando solo ~0.5% de los parámetros totales.

### 2.4 Agentes LangChain
LangChain permite construir agentes que combinan LLMs con herramientas externas usando el patrón ReAct (Reasoning + Acting). El agente razona sobre la tarea, selecciona la herramienta apropiada, ejecuta la acción y observa el resultado.

---

## 3. Diseño del Sistema

### 3.1 Arquitectura General

```
FASE 1: Generación de Datos → Llama 3.1 70B (Databricks) → JSONL
FASE 2: Fine-Tuning → Qwen2.5-7B-Instruct + QLoRA (Colab GPU T4) → Adapter LoRA
FASE 3: Evaluación → ROUGE-L + BERTScore → Métricas JSON
FASE 4: Demo → Agente LangChain con 4 herramientas → Análisis contractual
```

### 3.2 Justificación: Fine-Tuning sobre RAG
Se eligió fine-tuning sobre RAG (Retrieval-Augmented Generation) por las siguientes razones:
- **Privacidad:** El modelo fine-tuneado vive localmente; los contratos nunca salen del entorno
- **Latencia:** No requiere búsqueda en bases de datos vectoriales en tiempo de inferencia
- **Especialización:** El modelo internaliza patrones de análisis contractual durante el entrenamiento

---

## 4. Pipeline de Datos

### 4.1 Generación Sintética
Se utilizó Llama 3.1 70B como modelo teacher a través de la API de Databricks para generar 1000 ejemplos sintéticos distribuidos en 4 tipos de tarea:
- `analizar_clausula` (250 ejemplos)
- `comparar_propuestas` (250 ejemplos)
- `alerta_renovacion` (250 ejemplos)
- `estrategia_negociacion` (250 ejemplos)

### 4.2 Validación
Cada ejemplo generado se validó para asegurar la presencia de los campos requeridos (`instruccion`, `entrada`, `salida`, `tipo_tarea`) y la coherencia del contenido en español.

---

## 5. Fine-Tuning

### 5.1 Modelo Base
Qwen2.5-7B-Instruct fue seleccionado por:
- Mejores benchmarks de razonamiento frente a Llama 3.1 8B
- Modelo más reciente con mejor soporte de instrucciones en español
- Compatible con QLoRA en GPU T4

### 5.2 Hiperparámetros
- LoRA: r=16, alpha=32, dropout=0.05, módulos q/v/k/o_proj
- Entrenamiento: 3 epochs, batch=2, grad_accum=4, lr=2e-4, cosine scheduler
- Cuantización: 4-bit NF4 con double quantization

### 5.3 Curvas de Pérdida
Las curvas de pérdida de entrenamiento y evaluación se registran en `metricas/perdida_entrenamiento.json` y se visualizan en el notebook 02.

---

## 6. Evaluación

### 6.1 Métricas Cuantitativas
- **ROUGE-L:** Mide la superposición de subsecuencias más largas entre la respuesta generada y la referencia
- **BERTScore F1:** Utiliza embeddings del modelo `dccuchile/bert-base-spanish-wwm-cased` para medir similitud semántica en español

### 6.2 Análisis Cualitativo
Se presentan 20 ejemplos (5 por tipo de tarea) comparando las salidas del modelo base y el modelo fine-tuneado contra las respuestas de referencia.

---

## 7. Demo del Agente

### 7.1 Herramientas
| Herramienta | Función |
|-------------|---------|
| `analizar_clausula` | Identifica riesgos con severidad y recomendación ejecutiva |
| `comparar_propuestas` | Puntaje 1-10 por dimensión + decisión justificada |
| `alerta_renovacion` | Nivel de urgencia + fecha límite + checklist |
| `negociar` | BATNA + 3 tácticas concretas + líneas rojas |

### 7.2 Casos de Uso
1. Cláusula de renovación automática (12 meses, aviso 90 días)
2. Comparación AWS vs Azure para migración bancaria (~$500K/año)
3. Contrato SaaS CRM venciendo en 45 días (USD 120K/año)

---

## 8. Conclusiones

### 8.1 Resultados Obtenidos
- Pipeline completo de fine-tuning ejecutable en Google Colab gratuito
- Modelo especializado en contratos tecnológicos con mejora medible sobre el modelo base
- Agente funcional con 4 herramientas para asistencia contractual
- Costo total del proyecto inferior a $20 USD

### 8.2 Limitaciones
- Dataset sintético: las respuestas pueden no reflejar la complejidad de contratos reales
- GPU T4 limita el tamaño del modelo y el batch de entrenamiento
- El agente no tiene acceso a bases de datos legales o precedentes reales
- La evaluación automatizada (ROUGE-L, BERTScore) no captura completamente la calidad legal

### 8.3 Trabajo Futuro
- Incorporar contratos reales anonimizados para fine-tuning adicional
- Explorar modelos más grandes (14B) con GPUs A100
- Agregar herramientas de búsqueda en bases de datos de normativa legal chilena
- Implementar evaluación humana por expertos legales
- Considerar RAG complementario para actualización de normativas
