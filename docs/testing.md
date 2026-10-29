# Test Strategy

- Define test requirements during the internal analysis.

# Reused Testing from Central Memory

## testing/test_pyramid.md

# Test Pyramid

## Intent
Balance fast feedback and broad confidence by using many focused tests, fewer integration tests and a smaller set of end-to-end tests.

## Applicability
Use as a planning heuristic for automated verification of software where different test levels provide different cost, speed and isolation characteristics.

## Non-applicability
Do not treat the pyramid as a fixed percentage rule. Systems dominated by integration boundaries, distributed behavior or user journeys may require a different distribution based on risk.

## Structure / responsibilities
- **Unit tests:** verify small units in isolation.
- **Integration tests:** verify real interactions between components or infrastructure boundaries.
- **End-to-end tests:** verify critical user-visible workflows across the system.
- **Test strategy:** maps risk and requirements to the appropriate level.
- **Quality gate:** prevents release when required critical behavior is unverified.

## Implementation guidance
1. Identify critical business and security risks.
2. Cover deterministic business logic with fast unit tests.
3. Use integration tests for persistence, APIs, queues and important boundaries.
4. Reserve end-to-end tests for critical workflows that require system-level confidence.
5. Add regression tests for defects at the lowest level that reproduces the failure reliably.
6. Keep tests deterministic and independent.
7. Measure useful coverage and mutation/risk signals rather than chasing a percentage alone.

## Validation criteria
- Critical requirements map to explicit tests.
- Unit tests remain fast and isolated.
- Integration tests exercise important real boundaries.
- Critical user journeys have appropriate end-to-end coverage.
- Defects gain durable regression tests.

## Security considerations
Security-sensitive behavior must be tested at the boundary where controls actually operate. Include authentication, authorization, validation, injection resistance, sensitive-data handling and security regression tests according to risk.

## Trade-offs
More unit tests usually improve speed and diagnosis, while integration and end-to-end tests provide stronger boundary confidence at higher cost and fragility. The optimal distribution follows system risk rather than a universal ratio.

## Related skills and patterns
- `testing.test_strategy`
- `testing.unit_testing`
- `testing.integration_testing`
- `testing.e2e_testing`
- `testing.quality_gate`
- `testing.contract_testing`

## Example decision signals
- **Pure deterministic logic:** prioritize unit tests.
- **Database transaction behavior:** add integration coverage.
- **Critical login journey:** add end-to-end coverage.
- **API provider/consumer boundary:** add contract tests.

Obsoleto: usar content en su lugar.

## Tests required by the internal analysis

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

