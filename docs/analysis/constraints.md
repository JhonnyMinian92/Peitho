# Constraints

Record mandatory technical, business, security, compatibility and operational constraints.

## Proposed changes (internal analysis)

- Estructuración del repositorio en dos carpetas raíz dedicadas: 'app-peitho-ecuapp' para el frontend Angular y 'microservices-peitho-ecuapp' para el backend FastAPI y workers analíticos.
- Formalización de la base de datos PostgreSQL ampliada con pgvector, definiendo 41 tablas relacionales y semánticas con Row-Level Security (RLS) multiempresa obligatorio.
- Blindaje de integración con Inventary mediante usuario de base de datos con SELECT exclusivo sobre vistas/tablas de réplica, prohibiendo cualquier capacidad de escritura desde Peitho.
- Exclusión estricta de cualquier dispatcher o credencial de envío/publicación hacia WhatsApp, SMS, llamadas o redes sociales, garantizando que Peitho únicamente genera propuestas auditables para revisión humana.
- Autenticación local segura en el lanzamiento: Argon2id, MFA mediante TOTP o passkeys/WebAuthn, TLS, sesiones protegidas y auditoría; sin federación OIDC/SAML inicial. La federación queda como extensión futura aceptada mediante ADR.



## Decisiones de arquitectura confirmadas (2026-10-29)

- **Inventary:** réplica lógica dedicada hacia PostgreSQL de Peitho como integración principal, más API interna autenticada de solo lectura para verificaciones puntuales. No se ejecutan cargas analíticas intensivas en la base transaccional ni se escribe en Inventary.
- **Frescura de datos:** registrar timestamps de origen/recepción/aplicación, watermark, lag, versión de esquema y conciliación por conjunto de datos. Coordinar DDL; vigilar LSN, slots, WAL, disco y recuperación. Datos stale, no conciliados o con esquema incompatible no habilitan decisiones confirmadas dependientes de stock/margen.
- **IA local:** vLLM para inferencia productiva cuando la infraestructura lo permita; microservicio Python/Sentence Transformers para embeddings locales en pgvector; Ollama solo desarrollo local. No configurar proveedores de inferencia de pago inicialmente. Seleccionar modelos después de benchmarks y revisión de licencias.
- **Mensajería comercial:** Peitho prepara análisis y borradores; no envía ni publica campañas ni inicia llamadas. Las acciones se ejecutan externamente y los resultados pueden importarse.
- **Organización del código:** frontend objetivo `app-peitho-ecuapp`, backend objetivo `microservices-peitho-ecuapp`, backend modular con contratos JSON request/response, auditoría, sondas de salud, controles de escalabilidad y tests; formularios largos por stepper y sin `alert` nativo.
- **SSOT:** [ADR-0001](../architecture/decisions/ADR-0001-inventary-logical-replication.md), [ADR-0002](../architecture/decisions/ADR-0002-autenticacion-local.md), [ADR-0003](../architecture/decisions/ADR-0003-pasarela-ia-autogestionada.md). En caso de divergencia, prevalece la decisión aceptada más reciente, no la formulación heredada del plan inicial.
