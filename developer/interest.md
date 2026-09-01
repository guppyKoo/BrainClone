---
title: 개발 관심사
area: developer
tags: [interests, AI, agents, CRDT, learning, database-design, PostgreSQL, MongoDB, cdc, C, open-source, translation]
created: 2026-08-03
updated: 2026-09-01
status: draft
---

# 개발 관심사

호기심/학습 단계의 개발 주제. **실무에서 반복 사용하게 되면 [[stack]]으로 승격**하고,
여기에는 "더 깊이 파고 싶은" 항목만 남긴다. (비개발 관심사는 `interest/`에 있다.)

## AI 에이전트 & Claude Code 생태계

가장 활발한 주제. 단순 사용을 넘어 **개발 워크플로 자체를 에이전트 중심으로 재설계하는 중**.

- 현재: 대규모 ECC 룰/skill/agent 운용, hook 기반 품질 게이트(GateGuard), 다수의 MCP 서버(Figma/Notion/MongoDB/GitHub 등), AI가 읽고 쓰는 지식베이스로서의 BrainClone 설계
- **클라이언트 간 BrainClone 통합** (2026-08-18): Claude Code, Codex CLI, ChatGPT가 같은 BrainClone을 단일 진실 공급원으로 공유하기를 원한다. OKF 구조와 Git 이력을 보존하면서 ChatGPT가 로컬 지식베이스를 읽고 쓸 방법을 탐색 중
- 하위 관심: 멀티 에이전트 오케스트레이션, PreToolUse/PostToolUse hook으로 AI 행동 강제하기, 세션 간 메모리/컨텍스트 관리
- 다음: Claude Agent SDK 커스텀 에이전트, 에이전트 평가(eval) 시스템
- **에이전트 간 컨텍스트 공유** (2026-08-14): 에이전트 A가 에이전트 B의 컨텍스트를 볼 수 있는지 물었다.
  결론은 메커니즘 쪽이었다 — 읽을 수 있는 공유 메모리는 없고, B를 "본다"는 것은 결국 B의 히스토리를
  A의 프롬프트에 주입하는 것으로 환원된다. 그래서 진짜 설계 질문은 얼마나, 언제, 무엇을 빼고 넣을지다
  (B의 실패한 시도를 그대로 재생하면 A에게는 검증된 컨텍스트가 되어버린다). Claude Code의 서브에이전트는
  부모↔자식 관계만 있고, 형제끼리는 공유 파일을 통해 조율한다.

## 실시간 협업 편집 (CRDT / Yjs)

실무에서 숙련됨 — 스택 자체는 [[stack]]에 있다. 남은 호기심:

- Yjs 내부 구조(item 병합, GC), 다른 CRDT와의 비교(Automerge, Loro)

## 데이터베이스 설계

- 별도 ChatGPT 프로젝트 `데이터베이스설계`를 두고 있다. 현재 로컬 미러에는 첨부 자료와 과거 대화가
  없어 구체적인 학습 목표·진도는 확인되지 않았다.
- 이미 다뤄 온 구현 축은 **Prisma + PostgreSQL/pgvector**와 **Mongoose + MongoDB**다. 단순 CRUD보다
  스키마·식별자·인덱스·마이그레이션·일관성 보장을 함께 설계하는 문제에 관심이 이어진다.
- 현재 연결되는 설계 주제:
  - MongoDB `_id`와 외부에 노출하는 문자열 도메인 `id`의 역할 분리 ([[coding-style]])
  - `autoIndex: false` 환경에서 실제 마이그레이션을 테스트 DB에도 실행해 인덱스 드리프트를 잡는 법
    ([[nestjs-e2e-test-harness]])
  - 동일 밀리초의 `createdAt`만으로 정렬하지 않고 `_id`를 최종 타이브레이커로 두는 안정적 정렬
    ([[nestjs-e2e-test-harness]])
  - 같은 DB 안의 원자성은 다중 문서 트랜잭션으로, 서비스 경계를 넘으면 Transactional Outbox와
    이벤트로 해결하는 캐스케이드 삭제 전략 ([[mongodb-cascade-strategies]])
  - 앱 코드의 관례보다 DB 엔진이 무결성을 강제하는 설계를 선호한다 ([[philosophy]])

## 학습 단계 기술 (호기심 ~ 초급)

- **오픈소스 SW 수업** — 영어로 된 강의자료를 한국어로 번역하며 학습한다. 현재 프로젝트는 강의자료의 핵심 의미와 기술 용어를 보존한 한국어 번역을 지원하는 용도다
- **C 언어** — 별도 ChatGPT 프로젝트 `C언어`를 두고 있다. 현재 로컬 미러에는 참고 자료가 없어 구체적인 학습 목표·진도·숙련도는 확인되지 않았다.
- **Python 코딩 테스트** — Python으로 코딩 테스트 문제를 풀이하는 프로젝트를 진행한다
- **Flutter** — React/JS 개념에 대응시킨 학습 로드맵을 만들었다. 모바일까지 커버하는 게 목표로 보인다
- **Playwright** — 브라우저 자동화 / E2E
- **AWS SAA-C03** — 10~12주 학습 계획 초안 작성 (진척은 미확인)
- **Appsmith** — 사내 도구용으로 조사 (JSONForm 위젯, API body 직렬화 이슈)
- **로컬 AI 애플리케이션** — 로컬 OpenAI 호환 챗 서버를 돌려봤고, 프롬프트 커스터마이징과 LangChain(경우에 따라 LangGraph)으로 페르소나를 바꿀 수 있는 챗 애플리케이션을 만들고 싶어 한다
- **Godot 게임 개발** — 비주얼 노벨식 진행, RPG 요소, 턴제 전략을 결합한 스토리 중심 인디 게임을 만들면서 게임 개발을 배우고 싶어 한다

## 메시징/인프라 주변

- Kafka, BullMQ, Supabase vs Firebase, Turborepo 리모트 캐싱 — 실무 협업 편집 맥락에서 비교했다
- **MongoDB Change Streams / CDC** (2026-08-14) — 레거시 Kafka 캐스케이드 경로를 더 가벼운 것으로
  대체할 방법을 찾다가 깊이 파고들었다. 전체 노트: [[mongodb-cascade-strategies]].
  아직 안 밟은 인접 영역: Debezium, Transactional Outbox, Atlas Triggers, Redis Streams vs Pub/Sub.

## 만들고 싶은 것

- (추론) Claude Agent SDK 기반 개인 에이전트 — BrainClone을 읽는 "디지털 트윈"
- (추론) Bulk Mail 정식 릴리스 — 코드사이닝/공증까지
- (추론) INOS 공개 런칭 — 현재 홍보와 초기 사용자 확보를 고민 중
- MCP로 Claude Code ↔ Claude Design을 잇는 디자인→구현 파이프라인
- 모델 페르소나를 설정할 수 있는 로컬 AI 챗 애플리케이션
- 백년 전쟁 이후를 배경으로 세 개의 플레이 가능한 세력과 분기하는 전략적 선택지를 가진, 혼자 개발하는 정치 판타지 게임

## 변경 이력

- 2026-09-01: `데이터베이스설계` 프로젝트와 기존 DB 설계 지식의 연결 지도를 추가; 자료 부재로 세부 목표와 진도는 유보
- 2026-09-01: 오픈소스 SW 수업 수강과 영어 강의자료 번역 학습을 추가
- 2026-09-01: ChatGPT의 `C언어` 프로젝트 존재를 학습 관심사로 기록; 자료 부재로 세부 목표와 진도는 유보
- 2026-09-01: Python 코딩 테스트 문제 풀이 프로젝트 추가
- 2026-09-01: 문서를 한국어로 전환
- 2026-08-18: 이전 ChatGPT 대화에서 로컬 AI 챗과 Godot 턴제 서사 게임 관심사 추가
- 2026-08-18: Claude Code, Codex CLI, ChatGPT를 아우르는 BrainClone 연결 계획 기록
- 2026-08-14: MongoDB Change Streams / CDC와 에이전트 간 컨텍스트 공유 추가
- 2026-08-03: interest/topics (ai-agent-tooling, realtime-collaboration) + 개발 위시리스트 항목을 병합해 생성
- 2026-08-03: 학습 단계 기술(Flutter, Playwright, AWS SAA-C03, Appsmith)과 인프라 조사 이력 추가
- 2026-08-03: 영어로 전환
