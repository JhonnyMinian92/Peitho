<!-- Historical snapshot: this is the initial internal plan, not the current task queue or activation status. The 14-module breakdown and preliminary 41-table estimate are superseded by docs/roadmap/implementation-checklist.md and docs/roadmap/task-graph.json. The 41-table figure refers to an early proposed Peitho-owned schema, not Inventary. Current readiness is determined by AGENTS.md and the Mnemosine operational state; see docs/roadmap/agent-readiness-audit.md. Do not infer that implementation may start from the phrase below saying preparation was completed. -->

# Initial Plan (internal analysis)

Preparación y gobierno inicial completados para Peitho 3.0 Enterprise. Se consolida la arquitectura monorepo modular con frontend Angular (app-peitho-ecuapp) y backend Python/FastAPI (microservices-peitho-ecuapp), PostgreSQL con pgvector para búsqueda híbrida, y conexión estrictamente de solo lectura con Inventary. Se establece el mapa completo del producto final en 14 módulos (cimientos, áreas funcionales de inteligencia/CRM/propuestas, y endurecimiento para producción), sin tareas en esta fase, cubriendo el 100% de los requisitos del perfil de producción y respetando la regla fundamental de que Peitho jamás envía ni publica campañas comerciales.

## Changes

- Estructuración del repositorio en dos carpetas raíz dedicadas: 'app-peitho-ecuapp' para el frontend Angular y 'microservices-peitho-ecuapp' para el backend FastAPI y workers analíticos.
- Formalización de la base de datos PostgreSQL ampliada con pgvector, definiendo 41 tablas relacionales y semánticas con Row-Level Security (RLS) multiempresa obligatorio.
- Blindaje de integración con Inventary mediante usuario de base de datos con SELECT exclusivo sobre vistas/tablas de réplica, prohibiendo cualquier capacidad de escritura desde Peitho.
- Exclusión estricta de cualquier dispatcher o credencial de envío/publicación hacia WhatsApp, SMS, llamadas o redes sociales, garantizando que Peitho únicamente genera propuestas auditables para revisión humana.
- Definición del esquema de autenticación enterprise con TLS obligatorio, reto de un solo uso en frontend, WebCrypto y hash Argon2id en backend, junto con soporte OIDC/WebAuthn.

## Mapa del producto

Mnemosine planifica todas las tareas de cada módulo después de este análisis (analysis/product_plan.md).

- **M1** Cimientos del Sistema y Esqueleto Operacional de Punta a Punta (foundation)
  - Establecer la estructura del repositorio monorepo (app-peitho-ecuapp y microservices-peitho-ecuapp), framework de tests con ejecución en CI en cada pull request, arranque de frontend y backend en local y desarrollo, configuración desacoplada por variables de entorno (.env.example), y endpoints de salud operacional (/health).
  - Requisitos de producción: PRD-QA-01, PRD-QA-02, PRD-QA-03, PRD-SEC-01, PRD-SEC-02, PRD-SEC-03, PRD-OPS-01, PRD-DOC-01, PRD-DATA-04, PRD-DEP-01, PRD-DEP-02, PRD-DEP-04, WEB-SEC-01, WEB-UX-01, WEB-UX-03, API-01, API-03, API-04
- **M2** Capa Transversal de Persistencia, Multiempresa (RLS) y Seguridad de Acceso (foundation)
  - Construir la infraestructura de PostgreSQL con pgvector, migraciones versionadas y reversibles con Alembic, aislamiento estricto multiempresa (RLS por tenant_id), autenticación empresarial con reto de aplicación de un solo uso, hash Argon2id, gestión de sesiones y control de acceso RBAC/ABAC.
  - Requisitos de producción: PRD-DATA-01, PRD-DATA-03, PRD-DATA-04, PRD-DATA-05, PRD-SEC-03, PRD-OPS-01, WEB-SEC-02, API-01, API-02, API-04
- **M3** Ingesta Confiable, Conector de Lectura de Inventary y Reconciliación (feature)
  - Implementar el adaptador de conexión de solo lectura hacia Inventary, ingesta de eventos con verificación de idempotencia, deduplicación, registro canónico de eventos, cola duradera outbox/inbox, cuarentena de datos anómalos y jobs de reconciliación periódica con marcas de frescura (watermark).
  - Requisitos de producción: PRD-DATA-04, PRD-DATA-05, PRD-OPS-01, API-01, API-02, API-04
- **M4** Modelo Operativo de Inventario, Demanda Censurada y Pronósticos Base (feature)
  - Construir el modelo derivado de inventario distinguiendo existencias físicas, reservadas, no vendibles, en tránsito y disponibles oficiales (ATP); detección de periodos de rotura de stock para estimar demanda censurada; y cálculo de pronósticos de demanda por SKU y almacén mediante baselines estadísticos (Croston, TSB, medias móviles).
  - Requisitos de producción: PRD-OPS-01, API-01, API-02, API-04
- **M5** CRM Comercial 360, Resolución de Identidad y Consentimiento LOPDP (feature)
  - Desarrollar el CRM unificado de Peitho para gestión del ciclo preventa-venta-postventa: captura de leads, resolución determinista de identidad sin fusiones ambiguas, gestión de oportunidades por etapas, registro de actividades y objeciones, seguimiento de SLA comercial y registro auditable de consentimiento/oposición conforme a la LOPDP de Ecuador.
  - Requisitos de producción: WEB-LEG-01, PRD-SEC-03, API-01, API-02, API-04
- **M6** Analítica Comercial Avanzada, Embudo, RFM y Medición de Atribución (feature)
  - Construir el motor analítico de ventas netas, análisis de cohortes, probabilidad de recompra/abandono, segmentación RFM descriptiva, Customer Lifetime Value (CLV) con guardas de suficiencia de datos, y motor de marketing para comparar ventas reales con gasto publicitario separando correlación de causalidad.
  - Requisitos de producción: PRD-OPS-01, API-01, API-02, API-04
- **M7** Motor Determinista de Decisiones y Guardas Comerciales (feature)
  - Implementar el núcleo prescriptivo de Peitho: evaluación continua de señales (demanda, leads, márgenes), aplicación de restricciones no negociables (stock disponible, margen mínimo, consentimiento verificado, frescura de datos) y síntesis de propuestas de acción comercial justificadas económicamente con escenarios y contrafactuales.
  - Requisitos de producción: PRD-SEC-03, PRD-OPS-01, API-01, API-02, API-04
- **M8** Frontend Corporativo Angular: Centro de Decisiones y Studio de Propuestas (feature)
  - Construir la SPA corporativa en Angular (app-peitho-ecuapp) con diseño enterprise exclusivo, shell global con selector de tenant, Centro de Decisiones accionable, CRM 360, explorador de inventario/demanda, Studio de propuestas con previsualización creativa de borradores, flujo de aprobación humana y exportación manual, sin botones ni acciones de envío saliente.
  - Requisitos de producción: PRD-QA-02, WEB-UX-01, WEB-UX-02, WEB-UX-03, WEB-PERF-01, WEB-SEO-01, WEB-SEC-01, API-04
- **M9** Servicios Semánticos con pgvector y Pasarela de Políticas de IA (feature)
  - Integrar la capacidad de búsqueda híbrida (vectorial con pgvector + texto completo + filtros SQL por tenant) para recuperar conocimiento aprobado, casos históricos y objeciones; e implementar el gateway de IA con control de tokens, redacción de PII, prevención de inyecciones y validación estricta de salida para generar borradores de texto y briefs sin alucinaciones.
  - Requisitos de producción: PRD-SEC-03, PRD-OPS-01, API-01, API-02
- **M10** Servidor MCP Protegido y API para Agentes Externos de IA (feature)
  - Exponer un servidor MCP bajo transporte HTTP seguro conforme a la especificación, con catálogo cerrado de herramientas de solo lectura y creación de borradores (get_business_snapshot, get_product_state, draft_proposal), autenticación OAuth 2.1, scopes por tenant, sin acceso a SQL directo ni capacidades de envío externo.
  - Requisitos de producción: PRD-SEC-03, API-01, API-02, API-03, API-04
- **M11** Plataforma de APIs B2B, Cuotas y Portal de Integración (feature)
  - Construir las APIs B2B-first para clientes empresariales que consumen el backend de Peitho de forma desacoplada: autenticación OAuth 2.1 client credentials, rotación de llaves, cuotas y rate limiting por tenant, idempotencia obligatoria en escrituras (Idempotency-Key), especificación OpenAPI completa y entorno sandbox.
  - Requisitos de producción: PRD-SEC-03, API-01, API-02, API-03, API-04
- **M12** Endurecimiento de Seguridad, Auditoría Append-Only y Gobernanza LOPDP (production)
  - Implementar el registro inmutable de auditoría (linaje completo desde dato fuente hasta propuesta y decisión humana), suite de pruebas de aislamiento cross-tenant (RLS, cache, storage), escaneo de vulnerabilidades de dependencias y contenedores, cabeceras web seguras (CSP, HSTS) y flujos de derechos del titular (acceso, rectificación, eliminación) según LOPDP.
  - Requisitos de producción: PRD-SEC-01, PRD-SEC-02, PRD-SEC-03, PRD-DATA-05, WEB-SEC-01, WEB-SEC-02, WEB-UX-02, WEB-LEG-01, API-02, API-03
- **M13** Observabilidad Integral, Métricas SRE, Backups y Recuperación ante Desastres (production)
  - Configurar la instrumentación completa con OpenTelemetry (trazas distribuidas, métricas golden signals y logs estructurados sin secretos), tableros de salud operativa, alertas por retraso en replicación de Inventary o drift de modelos, y procedimientos automatizados de respaldo y restauración PITR comprobados.
  - Requisitos de producción: PRD-OPS-01, PRD-DATA-03
- **M14** Empaquetado Reproducible, Ambientes y Guía Operativa de Despliegue (production)
  - Consolidar el empaquetado multi-contenedor reproducible con Dockerfile y Docker Compose para desarrollo, local y producción; verificar la ejecución completa de la suite de pruebas E2E e integración en CI; generar el manual de despliegue paso a paso (DEPLOY.md) y README.md; y comprobar la respuesta de los ambientes desplegados.
  - Requisitos de producción: PRD-QA-01, PRD-QA-02, PRD-QA-03, PRD-DOC-01, PRD-DEP-01, PRD-DEP-02, PRD-DEP-03, PRD-DEP-05, PRD-DEP-06, WEB-PERF-01

## Tasks

- Se planifican por módulo.

## Tests

- test_backend_health_endpoint: Comprobar que /health responde HTTP 200 con estado 'healthy' y conectividad a base de datos.
- test_tenant_row_level_security_isolation: Verificar que una consulta con tenant_id de una empresa no puede leer ni modificar filas de otra empresa.
- test_inventary_readonly_db_permissions: Probar que el usuario de Peitho sobre la base de datos de Inventary falla con PermissionError al intentar INSERT, UPDATE o DELETE.
- test_application_level_encryption_login: Validar el reto de login de un solo uso, descifrado con WebCrypto/ECDH y verificación de contraseña con Argon2id.
- test_idempotent_event_ingestion: Enviar el mismo evento dos veces con la misma idempotency key y confirmar que se procesa una sola vez sin duplicar registros.
- test_stockout_censored_demand_interval: Validar que los períodos con stock 0 se marquen como censurados y no reduzcan la estimación base de demanda.
- test_croston_and_tsb_forecasting_models: Comprobar que los algoritmos de demanda intermitente calculan intervalos P50, P80 y P90 superiores a cero para datos dispersos.
- test_commercial_guard_blocks_low_stock_proposal: Verificar que el motor de decisiones rechaza generar una propuesta si el stock disponible es menor a la demanda proyectada.
- test_no_outbound_communication_dispatcher_exists: Análisis estático de código para asegurar la inexistencia de clientes HTTP o SDKs de envío de WhatsApp, SMS o correo comercial.
- test_pgvector_hybrid_retrieval_tenant_filtering: Comprobar que la búsqueda de similitud vectorial aplica el filtro de tenant_id antes del cálculo de ranking.
- test_mcp_server_tool_allowlist_and_read_only: Verificar que las herramientas MCP expuestas son exclusivamente de lectura y borrador, rechazando cualquier invocación de mutación externa.
- test_b2b_oauth_client_credentials_and_rate_limit: Validar la emisión de tokens B2B y la respuesta HTTP 429 con cabecera Retry-After al superar la cuota configurada.
- test_audit_event_append_only_integrity: Comprobar que los registros de auditoría no pueden ser modificados ni eliminados por usuarios estándar.
- test_angular_wcag_accessibility_axe: Ejecutar axe-core en componentes Angular para asegurar cumplimiento de nivel AA sin violaciones críticas.
- test_multi_environment_configuration_loading: Verificar que la configuración se cargue estrictamente desde variables de entorno según el entorno activo.

## Rationale

- Se respeta íntegramente la exigencia del usuario de organizar el repositorio en 'app-peitho-ecuapp' (frontend Angular) y 'microservices-peitho-ecuapp' (backend Python/FastAPI), manteniendo una arquitectura limpia y modular que evita código espagueti.
- Se mantiene el principio no negociable de que Peitho opera en modo estrictamente de solo lectura respecto a Inventary, asegurando que no existan credenciales ni endpoints capaces de modificar stock, ventas o compras transaccionales.
- Se excluye formalmente cualquier mecanismo o credencial de envío/publicación directa en WhatsApp, SMS o redes sociales: Peitho es un cerebro analítico y prescriptivo que genera propuestas auditables para revisión humana.
- El aislamiento multiempresa se protege en múltiples capas (Defensa en Profundidad): resolución en token, validación en repositorios y Row-Level Security (RLS) en PostgreSQL, impidiendo fugas de datos entre organizaciones.
- La búsqueda híbrida con pgvector se acopla a las tablas relacionales existentes en PostgreSQL, permitiendo recuperación semántica de contexto de forma eficiente sin requerir motores adicionales complejos como Elasticsearch en la fase inicial.
- Todas las exigencias del production_profile quedan distribuidas y cubiertas en el mapa de producto, garantizando que el resultado final sea un producto robusto listo para producción.

## Warnings

- La cuenta de base de datos que conecte a Peitho con la réplica de Inventary debe crearse con permisos exclusivos de SELECT. La CI validará que cualquier mutación sea rechazada a nivel de motor de datos.
- El cifrado a nivel de aplicación en el login mediante WebCrypto no sustituye el uso obligatorio de HTTPS/TLS ni protege contra scripts maliciosos si la SPA sufriera vulnerabilidades XSS; se debe mantener una estricta Content Security Policy.
- Ninguna tarea de desarrollo debe crear endpoints ni librerías de despacho de campañas comerciales (correo, WhatsApp o SMS a clientes finales); toda propuesta comercial es únicamente para revisión y exportación por personal autorizado.

## Missing information

- Esquema exacto de tablas y versión de migración del commit actualmente desplegado en Inventary (se comenzará con el contrato canónico acordado y se verificará en el Módulo 3).
- Tarifario oficial vigente descargable de Meta WhatsApp para Ecuador al momento del despliegue en producción (se implementa como tabla de tarifas configurables sin requerir cambios de código fuente).

## Assumptions

- Inventary cuenta con una réplica de base de datos PostgreSQL accesible por red interna o túnel seguro para consultas analíticas de solo lectura.
- La infraestructura de base de datos de Peitho soporta la extensión oficial pgvector sobre PostgreSQL 16.
- El usuario opera en una sola empresa activa por sesión, con selección explícita y cambio de contexto auditado.
- Todos los timestamps se normalizan y almacenan internamente en UTC, presentándose en la zona horaria comercial de la empresa cliente (ej. America/Guayaquil).

## Memory changes

- Aplicado: rules.md — Incorporar las reglas obligatorias de calidad de Mnemosine (nomenclatura de tests para criterios de aceptación automáticos, eliminación estricta de código muerto y dependencias reemplazadas) y los principios no negociables de Peitho 3.0.
- Aplicado: contracts/schemas/database.md — Actualizar la documentación humana del modelo de base de datos para que coincida exactamente con contracts/schemas/database.json, conservando las notas del usuario y los patrones de Central Memory.
- Aplicado: architecture/overview.md — Documentar la visión de arquitectura completa del sistema Peitho 3.0: patrón monorepo modular, arquitectura hexagonal y en capas, desacoplamiento de lectura con Inventary, búsqueda híbrida pgvector, seguridad de aplicación, servidor MCP y APIs B2B.
- Aplicado: architecture/hierarchy.md — Establecer la jerarquía física de directorios respetando explícitamente los nombres exigidos por el usuario ('app-peitho-ecuapp' para frontend y 'microservices-peitho-ecuapp' para backend) y la separación de responsabilidades.
- Aplicado: architecture/requirements.md — Actualizar los requisitos técnicos formales para reflejar Python 3.12, FastAPI, SQLAlchemy 2, Alembic, PostgreSQL 16+ con pgvector, Angular 18+, TypeScript 5+, Docker y OpenTelemetry en los 3 ambientes del proyecto.
- Aplicado: security.md — Especificar la arquitectura de seguridad integral: aislamiento multiempresa por RLS, login con cifrado de aplicación en WebCrypto, hashing con Argon2id, protección estricta contra cross-tenant leaks, y cumplimiento de la LOPDP de Ecuador.
- Aplicado: testing.md — Definir la estrategia de testing multinivel (pirámide de pruebas, trazabilidad de criterios de aceptación con códigos Txx-Cxx, pruebas de RLS cross-tenant, accesibilidad y no-envío de campañas).
- Aplicado: knowledge/design.md — Actualizar el sistema de diseño visual de Angular para Peitho: componentes corporativos sobrios, cero alertas nativas (window.alert), steppers para procesos largos, modales para confirmaciones, skeletons y cumplimiento WCAG 2.2 AA.
- Aplicado: analysis/qa_checklist.md — Consolidar las listas de verificación de aseguramiento de calidad (QA) abarcando funcionalidad, accesibilidad, desempeño, seguridad estricta y preparación para despliegue en producción.
- Rechazado: contracts/schemas/database.json — una operación de anexar requiere un archivo existente
