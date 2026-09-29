# SAMAJ Community Portal

## What this is
Web portal for a Kenyan Indian community organisation (the SAMAJ). Replaces paper forms + Excel.
Two registries: SAMAJ (all members + family) and Medical (members enrolled in medical assistance).
Only approved SAMAJ members can enrol in Medical.

## Phase 1 scope (only build this)
Auth, SAMAJ registration, admin approval queue, medical enrolment, member read-only profile,
admin dashboard with search/filter/export, Excel seed import. Nothing else
(no bill uploads, payments, NFC, mobile app).

## Stack (do not change without asking)
- Monorepo with pnpm workspaces: apps/web, apps/api, packages/shared
- Frontend: React + Vite + TypeScript, Tailwind, React Router, TanStack Query, TanStack Table,
  React Hook Form + Zod
- Backend: Node 20 + Express + TypeScript, Prisma ORM, Zod validation
- DB: PostgreSQL 16 (Docker locally)
- Email: Resend (use a console logger in dev)
- Excel: exceljs
- Tests: Vitest + Supertest (api), Playwright (e2e)

## Roles
- SUPER_ADMIN: everything + manage admin accounts
- ADMIN: view/search/filter/export all members, approve/reject registrations, edit members
- MEMBER: register, log in, view OWN profile/family/medical only. Read-only in Phase 1.

## Core business rules (never break these)
1. NHIF No. is mandatory and UNIQUE, enforced by a DB unique constraint (including soft-deleted rows).
   Internal primary key is a UUID; NHIF is never used in URLs.
2. Registration flow: user signs up -> member row status=PENDING -> admin approves ->
   status=APPROVED and membership_no auto-generated from a Postgres sequence (format SJ-000001),
   all in ONE transaction -> email sent -> only then can the member enrol in Medical.
3. Medical enrolment is blocked at 3 layers: frontend route guard, API check, and a DB trigger.
4. Soft deletes only (deleted_at). No hard deletes anywhere.
5. Every create/update/delete/approve/reject/export writes to audit_log (actor, action, entity,
   before/after JSON, timestamp).
6. Age, U18 flag, family counts are DERIVED (DB view), never stored.
7. Members never pass an ID to see their data: they use /api/me/* and identity comes from the token.

## Security rules
- bcrypt (cost 12) for passwords
- JWT access token (15 min) + refresh token (7 days, rotated, stored hashed in DB)
- Tokens only in httpOnly, Secure, SameSite=Lax cookies. Never localStorage.
- helmet, CORS allowlist from env, rate limit on /auth routes
- Zod validation on every request body/query
- CSV/Excel export: prefix cells starting with = + - @ with an apostrophe
- No secrets in code; everything via .env (commit .env.example only)

## Code conventions
- API layers: routes -> controllers -> services (business logic) -> Prisma
- Shared Zod schemas + types live in packages/shared and are used by BOTH web and api
- Consistent API error shape: { error: { code, message, details? } }
- Small focused files. No giant components.
- After every task: run typecheck, lint and tests, and tell me the exact commands to verify.