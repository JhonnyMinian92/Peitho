# ADR-0001: réplica lógica dedicada de Inventary

- **Estado:** Aceptado
- **Fecha:** 2026-10-29
- **Ámbito:** integración de datos Inventary → Peitho

## Contexto
Peitho debe analizar ventas, compras, recepciones, devoluciones y stock sin someter la base transaccional de Inventary a consultas analíticas intensivas. Inventary conserva la autoridad de sus operaciones.

## Decisión
1. La integración principal será una **réplica lógica dedicada de Inventary hacia PostgreSQL de Peitho**.
2. Se complementará con una **API interna autenticada de solo lectura** para validaciones puntuales como disponibilidad y precio vigente.
3. No se ejecutarán cargas analíticas intensivas en la base transaccional ni se usará una read-replica transaccional como almacén analítico principal. Peitho no escribe en Inventary.
4. Publicaciones/tablas/columnas se acordarán con los responsables de Inventary; no se presume que APIs, eventos o publicaciones ya existen.

## Salvaguardas obligatorias
- La replicación lógica no sincroniza automáticamente esquema ni DDL. Coordinar, versionar y probar migraciones de publicador/suscriptor.
- Registrar por conjunto de datos source_updated_at, received_at, applied_at, watermark/LSN, lag_seconds, schema_version, snapshot_id y estado/fecha de conciliación.
- Monitorizar posiciones de envío/aplicación, lag, errores de suscripción/apply, progreso de slots, crecimiento y retención de WAL, disco, salud y recuperación. Un consumidor detenido puede retener WAL.
- Conciliar registros, cantidades, importes y saldos por ventana/producto/almacén cuando los datos lo permitan.
- Definir umbrales de warning/block por ambiente antes de producción. Datos desconocidos, obsoletos, no conciliados o incompatibles no habilitan propuestas confirmadas dependientes de stock/margen.
- La API no expone SQL arbitrario; requiere identidad de servicio, permisos mínimos, límites, TLS y auditoría.
- Si la API no valida el estado actual, la propuesta queda no confirmada y muestra timestamp/fuente del snapshot.

## Consecuencias
Aumenta el aislamiento de carga, pero requiere operar lag, conciliación, WAL, cambios DDL y recuperación explícitamente.

## Criterios de aceptación
- Tests negativos prueban que Peitho no puede ejecutar INSERT, UPDATE, DELETE, TRUNCATE, DDL ni funciones de escritura sobre Inventary.
- Pruebas cubren atraso, ruptura de suscripción, esquema incompatible, conciliación fallida y verificación puntual por API.
- Un snapshot de stock nunca se muestra como actual sin timestamp/lag y estado de conciliación.


## Contrato versionado del esquema de origen

Peitho mantiene el catálogo de esquema en [docs/integration/inventary-database-schema.md](../../integration/inventary-database-schema.md) y su versión legible por máquina en [inventary-source-schema-catalog-v18.json](../../contracts/schemas/inventary-source-schema-catalog-v18.json).

El snapshot fuente revisado es el commit `083fc9f4ec97b8673de9f0e2adef6af195df65e5` del repositorio Inventary. La cadena SQL ubicada en `microservices-inventary-ecuapp/src/main/resources/db/migration/` contiene V1–V18 y define 49 tablas. El catálogo se deriva de esas migraciones, no de una introspección de la base productiva. El README de Inventary aún declara V1–V15; además, hay copias atrasadas bajo una ruta de salida de compilación. Antes de activar la réplica debe comprobarse el SHA de código desplegado y el historial Flyway aplicado en una instalación verificada.

La publicación real sigue **sin aprobar**. El catálogo marca tablas candidatas/excluidas y campos potencialmente sensibles; no concede permiso automático para replicarlos. Por defecto se excluyen credenciales, RBAC de Inventary, auditoría libre, Outbox, Inbox e idempotencia, y se minimizan PII/texto libre. Una necesidad de incorporar columnas personales exige finalidad, permisos, retención, cifrado y auditoría documentados.

La replicación lógica nativa preserva los nombres de esquema/tabla y no realiza transformaciones arbitrarias. Mantener las tablas espejo en un namespace compatible con el publicador y las tablas propias de Peitho en otro esquema, evitando colisiones de nombre. No añadir `tenant_id` ni campos derivados dentro del espejo: la asignación de la instancia fuente al tenant de Peitho se administra en la configuración de la conexión y las proyecciones propias.
