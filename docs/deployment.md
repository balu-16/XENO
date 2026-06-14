# Deployment

This document covers Docker configuration, CI/CD pipeline, environment variables, and production deployment for Xeno.

## Docker

### Docker Compose (Local Development)

The `docker-compose.yml` at the project root provides a complete local development environment:

```bash
docker-compose up -d
```

**Services:**

| Service | Port | Description |
|---------|------|-------------|
| `postgres` | 5432 | PostgreSQL 16 Alpine with health checks |
| `crm-api` | 3000 | CRM API (NestJS) |
| `channel` | 3001 | Channel Simulator (NestJS) |
| `frontend` | 5173 | Frontend (Vite dev server) |

**Volumes:**
- `pgdata` — Persistent PostgreSQL data

**Networks:**
- `xeno-net` — Bridge network connecting all services

### Dockerfiles

Each service has its own multi-stage Dockerfile:

**Frontend (`Frontend/Dockerfile`):**
```dockerfile
# Stage 1: Build
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:22-alpine AS production
WORKDIR /app
COPY --from=build /app ./
EXPOSE 5173
CMD ["npm", "run", "preview"]
```

**CRM API (`Backend/crm/Dockerfile`):**
```dockerfile
# Stage 1: Build
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate && npm run build

# Stage 2: API
FROM node:22-alpine AS api
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/prisma ./prisma
EXPOSE 3000
CMD ["node", "dist/src/main.js"]

# Stage 3: Worker
FROM node:22-alpine AS worker
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["node", "dist/src/worker.js"]
```

**Channel Simulator (`Backend/channel/Dockerfile`):**
```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3001
CMD ["node", "dist/src/main.js"]
```

### Running with Docker

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f crm-api
docker-compose logs -f channel
docker-compose logs -f frontend

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

## CI/CD Pipeline

The GitHub Actions CI pipeline (`.github/workflows/ci.yml`) runs on every push and PR to `main`.

### Pipeline Structure

```
┌─────────────┐  ┌──────────────┐  ┌─────────────────┐
│  Frontend   │  │ Backend CRM  │  │ Backend Channel │
│             │  │              │  │                 │
│  • Lint     │  │ • Prisma Gen │  │ • Lint          │
│  • Typecheck│  │ • Lint       │  │ • Typecheck     │
│  • Test     │  │ • Typecheck  │  │ • Test          │
│  • Build    │  │ • Test       │  │ • Build         │
│             │  │ • Build      │  │                 │
└──────┬──────┘  └──────┬───────┘  └────────┬────────┘
       │                │                    │
       └────────────────┼────────────────────┘
                        ▼
              ┌──────────────────┐
              │   All Passed     │
              │   (Gate check)   │
              └──────────────────┘
```

### Jobs

**Frontend:**
1. Checkout code
2. Setup Node.js 22
3. Cache `node_modules`
4. `npm ci`
5. `npm run lint`
6. `npm run typecheck`
7. `npm run test`
8. `npm run build`

**Backend CRM:**
1. Checkout code
2. Setup Node.js 22
3. Cache `node_modules`
4. `npm ci`
5. `npx prisma generate`
6. `npm run lint`
7. `npm run typecheck`
8. `npm run test`
9. `npm run build`

**Backend Channel:**
1. Checkout code
2. Setup Node.js 22
3. Cache `node_modules`
4. `npm ci`
5. `npm run lint`
6. `npm run typecheck`
7. `npm run test`
8. `npm run build`

**All Passed (Gate):**
- Runs after all three jobs complete
- Fails if any required job failed
- Required for merge to `main`

### CI Environment Variables

The CRM backend CI job uses dummy environment variables for testing:

```yaml
DATABASE_URL: postgresql://user:password@localhost:5432/xeno_ci
DIRECT_URL: postgresql://user:password@localhost:5432/xeno_ci
JWT_SECRET: ci-dummy-jwt-secret-for-testing-only
SEED_ADMIN_PASSWORD: ci-dummy-admin-password
CHANNEL_WEBHOOK_SECRET: ci-dummy-webhook-secret
ANTHROPIC_BASE_URL: https://api.example.com/anthropic
```

### Concurrency

The pipeline uses concurrency groups to cancel in-progress runs:

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

## Environment Variables

### CRM API (`Backend/crm/.env`)

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | Yes | Pooled PostgreSQL connection string | `postgresql://user:pass@host:6543/db?pgbouncer=true` |
| `DIRECT_URL` | Yes | Direct PostgreSQL connection (for migrations) | `postgresql://user:pass@host:5432/db` |
| `JWT_SECRET` | Yes | Secret for signing JWT tokens | Random 32+ char string |
| `SEED_ADMIN_EMAIL` | Yes | Admin user email for seeding | `admin@xeno.local` |
| `SEED_ADMIN_PASSWORD` | Yes | Admin user password for seeding | Secure password |
| `CHANNEL_WEBHOOK_SECRET` | Yes | Shared HMAC secret for webhook verification | Random 32+ char string |
| `CHANNEL_SERVICE_URL` | Yes | Channel simulator URL | `http://localhost:3001` |
| `FRONTEND_URL` | Yes | Frontend URL (for CORS) | `http://localhost:5173` |
| `ANTHROPIC_API_KEY` | No | Anthropic API key | `sk-ant-...` |
| `ANTHROPIC_BASE_URL` | No | Custom Anthropic gateway URL | `https://api.anthropic.com` |
| `ANTHROPIC_MODEL` | No | Model override | `claude-sonnet-4-20250514` |
| `AI_MAX_ROUNDS` | No | Max LLM rounds per message | `8` |
| `AI_MAX_TOOL_CALLS` | No | Max tool calls per message | `8` |
| `AI_MAX_EXECUTION_MS` | No | Max execution time (ms) | `25000` |
| `AI_MAX_INPUT_TOKENS` | No | Max input tokens per call | `100000` |
| `AI_MAX_TOKENS_PER_QUERY` | No | Max total tokens per query | `50000` |
| `AI_CONFIRMATION_TTL_MS` | No | Confirmation expiry (ms) | `900000` |
| `AI_HISTORY_LIMIT` | No | Messages in context window | `40` |
| `AI_RETRY_MAX_ATTEMPTS` | No | Tool retry attempts | `1` |
| `AI_RETRY_BACKOFF_MS` | No | Retry backoff (ms) | `1000` |
| `SUPABASE_URL` | No | Supabase project URL | `https://xxx.supabase.co` |

### Channel Simulator (`Backend/channel/.env`)

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `CRM_SERVICE_URL` | Yes | CRM API URL for webhooks | `http://localhost:3000` |
| `CHANNEL_WEBHOOK_SECRET` | Yes | Shared HMAC secret | Same as CRM |

### Frontend (`Frontend/.env`)

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `VITE_API_URL` | Yes | CRM API base URL | `http://localhost:3000/api` |

## Production Deployment

### Vercel

Both the CRM API and Channel Simulator include `vercel.json` for serverless deployment:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "dist/src/main.js",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "dist/src/main.js"
    }
  ]
}
```

The frontend can be deployed to Vercel as a static site or with server-side rendering.

### Database

**Neon (Recommended for production):**
- Serverless PostgreSQL with branching
- Connection pooling via PgBouncer
- Use `DATABASE_URL` for pooled connections, `DIRECT_URL` for migrations

**Supabase:**
- Managed PostgreSQL with built-in connection pooler
- Use pooler URL (port 6543) for `DATABASE_URL`
- Use direct URL (port 5432) for `DIRECT_URL`

### Redis

**Upstash (Recommended):**
- Serverless Redis with REST API
- Compatible with BullMQ
- Configure via `UPSTASH_REDIS_REST_URL` and `UPSTASH_REDIS_REST_TOKEN`

### Running Migrations in Production

```bash
cd Backend/crm
npm run prisma:generate
npm run prisma:migrate
```

Migrations should be applied before deploying new code. The `prisma:migrate` command runs all pending migrations.

## Monitoring

### Health Endpoints

```
GET /api/health       → 200 OK (liveness)
GET /api/health/ready → 200 OK if DB connected, 503 if not
```

### Logs

The CRM API uses **Pino** for structured JSON logging:

```json
{
  "level": 30,
  "time": "2026-06-14T10:00:00.000Z",
  "context": "AIService",
  "msg": "Tool getCampaignAnalytics completed in 45ms"
}
```

### Processing Failures

Failed background jobs are logged to the `ProcessingFailure` table with:
- Queue name
- Job ID
- Correlation ID
- Error reason
- Full diagnostics JSON

## Security Checklist

- [ ] Rotate all credentials that have been pasted into source material or chat
- [ ] Use strong, random secrets for `JWT_SECRET` and `CHANNEL_WEBHOOK_SECRET`
- [ ] Enable HTTPS in production
- [ ] Configure CORS to only allow your frontend domain
- [ ] Use connection pooling for database connections
- [ ] Set appropriate rate limits via the throttler module
- [ ] Enable Helmet security headers
- [ ] Verify webhook signatures on all incoming callbacks
- [ ] Use httpOnly cookies for JWT tokens
- [ ] Enable email verification for new user registrations
