---
title: Bulk Mail — Electron desktop app
area: career
tags: [portfolio, Electron, React, side-project]
created: 2026-08-03
updated: 2026-08-07
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

## Actual usage (dogfooding)

- Used continuously since **2026-04** to send a personal daily/weekly letter to friends — the app's real proving ground, not a demo.
- Current list: **7 recipients**, including the user's own work address (self-check copy).
- Cadence: irregular but roughly 2–4 times a week, sent on weekday afternoons (KST).
- Started as a one-to-one letter (addressed to a single person by name) and grew into a group send.
- Early sends carried a fixed sign-off asking recipients not to block the address; later ones dropped it.
- Recipients reply to the letters, so each send turns into individual threads — the letter functions as a conversation starter.
- **Known incident (2026-07-14)**: a mail already sent on 2026-07-01 went out again unprompted, and the user
  sent a follow-up saying they had not triggered it. Suggests a re-send/scheduling bug worth reproducing.
- **Known incident (2026-08-10, 13:35 KST)**: another unprompted send. Evidence points to a **legacy script, not the app** —
  it went to a single address (the user's own work account) instead of the 7-person list, and its body carries the
  April-era sign-off template the app stopped using in May. The user suspects a leftover macOS Automator job.
  Note the two incidents differ in shape (7/14 re-sent a real campaign to all 7; 8/10 sent a fixed debug template to one),
  so they may be two separate causes.

## Direction — public subscription letter (decided 2026-08-07)

- Turning the personal letter into an **opt-in newsletter**: anyone can subscribe, and the letter lands in their inbox.
- Concept: a deliberately useless, funny letter arriving in the early-afternoon slump at work or school.
- **Subscribe and unsubscribe both handled by Instagram DM**, processed manually — no signup form, no database.
  MVP-first, on the principle that shipping beats polishing.
- Deferred until volume demands it: automated subscribe/unsubscribe, dedicated sending domain with SPF/DKIM/DMARC,
  an SMTP relay instead of personal Gmail, and a threshold for when to switch.
- Open item: put unsubscribe instructions in the mail footer before taking public signups, and settle the
  unprompted-send incidents first.

## Talking points

- When the platform blocked the goal, swapped the entire stack and **finished the full migration + release solo in a short window** (started 2026-07-13)
- Incremental migration strategy passing typecheck/build/dev at every phase

## Changelog

- 2026-08-10: logged the 2026-08-10 unprompted send (legacy-script evidence); added the public-subscription direction

- 2026-08-07: added real-usage section (7-recipient personal letter since 2026-04) and the 2026-07-14 duplicate-send incident
- 2026-08-03: migrated to English
