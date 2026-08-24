---
title: INOS — humanities meetup platform
area: career
tags: [portfolio, NestJS, AI, monorepo, side-project]
created: 2026-08-03
updated: 2026-08-24
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

## Feature build-out (2026-08-24)

A long solo session took the product from "prompts generate" to "a club can actually be run on it".

- **Notifications**: four email triggers (date confirmed, prompts ready, 3h before, availability
  nudge at 48h), all on the existing SMTP + BullMQ infrastructure — no new secrets or services.
  Duplicate sends are prevented structurally by a unique `(meeting, recipient, type)` log row
  rather than by application checks. Delayed jobs are re-scheduled when the date changes and
  cancelled when the meeting ends or is deleted.
- **In-app inbox** reuses that same log table as its store — the dedup ledger already recorded
  "who was told what, when", so the inbox needed only a `readAt` column, not a new table.
- **Meeting time**: `confirmedTime` (`HH:mm`, nullable) so the 3h reminder is computed from the
  real start instead of an assumed evening hour (`MEETING_DEFAULT_HOUR` stays as the fallback).
- **Board (`하고싶은 말`)** with a hand-written markdown subset + toolbar, likes, pagination.
- **Auth widened**: local email/password signup beside Google OAuth, plus copyable invite links —
  both added because email-only invitations were a real barrier for less technical members.
- **Presentation mode**: fullscreen one-question-at-a-time view for running the meeting itself.
- **Failure path for AI generation**: failures used to leave the discussion stuck at `GENERATING`
  forever with no recovery (two rows had sat stuck for 31 days). Added a `FAILED` state written
  from both servers, and gave the owner a "re-check the title/author/director, then regenerate"
  panel. Wrong artwork metadata is the likeliest cause, so the fix is a correction form rather
  than a bare retry button.

## Talking points

- Solved a real pain point of his own hobby with full-stack + AI — he is the club's film curator ([[humanities]]) and a user
- Production-grade components (vector search, queues, OAuth) designed solo in a personal project
- **Prefers reusing an existing mechanism over adding one**: the notification inbox rides on the
  email dedup log; realtime updates ride on the discussion socket gateway. A good answer for
  "how do you decide when to add infrastructure?"
- **Realtime lesson worth retelling**: saving an impression was unreliable while the main server
  wrote the row and then asked the AI server over HTTP to broadcast it. Moving the write to the
  server that owns the socket gateway fixed it — whoever owns the gateway should own the write.

## Changelog

- 2026-08-24: recorded the notification/inbox/board/auth/presentation/failure-recovery build-out and two reusable talking points
- 2026-08-03: added product strategy (invite-only, AI-prompt hypothesis, paid feature) and Electron desktop progress
- 2026-08-03: migrated to English
