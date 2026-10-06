---
title: 기술 스택 & 숙련도
area: developer
tags: [stack, JavaScript, TypeScript, React, Node.js, Express, oRPC, Electron, NestJS]
created: 2026-08-03
updated: 2026-10-06
status: confirmed
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
| **Prisma + PostgreSQL** | INOS — 모노레포 공유 스키마. pgvector는 써봤지만 오버엔지니어링이라 제거 (추천 시스템 구현 시 재도입 검토) |
| **NestJS + Express + Mongoose/MongoDB** | Edu Vibe 서버 (2026-08) — 전역 `ValidationPipe`/`ClassSerializerInterceptor` 방어, 리포지토리 레이어, `migrate-mongo` 마이그레이션. 함정: [[nestjs-mongoose-pitfalls]] |
| **Express 5 + oRPC + Inversify + Mongoose** | goorm `edu-ai-course` — Zod/oRPC 계약 공유, feature별 router/service/module, DI container, MongoDB 트랜잭션. 구조: [[edu-ai-course-architecture]] |

## 실무 도메인 전문 영역

- **실시간 협업 편집**: Yjs, hocuspocus, y-prosemirror, ProseMirror/TipTap — 라이브러리를 직접 패치했다 ([[y-prosemirror-nodeselection-crash]])
  - goorm `goorm-hocuspocus` + `edu-core` — **epoch 기반 문서 버저닝**을 설계하고 운영 중
- **에디터/블록 시스템**: goorm edu-core
- **사내 AI 인프라**: Claude Code를 goorm 사내 LiteLLM 프록시에 연결. GitHub MCP를 HTTP 엔드포인트로 연동

## 도구 & 인프라

- pnpm workspace + Turborepo 모노레포
- **Kafka**: 현 회사 제품의 application server에서 사용 중. producer/consumer를 애플리케이션에 연동해 본 경험은
  있지만 Kafka cluster를 직접 구축하거나 운영한 경험은 없다.
- **Redis**: 현 회사에서 캐싱 용도로 적극 사용하며, INOS에서는 BullMQ + ioredis 기반 기능을 직접 구현했다.
  Redis cluster 자체를 구축·운영한 경험과 대규모 캐시 최적화 경험은 별도로 확인되지 않았다.
- BullMQ + ioredis (큐), JWT + Passport + Google OAuth
- LangChain/LangGraph, OpenAI·Anthropic 멀티 provider LLM 구성, retry/backoff와 token·latency 계측 ([[llm-rate-limit-defense]])
- TanStack Router/Query, Radix UI, CVA, sonner
- nodemailer, OpenAI API (Images, 스트리밍)
- **Playwright**: Edu Vibe LLM eval에서 생성 HTML을 브라우저로 실행·관측 (시계·타임존 고정 `addInitScript`,
  `<select>` 조작 등) → [[2026-09-16-edu-vibe-eval-instrument-contract]]. 2026-10-06 interest에서 승격
- **Appsmith**: 회사에서 사내 도구용으로 사용 중 (JSONForm 위젯, API body 직렬화 이슈 경험). 2026-10-06 interest에서 승격
- Claude Code 파워 유저 — skill/agent/hook/MCP를 대규모로 운용
- 배포/인프라: EKS 기반 Kubernetes, Jenkins 파이프라인. ArgoCD, Vault, Istio(HTTPRoute), Elastic APM도 사용하지만
  전부 **이미 구축된 사내 플랫폼 위에서 쓰는 수준**이다. 직접 구축·운영한 경험은 없다.

## 테스트 & 품질 보증

- **Edu Vibe 서버**: spec-first TDD로 RED를 구현 갭 목록으로 사용하고, spec 327개와 e2e 94개까지
  확장했다. 실제 마이그레이션과 운영 전역 설정을 테스트에서도 공유하고, 테스트 DB 격리 실패가
  닫히는 쪽으로 동작하도록 하네스를 보강했다 ([[nestjs-e2e-test-harness]]).
- **edu-core**: 품질 보증과 고객 CS가 발생하기 전 오류를 선제적으로 검출하기 위한 e2e 테스트 작성 경험이 있다.
- 따라서 테스트는 새로 확보해야 할 기초 역량이 아니다. 다음 과제는 개인 프로젝트 INOS에도 자동화
  테스트를 적용하고, 운영 위험에 맞게 어떤 계층을 보호할지 결정하는 것이다.

## AI/Data Platform 전환 관점의 경험 경계

- **AI model serving**: 회사의 Edu Vibe와 `edu-ai-course`, 개인 프로젝트 INOS에서 모델을 서비스에
  연결하고 API·비동기 작업·스트리밍 흐름을 구현했다. 대규모 트래픽의 고가용성 서빙과 모델 배포
  플랫폼을 직접 구축한 경험은 부족하다.
- **Vector 검색**: INOS에서 PostgreSQL/pgvector를 써봤지만 오버엔지니어링이라 판단해 뺐다.
  나중에 추천 시스템을 구현할 때 다시 추가할 수 있다. RAG 파이프라인을 구축하거나
  검색 품질·성능을 최적화한 경험은 없다.
- **Python**: 코딩 테스트 풀이 경험이 많다. 프로덕션 AI/Data 애플리케이션 개발 경험으로 보기는 어렵다.
- **JVM**: Java를 학교 수업에서 사용한 정도이며 실무 경험은 많지 않다. NestJS의 모듈·DI·데코레이터 기반
  구조 경험을 Spring 학습의 기반으로 활용할 수 있다.
- **Airflow**: 사용 경험이 없다.
- **핀테크**: 결제·송금 등 핀테크 도메인 경험과 지식이 아직 부족하다.

## 개발 환경

- **MacBook Pro (Mac15,6) / Apple M3 Pro** — 11코어 CPU (5P+6E), 14코어 GPU, 18GB RAM, macOS arm64
  - 모노레포 동시 빌드 + 로컬 DB + Electron을 함께 돌릴 때 18GB가 병목이다. 동시 프로세스 수를 주의할 것
- 디스플레이: LG 울트라와이드 2560x1080 메인 + 내장 Liquid Retina XDR (항상 듀얼 모니터)

## 변경 이력

- 2026-10-06: Playwright·Appsmith를 [[interest]]에서 승격 (사용자 확인)
- 2026-10-06: confirmed 승격. mist-blocks-react(추론) 제거, pgvector를 "사용 후 오버엔지니어링으로 제거"로 정정,
  배포 스택(ArgoCD·Vault·Istio·APM)을 "구축된 플랫폼 위 사용" 수준으로 명시
- 2026-09-10: Edu Vibe의 TDD·e2e와 edu-core의 선제적 오류 검출 e2e 경험을 품질 보증 역량으로 명시
- 2026-09-09: Kafka·Redis의 회사 사용 경험과 구축 경험의 경계, AI model serving·Python·JVM 및
  Airflow/RAG/핀테크 경험 수준을 사용자 확인 내용으로 추가
- 2026-09-01: 프로젝트 지침에서 확인된 주력 역할과 기술을 반영해 JavaScript 풀스택, React, Node.js(Express)를 명시
- 2026-09-01: `edu-ai-course`에서 반복 사용한 Express 5 + oRPC + Inversify와 LangChain/LangGraph 운영 패턴 추가
- 2026-09-01: 문서를 한국어로 전환
- 2026-08-19: 이전 오류 정정 — `react-router-dom` v7은 React 18을 강제하지 않는다 (peer가 `>=18`). Edu Vibe 프론트를 React 19로 이동
- 2026-08-14: NestJS+Express+Mongoose/MongoDB (Edu Vibe) 추가, React 행 정정
- 2026-08-03: 실무 협업 편집 상세(epoch 버저닝, LiteLLM 프록시)와 정확한 머신 스펙 추가
- 2026-08-03: 영어로 전환
