# Docker Strategy

## Goal

The full system must be run through containers, without relying on local installation of PHP, Composer, Node, or MySQL.

## Runtime components

- MySQL container
- Laravel API container
- React app container
- optional seed or validation container

## Principle

The infrastructure repository provides containers and orchestration, but the schema and application logic remain in the backend service.

## Container responsibilities

### Database

- starts MySQL 8.4
- uses a named volume for persistence
- exposes only the required port
- initializes the empty database

### API

- runs Laravel application
- executes migrations during startup or deployment step
- exposes the API to the frontend
- depends on database health

### Frontend

- runs the React application
- serves through a dev server or static build
- communicates with the API

## Startup requirements

- database must be healthy before the API starts
- migrations must be executed before app-level operations are used
- environment variables must be provided through `.env` files or compose variables
- secrets must not be hard-coded

## Environment validation

Before implementation is considered complete, the system must be verifiable through Docker with:

- full container bootstrap
- database connectivity
- API start-up
- frontend start-up
- required service health checks
