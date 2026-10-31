# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Order Management System REST API: Node 20 + TypeScript (ESM), Express, Prisma + MySQL 8, Zod, JWT, Pino, Vitest + Supertest. Every new feature must be documented within `/docs`.

## Commands
- DB: `docker compose up -d` (MySQL on :3306) → `cp .env.example .env` → `npm run db:migrate` → `npm run db:seed`
- Dev server: `npm run dev` (tsx watch, loads `.env`) · build: `npm run build` · prod: `npm start`
- Test: `npm test` · single: `npx vitest run tests/orders.test.ts -t "creates an order"`
- Lint/format: `npm run lint` · `npm run format`
- Reset DB: `npm run db:reset`

## Architecture
- Modules in `src/modules/<name>/`: `*.routes.ts` → `*.controller.ts` → `*.service.ts` → `*.repository.ts`, plus `*.schemas.ts` (Zod). Manual DI wiring in `buildControllers()` (`src/app.ts`); routers mounted under `/api/v1` in `src/routes/index.ts`.
- Errors: throw `AppError` subclasses from `src/shared/errors/` (`statusCode` + `errorCode` like `INVALID_STATUS_TRANSITION`); `src/middlewares/error.middleware.ts` maps them, plus `ZodError` and Prisma P2002/P2025, to `{ error: { code, message, details? } }`.
- Controllers wrap handlers in try/catch → `next(err)`; request validation via `validate({ body, query, params })`.
- Auth: `authenticate` + `requireRole('ADMIN' | 'OPERATOR')` in `src/middlewares/auth.middleware.ts`.
- Orders: state machine in `src/modules/orders/order.status.ts`; `OrderService.changeStatus` runs in one `prisma.$transaction` (transition check, stock debit on PENDING→PAID, replenish on cancel from PAID/PROCESSING, `OrderStatusHistory` audit row). No events/queues/webhooks exist yet.
- Logging: Pino singleton in `src/shared/logger/` with redaction; snake_case event messages (`server_started`).
- `src/config/env.ts` validates env with Zod and exits on failure (`DATABASE_URL`, `JWT_SECRET` ≥16 chars required).

## Code style
- Relative imports end in `.js` (ESM); use `import type` for type-only imports (lint-enforced).
- Money as integer cents (`priceCents`, `totalCents`).

## Workflow
- Branches: `<bugfix|hotfix|feature>/<Linear issue id>` (e.g. `feature/DEV-1`); ask for the issue id if unknown.
- Commits: concise. Subject `<refactor|fix|chore>: <short summary> — <why>`, then one bullet per touched file:
  ```
  fix: correct changeStatus file path in FDD — tracker pointed to nonexistent file

  - FDD.md - fix integration section path to src/modules/orders/order.service.ts
  - TRACKER.md - update matching CODIGO row
  ```

## Environment
- Tests hit the real MySQL in `DATABASE_URL` and wipe all tables in `beforeEach` (`tests/setup.ts`); run serially in a single fork. `npm test` does not load `.env` — export it first (`set -a; . ./.env; set +a`).

## Rules
- When asked to work in a documentation context, never modify `src/`, `prisma/`, `tests/` or config files; deliverables are Markdown docs only.
- Never edit `TRANSCRICAO.md`.
- When creating documents based on a meeting transcription, every doc statement must trace to `TRANSCRICAO.md` (`[hh:mm] Name`) or a real file path in the code; never cite nonexistent files or invent requirements.
- Webhook error codes use the `WEBHOOK_*` prefix.
