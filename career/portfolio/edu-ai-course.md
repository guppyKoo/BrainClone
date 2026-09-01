---
title: edu-ai-course — AI 맞춤형 교육 플랫폼
area: career
tags: [portfolio, goorm, AI, education, React, oRPC]
created: 2026-09-01
updated: 2026-09-01
status: draft
---

# edu-ai-course — AI 맞춤형 교육 플랫폼

사용자 온보딩을 바탕으로 AI가 코스를 생성하고, 이론·퀴즈·코딩 실습·AI 튜터를 제공하는 goorm의
교육 플랫폼. 저장소명은 `edu-ai-course`, 로컬 디렉터리와 이전 원격 저장소명은 `new-edu`다.

## 참여 범위

- Git 이력 기준 2025-11부터 2026-04까지 `guppy.koo` 이름으로 약 500개 commit에 참여했다.
  merge와 중복 이력이 포함되므로 이를 그대로 성과 개수로 사용하지 않는다.
- (추론) 한 화면이나 한 레이어에 한정되지 않고 학습 UI, API, 테스트, 공용 UI, 모킹,
  AI 코스 편집 도구까지 제품의 세로 단면을 넓게 맡았다.

## 확인되는 주요 작업

- 코스 수강 화면의 진행 상태, 잠김/완료 처리, 현재 레슨 이동, 곡선형 커리큘럼 UI를 구현했다.
- 온보딩 레이아웃과 높이 기반 breakpoint, 전환 animation container를 만들었다.
- 코스 생성·커리큘럼 갱신 API와 course/project schema를 다뤘다.
- 공용 `packages/ui` 구조와 icon/component export를 정리하고 Vapor UI 1.1 마이그레이션을 진행했다.
- SSE 흐름과 course/onboarding utility에 Vitest 테스트를 추가하고 mock data 구조를 정비했다.
- 국방 AI 데모의 코스 생성·커리큘럼 재구성 UI, 이후 강좌 편집 도구까지 확장했다.
- Google Sheets 기반 콘텐츠 입력에서 JSON parsing, feature 단위 update, 재업로드 시 DB ID 재사용과
  upsert를 구현했다.
- 편집 충돌을 막기 위해 코스 단위 동시 편집 제한을 추가했다.

## 기술적으로 다시 이야기할 문제

### 외부 편집 키와 내부 DB ID를 분리

Google Sheets의 사람이 관리하는 ID를 MongoDB ID로 바로 쓰지 않았다. 최초 업로드에서 DB ID를 만들고
시트의 `(DB)` 컬럼에 다시 기록한 다음, 재업로드에서는 그 ID를 읽어 `bulkWrite` upsert에 재사용했다.
project와 course는 같은 suffix를 써 관계를 결정적으로 복원하고, lesson/step도 별도 ID map으로 참조를
재구성했다. 외부 원본을 반복 import해도 내부 참조가 끊기지 않는 ETL 패턴이다. 상세는
[[spreadsheet-import-stable-ids]].

### 편집 충돌을 기능 경계에서 차단

관리자 강좌 편집 도구에 코스 단위 lock을 추가했다. Socket.IO room으로 상태를 알리고, socket ID를
소유권으로 삼아 다른 사용자의 편집을 overlay로 차단하며 disconnect 때 lock을 해제했다.

현재 구현은 process memory의 `Map`을 사용하므로 **단일 서버 process에서만 유효하다**. 다중 instance로
확장하면 Redis 같은 공유 저장소, TTL/heartbeat, 원자적 acquire가 필요하다. 포트폴리오에서는 이 조건까지
함께 말해야 한다.

### 제품 전반의 UI 기반을 한 번에 이관

Vapor UI 1.1 업데이트에서 약 90개 파일을 함께 바꾸며 component 사용법, token, toast, stack layout을
새 API로 정리했다. 단순 버전 bump가 아니라 사내 design system skill의 component/token mapping 문서도
같이 갱신해 이후 작업 규칙까지 맞췄다.

## 포트폴리오 표현 초안 (추론)

- 계약 공유형 TypeScript 모노레포에서 코스 학습 UI, 온보딩, Express/oRPC API, 관리자 편집 도구까지
  제품 기능을 end-to-end로 개발했다.
- Google Sheets 재업로드에서도 기존 DB ID를 보존하는 upsert 경로를 만들어 외부 입력 갱신과 내부 참조의
  안정성을 함께 지켰다.
- 코스 단위 동시 편집 lock을 Socket.IO로 구현해 충돌을 막고, 연결 종료 시 자동 해제되는 수명 주기를 구성했다.
- Vapor UI 메이저 사용 패턴 변경을 제품 전반에 적용하면서 공용 컴포넌트와 design system 작업 규칙을 함께 정리했다.

정량 효과와 본인의 공식 역할명은 저장소만으로 확인할 수 없어 넣지 않았다.

## 변경 이력

- 2026-09-01: 저장소 구조와 `guppy.koo` commit 이력에서 참여 범위, 주요 작업, 면접용 문제 해결 서사를 추출
