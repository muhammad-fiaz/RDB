# API Reference

This document describes the HTTP API endpoints for RDB.

## Base URL

All API requests should be made to `http://localhost:8080` (or your configured host/port).

## Authentication

When authentication is enabled, include the JWT token in the `Authorization` header:

```
Authorization: Bearer <your-jwt-token>
```

## Endpoints

### GET /

**Description:** Health check endpoint.

**Response:**
```json
"RDB Server is running"
```

### GET /status

**Description:** Server status and version information.

**Response:**
```json
{
  "version": "0.1.0",
  "status": "healthy"
}
```

### POST /login

**Description:** Authenticate and get a JWT token.

**Request Body:**
```json
{
  "username": "your_username",
  "password": "your_password"
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

**Error Response (401):**
```json
{
  "status": "error",
  "message": "Invalid credentials"
}
```

### POST /query

**Description:** Execute database queries.

**Request Body:** JSON query object (see [Query Language](../querying.md) for details)

**Example Request:**
```json
{
  "CreateTable": {
    "database": "main",
    "table": "users",
    "columns": [
      {
        "name": "id",
        "type": "int",
        "primary_key": true
      },
      {
        "name": "name",
        "type": "string"
      }
    ]
  }
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "message": "Table 'users' created successfully"
}
```

**Error Response (400):**
```json
{
  "status": "error",
  "message": "Table already exists"
}
```

**Authentication Required:** Yes (if auth is enabled)

## Response Format

All responses follow this general structure:

```json
{
  "status": "success|error",
  "message": "Human-readable message",
  "data": { ... },  // Only present for successful data-returning queries
  "token": "..."    // Only present for login
}
```

## Error Codes

- `200`: Success
- `400`: Bad Request (invalid query)
- `401`: Unauthorized (missing/invalid auth)
- `403`: Forbidden (insufficient permissions)
- `500`: Internal Server Error

## Rate Limiting

Currently no rate limiting is implemented.
