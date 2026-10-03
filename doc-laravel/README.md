# Spec Laravel — Simple Stock Flow

This folder contains the project-specific documentation for the Laravel + React implementation derived from the original specification.

## Purpose

This documentation is the technical translation of the original project specification into the target stack:

- Backend: Laravel (PHP)
- Frontend: React
- Persistence: MySQL
- Runtime: Docker
- Architecture: Onion Architecture with a Bootstrap composition layer

It is intended to preserve the original business rules while adapting them to the real stack used in the challenge.

## Source of truth

The original business specification remains in:

- `docs/spec-python/`
- `docs/spec-.net/`

This folder is the implementation guide for the actual delivery stack, not a replacement for the business specification.

## Principles applied

### 1. SDD first

The implementation must be driven by the specification before code is written.

### 2. Onion Architecture

The business core must remain independent from the framework, the HTTP layer, and the persistence layer.

### 3. Dependency direction

Dependencies must point inward toward the business domain.

### 4. Laravel and React as technical implementation only

Laravel and React are not the business model. They are execution technologies used to expose and operate the rules defined by the specification.

### 5. Docker as the execution environment

The complete system must run through Docker, including database, api, and frontend services.

## Structure inside this folder

- `architecture.md` — architecture and layer definitions
- `backend.md` — backend rules and Laravel structure
- `frontend.md` — React responsibilities and UI boundaries
- `phase-5-api.md` — Phase 5 API implementation scope, tests, and acceptance criteria
- `phase-6-frontend.md` — Phase 6 React workflows, UI states, tests, and acceptance criteria
- `migration.md` — migration, schema, and seed strategy
- `docker.md` — Docker runtime and startup flow
- `adr/` — architecture decisions aligned to the implementation stack

## Relationship with the original spec

This folder does not change the business meaning of the project. It only translates the specification into the real implementation stack used by the challenge.
