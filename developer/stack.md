---
title: 기술 스택과 숙련도
area: developer
tags: [스택, TypeScript, React, Electron, NestJS]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# 기술 스택과 숙련도

> 실제 프로젝트 이력에서 추출. 숙련도 등급(상/중/하)은 본인 검토 필요.

## 주력 (실무 + 사이드에서 반복 사용)

| 기술 | 근거 |
|---|---|
| **TypeScript** (strict) | 모든 프로젝트 공통. 주 언어 |
| **React 19** | Bulk Mail, INOS web, 실무(edu-core 계열) |
| **Electron** | Bulk Mail — electron-vite + electron-builder로 DMG 배포까지 완주 |
| **Tailwind CSS v4** | Bulk Mail(`@theme inline`, `rgb(from ...)` 토큰 파생), INOS(DaisyUI) |
| **NestJS + Fastify** | INOS server/ai-server — SSE 스트리밍 포함 |
| **Prisma + PostgreSQL** | INOS — pgvector 벡터 검색, 모노레포 공유 스키마 |

## 실무 도메인 특화

- **실시간 동시편집**: Yjs, hocuspocus, y-prosemirror, ProseMirror/TipTap — 라이브러리 패치 경험 ([[y-prosemirror-nodeselection-crash]])
  - 구름 `goorm-hocuspocus` + `edu-core` — **epoch 기반 문서 버저닝** 설계/운영
- **에디터/블록 시스템**: 구름 edu-core, mist-blocks-react (추정)
- **사내 AI 인프라 연동**: 구름 내부 LiteLLM 프록시에 Claude Code 연결, GitHub MCP를 HTTP 엔드포인트로 구성

## 도구·인프라

- pnpm workspace + Turborepo 모노레포
- BullMQ + ioredis (큐), JWT + Passport + Google OAuth
- TanStack Router/Query, Radix UI, CVA, sonner
- nodemailer, OpenAI API (Images, 스트리밍)
- Claude Code 파워유저 — 스킬/에이전트/훅/MCP 대량 운용

## 환경

- **MacBook Pro (Mac15,6) / Apple M3 Pro** — CPU 11코어(성능 5 + 효율 6), GPU 14코어, RAM 18GB, macOS arm64
  - 메모리 18GB는 모노레포 동시 빌드·로컬 DB·Electron 동시 구동 시 병목 — 동시 프로세스 개수 주의
- 디스플레이: LG 울트라와이드 2560x1080 메인 + 내장 Liquid Retina XDR (상시 듀얼 모니터 환경)

## 변경 이력

- 2026-08-03: 실무 동시편집 상세(epoch 버저닝, LiteLLM 프록시) 및 정확한 개발 환경 사양 반영

## TODO

- 각 기술 숙련도 자가 평가 (상/중/하), 연차·경력 기간
- 다뤄봤지만 주력이 아닌 것들 (예: 다른 언어/프레임워크)
