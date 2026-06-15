# AI Copilot

The Xeno AI Copilot is a tool-augmented conversational assistant built into the CRM. It uses Anthropic Claude to understand natural language requests, execute CRM operations through authorized tools, and generate grounded responses backed by real data.

## Architecture

```
User Message
    │
    ▼
┌─────────────────────┐
│  Input Sanitization  │  Strip control chars, truncate to 4000 chars
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Injection Detection │  Regex patterns for prompt injection attempts
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Context Assembly    │  Last 40 messages + system prompt with tool routing
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Anthropic Claude    │  Intent classification + tool selection
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Tool Execution      │  Parallel fan-out of validated tools
│  Fan-Out             │  with retry for transient failures
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Grounding           │  Every claim must trace to tool output
│  Verification        │  Ungrounded claims replaced with fallback
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Response Generation │  Formatted markdown, never raw JSON
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Persistence         │  AIConversation + AIMessage + AIToolExecution
└─────────────────────┘
```

## Pipeline Stages

### 1. Input Sanitization

Every user message is sanitized before processing:

- **Control character removal:** Strips ASCII control characters (`\x00-\x08`, `\x0B`, `\x0C`, `\x0E-\x1F`)
- **Truncation:** Maximum 4000 characters
- **Trimming:** Leading/trailing whitespace removed

### 2. Injection Detection

Messages are checked against known prompt injection patterns:

```
- "ignore previous instructions"
- "you are now a ..."
- "system: ..."
- "forget everything"
- "override your ..."
- "disregard all ..."
- "new instructions:"
- "repeat the system prompt"
- "[INST]"
- "<|im_start|>"
```

If detected, the copilot responds with a safety message and refuses to process the request.

### 3. Context Assembly

The copilot assembles context from:

- **System prompt:** Contains tool routing rules, segment creation workflow, mandatory tool selection guide, and behavioral instructions
- **Conversation history:** Last 40 messages from the `AIMessage` table
- **User input:** Wrapped in `<user_input>` tags to prevent confusion with system instructions

### 4. LLM Processing

Anthropic Claude processes the assembled context and returns one of:

- **Text response:** Direct answer (for casual/conversational messages)
- **Tool calls:** One or more tool invocations (for operational CRM requests)

The LLM is instructed to **always call a tool for any question about CRM data** and never answer from memory.

### 5. Tool Execution

When the LLM requests tool calls:

1. **Validation:** Each tool input is validated against its Zod schema
2. **Role check:** User's role is verified against the tool's `allowedRoles`
3. **Confirmation check:** Destructive tools (delete, etc.) require user confirmation before execution
4. **Parallel execution:** Validated tools execute concurrently via `Promise.all`
5. **Retry logic:** Transient failures (timeouts, rate limits, 5xx) are retried once with 1-second backoff

### 6. Grounding Verification

After tool execution, the copilot verifies every claim in the response:

- **Claim extraction:** Numbers, dates, statuses, emails, phone numbers, revenue figures
- **Source matching:** Each claim must appear in at least one tool output
- **Fallback replacement:** If ungrounded claims are found, the entire response is replaced with a safe fallback that only contains verified tool data

### 7. Response Generation

Responses are formatted as clean markdown:

- **Lists:** Use markdown tables, never numbered lists (numbers can be mistaken for CRM facts)
- **Confirmations:** Use checkmark emojis (✅)
- **Errors:** Plain language explanations
- **Data:** Bold key values, tables for lists

## Available Tools

The copilot has access to 30+ tools organized by category:

### Customer Tools

| Tool | Description | Confirmation |
|------|-------------|--------------|
| `getCustomers` | List customers with pagination | No |
| `getCustomerById` | Get customer by ID | No |
| `getCustomerByEmail` | Find customer by email | No |
| `createCustomer` | Create a new customer | No |
| `updateCustomer` | Update customer fields | No |
| `deleteCustomer` | Delete a customer | **Yes** |
| `getCustomerStats` | Get customer count and stats | No |

### Segment Tools

| Tool | Description | Confirmation |
|------|-------------|--------------|
| `getSegments` | List all segments | No |
| `getSegment` | Get segment by ID | No |
| `createSegment` | Create a new segment | No |
| `updateSegment` | Update segment rules | No |
| `deleteSegment` | Delete a segment | **Yes** |
| `getSegmentCustomerCount` | Count customers matching segment | No |
| `generateSegmentRules` | Generate rules from natural language | No |

### Campaign Tools

| Tool | Description | Confirmation |
|------|-------------|--------------|
| `getCampaigns` | List campaigns (optional status filter) | No |
| `getCampaign` | Get campaign by ID | No |
| `createCampaign` | Create a new campaign | No |
| `launchCampaign` | Launch a draft campaign | No |
| `pauseCampaign` | Pause a running campaign | No |
| `deleteCampaign` | Delete a campaign | **Yes** |
| `retryCampaign` | Retry failed messages | No |
| `generateCampaignMessage` | Generate message text from description | No |

### Analytics Tools

| Tool | Description | Confirmation |
|------|-------------|--------------|
| `getDashboardMetrics` | Full dashboard KPIs | No |
| `getCampaignPerformance` | Campaign funnel and rates | No |
| `getRevenueAnalytics` | Revenue trends and attribution | No |
| `getDeliveryAnalytics` | Delivery rates across channels | No |
| `getSegmentAnalytics` | Segment performance comparison | No |

### AI Strategy Tools

| Tool | Description | Confirmation |
|------|-------------|--------------|
| `recommendAudience` | Recommend best segment for a goal | No |
| `diagnoseCampaignFailure` | Analyze why a campaign underperformed | No |
| `getBestSendTime` | Optimal send time analysis | No |
| `suggestABTest` | A/B test recommendations | No |
| `getInsights` | Get AI-generated business insights | No |

## Guardrails

The copilot operates within strict safety limits:

| Guardrail | Default | Env Variable |
|-----------|---------|--------------|
| Max rounds per message | 8 | `AI_MAX_ROUNDS` |
| Max tool calls per message | 8 | `AI_MAX_TOOL_CALLS` |
| Max execution time | 25s | `AI_MAX_EXECUTION_MS` |
| Max input tokens per call | 100,000 | `AI_MAX_INPUT_TOKENS` |
| Max total tokens per query | 150,000 | `AI_MAX_TOKENS_PER_QUERY` |
| Confirmation TTL | 15 min | `AI_CONFIRMATION_TTL_MS` |
| History context window | 40 messages | `AI_HISTORY_LIMIT` |
| Request log size | 200 entries | `AI_REQUEST_LOG_LIMIT` |
| Retry attempts | 1 | `AI_RETRY_MAX_ATTEMPTS` |
| Retry backoff | 1000ms | `AI_RETRY_BACKOFF_MS` |

Guardrail breaches are logged to the `ProcessingFailure` table with queue name `ai-guardrails`.

## Confirmation Flow

Destructive operations (deleteCustomer, deleteSegment, deleteCampaign) require explicit user confirmation:

```
1. LLM requests deleteSegment(id: "abc")
2. Copilot validates input, creates PENDING_CONFIRMATION execution
3. SSE event sent to frontend with confirmation UI
4. User clicks "Confirm" or "Cancel"
5. If confirmed: tool executes, LLM continues with result
6. If canceled: execution marked CANCELED, no data changed
7. If expired (15 min): execution auto-canceled
```

## Segment Creation Workflow

The copilot follows a specific workflow for segment creation:

```
User: "Create a segment for high-value Mumbai customers"

Step 1: Call generateSegmentRules(prompt: "high-value Mumbai customers")
        → Returns: { operator: "AND", conditions: [
            { field: "city", operator: "contains", value: "Mumbai" },
            { field: "totalSpent", operator: ">", value: 5000 }
          ]}

Step 2: IMMEDIATELY call createSegment(
          name: "High Value Mumbai Customers",
          rules: <result from step 1>
        )

Step 3: Report success to user
```

The copilot is instructed to **never stop after generateSegmentRules** without calling createSegment, and to **auto-generate a name** if the user didn't provide one.

## Tool Selection Guide

The system prompt includes a mandatory tool routing table:

| User Intent | Tool to Call |
|-------------|--------------|
| "What is our revenue?" | `getRevenueAnalytics` |
| "Show me the dashboard" | `getDashboardMetrics` |
| "How many customers?" | `getCustomerStats` |
| "List customers" | `getCustomers` |
| "Find customer by email X" | `getCustomerByEmail` |
| "Show campaigns" | `getCampaigns` |
| "Campaign performance" | `getCampaignAnalytics` |
| "Show segments" | `getSegments` |
| "Delivery stats" | `getDeliveryAnalytics` |
| "Why did campaign fail?" | `diagnoseCampaignFailure` |
| "Create segment for X" | `generateSegmentRules` → `createSegment` |
| "What opportunities exist?" | `getInsights` |
| "Critical issues" | `getInsights(priority: 'CRITICAL')` |

## Conversation Storage

Every conversation is fully persisted:

- **AIConversation** — Conversation metadata (user, title, timestamps)
- **AIMessage** — Each user and assistant message with grounding data
- **AIToolExecution** — Every tool call with input, output, duration, status
- **AIDecisionLog** — Audit trail for orchestration decisions

Grounding data attached to each assistant message includes:
```json
{
  "tool": "getCampaignAnalytics",
  "tools": ["getCampaignAnalytics", "getDeliveryAnalytics"],
  "sources": ["campaign-analytics", "delivery-stats"],
  "executionId": "uuid",
  "executionIds": ["uuid1", "uuid2"]
}
```

## Streaming

The copilot supports Server-Sent Events (SSE) for real-time updates:

```
event: tool-call
data: {"type":"tool-call","execution":{...}}

event: tool-result
data: {"type":"tool-result","execution":{...}}

event: final-response
data: {"type":"final-response","result":{...}}
```

## AI Provider Configuration

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | API key for Anthropic Claude |
| `ANTHROPIC_BASE_URL` | Custom gateway URL (e.g., Xiaomi proxy) |
| `ANTHROPIC_MODEL` | Model identifier (default: from provider) |

The provider service abstracts the Anthropic SDK, allowing custom gateway endpoints for proxying or cost optimization.
