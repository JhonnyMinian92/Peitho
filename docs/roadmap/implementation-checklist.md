# Plan integral de implementación — Peitho

**Objetivo:** completar Peitho como producto empresarial completo, seguro, observable, probado y operable. No es un MVP ni una lista para recortar a una primera demostración. Se permite desarrollar y entregar por incrementos, pero la definición de finalización corresponde al producto completo definido por los requisitos, no a un subconjunto reducido.

**Estado real al redactar este plan:** la rama contiene principalmente especificaciones y contratos documentales. Las carpetas de frontend/backend, Dockerfiles, Compose y servicios ejecutables aún deben crearse. Por tanto, todas las casillas parten pendientes; este documento no declara código implementado.

## Reglas de seguimiento

- [ ] significa que la tarea aún no está demostrada como completa. [x] solo se usa cuando existe evidencia verificable en el repositorio y la CI.
- Cada tarea debe dejar código mantenible, validación, pruebas automáticas, observabilidad, documentación y manejo seguro de errores según corresponda.
- Para cerrar una tarea, adjuntar referencia al PR/commit, tests ejecutados y resultado. Para criterios manuales, dejar los pasos de comprobación.
- Las tareas siguen los identificadores PEI-###. Añadir tareas descubiertas durante implementación en este plan y al grafo JSON; nunca omitirlas para preservar un porcentaje.
- Una tarea solo puede declararse no aplicable mediante ADR aprobado, explicación, impacto, alternativa y pruebas que acrediten que la exclusión no rompe otro requisito.
- La lista y el grafo complementan el estado operativo de tareas de Mnemosine descrito en AGENTS.md; no lo sustituyen.
- No tratar como implementado algo que solo esté dibujado, descrito en documentación, mockeado o devuelto por datos de ejemplo.

## Invariantes del producto

- [ ] Inventary conserva la autoridad transaccional sobre productos, compras, ventas, reservas, movimientos y existencias. Peitho no escribe en Inventary ni contabiliza operaciones allí.
- [ ] La ruta principal de datos de Inventary es réplica lógica dedicada hacia PostgreSQL de Peitho, acompañada por una API interna de solo lectura para validar puntualmente precio/disponibilidad.
- [ ] La autenticación inicial es local: Argon2id, MFA TOTP/passkeys, sesiones, recuperación, revocación, auditoría y aislamiento por empresa; no depender inicialmente de un proveedor externo de identidad.
- [ ] La inferencia por defecto es autogestionada. vLLM es el objetivo de producción si el hardware lo permite; Sentence Transformers genera embeddings en un servicio Python separado; Ollama se limita a desarrollo y pruebas locales.
- [ ] No configurar OpenAI API, Azure OpenAI, Anthropic, Vertex AI ni endpoints de inferencia de pago como dependencia predeterminada.
- [ ] Peitho prepara análisis, recomendaciones, borradores y propuestas revisables. No envía ni publica campañas, mensajes, correos, SMS, llamadas ni publicaciones sociales. La ejecución sucede fuera de Peitho y sus resultados pueden importarse.
- [ ] Los cálculos se disparan por eventos o programaciones; no ejecutar un LLM permanentemente en un bucle sin propósito ni controles.
- [ ] El stack debe ser modular, tipado, trazable y testeable; no crear funciones gigantes, spaghetti, duplicación funcional ni microservicios por moda.
- [ ] El frontend objetivo vive en app-peitho-ecuapp/ (Angular + TypeScript). El backend objetivo vive en microservices-peitho-ecuapp/ (Python + FastAPI modular, workers e integraciones).
- [ ] El desarrollo local usa Docker para aplicaciones y servicios. PostgreSQL se instala y ejecuta localmente en el host, fuera de Docker Compose, con pgvector disponible. Nunca arrancar un segundo PostgreSQL en Compose para el entorno local indicado por este plan.

## Fase 0 — Descubrimiento, requisitos y trazabilidad

**Dependencias:** ninguna. **Cierre de fase:** todas las capacidades tienen dueño técnico, requisitos, dependencias, criterios de aceptación y riesgos trazables.

- [ ] **PEI-001** Inventariar el repositorio, ramas, documentos, artefactos, contratos y estado real de cada componente sin asumir que un documento equivale a implementación.
- [ ] **PEI-002** Convertir docs/context.md, docs/flow.md, docs/functionality.md y la especificación enterprise en una matriz requisito → módulo → API/evento → modelo de datos → pantalla → prueba → evidencia.
- [ ] **PEI-003** Definir criterios de aceptación funcionales, de seguridad, rendimiento, privacidad y accesibilidad para cada requisito.
- [ ] **PEI-004** Construir y mantener el grafo de dependencias en docs/roadmap/task-graph.json; cada nodo debe enlazar tareas, documentos y criterios de cierre.
- [ ] **PEI-005** Revisar riesgos, amenazas, límites de confianza, superficies de ataque, datos personales y obligaciones de privacidad aplicables.
- [ ] **PEI-006** Registrar ADR para las decisiones técnicas aún abiertas: versiones de runtime/base de datos, estrategia de colas, almacenamiento de objetos, autenticación de servicios, observabilidad, backups, despliegue y recuperación.
- [ ] **PEI-007** Definir flujos completos y roles de usuario, incluyendo errores, permisos insuficientes, datos faltantes, datos obsoletos y recuperación.
- [ ] **PEI-008** Definir el criterio de priorización empresarial sin recortar el alcance final: los incrementos sirven para reducir riesgo y entregar valor, no para redefinir el producto como MVP.
- [ ] **PEI-009** Resolver por ADR las capacidades que necesitan hardware, dispositivos o datos específicos (por ejemplo, memoria háptica, olfativa o gustativa) antes de declararlas implementables o fuera de alcance.
- [ ] **PEI-010** Mantener un registro de decisiones, riesgos, supuestos, dependencias externas, responsables y evidencia de aceptación.

## Fase 1 — Estructura del repositorio y entorno local

**Dependencias:** PEI-001 a PEI-010. **Cierre de fase:** un desarrollador puede levantar las aplicaciones mediante Docker y usar la instalación local de PostgreSQL sin credenciales incrustadas.

- [ ] **PEI-011** Crear app-peitho-ecuapp/ con Angular + TypeScript, configuración estricta, gestor de paquetes y lockfile versionado.
- [ ] **PEI-012** Crear microservices-peitho-ecuapp/ con Python, FastAPI, pyproject.toml, dependencias fijadas, lockfile y tipado estático.
- [ ] **PEI-013** Crear estructura modular por dominios/capas, tests, migraciones, scripts y documentación; evitar un backend de funciones gigantes.
- [ ] **PEI-014** Crear Dockerfile multi-stage para Angular: construcción reproducible y runtime mínimo, usuario no root y healthcheck cuando aplique.
- [ ] **PEI-015** Crear Dockerfile para FastAPI, workers y tareas programadas con imagen mínima, dependencias fijadas, usuario no root y señales/cierre correctos.
- [ ] **PEI-016** Crear compose.yaml de desarrollo para frontend, API, worker/scheduler, embeddings y dependencias de aplicación necesarias; usar perfiles para componentes pesados.
- [ ] **PEI-017** Verificar que el Compose local NO define ni arranca un servicio postgres: la base PostgreSQL de desarrollo se ejecuta en el host.
- [ ] **PEI-018** Configurar conexión desde contenedores hacia el PostgreSQL del host mediante host.docker.internal; incluir host-gateway para Docker Engine en Linux cuando aplique.
- [ ] **PEI-019** Documentar configuración segura de listen_addresses, pg_hba.conf y firewall para permitir solo la red/gateway de desarrollo, sin exponer PostgreSQL a redes públicas.
- [ ] **PEI-020** Crear .env.example con nombres de variables y valores de muestra no secretos; ignorar .env, llaves, certificados, dumps y datos privados en Git.
- [ ] **PEI-021** Separar variables por local, test, staging y producción; validar configuración al iniciar y fallar con un mensaje claro si falta una variable obligatoria.
- [ ] **PEI-022** Fijar y documentar las versiones compatibles de PostgreSQL y pgvector mediante ADR; comprobar que la extensión está instalada en la instancia local.
- [ ] **PEI-023** Crear la base local peitho_dev, extensiones necesarias, roles de migración y aplicación con privilegio mínimo y contraseñas separadas.
- [ ] **PEI-024** Crear scripts reproducibles para comprobar Docker, conectividad PostgreSQL, migraciones, seed sintético, inicio, logs, pruebas y apagado seguro.
- [ ] **PEI-025** Añadir healthchecks de contenedores, dependencias con orden saludable, redes internas y límites básicos de recursos.
- [ ] **PEI-026** Verificar una instalación limpia siguiendo la guía de Docker en Windows, macOS y Linux que el proyecto decida soportar.
- [ ] **PEI-027** Probar que se puede reconstruir el stack desde cero sin datos personales reales ni depender de un servicio de pago.
- [ ] **PEI-028** Documentar resolución de errores frecuentes: puerto ocupado, DNS del host, firewall, pg_hba.conf, pgvector, permisos y credenciales.
- [ ] **PEI-029** Ejecutar y registrar la primera prueba real de docker compose build/up/ps/logs/down cuando los servicios hayan sido creados.

## Fase 2 — Arquitectura, contratos y cimientos del backend

**Dependencias:** Fase 1. **Cierre de fase:** todos los endpoints y trabajos respetan contratos, permisos, trazabilidad y tratamiento de errores uniformes.

- [ ] **PEI-030** Definir módulos de dominio: IAM/tenants, CRM, catálogo, integración Inventary, ingestión, analítica, ML, reglas, propuestas, experimentación, IA, integraciones y auditoría.
- [ ] **PEI-031** Definir límites entre dominio, aplicación y adaptadores de infraestructura; evitar dependencias de negocio hacia FastAPI, PostgreSQL o proveedores concretos.
- [ ] **PEI-032** Definir inyección de dependencias y unidades transaccionales; mantener transacciones cortas y control de concurrencia explícito.
- [ ] **PEI-033** Implementar contrato común JSON de solicitud y respuesta con request_id, timestamp UTC, status, payload o error estructurado.
- [ ] **PEI-034** Publicar OpenAPI para todos los endpoints, con esquemas de entrada/salida, códigos HTTP, permisos, paginación y ejemplos sintéticos.
- [ ] **PEI-035** Validar entradas con modelos Pydantic y respuestas con response models; nunca exponer campos internos sin autorización.
- [ ] **PEI-036** Implementar errores tipados, códigos estables, mensajes comprensibles y trazas internas sin filtrar secretos al cliente.
- [ ] **PEI-037** Añadir request_id/correlation_id a logs, trazas, eventos y tareas asíncronas.
- [ ] **PEI-038** Implementar paginación por cursores, filtros, ordenamientos permitidos, límites de payload y consultas eficientes.
- [ ] **PEI-039** Añadir versionado de APIs y política de compatibilidad de contratos.
- [ ] **PEI-040** Implementar idempotencia para endpoints de escritura controlada y consumidores de eventos que puedan recibir duplicados.
- [ ] **PEI-041** Definir contratos de eventos versionados, esquema, origen, timestamp, tenant, identificador único y política de evolución.
- [ ] **PEI-042** Implementar workers, scheduler, reintentos limitados, backoff, timeouts y Dead-Letter Queue (DLQ) con reproducción auditada.
- [ ] **PEI-043** Implementar /health, /ready y sondas de trabajadores que no filtren estructura interna o secretos.
- [ ] **PEI-044** Implementar transacciones, locks/concurrencia, límites de ejecución y protección contra tareas duplicadas.
- [ ] **PEI-045** Añadir migraciones de esquema versionadas, revisión de compatibilidad y procedimiento de reversión/forward-fix.

## Fase 3 — Frontend, diseño y experiencia de usuario

**Dependencias:** PEI-011, PEI-033 a PEI-039. **Cierre de fase:** los recorridos funcionales completos tienen estados accesibles, validación y errores explicados.

- [ ] **PEI-046** Crear shell Angular, navegación por permisos, módulos lazy-loaded, layouts y rutas protegidas sin confiar en la interfaz como control de seguridad.
- [ ] **PEI-047** Crear sistema visual coherente: tokens, tipografía, espaciado, iconografía, tablas, filtros, botones, formularios y componentes reutilizables.
- [ ] **PEI-048** Crear clientes API tipados y un interceptor común de autenticación, correlación, errores y reintentos seguros.
- [ ] **PEI-049** Prohibir alert() nativo; usar notificaciones accesibles, confirmaciones de acciones destructivas y mensajes contextuales.
- [ ] **PEI-050** Mostrar skeletons de carga, estados vacíos, errores recuperables, resultados parciales, stale/degraded y acciones para continuar.
- [ ] **PEI-051** Usar modales solo para tareas breves. Formularios extensos o con varios conjuntos de datos usan stepper con progreso, validación por paso, resumen final y recuperación.
- [ ] **PEI-052** Validar formularios en cliente y servidor; explicar cómo corregir el error y conservar valores seguros al fallar una petición.
- [ ] **PEI-053** Implementar tablas grandes con paginación, filtros, ordenamiento, selección masiva segura y vistas adaptables.
- [ ] **PEI-054** Asegurar navegación por teclado, etiquetas, foco, contraste WCAG 2.1 AA, estados no dependientes solo de color y lectores de pantalla.
- [ ] **PEI-055** Probar responsive desde 360 px hasta escritorio; evitar desbordes, densidad visual excesiva y acciones difíciles de descubrir.
- [ ] **PEI-056** Implementar español de Ecuador, zona horaria configurable (por defecto America/Guayaquil), moneda y formato de fechas explícitos.
- [ ] **PEI-057** Crear flujos de login, dashboard, perfiles, ventas, inventario analítico, CRM, propuestas, experimentos, administración y auditoría.
- [ ] **PEI-058** Añadir confirmación de consecuencias y vista previa antes de exportar datos o aprobar propuestas sensibles.
- [ ] **PEI-059** Evitar visualizaciones engañosas: mostrar unidad, intervalo, fecha del dato, estado de fuente y diferencia entre observado, inferido y pronosticado.
- [ ] **PEI-060** Probar accesibilidad y journeys con Playwright/axe o herramientas equivalentes en CI, no solo inspección manual.

## Fase 4 — Multiempresa, identidad, autorización y privacidad

**Dependencias:** Fase 2. **Cierre de fase:** aislamiento de tenant probado desde API, tareas, búsqueda semántica, exportaciones y herramientas de agentes.

- [ ] **PEI-061** Implementar organización/tenant, membresías, usuarios, roles, permisos y ciclo de vida de invitaciones/activación/desactivación.
- [ ] **PEI-062** Implementar autenticación local con hash Argon2id, política de contraseñas, protección de fuerza bruta y mensajes que no revelen si una cuenta existe.
- [ ] **PEI-063** Implementar TOTP y passkeys/WebAuthn con recuperación, revocación y MFA obligatorio para administración y acciones de riesgo.
- [ ] **PEI-064** Implementar sesiones seguras, expiración absoluta/inactiva, rotación, cookies seguras, cierre de sesión global y revocación de dispositivos.
- [ ] **PEI-065** Implementar recuperación de cuenta con tokens de un solo uso, expiración, controles antiabuso y auditoría.
- [ ] **PEI-066** Implementar RBAC y ABAC para funciones y atributos sensibles, incluyendo segregación de leads propios, costes, exportaciones y administración.
- [ ] **PEI-067** Aplicar Row-Level Security en PostgreSQL con políticas fail-closed y contexto de tenant transaccional; probar conexiones reutilizadas y pool.
- [ ] **PEI-068** Derivar tenant y scopes de identidad verificada; jamás confiar en tenant_id recibido del navegador como autoridad.
- [ ] **PEI-069** Crear auditoría append-only de accesos a PII, exportaciones, cambios críticos, aprobación/rechazo, gestión de permisos y actividad de agentes.
- [ ] **PEI-070** Cifrar datos/campos sensibles en aplicación cuando corresponda con gestión y rotación de claves separada; TLS sigue siendo obligatorio y contraseñas se hashean, no se cifran reversiblemente.
- [ ] **PEI-071** Implementar gestión de secretos fuera del repositorio, rotación, scopes mínimos y limpieza de secretos en logs, errores, traces y prompts.
- [ ] **PEI-072** Configurar CORS de orígenes conocidos, CSRF según arquitectura, CSP, cabeceras de seguridad, límites de peticiones y controles de subida de archivos.
- [ ] **PEI-073** Definir retención, exportación, rectificación, anonimización y purga por propósito; registrar consentimiento y base legitimadora cuando aplique.
- [ ] **PEI-074** Probar aislamiento multi-tenant en endpoints, jobs, webhooks, eventos, RAG/vector search, cachés, archivos y exportaciones.
- [ ] **PEI-075** Implementar permisos de servicio para réplica, API interna Inventary, workers, MCP y APIs B2B con credenciales independientes.

## Fase 5 — PostgreSQL, modelo canónico e integración con Inventary

**Dependencias:** Fase 1, Fase 2 y Fase 4. **Cierre de fase:** datos trazables, actualizados según watermark, reconciliados y con escritura en Inventary imposible para Peitho.

- [ ] **PEI-076** Diseñar modelo relacional de tenant, usuarios, productos, variantes, categorías, locales/almacenes, listas de precios, costes analíticos, compras, recepciones, ventas, líneas, devoluciones, reservas y movimientos reflejados.
- [ ] **PEI-077** Definir claves externas de origen, IDs canónicos, tipos de dato, moneda, zona horaria, unidades, nulabilidad y reglas de normalización.
- [ ] **PEI-078** Crear migraciones, índices, constraints, claves únicas idempotentes y estrategia de evolución sin pérdida.
- [ ] **PEI-079** Habilitar pgvector en PostgreSQL local y ambientes objetivo; comprobar dimensiones/indexación del modelo de embedding seleccionado.
- [ ] **PEI-080** Definir publicación y suscripción de replicación lógica dedicadas con allowlist de tablas/columnas, permisos mínimos y procedimiento de bootstrap.
- [ ] **PEI-081** Documentar que la replicación lógica no copia DDL automáticamente; coordinar cambios de esquema entre publicador y suscriptor con compatibilidad probada.
- [ ] **PEI-082** Implementar backfill inicial, checkpoint/watermark, sincronización incremental, tratamiento de borrados y deduplicación/reordenamiento seguro.
- [ ] **PEI-083** Registrar por conjunto de datos source_updated_at, received_at, applied_at, watermark/LSN, lag, schema_version, snapshot_id y estado/fecha de conciliación.
- [ ] **PEI-084** Monitorizar retraso de replicación, posiciones de envío/aplicación, slots, retención/crecimiento de WAL, errores, disco y recuperación.
- [ ] **PEI-085** Implementar reconciliación periódica de recuentos, unidades, importes y saldos por ventana, producto, almacén y documento según disponibilidad de datos.
- [ ] **PEI-086** Implementar recuperación/reinicio/resincronización por lotes, con límite de recursos, auditoría y pruebas de caída de suscripción y pérdida de rangos.
- [ ] **PEI-087** Implementar API interna Inventary de solo lectura, autenticada, rate-limited, auditada y acotada a operaciones de consulta permitidas; nunca SQL arbitrario.
- [ ] **PEI-088** Configurar cuenta de lectura separada en Inventary y probar por permisos de base de datos que INSERT, UPDATE, DELETE, TRUNCATE, DDL y funciones de escritura fallan.
- [ ] **PEI-089** Bloquear o dejar no confirmadas las propuestas sensibles a precio/stock si la fuente está stale, no conciliada, desconocida o incompatible.
- [ ] **PEI-090** Exponer frescura, lag y procedencia en API/UI; nunca etiquetar un snapshot como actual sin evidencia.
- [ ] **PEI-091** Modelar cancelaciones, anulaciones, reembolsos y devoluciones para obtener venta neta sin duplicar efectos.
- [ ] **PEI-092** Mantener historial temporal de stock y marcar períodos de rotura; cero ventas con stock cero no significa cero demanda.
- [ ] **PEI-093** Calcular ATP usando stock vendible, reservas, cuarentena y recepciones confirmadas solo cuando las semánticas de origen estén verificadas.
- [ ] **PEI-094** Estimar lead time de proveedores con historial y comunicar cobertura/incertidumbre.
- [ ] **PEI-095** Implementar detección de sobrestock, días de cobertura, riesgo de obsolescencia y tratamiento explícito de devoluciones todavía no vendibles.

## Fase 6 — Ingesta, normalización, eventos y conectores

**Dependencias:** Fase 5 y contratos de Fase 2. **Cierre de fase:** conectores autenticados, idempotentes, monitoreados y con reproceso seguro.

- [ ] **PEI-096** Crear pipelines para catálogos, ERP, ventas, compras, logística, recepciones, transferencias, despachos, reservas y devoluciones mediante contratos permitidos.
- [ ] **PEI-097** Crear ingestión de eventos first-party de web/ecommerce (vista de producto, carrito, abandono, checkout) respetando consentimiento y finalidad.
- [ ] **PEI-098** Crear adaptadores para importar métricas agregadas de plataformas publicitarias, con configuración de ventana de atribución y procedencia.
- [ ] **PEI-099** Crear importadores autorizados de metadatos y texto de conversaciones (WhatsApp, correo, SMS, llamadas) desde sistemas externos; no enviar comunicaciones.
- [ ] **PEI-100** Canonicalizar fechas UTC/zonas horarias, moneda, unidades, claves de cliente/producto y formatos de identificadores.
- [ ] **PEI-101** Validar calidad, campos obligatorios, límites, duplicados, anomalías, esquemas y privacidad antes de aceptar datos.
- [ ] **PEI-102** Aplicar claves de idempotencia, retención de checkpoint, reintentos exponenciales con jitter y DLQ con causa saneada.
- [ ] **PEI-103** Implementar replay controlado y auditable de mensajes DLQ, sin duplicar efectos ni reabrir accesos revocados.
- [ ] **PEI-104** Versionar conectores y contratos; probar fallos, expiración de credenciales, rate limits, timeouts y cambios de esquema.
- [ ] **PEI-105** Crear tablero de estado por fuente: última recepción, último evento aplicado, frescura, lag, errores y conciliación.
- [ ] **PEI-106** Definir límites de tamaño, protección anti-SSRF/URL maliciosa para importaciones y escaneo de archivos cuando proceda.
- [ ] **PEI-107** Mantener fixture y datos sintéticos reproducibles para cada conector; nunca copiar datos reales al repositorio.

## Fase 7 — CRM, identidad comercial y vista 360

**Dependencias:** Fases 4 a 6. **Cierre de fase:** cada perfil y actividad se encuentra autorizado, se puede deshacer una fusión y la línea de tiempo conserva procedencia.

- [ ] **PEI-108** Implementar personas, organizaciones/cuentas, contactos, consentimientos, preferencias, oportunidades, actividades y relaciones comerciales propias de Peitho.
- [ ] **PEI-109** Resolver identidad mediante reglas determinísticas aprobadas; no fusionar perfiles por puntuaciones probabilísticas sin autorización.
- [ ] **PEI-110** Implementar separación/unmerge con historial de decisiones, reconstrucción de relaciones y pruebas de colisión por teléfono/correo compartido.
- [ ] **PEI-111** Implementar embudos configurables por tenant con etapas, criterios de entrada/salida, permisos y auditoría.
- [ ] **PEI-112** Implementar jerarquías B2B padre/hijo, contactos múltiples, decisores, rol de compra y relaciones entre organizaciones.
- [ ] **PEI-113** Implementar tareas, recordatorios, SLA comercial, prioridad, asignación, vencimiento y escalamiento.
- [ ] **PEI-114** Tipificar objeciones y motivos de pérdida para alimentar análisis posteriores con datos trazables.
- [ ] **PEI-115** Gestionar opt-in/opt-out, propósito, evidencia, fecha, canal y preferencias de frecuencia; respetar el opt-out en cualquier recomendación.
- [ ] **PEI-116** Crear timeline cronológica de navegación, anuncios, interacciones importadas, pedidos, reclamos, devoluciones, tareas y propuestas.
- [ ] **PEI-117** Procesar notas extensas o audio autorizado, transcribir mediante componente local/autogestionado si se habilita y redactar PII antes de indexar.
- [ ] **PEI-118** Crear vista 360 con historial, valor, actividad, consentimientos, riesgo y recomendaciones; separar hechos, inferencias y estimaciones.
- [ ] **PEI-119** Aplicar permiso por propietario, equipo, cuenta y dato sensible en API, búsquedas, exportaciones y pantallas.
- [ ] **PEI-120** Probar creación/edición, fusiones, unmerge, etapas, SLA, opt-out, auditoría y aislamiento de tenant de extremo a extremo.

## Fase 8 — Contexto operativo y métricas de negocio

**Dependencias:** Fases 5 a 7. **Cierre de fase:** todas las métricas tienen fórmula, datos base, zona temporal, reglas de neteo y explicación reproducible.

- [ ] **PEI-121** Definir catálogo de métricas y fórmulas para preventa, ventas y postventa por producto, cliente, canal, ubicación, día y hora.
- [ ] **PEI-122** Separar venta bruta, anulaciones, devoluciones, reembolsos, impuestos, descuentos, costes y contribución neta según datos disponibles.
- [ ] **PEI-123** Analizar estacionalidad, horarios, día de semana, promociones, cambios de precio y segmentos sin inferir causalidad de una simple correlación.
- [ ] **PEI-124** Calcular cohortes, RFM, frecuencia de compra, recurrencia y CLV con supuestos y horizonte visibles.
- [ ] **PEI-125** Construir features con fecha de corte, procedencia, reglas de actualización y protección contra fuga de información futura.
- [ ] **PEI-126** Implementar clasificación no supervisada solo con selección de variables, escalado, validación, explicación de clusters y control de estabilidad.
- [ ] **PEI-127** Implementar recomendaciones de productos complementarios/bundles basadas en compra conjunta observada y reglas de compatibilidad verificadas.
- [ ] **PEI-128** Implementar optimización de rutas cuando haya coordenadas, capacidades, ventanas de entrega y restricciones suficientes; mostrar supuestos y alternativas.
- [ ] **PEI-129** Diferenciar stock no disponible, cero demanda observada y datos faltantes en todas las métricas.
- [ ] **PEI-130** Crear paneles con filtros y exportación autorizada; mostrar unidad, rango temporal, última actualización y calidad de fuente.
- [ ] **PEI-131** Validar métricas con casos pequeños calculados manualmente y reconciliación contra fuentes autorizadas.

## Fase 9 — Motor de Machine Learning y gobierno de modelos

**Dependencias:** Fase 8 y plataforma de jobs. **Cierre de fase:** modelos comparados contra baseline, evaluados fuera de muestra, calibrados y monitoreados.

- [ ] **PEI-132** Implementar pronóstico jerárquico de demanda para SKU, familia, almacén y canal con agregación coherente.
- [ ] **PEI-133** Establecer Seasonal Naive como baseline; no promover un modelo complejo que no lo supere según métrica y validación acordadas.
- [ ] **PEI-134** Implementar modelos de demanda intermitente (Croston/SBA/TSB según validación) y pruebas para series de baja rotación.
- [ ] **PEI-135** Generar intervalos P50/P80/P90 y comprobar calibración; no presentar una predicción puntual como certeza.
- [ ] **PEI-136** Incorporar periodos de stockout como demanda censurada y probar que no se confundan con falta de demanda.
- [ ] **PEI-137** Implementar scoring de leads con vectores separados de intención, valor comercial y urgencia, explicación de señales y control de sesgo.
- [ ] **PEI-138** Estimar probabilidad de cierre del pipeline y tiempo restante usando validación temporal, calibración y análisis de error.
- [ ] **PEI-139** Modelar abandono/churn temporal con análisis de supervivencia y ventanas de predicción verificables.
- [ ] **PEI-140** Implementar afinidad o market basket con límites de soporte/confianza y reglas de compatibilidad de producto.
- [ ] **PEI-141** Validar optimización de rutas contra heurísticas base, restricciones operativas y métricas de distancia/tiempo/coste.
- [ ] **PEI-142** Implementar registro de modelo: versión, datos, features, métricas, fecha, licencia, código, artefacto y aprobación.
- [ ] **PEI-143** Monitorizar drift de datos y rendimiento, establecer umbrales y fallback seguro a baseline/reglas.
- [ ] **PEI-144** Automatizar entrenamiento/reentrenamiento solo con ventanas y criterios aprobados; conservar reproducibilidad y rollback.
- [ ] **PEI-145** Probar leakage, selección temporal, clases desbalanceadas, datos faltantes, calibración, estabilidad y degradación.
- [ ] **PEI-146** Exponer explicación, incertidumbre, versión de modelo, datos de entrada y fecha de cálculo en cada resultado analítico.
- [ ] **PEI-147** Impedir que un modelo ML autorice stock, margen, consentimiento o permisos: esas guardas pertenecen a reglas determinísticas.

## Fase 10 — Motor prescriptivo y decisiones comerciales

**Dependencias:** Fases 5, 8 y 9. **Cierre de fase:** toda propuesta es reproducible, económicamente evaluada y bloqueada cuando viola guardas duras.

- [ ] **PEI-148** Implementar motor de reglas versionadas con orden de evaluación, prioridad, explicación y pruebas determinísticas.
- [ ] **PEI-149** Bloquear propuestas que incumplan margen mínimo, consentimiento, disponibilidad/antigüedad de stock, límites de riesgo o política del tenant.
- [ ] **PEI-150** Simular escenarios conservador/base/optimista restando descuento, coste de canal importado/configurado, fulfillment y soporte según datos disponibles.
- [ ] **PEI-151** Documentar fuente, fecha y vigencia de tarifas de canal; si no hay tarifa fiable, declarar el supuesto en vez de inventar un coste.
- [ ] **PEI-152** Implementar guardas de fatiga/saturación por cliente, canal, frecuencia y periodo con opt-out y excepciones auditadas.
- [ ] **PEI-153** Detectar riesgo de canibalización entre venta orgánica y promocionada con supuestos y evidencia.
- [ ] **PEI-154** Mostrar cada recomendación con hipótesis, datos empleados, regla activada, coste previsto, contribución estimada e incertidumbre.
- [ ] **PEI-155** Recalcular o invalidar recomendaciones cuando cambien datos, stock, coste, consentimiento o versión de regla.
- [ ] **PEI-156** Mantener separación técnica entre aprobar/preparar una propuesta y ejecutar la comunicación; no añadir un endpoint de envío/publicación.

## Fase 11 — Pasarela de IA local, búsqueda semántica y memoria operativa

**Dependencias:** Fases 1, 4, 5 y 8. **Cierre de fase:** inferencia local observable, búsqueda filtrada por permisos, versiones reproducibles y memoria evaluada.

- [ ] **PEI-157** Definir interfaz de AI Gateway desacoplada del modelo, contrato de solicitud/respuesta, timeout, cancelación, cuotas y límites de contexto.
- [ ] **PEI-158** Integrar vLLM como runtime de producción previsto y Ollama como perfil de desarrollo; nunca cambiar automáticamente a inferencia de pago.
- [ ] **PEI-159** Seleccionar modelo LLM concreto mediante benchmarks de español y vocabulario comercial ecuatoriano, licencia, calidad, latencia, consumo y throughput documentados.
- [ ] **PEI-160** Crear servicio Python separado para embeddings con Sentence Transformers y APIs tipadas, healthcheck, límites y modelo/revisión fijados.
- [ ] **PEI-161** Registrar modelo, revisión, dimensiones, idioma/licencia, versión de corpus e índice vectorial; no mezclar embeddings incompatibles.
- [ ] **PEI-162** Implementar reindexación idempotente, evaluación Recall@k/precisión, cambio atómico de índice y rollback al cambiar el encoder.
- [ ] **PEI-163** Aplicar tenant, permisos, finalidad, vigencia y clasificación de datos antes de recuperar documentos para RAG; comprobarlo también en SQL y pruebas negativas.
- [ ] **PEI-164** Crear protección ante prompt injection, instrucciones no confiables, datos sensibles en contexto y herramientas no autorizadas; el LLM nunca ejecuta SQL arbitrario.
- [ ] **PEI-165** Aplicar salida estructurada y validación determinística posterior al modelo; ante error, rechazo o timeout, usar fallback seguro.
- [ ] **PEI-166** Versionar prompts, plantillas, contexto recuperado y configuración de generación sin almacenar secretos ni PII innecesaria.
- [ ] **PEI-167** Probar que la configuración predeterminada no necesita credenciales de inference providers externos y restringe egress no autorizado.
- [ ] **PEI-168** Implementar las capacidades de memoria definidas en docs/roadmap/memory-capability-checklist.md, con límites de privacidad, propósito, caducidad y auditoría.
- [ ] **PEI-169** Separar memoria de sesión/operativa de memoria persistente; controlar escritura, lectura, olvido/retención, indexado y reconstrucción.
- [ ] **PEI-170** Evaluar recuperación y actualización de memoria con casos sintéticos: precisión, recall, frescura, autorización, contaminación cruzada y borrado.
- [ ] **PEI-171** Implementar observabilidad de IA: latencia, tokens/recursos, tasa de errores, evaluación de salida, rechazo, versión y origen del contexto.
- [ ] **PEI-172** Documentar dimensionamiento de CPU/GPU/RAM/disco, límites de concurrencia, coste operativo, estrategia de modelos y umbrales de degradación.

## Fase 12 — Studio de propuestas, generación y aprobación humana

**Dependencias:** Fases 7, 10 y 11. **Cierre de fase:** borradores y propuestas son colaborativos y auditables; no se pueden enviar desde Peitho.

- [ ] **PEI-173** Generar borradores de copy para redes sociales, SMS, email y guiones de llamada, siempre marcados como borradores.
- [ ] **PEI-174** Generar variaciones A/B/n con hipótesis, audiencia autorizada, diferencias explícitas y métrica esperada.
- [ ] **PEI-175** Generar briefs visuales estructurados (paleta sugerida, mensaje, formatos, fotografía requerida y restricciones), sin afirmar que la pieza fue publicada.
- [ ] **PEI-176** Generar contraargumentos de ventas con evidencia verificable y prohibir características, certificaciones, disponibilidad, descuentos o urgencia inventados.
- [ ] **PEI-177** Implementar tono de marca, restricciones léxicas, idioma, formato, palabras prohibidas y políticas por tenant.
- [ ] **PEI-178** Validar compliance comercial/legal con reglas determinísticas y revisión humana para los casos de riesgo.
- [ ] **PEI-179** Enrutar propuestas a Marketing, Ventas, Finanzas u otros revisores según riesgo, canal, presupuesto y política.
- [ ] **PEI-180** Implementar comentarios inline, edición, diferencias de versión, comparación A/B y registro de autor.
- [ ] **PEI-181** Versionar propuesta, datos usados, watermark, modelos, prompts, reglas y aprobadores de forma inmutable.
- [ ] **PEI-182** Registrar manualmente el check-in de ejecución externa: plataforma, fecha, variantes y responsable, sin ejecutar la campaña.
- [ ] **PEI-183** Importar resultados reales de la ejecución externa y distinguirlos de proyecciones, datos estimados y métricas atribuidas.
- [ ] **PEI-184** Probar que los botones, APIs, tools MCP y jobs no pueden enviar, llamar, publicar ni iniciar campañas accidentalmente.

## Fase 13 — Medición causal y aprendizaje organizacional

**Dependencias:** Fases 8 a 12. **Cierre de fase:** conclusiones causales solo se publican cuando el diseño y los datos permiten sustentarlas.

- [ ] **PEI-185** Diseñar experimentos con hipótesis, métrica primaria, métricas de guarda, duración, presupuesto y criterios de parada.
- [ ] **PEI-186** Calcular tamaño de muestra/potencia estadística y documentar supuestos antes de iniciar el experimento.
- [ ] **PEI-187** Implementar asignación y lectura de grupos de control/test con detección de contaminación y análisis predefinido.
- [ ] **PEI-188** Medir uplift/incrementalidad con intervalos de incertidumbre y comparación contra baseline.
- [ ] **PEI-189** Añadir controles sintéticos para casos donde un A/B puro no sea factible, marcando límites de validez.
- [ ] **PEI-190** Implementar Marketing Mix Modeling solo cuando existan series históricas suficientes y validar supuestos macroeconómicos.
- [ ] **PEI-191** Mantener centro de evidencia indexado de experimentos positivos y negativos, diseño, datos, modelo, decisión y resultado.
- [ ] **PEI-192** Evitar atribuir causalidad desde correlación, click-through u observaciones sin control adecuado.

## Fase 14 — APIs B2B, webhooks, SSE y agentes externos

**Dependencias:** Fases 2 y 4. **Cierre de fase:** integraciones autenticadas, versionadas, limitadas, idempotentes y auditadas.

- [ ] **PEI-193** Implementar REST versionado para dominios aprobados, con OpenAPI, scopes, paginación y compatibilidad.
- [ ] **PEI-194** Implementar GraphQL solo con esquema, límites de profundidad/coste, autorización por campo y justificación documentada de los casos de uso.
- [ ] **PEI-195** Implementar gestión de clientes B2B, credenciales OAuth 2.1, scopes, cuotas, rotación/revocación y entorno sandbox.
- [ ] **PEI-196** Implementar ETag/caché condicional, rate limits por tenant/credencial, límites de payload y cursor pagination.
- [ ] **PEI-197** Implementar webhooks salientes solo para eventos no sensibles permitidos, firma HMAC, timestamp, anti-replay y reintentos exponenciales.
- [ ] **PEI-198** Implementar SSE para progreso/actualizaciones cuando corresponda, con autorización, reconexión y límites de conexiones.
- [ ] **PEI-199** Exponer servidor MCP con tools/resources tipadas de solo lectura y generación de borradores; verificar permisos en cada llamada.
- [ ] **PEI-200** Prohibir SQL arbitrario, elevación de privilegios, escritura en Inventary, publicación y envío de comunicaciones en herramientas de agentes.
- [ ] **PEI-201** Auditar llamada, principal, tenant, scope, herramienta, resultado, latencia y decisión de autorización sin registrar secretos.
- [ ] **PEI-202** Probar revocación, replay, firma inválida, duplicados, expiración, cuota, aislamiento, reintentos y fallos del destino.

## Fase 15 — Observabilidad, rendimiento y operación

**Dependencias:** fases anteriores. **Cierre de fase:** fallos detectables, recuperables y diagnosticables sin exponer datos sensibles.

- [ ] **PEI-203** Implementar logs estructurados con niveles, correlación y redacción de PII/secretos.
- [ ] **PEI-204** Implementar métricas y trazas para API, workers, conectores, replicación, ML, AI Gateway, PostgreSQL y acciones de usuario.
- [ ] **PEI-205** Definir SLO, presupuesto de errores, alertas y paneles operativos para disponibilidad, latencia, frescura, errores y cola.
- [ ] **PEI-206** Añadir readiness/liveness separadas; no marcar healthy si una dependencia crítica está degradada.
- [ ] **PEI-207** Probar timeouts, circuit breaker, backpressure, cancelación, límites de concurrencia y protección ante tormentas de reintentos.
- [ ] **PEI-208** Analizar índices, consultas, N+1, paginación, locks, pool de conexiones y consultas vectoriales con perfilado real.
- [ ] **PEI-209** Ejecutar pruebas de carga/estrés/soak, establecer capacidad de CPU/RAM/GPU/disco/IO y documentar límites y escalamiento.
- [ ] **PEI-210** Implementar copias de seguridad cifradas, políticas de retención y procedimiento reproducible de restauración.
- [ ] **PEI-211** Ejecutar una restauración completa y medir RPO/RTO; documentar estrategia de recuperación de réplica, índices vectoriales y artefactos ML.
- [ ] **PEI-212** Documentar rotación de secretos/certificados, actualización de modelos, migraciones, rollback, incidentes y recuperación de servicio.
- [ ] **PEI-213** Probar despliegues repetibles, migraciones compatibles, apagado ordenado, tareas en curso y recuperación tras reinicio.
- [ ] **PEI-214** Revisar política de datos, accesos administrativos, privacidad, exportación, borrado y auditoría en operación real.

## Fase 16 — Calidad, seguridad y CI/CD

**Dependencias:** aplica a todo el proyecto. **Cierre de fase:** todo requisito crítico tiene pruebas automáticas y ninguna liberación pasa con controles bloqueantes fallidos.

- [ ] **PEI-215** Configurar CI para instalar desde lockfiles, lint, typecheck, build y ejecutar tests en cada cambio.
- [ ] **PEI-216** Exigir tests unitarios para reglas, métricas, validación, normalización, algoritmos, scoring y lógica determinística.
- [ ] **PEI-217** Ejecutar tests de integración contra PostgreSQL real de CI con pgvector y migraciones limpias.
- [ ] **PEI-218** Ejecutar tests de contrato para OpenAPI, envelopes JSON, eventos, webhooks y compatibilidad consumidor/productor.
- [ ] **PEI-219** Ejecutar tests de integración/recuperación de replicación: lag, WAL retenido, schema incompatible, duplicados, missing ranges y resincronización.
- [ ] **PEI-220** Ejecutar tests de seguridad para autorización, RLS, aislamiento tenant, SSRF, SQLi, XSS, CSRF, rate limits y subida de archivos.
- [ ] **PEI-221** Ejecutar tests negativos de Inventary y agentes que prueben imposibilidad de escritura, DDL o ejecución de SQL arbitrario.
- [ ] **PEI-222** Ejecutar backtests temporales, baseline, incertidumbre, calibración, leakage, drift y regresión de modelos.
- [ ] **PEI-223** Ejecutar tests de recuperación de memoria, permisos, tenant, retención, borrado y contaminación entre organizaciones.
- [ ] **PEI-224** Ejecutar E2E Playwright para journeys críticos y errores típicos; no validar solo el camino feliz.
- [ ] **PEI-225** Ejecutar pruebas visuales/responsive y accesibilidad automatizada y manual para pantallas críticas.
- [ ] **PEI-226** Auditar dependencias, lockfiles, vulnerabilidades, imágenes de contenedor, licencias y SBOM; bloquear vulnerabilidades críticas/altas no aceptadas.
- [ ] **PEI-227** Medir cobertura útil por riesgo/requisito; no sustituir cobertura real por un porcentaje global arbitrario.
- [ ] **PEI-228** Añadir prueba de regresión para cada bug corregido; eliminar código muerto, duplicado, comentado y dependencias sin uso.
- [ ] **PEI-229** Asegurar que los tests nunca usan datos ni credenciales de producción.
- [ ] **PEI-230** Guardar reportes de CI, cobertura, benchmarks, escáneres y artefactos de release como evidencia consultable.

## Fase 17 — Documentación final, despliegue y aceptación del producto completo

**Dependencias:** todas las fases anteriores. **Cierre de fase:** no quedan requisitos pendientes sin decisión explícita, y el producto puede instalarse, operarse, recuperarse y mantenerse.

- [ ] **PEI-231** Actualizar README con prerrequisitos, Docker local, PostgreSQL local, variables, migraciones, arranque, pruebas, troubleshooting y arquitectura.
- [ ] **PEI-232** Documentar despliegue local, staging y producción, perfiles Compose, redes, secretos, almacenamiento, healthchecks y actualización.
- [ ] **PEI-233** Documentar modelo de datos, ERD, migraciones, permisos, retención, restauración y datos sintéticos.
- [ ] **PEI-234** Documentar API OpenAPI, eventos, webhooks, MCP, OAuth, límites, errores y ejemplos sin datos reales.
- [ ] **PEI-235** Documentar runbooks de Inventary replication, DLQ, datos obsoletos, recuperación, AI Gateway, embeddings, drift y backups.
- [ ] **PEI-236** Validar que cada requisito de la matriz PEI-002 está satisfecho con código real, pruebas y documentación o tiene una exclusión aprobada.
- [ ] **PEI-237** Completar los recorridos de extremo a extremo de preventa, venta, postventa, CRM, stock analítico, propuestas, aprobación y medición.
- [ ] **PEI-238** Ejecutar revisión de seguridad/privacidad final y comprobar separación entre tenants y permisos de extremo a extremo.
- [ ] **PEI-239** Ejecutar restauración, rollback, carga, observabilidad, migraciones y validación de experiencia de usuario antes del release.
- [ ] **PEI-240** Confirmar que todas las capacidades requeridas, incluidas memoria, IA, integración, analytics, aprobación y APIs, están implementadas o formalmente justificadas como no aplicables.
- [ ] **PEI-241** Cerrar todos los bugs críticos/altos y registrar aceptación de riesgos residuales; ninguna tarea bloqueante puede quedarse abierta.
- [ ] **PEI-242** Publicar release reproducible con versión, commit, changelog, configuración y reportes de CI.
- [ ] **PEI-243** Confirmar que el producto puede ser mantenido por otro agente/desarrollador usando el contrato de ejecución, el grafo y los runbooks sin depender de conocimiento oral.


## Fase 18 — Contrato versionado del esquema fuente de Inventary y detección de drift

**Dependencias:** PEI-001 a PEI-010 para trazabilidad; debe cerrarse antes de activar la publicación definitiva de Inventary. **Fuente de referencia al documentar:** `JhonnyMinian92/inventary@083fc9f4ec97b8673de9f0e2adef6af195df65e5`, migraciones Flyway V1–V18 bajo `src/main/resources/db/migration`, 49 tablas declaradas por SQL. Ver `docs/integration/inventary-database-schema.md` y `docs/contracts/schemas/inventary-source-schema-catalog-v18.json`.

- [ ] **PEI-244** Mantener el catálogo de tablas/campos de Inventary derivado de los archivos Flyway fuente, con commit de origen, lista y SHA de migraciones, claves, restricciones y relaciones; nunca inferir esquema desde nombres de endpoints ni documentación obsoleta.
- [ ] **PEI-245** Crear una matriz tabla/columna → propósito Peitho → proyección analítica → clasificación de sensibilidad → base de acceso → retención → tests. Distinguir explícitamente el esquema de origen del esquema propio de Peitho.
- [ ] **PEI-246** Proponer y obtener aprobación explícita de la allowlist de replicación a nivel tabla/columna. Excluir credenciales/RBAC/infraestructura de mensajería y PII/texto libre por defecto; probar la estrategia de filtrado/seudonimización elegida.
- [ ] **PEI-247** Implementar detección de schema drift entre el catálogo JSON, las migraciones fuente del SHA acordado y el esquema realmente aplicado en una base de prueba. Bloquear activación/deploy de una réplica incompatible y generar diff legible.
- [ ] **PEI-248** Validar publicación, replica identity, permisos, nombres de esquema/tablas, namespace de espejo y compatibilidad del PostgreSQL desplegado. Mantener las tablas replicadas aisladas de las tablas propias de Peitho, sin cambios DDL locales en el espejo.
- [ ] **PEI-249** Crear pruebas reproducibles contra una base temporal construida con el historial Flyway completo aprobado de Inventary, comprobar tablas/columnas/constraints/catalogo y probar bootstrap, cambios incrementales, conciliación, recuperación y denegación de escritura.

**Gate:** el commit documental revisado actualmente encuentra V1–V18 en la ruta de fuente, mientras el README raíz de Inventary todavía indica V1–V15. No activar producción hasta confirmar SHA desplegado, historial Flyway real, estructura efectiva y aprobación de la publicación. El catálogo está derivado del SQL versionado; no es una certificación de la base desplegada.

## Criterio objetivo de completitud del 100%

Peitho solo podrá declararse completo cuando:

- [ ] Todas las tareas PEI-001 a PEI-249 estén cerradas con evidencia, o exista una ADR de no aplicabilidad aprobada y verificable para cada excepción.
- [ ] Cada requisito funcional de docs/context.md, docs/flow.md, docs/functionality.md y de la especificación enterprise tenga vínculo a implementación, prueba y evidencia.
- [ ] Todas las migraciones puedan aplicarse a una base vacía de pruebas y actualizar una instancia soportada sin pérdida no aceptada.
- [ ] CI pase en el commit entregado, incluidos lint, tipos, build, unit/integration/contract/E2E y escáneres definidos como bloqueantes.
- [ ] La guía Docker funcione en una máquina limpia y las aplicaciones contenedorizadas conecten a PostgreSQL local con pgvector sin que Compose levante PostgreSQL.
- [ ] Se demuestre el aislamiento multiempresa y la imposibilidad de escribir en Inventary o enviar comunicaciones desde Peitho.
- [ ] Existan evidencias de recuperación de backups, replicación, índices vectoriales, tareas fallidas y rollback.
- [ ] No haya bugs bloqueantes conocidos, secretos en repositorio ni código muerto relevante; el catálogo de riesgos esté actualizado.
- [ ] Las pantallas principales sean accesibles, responsive y muestren procedencia/frescura/incertidumbre de los datos.
- [ ] Documentación, código, migraciones, contratos y ejecución estén sincronizados.

**Importante:** no se marca una tarea como completada porque su título aparezca en el código, porque compile aisladamente o porque el agente haya generado archivos. Se exige comportamiento probado y evidencia ligada a un commit/CI.
