# API contracts

Los contratos de la API son la fuente de verdad de los modelos de petición/respuesta. Los endpoints concretos deberán publicarse como OpenAPI cuando el backend FastAPI se implemente.

## Contrato común

- [Petición JSON](../schemas/api-request-envelope.schema.json) — versión de esquema, `request_id`, datos específicos del endpoint y metadatos no sensibles.
- [Respuesta JSON](../schemas/api-response-envelope.schema.json) — `request_id` correlacionable, timestamp UTC, estado y payload de éxito o error.
- [Ejemplo de petición](../messages/api-request.example.json).
- [Ejemplo de éxito](../messages/api-response-success.example.json).
- [Ejemplo de error](../messages/api-response-error.example.json).

Cada endpoint necesita un esquema de `data` específico, HTTP status correcto, validación, paginación cuando aplique y pruebas de contrato. El `request_id` debe aparecer en logs/trazas de forma segura. El campo `tenant_id` enviado por cliente nunca es autoridad: el tenant se deriva de la sesión o credencial y del permiso vigente. B2B usa scopes, cuotas, autenticación de servicio e idempotencia; MCP expone herramientas tipadas, no SQL libre.

No se incluyen tokens, credenciales ni datos personales reales en ejemplos. Los contratos describen el objetivo documental, no una API ya desplegada.
