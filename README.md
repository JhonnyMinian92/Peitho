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


## Funcionalidad y estructura objetivo

Peitho analiza datos operativos y CRM a través de preventa, venta y postventa; desarrolla pronósticos, clasificación no supervisada, análisis de abandono y optimización de rutas cuando existan datos adecuados, y prepara propuestas y materiales comerciales omnicanal para revisión humana. Los resultados y supuestos deben ser trazables y separar datos observados de inferencias y pronósticos.

Estructura objetivo del repositorio:
- `app-peitho-ecuapp/`: frontend Angular + TypeScript.
- `microservices-peitho-ecuapp/`: backend Python/FastAPI modular, workers analíticos e integraciones.

Estos directorios y capacidades describen el destino de la arquitectura; no se afirma que todos estén implementados. Para formularios largos, la UX debe utilizar stepper; no usar `alert` nativo; incluir skeletons, validaciones, estados vacíos/error y componentes accesibles.

La automatización comercial de Peitho es analítica y de preparación de propuestas. De acuerdo con la decisión enterprise aceptada, no envía mensajes ni publica campañas desde la plataforma. La ejecución ocurre fuera de Peitho y sus resultados pueden registrarse/importarse para medir resultados reales.
