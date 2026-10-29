# Peitho

Peitho es una plataforma de inteligencia comercial y CRM para preventa, venta y postventa. Analiza datos operativos, produce pronósticos y recomendaciones explicables y prepara propuestas que una persona revisa y ejecuta fuera de Peitho.

## Límites de arquitectura

- **Inventary sigue siendo la autoridad transaccional** de productos, compras, ventas y existencias. Peitho solo lee y nunca contabiliza ventas ni modifica stock en Inventary.
- La integración prevista es una **réplica lógica dedicada hacia PostgreSQL de Peitho**, más una API interna de solo lectura para validaciones puntuales.
- La autenticación inicial es **local y segura** (Argon2id, MFA TOTP o passkeys, TLS, sesiones y auditoría), sin federación externa inicial.
- La IA se autogestiona: **vLLM** sirve el LLM y un microservicio Python con **Sentence Transformers** genera embeddings para **PostgreSQL + pgvector**. Ollama es para desarrollo local.
- Peitho **no envía ni publica campañas, correos, mensajes, llamadas ni contenido comercial**. Solo prepara borradores y propuestas revisables.

## Documentación principal

- [Especificación enterprise](docs/context.md)
- [Diagramas de flujos completos](docs/flow.md)
- [Decisiones técnicas cerradas](docs/question.md)
- [Arquitectura](docs/architecture.md)
- [Seguridad](docs/security.md)
- [Requisitos técnicos](docs/requirements.md)
- [Perfil y condiciones de producción](docs/analysis/production_profile.md)
- [Registros ADR](docs/architecture/decisions/)
- [Decisiones legibles por máquina](docs/architecture/decisions/decisions.json)
- [Configuraciones de ejemplo y esquemas JSON](docs/contracts/)

> Estos documentos especifican la arquitectura objetivo. No constituyen evidencia de que la réplica, la autenticación o los servicios de IA estén desplegados en producción.
