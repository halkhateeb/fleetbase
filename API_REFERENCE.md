# Fleetbase API Reference

## Overview

The Fleetbase API is a RESTful API that provides programmatic access to all Fleetbase functionality. It uses JSON for request and response payloads, and standard HTTP methods and status codes.

## Base URL

```
Production: https://api.fleetbase.io/v1
Self-Hosted: http://your-domain.com/api/v1
Local Development: http://localhost:8000/api/v1
```

## Authentication

### API Keys

All API requests must be authenticated using an API key in the request header:

```http
Authorization: Bearer YOUR_API_KEY
```

### Creating API Keys

API keys can be created from the Fleetbase Console:

1. Navigate to **Settings** → **API Keys**
2. Click **Create API Key**
3. Set permissions and expiration
4. Copy and securely store the key

### Key Types

- **Production Keys**: For live production use
- **Test Keys**: For development and testing
- **Restricted Keys**: Limited to specific permissions

## Request Format

### Headers

```http
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
Accept: application/json
```

### Query Parameters

Common query parameters across endpoints:

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | integer | Page number for pagination (default: 1) |
| `limit` | integer | Items per page (default: 20, max: 100) |
| `sort` | string | Sort field (prefix with `-` for descending) |
| `filter` | object | Filter criteria |
| `include` | string | Related resources to include |

## Response Format

### Success Response

```json
{
  "data": {
    "id": "order_123",
    "type": "order",
    "attributes": {
      "customer_name": "John Doe",
      "status": "pending"
    },
    "relationships": {
      "driver": {
        "data": {"id": "driver_456", "type": "driver"}
      }
    }
  },
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested order was not found",
    "status": 404
  }
}
```

### HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 204 | No Content (successful deletion) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Unprocessable Entity (validation error) |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

## Core Resources

### Orders

#### Create Order

```http
POST /v1/orders
```

**Request Body:**

```json
{
  "customer_name": "John Doe",
  "customer_phone": "+1234567890",
  "customer_email": "john@example.com",
  "pickup": {
    "name": "Warehouse A",
    "address": "123 Main St, City",
    "latitude": 40.7128,
    "longitude": -74.0060,
    "phone": "+1234567890"
  },
  "dropoff": {
    "name": "Customer Location",
    "address": "456 Oak Ave, City",
    "latitude": 40.7589,
    "longitude": -73.9851,
    "phone": "+1234567890"
  },
  "items": [
    {
      "name": "Product A",
      "quantity": 2,
      "weight": 5.5,
      "price": 29.99
    }
  ],
  "meta": {
    "notes": "Handle with care"
  }
}
```

**Response:**

```json
{
  "data": {
    "id": "order_abc123",
    "type": "order",
    "attributes": {
      "public_id": "FLB-12345",
      "customer_name": "John Doe",
      "status": "created",
      "created_at": "2024-01-01T12:00:00Z"
    }
  }
}
```

#### Get Order

```http
GET /v1/orders/{id}
```

#### List Orders

```http
GET /v1/orders?status=pending&limit=50&sort=-created_at
```

#### Update Order

```http
PATCH /v1/orders/{id}
```

#### Cancel Order

```http
POST /v1/orders/{id}/cancel
```

### Drivers

#### Create Driver

```http
POST /v1/drivers
```

**Request Body:**

```json
{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "phone": "+1234567890",
  "vehicle_type": "van",
  "vehicle_make": "Ford",
  "vehicle_model": "Transit",
  "vehicle_year": 2022,
  "license_number": "DL123456"
}
```

#### Get Driver

```http
GET /v1/drivers/{id}
```

#### List Drivers

```http
GET /v1/drivers?status=active
```

#### Update Driver Location

```http
POST /v1/drivers/{id}/location
```

**Request Body:**

```json
{
  "latitude": 40.7128,
  "longitude": -74.0060,
  "heading": 90,
  "speed": 45.5,
  "timestamp": "2024-01-01T12:00:00Z"
}
```

### Vehicles

#### Create Vehicle

```http
POST /v1/vehicles
```

#### Get Vehicle

```http
GET /v1/vehicles/{id}
```

#### List Vehicles

```http
GET /v1/vehicles
```

### Places

#### Create Place

```http
POST /v1/places
```

**Request Body:**

```json
{
  "name": "Warehouse A",
  "address": "123 Main St, City, State 12345",
  "latitude": 40.7128,
  "longitude": -74.0060,
  "type": "warehouse",
  "phone": "+1234567890",
  "email": "warehouse@example.com",
  "hours": {
    "monday": "09:00-17:00",
    "tuesday": "09:00-17:00"
  }
}
```

#### Search Places

```http
GET /v1/places/search?query=warehouse&coordinates=40.7128,-74.0060&radius=5000
```

### Tracking

#### Get Tracking Info

```http
GET /v1/tracking/{tracking_number}
```

**Response:**

```json
{
  "data": {
    "tracking_number": "FLB-12345",
    "status": "in_transit",
    "current_location": {
      "latitude": 40.7589,
      "longitude": -73.9851
    },
    "eta": "2024-01-01T15:30:00Z",
    "history": [
      {
        "status": "created",
        "timestamp": "2024-01-01T12:00:00Z",
        "location": "Warehouse A"
      },
      {
        "status": "picked_up",
        "timestamp": "2024-01-01T13:00:00Z",
        "location": "Warehouse A"
      }
    ]
  }
}
```

## Advanced Features

### Webhooks

Register webhooks to receive real-time notifications:

```http
POST /v1/webhooks
```

**Request Body:**

```json
{
  "url": "https://your-domain.com/webhook",
  "events": [
    "order.created",
    "order.status_changed",
    "driver.location_updated"
  ],
  "secret": "your_webhook_secret"
}
```

**Webhook Events:**

- `order.created`
- `order.updated`
- `order.status_changed`
- `order.assigned`
- `order.completed`
- `order.cancelled`
- `driver.location_updated`
- `driver.status_changed`

### Batch Operations

#### Bulk Create Orders

```http
POST /v1/orders/batch
```

**Request Body:**

```json
{
  "orders": [
    { /* order 1 */ },
    { /* order 2 */ },
    { /* order 3 */ }
  ]
}
```

### File Uploads

#### Upload File

```http
POST /v1/uploads
Content-Type: multipart/form-data
```

**Response:**

```json
{
  "data": {
    "id": "file_123",
    "url": "https://cdn.fleetbase.io/files/abc123.pdf",
    "type": "pdf",
    "size": 1024000
  }
}
```

## Real-Time WebSocket API

### Connection

```javascript
const socket = new WebSocket('wss://socket.fleetbase.io');

socket.onopen = () => {
  // Authenticate
  socket.send(JSON.stringify({
    action: 'authenticate',
    token: 'YOUR_API_KEY'
  }));
};
```

### Subscribe to Channels

```javascript
socket.send(JSON.stringify({
  action: 'subscribe',
  channel: 'orders.order_123'
}));
```

### Receive Updates

```javascript
socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received update:', data);
};
```

## Rate Limiting

- **Standard**: 100 requests per minute
- **Burst**: 1000 requests per hour
- **Headers**: Response includes rate limit information

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1609459200
```

## Pagination

### Cursor-Based Pagination

```http
GET /v1/orders?limit=20&cursor=eyJpZCI6IjEyMyJ9
```

**Response:**

```json
{
  "data": [ /* ... */ ],
  "pagination": {
    "next_cursor": "eyJpZCI6IjE0MyJ9",
    "has_more": true
  }
}
```

## Filtering

### Basic Filters

```http
GET /v1/orders?filter[status]=pending&filter[created_at][gte]=2024-01-01
```

### Advanced Filters

```http
GET /v1/orders?filter[customer_name][contains]=John&filter[total][between]=100,500
```

**Available Operators:**

- `eq` - Equal
- `ne` - Not equal
- `gt` - Greater than
- `gte` - Greater than or equal
- `lt` - Less than
- `lte` - Less than or equal
- `in` - In array
- `nin` - Not in array
- `contains` - Contains substring
- `between` - Between two values

## Including Related Resources

```http
GET /v1/orders/{id}?include=driver,vehicle,payload
```

## Field Selection

Request only specific fields:

```http
GET /v1/orders?fields=id,customer_name,status,created_at
```

## Error Handling

### Validation Errors

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "status": 422,
    "details": {
      "customer_email": ["The email field must be a valid email address"],
      "phone": ["The phone field is required"]
    }
  }
}
```

### Best Practices

1. **Always check HTTP status codes**
2. **Implement exponential backoff for rate limits**
3. **Store API keys securely**
4. **Use webhooks for real-time updates**
5. **Implement proper error handling**
6. **Cache responses when appropriate**
7. **Use pagination for large datasets**

## SDKs and Libraries

### Official SDKs

- **JavaScript/Node.js**: `npm install @fleetbase/sdk`
- **PHP**: `composer require fleetbase/fleetbase-php`
- **Python**: `pip install fleetbase`

### SDK Usage Example (JavaScript)

```javascript
import Fleetbase from '@fleetbase/sdk';

const fleetbase = new Fleetbase('YOUR_API_KEY');

// Create an order
const order = await fleetbase.orders.create({
  customer_name: 'John Doe',
  pickup: { /* ... */ },
  dropoff: { /* ... */ }
});

// Get order status
const status = await fleetbase.orders.get(order.id);

// List orders
const orders = await fleetbase.orders.list({
  status: 'pending',
  limit: 50
});
```

## Testing

### Test Environment

Use test API keys for development:

```
Test API URL: https://api.fleetbase.io/test/v1
```

### Test Cards

Test credit card numbers for payment testing:

- Success: `4242424242424242`
- Decline: `4000000000000002`

## Support

- **Documentation**: https://docs.fleetbase.io/api
- **API Status**: https://status.fleetbase.io
- **Support Email**: api-support@fleetbase.io
- **Developer Discord**: https://discord.gg/V7RVWRQ2Wm

## Changelog

Track API changes and updates:

- **v1.2.0** (2024-01-15): Added batch operations
- **v1.1.0** (2023-12-01): Added webhook support
- **v1.0.0** (2023-10-01): Initial API release

## Deprecation Policy

- Deprecated endpoints receive 6 months notice
- Old versions supported for 12 months after deprecation
- Breaking changes are released in new API versions
