# ADR-0003: pasarela de IA autogestionada con modelos versionados

- **Estado:** Aceptado
- **Fecha:** 2026-10-29
- **Ámbito:** inferencia LLM, embeddings y búsqueda vectorial

## Contexto
Peitho necesita modelos de lenguaje y embeddings sin tarifas por token o suscripciones de inferencia externas, manteniendo privacidad, trazabilidad y actualizaciones reproducibles.

## Decisión
1. **vLLM** sirve modelos de lenguaje en infraestructura propia con API compatible con OpenAI; compatibilidad de protocolo no significa usar OpenAI API.
2. Un microservicio independiente en **Python con Sentence Transformers** genera embeddings localmente; PostgreSQL + pgvector almacena vectores y metadatos de versión.
3. **Ollama solo para desarrollo/pruebas locales.** vLLM es la referencia de producción cuando la infraestructura lo permita.
4. La configuración inicial no activa OpenAI API, Azure OpenAI, Anthropic, Google Vertex AI ni otros endpoints de inferencia de pago.
5. La selección de modelos concretos queda pendiente de benchmarks en español/vocabulario comercial ecuatoriano, licencias, latencia, memoria y throughput.

## Versionado y calidad
- Versionar independientemente modelo/revisión de LLM y encoder de embeddings; registrar idioma, licencia, benchmark y fecha de aprobación.
- No fijar arbitrariamente dimensiones antes de elegir el encoder. Guardar dimensiones reales en metadatos.
- Al cambiar encoder, crear versión de índice, recalcular embeddings de forma idempotente, comparar Recall@k/precisión, latencia y recursos y cambiar tráfico solo tras validación.
- Filtrar tenant, permisos, finalidad y vigencia antes de usar documentos recuperados. El LLM no tiene SQL libre ni es autoridad de stock/precio/coste.
- Restringir salida de red para prevenir exportación accidental de prompts/datos a proveedores públicos.

## Costes
Se evitan tarifas por token y suscripciones, pero quedan costes de hardware/GPU/CPU, energía, almacenamiento, backups, monitorización y mantenimiento. Si la infraestructura no soporta vLLM, dimensionar la solución y no cambiar silenciosamente a un proveedor de pago.

## Criterios de aceptación
- Health checks de vLLM y embeddings con revisiones fijadas.
- Perfil productivo sin credenciales de proveedores de inferencia externos; Ollama no es el runtime productivo.
- Benchmarks/licencia documentados y pruebas que impidan mezclar espacios vectoriales incompatibles.
