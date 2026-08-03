---
title: Bulk Mail — Electron desktop app
area: career
tags: [portfolio, Electron, React, side-project]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# Bulk Mail (Post Office) — Electron desktop app

macOS bulk-mail desktop app. Personal project, solo-built.
Repos: github.com/guppyKoo (original: Bulk-email → migration: bulk-mail-electron)

## Problem & story

- First built on Glaze (low-code platform), which **couldn't produce a DMG** → decided in 2026-07 to rebuild on vanilla Electron
- Migrated in 3 phases from an electron-vite + electron-builder skeleton: backend → UI components → renderer
- **Core goal achieved**: arm64/x64 DMG builds + verified packaged-app run
- v1.1.0 since: Linear/Notion-style UI redesign, recipient tags, ad-hoc codesigning for the Gatekeeper issue

## Tech

- **Stack**: Electron 34, React 19, Tailwind v4, TanStack Router/Query, Radix, nodemailer, OpenAI Images API
- **Architecture**: main process split into 3 services (settings/mail/image) + IPC handlers + multi-window manager (broadcast)
- **UI**: ~30 hand-built Radix + CVA components, design tokens (`rgb(from ...)` derivation) + dark mode
- **Features**: rich text editor, recipient management (tags/sidebar), AI image generation, send-result report, settings window (⌘,)

## Talking points

- When the platform blocked the goal, swapped the entire stack and **finished the full migration + release solo in a short window** (started 2026-07-13)
- Incremental migration strategy passing typecheck/build/dev at every phase

## Changelog

- 2026-08-03: migrated to English
