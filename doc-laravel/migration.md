# Migration and Data Strategy

## Goal

The database schema must reflect the business model defined by the specification, while preserving the architectural rule that the domain remains independent from persistence details. The persistence layer exists to store facts, not to define the business rules themselves.

## Core principles

- migrations belong to the backend service, not the infrastructure repository
- schema changes must be versioned, reproducible, and backward-safe
- seed data must be explicit, idempotent, and compatible with the initial lifecycle of the system
- logical deletion must be used for products instead of physical deletion
- historical sales data must remain immutable once recorded
- derived values such as report totals must not be stored as independent source-of-truth fields
- the engine may enforce only the invariants it can express natively; business invariants remain in the domain and application layers

## Required persistence model

The system requires five primary persistence entities:

- product
- category
- sale
- sale_item
- user

These entities are not merely tables. Their structure must preserve the business semantics behind them.

## Entity responsibilities

### Product

The product is the main catalog entity.

Required characteristics:

- id: immutable unique identifier
- name: business name, visible in the catalog and frozen in historical sales lines
- price: positive value, represented in cents or a precise decimal-compatible type
- stock: integer quantity greater than or equal to zero
- category_id: required reference to a category
- deleted_at: soft-deletion marker used so historical sales remain valid
- created_at and updated_at: standard system timestamps

Business rules enforced at the domain and application layers:

- stock cannot become negative
- price must be greater than zero
- product names and prices are frozen at sale time and must not be recalculated from the current catalog state

### Category

The category is a reference entity used by products and reports.

Required characteristics:

- id
- name
- created_at and updated_at

Rules:

- category names must be unique enough to keep the catalog coherent
- categories should be stable and seeded before the catalog is used in production-like flows

### User

The user represents the authenticated operator of the system.

Required characteristics:

- id
- username
- password_hash
- role
- created_at and updated_at

Rules:

- username is unique and normalized consistently
- role must be restricted to the allowed domain values: seller or admin
- only admin users may create or manage catalog entries beyond basic seller operations

### Sale

Sales are immutable business facts.

Required characteristics:

- id
- seller_id
- total_amount
- created_at
- maybe version field for optimistic concurrency control, if needed by the implementation strategy

Rules:

- a sale must contain at least one line
- sale totals are derived from the lines, not arbitrary manual input
- sale records are not editable after creation
- a sale is a historical fact and must not be deleted

### Sale item

Sale items capture the frozen data of a product at the time of sale.

Required characteristics:

- id
- sale_id
- product_id
- product_name
- category_name
- quantity
- unit_price
- subtotal
- created_at

Rules:

- the item preserves the product name and price as they existed at the moment of sale
- the category name is frozen to guarantee report consistency when categories change later
- the item must remain available for historical reporting even if the original product is later renamed or logically deleted

## Business invariants to reflect in schema and code

The schema can enforce some invariants natively, but the actual semantic validation remains in the domain and application layers.

### Engine-level constraints

These are the constraints the database should enforce when possible:

- stock cannot be negative
- price must be positive
- quantity must be positive
- category_id must be non-null when creating a product
- username must be unique
- role must be within an allowed set
- sale_item quantity and unit_price must be non-negative
- product deletion must be soft, not hard

### Application- and domain-level invariants

These remain the real source of truth:

- a sale cannot be modified after creation
- stock checks must be validated before a sale is accepted
- inventory deduction and sale persistence must be executed atomically
- report calculations must aggregate data from the sales facts and not from the live catalog state

## Migration order

The migration sequence should be intentional and deterministic:

1. create categories
2. create users
3. create products
4. create sales
5. create sale_item rows linked to sales
6. add soft-deletion and optimistic concurrency support where required
7. add indexes for searching and reporting

This order ensures the schema remains stable and the business lifecycle is preserved.

## Migration guidelines

### 1. Keep migrations small and explicit

Each migration should handle one coherent change to the schema and be aligned with a single business decision.

### 2. Do not mix domain logic and schema logic

The migration should define the shape of storage; it should not encode business behavior beyond what the database can enforce natively.

### 3. Prefer engine-enforced constraints for native invariants

Examples:

- positive price
- positive quantity
- non-null foreign keys
- unique usernames
- limited role values

### 4. Keep historical data intact

The schema must preserve business history. Soft-deleted products should remain accessible for old sales, and sale records should not be rewritten or deleted.

### 5. Do not persist report aggregates as source data

Reports are derived views over sales and must not become a separate business table unless the specification explicitly requires it.

## Seed plan

Initial seed data must be explicit and safe.

### Categories

At minimum, categories must be seeded before the catalog becomes functional. They provide stable classification and will be used by both the product catalog and report aggregation.

### Initial admin user

The system needs a bootstrap admin user to create the operational environment. This must be created through a controlled, documented process rather than by ad hoc SQL or direct database manipulation.

### Demo data

The project may include representative demo data only when it does not compromise the contract. Demo data must not hide domain errors or break the historical integrity of reports.

## Logical deletion strategy

Products must not be physically removed after they have been sold.

This means:

- the product record remains in storage with a soft-delete marker
- catalog queries exclude deleted products
- historical sale items continue to reference the original product state for reporting and traceability
- a deleted product must not reappear in the active catalog

This behavior is essential to preserve the historical integrity of sales and reporting.

## Report consistency and frozen data

The report must remain stable even when catalog data changes.

That is why sale_item rows must keep:

- product name
- category name
- unit price
- quantity

This ensures that a report generated later remains consistent with the historical fact that was sold, even if the product is renamed or reclassified.

## Recommended migration checklist

Before implementation is considered valid, the migration layer should confirm:

- the database schema matches the required entities
- foreign keys are present and correctly constrained
- positive invariants are enforced at the engine level where possible
- soft delete is implemented for products
- unique constraints exist for usernames and category-level identity rules
- historical sales lines preserve frozen values
- seed data is deterministic and safe
- the system can bootstrap without manual SQL work

## Implementation note

Laravel migrations are the technical mechanism used to create this schema, but they are not the business model. The domain remains the source of behavioral truth. Migrations are a persistence contract that must align with the architecture and the specification, not an independent design authority.
