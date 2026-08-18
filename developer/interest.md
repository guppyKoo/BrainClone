---
title: Dev interests
area: developer
tags: [interests, AI, agents, CRDT, learning, cdc]
created: 2026-08-03
updated: 2026-08-18
status: draft
---

# Dev interests

Curiosity/learning-stage dev topics. **Promote to [[stack]] once used repeatedly at work**,
keeping only "dig deeper" items here. (Non-dev interests live in `interest/`.)

## AI agents & the Claude Code ecosystem

Most active topic. Beyond mere usage — **redesigning the dev workflow itself around agents**.

- Current: large ECC rule/skill/agent fleet, hook-based quality gates (GateGuard), many MCP servers (Figma/Notion/MongoDB/GitHub/…), designing BrainClone as an AI-read/write knowledge base
- **Cross-client BrainClone integration** (2026-08-18): wants Claude Code, Codex CLI, and ChatGPT to share the same BrainClone source of truth; exploring how ChatGPT can read and write the local knowledge base while preserving its OKF structure and Git history
- Sub-interests: multi-agent orchestration, enforcing AI behavior via PreToolUse/PostToolUse hooks, cross-session memory/context management
- Next up: Claude Agent SDK custom agents, agent evaluation (eval) systems
- **Inter-agent context sharing** (2026-08-14): asked whether agent A can observe agent B's context.
  Landed on the mechanics — there is no shared memory to read; "seeing" B always reduces to injecting
  B's history into A's prompt, so the real design questions are how much, when, and what to omit
  (a failed attempt of B's, replayed verbatim, becomes verified context for A). Claude Code subagents
  are parent↔child only; siblings coordinate through a shared file.

## Realtime collaborative editing (CRDT / Yjs)

Mastered at work — the stack itself lives in [[stack]]. Remaining curiosity:

- Yjs internals (item merging, GC); comparing other CRDTs (Automerge, Loro)

## Learning-stage tech (curiosity ~ beginner)

- **Flutter** — built a learning roadmap mapping React/JS concepts; goal appears to be mobile coverage
- **Playwright** — browser automation / E2E
- **AWS SAA-C03** — drafted a 10–12 week study plan (progress unconfirmed)
- **Appsmith** — researched for internal tooling (JSONForm widget, API body serialization issue)

## Around messaging/infra

- Kafka, BullMQ, Supabase vs Firebase, Turborepo remote caching — compared in the context of work collab-editing
- **MongoDB Change Streams / CDC** (2026-08-14) — studied in depth while looking for a lighter replacement
  for the legacy Kafka cascade path. Full note: [[mongodb-cascade-strategies]].
  Adjacent unexplored ground: Debezium, Transactional Outbox, Atlas Triggers, Redis Streams vs Pub/Sub.

## Want to build

- (inferred) A personal agent on the Claude Agent SDK — a "digital twin" that reads BrainClone
- (inferred) Bulk Mail proper release — codesigned/notarized
- (inferred) INOS public launch — currently thinking through promotion and first users
- A design→implementation pipeline connecting Claude Code ↔ Claude Design via MCP

## Changelog

- 2026-08-18: recorded the plan to connect BrainClone across Claude Code, Codex CLI, and ChatGPT
- 2026-08-14: added MongoDB Change Streams / CDC and inter-agent context sharing
- 2026-08-03: created by merging interest/topics (ai-agent-tooling, realtime-collaboration) + dev wishlist items
- 2026-08-03: added learning-stage tech (Flutter, Playwright, AWS SAA-C03, Appsmith) and infra research history
- 2026-08-03: migrated to English
