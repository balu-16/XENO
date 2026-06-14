# Architecture

Xeno is a distributed system composed of three independently deployable services that communicate via REST APIs, HMAC-signed webhooks, and background job queues.

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend (Port 5173)                       │
│              React + TanStack Start + Recharts                │
│              AI Copilot Panel + SSE Dashboard                 │
└──────────────────────┬──────────────────────────────────────┘
                       │ REST API + SSE
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    CRM API (Port 3000)                        │
│              NestJS + Prisma ORM + BullMQ                    │
│              Auth · Customers · Segments · Campaigns          │
│              AI Orchestrator (9+ tools) · Webhook Receiver    │
└──────┬──────────────────┬──────────────────┬────────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
┌──────────────┐  ┌───────────────┐  ┌──────────────────────┐
│  PostgreSQL  │  │  Upstash Redis│  │  Channel Simulator   │
│  (Neon/      │  │  (BullMQ)     │  │  (Port 3001)         │
│   Supabase)  │  │               │  │  NestJS + HMAC       │
└──────────────┘  └───────────────┘  └──────────┬───────────┘
                                                 │
                                          HMAC Webhooks
                                                 │
                                                 ▼
                                    CRM API Webhook Receiver
```

## Services

### Frontend (`Frontend/`)

- **Framework:** TanStack Start (SSR-capable React meta-framework)
- **Routing:** TanStack Router with file-based routes
- **State:** React Query for server state, local state via React hooks
- **UI:** Tailwind CSS 4 + shadcn/ui (Radix primitives)
- **Charts:** Recharts for analytics visualizations
- **AI Panel:** Persistent copilot sidebar with conversation history and tool execution visibility

**Key routes:**
| Route | Purpose |
|-------|---------|
| `/auth` | Login / registration |
| `/dashboard` | Main dashboard with KPIs |
| `/customers` | Customer list and management |
| `/segments` | Segment builder with rule editor |
| `/campaigns` | Campaign list, creation wizard, detail view |
| `/analytics` | Global analytics and funnel charts |
| `/ai` | Full-page AI copilot interface |
| `/insights` | AI-generated business insights |
| `/managers` | Admin user management |

### CRM API (`Backend/crm/`)

The main backend service handling all business logic.

**Modules:**
- **Auth** — JWT access tokens + refresh tokens, role-based access (ADMIN, MANAGER), email verification
- **Customers** — CRUD with GIN-indexed tags, search, pagination
- **Segments** — JSON rule engine with AND/OR groups, up to 3 levels deep
- **Campaigns** — Full lifecycle management (DRAFT → QUEUED → RUNNING → COMPLETED)
- **Analytics** — Real-time metrics, funnel analysis, revenue attribution
- **AI** — Tool-augmented copilot orchestration with grounding verification
- **Queue** — BullMQ job dispatch for campaign delivery and receipt processing
- **Webhooks** — HMAC-SHA256 signed webhook receiver for channel callbacks
- **Health** — Liveness and readiness probes

**Workers (run in same process, separate entry point):**
- **Receipt Worker** — Processes delivery status callbacks from the channel service
- **Analytics Worker** — Refreshes campaign analytics projections after events

### Channel Simulator (`Backend/channel/`)

An independently deployable service that simulates message delivery across channels (WhatsApp, SMS, Email, RCS).

**Responsibilities:**
- Receives dispatch jobs from the CRM API
- Simulates delivery lifecycle with deterministic timing
- Sends HMAC-signed webhook callbacks for each status transition
- Generates realistic delivery outcomes using SHA-256 scoring

**Delivery simulation scoring:**
| Score Range | Outcome |
|-------------|---------|
| 0–19 | Message failed (provider rejection or invalid destination) |
| 20–94 | Message delivered |
| 95–154 | Message opened (mobile or desktop) |
| 155–219 | Message clicked |
| 220–255 | Message converted (with attributed order) |

## Data Flow

### Campaign Lifecycle

```
1. User creates campaign (DRAFT)
2. User launches campaign → QUEUED
3. CRM dispatches jobs to Channel Simulator → RUNNING
4. Channel simulates delivery, sends webhooks back
5. Receipt Worker processes webhooks, updates CampaignLog + CampaignEvent
6. Analytics Worker refreshes CampaignAnalytics
7. When all messages processed → COMPLETED
```

### Webhook Flow

```
Channel Simulator                    CRM API
      │                                │
      │  POST /api/v1/webhooks/channel │
      │  Headers:                      │
      │    x-xeno-signature: sha256=...│
      │    x-correlation-id: uuid      │
      │  Body: ChannelWebhook          │
      │───────────────────────────────>│
      │                                │  1. Verify HMAC signature
      │                                │  2. Store WebhookReceipt
      │                                │  3. Enqueue receipt-processing job
      │                                │  4. Worker updates CampaignLog
      │                                │  5. Enqueue analytics-refresh job
      │  HTTP 200 OK                   │
      │<───────────────────────────────│
```

### AI Copilot Flow

```
User Message
    │
    ▼
Input Sanitization (strip control chars, truncate to 4000 chars)
    │
    ▼
Injection Detection (regex patterns for prompt injection)
    │
    ▼
Context Assembly (last 40 messages + system prompt)
    │
    ▼
Anthropic Claude (intent classification + tool selection)
    │
    ▼
Tool Execution Fan-Out (parallel validated tools)
    │
    ▼
Grounding Verification (every claim must trace to tool output)
    │
    ▼
Response Generation (formatted markdown, never raw JSON)
    │
    ▼
Conversation Persistence (AIConversation + AIMessage + AIToolExecution)
```

## Shared Contracts

Zod schemas and TypeScript interfaces are **duplicated** in each service (`contracts.ts`) to maintain deployment independence. Key shared types:

- `Channel` — WHATSAPP, SMS, EMAIL, RCS
- `CampaignStatus` — DRAFT, QUEUED, RUNNING, PAUSED, COMPLETED, FAILED
- `CampaignEventType` — CampaignCreated through MessageFailed
- `DeliveryStatus` — QUEUED through FAILED
- `SegmentCondition` — field/operator/value with validation
- `SegmentRuleGroup` — nested AND/OR groups (max 3 levels)
- `CampaignDispatchJob` — job payload for channel dispatch
- `ChannelWebhook` — webhook payload from channel to CRM
- `ApiResponse<T>` — unified success/error envelope

## Security

- **Authentication:** JWT access tokens in httpOnly cookies, refresh token rotation
- **Authorization:** Role-based (ADMIN, MANAGER) with guard decorators
- **Webhook verification:** HMAC-SHA256 signatures on all channel callbacks
- **AI safety:** Input sanitization, injection detection, grounding verification, confirmation for destructive operations
- **HTTP security:** Helmet headers, CORS configuration, rate limiting via throttler
