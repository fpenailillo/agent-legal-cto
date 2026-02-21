# Agente de Inteligencia Contractual para CTOs

**Magíster en Tecnologías de la Información — Universidad Técnica Federico Santa María**

Caso de estudio académico que implementa un agente de IA especializado en análisis de contratos tecnológicos, diseñado para asistir a CTOs en la gestión de proveedores cloud y SaaS en Chile y Latinoamérica.

---

## Problema que Resuelve

Los CTOs gestionan decenas de contratos con proveedores tecnológicos (AWS, Azure, SAP, Oracle, Salesforce). La revisión manual de cláusulas, comparación de propuestas, seguimiento de renovaciones y negociación de condiciones es un proceso lento y propenso a errores. Este proyecto construye un agente que automatiza estas tareas usando un modelo de lenguaje fine-tuneado que opera localmente, garantizando la privacidad de documentos sensibles.

---

## Arquitectura del Pipeline

```
┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐
│  01 — Generación    │     │  02 — Fine-Tuning   │     │  03 — Evaluación    │     │  04 — Demo Agente   │
│                     │     │                     │     │                     │     │                     │
│  Llama 3.3 70B      │────▶│  Qwen2.5-7B + QLoRA │────▶│  ROUGE-L            │────▶│  LangChain ReAct    │
│  (Databricks API)   │     │  (GPU T4, Colab)    │     │  BERTScore F1       │     │  4 herramientas     │
│                     │     │                     │     │                     │     │                     │
│  Salida: JSONL      │     │  Salida: Adapter    │     │  Salida: JSON       │     │  Salida: Análisis   │
│  1000 ejemplos      │     │  LoRA (~100 MB)     │     │  métricas           │     │  contractual        │
└─────────────────────┘     └─────────────────────┘     └─────────────────────┘     └─────────────────────┘
```

---

## Estructura del Repositorio

```
├── notebooks/
│   ├── 01_generacion_datos.ipynb      # Generación de 1000 ejemplos sintéticos
│   ├── 02_finetuning_qwen25.ipynb     # Fine-tuning QLoRA en GPU T4
│   ├── 03_evaluacion_modelo.ipynb     # Evaluación cuantitativa y cualitativa
│   └── 04_demo_agente.ipynb           # Demo del agente con 3 casos de uso
├── docs/
│   ├── decisiones_diseno.md           # Justificación de decisiones técnicas
│   ├── informe_caso_estudio.md        # Informe académico completo
│   └── resultados.md                  # Tabla de resultados y costos
├── requirements.txt                   # Dependencias con versiones fijas
└── README.md
```

---

## Paso a Paso de Implementación

### Requisitos Previos

- Cuenta de Google (para Colab y Drive)
- Cuenta en [Databricks Community Edition](https://community.cloud.databricks.com/) (costo ~$15-20 USD para generación de datos)
- No requiere hardware local — todo corre en Google Colab gratuito

### Paso 1 — Generación de Datos Sintéticos

**Notebook:** `notebooks/01_generacion_datos.ipynb`
**Tiempo estimado:** ~30 minutos
**Recurso:** API de Databricks (no requiere GPU)

1. Abrir el notebook en Google Colab
2. Configurar secret en Colab:
   - Panel izquierdo → icono de llave → Nuevo secreto
   - `DATABRICKS_TOKEN`: token de acceso personal de Databricks
3. Ejecutar todas las celdas (Ctrl+F9)
4. El notebook genera 1000 ejemplos de contratos usando Llama 3.3 70B como modelo teacher vía AI Gateway
5. Los ejemplos se distribuyen en 4 tipos de tarea (250 cada uno):
   - `analizar_clausula` — Análisis de riesgos en cláusulas contractuales
   - `comparar_propuestas` — Comparación de ofertas de proveedores
   - `alerta_renovacion` — Alertas de vencimiento con plan de acción
   - `estrategia_negociacion` — Estrategias de negociación con BATNA

**Salida:** `Drive/agente-contratos-cto/dataset/contratos_sft.jsonl`

### Paso 2 — Fine-Tuning con QLoRA

**Notebook:** `notebooks/02_finetuning_qwen25.ipynb`
**Tiempo estimado:** ~2-3 horas
**Recurso:** Google Colab con GPU T4 (gratuito)

1. Cambiar entorno de ejecución a GPU T4:
   - Entorno de ejecución → Cambiar tipo de entorno → GPU T4
2. Ejecutar todas las celdas
3. El notebook entrena Qwen2.5-7B-Instruct usando QLoRA:
   - Cuantización 4-bit NF4 para caber en 12.7 GB de VRAM
   - LoRA con r=16, alpha=32 en las 4 proyecciones de atención
   - 3 epochs, batch efectivo de 8, learning rate 2e-4 con cosine scheduler
4. Si la sesión de Colab se desconecta, re-ejecutar y el notebook reanuda desde el último checkpoint guardado en Drive

**Salida:** `Drive/agente-contratos-cto/adapter/` (adapter LoRA ~100 MB)

### Paso 3 — Evaluación del Modelo

**Notebook:** `notebooks/03_evaluacion_modelo.ipynb`
**Tiempo estimado:** ~20 minutos
**Recurso:** Google Colab con GPU T4

1. Ejecutar todas las celdas
2. El notebook evalúa 100 ejemplos del conjunto de prueba con dos métricas:
   - **ROUGE-L** — Superposición de subsecuencias entre respuesta generada y referencia
   - **BERTScore F1** — Similitud semántica usando `dccuchile/bert-base-spanish-wwm-cased`
3. Genera tabla comparativa: modelo base vs fine-tuned con porcentaje de mejora
4. Incluye evaluación cualitativa: 5 ejemplos por tipo de tarea (20 total)

**Salida:** `Drive/agente-contratos-cto/metricas/resultados_evaluacion.json`

### Paso 4 — Demo del Agente

**Notebook:** `notebooks/04_demo_agente.ipynb`
**Tiempo estimado:** ~10 minutos
**Recurso:** Google Colab con GPU T4

1. Ejecutar todas las celdas
2. El notebook carga el modelo fine-tuneado y construye un agente LangChain con patrón ReAct
3. Se demuestran 3 casos de uso con datos sintéticos:

| Caso | Escenario | Herramienta |
|------|-----------|-------------|
| 1 | Cláusula de renovación automática (12 meses, aviso 90 días, penalización 40%) | `analizar_clausula` |
| 2 | AWS ($480K) vs Azure ($520K) para migración bancaria | `comparar_propuestas` |
| 3 | CRM SaaS venciendo en 45 días, USD 120K/año, satisfacción 6.5/10 | `alerta_renovacion` |

4. Se ejecuta el agente completo con una consulta multiherramienta que encadena análisis, comparación y negociación
5. Se lanza una **interfaz interactiva con Gradio** que permite:
   - Subir un contrato en `.txt` o `.pdf` y analizarlo con cualquier herramienta
   - Comparar dos propuestas pegando el texto de cada una
   - Hacer consultas libres al agente ReAct que encadena herramientas automáticamente
   - Genera un link público temporal para compartir la demo

**Nota:** Todas las respuestas incluyen aviso legal indicando que el análisis es asistido por IA.

---

## Decisiones de Diseño Clave

| Decisión | Alternativa descartada | Razón |
|----------|----------------------|-------|
| Fine-tuning (SFT) | RAG | Privacidad: el modelo corre localmente, los contratos nunca salen del entorno |
| Qwen2.5-7B-Instruct | Llama 3.1 8B | Mejores benchmarks de razonamiento y soporte para español |
| QLoRA (4-bit) | Full fine-tuning | GPU T4 tiene solo 12.7 GB VRAM; full FT de 7B requiere ~14 GB |
| Datos sintéticos | Datos reales | No se dispone de contratos reales etiquetados; privacidad y escalabilidad |
| Google Drive | S3 / GCS | Costo cero, integración nativa con Colab, suficiente para ~150 MB |
| LangChain ReAct | Custom agent | Transparencia del razonamiento (Pensamiento → Acción → Observación) |

Documentación detallada en [`docs/decisiones_diseno.md`](docs/decisiones_diseno.md).

---

## Herramientas del Agente

| Herramienta | Entrada | Salida |
|-------------|---------|--------|
| `analizar_clausula` | Texto de cláusula contractual | Riesgos con severidad (alta/media/baja) + recomendación ejecutiva |
| `comparar_propuestas` | Propuesta A y B en texto libre | Puntaje 1-10 por dimensión + decisión final justificada |
| `alerta_renovacion` | Metadata del contrato (fechas, valor, proveedor) | Nivel de urgencia + fecha límite + checklist de acciones |
| `negociar` | Contexto del contrato + objetivos del CTO | BATNA + 3 tácticas concretas + líneas rojas |

---

## Costos

| Recurso | Plataforma | Costo |
|---------|------------|-------|
| Generación de datos (Llama 3.3 70B) | Databricks | ~$15-20 USD |
| Fine-tuning QLoRA (GPU T4) | Google Colab gratuito | $0 |
| Almacenamiento (adapter + dataset) | Google Drive (15 GB) | $0 |
| Evaluación y demo | Google Colab gratuito | $0 |
| **Total** | | **~$15-20 USD** |

---

## Stack Tecnológico

| Componente | Tecnología | Versión |
|------------|-----------|---------|
| Modelo base | Qwen/Qwen2.5-7B-Instruct | — |
| Modelo teacher | Llama 3.3 70B (Databricks) | — |
| Fine-tuning | QLoRA (peft + trl + bitsandbytes) | 0.12.0 / 0.10.1 / 0.43.3 |
| Framework LLM | transformers | 4.44.0 |
| Agente | LangChain + LangChain Community | 0.2.16 |
| Evaluación | evaluate + rouge-score + bert-score | 0.4.3 / 0.1.2 / 0.3.13 |
| Entorno | Google Colab (GPU T4) + Google Drive | — |

---

## Documentación

- [`docs/informe_caso_estudio.md`](docs/informe_caso_estudio.md) — Informe académico completo con marco teórico, diseño, evaluación y conclusiones
- [`docs/decisiones_diseno.md`](docs/decisiones_diseno.md) — Justificación detallada de cada decisión técnica con trade-offs
- [`docs/resultados.md`](docs/resultados.md) — Tabla de resultados cuantitativos, cualitativos y costos
