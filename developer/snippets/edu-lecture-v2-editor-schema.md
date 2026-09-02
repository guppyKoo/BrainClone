---
title: edit-lecture-v2 에디터 스키마와 왕복 변환 함정
area: developer
tags: [edu-core, tiptap, prosemirror, 스키마, 왕복-변환, 저장-버튼]
created: 2026-09-02
updated: 2026-09-02
status: draft
---

# edit-lecture-v2 에디터 스키마와 왕복 변환 함정

"내용을 수정하지 않았는데 저장 버튼이 활성화된다"를 추적하다 나온 것들.
기간·`time_set` 쪽은 [[edu-lesson-period-contract]]에 따로 있다.

## 저장 버튼은 세 플래그의 OR

```
disabledSave = ... || (!isLessonModified && !isEditorModified && !isFilesModified) || ...
```

셋 중 **하나만 true여도 켜진다.** 그래서 "왜 켜졌나"를 물을 때는 셋을 분리해서 봐야 한다.

- `isLessonModified` — 서버 lesson vs 설정 스토어 항목별 비교
- `isEditorModified` — 에디터 문서 vs 저장된 콘텐츠 비교
- `isFilesModified` — 첨부파일 개수 비교

## content expression을 실제 스키마에서 확인하라

tiptap 문서의 기본값을 믿으면 안 된다. 이 에디터는 덮어쓴 게 있다.

```
blockquote   inline+      ← tiptap 기본은 block+ 인데 레포가 .extend({content:'inline+'})
listItem     paragraph    ← 정확히 문단 1개. 중첩 리스트·항목 내 두 문단이 불가능
tableCell    block+       ← 셀에 문단 여러 개 가능
codeBlock    text*
paragraph    inline*
```

확인법: `editor.schema.nodes.<이름>.spec.content` 또는 `getSchema(extensions)`.

**`Node.fromJSON()`은 content 표현식을 검증하지 않는다.** 스키마상 불가능한 문서를
조용히 만들어낸다. 검증하려면 `node.check()`를 명시적으로 불러야 한다.
이걸 몰라서 잘못된 입력으로 테스트하고 **틀린 결론을 두 번 냈다** —
"인용구 본문이 전부 유실된다"는 진단이 그렇게 나왔고, 실제로는 정상이었다.

## 왕복 변환이 역함수가 아니다

저장은 `convertEditorToLessonFormat`(에디터 JSON → lesson 포맷),
재비교는 `convertLessonToEditorFormat`(그 반대). 둘이 어긋나면
**저장 직후부터 두 문서가 영구히 달라져** 저장 버튼이 꺼지지 않는다.

특징: **새로고침하면 사라진다.** 에디터를 서버 값으로 다시 만들기 때문.
그래서 "간헐적"으로 보인다.

실제로 확인된 손실 지점(전부 수정함):

| 케이스 | 증상 |
|---|---|
| 표 빈 셀 | `<td><br></td>`로 저장되는데 에디터는 빈 문단 → **표를 삽입만 해도** 버튼이 켜짐 |
| 표 셀의 둘째 문단 | 첫 문단만 직렬화 → 내용 유실 |
| 앞뒤 빈 문단 2개 이상 | 저장·비교가 각각 하나씩만 제거 → 2개부터 어긋남 |
| 빈 코드블럭 | 복원이 `content: []`를 만드는데 스키마를 거친 JSON엔 키가 없음 → **새로고침해도 안 꺼짐** |
| 빈 heading | `content` 없이 저장 → 복원 시 일반 문단으로 강등 |

## contenteditable 제목 비교

```ts
// 저장:  subjectRef.current.innerText.trim()
// 비교:  subjectRef.current?.innerText          ← trim 없었음
```

`innerText`는 렌더된 텍스트라 앞뒤 공백은 물론 **연속 공백·개행까지 축약**된다.
게다가 React는 `subject` prop이 그대로면 contenteditable DOM을 되돌리지 않는다.
그래서 제목 뒤에 공백을 넣어 저장하면 **저장이 오히려 불일치를 만들고**
새로고침 전까지 계속 "수정됨"으로 남았다. 양쪽에 같은 정규화를 걸어 해결.

## 전역 스토어를 컴포넌트 안에서 초기화하지 말 것

`savedLessonFiles`(전역 zustand)를 `LessonEditor` 안에서만 채우고 있었다.
에디터를 띄우지 않는 화면(시험 강의 등)에서는 초기화가 안 돌아 **이전 강의의
목록이 남고**, 개수 비교가 어긋나 저장 버튼이 켜졌다. 시험 강의는 `saveLesson`이
early return이라 저장해도 해소되지 않았다.
→ 모든 강의 타입이 거치는 `ContentComponent`로 옮기고 `?? []`로 항상 대입.

## 검증 하네스

브라우저 없이 **실제 변환 함수와 tiptap 스키마를 그대로 로드**해 왕복시킬 수 있다.

- `@babel/register` + `NODE_PATH=src`로 실제 `.ts/.tsx`를 Node에서 로드
- jsdom 16(`jest-environment-jsdom` 내부에 있음), `innerText`는 미구현이라 직접 shim
- `.scss` 등 자산은 스텁. **단 `@babel/register`보다 먼저 등록하면 `index.scss`가
  `index.ts`보다 먼저 잡혀 모듈 해석이 깨진다** — 등록 순서 주의
- 배럴 파일이 순환 참조를 만들면 노드뷰 컴포넌트를 더미로 `require.cache`에 심어 끊는다
- axios·toast도 같은 방식으로 스텁하면 저장 게이트·전송 payload까지 검증 가능

28개 케이스 + 상태 매트릭스 12건을 이 방식으로 돌렸다. 브라우저 확인보다 빠르고 재현 가능하다.

## 관련

- [[edu-lesson-period-contract]]
- [[y-prosemirror-nodeselection-crash]] — 같은 편집기의 동시편집 쪽 이슈

## 변경 이력

- 2026-09-02: 최초 작성
