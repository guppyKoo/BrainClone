---
title: edu-core 강의 기간 데이터 계약 (V1·V2 공유)
area: developer
tags: [edu-core, time_set, 날짜-포맷, isOpenTime, 레거시-데이터]
created: 2026-09-02
updated: 2026-09-02
status: draft
---

# edu-core 강의 기간 데이터 계약

V1(레거시 jQuery 편집기)과 V2(`edit-lecture-v2`)가 **같은 DB 필드를 공유**하기 때문에
생긴 제약들. 정책 판단은 [[2026-09-02-lesson-period-policy]]에 분리했다.

## 날짜 포맷 — ISO 8601이 아니다

```
PERIOD_DATE_FORMAT = 'YYYY-MM-DD hh:mm A'   예) 2026-09-02 02:30 PM
```

`src/apps/TeachConsoleApp/utils/periodDate.ts`. ISO 8601과 세 군데가 다르다.

| | ISO 8601 | 이 포맷 |
|---|---|---|
| 구분자 | `T` | 공백 |
| 시각 | 24시간제 | **12시간제 + AM/PM** |
| 타임존 | 오프셋 필수 | 없음 |

AM/PM은 ISO 8601에 아예 없는 표기다. V1이 쓰던 포맷을 V2가 물려받은 것이고,
**V1도 ISO가 아니다.** DB 필드가 `String` 타입(`modules/db/lesson_info.js`)이라
어떤 문자열이든 들어갈 수 있고, 실제로 `open_date: "undefined"` 같은 값도 존재한다.

이 포맷 때문에 생긴 방어 코드 두 개가 `periodDate.ts`에 있다.

- 다른 화면이 `dayjs.locale('ko')`를 전역 설정하면 `format('A')`가 `오전/오후`로 나와
  서버 `Date.parse`가 `NaN`이 된다 → `.locale('en')` 고정
- 이미 `오전/오후`로 저장된 값이 남아 있어 파싱 시 `AM/PM`으로 되돌려 읽는다

**교훈: 날짜를 문자열로, 그것도 로케일 의존 표기로 저장하면 이런 코드가 붙는다.**

## `time_set` — UI와 의미가 반대

`time_set` = "공개 기간을 설정해서 쓰는가"

- `false` → 항상 공개
- `true` → `open_date` ~ `close_date` 구간에만 공개

UI 체크박스는 둘 다 **"항상 공개"** 이므로 저장할 때 반전된다
(V1 `!configs.time_set.prop('checked')`, V2 `!alwaysOpen`).

## 서버 판정 — `isOpenTime`

`core/utils/lessonPeriod.js` (= `service/lesson.js`의 동일 로직).

```js
if (!timeSet) return true;                    // 항상 공개
// 아래 셋 중 하나면 열림
open유효 && close유효 && open <= now <= close
open유효 && close무효 && open <= now          // 시작만 → 허용
open무효 && close유효 && close >= now         // 마감만 → 허용
```

**`timeSet: true` + 날짜 둘 다 없음 → 세 절 모두 false → 영구히 닫힘.**
`Date.parse(null)`이 `NaN`이기 때문이다. 이 조합이 이 파일 전체에서 가장 중요한 함정이다.

읽는 곳은 셋이고 전부 이 함수를 거친다.

| 위치 | 하는 일 | false일 때 |
|---|---|---|
| `service/lesson.js:1182` | 강의 접근 판정 | `reason: 'view'` → 열람 차단 |
| `service/index.js:2162` | 수강 진행 확인 | `throw NotOpenTime` → 수강 차단 |
| `service/index.js:935` | 커리큘럼 생성 | `isLessonOpen` 계산 |

`isPrivate`는 이 함수가 보지 않는다. 즉 **비공개 여부와 기간 판정은 독립**이다.

## V1 다중 편집의 쓰기 누락

`routes/index.js`의 `exports.lesson.edit.multi`:

```js
if (time_set) { data.time_set = time_set; }        // ← false 면 기록 안 함
if (open_date) { ... } else { data.open_date = ''; }   // 날짜는 else 있음
```

`time_set`만 `else`가 없다. 그래서 **"항상 공개"로 바꾸는 조작(true → false)이
저장되지 않는다.** 날짜는 비워지므로 결과가 `time_set: true` + 빈 날짜가 되고,
`isOpenTime`이 항상 false → "항상 공개로 바꿨는데 오히려 완전히 닫히는" 동작.

수정은 `if (time_set !== undefined)` 한 줄이지만 V1 경로라 회귀 범위가 다르다.

## 오염 데이터 규모

로컬 config가 가리키는 DB(2026-09-01 조회) 기준, 전체 128,118건 중:

- `open_date`/`close_date`는 있는데 `time_set`이 true가 아님: **2,594건**
- `is_preview` 필드 자체가 없음: 1,384건
- `files` 필드 자체가 없음: 8,794건

2025-01-01 이후 생성분에서는 앞의 둘이 0건 — **전부 레거시**다.
2,594건의 출처가 위 V1 쓰기 누락이라는 것은 (추론)이며, 코드 경로상 일치한다는 근거뿐이다.

## 관련

- [[2026-09-02-lesson-period-policy]] — 이 계약 위에서 내린 검증 정책 판단
- [[edu-lecture-v2-editor-schema]] — 같은 편집기의 에디터 쪽 함정

## 변경 이력

- 2026-09-02: 최초 작성
