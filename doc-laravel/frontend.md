# Frontend Strategy

## Stack

- React
- Vite or equivalent SPA structure
- HTTP client for API communication

## Architectural intent

The frontend must consume the API and present the business flows without owning the domain rules. The frontend is a presentation layer.

## Frontend responsibilities

- login and session state
- product catalog browsing
- category filtering
- product creation and editing
- product image management
- sales registration
- sales listing
- report generation
- UI feedback and validation rendering

## Frontend structure proposal

```text
app/src/
├── domain/
│   ├── models/
│   └── errors/
├── application/
│   ├── ports/
│   ├── useCases/
│   └── state/
├── infrastructure/
│   ├── http/
│   ├── mappers/
│   └── providers/
├── features/
│   ├── auth/
│   ├── catalog/
│   ├── sales/
│   └── reports/
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
├── routes/
├── App.tsx
└── main.tsx
```

## Important rules

- the frontend must not duplicate business logic from the backend
- presentation components must call use cases or service abstractions rather than directly mixing data logic with view logic
- API contracts must be respected exactly
- UI state must reflect server validation and business errors clearly

## Frontend priorities

1. define the API client layer
2. build auth flow
3. build catalog flow
4. build sales flow
5. build reports flow
6. validate with production-like requests
