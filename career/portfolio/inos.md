---
title: INOS — humanities meetup platform
area: career
tags: [portfolio, NestJS, AI, monorepo, side-project]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# INOS — the OS of humanities

Humanities meetup platform: movie/book group selection, AI discussion prompts (SSE streaming), scheduling, archiving.
Personal project (`~/Practice/INOS`).

## Tech

- **Monorepo**: pnpm workspace + Turborepo
  - `apps/server` (3000): NestJS + Fastify — Auth/Group/Content/Schedule/Archive APIs
  - `apps/ai-server` (3001): NestJS + Fastify — dedicated SSE streaming for prompts/recommendations/summaries
  - `apps/web` (5173): React 19 + Vite + Tailwind v4 + DaisyUI
  - `packages/prisma|types|utils`: shared schema, DTOs, utilities
- **Data**: Prisma + PostgreSQL (Supabase) + **pgvector** — vector search via `$queryRaw`
- **Infra**: BullMQ + ioredis queues, JWT + Passport + Google OAuth

## Design points

- AI traffic (long-lived SSE) isolated into its own server, away from the API server
- Prisma schema shared as a package (symlink) for cross-app type consistency
- Streaming via NestJS `@Sse()` decorator

## Product strategy

- **Invite-only, friends-based** positioning — avoids the risks and churn of meeting strangers online
- Core hypothesis: **AI discussion prompts** close the insight gap vs. paid expert-led clubs
- Artwork interpretation/explanation AI under consideration as a future **paid feature**
- Current challenge: promotion and acquiring first users
- Desktop: macOS Electron app wrapping the existing web (session persistence via `partition`, BrowserWindow loading)

## Talking points

- Solved a real pain point of his own hobby with full-stack + AI — he is the club's film curator ([[humanities]]) and a user
- Production-grade components (vector search, queues, OAuth) designed solo in a personal project

## Changelog

- 2026-08-03: added product strategy (invite-only, AI-prompt hypothesis, paid feature) and Electron desktop progress
- 2026-08-03: migrated to English
