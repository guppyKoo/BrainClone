---
title: Code style & convention preferences
area: developer
tags: [coding-style, conventions, commits]
created: 2026-08-03
updated: 2026-08-19
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

- Prefers **prefixed string ids over ObjectId or bare UUID** — `class-`, `proj-`, `arti-` + 10 base36 chars
  (`class-pihtuke4hn`). Reason: the value alone says what it is in logs/URLs, and a wrong-kind id can be
  rejected at the DTO boundary. ObjectIds are all 24-char hex, so a mismatch only surfaces as an empty result
- Enum-ish fields carry meaningful strings, never magic numbers (`'up' | 'down' | null`, not `0 | 1 | 2`)

## Commits & Git

- Conventional Commits (`feat:`, `fix:` …) with **Korean messages**
  - e.g. `fix: macOS 빌드에 ad-hoc 코드사인 추가, Gatekeeper 손상 오류 안내 추가`
- Feature branch → PR → main (e.g. `ui-redesign-v1.1.0`, `fix/adhoc-signing`)

## UI taste

- Prefers minimal Linear/Notion-style UI (redesigned Bulk Mail in that style)
- Token-based theming: seed colors (`--fg`/`--bg`) derived via `rgb(from ...)`, dark mode via `.dark` class
- Prefers building his own Radix-based component set over adopting a kit wholesale

## Testing

- TDD-oriented, 80% coverage target (ECC rules)
- **Edu Vibe server testing strategy (stated 2026-08-19)**: use Jest with outside-in Red–Green–Refactor. Start from a failing API contract/E2E test, cover service business rules with repository mocks, verify each repository method through real Mongoose round trips with `mongodb-memory-server`, and use Supertest for guards, validation, serialization, and HTTP contracts. Do not force unit tests for one-line controller delegation.

## Changelog

- 2026-08-19: confirmed the Edu Vibe server TDD strategy and layer-specific test boundaries
- 2026-08-14: added file/directory naming (PascalCase, stated on Edu Vibe front) and the Identifiers section (prefixed string ids)
- 2026-08-03: migrated to English
