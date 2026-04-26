# AGENTS.md

## 1) Project overview
- **What it is:** A Duolingo-style language learning web app (“Lingo”) with course/unit/lesson/challenge progression, hearts/XP mechanics, subscriptions, and an admin CRUD interface for content management.
- **Stack:** Next.js App Router (TypeScript), React Server + Client Components, Clerk auth, Drizzle ORM with Neon Postgres, Stripe subscriptions/webhooks, Tailwind CSS + shadcn/ui, Zustand for modal state.
- **Architecture pattern:**
  - UI routes are under `app/` route groups (`(marketing)`, `(auth)`, `(main)`, `lesson`, `admin`).
  - Data access is centralized in `db/queries.ts` (cached read queries) and server actions in `actions/*.ts` (mutations + `revalidatePath`).
  - Admin UI uses `react-admin` backed by internal REST endpoints under `app/api/*`.

## 2) Repository map
- `app/`: Next.js routes/layouts/API routes.
  - `app/(marketing)`: public landing.
  - `app/(auth)`: sign-in/sign-up pages.
  - `app/(main)`: authenticated app screens (`learn`, `courses`, `shop`, `quests`, `leaderboard`).
  - `app/lesson`: lesson runtime UI.
  - `app/admin`: react-admin dashboard (client-only dynamic import).
  - `app/api/*`: REST endpoints for admin resources and Stripe webhook.
- `actions/`: server actions for progress/subscription mutations.
- `db/`: Drizzle connection, schema, and read-query layer.
- `components/`: reusable UI and feature components (`components/ui` for shadcn/radix wrappers).
- `lib/`: cross-cutting helpers (`stripe`, admin auth helper, utils).
- `store/`: Zustand modal stores.
- `scripts/prod.ts`: seed script that clears and reseeds DB content.
- Config: `package.json`, `tsconfig.json`, `eslint.config.mjs`, `.prettierrc.json`, `tailwind.config.ts`, `drizzle.config.ts`, `next.config.ts`, `proxy.ts`, `vercel.ts`.
- CI/workflows: `.github/workflows/update-readme.yml` auto-updates README structure/dependency sections on pushes to `main`.

**Usually avoid editing unless task requires it**
- `public/*` media assets (audio/images) used by seeded challenges.
- Auto-generated/ephemeral artifacts: `.next/`, `node_modules/`, build outputs.
- README dependency/folder sections are auto-maintained by workflow; manual edits there may be overwritten.

## 3) Setup and local development
1. Install dependencies:
   - `bun install --legacy-peer-deps`
2. Create env file from example:
   - `cp .env.example .env`
3. Fill required variables (see `.env.example`):
   - Clerk keys + Clerk redirect vars
   - `DATABASE_URL`
   - `STRIPE_API_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`
   - `NEXT_PUBLIC_APP_URL`
   - `CLERK_ADMIN_IDS` (comma+space separated IDs)
4. Initialize schema and seed data:
   - `bun run db:push`
   - `bun run db:prod`
5. Start local dev server:
   - `bun dev`

**Required external services**
- Clerk (authentication)
- Postgres/Neon (database)
- Stripe (subscriptions + webhook signatures)

## 4) Build, test, lint, and typecheck commands
- **Dev:** `bun dev`
- **Production build:** `bun run build`
- **Start built app:** `bun run start`
- **Lint:** `bun run lint`
- **Lint fix:** `bun run lint:fix`
- **Format check:** `bun run format`
- **Format write:** `bun run format:fix`
- **DB push:** `bun run db:push`
- **Seed script:** `bun run db:prod`
- **Drizzle Studio:** `bun run db:studio`

**Fast pre-finish checks for most code changes**
1. `bun run lint`
2. `bun run format`
3. `bun run build` (for route/action/schema-impacting changes)

**Notes/caveats**
- No dedicated test suite is configured in this repo (Unknown from repo inspection).
- `db:prod` is destructive to seeded tables (it deletes and reinserts app data).

## 5) Coding conventions
- TypeScript is strict (`strict: true`), prefer explicit typing at boundaries.
- Follow existing Next.js App Router patterns:
  - Server Components by default.
  - Use `"use client"` only when required (state/effects/browser-only libs).
  - Mutations belong in server actions (`actions/*`) or API routes.
- Reuse existing abstractions:
  - Reads from `db/queries.ts`.
  - DB schema/types from `db/schema.ts`.
  - Auth/admin helpers from `@/lib/admin` and Clerk server APIs.
  - UI building blocks from `components/ui/*` and `lib/utils.ts` (`cn`).
- Formatting/style:
  - Prettier: semicolons, double quotes, tailwind plugin sorting.
  - ESLint extends `next/core-web-vitals`, `next/typescript`, `prettier`.
- Data/cache behavior:
  - After mutations, revalidate impacted pages with `revalidatePath` (as existing actions do).

## 6) Change workflow for agents
1. Inspect impacted route + related query/action/schema before editing.
2. Prefer **small, focused diffs**; avoid broad refactors unless requested.
3. For data model changes:
   - Update `db/schema.ts` and impacted queries/actions/API/admin resources together.
   - Call out migration implications explicitly in your report.
4. For admin/API changes:
   - Keep auth checks (`getIsAdmin`) and consistent REST shapes for react-admin.
5. For UI changes:
   - Reuse existing component patterns and Tailwind conventions.
6. If behavior/config/env requirements change, update docs (`README.md` and/or this file).

## 7) Verification checklist
Before finalizing:
- [ ] Dependencies installed and env vars set for intended commands.
- [ ] `bun run lint` passes.
- [ ] `bun run format` passes (or run `format:fix`).
- [ ] `bun run build` passes for non-trivial changes.
- [ ] If DB logic changed, verify affected flows (course selection, lesson progress, hearts/XP, subscription gating) and admin CRUD paths.
- [ ] If Stripe or auth logic changed, verify webhook/admin/auth-protected route behavior.
- [ ] Update docs/config references when required.

## 8) Security and safety rules
- Never commit secrets or real credentials (`.env`, API keys, webhook secrets, private IDs).
- Do not weaken auth or authorization:
  - Keep Clerk protection in `proxy.ts`.
  - Preserve admin gating in API/admin paths (`getIsAdmin`).
- Do not bypass Stripe signature verification in `app/api/webhooks/stripe/route.ts`.
- Do not silently change production-sensitive config (`next.config.ts`, `vercel.ts`), billing logic, or schema semantics without explicit mention.
- Be careful with CORS/auth/API changes—they affect admin/API exposure.

## 9) PR / commit expectations
- No strict commit format is enforced by repo config; use clear, scoped commit messages (e.g., `docs: update AGENTS.md for repo workflow`).
- Final agent summary should include:
  - Files changed and why
  - Commands run and outcomes
  - Any risks, caveats, or follow-up work

## 10) Subdirectory-specific instructions
- `app/admin` + `app/api/*`: Keep react-admin resource compatibility and admin auth checks aligned.
- `actions/*` + `db/*`: Keep mutation/read model consistency and cache revalidation coverage.
- `scripts/prod.ts`: Treat as seed/bootstrap script; avoid casual edits because it controls initial learning content.

**Potential future nested AGENTS.md (not created now):**
- `app/admin/AGENTS.md` for react-admin/API resource conventions.
- `db/AGENTS.md` for schema/query mutation patterns and data-safety rules.
