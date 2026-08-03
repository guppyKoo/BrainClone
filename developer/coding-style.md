---
title: Code style & convention preferences
area: developer
tags: [coding-style, conventions, commits]
created: 2026-08-03
updated: 2026-08-03
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

## Changelog

- 2026-08-03: migrated to English
