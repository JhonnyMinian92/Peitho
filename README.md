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

## Plan de implementación hasta el 100%

El proyecto se define como **producto empresarial completo; no MVP**. El alcance se implementa por tareas verificables, sin recortar capacidades funcionales para considerar una primera entrega terminada.

- [Roadmap integral e implementación PEI-001 a PEI-249](docs/roadmap/implementation-checklist.md)
- [Grafo de fases y dependencias para agentes](docs/roadmap/task-graph.json)
- [Contrato de ejecución para Claude Code, Codex, Gemini y otros agentes](docs/roadmap/agent-execution-contract.md)
- [Checklist de memoria y grafo de conocimiento](docs/roadmap/memory-capability-checklist.md)
- [Guía local Docker + PostgreSQL del host](docs/development/local-docker-postgres.md)
- [Catálogo de esquema de Inventary: tablas, columnas, relaciones y reglas de réplica](docs/integration/inventary-database-schema.md)
- [Catálogo JSON de esquema Inventary V1–V18](docs/contracts/schemas/inventary-source-schema-catalog-v18.json)

## Funcionalidad y estructura objetivo

Peitho analiza datos operativos y CRM a través de preventa, venta y postventa; desarrolla pronósticos, clasificación no supervisada, análisis de abandono y optimización de rutas cuando existan datos adecuados, y prepara propuestas y materiales comerciales omnicanal para revisión humana. Los resultados y supuestos deben ser trazables y separar datos observados de inferencias y pronósticos.

Estructura objetivo del repositorio:
- app-peitho-ecuapp/: frontend Angular + TypeScript.
- microservices-peitho-ecuapp/: backend Python/FastAPI modular, workers analíticos e integraciones.

Estos directorios y capacidades describen el destino de la arquitectura; no se afirma que todos estén implementados. Para formularios largos, la UX debe utilizar stepper; no usar alert nativo; incluir skeletons, validaciones, estados vacíos/error y componentes accesibles.

## Desarrollo local

La configuración obligatoria de desarrollo ejecuta frontend, API, workers y servicios de IA en Docker; **PostgreSQL + pgvector se ejecuta en el host y no en Compose**. La guía describe conexión por host.docker.internal, roles de privilegio mínimo, migraciones, variables de entorno y validación de salud. Los comandos Docker solo se consideran operativos cuando los Dockerfiles, compose.yaml y aplicaciones estén implementados y verificados.

## Estado y criterio de verdad

Estos documentos especifican arquitectura y plan objetivo; no son evidencia de que los componentes estén desplegados. Una tarea solo se marca completa cuando existe implementación real, pruebas automáticas aprobadas, seguridad/observabilidad/documentación aplicables y referencia al commit/CI. El porcentaje final se calcula por cobertura requisito → tarea → prueba → evidencia, no por número de archivos ni líneas de código.
