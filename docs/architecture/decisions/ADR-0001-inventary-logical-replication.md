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
