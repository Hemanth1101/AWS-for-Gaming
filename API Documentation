# API Documentation

## Overview

This document provides comprehensive documentation for the RESTful APIs of our Cloud-Native SaaS Application. All endpoints are secured with JWT authentication unless specified otherwise.

## Base URL

Production: `https://api.yourcompany.com/v1`  
Development: `https://dev-api.yourcompany.com/v1`

## Authentication

### JWT Authentication

All API requests must include a valid JWT token in the Authorization header:

```
Authorization: Bearer <your_jwt_token>
```

Tokens are obtained through the authentication endpoints or via Cognito directly.

### API Keys

Some public endpoints may use API keys instead of JWT tokens:

```
X-API-Key: <your_api_key>
```

## Rate Limiting

API requests are rate-limited to:
- 100 requests per minute for authenticated users
- 20 requests per minute for unauthenticated users

## Endpoints

### Authentication

#### POST /auth/login

Authenticates a user and returns a JWT token.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600
}
```

**Status Codes:**
- 200 OK: Authentication successful
- 401 Unauthorized: Invalid credentials
- 429 Too Many Requests: Rate limit exceeded

#### POST /auth/refresh

Refreshes an expired JWT token.

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:**
Same as login endpoint.

### Users

#### GET /users/me

Returns the profile of the authenticated user.

**Response:**
```json
{
  "id": "usr_12345",
  "email": "user@example.com",
  "name": "John Doe",
  "role": "admin",
  "subscription": {
    "plan": "premium",
    "status": "active",
    "expiresAt": "2023-12-31T23:59:59Z"
  },
  "createdAt": "2022-01-01T12:00:00Z",
  "updatedAt": "2022-06-01T15:30:00Z"
}
```

#### PUT /users/me

Updates the profile of the authenticated user.

**Request Body:**
```json
{
  "name": "John Smith",
  "email": "john.smith@example.com"
}
```

**Response:**
Updated user object.

### Subscription Management

#### GET /subscriptions

Returns the user's active subscriptions.

**Response:**
```json
{
  "subscriptions": [
    {
      "id": "sub_67890",
      "plan": "premium",
      "status": "active",
      "startDate": "2022-01-01T00:00:00Z",
      "endDate": "2023-01-01T00:00:00Z",
      "autoRenew": true,
      "paymentMethod": {
        "type": "paypal",
        "lastFour": "1234"
      }
    }
  ]
}
```

#### POST /subscriptions

Creates a new subscription.

**Request Body:**
```json
{
  "plan": "premium",
  "paymentMethodId": "pm_12345",
  "couponCode": "WELCOME20"
}
```

**Response:**
```json
{
  "id": "sub_67890",
  "status": "active",
  "startDate": "2022-06-01T00:00:00Z",
  "endDate": "2023-06-01T00:00:00Z",
  "planDetails": {
    "name": "Premium",
    "features": ["feature1", "feature2", "feature3"],
    "price": 99.99,
    "billingCycle": "annual"
  },
  "paymentDetails": {
    "paymentUrl": "https://checkout.paypal.com/123456",
    "invoiceId": "inv_12345"
  }
}
```

### Data Management

#### GET /data

Returns paginated list of user data.

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 10, max: 100)
- `sortBy`: Field to sort by (default: "createdAt")
- `order`: Sort order ("asc" or "desc", default: "desc")

**Response:**
```json
{
  "items": [
    {
      "id": "data_12345",
      "name": "My Data 1",
      "type": "document",
      "size": 1024,
      "createdAt": "2022-05-01T10:00:00Z",
      "updatedAt": "2022-05-01T10:00:00Z"
    }
  ],
  "pagination": {
    "total": 42,
    "pages": 5,
    "page": 1,
    "limit": 10
  }
}
```

#### POST /data

Creates a new data item.

**Request Body:**
```json
{
  "name": "My New Data",
  "type": "document",
  "content": "Base64 encoded content..."
}
```

**Response:**
New data item object.

#### GET /data/{id}

Returns a specific data item.

**Path Parameters:**
- `id`: The ID of the data item

**Response:**
Data item object.

#### PUT /data/{id}

Updates a specific data item.

**Path Parameters:**
- `id`: The ID of the data item

**Request Body:**
```json
{
  "name": "Updated Name"
}
```

**Response:**
Updated data item object.

#### DELETE /data/{id}

Deletes a specific data item.

**Path Parameters:**
- `id`: The ID of the data item

**Response:**
```json
{
  "success": true,
  "message": "Data item deleted successfully"
}
```

### Analytics

#### GET /analytics/dashboard

Returns analytics data for the user dashboard.

**Query Parameters:**
- `period`: Time period for data ("day", "week", "month", "year", default: "month")
- `startDate`: Start date for custom period (ISO format)
- `endDate`: End date for custom period (ISO format)

**Response:**
```json
{
  "usage": {
    "storage": {
      "used": 1024000,
      "total": 5120000,
      "unit": "bytes"
    },
    "requests": {
      "count": 1500,
      "limit": 10000
    }
  },
  "activity": {
    "timeline": [
      {
        "date": "2022-05-01",
        "requests": 150,
        "newItems": 5
      },
      {
        "date": "2022-05-02",
        "requests": 160,
        "newItems": 3
      }
      // More entries...
    ]
  }
}
```

## Errors

All endpoints return standard error responses:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Required field missing",
    "details": {
      "field": "email",
      "reason": "This field is required"
    }
  }
}
```

### Common Error Codes

- `invalid_request`: Missing or invalid request parameters
- `unauthorized`: Authentication required or failed
- `forbidden`: Insufficient permissions
- `not_found`: Requested resource not found
- `rate_limited`: Too many requests
- `internal_error`: Unexpected server error

## Webhooks

The API supports webhooks for event notifications. Configure webhooks in the user dashboard.

### Event Types

- `subscription.created`: New subscription created
- `subscription.updated`: Subscription updated
- `subscription.canceled`: Subscription canceled
- `payment.succeeded`: Payment succeeded
- `payment.failed`: Payment failed
- `user.created`: New user created
- `data.created`: New data item created
- `data.updated`: Data item updated
- `data.deleted`: Data item deleted

### Webhook Payload

```json
{
  "id": "evt_12345",
  "type": "subscription.created",
  "created": "2022-06-01T15:30:00Z",
  "data": {
    "object": {
      "id": "sub_67890",
      "plan": "premium",
      "status": "active"
      // Full object details...
    }
  }
}
```

## SDK Support

We provide SDKs for the following platforms:

- JavaScript/TypeScript: `npm install @yourcompany/api-sdk`
- Python: `pip install yourcompany-api-sdk`
- Java: Available via Maven Central
- Ruby: Available via RubyGems

## Versioning

The API is versioned using URL path versioning. The current version is `v1`.

When a new version is released, the previous version will be supported for at least 12 months.

## Support

For API support, please contact:
- Email: api-support@yourcompany.com
- Developer Portal: https://developers.yourcompany.com
