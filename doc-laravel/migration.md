# Migration and Data Strategy

## Goal

The database must reflect the business model defined by the specification, with clear separation between domain rules and persistence constraints.

## Core principles

- migrations belong to the backend service, not the infrastructure repository
- the schema must be versioned and reproducible
- seed data must be explicit and idempotent
- deleted products must use logical deletion instead of strict physical deletion
- the sales data must preserve historical facts
- aggregate-derived values must not be persisted as independent stored columns

## Required database concepts

- product
- category
- sale
- sale_item
- user

## Business constraints to reflect in the schema

- stock cannot be negative
- price must be positive
- quantity must be positive
- category is required and exists
- usernames are unique
- roles are restricted to an allowed set
- sales are immutable
- for historical reports, product names and prices are frozen in sale lines

## Migration strategy

1. Define the schema in the backend migration layer.
2. Use Laravel migration files to create tables and constraints.
3. Keep migration files intentional and small.
4. Use seeds for reference data, especially categories.
5. Use database constraints to protect invariants that can be enforced at the engine level.
6. Keep domain-level validation in the application and domain layers.

## Seed strategy

Reference data must be seeded with consistency and repeatability:

- categories must exist at startup
- initial admin account must be created through a controlled process
- demo data must not compromise production invariants

## Important notes

- The database is not the source of business rules.
- The engine enforces only what can be enforced natively.
- Domain invariants remain the real contract.
