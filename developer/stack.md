---
title: Tech stack & proficiency
area: developer
tags: [stack, TypeScript, React, Electron, NestJS]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# Tech stack & proficiency

> Extracted from real project history.

## Core (used repeatedly at work + side projects)

| Tech | Evidence |
|---|---|
| **TypeScript** (strict) | Common to every project. Primary language |
| **React 19** | Bulk Mail, INOS web, work (edu-core family) |
| **Electron** | Bulk Mail — completed through DMG shipping with electron-vite + electron-builder |
| **Tailwind CSS v4** | Bulk Mail (`@theme inline`, `rgb(from ...)` token derivation), INOS (DaisyUI) |
| **NestJS + Fastify** | INOS server/ai-server — including SSE streaming |
| **Prisma + PostgreSQL** | INOS — pgvector search, monorepo shared schema |

## Work domain specialties

- **Realtime collaborative editing**: Yjs, hocuspocus, y-prosemirror, ProseMirror/TipTap — patched a library ([[y-prosemirror-nodeselection-crash]])
  - goorm `goorm-hocuspocus` + `edu-core` — designed/operates **epoch-based document versioning**
- **Editor/block systems**: goorm edu-core, mist-blocks-react (inferred)
- **In-house AI infra**: connected Claude Code to goorm's internal LiteLLM proxy; GitHub MCP over an HTTP endpoint

## Tools & infra

- pnpm workspace + Turborepo monorepo
- BullMQ + ioredis (queues), JWT + Passport + Google OAuth
- TanStack Router/Query, Radix UI, CVA, sonner
- nodemailer, OpenAI API (Images, streaming)
- Claude Code power user — large fleet of skills/agents/hooks/MCP
- Deploy/infra: EKS-based Kubernetes, Jenkins pipelines

## Environment

- **MacBook Pro (Mac15,6) / Apple M3 Pro** — 11-core CPU (5P+6E), 14-core GPU, 18GB RAM, macOS arm64
  - 18GB is the bottleneck for concurrent monorepo builds + local DB + Electron; watch concurrent process count
- Displays: LG ultrawide 2560x1080 main + built-in Liquid Retina XDR (always dual-monitor)

## Changelog

- 2026-08-03: added work collab-editing details (epoch versioning, LiteLLM proxy) and exact machine specs
- 2026-08-03: migrated to English
