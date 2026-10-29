# JSON schemas for Peitho

Los esquemas de esta carpeta se versionan junto con los contratos que validan.

## API común

- [api-request-envelope.schema.json](api-request-envelope.schema.json): envoltorio común de peticiones; el payload requiere además un esquema específico por endpoint.
- [api-response-envelope.schema.json](api-response-envelope.schema.json): respuesta uniforme de éxito/error con correlación, timestamp y detalles de error saneados.

## Integración, autenticación e IA

- [source-freshness.schema.json](source-freshness.schema.json): estado de sincronización por conjunto de datos de Inventary, timestamps, watermark, lag y conciliación.
- [ai-gateway-config.schema.json](ai-gateway-config.schema.json): configuración permitida para vLLM y embeddings locales con Sentence Transformers.
- [authentication-policy.schema.json](authentication-policy.schema.json): política local de identidad, contraseñas, MFA, sesión y ausencia de federación externa inicial.

Los archivos *.example.json de ../config/ son plantillas sin secretos, no una configuración ya desplegada. Los límites operativos deben completarse por ambiente antes de producción. Validar JSON y contratos con estos esquemas en CI.
