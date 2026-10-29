# Roadmap integral de Peitho

Este directorio contiene los artefactos para que personas y agentes implementen el producto completo de forma trazable. No representa la implementación ya realizada ni autoriza a recortar requisitos a un MVP.

- [Checklist integral PEI-001 a PEI-249](implementation-checklist.md): tareas desde la preparación inicial hasta la aceptación final.
- [Grafo JSON de dependencias](task-graph.json): fases, relaciones de prerequisito, invariantes y documentos de contexto para consumo automático.
- [Contrato para agentes](agent-execution-contract.md): fuentes obligatorias, ciclo de implementación, pruebas y reglas de cierre.
- [Checklist de memoria y grafo de conocimiento](memory-capability-checklist.md): taxonomía de memoria solicitada y traducción a capacidades de software.
- [Entorno Docker + PostgreSQL local](../development/local-docker-postgres.md): Docker para servicios de aplicación y PostgreSQL ejecutado en el host.
- [Catálogo del esquema fuente de Inventary](../integration/inventary-database-schema.md): 49 tablas derivadas de Flyway V1–V18 en el snapshot revisado; distingue campos candidatos, PII y exclusiones.
- [Catálogo JSON de Inventary](../contracts/schemas/inventary-source-schema-catalog-v18.json): referencia estructurada para agentes y validadores.

## Principios de mantenimiento

- Mantener el identificador PEI estable aunque cambie el texto descriptivo.
- Cuando una tarea crezca, desglosarla en subtareas con IDs nuevos; no borrar trazabilidad histórica.
- Mantener implementation-checklist.md y task-graph.json sincronizados en el mismo cambio.
- Cada tarea cerrada debe referenciar commit/PR, CI, pruebas y documentación aplicable.
- Si cambia una decisión aceptada o un invariante, actualizar ADR/contratos antes de cambiar código.
- Un agente nuevo debe poder reconstruir la intención y las dependencias leyendo los archivos enlazados sin depender de explicaciones en chat.
