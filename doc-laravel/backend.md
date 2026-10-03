# Backend Strategy

## Stack

- Laravel
- PHP
- MySQL
- Docker

## Architectural intent

The backend must be modeled as a use-case-driven service that exposes a stable API to the frontend while protecting the business rules in the domain.

## Core backend responsibilities

- product catalog management
- user authentication and authorization
- sale registration
- sales listing and retrieval
- reports by date range
- image handling
- database migration and seed execution

## Laravel structure proposal

```text
api/
├── app/
│   ├── Domain/
│   │   ├── Models/
│   │   ├── ValueObjects/
│   │   ├── Exceptions/
│   │   └── Rules/
│   ├── Application/
│   │   ├── Ports/
│   │   ├── UseCases/
│   │   └── Services/
│   ├── Infrastructure/
│   │   ├── Persistence/
│   │   ├── Security/
│   │   ├── Storage/
│   │   └── Transactions/
│   ├── Presentation/
│   │   ├── Http/
│   │   ├── Requests/
│   │   └── Resources/
│   └── Bootstrap/
│       └── Providers/
├── database/
│   ├── migrations/
│   └── seeders/
├── routes/
│   └── api.php
├── tests/
│   ├── Unit/
│   ├── Feature/
│   └── Architecture/
└── .env.example
```

## Important rules

- Domain logic must not be implemented in controllers.
- Eloquent models should not become the domain model itself.
- The application layer must define the ports consumed by infrastructure.
- Business rules must be enforced before persistence writes happen.
- Transactions must be orchestrated with an application-level abstraction instead of direct framework calls in the business layer.

## Backend priorities

1. Define domain entities and value objects.
2. Define use cases and ports.
3. Implement repository contracts.
4. Implement database adapters.
5. Expose HTTP endpoints.
6. Run validation and integration tests.
