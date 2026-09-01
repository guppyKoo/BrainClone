---
title: React 화면 전환에서 Monaco 모델 수명 관리
area: developer
tags: [Monaco, React, lifecycle, race-condition, debugging]
created: 2026-09-01
updated: 2026-09-01
status: draft
---

# React 화면 전환에서 Monaco 모델 수명 관리

외부 hook이 Monaco text model을 캐시하는데 `@monaco-editor/react`도 언마운트 시 같은 모델을 정리하면
소유권이 겹친다. background refetch가 에디터 트리까지 언마운트시키면 이 충돌이 race condition으로 변해
`Model is disposed!`가 간헐적으로 발생한다.

## 실패 구조

```text
외부 cache가 model 생성·보관
  → Editor가 model을 장착
  → refetch/loading 분기가 Editor 트리를 언마운트
  → Editor 기본 cleanup이 model.dispose()
  → 비동기 remount/onMount가 이전 model을 다시 setModel()
  → Model is disposed!
```

`@monaco-editor/react`의 `Editor`는 `keepCurrentModel=false`가 기본이라 언마운트 때 현재 모델을 dispose한다.
`DiffEditor`도 modified/original model에 별도 보존 옵션이 있다. 외부 cache가 undo/redo 기록을 위해 모델을
소유한다면 wrapper의 기본 cleanup과 충돌한다.

## 재사용 가능한 해결 규칙

### 소유자는 하나만 둔다

- 외부 cache가 model을 소유하면 `keepCurrentModel`을 켠다.
- Diff editor는 실제로 외부 관리하는 쪽에 `keepCurrentModifiedModel` 또는 `keepCurrentOriginalModel`을 켠다.
- wrapper에는 모델이 있을 때만 보존 옵션을 켜, 내부 생성 모델까지 무조건 누수시키지 않는다.

### 모든 경계에서 disposed 상태를 검사한다

- cache hit 반환 전 `!model.isDisposed()`
- `monaco.editor.getModel(uri)` 재사용 전 `!existing.isDisposed()`
- `editor.setModel(model)` 직전 `!model.isDisposed()`
- cache 전체 정리 시에도 중복 dispose를 피한다.

### URI를 안정적인 모델 identity로 쓴다

파일 경로를 정규화한 뒤 `file:///...` URI로 모델을 생성한다. 동일 파일이 새 객체로 들어와도
`monaco.editor.getModel(uri)`로 기존 모델을 찾을 수 있고, 파일별 undo/redo를 유지할 수 있다.

### data refetch와 UI lifecycle을 분리한다

초기 `isLoading`과 background `isRefetching`은 같지 않다. refetch마다 에디터 전체를 loading 화면으로
교체하면 모델, view state, loader promise가 모두 흔들린다. 초기 로딩만 트리를 대체하고 background refetch는
기존 에디터를 유지한 채 overlay나 비차단 상태로 표현한다.

### cache 무효화 단위를 화면 전환과 맞춘다

- 같은 step 안의 파일 전환: 모델과 view state 유지
- step/mission 변경: 열린 탭, active file, view state, model cache를 함께 초기화
- 화면 최종 unmount: 외부 소유 모델을 한 번만 dispose

모델 내용과 서버에서 새로 받은 content가 다르면 재사용 모델의 value도 동기화해야 한다. 그렇지 않으면
크래시는 없어져도 이전 step 코드가 남는다.

## 연쇄 오류 구분

Monaco loader 초기화 도중 컴포넌트가 사라지면 `operation is manually canceled`가 따라올 수 있다.
이는 `Model is disposed!`로 Error Boundary가 동작하며 생긴 2차 증상일 수 있다. 무시 목록에 넣기 전에
불필요한 mount/unmount와 모델 소유권 충돌을 먼저 고친다.

## 디버깅 체크리스트

1. model을 만든 주체와 dispose하는 주체를 각각 적는다.
2. loading/refetch 조건이 어느 Provider까지 언마운트하는지 컴포넌트 트리로 그린다.
3. mount, unmount, model URI, `isDisposed()`, query 상태를 같은 타임라인에 로깅한다.
4. 뒤로가기 → 재진입 → 즉시 step 이동처럼 refetch와 remount가 겹치는 경로를 반복한다.
5. normal editor와 diff editor를 따로 검증한다.
6. 크래시뿐 아니라 undo/redo, cursor/scroll 복원, 새 step content 동기화까지 확인한다.

## 연결 문서

- [[edu-ai-course-architecture]] — 이 에디터가 들어 있는 학습 플랫폼 구조
- [[edu-ai-course]] — 프로젝트 성과 서사

## 변경 이력

- 2026-09-01: `Model is disposed!`의 이중 소유권과 refetch 언마운트 원인, 모델 보존·검증·초기화 규칙을 추출
