---
title: 반복 가능한 Spreadsheet Import에서 ID를 보존하는 법
area: developer
tags: [ETL, Google-Sheets, MongoDB, upsert, idempotency]
created: 2026-09-01
updated: 2026-09-01
status: draft
---

# 반복 가능한 Spreadsheet Import에서 ID를 보존하는 법

Google Sheets 같은 사람이 편집하는 원본을 DB에 반복 적재할 때 매번 새 ID를 만들면 기존 참조와 사용자
진행 데이터가 끊긴다. 반대로 사람이 쓰는 ID를 DB primary key로 그대로 사용하면 이름 변경, 중복,
오타와 ID 형식 변경이 저장 계층까지 번진다.

`edu-ai-course`의 콘텐츠 import는 **사람용 source key와 시스템용 DB ID를 분리하고, 생성한 DB ID를
원본 시트에 write-back**하는 방식으로 이 문제를 풀었다.

## 데이터 흐름

```text
Sheets 읽기
  → 사람용 ID와 기존 (DB) 컬럼으로 idMap 구성
  → DB ID가 없으면 새로 생성
  → project/course/lesson/step 참조를 DB ID로 재매핑
  → idMap을 Sheets의 (DB) 컬럼에 write-back
  → MongoDB bulkWrite(upsert)
```

시트는 다음 두 역할을 함께 가진다.

- 사람이 편집하는 콘텐츠 원본
- 외부 행과 내부 엔티티를 잇는 ID registry

## 핵심 규칙

### ID map을 먼저 완성하고 본문을 파싱한다

step 행을 파싱하다가 그때그때 lesson ID를 만들면 순서 의존성과 중복 생성이 생긴다. 먼저 전체 구조를
훑어 `projects`, `courses`, `lessons`, `steps` map을 완성한 뒤 모든 참조를 변환한다.

### 기존 DB ID가 있으면 반드시 재사용한다

- `(DB)` 컬럼 값이 있으면 그 값을 신뢰한다.
- 없을 때만 새 ID를 만든다.
- 새로 만든 ID는 DB 저장 전에 시트에도 기록한다.

이 순서가 재업로드를 idempotent하게 만든다. write-back 뒤 DB 저장이 실패하면 다음 실행에서도 같은 ID를
다시 사용해 복구할 수 있다.

### 관련 엔티티의 파생 규칙은 결정적으로 만든다

프로젝트와 코스가 1:1이고 시트에 course DB ID 칸이 없다면 같은 random suffix에서 `proj_...`,
`course-...`를 파생할 수 있다. 단, 관계가 1:N으로 바뀔 가능성이 있으면 독립 registry 열을 두는 편이 낫다.

### 저장은 entity별 bulk upsert로 묶는다

완성된 steps/projects/courses를 각 model의 stable ID로 `bulkWrite({ upsert: true })`한다. 행마다 개별 저장하는
것보다 왕복을 줄이고, 신규와 갱신을 같은 경로로 처리한다.

## 실패 모드와 보강점

- **write-back 성공, DB 저장 실패**: 같은 ID로 재실행할 수 있어야 한다.
- **DB 저장 성공, write-back 실패**: 다음 실행에서 새 ID를 만들 위험이 있다. 실행 ID를 둔 staging,
  DB의 source key unique index, 또는 DB에서 기존 mapping을 역조회하는 보강이 필요하다.
- **사람이 `(DB)` 값을 수정**: 참조가 갈라질 수 있다. 열 보호, 형식 검증, 존재 여부 확인을 둔다.
- **source key 중복**: `Map`에서 조용히 덮어쓰지 말고 업로드 전에 중복을 오류로 반환한다.
- **부분 upsert**: entity 간 참조가 있어 transaction을 쓸 수 있으면 함께 묶고, 어렵다면 import 상태와
  재실행 전략을 둔다.
- **삭제 의미가 불분명**: 시트에서 사라진 행을 DB에서도 지울지는 별도 정책이다. import와 delete를
  암묵적으로 결합하지 않는다.

## 재사용 체크리스트

1. 사람용 source key와 시스템 DB ID를 분리했는가?
2. mapping의 영구 저장 위치가 있는가?
3. 전체 idMap을 만든 뒤 참조를 변환하는가?
4. source key와 DB ID의 중복·형식을 업로드 전에 검증하는가?
5. write-back과 DB upsert 사이의 부분 실패를 재실행할 수 있는가?
6. 삭제, 이동, 1:1 관계가 1:N으로 바뀌는 경우의 의미가 정의됐는가?

## 연결 문서

- [[edu-ai-course-architecture]] — import가 들어 있는 서버 구조
- [[edu-ai-course]] — 이 패턴을 구현한 프로젝트 성과

## 변경 이력

- 2026-09-01: Google Sheets 재업로드에서 DB ID를 보존한 구현을 일반 ETL/idempotency 패턴으로 추출
