# Xeno Mini

AI-native B2C marketing CRM implementing:

`Customers → Segments → Campaigns → Delivery Events → Analytics`

## Architecture

- `Frontend/`: TanStack Start, React Query, Recharts, persistent AI copilot.
- `Backend/crm/`: NestJS API plus receipt and analytics workers.
- `Backend/channel/`: independently deployable BullMQ channel simulator.
- Shared Zod schemas and TypeScript interfaces are copied into each app.
- Neon Postgres stores CRM data, immutable campaign events, projections, and AI history.
- Upstash native Redis carries `campaign-dispatch`, `receipt-processing`, and `analytics-refresh`.

AI requests follow:

`Intent Classification → Tool Selection → Tool Execution → Grounding Verification → Response Generation → Conversation Storage`

## Local Setup

1. Copy `Frontend/.env.example` to `Frontend/.env`, copy `Backend/.env.example` to `Backend/.env`, and supply a pooled Neon URL, direct Neon URL, native Upstash Redis credentials, JWT secret, webhook secret, and optional Anthropic-compatible gateway.
2. Rotate any credentials that have previously been pasted into source material or chat.
3. Install dependencies:

```bash
cd Frontend && npm install
cd ../Backend/crm && npm install
cd ../channel && npm install
```

4. Generate Prisma, migrate, and seed from `Backend/crm`:

```bash
npm run prisma:generate
npm run prisma:migrate
npm run seed
```

5. Start the four processes in separate terminals:

```bash
cd Backend/crm && npm run dev
cd Backend/crm && npm run worker:dev
cd Backend/channel && npm run dev
cd Frontend && npm run dev
```

Seeded login: values from `SEED_ADMIN_EMAIL` and `SEED_ADMIN_PASSWORD`.

## Evaluator Flow

1. Open `/monitor` and run the heavy production seed.
2. Inspect dashboard metrics.
3. Generate and save a segment with natural language.
4. Create a campaign through all five required stages.
5. Launch and observe the three BullMQ streams.
6. Watch simulated channel callbacks and SSE funnel updates.
7. Review global analytics.
8. Ask the copilot: `Why did Summer Sale fail?`

The seeded Summer Sale has a measurable delivery/open collapse and recorded destination failures, so the diagnosis is fully tool-grounded.

## Quality Gates

```bash
npm run typecheck
npm run lint
npm run test
npm run build
```

Run those commands inside `Frontend/`, `Backend/crm/`, and `Backend/channel/` as applicable.

The development seed endpoint is disabled outside development and requires an authenticated admin.
