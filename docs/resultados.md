# Resultados
## Agente de Inteligencia Contractual para CTOs

---

## 1. Generación de Datos

| Métrica | Valor |
|---------|-------|
| Total de ejemplos generados | 1000 |
| Distribución por tipo de tarea | 250 por tipo (balanceado) |
| Tipos de tarea | analizar_clausula, comparar_propuestas, alerta_renovacion, estrategia_negociacion |
| Formato de salida | JSONL con campos: instruccion, entrada, salida, tipo_tarea |
| Modelo teacher | Llama 3.3 70B (Databricks AI Gateway) |
| Tiempo de generación | ~30 minutos |
| Costo estimado | ~$15-20 USD |

---

## 2. Fine-Tuning

| Parámetro | Valor |
|-----------|-------|
| Modelo base | Qwen/Qwen2.5-7B-Instruct |
| Método | QLoRA (4-bit NF4 + LoRA) |
| Rango LoRA (r) | 16 |
| Alpha LoRA | 32 |
| Módulos target | q_proj, v_proj, k_proj, o_proj |
| Epochs | 3 |
| Batch size efectivo | 8 (2 × 4 gradient accumulation) |
| Learning rate | 2e-4 (cosine scheduler) |
| GPU | NVIDIA T4 (12.7 GB VRAM) |
| Tiempo de entrenamiento | ~2-3 horas |
| Tamaño del adapter | ~100 MB |

### Curva de Pérdida
Los valores de pérdida de entrenamiento y evaluación por epoch se encuentran en:
`/content/drive/MyDrive/agente-contratos-cto/metricas/perdida_entrenamiento.json`

---

## 3. Evaluación Cuantitativa

> **Nota:** Los valores exactos se completan tras ejecutar el notebook 03.

| Métrica | Modelo Base | Fine-Tuned | Mejora (%) |
|---------|-------------|------------|------------|
| ROUGE-L | _pendiente_ | _pendiente_ | _pendiente_ |
| BERTScore F1 | _pendiente_ | _pendiente_ | _pendiente_ |

- Evaluación sobre 100 ejemplos del conjunto de prueba
- BERTScore calculado con `dccuchile/bert-base-spanish-wwm-cased`
- Resultados detallados en: `/content/drive/MyDrive/agente-contratos-cto/metricas/resultados_evaluacion.json`

---

## 4. Evaluación Cualitativa

Se analizan 20 ejemplos representativos (5 por tipo de tarea) comparando las respuestas del modelo base y el modelo fine-tuneado. Los ejemplos completos se generan en el notebook 03.

### Observaciones Esperadas
- **analizar_clausula:** El modelo fine-tuneado debería identificar riesgos específicos con niveles de severidad, mientras que el modelo base tiende a dar respuestas genéricas.
- **comparar_propuestas:** El modelo fine-tuneado debería generar tablas comparativas estructuradas con puntajes por dimensión.
- **alerta_renovacion:** El modelo fine-tuneado debería producir checklists accionables con fechas límite concretas.
- **estrategia_negociacion:** El modelo fine-tuneado debería generar BATNAs y tácticas específicas al contexto.

---

## 5. Demo del Agente

### Caso 1: Análisis de Cláusula de Renovación Automática
- **Entrada:** Cláusula de renovación automática de 12 meses con aviso previo de 90 días
- **Salida esperada:** Identificación de riesgos (lock-in, penalización por cancelación anticipada), severidad, y recomendación ejecutiva

### Caso 2: Comparación de Propuestas AWS vs Azure
- **Entrada:** Dos propuestas para migración de infraestructura bancaria (~$500K/año)
- **Salida esperada:** Puntaje 1-10 por dimensión (costo, seguridad, soporte, escalabilidad) + decisión justificada

### Caso 3: Alerta de Renovación de Contrato CRM
- **Entrada:** Contrato SaaS con proveedor CRM venciendo en 45 días, gasto anual USD 120K
- **Salida esperada:** Nivel de urgencia alto, fecha límite de decisión, checklist de acciones

---

## 6. Costos Finales

| Recurso | Plataforma | Costo |
|---------|------------|-------|
| Generación datos sintéticos (Llama 3.3 70B) | Databricks Free Edition | ~$15-20 USD |
| Fine-tuning QLoRA Qwen2.5-7B (GPU T4) | Google Colab gratuito | $0 |
| Almacenamiento adapter y dataset | Google Drive gratuito (15GB) | $0 |
| Demo del agente | Google Colab gratuito | $0 |
| **Total** | | **~$15-20 USD** |
