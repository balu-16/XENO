# Xeno Documentation

Welcome to the Xeno project documentation. Xeno is an **AI-native B2C marketing CRM** that implements the full customer marketing lifecycle:

```
Customers → Segments → Campaigns → Delivery Events → Analytics
```

## Documentation Index

| Document | Description |
|----------|-------------|
| [Architecture](./architecture.md) | System architecture, service communication, and data flow |
| [API Reference](./api-reference.md) | REST API endpoints, request/response contracts, and authentication |
| [Database Schema](./database.md) | Prisma models, relationships, indexes, and data lifecycle |
| [AI Copilot](./ai-copilot.md) | AI orchestration pipeline, tools, guardrails, and grounding |
| [Frontend](./frontend.md) | Frontend architecture, routing, components, and state management |
| [Deployment](./deployment.md) | Docker setup, CI/CD pipeline, environment variables, and production deployment |

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | React 19, TanStack Start, TanStack Router, React Query, Recharts, Tailwind CSS 4, shadcn/ui |
| **CRM Backend** | NestJS 11, Prisma ORM, PostgreSQL (Neon/Supabase), BullMQ, Anthropic Claude |
| **Channel Service** | NestJS 11 (independently deployable) |
| **Infrastructure** | Docker, GitHub Actions CI, Vercel (serverless), Upstash Redis |

## Quick Start

### Prerequisites

- Node.js >= 22
- PostgreSQL database (Neon, Supabase, or local)
- Redis instance (Upstash or local)

### Installation

```bash
# Install all dependencies
cd Frontend && npm install
cd ../Backend/crm && npm install
cd ../channel && npm install
```

### Database Setup

```bash
cd Backend/crm
npm run prisma:generate
npm run prisma:migrate
npm run seed
```

### Running Locally

Start four processes in separate terminals:

```bash
# Terminal 1: CRM API
cd Backend/crm && npm run dev

# Terminal 2: CRM Workers (receipt processing, analytics)
cd Backend/crm && npm run worker:dev

# Terminal 3: Channel Simulator
cd Backend/channel && npm run dev

# Terminal 4: Frontend
cd Frontend && npm run dev
```

### Default Login

After seeding, log in with the credentials defined in your `.env`:
- **Email:** `SEED_ADMIN_EMAIL` (default: `admin@xeno.local`)
- **Password:** `SEED_ADMIN_PASSWORD`

## Project Structure

```
Xeno/
├── Frontend/              # React SPA (TanStack Start)
│   ├── src/
│   │   ├── components/    # UI components (shadcn/ui + custom)
│   │   ├── routes/        # File-based routing
│   │   ├── hooks/         # Custom React hooks
│   │   └── lib/           # Utilities, API client, contracts
│   └── package.json
├── Backend/
│   ├── crm/               # Main CRM API (NestJS)
│   │   ├── src/
│   │   │   ├── ai/        # AI copilot orchestration
│   │   │   ├── analytics/ # Analytics engine
│   │   │   ├── auth/      # JWT authentication
│   │   │   ├── campaigns/ # Campaign management
│   │   │   ├── customers/ # Customer CRUD
│   │   │   ├── segments/  # Segment rule engine
│   │   │   ├── workers/   # BullMQ background workers
│   │   │   └── dev/       # Development seed data
│   │   └── prisma/        # Database schema & migrations
│   └── channel/           # Channel simulator service
│       └── src/
├── docs/                  # This documentation
├── docker-compose.yml     # Local Docker setup
└── .github/workflows/     # CI pipeline
```

## Quality Gates

Run these commands in each package directory:

```bash
npm run typecheck    # TypeScript type checking
npm run lint         # ESLint
npm run test         # Vitest test suite
npm run build        # Production build
```

## Key Concepts

- **Segments** use a JSON rule engine with AND/OR groups, supporting fields like `totalSpent`, `orderCount`, `daysSinceLastOrder`, `city`, and `emailEngagement`.
- **Campaigns** follow a lifecycle: `DRAFT → QUEUED → RUNNING → COMPLETED/FAILED`.
- **Delivery events** are immutable and flow through: `Sent → Delivered → Opened → Clicked → Converted`.
- **The AI Copilot** uses tool-augmented LLM calls with grounding verification — every CRM claim must come from an authoritative tool result.
- **The Channel Simulator** generates deterministic delivery outcomes using SHA-256 hashing, simulating real-world message delivery patterns.
