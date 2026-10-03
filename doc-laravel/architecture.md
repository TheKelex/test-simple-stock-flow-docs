# Architecture

## Overview

This project is implemented using Laravel on the backend and React on the frontend, while preserving the original business rules from the specification.

The architecture follows Onion Architecture with a Bootstrap composition layer.

## Core principle

The direction of dependencies must always point inward:

Presentation -> Application -> Domain

Infrastructure implements the contracts defined by the inner layers.

## Layers

### Domain

The Domain layer contains:

- entities
- value objects
- business rules
- invariants
- domain exceptions

This layer must remain independent from:

- Laravel framework classes
- HTTP request objects
- Eloquent models
- database implementation details

### Application

The Application layer contains:

- use cases
- application services
- inbound ports
- outbound ports
- orchestration flows

This layer defines the business use cases without knowing how the infrastructure is implemented.

### Infrastructure

The Infrastructure layer contains:

- Eloquent models
- repositories
- database mappers
- security adapters
- file storage adapters
- transaction management
- MySQL-specific handling

This layer is the implementation of the application contracts.

### Presentation

The Presentation layer contains:

- Laravel controllers
- request validation
- API responses
- React pages, hooks, services, and views

This layer adapts external input into application-scope operations and returns the corresponding response.

### Bootstrap

The Bootstrap layer contains:

- dependency binding
- provider registration
- service composition
- application startup wiring

Bootstrap is responsible for assembling the application but must not contain business logic.

## Why this architecture fits the project

The specification is centered on strict business integrity and historical consistency. Those concerns must be protected in the domain, not mixed into framework or persistence code.

This architecture gives the project the following guarantees:

- business rules remain in the center
- persistence is replaceable
- the API can evolve without damaging the domain
- the system remains testable and traceable
- Docker can orchestrate the stack without changing the business model

## Dependency constraints

The following rule is mandatory:

- Domain must not import Laravel or ORM code.
- Application must only depend on Domain and its own interfaces.
- Infrastructure must implement the contracts of Application.
- Presentation must not contain business rules.
- Bootstrap must be the only composition entry point.

## Result

This structure preserves the original business intent while aligning the project with the actual implementation stack: Laravel, React, MySQL, and Docker.
