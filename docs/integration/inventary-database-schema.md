# Catálogo de esquema de Inventary consumido por Peitho

**Tipo:** contrato de integración y catálogo fuente, versionado con la documentación de Peitho.  
**Snapshot fuente revisado:** `JhonnyMinian92/inventary`, rama `master`, commit `083fc9f4ec97b8673de9f0e2adef6af195df65e5`.  
**Ruta autoritativa del esquema:** `microservices-inventary-ecuapp/src/main/resources/db/migration/`.  
**Cadena de migraciones fuente encontrada:** Flyway V1–V18.  
**Tablas declaradas por la cadena SQL:** 49.  
**Estado:** catálogo documental derivado de las migraciones versionadas; no es una extracción de una base de datos productiva y no garantiza que toda instalación desplegada ya haya aplicado V18.

## 1. Autoridad y límites de versión

La fuente de verdad para reconstruir el esquema esperado es la secuencia SQL de Flyway bajo `src/main/resources/db/migration`. En `application.yaml`, Flyway está habilitado, valida las migraciones y Hibernate usa `ddl-auto: validate`: Hibernate valida las entidades, no debe crear ni actualizar automáticamente el esquema.

**Discrepancia detectada en el snapshot consultado:** el README raíz de Inventary todavía indica Flyway V1–V15, pero la ruta de migraciones de fuente contiene V1–V18 (incluye unidades, margen/precio derivado y suscripciones/facturación de servicios). También hay copias de migraciones en una ruta literal de salida de compilación bajo `${project.build.directory}` que están atrasadas. Peitho debe seguir el directorio fuente de Flyway y verificar el artefacto construido por CI, no esas copias ni el README desactualizado.

Antes de establecer la publicación de replicación definitiva, comprobar contra el SHA desplegado de Inventary: versión de Flyway aplicada, migraciones instaladas, estructura física, permisos, índices, triggers y configuración de replicación. No se dispone aquí de acceso a la base de datos desplegada para ejecutar `pg_catalog`/ `information_schema`; la referencia de esta página es el SQL versionado del commit indicado.

## 2. Catálogo exacto de tablas y campos

En las listas, `PK` significa clave primaria, `FK` clave foránea, `NN` NOT NULL y `?` nullable. Los valores por defecto y restricciones relevantes aparecen después del punto y coma. Esta sección enumera los campos declarados al aplicar secuencialmente V1–V18; las restricciones nombradas pueden haber sido añadidas o sustituidas por una migración posterior.

### 2.1 Seguridad, roles y operación técnica

- **`app_user`** — `id UUID PK`; `username VARCHAR(80) NN`; `password_hash VARCHAR(100) NN`; `role VARCHAR(40) NN FK → app_role.code`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`; `token_version INTEGER NN DEFAULT 0`. V1 declara unicidad de username y V7 añade unicidad case-insensitive. V10 sustituye el conjunto fijo de roles; V14 enlaza el campo con el catálogo de roles. **No replicar** este catálogo de autenticación a Peitho.
- **`app_role`** — `id UUID PK`; `code VARCHAR(40) NN UNIQUE`; `display_name VARCHAR(100) NN`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. No replicar como fuente de permisos de Peitho.
- **`app_permission`** — `id UUID PK`; `code VARCHAR(64) NN UNIQUE`; `description VARCHAR(240) NN`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. No confundir estos permisos con los permisos locales de Peitho.
- **`app_role_permission`** — `role_id UUID NN FK → app_role.id`; `permission_id UUID NN FK → app_permission.id`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; PK compuesta `(role_id, permission_id)`.
- **`security_lock`** — `lock_key VARCHAR(40) PK`; `updated_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Bloqueo interno para proteger la existencia de un SUPER_ADMIN.
- **`audit_event`** — `id UUID PK`; `actor VARCHAR(80) NN`; `action VARCHAR(120) NN`; `resource_type VARCHAR(80) NN`; `resource_id UUID ?`; `outcome VARCHAR(20) NN`; `correlation_id VARCHAR(80) ?`; `metadata JSONB NN DEFAULT '{}'`; `occurred_at TIMESTAMPTZ NN`. `outcome`: SUCCESS, DENIED, FAILURE. No replicar metadatos libres por defecto.
- **`outbox_event`** — `id UUID PK`; `aggregate_type VARCHAR(80) NN`; `aggregate_id UUID NN`; `event_type VARCHAR(120) NN`; `payload JSONB NN`; `status VARCHAR(20) NN DEFAULT 'PENDING'`; `attempts INTEGER NN DEFAULT 0`; `next_attempt_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN`; `published_at TIMESTAMPTZ ?`; `locked_until TIMESTAMPTZ ?`; `last_error VARCHAR(500) ?`. Infraestructura de publicación; excluir de la réplica analítica.
- **`kafka_inbox_event`** — `event_id UUID PK`; `event_type VARCHAR(120) NN`; `aggregate_type VARCHAR(80) NN`; `aggregate_id UUID NN`; `payload_sha256 CHAR(64) NN`; `received_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Inbox de deduplicación; excluir.
- **`idempotency_record`** — `id UUID PK`; `scope VARCHAR(120) NN`; `idempotency_key VARCHAR(120) NN`; `request_hash VARCHAR(64) NN`; `status VARCHAR(16) NN`; `resource_type VARCHAR(80) ?`; `resource_id UUID ?`; `response_status SMALLINT ?`; `response_body JSONB ?`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `completed_at TIMESTAMPTZ ?`. Infraestructura transaccional; excluir.
  
### 2.2 Catálogo de productos, atributos, unidades y recursos

- **`category`** — `id UUID PK`; `name VARCHAR(120) NN`; `slug VARCHAR(140) NN`; `description VARCHAR(1000) ?`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`. Nombre y slug únicos; V7 refuerza nombre sin distinción de mayúsculas/minúsculas.
- **`category_attribute_definition`** — `id UUID PK`; `category_id UUID NN FK → category.id`; `attribute_key VARCHAR(80) NN`; `display_name VARCHAR(120) NN`; `value_type VARCHAR(24) NN`; `required BOOLEAN NN DEFAULT FALSE`; `constraints JSONB NN DEFAULT '{}'`; `options JSONB NN DEFAULT '[]'`; `schema_version INTEGER NN DEFAULT 1`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `updated_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Tipos: STRING, NUMBER, BOOLEAN, DATE, ENUM, STRING_LIST. Único por categoría, clave y versión.
- **`product`** — `id UUID PK`; `sku VARCHAR(64) NN`; `name VARCHAR(180) NN`; `description VARCHAR(2000) ?`; `category_id UUID NN FK → category.id`; `cost_price NUMERIC(14,2) NN DEFAULT 0`; `sale_price NUMERIC(14,2) NN DEFAULT 0`; `currency VARCHAR(3) NN DEFAULT 'USD'`; `image_url VARCHAR(2048) ?`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`; `version BIGINT NN DEFAULT 0`; `product_kind VARCHAR(24) NN DEFAULT 'STOCKED_GOOD'`; `unit_of_measure_id UUID NN FK → unit_of_measure.id`; `attributes JSONB NN DEFAULT '{}'`; `attribute_schema_version INTEGER NN DEFAULT 1`; `margin_percent NUMERIC(5,2) NN DEFAULT 25`; `service_billing VARCHAR(16) ?`; `billing_frequency VARCHAR(16) ?`; `term_months INTEGER ?`. `product_kind`: STOCKED_GOOD, NON_STOCKED_GOOD, SERVICE. `service_billing`: ONE_TIME, SUBSCRIPTION; `billing_frequency`: BIWEEKLY, MONTHLY, AT_END; `term_months`: 1, 3, 6 o 12. SKU es único también sin distinguir mayúsculas; precios/coste no negativos; margen de 0 a 1000.
- **`product_variant`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `sku VARCHAR(64) NN`; `barcode VARCHAR(128) ?`; `option_values JSONB NN DEFAULT '{}'`; `unit_of_measure_id UUID NN FK → unit_of_measure.id`; `sale_price NUMERIC(14,2) ?`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `updated_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. SKU/barcode únicos; existe índice único sobre SKU normalizado y combinaciones de opciones activas. La migración no define saldo/coste separado por variante.
- **`product_image`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `image_url VARCHAR(2048) NN`; `alt_text VARCHAR(300) ?`; `sort_order INTEGER NN DEFAULT 0`; `is_primary BOOLEAN NN DEFAULT FALSE`; `published BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`. Único por producto+URL y solo puede haber una imagen principal publicada por producto.
- **`unit_of_measure`** — `id UUID PK`; `code VARCHAR(24) NN`; `name VARCHAR(100) NN`; `dimension VARCHAR(32) NN`; `decimal_scale SMALLINT NN DEFAULT 3`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Código único; V14/V16 refuerzan normalización case-insensitive y cargan unidades comunes.
- **`unit_conversion`** — `id UUID PK`; `from_unit_id UUID NN FK → unit_of_measure.id`; `to_unit_id UUID NN FK → unit_of_measure.id`; `factor NUMERIC(20,9) NN`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Factor mayor que cero; unidad de origen distinta de destino; par único.
- **`business_partner`** — `id UUID PK`; `code VARCHAR(40) NN`; `partner_type VARCHAR(16) NN`; `name VARCHAR(180) NN`; `tax_id VARCHAR(32) ?`; `email VARCHAR(254) ?`; `phone VARCHAR(40) ?`; `address VARCHAR(500) ?`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`. Tipo CUSTOMER, SUPPLIER o BOTH; código e identificación tributaria únicos, código también case-insensitive. **tax_id, email, phone, address y nombres personales requieren decisión de finalidad, minimización y acceso antes de replicarse.**
- **`warehouse`** — `id UUID PK`; `code VARCHAR(40) NN`; `name VARCHAR(140) NN`; `address VARCHAR(500) ?`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`. Código único y case-insensitive.
- **`warehouse_location`** — `id UUID PK`; `warehouse_id UUID NN FK → warehouse.id`; `code VARCHAR(40) NN`; `name VARCHAR(120) NN`; `location_type VARCHAR(24) NN`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `updated_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Tipos STORAGE, TRANSIT, QUARANTINE, SCRAP, PICKING; código único dentro del almacén.

### 2.3 Saldos, movimientos, reservas, conteos y coste

- **`stock_balance`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `warehouse_id UUID NN FK → warehouse.id`; `on_hand NUMERIC(14,3) NN DEFAULT 0`; `reserved NUMERIC(14,3) NN DEFAULT 0`; `version BIGINT NN DEFAULT 0`; `updated_at TIMESTAMPTZ NN`; `location_id UUID NN FK → warehouse_location.id`. `on_hand >= 0`; `0 <= reserved <= on_hand`. El esquema conserva unicidad por producto+almacén y también por producto+ubicación durante la transición.
- **`stock_movement`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `warehouse_id UUID NN FK → warehouse.id`; `movement_type VARCHAR(32) NN`; `quantity NUMERIC(14,3) NN`; `unit_cost NUMERIC(14,2) NN DEFAULT 0`; `reference_type VARCHAR(40) ?`; `reference_id UUID ?`; `idempotency_key VARCHAR(120) ?`; `request_hash VARCHAR(64) ?`; `notes VARCHAR(1000) ?`; `created_by VARCHAR(80) NN`; `occurred_at TIMESTAMPTZ NN`; `from_location_id UUID ? FK → warehouse_location.id`; `to_location_id UUID ? FK → warehouse_location.id`; `unit_of_measure_id UUID ? FK → unit_of_measure.id`; `source_line_id UUID ?`; `correlation_id VARCHAR(80) ?`; `valuation_method VARCHAR(24) ?`. Tipos actuales incluyen ADJUSTMENT_IN, ADJUSTMENT_OUT, SALE, SALE_CANCEL_IN, RECEIPT, RETURN_IN, RETURN_OUT, TRANSFER_IN, TRANSFER_OUT, RESERVE, RELEASE. Cantidad positiva, coste no negativo. Ledger inmutable: correcciones mediante movimientos compensatorios, nunca UPDATE/DELETE.
- **`stock_reservation`** — `id UUID PK`; `reservation_number VARCHAR(48) NN UNIQUE`; `sale_item_id UUID ? FK → sale_item.id`; `product_id UUID NN FK → product.id`; `warehouse_location_id UUID NN FK → warehouse_location.id`; `quantity NUMERIC(14,3) NN`; `status VARCHAR(16) NN`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `expires_at TIMESTAMPTZ ?`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `released_at TIMESTAMPTZ ?`. Cantidad positiva; estados ACTIVE, RELEASED, CONSUMED, EXPIRED.
- **`reorder_rule`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `warehouse_id UUID NN FK → warehouse.id`; `minimum_quantity NUMERIC(14,3) NN`; `target_quantity NUMERIC(14,3) NN`; `lead_time_days INTEGER NN DEFAULT 0`; `preferred_supplier_id UUID ? FK → business_partner.id`; `active BOOLEAN NN DEFAULT TRUE`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`. Cantidades no negativas, objetivo ≥ mínimo, lead time ≥ 0; una regla por producto+almacén.
- **`inventory_count`** — `id UUID PK`; `count_number VARCHAR(48) NN UNIQUE`; `warehouse_id UUID NN FK → warehouse.id`; `status VARCHAR(16) NN DEFAULT 'OPEN'`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `posted_at TIMESTAMPTZ ?`. Estados OPEN, POSTED, CANCELLED; POSTED requiere posted_at y los otros estados lo prohíben.
- **`inventory_count_line`** — `id UUID PK`; `inventory_count_id UUID NN FK → inventory_count.id`; `product_id UUID NN FK → product.id`; `expected_quantity NUMERIC(14,3) NN`; `counted_quantity NUMERIC(14,3) NN`; `unit_cost NUMERIC(14,2) NN`. Cantidades y coste no negativos; una línea por producto/conteo. Líneas capturadas inmutables.
- **`inventory_costing_config`** — `id UUID PK`; `singleton_key VARCHAR(24) NN DEFAULT 'DEFAULT' UNIQUE`; `method VARCHAR(24) NN`; `effective_from TIMESTAMPTZ NN`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`; `updated_by VARCHAR(80) ?`; `updated_at TIMESTAMPTZ ?`. Método FIFO o MOVING_AVERAGE; singleton_key debe ser DEFAULT.
- **`inventory_cost_layer`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `warehouse_location_id UUID NN FK → warehouse_location.id`; `source_movement_id UUID NN FK → stock_movement.id`; `received_quantity NUMERIC(14,3) NN`; `remaining_quantity NUMERIC(14,3) NN`; `unit_cost NUMERIC(14,4) NN`; `currency VARCHAR(3) NN`; `received_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Cantidad recibida > 0, restante entre 0 y recibida, coste ≥ 0.
- **`inventory_valuation_entry`** — `id UUID PK`; `stock_movement_id UUID NN FK → stock_movement.id`; `cost_layer_id UUID ? FK → inventory_cost_layer.id`; `method VARCHAR(24) NN`; `quantity NUMERIC(14,3) NN`; `unit_cost NUMERIC(14,4) NN`; `total_cost NUMERIC(18,4) NN`; `currency VARCHAR(3) NN`; `created_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Método FIFO o MOVING_AVERAGE; cantidades/costes no negativos y cantidad > 0. Ledger inmutable.
- **`inventory_cost_state`** — `id UUID PK`; `product_id UUID NN FK → product.id`; `location_id UUID NN FK → warehouse_location.id`; `quantity NUMERIC(14,3) NN DEFAULT 0`; `total_value NUMERIC(18,4) NN DEFAULT 0`; `currency VARCHAR(3) NN`; `version BIGINT NN DEFAULT 0`; `updated_at TIMESTAMPTZ NN DEFAULT CURRENT_TIMESTAMP`. Cantidad y total_value no negativos; único por producto+ubicación.

### 2.4 Compras, recepciones y devoluciones a proveedor

- **`purchase_order`** — `id UUID PK`; `order_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(32) NN`; `supplier_id UUID NN FK → business_partner.id`; `warehouse_id UUID NN FK → warehouse.id`; `currency VARCHAR(3) NN`; `subtotal NUMERIC(14,2) NN`; `total NUMERIC(14,2) NN`; `notes VARCHAR(1000) ?`; `created_by VARCHAR(80) NN`; `ordered_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN`; `submitted_by VARCHAR(80) ?`; `submitted_at TIMESTAMPTZ ?`; `approved_by VARCHAR(80) ?`; `approved_at TIMESTAMPTZ ?`; `cancelled_by VARCHAR(80) ?`; `cancelled_at TIMESTAMPTZ ?`. Estados DRAFT, SUBMITTED, APPROVED, PARTIALLY_RECEIVED, RECEIVED, CANCELLED.
- **`purchase_order_line`** — `id UUID PK`; `purchase_order_id UUID NN FK → purchase_order.id`; `product_id UUID NN FK → product.id`; `sku_snapshot VARCHAR(64) NN`; `product_name_snapshot VARCHAR(180) NN`; `ordered_quantity NUMERIC(14,3) NN`; `received_quantity NUMERIC(14,3) NN DEFAULT 0`; `unit_cost NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Cantidad pedida > 0; recibida ≥ 0 y ≤ pedida; coste/total ≥ 0.
- **`goods_receipt`** — `id UUID PK`; `receipt_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(16) NN`; `purchase_order_id UUID NN FK → purchase_order.id`; `warehouse_id UUID NN FK → warehouse.id`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `created_by VARCHAR(80) NN`; `received_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN`. Estados POSTED, CANCELLED.
- **`goods_receipt_line`** — `id UUID PK`; `receipt_id UUID NN FK → goods_receipt.id`; `purchase_order_line_id UUID NN FK → purchase_order_line.id`; `product_id UUID NN FK → product.id`; `quantity NUMERIC(14,3) NN`; `unit_cost NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Cantidad > 0, coste/total ≥ 0; único por recepción+línea de orden.
- **`purchase_return`** — `id UUID PK`; `return_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(16) NN`; `purchase_order_id UUID NN FK → purchase_order.id`; `goods_receipt_id UUID NN FK → goods_receipt.id`; `warehouse_id UUID NN FK → warehouse.id`; `reason VARCHAR(1000) NN`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN`. Estados POSTED, CANCELLED.
- **`purchase_return_line`** — `id UUID PK`; `purchase_return_id UUID NN FK → purchase_return.id`; `source_goods_receipt_line_id UUID NN FK → goods_receipt_line.id`; `product_id UUID NN FK → product.id`; `quantity NUMERIC(14,3) NN`; `unit_cost NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Cantidad > 0, costes/total ≥ 0; única por devolución+línea origen.

### 2.5 Ventas, anulaciones, devoluciones de clientes y suscripciones

- **`sale`** — `id UUID PK`; `sale_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(24) NN`; `customer_name VARCHAR(160) ?`; `customer_phone VARCHAR(40) ?`; `channel VARCHAR(24) NN`; `warehouse_id UUID NN FK → warehouse.id`; `subtotal NUMERIC(14,2) NN`; `tax NUMERIC(14,2) NN DEFAULT 0`; `total NUMERIC(14,2) NN`; `currency VARCHAR(3) NN DEFAULT 'USD'`; `idempotency_key VARCHAR(120) ? UNIQUE`; `request_hash VARCHAR(64) ?`; `created_by VARCHAR(80) NN`; `sold_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN`; `customer_id UUID ? FK → business_partner.id`; `cancellation_reason VARCHAR(1000) ?`; `cancellation_requested_by VARCHAR(80) ?`; `cancellation_requested_at TIMESTAMPTZ ?`; `cancellation_approved_by VARCHAR(80) ?`; `cancellation_approved_at TIMESTAMPTZ ?`. Estados COMPLETED, CANCELLATION_REQUESTED, CANCELLED; canales POS, WHATSAPP, WEB, MANUAL, SUBSCRIPTION. La restricción de cancelación exige metadatos de solicitud y aprobación consistentes. Nombre/teléfono histórico deben tratarse como PII y no incluirse por defecto.
- **`sale_item`** — `id UUID PK`; `sale_id UUID NN FK → sale.id`; `product_id UUID NN FK → product.id`; `sku_snapshot VARCHAR(64) NN`; `product_name_snapshot VARCHAR(180) NN`; `quantity NUMERIC(14,3) NN`; `unit_price NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Cantidad > 0 y valores monetarios no negativos.
- **`sales_return`** — `id UUID PK`; `return_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(16) NN`; `source_sale_id UUID NN FK → sale.id`; `warehouse_id UUID NN FK → warehouse.id`; `reason VARCHAR(1000) NN`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN`. Estados POSTED, CANCELLED.
- **`sales_return_line`** — `id UUID PK`; `sales_return_id UUID NN FK → sales_return.id`; `source_sale_item_id UUID NN FK → sale_item.id`; `product_id UUID NN FK → product.id`; `quantity NUMERIC(14,3) NN`; `disposition VARCHAR(16) NN`; `unit_price NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Disposición RESTOCK o SCRAP; cantidad > 0 y valores monetarios no negativos; única por devolución+línea de venta origen.
- **`subscription`** — `id UUID PK`; `subscription_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(16) NN`; `customer_id UUID NN FK → business_partner.id`; `product_id UUID NN FK → product.id`; `warehouse_id UUID NN FK → warehouse.id`; `billing_frequency VARCHAR(16) NN`; `term_months INTEGER NN`; `monthly_price NUMERIC(14,2) NN`; `total_amount NUMERIC(14,2) NN`; `currency VARCHAR(3) NN`; `start_date DATE NN`; `end_date DATE NN`; `prepaid BOOLEAN NN DEFAULT FALSE`; `prepaid_discount_percent NUMERIC(5,2) ?`; `prepaid_amount NUMERIC(14,2) ?`; `cancellation_reason VARCHAR(1000) ?`; `cancelled_by VARCHAR(80) ?`; `cancelled_at TIMESTAMPTZ ?`; `last_billing_error VARCHAR(500) ?`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`; `version BIGINT NN DEFAULT 0`. Estado ACTIVE, CANCELLED, COMPLETED; frecuencia BIWEEKLY, MONTHLY, AT_END; plazo 1, 3, 6, 12 meses; end_date > start_date.
- **`subscription_installment`** — `id UUID PK`; `subscription_id UUID NN FK → subscription.id`; `sequence INTEGER NN`; `period_start DATE NN`; `period_end DATE NN`; `billing_date DATE NN`; `amount NUMERIC(14,2) NN`; `status VARCHAR(16) NN`; `sale_id UUID ? FK → sale.id`; `invoiced_at TIMESTAMPTZ ?`. Secuencia > 0, periodo final > inicial, estados SCHEDULED/INVOICED/CANCELLED; INVOICED ocurre exactamente cuando sale_id no es NULL.

### 2.6 Traslados, movimientos entre almacenes y conteos físicos

- **`stock_transfer`** — `id UUID PK`; `transfer_number VARCHAR(48) NN UNIQUE`; `status VARCHAR(20) NN`; `source_warehouse_id UUID NN FK → warehouse.id`; `destination_warehouse_id UUID NN FK → warehouse.id`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `created_by VARCHAR(80) NN`; `created_at TIMESTAMPTZ NN`; `dispatched_at TIMESTAMPTZ ?`; `completed_at TIMESTAMPTZ ?`; `cancelled_at TIMESTAMPTZ ?`; `cancelled_by VARCHAR(80) ?`; `cancellation_reason VARCHAR(1000) ?`; `reversed_at TIMESTAMPTZ ?`; `reversed_by VARCHAR(80) ?`; `reversal_reason VARCHAR(1000) ?`. Estados DRAFT, IN_TRANSIT, COMPLETED, CANCELLED, REVERSED; almacenes origen/destino deben ser diferentes y los metadatos terminales deben concordar con el estado.
- **`stock_transfer_line`** — `id UUID PK`; `transfer_id UUID NN FK → stock_transfer.id`; `product_id UUID NN FK → product.id`; `sku_snapshot VARCHAR(64) NN`; `product_name_snapshot VARCHAR(180) NN`; `requested_quantity NUMERIC(14,3) NN`; `received_quantity NUMERIC(14,3) NN DEFAULT 0`; `unit_cost NUMERIC(14,2) NN`. Cantidad solicitada > 0; recibida entre cero y solicitada; único por traslado+producto.

### 2.7 Carritos, cotizaciones y pedidos de WhatsApp

- **`whatsapp_cart`** — `id UUID PK`; `customer_ref VARCHAR(120) NN`; `status VARCHAR(16) NN`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`; `expires_at TIMESTAMPTZ NN`. Estados OPEN, QUOTED, ORDERED, EXPIRED; solo un carrito OPEN/QUOTED por customer_ref. Tratar customer_ref como identificador potencialmente personal.
- **`whatsapp_cart_item`** — `id UUID PK`; `cart_id UUID NN FK → whatsapp_cart.id`; `product_id UUID NN FK → product.id`; `quantity NUMERIC(14,3) NN`; `updated_at TIMESTAMPTZ NN`. Cantidad > 0; único por carrito+producto.
- **`whatsapp_quote`** — `id UUID PK`; `quote_number VARCHAR(48) NN UNIQUE`; `cart_id UUID NN FK → whatsapp_cart.id`; `warehouse_id UUID NN FK → warehouse.id`; `status VARCHAR(16) NN`; `currency VARCHAR(3) NN`; `subtotal NUMERIC(14,2) NN`; `tax NUMERIC(14,2) NN DEFAULT 0`; `total NUMERIC(14,2) NN`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `expires_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN`. Estados ACTIVE, ORDERED, EXPIRED.
- **`whatsapp_quote_item`** — `id UUID PK`; `quote_id UUID NN FK → whatsapp_quote.id`; `product_id UUID NN FK → product.id`; `sku_snapshot VARCHAR(64) NN`; `product_name_snapshot VARCHAR(180) NN`; `quantity NUMERIC(14,3) NN`; `unit_price NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Cantidad > 0, importes no negativos; único por cotización+producto.
- **`whatsapp_order`** — `id UUID PK`; `order_number VARCHAR(48) NN UNIQUE`; `cart_id UUID NN FK → whatsapp_cart.id`; `quote_id UUID NN UNIQUE FK → whatsapp_quote.id`; `warehouse_id UUID NN FK → warehouse.id`; `customer_ref VARCHAR(120) NN`; `status VARCHAR(24) NN`; `currency VARCHAR(3) NN`; `subtotal NUMERIC(14,2) NN`; `tax NUMERIC(14,2) NN DEFAULT 0`; `total NUMERIC(14,2) NN`; `idempotency_key VARCHAR(120) NN UNIQUE`; `request_hash VARCHAR(64) NN`; `expires_at TIMESTAMPTZ NN`; `created_at TIMESTAMPTZ NN`; `updated_at TIMESTAMPTZ NN`; `sale_id UUID ? UNIQUE FK → sale.id`; `payment_reference VARCHAR(120) ?`; `payment_method VARCHAR(40) ?`; `confirmed_by VARCHAR(80) ?`; `confirmed_at TIMESTAMPTZ ?`. Estados AWAITING_PAYMENT, CONFIRMED, CANCELLED, EXPIRED. CONFIRMED requiere sale_id, referencia/método de pago, responsable y fecha.
- **`whatsapp_order_item`** — `id UUID PK`; `order_id UUID NN FK → whatsapp_order.id`; `product_id UUID NN FK → product.id`; `sku_snapshot VARCHAR(64) NN`; `product_name_snapshot VARCHAR(180) NN`; `quantity NUMERIC(14,3) NN`; `unit_price NUMERIC(14,2) NN`; `line_total NUMERIC(14,2) NN`. Cantidad > 0, importes no negativos; único por pedido+producto.

## 3. Conteo y agrupación del esquema

La cadena de migraciones de fuente declara **49 tablas** agrupadas así:

| Grupo | Tablas |
|---|---|
| Seguridad/operación técnica | app_user, app_role, app_permission, app_role_permission, security_lock, audit_event, outbox_event, kafka_inbox_event, idempotency_record |
| Catálogo/productos | category, category_attribute_definition, product, product_variant, product_image, unit_of_measure, unit_conversion |
| Socios/almacenes | business_partner, warehouse, warehouse_location |
| Stock/costeo | stock_balance, stock_movement, stock_reservation, reorder_rule, inventory_count, inventory_count_line, inventory_costing_config, inventory_cost_layer, inventory_valuation_entry, inventory_cost_state |
| Compras/recepciones | purchase_order, purchase_order_line, goods_receipt, goods_receipt_line, purchase_return, purchase_return_line |
| Ventas/devoluciones/suscripciones | sale, sale_item, sales_return, sales_return_line, subscription, subscription_installment |
| Traslados | stock_transfer, stock_transfer_line |
| WhatsApp/preventa | whatsapp_cart, whatsapp_cart_item, whatsapp_quote, whatsapp_quote_item, whatsapp_order, whatsapp_order_item |

## 4. Reglas de interpretación del negocio para Peitho

1. **Ventas netas:** no basta con sumar `sale.total`. Filtrar el estado, descontar anulaciones aprobadas y tratar devoluciones `sales_return` + `sales_return_line` según su estado/disposición. No asumir cómo se reflejan impuestos o notas fiscales sin contrato adicional.
2. **Compras y stock recibido:** una orden de compra no incrementa stock por el mero hecho de existir. Usar recepciones publicadas `goods_receipt`/líneas y movimientos de inventario según su semántica efectiva; una devolución a proveedor puede reducir inventario.
3. **Historial de saldo:** `stock_balance` es el estado vigente, no una tabla histórica por fecha. Para reconstrucción temporal usar `stock_movement`, recepciones, ventas, devoluciones, traslados y conteos según eventos/ledger consistentes.
4. **Ledger inmutable:** `stock_movement`, `inventory_valuation_entry` y líneas de conteo capturadas tienen triggers de inmutabilidad en las migraciones. Las correcciones deben modelarse mediante movimientos/operaciones compensatorias, no sobrescribir el pasado.
5. **Ubicaciones:** el esquema permite `warehouse_location`, pero la restricción de `stock_balance` conserva unicidad por producto+almacén durante la transición. No asumir que existe picking multi-bin operativo.
6. **Variantes:** `product_variant` conserva SKU, barcode, opciones, unidad y eventual precio; la migración no crea saldo ni coste por variante. No inferir stock individual de variante si el backend no lo implementa.
7. **Costeo:** la configuración acepta FIFO o MOVING_AVERAGE; `inventory_cost_layer`, `inventory_cost_state` e `inventory_valuation_entry` son distintas capas del modelo. Validar que la operación publicada aplica el costeo esperado antes de tratar el coste/COGS como dato financiero certificado.
8. **WhatsApp y suscripciones:** carritos/cotizaciones/pedidos son estados de preventa/orden; solo el enlace `whatsapp_order.sale_id` identifica la venta generada. Una cuota de suscripción facturada enlaza con `subscription_installment.sale_id`. No contabilizar pedidos/cotizaciones como ventas.
9. **Moneda y precisión:** montos normalmente `NUMERIC(14,2)`, cantidades `NUMERIC(14,3)`, coste de valoración `NUMERIC(14,4)`/total `NUMERIC(18,4)`; conservar precisión y moneda de origen. No convertir unidades sin usar `unit_of_measure`/`unit_conversion`.
10. **Single-company:** las migraciones analizadas no declaran `tenant_id` en las tablas operativas. Un enlace de Inventary a Peitho debe asignarse a un tenant de Peitho mediante configuración de conexión/fuente, fuera de las filas replicadas. Nunca permitir que los datos de una instancia se compartan con otra empresa por una coincidencia de claves.
11. **Zona temporal:** marcas temporales son `TIMESTAMPTZ`; normalizar y presentar en la zona configurada sin perder el instante original. Las fechas de cuotas usan `DATE`.
12. **IDs externos:** conservar UUID e IDs fuente originales. Añadir IDs internos de Peitho en sus propias proyecciones; no reescribir claves de la réplica.
13. **PII:** nombres, teléfono, tax_id, correo, dirección, customer_ref, referencias de pago, usuarios/actores y metadata JSONB pueden ser identificables o contener texto libre. La publicación inicial debe minimizar/omitir campos sensibles por defecto. Si una necesidad exige PII, aprobar finalidad, alcance, permisos, retención, cifrado y auditoría antes de cambiar la lista.
14. **Borrados:** las FK con ON DELETE RESTRICT y ledger inmutable hacen que los cambios no sean equivalentes a borrado físico libre. Comprobar cómo los cambios se reflejan en la réplica y reconciliación; no inferir borrado lógico si no existe columna para ello.
15. **Frescura:** el snapshot debe registrar timestamp de fuente, recepción y aplicación, watermark/LSN, lag, checksum/versión de esquema y resultado de conciliación.

## 5. Diseño de réplica recomendado para Peitho

### 5.1 Separación de espacios de nombres

Las tablas fuente se crean en el esquema PostgreSQL usado por Inventary (normalmente `public`). La replicación lógica nativa no transforma arbitrariamente nombres de tablas, esquemas ni valores. Para evitar colisiones entre las tablas replicadas y el modelo propio de Peitho:

- Reservar en el destino un esquema de lectura para las tablas fuente con nombres compatibles con el publicador, o reservar `public` exclusivamente para el espejo y poner las tablas propias de Peitho en un esquema distinto como `peitho`.
- Confirmar el mapeo exacto con PostgreSQL y probar bootstrap/suscripción antes de aprobar el diseño. No cambiar nombres en el destino esperando que la replicación los traduzca.
- Crear el modelo canónico/analítico en tablas o vistas propias de Peitho; no alterar las tablas replicadas para añadir `tenant_id`, columnas calculadas o estados que no existen en Inventary.
- No aplicar migraciones de Peitho sobre las tablas replicadas. El DDL se coordina con el origen y su cadena Flyway.

### 5.2 Tablas de negocio candidatas por dominio

La lista siguiente es una **candidata para revisión**, no una publicación ya aprobada. La allowlist y las columnas exactas deberán aprobarse en conjunto con el responsable de Inventary y la política de privacidad antes de crear la publicación.

- **Catálogo:** category, category_attribute_definition, product, product_variant, product_image, unit_of_measure, unit_conversion.
- **Socios (columnas mínimas):** business_partner; usar primero ID/code/type/active y evaluar el nombre. Omitir tax_id, email, phone y address de la réplica por defecto.
- **Ubicaciones:** warehouse, warehouse_location.
- **Existencias y costeo:** stock_balance, stock_movement, stock_reservation, reorder_rule, inventory_count, inventory_count_line, inventory_costing_config, inventory_cost_layer, inventory_valuation_entry, inventory_cost_state.
- **Compras:** purchase_order, purchase_order_line, goods_receipt, goods_receipt_line, purchase_return, purchase_return_line.
- **Ventas:** sale, sale_item, sales_return, sales_return_line.
- **Traslados:** stock_transfer, stock_transfer_line.
- **Preventa/pedidos:** whatsapp_cart, whatsapp_cart_item, whatsapp_quote, whatsapp_quote_item, whatsapp_order, whatsapp_order_item; excluir o transformar `customer_ref` en un identificador seudónimo fuera de la publicación nativa.
- **Servicios:** subscription, subscription_installment.
- **Excluir de la publicación analítica inicial:** app_user, app_role, app_permission, app_role_permission, security_lock, audit_event, outbox_event, kafka_inbox_event e idempotency_record. Añadirlos exige justificación concreta, minimización y revisión de seguridad.

La lista de tablas anterior no resuelve por sí sola las columnas. Una publicación de tablas completas puede copiar PII que el producto no necesita. Si hay que filtrar columnas o transformar identificadores, verificar capacidades de publicación de la versión PostgreSQL desplegada y diseñar/pruebar la proyección de ingestión correspondiente; la réplica no es un mecanismo de anonimización.

### 5.3 Permisos y límites

- Un usuario técnico de replicación, con los privilegios mínimos necesarios, no es el mismo usuario de consulta de la API.
- El usuario de consultas y los servicios de Peitho no reciben INSERT, UPDATE, DELETE, TRUNCATE, DDL ni permisos de ejecución arbitraria en Inventary.
- La API de verificación expone operaciones tipadas y acotadas (disponibilidad/precio autorizados), no SQL recibido desde navegador, LLM o agente.
- Aplicar tenant, scope y autorización a las proyecciones de Peitho; las filas fuente no incluyen tenant_id.
- Registrar versión del catálogo, migraciones aplicadas, publication/table allowlist, configuración efectiva, marcas de tiempo, lag y resultados de reconciliación.

## 6. Mapa de uso de datos en Peitho

| Capacidad de Peitho | Tablas fuente de referencia | Regla |
|---|---|---|
| Catálogo, SKU y variantes | category, product, product_variant, category_attribute_definition, unit_of_measure, unit_conversion | Snapshot de origen; respeta unidad y atributos versionados |
| Stock vigente | stock_balance, stock_reservation, warehouse_location, warehouse | No afirmar actualidad si la réplica está retrasada; validar punto a punto si la decisión es sensible |
| Kardex/histórico | stock_movement, inventory_count, inventory_count_line, transfers, recepciones, devoluciones | Ledger y documentos deben conciliar; no confiar solo en saldo actual |
| Compra/recepción/proveedor | business_partner, purchase_order, purchase_order_line, goods_receipt, goods_receipt_line, purchase_return, purchase_return_line | Diferenciar orden, recepción efectiva y devolución |
| Venta neta | sale, sale_item, sales_return, sales_return_line | Manejar cancelación solicitada/aprobada y devoluciones |
| Demanda y ventas omnicanal | sale.channel, sale.sold_at, sale_item; ejecución externa importada | Separar venta observada de atribución de campaña |
| Preventa de WhatsApp | whatsapp_cart*, whatsapp_quote*, whatsapp_order* | Carrito/cotización/pedido no son venta confirmada; el teléfono/customer_ref necesita tratamiento de PII |
| MRR/recurrencia de servicio | product, subscription, subscription_installment, sale | Una cuota solo se considera facturada si está ligada a la venta conforme a la restricción |
| Margen/valoración | product.cost_price, margin_percent, stock movement y tablas de valoración | Identificar el método, la fecha efectiva y el estado de implementación/conciliación |
| Riesgo de obsolescencia | stock_balance, stock_movement, sale/sale_item y periodos stockout | No interpretar ausencia de ventas como falta de demanda durante una rotura de stock |

## 7. Lista de migraciones del snapshot

Las 18 migraciones encontradas en la ruta fuente son:

| Versión | Archivo |
|---|---|
| V1 | V1__create_core_schema.sql |
| V2 | V2__partners_and_procurement.sql |
| V3 | V3__audit_transfers_returns_and_media.sql |
| V4 | V4__purchase_returns_and_reorder_rules.sql |
| V5 | V5__whatsapp_carts_quotes_and_orders.sql |
| V6 | V6__sale_cancellation_movements.sql |
| V7 | V7__case_insensitive_business_keys.sql |
| V8 | V8__link_sales_to_customers.sql |
| V9 | V9__complete_inventory_domain_schema.sql |
| V10 | V10__security_rbac_and_token_revocation.sql |
| V11 | V11__inventory_counts_and_ledger_hardening.sql |
| V12 | V12__approved_sale_cancellations.sql |
| V13 | V13__transfer_compensation_and_consistency.sql |
| V14 | V14__costing_roles_and_operational_completion.sql |
| V15 | V15__idempotent_kafka_consumer_inbox.sql |
| V16 | V16__common_units_of_measure.sql |
| V17 | V17__product_margin_and_derived_prices.sql |
| V18 | V18__service_billing_and_subscriptions.sql |

El JSON [inventary-source-schema-catalog-v18.json](../contracts/schemas/inventary-source-schema-catalog-v18.json) es la referencia de consumo por agentes con nombres de tabla/campos, snapshot de origen y clasificación preliminar de replicación.

## 8. Gates para aprobar la integración

- [ ] Obtener el SHA exacto desplegado en Inventary y confirmar que la CI pasó para ese SHA.
- [ ] Confirmar V1–V18 aplicadas en la instalación origen; resolver formalmente cualquier migración faltante.
- [ ] Consultar `information_schema.columns`, `pg_constraint`, `pg_indexes` y `pg_trigger` de una base de prueba restaurada del esquema esperado y contrastar el catálogo con esta documentación.
- [ ] Aprobar allowlist real de tablas y columnas, incluyendo tratamiento de PII y de JSONB/texto libre.
- [ ] Confirmar soporte de PostgreSQL para la publicación seleccionada, claves de réplica, permisos de replicación y esquema/nombres en el destino.
- [ ] Diseñar el esquema/namespace del destino para que el espejo no colisione con tablas propias de Peitho.
- [ ] Crear migraciones de Peitho independientes y no ejecutar DDL en las tablas fuente replicadas.
- [ ] Ejecutar bootstrap y reconciliación sobre datos sintéticos o snapshot autorizado; probar inserciones, actualizaciones, deletes permitidos, fallos y reanudación.
- [ ] Probar lag/WAL/slots/disco, restauración y resincronización y alertar al superar umbrales.
- [ ] Probar permisos negativos: Peitho no puede escribir ni ejecutar DDL en Inventary y los agentes/LLM no pueden ejecutar SQL arbitrario.
- [ ] Añadir prueba automática de drift para que una nueva migración en Inventary obligue a actualizar el catálogo y el plan de integración de Peitho.
- [ ] Actualizar este catálogo cuando cambien migraciones, fijando nuevo SHA fuente, versión de esquema y evidencia de compatibilidad.

## Fuentes fuente

- [Repositorio Inventary](https://github.com/JhonnyMinian92/inventary)
- [Migración V18 — servicios y suscripciones](https://github.com/JhonnyMinian92/inventary/blob/master/microservices-inventary-ecuapp/src/main/resources/db/migration/V18__service_billing_and_subscriptions.sql)
- [Configuración de Flyway/Hibernate](https://github.com/JhonnyMinian92/inventary/blob/master/microservices-inventary-ecuapp/src/main/resources/application.yaml)
- [README de Inventary](https://github.com/JhonnyMinian92/inventary/blob/master/README.md) — se señala arriba la discrepancia de versión documentada.

