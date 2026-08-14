---
title: Development philosophy
area: developer
tags: [philosophy, architecture, AI-collaboration, portability, data-integrity]
created: 2026-08-03
updated: 2026-08-14
status: draft
---

# Development philosophy

> Draft reverse-engineered from how he works. Worth rewriting in his own words.

## Observed philosophy (inferred)

1. **Ship all the way**: "it builds" isn't done — "the packaged app runs" is done.
   Bulk Mail was verified through typecheck → build → dev → DMG → packaged-app run before calling it complete.
2. **Dig to the root cause**: went down into library source (y-prosemirror) to add a null guard instead of routing around symptoms.
   Disproved the wrong hypothesis with tests and kept the record.
3. **Rules as systems, not documents**: conventions are enforced by hooks (GateGuard), rulesets (ECC), and frameworks (OKF), not memory.
4. **Stand on proven ground, but own it**: uses proven bases (Radix/TanStack), yet re-implemented ~30 UI components to own and understand the core layer.
5. **AI executes, humans design the structure**: define folder structure and rules first, then delegate execution to AI.

## Stated principles

- **Portable over machine-local**: agent config and conventions must work on every machine he uses.
  Enforcement that lives on a single device is rejected even when it works — the rule text in a
  synced file wins over a locally installed script. This refines observed principle 3: rules should
  be systems, but not device-bound ones.
- **Guarantees belong in the engine, not in convention** (stated 2026-08-14): prefers RDBMS
  specifically because `ON DELETE CASCADE` is enforced regardless of how many app instances run or
  which path issued the delete. Frames MongoDB's app-level cascade as the messy alternative rather
  than as a legitimate trade. Same shape as principle 3 — a rule that depends on everyone remembering
  it is not a rule. See [[mongodb-cascade-strategies]].
  - Corollary he accepts once shown: this cleanliness is conditional on a single un-sharded database.
    Once services or shards split, RDBMS lands in the same place (Outbox + events).

## Architectural leanings (inferred)

- Monorepo + shared packages (prisma/types/utils) for type consistency
- Service separation (INOS: API server / AI server; Bulk Mail: settings/mail/image services)
- Explicit bridges at IPC/API boundaries (`window.api` preload bridge)
- (inferred) Skeptical of infrastructure added to solve a problem that a smaller change would solve —
  pushed back hard on the idea of introducing Redis Streams merely to deduplicate Change Stream consumers

## Changelog

- 2026-08-14: added stated principle — engine-enforced guarantees over app-level convention
  (RDBMS cascade preference, from the MongoDB cascade design discussion)
- 2026-08-07: added stated principle — portable agent config over machine-local enforcement
  (declined a Stop hook for the BrainClone auto-record rule because it would live on one machine only)
- 2026-08-03: migrated to English
