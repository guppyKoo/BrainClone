---
title: Now — current snapshot
area: profile
tags: [now, snapshot, in-progress]
created: 2026-08-03
updated: 2026-08-24
status: draft
---

# Now — current snapshot

> Current-state snapshot. AI reads `index.md` and this file first, for any task.
> Highest update frequency — past states live in git history.

Last updated: 2026-08-24

## In progress

### Dev

- **BrainClone**: structure settled (developer-centric split), migrated to English. Next: fill gaps, promote drafts to confirmed
  - Big picture is a [[second-brain]] concept — the top requirement is phone/work machine/home machine sharing **one** knowledge base
- **Bulk Mail**: v1.1.x — handling macOS ad-hoc codesigning (`fix/adhoc-signing` branch)
- **INOS**: humanities meetup platform in development + Electron desktop app wrapping the existing web
  - **2026-08-24**: big feature push — email notifications + in-app inbox, meeting start time, board
    (`하고싶은 말`), local email/password auth and invite links, presentation mode, and a recovery path
    for failed AI prompt generation. Details in [[inos]]. Work sits on open PRs in `yunchan312/INOS`;
    nothing from this push merged to `master` yet
  - Open question raised while wrapping up: **which model should generate the discussion prompts**
    (GPT / Gemini / Grok / Claude, and whether a small model suffices). No evaluation run yet
- **Work (goorm) — Edu Vibe**: current focus. Owns two repos (`edu-vibe-front`, `edu-vibe-server`). As of 2026-08-14 **both are past M0** — front has routing + Tailwind/Vapor-ui scaffolding, server has schemas, a repository layer and the full MVP API (verified end-to-end against local Mongo). Wrote the V2 Tech Spec, the DB schema doc and the API spec himself, using AI as a critical reviewer rather than an author
  - Server: NestJS 11 (**Express** adapter, for a possible 전자정부표준프레임워크/Spring migration) + MongoDB + **Mongoose** + `migrate-mongo` — no in-house precedent for migrations, so the convention was set from scratch
  - Front: **single package, not a workspace** — pnpm + Vite + **React 19** + `react-router-dom` v7 + Tailwind v4 + Vapor-ui + TanStack Query, on **Node 24**. Turborepo was dropped; a one-package workspace is pure overhead
  - Not wired yet: LiteLLM proxy, S3 storage, web deploy — the three are explicit boundaries in the server code
  - Prisma was evaluated and rejected for MongoDB — no Prisma Migrate on Mongo, aggregation escapes the type system, replica set needed even for plain nested writes
  - The 학습창 is assumed to arrive as `@devth/learn-*` SDK packages from another team (Devth) — this assumption is what the whole architecture rests on
  - In-house convention references: `gem-server` (NestJS+Express+Mongoose), `new-edu` (pnpm+Turbo+Vite+Vapor-ui)
  - **EDU VIBE practice-page boundary discussion (2026-08-18)**: EDU proposed four DEVEL collaboration options for reusable coding-practice and AI-chat capabilities. The preferred direction is a standalone API service owning projects, files, chats, builds, and artifacts; an SDK with consumer-owned data is the main alternative. No decision has been finalized.
- **Work (goorm)**: edu-core / goorm-hocuspocus realtime collaborative editing — epoch-based document versioning
- **Cascade-delete redesign (exploring, 2026-08-14)**: looking for a lighter replacement for the legacy
  Kafka-based cascade path on MongoDB. Options weighed and the decision table are in
  [[mongodb-cascade-strategies]]. Current default recommendation is multi-document transactions;
  no change made yet. Relevant constraint: the app runs multiple instances on one server

### Non-dev

- **Writing**: Hongcheon travel essay **published on velog 2026-08** as two posts — 「뇌가 뜨거워서 그랬어: 첫날」
  (08-15) and 「마지막 날」 (08-17). Travel series now 6 posts. Details in [[writing|interest/hobbies/writing.md]]. Also started a deliberate prose-practice routine
  (transcription + weekly revision + read-aloud) aimed at sentence-level control
- **Humanities club**: film curator in a 4-person monthly club. Current block is **two films over two
- **OPIc**: 현재 IM3. 난이도 5-5와 선택한 Background Survey를 기준으로 영어 실전 질문에 답하고,
  답변별 피드백을 받는 방식으로 준비 중. 상세는 [[opic|interest/topics/opic.md]].
  months, discussed in one session** — *Lost in Translation* (focus: composition) then *The Substance*
  (focus: sound). Pre-watch guide written 2026-08-24. Method and the six-element framework:
  [[humanities|interest/hobbies/humanities.md]]. The Coen brothers curriculum is the earlier plan
- **Band**: trying to write original songs instead of covers (Sing Street aftermath)
- **Fitness**: maintaining a 6-day/week gym routine

## Digging into

- AI agent workflows (Claude Code hooks, skills, multi-agent) → details: [[interest|developer/interest]]
- MongoDB Change Streams / CDC → [[mongodb-cascade-strategies]]
- Flutter, Playwright — learning stage
- Investing & macroeconomics → [[investing|interest/topics/investing.md]]

## Open questions (AI should ask)

- Jeju Bio AX hackathon — registration closed end of July 2026. Did the user participate? Result? (unconfirmed, so excluded from In progress)
- Cascade redesign: **is the legacy Kafka topic consumed by the same service or a different one?**
  Asked repeatedly on 2026-08-14, never answered. Same service ⇒ drop Kafka; different service ⇒ keep it and add an Outbox
- Cascade redesign: is the MongoDB deployment a replica set? Decides whether transactions are available at all

## This quarter's priorities

## Changelog

- 2026-08-24: INOS feature push logged (notifications/inbox, meeting time, board, local auth +
  invite links, presentation mode, generation-failure recovery) and the model-choice question opened
- 2026-08-24: humanities club — two-film block (Lost in Translation / The Substance) with
  per-film viewing focus; curation method recorded in [[humanities]]
- 2026-08-18: Hongcheon essay published on velog (2 posts); travel series now 6 posts
- 2026-08-18: logged the EDU VIBE/DEVEL practice-page modularization agenda; decision pending

- 2026-08-19: Edu Vibe front moved to React 19 — the earlier "pinned to 18" note was wrong (`react-router-dom` v7 accepts React >=18)
- 2026-08-14: Edu Vibe corrected — both repos are past M0 (front routing scaffolded, server API built and verified); front is a single package, not a Turborepo workspace; React pinned to 18, Node 24
- 2026-09-01: OPIc 준비 현황 추가 — 현재 IM3, 난이도 5-5, Background Survey 기반 영어 문답 연습
- 2026-08-14: added the cascade-delete redesign thread and its two open questions; logged travel-essay part 2 and the writing-practice routine
- 2026-08-12: work focus updated — Edu Vibe (V2 Tech Spec, NestJS+Mongoose stack decided, M0 scaffolding pending)
- 2026-08-03: added frontmatter (OKF compliance), expanded with work & non-dev activities
- 2026-08-03: migrated to English
