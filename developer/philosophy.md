---
title: Development philosophy
area: developer
tags: [philosophy, architecture, AI-collaboration]
created: 2026-08-03
updated: 2026-08-03
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

## Architectural leanings (inferred)

- Monorepo + shared packages (prisma/types/utils) for type consistency
- Service separation (INOS: API server / AI server; Bulk Mail: settings/mail/image services)
- Explicit bridges at IPC/API boundaries (`window.api` preload bridge)

## Changelog

- 2026-08-03: migrated to English
