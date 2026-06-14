# Frontend Architecture

The Xeno frontend is a React Single Page Application built with TanStack Start, providing a modern dashboard experience with real-time analytics, campaign management, and an integrated AI copilot.

## Tech Stack

| Category | Technology | Purpose |
|----------|------------|---------|
| **Framework** | TanStack Start | SSR-capable React meta-framework |
| **Routing** | TanStack Router | Type-safe file-based routing |
| **Data Fetching** | React Query (TanStack Query) | Server state management, caching, optimistic updates |
| **Styling** | Tailwind CSS 4 | Utility-first CSS framework |
| **Components** | shadcn/ui (Radix) | Accessible, composable UI primitives |
| **Charts** | Recharts | Analytics visualizations |
| **Forms** | React Hook Form + Zod | Form state management with schema validation |
| **Notifications** | Sonner | Toast notifications |
| **Icons** | Lucide React | Consistent icon system |

## Project Structure

```
Frontend/src/
├── components/
│   ├── ui/                 # shadcn/ui base components (40+ components)
│   ├── AppShell.tsx        # Main layout with sidebar navigation
│   ├── AIPanel.tsx         # Persistent AI copilot sidebar
│   ├── CommandPalette.tsx  # Keyboard-driven command launcher (Cmd+K)
│   ├── CustomerDetailDialog.tsx
│   ├── CampaignDetailDialog.tsx
│   ├── NotificationCenter.tsx
│   ├── PageHeader.tsx      # Consistent page header component
│   ├── Funnel.tsx          # Campaign delivery funnel visualization
│   ├── EditableText.tsx    # Inline editable text component
│   ├── QueryState.tsx      # Loading/error/empty state handler
│   └── Skeleton.tsx        # Loading skeleton components
├── routes/
│   ├── __root.tsx          # Root layout
│   ├── index.tsx           # Landing/redirect
│   ├── auth.tsx            # Login/register page
│   ├── verify-email.tsx    # Email verification
│   └── _app.tsx            # Authenticated layout wrapper
│       ├── _app.dashboard.tsx    # Main dashboard
│       ├── _app.customers.tsx    # Customer management
│       ├── _app.segments.tsx     # Segment builder
│       ├── _app.campaigns.tsx    # Campaign list
│       ├── _app.campaigns.$id.tsx # Campaign detail
│       ├── _app.analytics.tsx    # Global analytics
│       ├── _app.ai.tsx           # Full-page AI copilot
│       ├── _app.insights.tsx     # AI insights
│       └── _app.managers.tsx     # Admin user management
├── hooks/
│   ├── use-mobile.tsx      # Mobile viewport detection
│   └── use-notification-state.tsx # Notification state management
├── lib/
│   ├── api.ts              # API client with auth interceptors
│   ├── contracts.ts        # Shared Zod schemas and TypeScript types
│   ├── config.server.ts    # Server-side configuration
│   ├── utils.ts            # Utility functions (cn, formatters)
│   ├── error-capture.ts    # Error boundary utilities
│   ├── error-page.ts       # Error page component
│   ├── useCountUp.ts       # Animated number counter hook
│   └── useInView.ts        # Intersection observer hook
├── test/
│   └── setup.ts            # Vitest test setup
├── router.tsx              # Router configuration
├── routeTree.gen.ts        # Auto-generated route tree
├── server.ts               # Server entry point
└── start.ts                # Application entry point
```

## Routing

TanStack Router provides type-safe file-based routing. Routes are defined in `src/routes/` and the route tree is auto-generated in `routeTree.gen.ts`.

### Route Layout

```
/                        → Redirect to /dashboard or /auth
/auth                    → Login and registration
/verify-email            → Email verification handler
/_app                    → Authenticated shell (AppShell)
  /dashboard             → Main dashboard with KPIs
  /customers             → Customer list with search/filter
  /segments              → Segment list with rule preview
  /campaigns             → Campaign list with status filters
  /campaigns/:id         → Campaign detail with funnel
  /analytics             → Global analytics charts
  /ai                    → Full-page AI copilot
  /insights              → AI-generated business insights
  /managers              → Admin user management (ADMIN only)
```

### Route Configuration

Each route file exports a `Route` object:

```typescript
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/_app/dashboard")({
  component: DashboardPage,
});
```

SSR is disabled for authenticated routes (`ssr: false`) to simplify auth handling — the app renders as a client-side SPA after initial load.

## Components

### AppShell

The main application layout (`AppShell.tsx`) provides:

- **Sidebar navigation** with collapsible menu items
- **AI Panel** toggle (persistent sidebar copilot)
- **User menu** with profile, settings, logout
- **Notification center** for system alerts
- **Breadcrumbs** for nested navigation
- **Responsive layout** that collapses sidebar on mobile

### AIPanel

The AI copilot panel (`AIPanel.tsx`) is a persistent sidebar that:

- Maintains conversation history across page navigation
- Shows tool execution status in real-time
- Supports confirmation prompts for destructive operations
- Displays grounding sources for transparency
- Allows conversation management (new, rename, delete)
- Supports both synchronous and streaming message modes

### CommandPalette

A keyboard-driven command launcher (`Cmd+K` / `Ctrl+K`) providing:

- Quick navigation to any page
- Customer/campaign/segment search
- AI copilot quick actions
- Keyboard shortcuts for power users

### shadcn/ui Components

The `components/ui/` directory contains 40+ Radix-based components:

accordion, alert-dialog, avatar, badge, button, calendar, card, carousel, chart, checkbox, collapsible, command, context-menu, dialog, dropdown-menu, form, hover-card, input, input-otp, label, menubar, navigation-menu, popover, progress, radio-group, resizable, scroll-area, select, separator, sheet, skeleton, slider, sonner, switch, table, tabs, textarea, toggle, toggle-group, tooltip

## State Management

### Server State (React Query)

All API data is managed through React Query:

```typescript
// Query for fetching data
const { data, isLoading, error } = useQuery({
  queryKey: ["customers", { page, search }],
  queryFn: () => api.getCustomers({ page, search }),
});

// Mutation for creating data
const mutation = useMutation({
  mutationFn: api.createCustomer,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["customers"] });
    toast.success("Customer created");
  },
});
```

### API Client

The API client (`lib/api.ts`) provides:

- **Base URL configuration** from `VITE_API_URL` environment variable
- **Auth interceptors** that attach JWT tokens and handle refresh
- **Error handling** with automatic retry for 401s
- **Type-safe methods** matching backend API contracts

### Contracts

Shared Zod schemas and TypeScript types (`lib/contracts.ts`) ensure type safety between frontend and backend:

- Channel types (WHATSAPP, SMS, EMAIL, RCS)
- Campaign status lifecycle
- Segment rule validation
- API response envelopes
- AI tool names and types
- Insight types and priorities

## Analytics & Charts

The analytics pages use **Recharts** for data visualization:

### Dashboard Charts
- **Revenue trends** — Area chart showing daily revenue
- **Campaign performance** — Bar chart comparing sent vs converted
- **Channel performance** — Radar chart across channels
- **Segment performance** — Horizontal bar chart

### Campaign Detail Charts
- **Delivery funnel** — Funnel visualization (sent → delivered → opened → clicked → converted)
- **Rate gauges** — Delivery rate, open rate, click rate, conversion rate
- **Failure breakdown** — Pie chart of failure reasons

### Global Analytics
- **Revenue attribution** — Stacked area chart by campaign
- **Delivery trends** — Line chart across time
- **Channel comparison** — Grouped bar chart
- **Segment effectiveness** — Scatter plot

## Forms

Forms use **React Hook Form** with **Zod** validation:

```typescript
const form = useForm<CreateCustomerInput>({
  resolver: zodResolver(createCustomerSchema),
  defaultValues: { name: "", email: "", phone: "", tags: [] },
});

const onSubmit = (data: CreateCustomerInput) => {
  createCustomerMutation.mutate(data);
};
```

## Responsive Design

The frontend is fully responsive:

- **Desktop:** Full sidebar + AI panel + content area
- **Tablet:** Collapsible sidebar, AI panel as overlay
- **Mobile:** Bottom navigation, full-screen dialogs, stacked layouts

Mobile detection uses the `use-mobile.tsx` hook with a 768px breakpoint.

## Testing

Tests use **Vitest** with **React Testing Library**:

```bash
npm run test         # Run all tests
npm run test:watch   # Watch mode
```

Test files are co-located with components (e.g., `AppShell.test.tsx`, `campaigns.test.tsx`).

Test setup in `src/test/setup.ts` configures:
- jsdom environment
- Custom matchers from @testing-library/jest-dom
- Mock for window.matchMedia and IntersectionObserver

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `VITE_API_URL` | CRM API base URL | `http://localhost:3000/api` |

## Development

```bash
cd Frontend
npm install
npm run dev          # Start dev server (port 5173)
npm run lint         # ESLint check
npm run typecheck    # TypeScript check
npm run format       # Prettier formatting
npm run build        # Production build
```

## Key Design Decisions

1. **No SSR for authenticated routes:** Simplifies auth handling; the app hydrates as a SPA after initial load
2. **File-based routing:** TanStack Router's file convention keeps routes organized and type-safe
3. **Persistent AI Panel:** The copilot maintains state across navigation for a seamless conversational experience
4. **Shared contracts:** Zod schemas are duplicated (not shared via monorepo) to maintain deployment independence
5. **shadcn/ui:** Components are copied into the project (not installed as a package) for full customization control
