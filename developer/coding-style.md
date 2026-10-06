---
title: 코드 스타일 & 컨벤션 선호
area: developer
tags: [coding-style, conventions, commits]
created: 2026-08-03
updated: 2026-10-06
status: confirmed
---

# 코드 스타일 & 컨벤션 선호

> 실제로 사용 중인 ECC 룰셋(`~/.claude/rules/ecc/`)과 커밋 이력에서 추출했다.

## 핵심 원칙 (ECC 룰로 강제됨)

- **불변성 우선**: 제자리에서 변경하지 말고 새 복사본을 반환한다
- **KISS / DRY / YAGNI**: 동작하는 가장 단순한 해법. 실제 반복만 추상화. 필요보다 앞서 만들지 않는다
- **작은 파일 여러 개 > 큰 파일 몇 개**: 200~400줄이 보통, 최대 800줄. 기능/도메인 단위로 구성
- **깊은 중첩 금지**: 4단계 이상 대신 early return
- **매직 넘버 금지**: 의미 있는 임계값은 이름 있는 상수로
- **명시적 에러**: 조용히 삼키지 않는다. UI에는 친절한 메시지, 서버에는 상세 로그

## 네이밍

- 변수/함수는 `camelCase`, 불리언은 `is/has/should/can` 접두사
- 타입/컴포넌트는 `PascalCase`, 상수는 `UPPER_SNAKE_CASE`, 훅은 `use` 접두사
- **파일 & 디렉터리: TS/TSX 모듈과 그것을 담는 디렉터리는 PascalCase** — 예:
  `src/pages/ClassHome/ClassHomePage.tsx`, `src/Routes.tsx`. Edu Vibe 프론트에서 명시적으로 진술(2026-08-13).
  설정 파일과 CSS 파일은 소문자 유지 (`vite.config.ts`, `global.css`)

## 식별자

- MongoDB 내부의 `_id`는 자동 생성 ObjectId로 둔다. 애플리케이션/도메인 ID를 `_id`에 넣지 않는다.
  스키마는 서비스 조회, 참조, URL, API 응답용으로 별도의 문자열 `id` 필드를 노출한다.
- **접두사가 붙은 문자열 도메인 ID**를 선호한다 — `class-`, `proj-`, `arti-` + base36 10자
  (`class-pihtuke4hn`). 접두사 덕분에 로그와 URL에서 엔티티 타입이 보이고, DTO 검증에서 종류가 다른 ID를
  거를 수 있다. 이 선호는 MongoDB의 ObjectId `_id`를 대체하지 않는다. 두 식별자는 역할이 다르다.
- enum성 필드는 매직 넘버가 아니라 의미 있는 문자열을 담는다 (`0 | 1 | 2`가 아니라 `'up' | 'down' | null`)

## 커밋 & Git

- Conventional Commits (`feat:`, `fix:` 등) + **한국어 메시지**
  - 예: `fix: macOS 빌드에 ad-hoc 코드사인 추가, Gatekeeper 손상 오류 안내 추가`
- **Edu Vibe 브랜치 흐름**: 일상 작업은 feature 브랜치 → PR → `develop`. `develop` → `master`는 배포/릴리스에만

## UI 취향

- Linear/Notion 스타일의 미니멀한 UI를 선호한다 (Bulk Mail을 그 스타일로 다시 디자인했다)
- 토큰 기반 테마: 시드 색상(`--fg`/`--bg`)에서 `rgb(from ...)`으로 파생, 다크 모드는 `.dark` 클래스로
- 키트를 통째로 도입하기보다 Radix 기반 컴포넌트 세트를 직접 만드는 것을 선호한다
- **UI 라이브러리 래퍼는 감싼 컴포넌트의 props를 상속한다.** 예를 들어 Edu Vibe의 `CaretButton`은
  Vapor UI의 `Button`을 감싸므로 props가 `Button.Props`를 확장하고 나머지 props를 그대로 넘긴다.
  더 좁고 호환되지 않는 버튼 API를 새로 정의하지 않는다.
- **필요가 좁을 때는 라이브러리를 끌어오지 않고 작은 컴포넌트를 직접 쓴다.** INOS에서(2026-08-24)
  직접 작성한 마크다운 렌더러 + 툴바(제품이 지원하는 문법만, React 노드로 렌더해 raw HTML 주입이
  불가능하게)와 네이티브 `time` 입력 대신 손으로 만든 시/분 TimePicker를 채택했다. 둘 다 기존의
  2px 잉크 보더 디자인 시스템에 맞춰 스타일링했다.
- **크롬(chrome)이 콘텐츠보다 크게 말하면 안 된다.** INOS에서 반복된 교정(2026-08-24): 토론 발제문의
  순번 숫자를 줄이고(40px → 14px, muted), 독자가 어떤 책/영화를 보고 있는지 알 수 있도록 작품 제목을
  키우고, 발제문 본문은 semibold가 아니라 `font-normal`로.
- **상태 표시는 개수가 아니라 존재를 보여준다.** 읽지 않은 알림은 숫자 배지가 아니라 단순한 점으로
  렌더링하고, 정확한 개수는 스크린 리더를 위해 `aria-label`에 남긴다.
- **한국어 장문은 한 문단으로 감싸지 않고 문장 단위로 줄바꿈해 렌더링한다.** 여기에
  `whitespace-pre-wrap break-keep break-words` 3종 세트를 함께 쓴다 — 이 규칙을 만든 오버플로 함정은
  [[korean-text-wrapping]] 참고.

## 테스트

- **TDD 지향. 단 프론트는 테스트를 쓰지 않는다.**
  - 서버 API는 계약이 확실하기 때문에 TDD를 선호한다. ECC 룰의 "커버리지 80%" 목표는 서버에만 해당한다.
  - 프론트는 초기 개발에서 사람의 눈으로 확인하는 것이 더 싸고, 유지보수에서도 테스트 때문에
    공수가 2배가 된다고 본다.
  - 원칙은 [[philosophy]]의 "API는 TDD, 프론트는 사람의 눈", 적용 결정은 [[2026-09-09-edu-vibe-front-no-tests]].
- **Edu Vibe 서버 테스트 전략 (2026-08-19 진술)**: Jest로 아웃사이드-인 Red–Green–Refactor. 실패하는 API 계약/E2E 테스트에서 시작하고, 서비스 비즈니스 규칙은 리포지토리 목으로 커버하고, 각 리포지토리 메서드는 `mongodb-memory-server`로 실제 Mongoose 왕복을 통해 검증하고, 가드·검증·직렬화·HTTP 계약은 Supertest로 확인한다. 한 줄짜리 컨트롤러 위임까지 억지로 유닛 테스트하지 않는다.
- **GREEN은 계약의 모든 표현이 일치한다는 뜻이다** (Edu Vibe TDD 진행에서 확인, 2026-08-24):
  RED 기준선을 개수와 사유와 함께 기록하고, 구현 단계마다 건드린 spec만이 아니라 e2e 스위트 전체를
  돌리고, HTTP 동작·마이그레이션이 생성한 인덱스·직렬화된 응답 키·생성된 OpenAPI가 전부 일치하기 전까지는
  작업을 완료로 부르지 않는다. 테스트 기대값이 틀려 보인다면 구현에 맞춰 고치지 말고, 먼저 명세와 대조한다.

## 변경 이력

- 2026-10-06: 리뷰 중 정리 — 학생 참여 레코드 id 항목은 코드 스타일이 아니라는 사용자 판단으로 제거
  (해당 내용은 [[2026-09-09-edu-vibe-student-id-pii]]에 있음). TDD 범위를 사용자 진술대로
  "서버 API는 TDD, 프론트는 테스트 없음"으로. confirmed 승격
- 2026-09-01: 문서를 한국어로 전환
- 2026-08-24: INOS에서 관찰한 UI 규칙 네 가지 추가 — 작은 컴포넌트 직접 작성, 콘텐츠보다 조용한 크롬, 개수 대신 점 배지, 한국어 문장 단위 줄바꿈
- 2026-08-24: 접두사 ID는 별도의 도메인 `id`에 속하고 MongoDB `_id`는 자동 ObjectId로 남는다는 점 명확화
- 2026-08-24: Edu Vibe TDD 실행에서 배운 GREEN의 전체 계약 정의 추가
- 2026-08-20: Edu Vibe 브랜치 흐름을 작업 PR은 `develop`, `master`는 배포 전용으로 변경
- 2026-08-20: UI 컴포넌트 래퍼의 prop 상속 규칙 추가
- 2026-08-19: Edu Vibe 서버 TDD 전략과 레이어별 테스트 경계 확정
- 2026-08-14: 파일/디렉터리 네이밍(PascalCase, Edu Vibe 프론트에서 진술)과 식별자 섹션(접두사 문자열 id) 추가
- 2026-08-03: 영어로 전환
