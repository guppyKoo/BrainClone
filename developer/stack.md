---
title: 기술 스택 & 숙련도
area: developer
tags: [stack, JavaScript, TypeScript, React, Node.js, Express, oRPC, Electron, NestJS]
created: 2026-08-03
updated: 2026-09-01
status: draft
---

# 기술 스택 & 숙련도

> 실제 프로젝트 이력에서 추출했다.

## 코어 (실무 + 사이드 프로젝트에서 반복 사용)

| 기술 | 근거 |
|---|---|
| **TypeScript** (strict) | 모든 프로젝트 공통. 주력 언어 |
| **Node.js + Express** | 프론트와 백엔드를 모두 다루는 JavaScript 풀스택 개발자로서 React와 함께 주로 사용 |
| **React 19** | Bulk Mail, INOS 웹, 실무(edu-core 계열), Edu Vibe 프론트. `react-router-dom`은 v7에서 멈췄지만 peer가 `react: >=18`이라 19에서도 돈다 — 패키지 이름 변경을 강제하는 건 `react-router` **v8**뿐 |
| **Electron** | Bulk Mail — electron-vite + electron-builder로 DMG 배포까지 완주 |
| **Tailwind CSS v4** | Bulk Mail (`@theme inline`, `rgb(from ...)` 토큰 파생), INOS (DaisyUI) |
| **NestJS + Fastify** | INOS server/ai-server — SSE 스트리밍 포함 |
| **Prisma + PostgreSQL** | INOS — pgvector 검색, 모노레포 공유 스키마 |
| **NestJS + Express + Mongoose/MongoDB** | Edu Vibe 서버 (2026-08) — 전역 `ValidationPipe`/`ClassSerializerInterceptor` 방어, 리포지토리 레이어, `migrate-mongo` 마이그레이션. 함정: [[nestjs-mongoose-pitfalls]] |
| **Express 5 + oRPC + Inversify + Mongoose** | goorm `edu-ai-course` — Zod/oRPC 계약 공유, feature별 router/service/module, DI container, MongoDB 트랜잭션. 구조: [[edu-ai-course-architecture]] |

## 실무 도메인 전문 영역

- **실시간 협업 편집**: Yjs, hocuspocus, y-prosemirror, ProseMirror/TipTap — 라이브러리를 직접 패치했다 ([[y-prosemirror-nodeselection-crash]])
  - goorm `goorm-hocuspocus` + `edu-core` — **epoch 기반 문서 버저닝**을 설계하고 운영 중
- **에디터/블록 시스템**: goorm edu-core, mist-blocks-react (추론)
- **사내 AI 인프라**: Claude Code를 goorm 사내 LiteLLM 프록시에 연결. GitHub MCP를 HTTP 엔드포인트로 연동

## 도구 & 인프라

- pnpm workspace + Turborepo 모노레포
- BullMQ + ioredis (큐), JWT + Passport + Google OAuth
- LangChain/LangGraph, OpenAI·Anthropic 멀티 provider LLM 구성, retry/backoff와 token·latency 계측 ([[llm-rate-limit-defense]])
- TanStack Router/Query, Radix UI, CVA, sonner
- nodemailer, OpenAI API (Images, 스트리밍)
- Claude Code 파워 유저 — skill/agent/hook/MCP를 대규모로 운용
- 배포/인프라: EKS 기반 Kubernetes, Jenkins 파이프라인

## 개발 환경

- **MacBook Pro (Mac15,6) / Apple M3 Pro** — 11코어 CPU (5P+6E), 14코어 GPU, 18GB RAM, macOS arm64
  - 모노레포 동시 빌드 + 로컬 DB + Electron을 함께 돌릴 때 18GB가 병목이다. 동시 프로세스 수를 주의할 것
- 디스플레이: LG 울트라와이드 2560x1080 메인 + 내장 Liquid Retina XDR (항상 듀얼 모니터)

## 변경 이력

- 2026-09-01: 프로젝트 지침에서 확인된 주력 역할과 기술을 반영해 JavaScript 풀스택, React, Node.js(Express)를 명시
- 2026-09-01: `edu-ai-course`에서 반복 사용한 Express 5 + oRPC + Inversify와 LangChain/LangGraph 운영 패턴 추가
- 2026-09-01: 문서를 한국어로 전환
- 2026-08-19: 이전 오류 정정 — `react-router-dom` v7은 React 18을 강제하지 않는다 (peer가 `>=18`). Edu Vibe 프론트를 React 19로 이동
- 2026-08-14: NestJS+Express+Mongoose/MongoDB (Edu Vibe) 추가, React 행 정정
- 2026-08-03: 실무 협업 편집 상세(epoch 버저닝, LiteLLM 프록시)와 정확한 머신 스펙 추가
- 2026-08-03: 영어로 전환
