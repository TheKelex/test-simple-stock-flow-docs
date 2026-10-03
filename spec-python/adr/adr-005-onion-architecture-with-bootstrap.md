# ADR-005 — Onion Architecture with Bootstrap Composition

**Status:** Accepted · **Date:** 2026-10-03

## Context

The project is a stock management and sales system whose business constraints are strict and non-negotiable:

- stock must never become negative
- a sale is immutable once recorded
- closed-period reports must remain historically stable
- user permissions are role-based
- catalog changes must not corrupt past sales data
- the system must run through Docker containers
- the business logic must remain independent from the persistence layer, HTTP layer, and framework details

The requirement is not simply to build an application. The project is explicitly a spec-driven exercise. According to the specification and the architecture documents, the business and the technical implementation must be separated, and the architecture must protect the core rules from external change.

The system is also intentionally split into multiple deployment units:

- frontend application
- backend API service
- database engine
- storage volume

Because of that, the architecture must define boundaries before implementation begins. Otherwise, database access, HTTP concerns, and framework behavior will leak into the business layer and violate the project rules.

## Decision

We will implement an Onion Architecture with a fifth composition layer named Bootstrap.

The system structure will be:

1. Domain
2. Application
3. Infrastructure
4. Presentation
5. Bootstrap

### 1. Domain

The Domain layer contains the business core:

- entities
- value objects
- business invariants
- domain exceptions
- business policies

This layer must not depend on Laravel, HTTP libraries, persistence frameworks, database models, or external infrastructure.

### 2. Application

The Application layer contains:

- use cases
- application services
- inbound ports
- outbound ports
- orchestration of business workflows

This layer owns the execution flow of the system but remains independent from concrete infrastructure implementations.

### 3. Infrastructure

The Infrastructure layer contains:

- database repositories
- ORM models
- persistence adapters
- security adapters
- file-storage adapters
- JWT/token implementations
- transaction managers
- MySQL-specific implementations

This layer implements the contracts defined by the Application layer. It is the technical boundary where concrete technologies are used.

### 4. Presentation

The Presentation layer contains:

- HTTP controllers
- request validation
- response serialization
- API routing
- frontend integration points

This layer translates external requests into application-level actions and translates the results back to the client.

### 5. Bootstrap

The Bootstrap layer contains:

- dependency injection
- service registration
- wiring of ports to implementations
- application startup
- environment composition

Bootstrap is not a source of business logic. Its purpose is to compose the application in a single place so the dependency direction remains clean and visible.

## Alternatives considered

| Alternative | Why not |
|---|---|
| Monolithic Laravel-style service layer with business logic mixed into controllers and models | This would collapse the business layer into the framework itself and violate the project’s dependency rules |
| Domain-first architecture without a Bootstrap layer | This is valid conceptually, but it leaves composition scattered across the codebase and weakens visibility of the dependency flow |
| Infrastructure-led design | This makes the database and framework the center of the system, which contradicts the business rules and the project constitution |
| Pure hexagonal architecture without an explicit composition layer | It is close, but Bootstrap gives a clearer composition boundary and makes the structure easier to maintain in a multi-layer Laravel + React setup |

## Consequences

### Positive consequences

- The business rules remain central and testable.
- The domain is independent from persistence, HTTP, and framework details.
- The application remains stable even if infrastructure changes.
- The system is easier to validate against the specification.
- The dependency direction is explicit and enforceable.
- The system supports Docker execution without violating core architecture constraints.

### Negative consequences / trade-offs

- There is additional structure and some indirection.
- Interfaces, ports, and mappers must be maintained intentionally.
- The team must respect the boundary between business logic and infrastructure.
- Writing the wiring correctly is mandatory; otherwise the architecture becomes decorative instead of real.

## Validation

Before implementation begins, the project will validate the following:

- Domain does not import Laravel or database libraries.
- Application depends only on Domain and its own contracts.
- Infrastructure implements the application ports.
- Presentation does not contain business rules.
- Bootstrap is the only composition point.
- Docker orchestrates the full stack with isolated services.
- The migration strategy is defined before migration execution.
- The system is reviewed against the specification before code is considered complete.

This ADR defines the architectural decision that the project will follow before writing code, in agreement with the SDD workflow and the Onion dependency model.
