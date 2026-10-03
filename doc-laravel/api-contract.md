# API Contract

## Objective

This document defines the interaction contract between the frontend application and the Laravel backend. It is the source of truth for the external behavior of the system and must be respected before implementation begins.

## General principles

- the API must reflect the business specification
- authentication is enforced by role and token
- business invariants are returned as domain-level errors, not as framework leaks
- the frontend must consume this contract exactly
- routes and payloads must be stable and documented before code is written

## Base URL

```text
/api
```

## Authentication

### Login

**POST** `/api/auth/login`

Request body:

```json
{
  "username": "seller01",
  "password": "secret"
}
```

Success response:

```json
{
  "token": "jwt_token",
  "expires_at": "2026-10-03T12:00:00Z",
  "user": {
    "id": "uuid",
    "username": "seller01",
    "role": "seller"
  }
}
```

Error responses:

- invalid credentials: 401 or equivalent domain rejection
- invalid payload: 400
- internal server error: 500

### Register user

**POST** `/api/auth/register`

Requires admin role.

Request body:

```json
{
  "username": "seller02",
  "password": "secret",
  "role": "seller"
}
```

Success response:

```json
{
  "id": "uuid",
  "username": "seller02",
  "role": "seller"
}
```

## Catalog

### List products

**GET** `/api/products`

Query parameters:

- `search` optional
- `category` optional
- `page` optional
- `perPage` optional

Success response:

```json
{
  "data": [
    {
      "id": "uuid",
      "name": "Café",
      "price": 2500,
      "stock": 12,
      "category": {
        "id": "uuid",
        "name": "Beverages"
      },
      "image_url": null
    }
  ],
  "pagination": {
    "currentPage": 1,
    "perPage": 10,
    "totalItems": 12,
    "totalPages": 2
  }
}
```

### Get single product

**GET** `/api/products/{id}`

Success response:

```json
{
  "id": "uuid",
  "name": "Café",
  "price": 2500,
  "stock": 12,
  "category": {
    "id": "uuid",
    "name": "Beverages"
  },
  "image_url": null
}
```

### Create product

**POST** `/api/products`

Requires admin role.

Request body:

```json
{
  "name": "Café",
  "price": 2500,
  "stock": 12,
  "category_id": "uuid",
  "image": null
}
```

Success response:

```json
{
  "id": "uuid",
  "name": "Café",
  "price": 2500,
  "stock": 12,
  "category_id": "uuid"
}
```

### Update product

**PUT** `/api/products/{id}`

Requires admin role.

### Delete product

**DELETE** `/api/products/{id}`

Requires admin role.

This is a logical deletion, not a physical removal.

### Upload product image

**POST** `/api/products/{id}/image`

Requires admin role.

Multipart form upload is expected.

## Categories

### List categories

**GET** `/api/categories`

Success response:

```json
{
  "data": [
    { "id": "uuid", "name": "Beverages" },
    { "id": "uuid", "name": "Snacks" },
    { "id": "uuid", "name": "Cleaning" }
  ]
}
```

## Sales

### Create sale

**POST** `/api/sales`

Requires authenticated user.

Request body:

```json
{
  "lines": [
    {
      "product_id": "uuid",
      "quantity": 2
    },
    {
      "product_id": "uuid-2",
      "quantity": 1
    }
  ]
}
```

Success response:

```json
{
  "id": "uuid",
  "sold_by": "seller01",
  "created_at": "2026-10-03T09:30:00Z",
  "total": 5000,
  "lines": [
    {
      "product_id": "uuid",
      "product_name": "Café",
      "category_name": "Beverages",
      "quantity": 2,
      "unit_price": 2500,
      "subtotal": 5000
    }
  ]
}
```

### List sales

**GET** `/api/sales`

Optional query parameters:

- `from`
- `to`
- `page`
- `perPage`

### Get sale by id

**GET** `/api/sales/{id}`

## Reports

### Sales report by date range

**GET** `/api/reports/sales?from=2026-09-01&to=2026-09-30`

Success response:

```json
{
  "from": "2026-09-01",
  "to": "2026-09-30",
  "totalSales": 12,
  "totalAmount": 230000,
  "rows": [
    {
      "product_id": "uuid",
      "product_name": "Café",
      "category_name": "Beverages",
      "units_sold": 10,
      "amount": 25000,
      "sales_count": 5
    }
  ]
}
```

## Error contract

The API must standardize business and validation errors in a predictable way.

### Domain validation errors

Return 422 for business-rule violations.

```json
{
  "code": "BUSINESS_RULE_VIOLATION",
  "message": "Insufficient stock for product selected",
  "details": {
    "field": "stock"
  }
}
```

### Input validation errors

Return 400 for malformed request data.

```json
{
  "code": "VALIDATION_ERROR",
  "message": "The request payload is invalid",
  "errors": {
    "price": ["Price must be greater than zero"]
  }
}
```

### Authentication and authorization errors

- 401: missing or invalid token
- 403: insufficient permissions
- 404: resource not found
- 405: method not allowed

## Important business contract rules

- sale lines are immutable after creation
- product names and prices are preserved in sale lines
- deleted products must not appear in active catalog queries
- reports must aggregate in the database, not in application memory
- the frontend must not bypass the API contract

## Implementation note

The Laravel backend must expose these routes using the exact contracts above. The frontend must consume the same contract without inventing alternative payload shapes.
