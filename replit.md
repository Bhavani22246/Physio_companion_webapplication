# PhysioAI Coach

AI-powered physiotherapy rehabilitation web app with live camera pose detection, voice guidance, pain tracking, and therapist monitoring.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000, proxied at /api)
- `pnpm --filter @workspace/physio-ai run dev` — run the React frontend (port 20412, proxied at /)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — session secret

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React 18 + Vite, TailwindCSS v4, Framer Motion, Recharts, Wouter
- API: Express 5 + Pino logging
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth for all contracts)
- `lib/api-zod/src/generated/api.ts` — Zod schemas (generated)
- `lib/api-client-react/src/generated/api.ts` — React Query hooks (generated)
- `lib/db/src/schema/` — Drizzle ORM schema (users, exercises, sessions, session_exercises, pain_logs, achievements)
- `artifacts/api-server/src/routes/` — Express route handlers
- `artifacts/physio-ai/src/pages/` — All frontend pages
- `artifacts/physio-ai/src/components/` — Shared components (layout, UI)

## Architecture decisions

- Contract-first API design: OpenAPI spec → Zod + React Query hooks via Orval codegen
- All Drizzle Date objects are serialized via `serializeDates()` helper before Zod parsing (Drizzle returns JS Date, Zod expects string)
- Demo user: userId=1 (Sarah Johnson, ACL patient), therapistId=2 (Dr. Michael Chen)
- Pose detection uses browser Canvas API with animated skeleton overlay (MediaPipe ready)
- Web Speech API provides real-time voice coaching during exercise sessions

## Product

- **Landing page**: Animated dark hero, feature cards, testimonials, stats, CTA
- **Login/Register**: Role-based auth (patient vs therapist), smooth transitions
- **Patient Dashboard**: Live API data — streak, recovery %, weekly goal, session history, achievements, pain banner
- **Exercise Library**: Searchable/filterable grid, recommendations, difficulty badges, category filters
- **Live Exercise Session**: Camera + animated AI skeleton overlay, rep counter, pose score, voice cues, form tips
- **Recovery Progress**: Circle gauge, weekly area chart, pain trend line chart, achievement badges
- **Pain Tracker**: Slider input (0–10), body area picker, history list with color-coded severity
- **Therapist Dashboard**: Patient roster with status badges, at-risk alerts, recovery bars, activity feed

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Always run `serializeDates()` on Drizzle query results before Zod `.parse()` — Drizzle returns Date objects, Zod schemas expect ISO strings
- `lib/api-zod/src/index.ts` must only have `export * from "./generated/api";` — orval regenerates it each codegen run
- Do NOT add `totalExercises` to CreateSessionBody — only `userId` and `painLevelBefore` are valid
- Hook name for adding exercises to a session is `useAddSessionExercise` (not `useCreateSessionExercise`)

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
