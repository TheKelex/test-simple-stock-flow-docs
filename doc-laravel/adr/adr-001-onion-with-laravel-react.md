# ADR-001 — Onion Architecture with Laravel and React

**Status:** Accepted

## Context

The original specification defines a business system with strict rules around inventory, sale immutability, and reporting behavior. The implementation stack is not Python or .NET; it is Laravel for the backend and React for the frontend.

The technical stack must preserve the business rules while adapting to the real runtime environment.

## Decision

We will implement the project using Onion Architecture, adapted to Laravel and React.

The business logic will live in the core layers, while Laravel, Eloquent, and React will be treated as implementation details.

## Consequences

### Positive

- business rules remain central and protected
- infrastructure can change without rewriting the business model
- tests can be organized by domain and application boundaries
- the project remains aligned with the original specification

### Negative

- the architecture requires clearer separation of responsibilities
- more abstractions are necessary than in a monolithic implementation
- the team must respect boundaries consistently

## Reasoning

Laravel and React are implementation technologies, not the domain model. The project is therefore structured around the domain and application layers, with infrastructure and presentation on the outside.
