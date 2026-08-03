---
title: INOS — 인문학 모임 플랫폼
area: career
tags: [포트폴리오, NestJS, AI, 모노레포, 사이드프로젝트]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# INOS — 인문학의 OS

인문학 모임 플랫폼. 영화/책 그룹 선택, AI 발제문 생성(SSE 스트리밍), 모임 일정 조율, 아카이빙.
개인 프로젝트 (`~/Practice/INOS`).

## 기술 구성

- **모노레포**: pnpm workspace + Turborepo
  - `apps/server` (3000): NestJS + Fastify — Auth/Group/Content/Schedule/Archive API
  - `apps/ai-server` (3001): NestJS + Fastify — SSE 스트리밍 발제문·추천·요약 전담
  - `apps/web` (5173): React 19 + Vite + Tailwind v4 + DaisyUI
  - `packages/prisma|types|utils`: 공유 스키마·DTO·유틸
- **데이터**: Prisma + PostgreSQL(Supabase) + **pgvector** — `$queryRaw` 벡터 검색
- **인프라**: BullMQ + ioredis 큐, JWT + Passport + Google OAuth

## 설계 포인트

- AI 트래픽(장시간 SSE)을 별도 서버로 분리해 API 서버와 격리
- Prisma 스키마를 패키지로 공유(symlink)해 앱 간 타입 일관성 확보
- NestJS `@Sse()` 데코레이터 기반 스트리밍

## 어필 포인트

- 취미(인문학 모임)의 실제 페인포인트를 풀스택 + AI로 해결한 프로젝트
- 벡터 검색·큐·OAuth 등 프로덕션급 구성 요소를 개인 프로젝트에서 직접 설계

## TODO

- 현재 진행 상태(개발 중/운영 중), 사용자 수, 데모 URL
