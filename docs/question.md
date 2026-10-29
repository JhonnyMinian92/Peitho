# Decisiones técnicas cerradas

**Estado al 29 de octubre de 2026:** las tres preguntas iniciales están respondidas. Las decisiones solo podrán cambiar mediante una revisión documentada (ADR); no se afirma que los componentes ya estén desplegados.

## Q-INV-01 — Integración de lectura con Inventary

**Respuesta aceptada:** utilizar una **réplica lógica dedicada de Inventary hacia PostgreSQL de Peitho como mecanismo principal**. Complementarla con una **API interna de solo lectura** para verificaciones puntuales de disponibilidad/precio antes de confirmar propuestas sensibles al estado actual. No ejecutar consultas analíticas intensivas sobre la base transaccional ni utilizar una read-replica transaccional como almacén analítico principal.

La replicación lógica no replica automáticamente DDL; los cambios de esquema deben coordinarse entre publicador y suscriptor. Por conjunto de datos se registran horas de origen, recepción/aplicación, watermark, retraso y conciliación. Se monitorizan posiciones de replicación, slots, retención/crecimiento de WAL, disco, fallos y recuperación. Si la frescura o conciliación no es suficiente, Peitho bloquea decisiones dependientes del stock o las deja explícitamente no confirmadas. Peitho nunca contabiliza ventas ni modifica existencias en Inventary.

Referencia: [ADR-0001](architecture/decisions/ADR-0001-inventary-logical-replication.md) y [configuración de ejemplo](contracts/config/inventary-readonly.example.json).

## Q-AUTH-01 — Autenticación inicial

**Respuesta aceptada:** comenzar con **autenticación local segura**, sin Microsoft Entra, Google Workspace, Keycloak ni servicios externos de identidad en el lanzamiento inicial. Implementar gestión local de usuarios, roles/permisos por empresa, contraseñas con Argon2id, MFA mediante TOTP o passkeys/WebAuthn, TLS, sesiones seguras, recuperación controlada, revocación y auditoría. El cifrado de aplicación para intercambio de credenciales se usa cuando corresponda y no reemplaza TLS ni hash de contraseñas. OIDC/SAML quedan como extensiones futuras si una necesidad empresarial real lo justifica.

Referencia: [ADR-0002](architecture/decisions/ADR-0002-autenticacion-local.md) y [política de ejemplo](contracts/config/authentication.example.json).

## Q-LLM-01 — Modelos de lenguaje y embeddings

**Respuesta aceptada:** configurar una **pasarela de IA propia y autogestionada**, sin OpenAI API, Azure OpenAI, Anthropic, Google Vertex AI ni servicios de inferencia de pago. Utilizar **vLLM** para servir modelos de lenguaje con API compatible con OpenAI y un **microservicio independiente en Python con Sentence Transformers** para embeddings locales almacenados en PostgreSQL + pgvector. **Ollama** queda permitido para desarrollo/pruebas locales; **vLLM** es la referencia de producción cuando la infraestructura lo permita.

La elección de modelos se hará mediante benchmarks en español y vocabulario comercial ecuatoriano, calidad de recuperación, precisión, memoria, latencia, rendimiento y licencia. LLM y embeddings tendrán versiones independientes; los cambios del encoder requieren reindexación versionada y verificación de dimensiones/calidad. Se eliminan tarifas por token y suscripciones de inferencia, no los costes de hardware, energía, almacenamiento y mantenimiento.

Referencia: [ADR-0003](architecture/decisions/ADR-0003-pasarela-ia-autogestionada.md) y [configuración de ejemplo](contracts/config/ai-gateway.example.json).

## Preguntas pendientes

No quedan preguntas pendientes entre Q-INV-01, Q-AUTH-01 y Q-LLM-01. Los umbrales operativos de lag/conciliación y la capacidad se deben definir durante dimensionamiento y pruebas de carga; los modelos concretos se seleccionan con benchmark, no por suposición.
