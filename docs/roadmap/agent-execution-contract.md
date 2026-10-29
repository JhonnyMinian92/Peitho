# Contrato de ejecución para agentes de código

## Objetivo no negociable

Entregar el producto Peitho completo definido por la especificación, no un MVP, prototipo ni demo. La implementación puede dividirse en tareas pequeñas y releases internos, pero no reducir el alcance final sin decisión explícita y aprobada. Ningún agente debe concluir que “ya está” porque compila, porque existe una pantalla mock o porque terminó una primera fase.

## Fuentes que debe leer el agente

1. AGENTS.md: reglas de gobierno y ejecución, seguridad y conexión con Mnemosine.
2. README.md: propósito, límites e índice de documentación.
3. docs/context.md, docs/flow.md y docs/functionality.md: especificación funcional completa y flujos.
4. docs/architecture.md y docs/architecture/decisions/: arquitectura y decisiones aceptadas.
5. docs/analysis/constraints.md y docs/analysis/production_profile.md: restricciones y gates de producción.
6. docs/contracts/: esquemas JSON, sobres request/response y configuraciones de referencia.
7. docs/roadmap/implementation-checklist.md: tareas PEI y condiciones de completitud.
8. docs/roadmap/task-graph.json: dependencias entre fases y componentes.
9. docs/development/local-docker-postgres.md: ejecución Docker con PostgreSQL local del host.
10. docs/roadmap/memory-capability-checklist.md: taxonomía y capacidades de memoria/grafo.

Si documentos entran en contradicción, respetar primero las decisiones aceptadas y los invariantes del producto; documentar contradicciones y proponer ADR, no cambiar el comportamiento en silencio.

## Ciclo obligatorio por tarea

- [ ] Obtener briefing y tarea lista desde el sistema de tareas indicado en AGENTS.md, cuando esté disponible.
- [ ] Leer requisito, dependencias, contratos, ADR, código real y tests relacionados.
- [ ] Confirmar que las dependencias están completas; si no, marcar bloqueada con causa y evidencia, no simular finalización.
- [ ] Implementar la solución completa en el módulo correcto, respetando separación de capas, multi-tenant, configuración y errores.
- [ ] Añadir/actualizar migraciones y contratos si corresponde.
- [ ] Añadir tests unitarios, integración, contrato, seguridad y E2E según el riesgo y los criterios de aceptación.
- [ ] Ejecutar lint, typecheck, build, test y verificaciones de seguridad exigidas por el módulo.
- [ ] Probar comportamiento real; no sustituir funcionalidad de servidor por mocks fuera de los tests.
- [ ] Actualizar documentación, tareas, grafo de dependencias y reportes, conservando el estado de las fuentes de verdad.
- [ ] Registrar archivos modificados, commit/PR, tests/comandos y resultados, riesgos abiertos y pasos manuales pendientes.
- [ ] Marcar la tarea como completa solo cuando todas sus condiciones y evidencia estén satisfechas.

## Reglas de implementación

- PostgreSQL local del host es la base de datos para desarrollo local. Compose no debe iniciar otro PostgreSQL; los contenedores utilizan host.docker.internal y un usuario con privilegio mínimo.
- Inventary es de solo lectura para Peitho. No permitir escrituras ni SQL arbitrario desde backend, agente, LLM o MCP.
- Por defecto, la inferencia no sale hacia servicios pagos externos. No usar proveedor externo de LLM para “desbloquear” una tarea.
- No enviar ni publicar campañas, correos, SMS, llamadas, mensajes o contenido social desde Peitho. Solo preparar borradores y registrar/importar resultados de ejecución externa.
- La autorización se comprueba en backend y dentro de consultas, workers, eventos, cachés, vector search, herramientas MCP, exportaciones y webhooks.
- Diferenciar datos fuente, datos derivados, inferencias, pronósticos y supuestos. Un dato stale o sin reconciliación no puede habilitar una propuesta sensible.
- No guardar secretos, prompts sensibles, códigos MFA ni PII innecesaria en logs, memoria del agente, ejemplos ni commits.
- No introducir código duplicado, funciones enormes, módulos globales con dependencias implícitas o abstracciones sin propósito.
- No cambiar un checkbox a completado por existencia de un archivo: exige implementación probada y trazabilidad a commit/CI.
- Una exclusión requiere ADR aprobada. No se acepta “fuera de MVP” como explicación.
- No modificar a mano archivos generados/derivados de Mnemosine como si fueran fuente de verdad; seguir el procedimiento del proyecto para actualizar la memoria sincronizada.

## Definición de tarea terminada

La tarea está terminada únicamente cuando cumple sus criterios funcionales, pruebas automáticas aprobadas, seguridad/permisos revisados, errores/observabilidad resueltos, documentación actualizada y CI verde sobre el commit entregado. Los criterios manuales se documentan con instrucciones claras y no se presentan como testeados automáticamente.

## Definición del producto al 100%

Usar la sección “Criterio objetivo de completitud del 100%” en el checklist integral. El porcentaje no se calcula contando archivos ni líneas de código: se calcula mediante cobertura requisito-tarea-prueba-evidencia, con bloqueo total cuando queda un requisito obligatorio no resuelto.
