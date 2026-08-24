---
title: Code style & convention preferences
area: developer
tags: [coding-style, conventions, commits]
created: 2026-08-03
updated: 2026-08-24
status: draft
---

# Code style & convention preferences

> Extracted from the ECC ruleset actually in use (`~/.claude/rules/ecc/`) and commit history.

## Core principles (enforced via ECC rules)

- **Immutability first**: return new copies instead of mutating in place
- **KISS / DRY / YAGNI**: simplest working solution; abstract only real repetition; don't build ahead of need
- **Many small files > few large files**: 200–400 lines typical, 800 max; organize by feature/domain
- **No deep nesting**: early returns instead of 4+ levels
- **No magic numbers**: named constants for meaningful thresholds
- **Explicit errors**: never swallow silently; friendly messages in UI, detailed logs on server

## Naming

- Variables/functions `camelCase`; booleans prefixed `is/has/should/can`
- Types/components `PascalCase`; constants `UPPER_SNAKE_CASE`; hooks prefixed `use`
- **Files & directories: PascalCase for TS/TSX modules and the directories holding them** — e.g.
  `src/pages/ClassHome/ClassHomePage.tsx`, `src/Routes.tsx`. Stated explicitly on Edu Vibe front (2026-08-13).
  Config and CSS files stay lowercase (`vite.config.ts`, `global.css`)

## Identifiers

- Keeps MongoDB's internal `_id` as an automatically generated ObjectId. Never place application/domain IDs in
  `_id`; schemas expose a separate string `id` field for service lookups, references, URLs, and API responses.
- Prefers **prefixed string domain IDs** — `class-`, `proj-`, `arti-` + 10 base36 chars
  (`class-pihtuke4hn`). The prefix makes the entity type visible in logs and URLs and lets DTO validation reject
  wrong-kind IDs. This preference does not replace MongoDB's ObjectId `_id`; the two identifiers have separate roles.
- Student participation records follow the same separation: `_id` is ObjectId, while `User.id` is
  `{entryCode}-{nickName}` and is uniquely indexed.
- Enum-ish fields carry meaningful strings, never magic numbers (`'up' | 'down' | null`, not `0 | 1 | 2`)

## Commits & Git

- Conventional Commits (`feat:`, `fix:` …) with **Korean messages**
  - e.g. `fix: macOS 빌드에 ad-hoc 코드사인 추가, Gatekeeper 손상 오류 안내 추가`
- **Edu Vibe branch flow**: feature branch → PR → `develop` for day-to-day work; `develop` → `master` only for deployment/release

## UI taste

- Prefers minimal Linear/Notion-style UI (redesigned Bulk Mail in that style)
- Token-based theming: seed colors (`--fg`/`--bg`) derived via `rgb(from ...)`, dark mode via `.dark` class
- Prefers building his own Radix-based component set over adopting a kit wholesale
- **UI library wrappers inherit the wrapped component's props.** For example, Edu Vibe's `CaretButton`
  wraps Vapor UI's `Button`, so its props extend `Button.Props` and pass the remaining props through instead
  of redefining a narrower, incompatible button API.

## Testing

- TDD-oriented, 80% coverage target (ECC rules)
- **Edu Vibe server testing strategy (stated 2026-08-19)**: use Jest with outside-in Red–Green–Refactor. Start from a failing API contract/E2E test, cover service business rules with repository mocks, verify each repository method through real Mongoose round trips with `mongodb-memory-server`, and use Supertest for guards, validation, serialization, and HTTP contracts. Do not force unit tests for one-line controller delegation.
- **GREEN means every representation of the contract agrees** (confirmed through the Edu Vibe TDD run,
  2026-08-24): record the RED baseline with counts and reasons; after each implementation wave, run the
  full e2e suite rather than only the touched spec; and do not call the work complete until HTTP behavior,
  migration-created indexes, serialized response keys, and generated OpenAPI all agree. If a test expectation
  appears wrong, do not edit it to fit the implementation — reconcile it with the specification first.

## Changelog

- 2026-08-24: clarified that prefixed IDs belong in a separate domain `id`; MongoDB `_id` remains an automatic ObjectId
- 2026-08-24: added the full-contract definition of GREEN learned from the Edu Vibe TDD execution
- 2026-08-20: changed the Edu Vibe branch flow to use `develop` for work PRs and reserve `master` for deployment
- 2026-08-20: added the prop-inheritance rule for UI component wrappers
- 2026-08-19: confirmed the Edu Vibe server TDD strategy and layer-specific test boundaries
- 2026-08-14: added file/directory naming (PascalCase, stated on Edu Vibe front) and the Identifiers section (prefixed string ids)
- 2026-08-03: migrated to English
