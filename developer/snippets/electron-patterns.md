---
title: Electron patterns from real projects
area: developer
tags: [Electron, electron-vite, electron-builder, patterns]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# Electron patterns

Patterns proven during the [[bulk-mail-electron]] migration.

## Project skeleton

- **electron-vite + electron-builder** was the smoothest path from scaffold to DMG
- Structure: `main/services/*` (domain services) + `main/handlers` (IPC) + `window-manager` (multi-window + broadcast)
- Preload exposes a single `window.api` (invoke/on) bridge — the renderer never touches IPC channels directly

## Pitfalls & fixes

- **safeStorage is synchronous**: don't `await` it out of web habit (sync API in Electron)
- **Dark mode sync**: `nativeTheme` `updated` event → broadcast to all windows → renderer hook toggles `.dark`.
  Theme setting persisted to `userData/*.json`
- **Unsigned macOS distribution**: building with `identity: null` triggers Gatekeeper "damaged" errors →
  add ad-hoc codesigning + user guidance (fixed in v1.1.x)
- **Settings window**: menu `⌘,` → settings window; changes reach the main window via broadcast

## Build

- `npm run build:mac` → arm64 + x64 DMGs (~94–99MB)
- App icon: place `build/icon.icns`

## Changelog

- 2026-08-03: migrated to English
