# Database Schema

Xeno uses **PostgreSQL** (via Neon or Supabase) with **Prisma ORM** for type-safe database access. The schema is defined in `Backend/crm/prisma/schema.prisma`.

## Entity Relationship Overview

```
User ──────────────┬── AIConversation ── AIMessage
                   │                 ├── AIToolExecution
                   │                 └── AIDecisionLog
                   ├── RefreshToken
                   └── EmailVerification

Customer ──────────┬── Order
                   ├── CampaignLog
                   └── CampaignEvent

Segment ──────────── Campaign ──────── CampaignEvent
                              │    ├── CampaignLog
                              │    ├── CampaignAnalytics
                              │    └── WebhookReceipt

AIInsight ──────────┬── AIInsightAction
                   ├── AIInsightOutcome
                   └── AIInsightFeedback

AIInsightCorrelation (standalone)
AIExecutiveScore (standalone)
ProcessingFailure (standalone)
CustomerLoginLog (standalone)
AdminLoginLog (standalone)
```

## Models

### User

Admin and manager accounts for the CRM system.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK, auto-generated |
| `name` | String | Required |
| `email` | String | Unique |
| `passwordHash` | String | Argon2 hashed |
| `emailVerified` | Boolean | Default: false |
| `role` | Enum | ADMIN or MANAGER (default: MANAGER) |
| `approvalStatus` | Enum | PENDING, APPROVED, REJECTED (default: PENDING) |
| `createdAt` | DateTime | Auto |
| `updatedAt` | DateTime | Auto |

### Customer

End customers targeted by marketing campaigns.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `name` | String | Indexed |
| `email` | String | Unique |
| `phone` | String | Required |
| `tags` | String[] | GIN-indexed array |
| `metadata` | JSON | Flexible key-value store |
| `createdAt` | DateTime | Indexed |

**Indexes:** `name`, `createdAt`, `tags` (GIN for array containment queries)

### Order

Customer purchase orders used for revenue attribution.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `customerId` | UUID | FK → Customer (cascade delete) |
| `amount` | Decimal(12,2) | Required |
| `items` | JSON | Order line items |
| `createdAt` | DateTime | Indexed |

**Indexes:** `(customerId, createdAt)`, `createdAt`

### Segment

Audience segments defined by JSON rule groups.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `name` | String | Indexed |
| `description` | String? | Optional |
| `rules` | JSON | SegmentRuleGroup (AND/OR nested conditions) |
| `createdAt` | DateTime | Indexed |
| `updatedAt` | DateTime | Auto |

**Rules JSON structure:**
```json
{
  "operator": "AND",
  "conditions": [
    { "field": "totalSpent", "operator": ">", "value": 5000 },
    { "field": "city", "operator": "contains", "value": "Mumbai" }
  ]
}
```

### Campaign

Marketing campaigns targeting customer segments.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `name` | String | Required |
| `segmentId` | UUID | FK → Segment |
| `channel` | Enum | WHATSAPP, SMS, EMAIL, RCS |
| `status` | Enum | DRAFT, QUEUED, RUNNING, PAUSED, COMPLETED, FAILED |
| `subject` | String? | Email subject line |
| `message` | String | Campaign message body |
| `audienceSizeSnapshot` | Int | Segment size at campaign creation |
| `scheduledAt` | DateTime? | Optional scheduling |
| `launchedAt` | DateTime? | Set when campaign is launched |
| `completedAt` | DateTime? | Set when all messages processed |

**Indexes:** `segmentId`, `(status, createdAt)`, `channel`, `scheduledAt`

### CampaignEvent

Immutable event log for all campaign-related occurrences.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `eventId` | UUID | Unique (idempotency key) |
| `type` | Enum | CampaignCreated, CampaignLaunched, MessageQueued, MessageSent, MessageDelivered, MessageOpened, MessageClicked, MessageConverted, MessageFailed |
| `campaignId` | UUID | FK → Campaign (cascade delete) |
| `customerId` | UUID? | FK → Customer (set null on delete) |
| `attributedOrderId` | UUID? | FK → Order |
| `correlationId` | UUID | Links related events |
| `payload` | JSON | Event-specific data |
| `occurredAt` | DateTime | When the event actually happened |

**Indexes:** `(campaignId, type, occurredAt)`, `(customerId, occurredAt)`, `correlationId`

### CampaignLog

Per-customer delivery status within a campaign (latest state).

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `campaignId` | UUID | FK → Campaign (cascade delete) |
| `customerId` | UUID | FK → Customer (cascade delete) |
| `status` | Enum | QUEUED, SENT, DELIVERED, OPENED, CLICKED, CONVERTED, FAILED |
| `failureReason` | String? | Error description if failed |
| `attributedOrderId` | UUID? | Unique — links to converted order |
| `lastEventAt` | DateTime | Timestamp of last status change |

**Constraints:** Unique on `(campaignId, customerId)` — one log entry per customer per campaign.

### CampaignAnalytics

Pre-computed analytics projections for campaigns.

| Field | Type | Constraints |
|-------|------|-------------|
| `campaignId` | UUID | PK, FK → Campaign (cascade delete) |
| `totalAudience` | Int | Segment size |
| `totalQueued` | Int | Messages queued |
| `totalSent` | Int | Messages sent |
| `totalDelivered` | Int | Messages delivered |
| `totalOpened` | Int | Messages opened |
| `totalClicked` | Int | Messages clicked |
| `totalConverted` | Int | Messages converted |
| `totalFailed` | Int | Messages failed |
| `deliveryRate` | Float | totalDelivered / totalSent |
| `openRate` | Float | totalOpened / totalDelivered |
| `clickRate` | Float | totalClicked / totalOpened |
| `conversionRate` | Float | totalConverted / totalClicked |
| `revenueAccrued` | Decimal(14,2) | Sum of attributed order amounts |

### WebhookReceipt

Raw webhook payloads received from the channel service.

| Field | Type | Constraints |
|-------|------|-------------|
| `id` | UUID | PK |
| `eventId` | UUID | Unique (idempotency) |
| `campaignId` | UUID | FK → Campaign |
| `customerId` | UUID | Required |
| `type` | Enum | CampaignEventType |
| `correlationId` | UUID | Links to dispatch job |
| `payload` | JSON | Raw webhook body |
| `receivedAt` | DateTime | Indexed |
| `processedAt` | DateTime? | Set after worker processes |
| `attempts` | Int | Processing attempt count |
| `error` | String? | Last processing error |

### AI Conversation Models

#### AIConversation
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | PK |
| `userId` | UUID? | FK → User (set null on delete) |
| `title` | String | Auto-generated from first message |

#### AIMessage
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | PK |
| `conversationId` | UUID | FK → AIConversation |
| `role` | Enum | USER or ASSISTANT |
| `content` | String | Message text |
| `grounding` | JSON? | Tool sources and execution IDs |

#### AIToolExecution
| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | PK |
| `toolName` | String | Tool that was called |
| `status` | Enum | STARTED, PENDING_CONFIRMATION, COMPLETED, CANCELED, FAILED |
| `input` | JSON | Validated tool input |
| `output` | JSON? | Tool output |
| `durationMs` | Int? | Execution time |
| `requiresConfirmation` | Boolean | Whether user confirmation was needed |

#### AIDecisionLog
Audit trail for AI orchestration decisions, tracking which tools were chosen, token usage, and execution time.

### AIInsight System

The insight system generates, tracks, and measures business recommendations:

- **AIInsight** — Core insight with type, priority, confidence scoring, and status lifecycle
- **AIInsightAction** — Clickable actions attached to insights (e.g., "Create Segment", "Launch Campaign")
- **AIInsightOutcome** — Measures predicted vs actual impact after action is taken
- **AIInsightFeedback** — User feedback (USEFUL / NOT_USEFUL) for insight quality tracking
- **AIInsightCorrelation** — Links related insights and identifies root causes
- **AIExecutiveScore** — Overall business health score with component breakdowns

### Utility Models

- **ProcessingFailure** — Audit log for failed background jobs (queue name, job ID, diagnostics)
- **CustomerLoginLog** — Customer authentication audit trail
- **AdminLoginLog** — Admin/manager authentication audit trail
- **RefreshToken** — Refresh token storage with expiry
- **EmailVerification** — Email verification tokens with expiry

## Enums

| Enum | Values |
|------|--------|
| `UserRole` | ADMIN, MANAGER |
| `ApprovalStatus` | PENDING, APPROVED, REJECTED |
| `ChannelType` | WHATSAPP, SMS, EMAIL, RCS |
| `CampaignStatus` | DRAFT, QUEUED, RUNNING, PAUSED, COMPLETED, FAILED |
| `CampaignEventType` | CampaignCreated, CampaignLaunched, MessageQueued, MessageSent, MessageDelivered, MessageOpened, MessageClicked, MessageConverted, MessageFailed |
| `DeliveryStatus` | QUEUED, SENT, DELIVERED, OPENED, CLICKED, CONVERTED, FAILED |
| `MessageRole` | USER, ASSISTANT |
| `ToolExecutionStatus` | STARTED, PENDING_CONFIRMATION, COMPLETED, CANCELED, FAILED |
| `InsightType` | REVENUE, CUSTOMER, CAMPAIGN, SEGMENT, CHURN, DELIVERY, CONVERSION, OPPORTUNITY, ANOMALY, PREDICTION |
| `InsightPriority` | LOW, MEDIUM, HIGH, CRITICAL |
| `InsightStatus` | ACTIVE, DISMISSED, COMPLETED, EXPIRED, ACTIONED |

## Migrations

Migrations are managed via Prisma and stored in `Backend/crm/prisma/migrations/`. Key migrations:

| Migration | Description |
|-----------|-------------|
| `20260610160000_initial` | Core schema (users, customers, segments, campaigns, events) |
| `20260611120000_cascade_fix_and_ai_auth` | Cascade delete fixes, AI auth integration |
| `20260611123114_add_login_logs` | Customer and admin login audit logs |
| `20260611130000_add_refresh_tokens` | Refresh token storage |
| `20260611140000_add_manager_role` | MANAGER role and approval workflow |
| `20260611150000_add_email_verification_and_tags` | Email verification, customer tags (GIN index) |
| `20260611160000_ai_tool_calling_expansion` | Expanded AI tool execution tracking |
| `20260611170000_ai_orchestration_audit` | AI decision logging, guardrail breach tracking |

## Running Migrations

```bash
cd Backend/crm

# Generate Prisma client
npm run prisma:generate

# Apply migrations in production
npm run prisma:migrate

# Create a new migration in development
npm run prisma:dev
```

## Seeding

```bash
cd Backend/crm
npm run seed
```

The seed script (`prisma/seed.ts`) creates:
- An admin user with credentials from `SEED_ADMIN_EMAIL` / `SEED_ADMIN_PASSWORD`
- Sample customers with orders
- Sample segments with rule definitions
- Sample campaigns with delivery events and analytics
- AI conversation history with tool executions
