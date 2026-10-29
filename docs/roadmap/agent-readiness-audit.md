# Auditoría de preparación para implementación autónoma de Peitho

**Dictamen al commit base `3f985074aa5c023fc626adc533ae5e6ed15e9198`: NO-GO para iniciar ahora la generación autónoma del código.** La documentación funcional y el roadmap han mejorado, pero el propio protocolo del proyecto impide comenzar mientras la preparación de Mnemosine siga incompleta. No se debe anular esa regla editando a mano sus archivos derivados.

Esta auditoría no significa que Peitho deba reducirse a un MVP. El objetivo sigue siendo el producto empresarial completo; el dictamen solo indica que todavía no se ha completado el procedimiento obligatorio de activación ni se ha materializado el backlog en la fuente operativa de tareas.

## 1. Cobertura y verificaciones

- [x] Se recorrió el árbol de `master`: 69 archivos en el snapshot auditado, con 46 Markdown, 22 JSON y un workflow YAML.
- [x] Se validó sintácticamente el contenido de los 22 archivos JSON.
- [x] Se comprobó que el roadmap define 249 IDs de tarea PEI únicos y consecutivos, PEI-001 a PEI-249.
- [x] Se comprobó que `task-graph.json` referencia esas 249 tareas una sola vez, con 19 nodos y 41 aristas cuyos extremos existen.
- [x] Se cotejaron los 49 nombres de tabla del catálogo JSON de Inventary con los 49 encabezados del catálogo Markdown.
- [x] Se localizaron y corrigieron enlaces relativos rotos en `docs/architecture.md` y referencias a dos artefactos visuales ausentes en `docs/context.md`.
- [x] Se separó la estimación antigua de 41 tablas del modelo propio de Peitho del catálogo de 49 tablas de Inventary. Son contratos distintos.
- [ ] No se ejecutaron build, tests de aplicación, migraciones, Docker ni pruebas E2E: el repositorio auditado aún no contiene el código de las aplicaciones.
- [ ] No se pudo comprobar una CI verde para el SHA revisado; la consulta de estados de ese SHA no devolvió verificaciones registradas en esta revisión.

## 2. Bloqueadores del arranque

### GATE-01 — La preparación obligatoria de Mnemosine sigue incompleta

`AGENTS.md` indica que no se debe comenzar implementación mientras el proyecto está en preparación y establece como prerrequisitos: materialización de la memoria central seleccionada, análisis interno completo, respuestas requeridas completas y ausencia de preguntas pendientes.

El snapshot observado contradice la idea de que el proyecto ya está activado:

- `docs/memory/state.json`: `status = pending`, `maturity = INITIALIZED`, progreso 0, cero tareas, `active_task_uuid = null` y `next_ready_task_uuid = null`.
- `docs/memory/context.json`: `loaded_context = []`.
- `docs/memory/gemini_final_analysis.json`: objeto vacío `{}`.
- `docs/analysis/gemini_final_plan.md`: sigue indicando que el análisis interno final está pendiente.
- `docs/hierarchy.md`: sigue indicando que el análisis de la jerarquía está pendiente.
- `docs/task.md`: el snapshot operativo declara que todavía no hay tareas.

Esos archivos constituyen evidencia de estado, no campos que el agente de código deba editar para simular preparación. La memoria operativa y el estado real de tareas pertenecen a Mnemosine; según `AGENTS.md`, `docs/` es una copia de recuperación y no reemplaza la API de Mnemosine.

**Acción requerida:** completar el flujo real de preparación en Mnemosine, materializar las capacidades/memorias seleccionadas y obtener un briefing final que confirme estado ACTIVE, análisis final completo, preguntas resueltas y al menos una tarea lista. No fabricar ni modificar manualmente `state.json`, `context.json`, `gemini_final_analysis.json` o `task.md` para saltarse el gate.

### GATE-02 — Las 249 tareas del roadmap no son todavía tareas operativas

El checklist y el grafo definen el destino de implementación, pero el estado de Mnemosine indica cero tareas. El agente no debe interpretar que los checkboxes Markdown sustituyen el backlog operativo.

**Acción requerida:** materializar PEI-001…PEI-249 en Mnemosine como tareas reales, respetando dependencias y desglosando tareas demasiado grandes. Cada tarea operativa debe tener:

- ID estable vinculado al PEI padre.
- Dependencias explícitas y definición de lista para iniciar.
- Criterios de aceptación específicos, incluyendo IDs para criterios automatizados (por ejemplo `PEI-011-C1`) y pruebas cuyo nombre incluya esos IDs.
- Criterios manuales e instrucciones para validarlos.
- Archivos/módulos esperados, riesgos, permisos requeridos y contratos afectados.
- Evidencia de cierre: commit/PR, comandos, resultado de tests, CI y documentación.

Si el agente no dispone de permisos para crear o reclamar tareas en Mnemosine, ese es un bloqueo que debe reportar, no una razón para crear código fuera del flujo gobernado.

### GATE-03 — El repositorio no contiene aún el producto ejecutable

En el snapshot auditado no existen `app-peitho-ecuapp/`, `microservices-peitho-ecuapp/`, Dockerfiles de Peitho, `compose.yaml`, migraciones de Peitho, módulos de aplicación ni tests de producto. El workflow presente corresponde a Mnemosine CI y no equivale a una validación de una aplicación que todavía no está creada.

Esto es trabajo previsto en PEI-011…PEI-029, no un error en sí mismo. Significa que no se puede afirmar que Docker arranca, que PostgreSQL local se conecta o que la API está lista.

### GATE-04 — Los detalles que requieren aprobación o comprobación real deben seguir siendo gates

- El esquema fuente documentado de Inventary se deriva del SQL Flyway V1–V18 en el commit `083fc9f4ec97b8673de9f0e2adef6af195df65e5`. No se inspeccionó directamente una base desplegada. El README de Inventary aún declara V1–V15. Antes de activar replicación se debe verificar el SHA desplegado, el historial de migraciones aplicado y la estructura real.
- La lista de tablas/columnas autorizadas para replicación sigue sin aprobarse. Los campos PII, texto libre, credenciales y tablas de infraestructura deben excluirse por defecto.
- El número preliminar de 41 tablas de Peitho no es un esquema aprobado. El modelo propio debe diseñarse como tareas de PEI-076…PEI-079; no debe inferirse desde la base de Inventary.
- Las versiones exactas de PostgreSQL/pgvector, runtime operativo, estrategia de cola y almacenamiento, capacidad/recursos de IA y modelos concretos deben resolverse mediante tareas y ADR/benchmarks. No se deben inventar decisiones en el código para cerrar artificialmente el checklist.

Estos gates no impiden trabajar en tareas independientes una vez superada la activación general, pero sí impiden declarar finalizados esos módulos concretos antes de disponer de la evidencia requerida.

## 3. Defectos documentales corregidos en esta revisión

- [x] Se corrigieron tres enlaces relativos a ADR en `docs/architecture.md`.
- [x] Se eliminaron como enlaces de navegación las referencias a `DEMO_PEITHO_CONCEPTO.html` y `DEMO_PEITHO_VISTA_PREVIA.png`, que no existen en el snapshot actual; quedaron señalados como artefactos no incluidos.
- [x] Se marcó `docs/analysis/gemini_initial_plan.md` como snapshot histórico, para que no suplante el roadmap vigente ni la compuerta de preparación.
- [x] Se aclaró en `docs/analysis/constraints.md` que las 41 tablas eran una estimación temprana de Peitho, separada de las 49 tablas del esquema fuente de Inventary.
- [x] Se consolidó un contrato de ejecución que respeta el gate de Mnemosine y contempla continuidad entre tareas independientes.
- [x] Se añadió validación automatizada para prevenir que vuelvan a divergir JSON, PEI, grafo, catálogo y enlaces Markdown.

## 4. Procedimiento exigido para trabajar “sin parar”

“Sin parar” significa mantener un ciclo de ejecución continuo después de que los gates de activación estén satisfechos. No significa saltarse dependencias, ignorar fallos, fingir que una tarea está lista o esperar indefinidamente sin informar.

1. Verificar el briefing vigente de Mnemosine, estado ACTIVE, memoria materializada, análisis final completo, preguntas resueltas y tarea lista.
2. Reclamar una tarea con dependencias satisfechas y leer contratos, ADR, código y pruebas relacionados.
3. Implementar la capacidad funcional completa del alcance asignado; no sustituirla por un mock, TODO o pantalla vacía.
4. Ejecutar pruebas según criterios y riesgos, arreglar fallos y actualizar documentación/grafo cuando corresponda.
5. Registrar commit/PR, evidencias, resultados de tests y estado final de tarea en el sistema de origen.
6. Continuar con la siguiente tarea lista hasta que todo el alcance esté implementado y probado.
7. Si una tarea está bloqueada, informar la causa y evidencia, marcarla bloqueada y continuar con otra tarea independiente lista. Si no queda ninguna tarea lista, resolver la dependencia o declarar el bloqueo con precisión; no inventar progreso ni dejar un proceso de espera sin límite.
8. Si no hay tareas operativas pero el checklist aún contiene pendientes, materializar las tareas faltantes en Mnemosine antes de seguir. No declarar 100 % porque no existan tareas en la cola.
9. Detener únicamente acciones que requieran aprobación humana, acceso/credenciales no disponibles, una migración destructiva no aprobada, un riesgo de seguridad crítico o una contradicción arquitectónica que exija decisión. Continuar el resto del trabajo seguro e independiente cuando sea posible.

## 5. Criterios para cambiar el dictamen a GO

- [ ] Mnemosine confirma la activación y los prerrequisitos de `AGENTS.md`.
- [ ] El briefing aporta capacidades/memorias requeridas y la tarea lista siguiente.
- [ ] El backlog PEI está materializado, con dependencias y criterios de aceptación verificables.
- [ ] El agente entiende cuáles documentos son SSOT, cuáles son snapshots derivados y qué no puede modificarse manualmente.
- [ ] El agente puede ejecutar y registrar pruebas en CI y no utiliza la documentación como sustituto de comportamiento real.

Una vez satisfechos los cinco puntos, puede iniciarse la implementación autónoma en el orden del grafo. El 100 % solo se declara cuando pasan los criterios de completitud de `docs/roadmap/implementation-checklist.md`, incluyendo las subtareas creadas durante el desarrollo.
