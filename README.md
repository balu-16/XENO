<div align="center">

# ✨ Xeno Mini

### AI-Native B2C Marketing CRM

<p>
  <strong>Customers → Segments → Campaigns → Delivery Events → Analytics</strong>
</p>

<br/>

[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue?logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)](https://react.dev/)
[![NestJS](https://img.shields.io/badge/NestJS-11-E0234E?logo=nestjs&logoColor=white)](https://nestjs.com/)
[![Prisma](https://img.shields.io/badge/Prisma-6.19-2D3748?logo=prisma&logoColor=white)](https://www.prisma.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

<br/>

<p>
  <a href="#-features">Features</a> •
  <a href="#%EF%B8%8F-tech-stack">Tech Stack</a> •
  <a href="#-architecture">Architecture</a> •
  <a href="#-quick-start">Quick Start</a> •
  <a href="#-api-reference">API Reference</a> •
  <a href="#-deployment">Deployment</a>
</p>

</div>

---

## 🚀 Overview

**Xeno Mini** is a full-stack, AI-native marketing CRM platform built for modern B2C businesses. It combines powerful campaign management with an intelligent AI copilot that understands your data and helps you make smarter marketing decisions.

### 🎯 What Makes Xeno Mini Special?

- 🤖 **AI-Powered Copilot** — Ask questions in natural language and get grounded, tool-verified insights
- 🎯 **Smart Segmentation** — Build audiences using natural language or a visual rule builder
- 📊 **Real-Time Analytics** — Live dashboard with SSE-powered updates and funnel visualization
- 🔄 **Multi-Channel Delivery** — Email, SMS, WhatsApp, and RCS campaign support
- 🔐 **Enterprise Security** — JWT auth, role-based access, HMAC webhooks, and AI safety guardrails
- 📱 **Responsive Design** — Beautiful UI that works on desktop and mobile

---

## ✨ Features

### 📊 Dashboard
- 📈 Real-time KPI cards (customers, orders, revenue, active campaigns)
- 📉 Campaign performance trends with area charts
- 💰 Revenue attribution analytics
- 🔄 Channel performance comparison (pie chart)
- 🎯 Segment performance visualization
- 📋 Recent activity feed with live SSE updates

### 👥 Customers
- 🔍 Searchable customer profiles with pagination
- 🏷️ Tag-based filtering and categorization
- 💎 Lifetime value tracking and order history
- 📤 CSV export functionality
- 👤 Detailed customer dialog with full history

### 🎯 Segments
- 🤖 AI-powered segment generation from natural language
- 📝 Visual rule builder with AND/OR logic groups
- 👁️ Real-time audience size preview
- 🔢 Support for fields: `totalSpent`, `orderCount`, `daysSinceLastOrder`, `city`, `emailEngagement`
- ✏️ Inline segment renaming

### 📣 Campaigns
- 📧 Multi-channel support: Email, SMS, WhatsApp, RCS
- 🤖 AI message generation
- 🔄 Full lifecycle: `DRAFT → QUEUED → RUNNING → COMPLETED/FAILED`
- 📊 Real-time delivery tracking with SSE updates
- 📤 CSV export for campaign data
- 🔍 Search and filter by status

### 📈 Analytics
- 📊 Global delivery, open, click, and conversion rates
- 🔀 Lifecycle funnel visualization (Sent → Delivered → Opened → Clicked → Converted)
- 📋 Campaign comparison table
- 💰 Revenue attribution per campaign
- 📤 Data export capabilities

### 🤖 AI Copilot
- 💬 Persistent conversation history
- 🔧 9+ integrated tools for CRM operations
- 🛡️ Grounding verification — every claim traces to tool output
- 📚 Conversation management (rename, delete)
- 💡 Skill suggestions for campaigns, customers, segments, and analytics
- 🔍 Tool execution visibility and source tracking

### 💡 AI Insights
- 🧠 Proactive business intelligence generation
- 📊 Insight types: Revenue, Customer, Campaign, Segment, Churn, Delivery, Conversion, Opportunity, Anomaly, Prediction
- 🎯 Priority levels: Critical, High, Medium, Low
- 📈 Confidence and impact scoring
- ✅ Action management (dismiss, complete)
- 🔄 Auto-refresh and manual regeneration

### 👨‍💼 Managers (Admin)
- ✅ Manager approval workflow (Pending → Approved/Rejected)
- 👥 All managers overview with status badges
- 🗑️ Manager deletion with confirmation
- 🔐 Role-based navigation (Admin sees extra pages)

---

## 🛠️ Tech Stack

<table>
  <tr>
    <th>Layer</th>
    <th>Technology</th>
    <th>Purpose</th>
  </tr>
  <tr>
    <td><strong>🎨 Frontend</strong></td>
    <td>React 19, TanStack Start, TanStack Router</td>
    <td>SSR-capable SPA with file-based routing</td>
  </tr>
  <tr>
    <td><strong>📦 State Management</strong></td>
    <td>TanStack Query (React Query)</td>
    <td>Server state, caching, real-time updates</td>
  </tr>
  <tr>
    <td><strong>🎨 UI Framework</strong></td>
    <td>Tailwind CSS 4, shadcn/ui, Radix UI</td>
    <td>Beautiful, accessible component library</td>
  </tr>
  <tr>
    <td><strong>📊 Charts</strong></td>
    <td>Recharts</td>
    <td>Interactive data visualizations</td>
  </tr>
  <tr>
    <td><strong>⚙️ Backend API</strong></td>
    <td>NestJS 11, Express</td>
    <td>Modular, enterprise-grade API framework</td>
  </tr>
  <tr>
    <td><strong>🗄️ ORM</strong></td>
    <td>Prisma 6.19</td>
    <td>Type-safe database access and migrations</td>
  </tr>
  <tr>
    <td><strong>🐘 Database</strong></td>
    <td>PostgreSQL 16 (Neon/Supabase)</td>
    <td>Primary data store with GIN indexes</td>
  </tr>
  <tr>
    <td><strong>📮 Queue</strong></td>
    <td>BullMQ (Upstash Redis)</td>
    <td>Background job processing</td>
  </tr>
  <tr>
    <td><strong>🤖 AI</strong></td>
    <td>Anthropic Claude (via Gateway)</td>
    <td>AI copilot and insights generation</td>
  </tr>
  <tr>
    <td><strong>🔐 Auth</strong></td>
    <td>JWT, Argon2, Cookie-based</td>
    <td>Secure authentication and authorization</td>
  </tr>
  <tr>
    <td><strong>🐳 Containerization</strong></td>
    <td>Docker, Docker Compose</td>
    <td>Consistent development and deployment</td>
  </tr>
  <tr>
    <td><strong>🔄 CI/CD</strong></td>
    <td>GitHub Actions</td>
    <td>Automated testing and deployment</td>
  </tr>
  <tr>
    <td><strong>☁️ Hosting</strong></td>
    <td>Vercel (Serverless)</td>
    <td>Edge-optimized deployment</td>
  </tr>
</table>

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    🌐 Frontend (Port 5173)                    │
│              React + TanStack Start + Recharts                │
│              AI Copilot Panel + SSE Dashboard                 │
└──────────────────────┬──────────────────────────────────────┘
                       │ REST API + SSE
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    ⚙️ CRM API (Port 3000)                     │
│              NestJS + Prisma ORM + BullMQ                    │
│              Auth · Customers · Segments · Campaigns          │
│              🤖 AI Orchestrator (9+ tools) · Webhook Receiver │
└──────┬──────────────────┬──────────────────┬────────────────┘
       │                  │                  │
       ▼                  ▼                  ▼
┌──────────────┐  ┌───────────────┐  ┌──────────────────────┐
│  🐘 PostgreSQL │  │  📮 Upstash   │  │  📡 Channel          │
│  (Neon/       │  │  Redis (BullMQ)│  │  Simulator (Port 3001)│
│   Supabase)   │  │               │  │  NestJS + HMAC       │
└──────────────┘  └───────────────┘  └──────────┬───────────┘
                                                 │
                                          🔐 HMAC Webhooks
                                                 │
                                                 ▼
                                    CRM API Webhook Receiver
```

### 🔄 Data Flow

1. **👤 User** creates a campaign targeting a segment
2. **⚙️ CRM API** validates and queues the campaign
3. **📮 BullMQ** dispatches jobs to the Channel Simulator
4. **📡 Channel Simulator** simulates delivery with deterministic outcomes
5. **🔐 HMAC-signed webhooks** report status back to CRM
6. **👷 Workers** process receipts and refresh analytics
7. **📊 Dashboard** updates in real-time via SSE

---

## 📁 Project Structure

```
📦 Xeno/
├── 🎨 Frontend/                    # React SPA (TanStack Start)
│   ├── 📂 src/
│   │   ├── 🧩 components/          # UI components (shadcn/ui + custom)
│   │   │   ├── ui/                 # Base UI primitives
│   │   │   ├── AppShell.tsx        # Main layout with sidebar
│   │   │   ├── AIPanel.tsx         # Persistent AI copilot
│   │   │   └── ...                 # Feature components
│   │   ├── 🛤️ routes/              # File-based routing
│   │   │   ├── _app.dashboard.tsx  # Dashboard page
│   │   │   ├── _app.customers.tsx  # Customers page
│   │   │   ├── _app.segments.tsx   # Segments page
│   │   │   ├── _app.campaigns.tsx  # Campaigns page
│   │   │   ├── _app.analytics.tsx  # Analytics page
│   │   │   ├── _app.ai.tsx         # AI Copilot page
│   │   │   ├── _app.insights.tsx   # AI Insights page
│   │   │   ├── _app.managers.tsx   # Manager management
│   │   │   └── auth.tsx            # Login / Registration
│   │   ├── 🪝 hooks/               # Custom React hooks
│   │   └── 📚 lib/                 # Utilities, API client, contracts
│   ├── 🐳 Dockerfile
│   └── 📦 package.json
│
├── ⚙️ Backend/
│   ├── 🔧 crm/                     # Main CRM API (NestJS)
│   │   ├── 📂 src/
│   │   │   ├── 🤖 ai/              # AI copilot orchestration
│   │   │   ├── 📊 analytics/       # Analytics engine
│   │   │   ├── 🔐 auth/            # JWT authentication
│   │   │   ├── 📣 campaigns/       # Campaign management
│   │   │   ├── 👥 customers/       # Customer CRUD
│   │   │   ├── 🎯 segments/        # Segment rule engine
│   │   │   ├── 📡 webhooks/        # HMAC webhook receiver
│   │   │   ├── 💡 ai-insights/     # Proactive insights
│   │   │   ├── 📈 monitor/         # Monitoring endpoints
│   │   │   └── 🔧 dev/             # Development seed data
│   │   ├── 📂 prisma/
│   │   │   ├── schema.prisma       # Database schema (20+ models)
│   │   │   └── seed.ts             # Seed data script
│   │   ├── 🐳 Dockerfile
│   │   └── 📦 package.json
│   │
│   └── 📡 channel/                 # Channel Simulator Service
│       ├── 📂 src/
│       │   ├── dispatch.controller.ts
│       │   ├── channel-simulator.service.ts
│       │   └── health.controller.ts
│       ├── 🐳 Dockerfile
│       └── 📦 package.json
│
├── 📚 docs/                        # Comprehensive documentation
│   ├── architecture.md
│   ├── api-reference.md
│   ├── database.md
│   ├── ai-copilot.md
│   ├── frontend.md
│   └── deployment.md
│
├── 🐳 docker-compose.yml           # Local Docker setup
├── 🔄 .github/workflows/ci.yml     # CI pipeline
└── 📄 README.md                    # This file
```

---

## 🚀 Quick Start

### 📋 Prerequisites

- 📦 **Node.js** >= 22
- 🐘 **PostgreSQL** database (Neon, Supabase, or local)
- 📮 **Redis** instance (Upstash or local)
- 🤖 **Anthropic API** access (optional, for AI features)

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/your-username/xeno-mini.git
cd xeno-mini
```

### 2️⃣ Configure Environment

```bash
# Frontend
cp Frontend/.env.example Frontend/.env

# Backend CRM
cp Backend/crm/.env.example Backend/crm/.env

# Edit .env files with your credentials:
# - DATABASE_URL (Neon/Supabase PostgreSQL)
# - DIRECT_URL (Direct connection for migrations)
# - JWT_SECRET (Random secure string)
# - CHANNEL_WEBHOOK_SECRET (Shared secret)
# - ANTHROPIC_BASE_URL (AI gateway)
# - XIAOMI_AUTH_TOKEN (AI auth token)
```

### 3️⃣ Install Dependencies

```bash
# Frontend
cd Frontend && npm install

# Backend CRM
cd ../Backend/crm && npm install

# Channel Simulator
cd ../channel && npm install
```

### 4️⃣ Setup Database

```bash
cd Backend/crm

# Generate Prisma client
npm run prisma:generate

# Run migrations
npm run prisma:migrate

# Seed initial data
npm run seed
```

### 5️⃣ Start Development Servers

Open **4 separate terminals**:

```bash
# Terminal 1: CRM API
cd Backend/crm && npm run dev

# Terminal 2: CRM Workers
cd Backend/crm && npm run worker:dev

# Terminal 3: Channel Simulator
cd Backend/channel && npm run dev

# Terminal 4: Frontend
cd Frontend && npm run dev
```

### 6️⃣ Access the Application

| Service | URL |
|---------|-----|
| 🌐 Frontend | http://localhost:5173 |
| ⚙️ CRM API | http://localhost:3000/api/v1 |
| 📚 API Docs | http://localhost:3000/api/v1/docs |
| 📡 Channel | http://localhost:3001 |

### 🔑 Default Login

```
📧 Email:    admin@xeno.local
🔑 Password: XenoDemo123!
```

---

## 🐳 Docker Setup

### Local Development with Docker

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

### Docker Services

| Service | Port | Description |
|---------|------|-------------|
| `postgres` | 5432 | PostgreSQL 16 database |
| `crm-api` | 3000 | CRM REST API |
| `channel` | 3001 | Channel simulator |
| `frontend` | 5173 | React application |

---

## 📚 API Reference

### 🔐 Authentication

```http
POST /api/v1/auth/login
POST /api/v1/auth/register
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET  /api/v1/auth/me
```

### 👥 Customers

```http
GET    /api/v1/customers?page=1&search=john&tag=vip
GET    /api/v1/customers/:id
POST   /api/v1/customers
PUT    /api/v1/customers/:id
DELETE /api/v1/customers/:id
GET    /api/v1/customers/tags
```

### 🎯 Segments

```http
GET    /api/v1/segments
GET    /api/v1/segments/:id
POST   /api/v1/segments
PUT    /api/v1/segments/:id
POST   /api/v1/segments/preview
```

### 📣 Campaigns

```http
GET    /api/v1/campaigns?page=1&search=summer&status=RUNNING
GET    /api/v1/campaigns/:id
POST   /api/v1/campaigns
POST   /api/v1/campaigns/:id/launch
DELETE /api/v1/campaigns/:id
```

### 📊 Analytics

```http
GET /api/v1/analytics/dashboard
GET /api/v1/analytics/campaigns
GET /api/v1/analytics/stream    # SSE endpoint
```

### 🤖 AI Copilot

```http
GET    /api/v1/ai/conversations
POST   /api/v1/ai/conversations
GET    /api/v1/ai/conversations/:id
POST   /api/v1/ai/conversations/:id/messages
PATCH  /api/v1/ai/conversations/:id
DELETE /api/v1/ai/conversations/:id
```

### 💡 AI Insights

```http
GET    /api/v1/insights?type=REVENUE&priority=HIGH&status=ACTIVE
GET    /api/v1/insights/summary
GET    /api/v1/insights/status
POST   /api/v1/insights/refresh
PATCH  /api/v1/insights/:id/dismiss
PATCH  /api/v1/insights/:id/complete
```

### 👨‍💼 Managers (Admin Only)

```http
GET    /api/v1/managers/pending
GET    /api/v1/managers
POST   /api/v1/managers/:id/approve
POST   /api/v1/managers/:id/reject
DELETE /api/v1/managers/:id
```

### 📡 Webhooks

```http
POST /api/v1/webhooks/channel    # HMAC-signed delivery callbacks
```

---

## 🗄️ Database Schema

### Core Models

| Model | Description |
|-------|-------------|
| `User` | Admin and manager accounts with role-based access |
| `Customer` | Customer profiles with tags and metadata |
| `Order` | Customer orders with amount and items |
| `Segment` | Audience segments with JSON rule definitions |
| `Campaign` | Marketing campaigns across channels |
| `CampaignLog` | Per-customer delivery status tracking |
| `CampaignEvent` | Immutable event stream for analytics |
| `CampaignAnalytics` | Aggregated campaign metrics |
| `WebhookReceipt` | Channel callback processing |

### AI Models

| Model | Description |
|-------|-------------|
| `AIConversation` | Chat sessions with the copilot |
| `AIMessage` | Individual messages with grounding data |
| `AIToolExecution` | Tool calls with input/output tracking |
| `AIDecisionLog` | AI decision audit trail |
| `AIInsight` | Proactive business insights |
| `AIInsightAction` | Insight action tracking |
| `AIInsightOutcome` | Insight outcome measurement |

---

## 🧪 Quality Gates

Run these commands in each package directory:

```bash
# Type checking
npm run typecheck

# Linting
npm run lint

# Testing
npm run test

# Production build
npm run build
```

### CI Pipeline

The GitHub Actions CI pipeline runs on every push and PR:

- ✅ **Frontend**: Lint → Typecheck → Test → Build
- ✅ **Backend CRM**: Prisma Generate → Lint → Typecheck → Test → Build
- ✅ **Backend Channel**: Lint → Typecheck → Test → Build
- ✅ **All Passed Gate**: All three jobs must pass before merge

---

## 🚀 Deployment

### Vercel (Recommended)

The project is configured for Vercel deployment:

```bash
# Frontend
cd Frontend && vercel deploy

# Backend CRM
cd Backend/crm && vercel deploy

# Channel
cd Backend/channel && vercel deploy
```

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `DATABASE_URL` | PostgreSQL connection string (pooled) | ✅ |
| `DIRECT_URL` | Direct PostgreSQL for migrations | ✅ |
| `JWT_SECRET` | JWT signing secret | ✅ |
| `CHANNEL_WEBHOOK_SECRET` | HMAC webhook secret | ✅ |
| `ANTHROPIC_BASE_URL` | AI gateway URL | ✅ |
| `XIAOMI_AUTH_TOKEN` | AI authentication token | ✅ |
| `XIAOMI_MODEL` | AI model identifier | ✅ |
| `FRONTEND_URL` | Frontend URL for CORS | ✅ |
| `SEED_ADMIN_EMAIL` | Initial admin email | ✅ |
| `SEED_ADMIN_PASSWORD` | Initial admin password | ✅ |
| `SUPABASE_URL` | Supabase project URL | Optional |
| `SUPABASE_ANON_KEY` | Supabase anonymous key | Optional |

---

## 🤖 AI Copilot Pipeline

```
User Message
    │
    ▼
🧹 Input Sanitization (strip control chars, truncate to 4000 chars)
    │
    ▼
🛡️ Injection Detection (regex patterns for prompt injection)
    │
    ▼
📋 Context Assembly (last 40 messages + system prompt)
    │
    ▼
🧠 Anthropic Claude (intent classification + tool selection)
    │
    ▼
🔧 Tool Execution Fan-Out (parallel validated tools)
    │
    ▼
✅ Grounding Verification (every claim must trace to tool output)
    │
    ▼
📝 Response Generation (formatted markdown, never raw JSON)
    │
    ▼
💾 Conversation Persistence (AIConversation + AIMessage + AIToolExecution)
```

### Available AI Tools

| Tool | Description |
|------|-------------|
| `list_campaigns` | List and filter campaigns |
| `get_campaign` | Get campaign details and analytics |
| `create_campaign` | Create a new campaign |
| `list_customers` | Search and filter customers |
| `get_customer` | Get customer profile and history |
| `create_customer` | Create a new customer |
| `list_segments` | List audience segments |
| `create_segment` | Create a segment from rules |
| `get_analytics` | Get campaign analytics |
| `get_dashboard` | Get dashboard metrics |

---

## 📊 Campaign Delivery Simulation

The Channel Simulator uses deterministic SHA-256 scoring for realistic outcomes:

| Score Range | Outcome | Description |
|-------------|---------|-------------|
| 0–19 | ❌ Failed | Provider rejection or invalid destination |
| 20–94 | ✅ Delivered | Message successfully delivered |
| 95–154 | 👁️ Opened | Message opened on mobile or desktop |
| 155–219 | 🖱️ Clicked | Link clicked in message |
| 220–255 | 💰 Converted | Conversion with attributed order |

---

## 🔐 Security Features

- 🔑 **JWT Authentication** — HttpOnly cookies with refresh token rotation
- 👥 **Role-Based Access** — ADMIN and MANAGER roles with guard decorators
- 🔐 **HMAC Webhooks** — SHA-256 signed channel callbacks
- 🛡️ **AI Safety** — Input sanitization, injection detection, grounding verification
- 🚦 **Rate Limiting** — Throttler-based API rate limiting
- 🔒 **HTTP Security** — Helmet headers, CORS configuration
- 🚫 **Cache Prevention** — No-store headers for authenticated pages

---

## 📖 Documentation

| Document | Description |
|----------|-------------|
| [📐 Architecture](./docs/architecture.md) | System architecture and data flow |
| [📡 API Reference](./docs/api-reference.md) | Complete REST API documentation |
| [🗄️ Database Schema](./docs/database.md) | Prisma models and relationships |
| [🤖 AI Copilot](./docs/ai-copilot.md) | AI orchestration and tools |
| [🎨 Frontend](./docs/frontend.md) | Frontend architecture and components |
| [🚀 Deployment](./docs/deployment.md) | Deployment guides and configuration |

---

## 🧪 Evaluator Flow

Follow these steps to fully evaluate the application:

1. 📊 Open `/dashboard` and run the production seed
2. 📈 Inspect dashboard metrics and charts
3. 🎯 Generate and save a segment with natural language
4. 📣 Create a campaign through all five lifecycle stages
5. 🚀 Launch and observe the three BullMQ streams
6. 📡 Watch simulated channel callbacks and SSE funnel updates
7. 📊 Review global analytics
8. 🤖 Ask the copilot: `"Why did Summer Sale fail?"`

> 💡 The seeded **Summer Sale** campaign has a measurable delivery/open collapse and recorded destination failures, making the diagnosis fully tool-grounded.

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'feat: add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Commit Convention

```
feat:     ✨ New feature
fix:      🐛 Bug fix
docs:     📝 Documentation
style:    💄 Formatting
refactor: ♻️  Code refactoring
test:     ✅ Adding tests
chore:    🔧 Maintenance
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- [TanStack](https://tanstack.com/) for the amazing React ecosystem
- [NestJS](https://nestjs.com/) for the enterprise-grade backend framework
- [Prisma](https://www.prisma.io/) for type-safe database access
- [shadcn/ui](https://ui.shadcn.com/) for the beautiful component library
- [Anthropic](https://www.anthropic.com/) for the AI capabilities
- [Recharts](https://recharts.org/) for the chart library

---

<div align="center">

**Built with ❤️ by the Xeno Team**

[⬆ Back to Top](#-xeno-mini)

</div>
