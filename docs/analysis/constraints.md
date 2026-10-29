# Constraints

Record mandatory technical, business, security, compatibility and operational constraints.

## Proposed changes (internal analysis)

- Estructuración del repositorio en dos carpetas raíz dedicadas: 'app-peitho-ecuapp' para el frontend Angular y 'microservices-peitho-ecuapp' para el backend FastAPI y workers analíticos.
- Formalización de la base de datos PostgreSQL ampliada con pgvector, definiendo 41 tablas relacionales y semánticas con Row-Level Security (RLS) multiempresa obligatorio.
- Blindaje de integración con Inventary mediante usuario de base de datos con SELECT exclusivo sobre vistas/tablas de réplica, prohibiendo cualquier capacidad de escritura desde Peitho.
- Exclusión estricta de cualquier dispatcher o credencial de envío/publicación hacia WhatsApp, SMS, llamadas o redes sociales, garantizando que Peitho únicamente genera propuestas auditables para revisión humana.
- Definición del esquema de autenticación enterprise con TLS obligatorio, reto de un solo uso en frontend, WebCrypto y hash Argon2id en backend, junto con soporte OIDC/WebAuthn.

