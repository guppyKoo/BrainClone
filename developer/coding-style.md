---
title: 코드 스타일과 컨벤션 취향
area: developer
tags: [코딩스타일, 컨벤션, 커밋]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# 코드 스타일과 컨벤션 취향

> 실제로 운용 중인 ECC 룰셋(`~/.claude/rules/ecc/`)과 커밋 이력에서 추출.

## 핵심 원칙 (ECC 룰셋으로 강제 중)

- **불변성 우선**: 객체를 제자리에서 변경하지 않고 새 복사본을 반환
- **KISS / DRY / YAGNI**: 동작하는 가장 단순한 해법, 실재하는 반복만 추상화, 미리 만들지 않기
- **작은 파일 다수 > 큰 파일 소수**: 200–400줄 권장, 800줄 상한. 기능/도메인 기준으로 조직
- **깊은 중첩 금지**: 4단계 이상 중첩 대신 early return
- **매직 넘버 금지**: 의미 있는 임계값은 이름 있는 상수로
- **에러는 명시적으로**: 조용히 삼키지 않기, UI에는 친절한 메시지·서버에는 상세 로그

## 네이밍

- 변수/함수 `camelCase`, 불리언은 `is/has/should/can` 접두사
- 타입/컴포넌트 `PascalCase`, 상수 `UPPER_SNAKE_CASE`, 훅 `use` 접두사

## 커밋 & Git

- Conventional Commits (`feat:`, `fix:` …) + **한국어 메시지**
  - 예: `fix: macOS 빌드에 ad-hoc 코드사인 추가, Gatekeeper 손상 오류 안내 추가`
- 기능 브랜치 → PR → main 병합 흐름 (예: `ui-redesign-v1.1.0`, `fix/adhoc-signing`)

## UI 취향

- Linear/Notion 스타일의 미니멀 UI 선호 (Bulk Mail을 해당 스타일로 리디자인)
- 디자인 토큰 기반 테마: seed 색상(`--fg`/`--bg`)에서 `rgb(from ...)`로 파생 + `.dark` 클래스 다크모드
- Radix 기반 자체 UI 컴포넌트 셋 구축 선호 (외부 킷 통째로 쓰기보다 직접 구성)

## 테스트

- TDD 지향, 커버리지 80% 목표 (ECC 규칙) — 실제 준수 정도는 **TODO: 본인 평가**
