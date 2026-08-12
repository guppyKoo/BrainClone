---
title: Now — current snapshot
area: profile
tags: [now, snapshot, in-progress]
created: 2026-08-03
updated: 2026-08-12
status: draft
---

# Now — current snapshot

> Current-state snapshot. AI reads `index.md` and this file first, for any task.
> Highest update frequency — past states live in git history.

Last updated: 2026-08-12

## In progress

### Dev

- **BrainClone**: started today. Structure settled (developer-centric split), migrated to English. Next: fill gaps, promote drafts to confirmed
  - Big picture is a [[second-brain]] concept — the top requirement is phone/work machine/home machine sharing **one** knowledge base
- **Bulk Mail**: v1.1.x — handling macOS ad-hoc codesigning (`fix/adhoc-signing` branch)
- **INOS**: humanities meetup platform in development + Electron desktop app wrapping the existing web
- **Work (goorm) — Edu Vibe**: current focus. Owns two new repos (`edu-vibe-front`, `edu-vibe-server`), both still empty at M0. Wrote/refined the V2 Tech Spec himself
  - Stack decided: NestJS 11 (**Express** adapter, for a possible 전자정부표준프레임워크/Spring migration) + MongoDB + **Mongoose**; front is pnpm workspace + Turborepo + React + Vite + Tailwind v4 + Vapor-ui + TanStack Query
  - Prisma was evaluated and rejected for MongoDB — no Prisma Migrate on Mongo, aggregation escapes the type system, replica set needed even for plain nested writes
  - The 학습창 is assumed to arrive as `@devth/learn-*` SDK packages from another team (Devth) — this assumption is what the whole architecture rests on
  - In-house convention references: `gem-server` (NestJS+Express+Mongoose), `new-edu` (pnpm+Turbo+Vite+Vapor-ui)
- **Work (goorm)**: edu-core / goorm-hocuspocus realtime collaborative editing — epoch-based document versioning

### Non-dev

- **Humanities club**: film curator in a 4-person monthly club — running a 6-month Coen brothers curriculum
- **Band**: trying to write original songs instead of covers (Sing Street aftermath)
- **Fitness**: maintaining a 6-day/week gym routine

## Digging into

- AI agent workflows (Claude Code hooks, skills, multi-agent) → details: [[interest|developer/interest]]
- Flutter, Playwright — learning stage
- Investing & macroeconomics → [[investing|interest/topics/investing.md]]

## Open questions (AI should ask)

- Jeju Bio AX hackathon — registration closed end of July 2026. Did the user participate? Result? (unconfirmed, so excluded from In progress)

## This quarter's priorities

## Changelog

- 2026-08-12: work focus updated — Edu Vibe (V2 Tech Spec, NestJS+Mongoose stack decided, M0 scaffolding pending)
- 2026-08-03: added frontmatter (OKF compliance), expanded with work & non-dev activities
- 2026-08-03: migrated to English
