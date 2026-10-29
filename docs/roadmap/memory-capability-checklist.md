# Checklist de capacidades de memoria y grafo operativo de Peitho

## Propósito y regla de interpretación

Esta taxonomía funciona como inventario de capacidades, vocabulario compartido y mapa de trazabilidad para agentes como Claude Code, Codex, Gemini u otros. No afirma que un sistema informático replique literalmente la cognición humana. Cada clasificación debe traducirse en una función de software verificable, con límites de datos, seguridad, evaluación y criterio de aplicabilidad.

En Peitho, memoria significa almacenamiento y recuperación gobernados de contexto de trabajo, hechos, eventos, conocimiento, procedimientos, resultados y evidencia. El procesamiento se activa por solicitudes, eventos y planificaciones; no significa que el LLM permanezca pensando continuamente.

Cada casilla se mantiene sin marcar hasta que la capacidad esté implementada y probada o exista ADR explícito de no aplicabilidad. La no aplicabilidad necesita causa y revisión; no debe usarse para ocultar trabajo pendiente.

## 1. Memoria sensorial

- [ ] **Memoria Icónica (visual)** — definir ingestión de imágenes autorizadas de productos, documentos, diagramas y recursos creativos; extraer metadatos, provenance, tenant, permisos y versión. Si se usa visión/OCR, medir precisión y conservar referencia al original.
- [ ] **Memoria Ecoica (auditiva)** — importar audio solo desde fuentes autorizadas; opcionalmente transcribir por servicio local, registrar idioma/calidad, redacción de PII, consentimiento y política de retención.
- [ ] **Memoria Háptica (táctil)** — definir contrato/adaptador solo si existe dispositivo o proceso real que produce datos táctiles útiles; no simular sensores inexistentes. Si no hay caso empresarial, documentar N/A mediante ADR.
- [ ] **Memoria Olfativa** — determinar si Peitho necesita telemetría de sensores de olor (por ejemplo, un proceso industrial concreto); sin sensor, datos autorizados y caso medible, documentar N/A mediante ADR.
- [ ] **Memoria Gustativa** — determinar si existe una fuente instrumental o flujo de cata/sensores que sea parte legítima del dominio; no inferir gustos personales como dato biométrico/comercial sin base, consentimiento y revisión. Formalizar N/A cuando no aplique.

**Nodo del grafo:** entrada multimodal autorizada → normalización → extracción de características/metadatos → almacenamiento de referencia → recuperación con permisos.

## 2. Memoria a corto plazo (MCP)

- [ ] **Memoria a Corto Plazo (MCP)** — mantener el contexto temporal de una solicitud o sesión: objetivo, resultado parcial, límites, fuentes consultadas y expiración. La sesión debe poder continuar sin convertir todo el prompt en memoria persistente.
- [ ] Definir TTL, límites de tokens/tamaño, aislamiento entre usuarios y tenants, cancelación, cierre de sesión y redacción de datos sensibles.
- [ ] Definir qué datos nunca se escriben en memoria temporal ni logs (contraseñas, secretos, claves, códigos MFA y contenido sensible innecesario).

**Nodo del grafo:** solicitud autenticada → contexto temporal → plan y herramientas autorizadas → respuesta verificable → expiración/limpieza.

## 3. Memoria de trabajo / operativa

- [ ] **Ejecutivo Central** — planificador/orquestador de tareas que selecciona objetivo, dependencias, herramientas permitidas, presupuesto y criterio de finalización; no puede modificar reglas de autorización.
- [ ] **Bucle Fonológico** — representar texto de la conversación, transcripciones y resúmenes de trabajo con procedencia, idioma, redacción de PII y límites de retención.
- [ ] **Agenda Visoespacial** — representar referencias visuales autorizadas, dimensiones, ubicación de almacenes, coordenadas, rutas y mapas con sistema de coordenadas y calidad explícitos.
- [ ] **Búfer Episódico** — unir de forma temporal los hechos relevantes a la tarea actual (usuario autorizado, entidades recuperadas, eventos recientes y resultados de herramientas) sin mezclar tenants ni promover inferencias a hechos.
- [ ] Implementar plan de tarea tipado, estado, checkpoint, cancelación, reanudación idempotente y límites por operación.
- [ ] Guardar la procedencia de cada dato del contexto: fuente, hora, versión, permisos, watermark y nivel de confianza.

**Nodo del grafo:** objetivo → plan → recuperación por permisos → herramientas → resultados tipados → verificación → respuesta/acción autorizada.

## 4. Memoria a largo plazo (MLP)

### Memoria explícita (declarativa)

- [ ] **Memoria Semántica** — catálogo y significado de productos, políticas de negocio, taxonomías, definiciones de métricas, documentos y reglas; versionada, indexada y recuperable por tenant.
- [ ] **Memoria Episódica** — historial de eventos de negocio, decisiones, propuestas, experimentos y ejecución externa importada; cada evento tiene origen y tiempo verificables.
- [ ] **Memoria Autobiográfica** — historial de las acciones y decisiones de un agente/proyecto solo para trazabilidad y continuidad autorizada; no es perfil psicológico de personas. Añadir retención, acceso, explicación y borrado.
- [ ] **Memoria Espacial** — almacenes, locales, áreas, rutas, distancias y restricciones logísticas con coordenadas, unidades, precisión y fecha de vigencia.
- [ ] Definir política de versionado, expiración, reindexación, rectificación y eliminación de documentos/embeddings relacionados.
- [ ] Implementar búsqueda híbrida/vectorial con PostgreSQL + pgvector y filtros de tenant, autorización, finalidad y vigencia antes de entregar contexto al LLM.

### Memoria implícita (no declarativa)

- [ ] **Memoria Procedimental** — playbooks, flujos aprobados, reglas determinísticas, runbooks y contratos de herramientas; control de versiones y pruebas, nunca instrucciones no verificadas recuperadas de un documento externo.
- [ ] **Priming (Imprimación)** — medir si contexto previamente autorizado mejora la recuperación de información pertinente; almacenar evaluación explícita, no usar asociaciones para manipulación comercial.
- [ ] **Condicionamiento Clásico** — si el producto analiza respuesta repetida a estímulos comerciales, tratarlo como análisis de comportamiento/campañas con aprobación ética, consentimiento y causalidad validada; no diseñar automatismos de manipulación psicológica.
- [ ] **Aprendizaje No Asociativo (Habituación / Sensibilización)** — implementar como analítica de fatiga, frecuencia, saturación y alertas repetidas con límites configurables; respetar opt-out y no hacer perfiles sensibles sin fundamento.

## 5. Otras clasificaciones

- [ ] **Memoria Prospectiva** — tareas futuras, compromisos, vencimientos y experimentos planificados con hora/zona, responsable, estado, idempotencia y cancelación.
- [ ] **Memoria Retrospectiva** — recuperación de hechos/eventos anteriores con referencias a la fuente, fecha, versión y filtros de acceso.
- [ ] **Memoria Emocional** — si se habilita análisis de sentimiento en texto autorizado, describirlo como inferencia probabilística de contenido, no como lectura de emociones reales; minimizar datos, probar sesgo y limitar usos.
- [ ] **Memoria Eidética (Fotográfica)** — no prometer memoria fotográfica perfecta. Implementar un archivo visual/documental con referencia al original, extracción versionada, calidad estimada y búsqueda explicable.
- [ ] **Memoria Colectiva / Transaccional** — memoria compartida de empresa basada en eventos, contratos, decisiones, experimentos y resultados confirmados; separar por tenant, versionar, auditar y corregir sin borrar la historia necesaria.
- [ ] Definir exportación, eliminación, conservación legal y reconstrucción de índices de toda memoria cuando una fuente se rectifica o se elimina.
- [ ] Probar que ningún recuerdo de una empresa aparece en consultas de otra, incluyendo caché, embeddings, resúmenes y contexto de agentes.

## 6. Grafo de conocimiento, dependencias y provenance

- [ ] Modelar nodos para requerimientos, decisiones ADR, dominios, entidades, eventos, APIs, jobs, modelos, tareas, pruebas y documentación.
- [ ] Modelar aristas tipadas: depends_on, produces, consumes, derived_from, validates, authorizes, supersedes y blocks.
- [ ] Cada nodo debe tener identificador estable, versión, estado, origen, propietario, criticidad y enlaces a código/tests/documentación.
- [ ] Cada arista debe indicar relación, motivo y evidencia; evitar enlaces ambiguos basados únicamente en nombres similares.
- [ ] Propagar cambios: cuando cambie un contrato, migración, modelo, regla o permiso, identificar módulos, pruebas, memoria e índices afectados.
- [ ] Crear comprobación de enlaces rotos y referencias no resueltas en CI.
- [ ] Crear consultas de grafo para: dependencias abiertas, tareas bloqueadas, cobertura de requisitos, impactos de cambios, modelos sin evaluación y endpoints sin pruebas.
- [ ] Evitar que la memoria del agente se convierta en autoridad transaccional: Inventary es fuente operativa y los hechos de Peitho tienen su propia procedencia.
- [ ] Separar hechos de origen, agregaciones calculadas, inferencias de IA, pronósticos y supuestos en almacenamiento y UI.
- [ ] Evaluar memoria con datasets sintéticos: precisión/recall, frescura, permisos, eliminación, recuperación tras restart y resistencia a prompt injection.

## 7. Criterios de cierre

- [ ] Cada clasificación está implementada con pruebas y evidencia o cerrada con ADR razonada de no aplicabilidad.
- [ ] Toda memoria persistente tiene finalidad, owner, retención, permisos, procedencia y mecanismo de rectificación/borrado.
- [ ] Toda recuperación semántica aplica permisos y tenant antes de que el resultado se entregue al modelo.
- [ ] El grafo identifica dependencias, tareas, bloqueos y cobertura sin depender de contexto oral del agente.
- [ ] El comportamiento del agente es acotado, reproducible y verificable; no se permite ejecución arbitraria ni autonomía fuera de los permisos aprobados.

## Relaciones principales del grafo

~~~mermaid
flowchart TD
  REQ[Requisitos y ADR] --> TASK[Tareas PEI trazables]
  TASK --> CODE[Código y migraciones]
  CODE --> TEST[Pruebas y CI]
  TEST --> EVID[Evidencia de aceptación]
  SOURCE[Fuentes autorizadas] --> ING[Ingesta y normalización]
  ING --> SEM[Memoria semántica]
  ING --> EPI[Memoria episódica]
  USER[Solicitud y permisos] --> MCP[Memoria a corto plazo]
  MCP --> WORK[Memoria de trabajo]
  SEM --> WORK
  EPI --> WORK
  WORK --> GATE[Reglas, herramientas y guardas]
  GATE --> RESULT[Propuesta/resultado verificable]
  RESULT --> EPI
  RESULT --> LEARN[Evaluación y aprendizaje]
  LEARN --> TASK
  TENANT[Identidad y tenant] --> MCP
  TENANT --> SEM
  TENANT --> EPI
~~~
