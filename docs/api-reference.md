# API Reference

This document covers the REST API endpoints exposed by the CRM API service (`Backend/crm/`), authentication flows, and shared contracts.

## Base URL

| Environment | URL |
|-------------|-----|
| Local | `http://localhost:3000` |
| Production | `https://xeno-backend-crm.vercel.app` |

## Authentication

### Login

```
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "admin@xeno.local",
  "password": "your-password"
}

Response 200:
{
  "success": true,
  "data": {
    "user": { "id": "...", "name": "...", "email": "...", "role": "ADMIN" },
    "accessToken": "eyJ..."
  }
}
```

The access token is also set as an `httpOnly` cookie for the frontend.

### Register

```
POST /api/v1/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secure-password"
}
```

New MANAGER accounts require admin approval. Email verification is sent automatically.

### Refresh Token

```
POST /api/v1/auth/refresh

Response 200:
{
  "success": true,
  "data": { "accessToken": "eyJ..." }
}
```

### Logout

```
POST /api/v1/auth/logout
```

Clears the refresh token and httpOnly cookie.

## Response Format

All endpoints return a unified envelope:

```typescript
// Success
{
  "success": true,
  "data": T,
  "meta?: { "total": number, "page": number, "pageSize": number },
  "requestId": "uuid"
}

// Error
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Customer not found",
    "details?: ...
  },
  "requestId": "uuid"
}
```

## Pagination

List endpoints support cursor-based pagination:

```
GET /api/v1/customers?page=1&pageSize=20&search=john
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | 1 | Page number (1-indexed) |
| `pageSize` | number | 20 | Items per page (max 100) |
| `search` | string | — | Search filter |
| `cursor` | string | — | Cursor for cursor-based pagination |

## Customers

### List Customers

```
GET /api/v1/customers
Authorization: Bearer <token>
```

### Get Customer by ID

```
GET /api/v1/customers/:id
Authorization: Bearer <token>
```

### Create Customer

```
POST /api/v1/customers
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "phone": "+91-9876543210",
  "tags": ["mumbai", "high-value"],
  "metadata": { "source": "website" }
}
```

### Update Customer

```
PATCH /api/v1/customers/:id
Authorization: Bearer <token>
```

### Delete Customer

```
DELETE /api/v1/customers/:id
Authorization: Bearer <token> (ADMIN only)
```

## Segments

### List Segments

```
GET /api/v1/segments
Authorization: Bearer <token>
```

### Get Segment by ID

```
GET /api/v1/segments/:id
Authorization: Bearer <token>
```

### Create Segment

```
POST /api/v1/segments
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "High Spenders Mumbai",
  "description": "Customers in Mumbai with total spend > 5000",
  "rules": {
    "operator": "AND",
    "conditions": [
      { "field": "city", "operator": "contains", "value": "Mumbai" },
      { "field": "totalSpent", "operator": ">", "value": 5000 }
    ]
  }
}
```

**Rule Engine:**

| Field | Type | Operators |
|-------|------|-----------|
| `totalSpent` | number | `>`, `>=`, `<`, `<=`, `=`, `!=` |
| `orderCount` | number | `>`, `>=`, `<`, `<=`, `=`, `!=` |
| `daysSinceLastOrder` | number | `>`, `>=`, `<`, `<=`, `=`, `!=` |
| `emailEngagement` | number | `>`, `>=`, `<`, `<=`, `=`, `!=` |
| `city` | string | `contains`, `=`, `!=` |

Rules support nested AND/OR groups up to 3 levels deep, with a maximum of 12 conditions per group.

### Update Segment

```
PATCH /api/v1/segments/:id
Authorization: Bearer <token>
```

### Delete Segment

```
DELETE /api/v1/segments/:id
Authorization: Bearer <token> (ADMIN only)
```

## Campaigns

### List Campaigns

```
GET /api/v1/campaigns?status=RUNNING
Authorization: Bearer <token>
```

Optional filter: `status` (DRAFT, QUEUED, RUNNING, PAUSED, COMPLETED, FAILED)

### Get Campaign by ID

```
GET /api/v1/campaigns/:id
Authorization: Bearer <token>
```

### Create Campaign

```
POST /api/v1/campaigns
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Summer Sale 2026",
  "segmentId": "uuid",
  "channel": "WHATSAPP",
  "message": "Get 20% off this summer!",
  "subject": null
}
```

Campaigns start in `DRAFT` status.

### Launch Campaign

```
POST /api/v1/campaigns/:id/launch
Authorization: Bearer <token>
```

Transitions: `DRAFT → QUEUED → RUNNING`. Dispatches jobs to the Channel Simulator.

### Pause Campaign

```
POST /api/v1/campaigns/:id/pause
Authorization: Bearer <token>
```

### Retry Campaign

```
POST /api/v1/campaigns/:id/retry
Authorization: Bearer <token>
```

Re-dispatches failed messages.

### Delete Campaign

```
DELETE /api/v1/campaigns/:id
Authorization: Bearer <token> (ADMIN only)
```

## Analytics

### Dashboard Metrics

```
GET /api/v1/analytics/dashboard
Authorization: Bearer <token>
```

Returns: totalCustomers, totalOrders, totalRevenue, activeCampaigns, delivery/open/click/conversion rates, trends, and activity feed.

### Campaign Performance

```
GET /api/v1/analytics/campaigns/:id
Authorization: Bearer <token>
```

Returns: funnel (sent/delivered/opened/clicked/converted/failed), rates, revenue, failure breakdown.

### Revenue Analytics

```
GET /api/v1/analytics/revenue
Authorization: Bearer <token>
```

### Delivery Analytics

```
GET /api/v1/analytics/delivery
Authorization: Bearer <token>
```

## AI Copilot

### Send Message

```
POST /api/v1/ai/conversations/:conversationId/messages
Authorization: Bearer <token>
Content-Type: application/json

{
  "message": "Create a segment for high-value Mumbai customers"
}
```

### Stream Message (SSE)

```
GET /api/v1/ai/conversations/:conversationId/messages/stream?message=...
Authorization: Bearer <token>
Accept: text/event-stream
```

Events:
- `tool-call` — Tool execution started
- `tool-result` — Tool execution completed
- `confirmation` — Destructive action requires confirmation
- `final-response` — Complete response with grounding data
- `error` — Error occurred

### Confirm Tool Execution

```
POST /api/v1/ai/tools/:executionId/confirm
Authorization: Bearer <token>
```

### Cancel Tool Execution

```
POST /api/v1/ai/tools/:executionId/cancel
Authorization: Bearer <token>
```

### List Conversations

```
GET /api/v1/ai/conversations
Authorization: Bearer <token>
```

### Get Conversation

```
GET /api/v1/ai/conversations/:id
Authorization: Bearer <token>
```

## Webhooks

### Channel Callback

```
POST /api/v1/webhooks/channel
Content-Type: application/json
x-xeno-signature: sha256=<hmac-signature>
x-correlation-id: <uuid>

{
  "eventId": "uuid",
  "type": "MessageDelivered",
  "occurredAt": "2026-06-14T10:00:00Z",
  "campaignId": "uuid",
  "customerId": "uuid",
  "correlationId": "uuid",
  "payload": { "provider": "xeno-channel-simulator" }
}
```

The signature is verified using HMAC-SHA256 with the shared `CHANNEL_WEBHOOK_SECRET`.

## Health

```
GET /api/health          → Liveness check
GET /api/health/ready    → Readiness check (includes DB connectivity)
```
