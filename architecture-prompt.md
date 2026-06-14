# Architecture Diagram Generation Prompts

> Use these prompts with ChatGPT (DALL-E), Gemini, or any AI image generator.
> Pick the style that suits your presentation.

---

## Prompt 1 — Clean Professional (Recommended for Demo)

```
Generate a clean, modern software architecture diagram for "Xeno: AI-Native B2C Marketing CRM" on a dark navy (#0f172a) background with a subtle grid pattern.

The diagram should show THREE horizontal layers connected by arrows:

--- TOP LAYER (label: "Frontend — Port 5173") ---
A rounded rectangle box in indigo (#6366f1) containing:
- "React + TanStack Start" 
- "React Query + Recharts"
- "AI Copilot Panel"
- "Real-Time SSE Dashboard"
Show a small browser icon on the left.

--- MIDDLE LAYER (label: "Backend Services") ---
Two side-by-side boxes:

LEFT BOX (emerald green #10b981, label: "CRM API — Port 3000"):
- "NestJS + Prisma ORM"
- "Auth (JWT + Refresh Tokens)"
- "Customers → Segments → Campaigns"
- "Campaign Events + Analytics"
- "AI Orchestrator (9 tools)"
- "Webhook Receiver (HMAC-SHA256)"
Show a small server icon.

RIGHT BOX (amber #f59e0b, label: "Channel Simulator — Port 3001"):
- "NestJS Service"
- "Dispatch Handler"
- "Delivery Lifecycle Simulation"
- "HMAC-Signed Callbacks"
- "Deterministic Outcome Scoring"
Show a small send/paper-plane icon.

A双向 arrow between the two boxes labeled "REST + HMAC Webhooks".

--- BOTTOM LAYER (label: "Data & AI Layer") ---
Two boxes side-by-side:

LEFT BOX (purple #8b5cf6, label: "PostgreSQL Database"):
Tables listed inside:
- "Customers (GIN indexed tags)"
- "Segments (JSON rule engine)"
- "Campaigns + CampaignEvents"
- "CampaignLog (per-customer status)"
- "AI Conversations + Tool Executions"
- "ProcessingFailure (audit log)"
Show a database cylinder icon.

RIGHT BOX (rose #f43f5e, label: "AI Provider"):
- "Anthropic Claude"
- "Tool Calling Pipeline"
- "Grounding Verification"
- "Injection Detection"
Show a brain/chip icon.

--- ARROWS / DATA FLOW ---
- Arrow from Frontend down to CRM API labeled "REST API + SSE"
- Arrow from CRM API to Channel Simulator labeled "Dispatch Jobs"
- Arrow from Channel Simulator back to CRM API labeled "Webhook Callbacks"
- Arrow from CRM API down to PostgreSQL labeled "Prisma ORM"
- Arrow from CRM API down to AI Provider labeled "Tool-Augmented LLM"
- Arrow from Frontend to AI Provider labeled "Streaming Copilot"

--- HEADER ---
Top center: "XENO — AI-Native B2C Marketing CRM"
Bottom center: "Customers → Segments → Campaigns → Delivery → Analytics"

Style: Flat design, no 3D, no shadows. Use clean sans-serif font (Inter or similar). Rounded corners on all boxes. Subtle drop shadows on boxes. Color-coded by layer. Professional infographic style suitable for a technical presentation. High resolution, 16:9 aspect ratio.
```

---

## Prompt 2 — Data Flow Focused (for Architecture Section)

```
Create a modern, clean data flow architecture diagram on a white background.

Title at top: "Xeno CRM — Data Flow Architecture"

Show a LEFT-TO-RIGHT pipeline with 5 stages connected by arrows:

STAGE 1 (Blue box): "CUSTOMERS"
- Icon: person/group
- Subtext: "Name, Email, Phone, Tags, Metadata"
- Subtext: "GIN-indexed tag arrays"

STAGE 2 (Green box): "SEGMENTS"  
- Icon: filter/funnel
- Subtext: "JSON Rule Engine"
- Subtext: "AND/OR groups, 3 levels deep"
- Subtext: "Fields: totalSpent, orderCount, city, daysSinceLastOrder"

STAGE 3 (Purple box): "CAMPAIGNS"
- Icon: megaphone
- Subtext: "Channel: WhatsApp | SMS | Email | RCS"
- Subtext: "Lifecycle: DRAFT → QUEUED → RUNNING → COMPLETED"

STAGE 4 (Orange box): "DELIVERY EVENTS"
- Icon: lightning bolt
- Subtext: "Immutable event log"
- Subtext: "Sent → Delivered → Opened → Clicked → Converted"
- Subtext: "HMAC-signed webhooks"

STAGE 5 (Red box): "ANALYTICS"
- Icon: chart/graph
- Subtext: "Real-time SSE streaming"
- Subtext: "Funnel rates, revenue attribution"
- Subtext: "AI-powered insights"

Below the pipeline, show a BANNER:
"AI Copilot spans all stages — natural language → tool execution → grounded response"

Style: Flat design, colorful but professional. Each stage box has a distinct color. Arrows are thick and labeled. Clean sans-serif font. 16:9 aspect ratio. Suitable for a technical presentation slide.
```

---

## Prompt 3 — Service Communication Diagram (Technical)

```
Generate a technical service communication diagram on a dark (#1e293b) background.

Show these components as boxes with connection lines:

[React Frontend :5173]
    │
    ├── REST API ──────────→ [CRM API :3000 (NestJS)]
    │                              │
    ├── SSE Stream ←───────────────┤
    │                              │
    └── AI Copilot ───────────────→│
                                   │
                    ┌──────────────┤
                    │              │
                    ▼              ▼
            [PostgreSQL]    [Anthropic Claude]
            (Supabase)      (Xiaomi Gateway)
                    ▲
                    │
                    │
            [Channel Simulator :3001 (NestJS)]
                    │
                    └── HMAC Webhook → [CRM API :3000]

Label each connection:
- Frontend → CRM: "REST (JWT auth, httpOnly cookies)"
- CRM → Channel: "POST /api/dispatch (campaign jobs)"
- Channel → CRM: "POST /api/v1/webhooks/channel (HMAC-SHA256 signed)"
- CRM → PostgreSQL: "Prisma ORM (migrations, indexes)"
- CRM → Claude: "Tool-augmented conversations (9 tools)"
- Frontend → Claude: "SSE streaming copilot"
- Channel → PostgreSQL: "No direct access (isolated)"

Add a legend box in the bottom-right:
- Solid arrow = synchronous request
- Dashed arrow = async/callback
- Color code: Green=Frontend, Blue=CRM, Orange=Channel, Purple=DB, Red=AI

Style: Dark mode, monospace labels, technical/SRE style. Clean lines with rounded corners. 16:9 aspect ratio.
```

---

## Prompt 4 — AI Pipeline Diagram (for AI Copilot Section)

```
Create a vertical flowchart showing the AI Copilot pipeline in Xeno CRM.

Title: "Xeno AI Copilot — Tool-Augmented Pipeline"

Show 6 steps as connected boxes flowing top to bottom:

STEP 1 (Input): "User Message"
- Example: "Create a segment for high-value Mumbai customers"
- Icon: chat bubble

STEP 2 (Sanitize): "Input Sanitization"
- Remove control characters
- Truncate to 4000 chars
- Detect injection patterns (ignore previous instructions, etc.)
- Icon: shield

STEP 3 (History): "Context Assembly"
- Load conversation history (last 40 messages)
- System prompt with tool routing rules
- Icon: stack/layers

STEP 4 (LLM): "Anthropic Claude"
- Intent classification
- Tool selection from 9 available tools
- Icon: brain

STEP 5 (Execute): "Tool Execution Fan-Out"
Parallel execution of validated tools:
- getDashboardMetrics
- getCampaignPerformance  
- generateSegmentRules
- createSegment
- createCampaign
- diagnoseCampaignFailure
- recommendAudience
- getCustomerStats
- getCampaignAnalytics
- Icon: lightning/gears

STEP 6 (Verify): "Grounding Verification"
- Check every claim against tool output
- Replace ungrounded claims with fallback
- Persist to AIConversation + AIMessage
- Icon: checkmark/shield

Show a side panel labeled "Guardrails":
- Max 8 rounds per conversation
- Max 15 tool calls
- 90-second timeout
- Token budget per query
- Confirmation required for destructive ops
- Icon: lock

Style: Vertical flowchart, clean flat design. Each step is a rounded rectangle with an icon on the left. Use a gradient from blue (input) to green (output). Professional infographic style. 9:16 or 16:9 aspect ratio.
```

---

## Prompt 5 — Simple Overview (Non-Technical Audience)

```
Create a simple, colorful architecture overview diagram for a CRM platform called "Xeno".

Show 4 main blocks arranged in a 2x2 grid:

TOP-LEFT (Blue, "📱 Frontend"):
- React web app
- Dashboard with charts
- AI chat assistant
- Customer management

TOP-RIGHT (Green, "⚙️ Backend API"):
- Handles all business logic
- Manages customers, segments, campaigns
- Secure authentication
- Real-time updates

BOTTOM-LEFT (Orange, "📨 Message Delivery"):
- Sends WhatsApp, SMS, Email
- Tracks delivery status
- Handles retries
- Simulates real provider behavior

BOTTOM-RIGHT (Purple, "💾 Database"):
- Stores all customer data
- Campaign history
- Analytics and metrics
- AI conversation logs

Center connecting all 4 blocks: A large "🤖 AI Brain" icon with label "AI Copilot — understands natural language and automates CRM operations"

Arrows connecting all blocks to the center AI brain.

Style: Friendly, colorful, rounded shapes, large icons, minimal text. Suitable for a non-technical audience or intro slide. Clean white background. 16:9 aspect ratio.
```

---

## Which Prompt to Use

| Prompt | Best For | Style |
|--------|----------|-------|
| **Prompt 1** | Main architecture slide in your demo | Professional, detailed, dark mode |
| **Prompt 2** | Explaining the data pipeline | Colorful left-to-right flow |
| **Prompt 3** | Technical deep-dive | SRE/DevOps style, dark mode |
| **Prompt 4** | AI copilot section | Vertical flowchart |
| **Prompt 5** | Non-technical intro | Simple, friendly, colorful |

**Recommendation:** Use **Prompt 1** for your main architecture overview slide, and **Prompt 4** when you get to the AI copilot section. Together they'll cover the full 0:40–1:30 architecture section of your speech.
