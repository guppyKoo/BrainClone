---
title: AI 교육 플랫폼의 계약 공유형 모노레포 구조
area: developer
tags: [monorepo, oRPC, contracts, Inversify, MSW, architecture]
created: 2026-09-01
updated: 2026-09-01
status: draft
---

# AI 교육 플랫폼의 계약 공유형 모노레포 구조

goorm `edu-ai-course`(`new-edu`)에서 반복 사용 중인 경계와 확장 패턴. 제품은 온보딩 정보를 받아
AI 코스를 생성하고, 이론·퀴즈·코딩 실습·AI 튜터를 한 학습 흐름으로 제공한다.

## 구성과 의존 방향

```text
web ────────────────┐
main server ────────┼──> contracts
contents server ────┘

web ──> ui
web ──> mock-presets
main server <──oRPC──> contents server
```

- `apps/web`: React 19 + Vite. oRPC 클라이언트, TanStack Query, Monaco, Tiptap을 사용한다.
- `apps/server`: Express 5 + oRPC + Inversify + Mongoose. 인증, 코스, 레슨, 스텝, AI 튜터 등
  제품 API와 영속성을 소유한다.
- `apps/contents-server`: Express 5 + oRPC + LangGraph + BullMQ. 장시간 걸리는 코스 구조·콘텐츠 생성을 소유한다.
- `packages/contracts`: Zod + oRPC 계약. `main-server`, `contents-server`, `shared` 진입점을 분리한다.
- `packages/ui`: React peer dependency를 둔 소스 직접 참조형 UI 패키지.
- `packages/mock-presets`: MSW 응답 JSON을 endpoint/scenario 경로로 export하는 fixture 패키지.

핵심은 **계약 패키지만 독립적으로 두고 각 실행 앱이 계약을 구현하거나 소비하게 하는 것**이다.
웹과 서버가 별도 DTO를 복사하지 않고, main server와 contents server도 같은 도메인 스키마를 공유한다.

## 서버 기능 추가 패턴

한 기능은 보통 다음 네 경계를 함께 움직인다.

1. `packages/contracts/src/main-server/<feature>`에 Zod schema, DTO, oRPC contract를 정의한다.
2. `apps/server/src/features/<feature>`에 `service`, `router`, `module`을 둔다.
3. Inversify `ContainerModule`에서 router/service를 singleton으로 바인딩한다.
4. `OrpcService.createRouter()`가 컨테이너에서 router를 꺼내 전역 계약 구현에 합친다.

Procedure는 인증 경계를 이름으로 드러낸다.

- `publicProcedure`: 인증 없이 접근
- `authedProcedure`: 사용자 세션 필요
- `apiKeyProcedure`: 서버 간 API key 필요

Router는 HTTP/oRPC 입출력 변환, service는 비즈니스 규칙을 맡는다. DB 접근이 복잡한 기능은
repository를 한 단계 더 둔다. 공통 에러 계층은 상태 코드와 외부 메시지, 내부 진단 데이터를 분리하며,
운영 환경의 500 응답은 일반 문구로 마스킹한다.

## fixture를 패키지 경계로 두는 방식

`mock-presets`는 `course/get/info/default.json`, `auth/get/me/unauthenticated.json`처럼
**endpoint + scenario**를 파일 경로로 표현하고 웹의 MSW handler가 이를 소비한다. fixture가 handler 코드와
분리돼 테스트·Storybook·로컬 개발 같은 다른 소비자가 같은 시나리오를 가져다 쓸 수 있다.

계약은 모양을 보장하고 fixture는 대표 시나리오를 보장한다. 둘은 대체 관계가 아니다.
현재 서버 MSW handler는 이 패키지를 소비하지 않으므로, “프론트와 서버가 같은 fixture를 쓴다”고까지
말하면 안 된다. 양쪽 일치를 원한다면 서버도 패키지를 import하게 만들거나 contract에서 fixture를 검증해야 한다.

## 경계를 지킬 때 얻는 것

- API 변경이 소비자 type-check까지 전파된다.
- AI 생성 서버를 웹에서 직접 알 필요가 없다. main server가 상태와 저장을, contents server가 생성 작업을 맡는다.
- UI와 mock fixture를 workspace 패키지로 빼되, 독립 배포가 필요 없는 패키지는 빌드 산출물 없이 소스를 직접 참조할 수 있다.
- 기능 탐색 시 contract → router → service → model 순서로 읽으면 요청에서 저장까지 빠르게 추적할 수 있다.

## 현재 저장소에서 확인해야 할 드리프트

문서와 실제 workspace 선언이 다르다. 루트 설명은 `contents-server`를 모노레포 앱으로 소개하고
스크립트도 `p:con`을 제공하지만, 현재 `pnpm-workspace.yaml`에는 `apps/web`, `apps/server`, `packages/**`만 있다.
2026-04-17 커밋에서도 contents server를 workspace에서 제외했다.

재사용할 때는 README보다 아래 순서로 현재 사실을 확인한다.

1. `pnpm-workspace.yaml` — 실제 workspace membership
2. 각 `package.json`의 `exports`, scripts, dependency
3. `turbo.json` — 실제 task graph
4. 마지막으로 설명 문서

## 연결 문서

- [[llm-rate-limit-defense]] — contents server의 중첩 병렬 생성과 429 대응
- [[monaco-model-lifecycle]] — 학습창 Monaco 모델 소유권과 화면 전환 race condition
- [[spreadsheet-import-stable-ids]] — 반복 import에서 외부 행과 내부 엔티티의 ID를 안정적으로 연결
- [[edu-ai-course]] — 프로젝트 성과 서사

## 변경 이력

- 2026-09-01: 현재 저장소에서 재사용 가능한 앱 경계, contract-first 기능 추가 순서,
  fixture 패키지 패턴과 workspace 드리프트 점검법을 추출
