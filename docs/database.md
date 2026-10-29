# Database

## Inventary: esquema fuente de solo lectura

La base de datos transaccional de Inventary es distinta de la base propia de Peitho. El inventario de tablas y campos fuente deriva de las migraciones Flyway de Inventary `V1–V18` en el snapshot `083fc9f4ec97b8673de9f0e2adef6af195df65e5` (49 tablas definidas por SQL). No se debe adivinar el esquema ni copiarlo como si fuera el modelo de Peitho.

- [Catálogo detallado de tablas, campos, relaciones, semántica y réplica](integration/inventary-database-schema.md)
- [Catálogo JSON para agentes/validadores](contracts/schemas/inventary-source-schema-catalog-v18.json)
- [ADR de réplica lógica](architecture/decisions/ADR-0001-inventary-logical-replication.md)
- [Configuración de ejemplo de lectura/replicación](contracts/config/inventary-readonly.example.json)

Peitho debe conservar las tablas fuente en un namespace de espejo aislado y crear su propio modelo canónico/analítico en un esquema distinto. La réplica no copia DDL automáticamente, no transforma valores ni debe replicar credenciales o PII sin aprobación explícita. La publicación real de tablas/columnas sigue pendiente de revisión; el catálogo no equivale a una publicación aprobada ni acredita que las migraciones estén aplicadas en producción.

Database: new (postgresql); the agent designs and creates it.

En el directorio raiz debe crear el directorio del frontend app-peitho-ecuapp y del backend que es microservices-peitho-ecuapp. El frontend no debe usar alert, siempre usar skelleton, modales, stepper, selectores modernos, input con validaciones, efectos, transiciones y experiencia de usuario facil e intuitiva. El backend debe tener estandar de json request y estadar de salida de json response. El backend debe tener cifrado de aplicacion, trasabilidad, escalamiento, auditoria, sonda, webhook, sse, machine learning, algoritmos de prediccion, algoritmos de clasificacion no supervisado, algoritmos de mejor ruta, algoritmos estadistico de probabilidad de abandono, entre otros. El backend debe ser el cerebro que constantemente piensa con los datos que siguen llegando de las ventas y compras. Ademas de conocer totalmente el estado actual del inventario. jamas hagas codigo spaguetti o monolitos con funciones gigantes de codigo fuente.

# Reused Database from Central Memory

## databases/optimistic_concurrency.md

# Optimistic Concurrency

## Intent
Detect conflicting updates with a version or revision check instead of holding long-lived locks.

## Applicability
Use when records can be edited concurrently, transactions should remain short, and conflicts are uncommon enough to resolve explicitly.

## Non-applicability
Avoid it when conflicts are frequent and expensive to resolve, or when correctness requires serialization that the data store can enforce more directly.

## Structure / responsibilities
- **Record:** stores a monotonic version/revision (or equivalent concurrency token).
- **Writer:** reads the token and includes it in the conditional update.
- **Database:** updates only when the expected version still matches.
- **Application:** translates a conflict into a retry, merge, or explicit user-facing resolution.

## Implementation guidance
1. Read the record and its current version.
2. Apply the intended change in memory.
3. Update with a condition such as `version = expected_version`.
4. Increment the version atomically on success.
5. Treat zero affected rows as a concurrency conflict, not as success.
6. Retry only when the operation is safe and the retry policy is bounded.
7. Record conflict metrics when contention matters operationally.

## Validation criteria
- Concurrent writers cannot silently overwrite one another.
- Version increments are atomic and monotonic.
- Conflict handling is covered by tests using concurrent or simulated writers.
- Retries cannot duplicate side effects.

## Security considerations
Do not expose internal revision values as authorization evidence. Authorization must be checked independently, and conflict responses must not reveal sensitive record state to an unauthorized caller.

## Trade-offs
This approach avoids long locks and often improves throughput, but conflicts require explicit resolution and can complicate write flows.

## Related skills and patterns
- Skill: `database.concurrency`
- Pattern: `database.transaction-boundary`
- Pattern: `error.retry-with-backoff`

## Example decision signals
Prefer this pattern when edits are short-lived and conflicts are possible but relatively infrequent.

## databases/transaction_boundary.md

# Explicit Transaction Boundary

## Intent
Define the smallest coherent unit of database work that must succeed or fail together, making consistency guarantees explicit.

## Applicability
Use when multiple writes represent one business operation, invariants span records, or partial persistence would leave invalid state.

## Non-applicability
Do not use a large transaction merely to simplify orchestration. Avoid holding transactions across slow network calls, user interaction, or unrelated work.

## Structure / responsibilities
- **Application/service boundary:** starts and ends the transaction around one business operation.
- **Domain/application logic:** defines the invariants that must hold when the transaction commits.
- **Repositories/data access:** participate in the caller's transaction instead of silently creating independent transactions.
- **Database:** provides atomic commit/rollback and isolation appropriate to the invariant.

## Implementation guidance
1. Identify the invariant and all writes required to preserve it.
2. Start the transaction at the application boundary that owns the use case.
3. Validate inputs and authorization before expensive transactional work where safe.
4. Perform the minimum required reads and writes.
5. Commit only after all invariants are satisfied.
6. Roll back on expected failure and propagate a structured error.
7. Keep external calls outside the transaction; use an outbox or equivalent when atomic publication is required.

## Validation criteria
- Partial execution cannot leave the protected invariant broken.
- Rollback behavior is tested for every relevant failure path.
- Isolation level is documented when it affects correctness.
- Transaction duration is bounded and observable.

## Security considerations
Enforce authorization before mutation, avoid exposing transaction errors with sensitive database details, and use parameterized queries. Do not rely on transaction atomicity as a substitute for access control or validation.

## Trade-offs
Transactions provide strong consistency but can reduce throughput and increase lock contention when they are too broad or long-lived.

## Related skills and patterns
- Skill: `database.transactions`
- Pattern: `database.optimistic-concurrency`
- Pattern: `integration.outbox`

## Example decision signals
Prefer this pattern when a single business action changes several records and observers must never see an invalid intermediate state.

Obsoleto: usar content en su lugar.


# Peitho-specific data integration and vector invariants

- PostgreSQL in Peitho owns Peitho's CRM, canonical events, proposals, audit records, source-freshness state and versioned embedding metadata. It is not the source of truth for Inventary transactions.
- Use dedicated logical replication from Inventary to Peitho for bulk operational data. Logical replication does not automatically propagate DDL; versioned schema changes must be coordinated and compatibility checked.
- Track source update time, receive/apply time, watermark/LSN, lag, schema version, snapshot ID and reconciliation status for each dataset. Stale or unreconciled datasets are excluded from stock-sensitive decisions.
- Keep an authenticated, scoped, read-only Inventary API for point-in-time availability/current-price checks. Never expose arbitrary SQL to the AI gateway and never grant Peitho write privileges on Inventary.
- pgvector stores embeddings, not business truth. Track embedding model ID/revision, dimensions, checksum and generation time. A model/dimension change requires versioned re-embedding and validated index cutover.
- Keep secrets outside JSON examples and source control. Example files refer to environment variable names, never credentials.

See [ADR-0001](architecture/decisions/ADR-0001-inventary-logical-replication.md), [Inventary config example](contracts/config/inventary-readonly.example.json) and [source-freshness schema](contracts/schemas/source-freshness.schema.json).
