# ADR 0001: Technology stack

- **Status:** Accepted (revisit at gate G2)
- **Date:** 2026-10-29

## Context

A solo or very small team will build the console with a coding agent. The stack must be productive, widely documented, and operable by one person. The product needs a web UI, an API, realtime events, a job queue, and strict tenancy.

## Decision

TypeScript across the stack. Next.js (App Router) for UI and API; Tailwind CSS with design tokens; Radix UI primitives; PostgreSQL 16 with Drizzle ORM and Row-Level Security; `pg-boss` for the queue; SSE for events and WebSocket for the mirror and runner gateway; S3-compatible object storage; Auth.js (GitHub OAuth); Stripe; Vitest and Playwright.

## Alternatives considered

- Separate backend (NestJS/Go) and SPA: more moving parts for the MVP.
- Redis/BullMQ or a managed queue: extra infrastructure; reconsider at scale.
- Firebase/Supabase: fast start, but tenancy, queue, and runner gateway needs are better served by explicit Postgres design; revisit if operations burden grows.

## Consequences

- One language and one deployable for the MVP; easy for agents to work in.
- Postgres carries more load (queue, events); acceptable for MVP, with a documented path to dedicated infrastructure.
- Next.js route handlers must stay thin; business logic lives in testable modules.
