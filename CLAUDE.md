# BrainClone 관리 규칙 & OKF 규격

이 저장소는 사용자(guppy)의 뇌를 복제하는 개인 지식 베이스다.
AI(Claude)는 이 문서의 규격을 따라 읽고 쓴다. 전역 Read/Write 트리거는 `~/.claude/CLAUDE.md`에 정의되어 있다.

## 1. OKF 문서 규격

모든 `.md` 문서(이 파일과 `index.md` 제외)는 아래 frontmatter로 시작한다.

```markdown
---
title: 문서 제목
area: profile | interest | developer | career
tags: [키워드1, 키워드2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: draft | confirmed
---
```

- **status: draft** — AI가 추론·초안으로 작성한 문서. 본문에서 확인되지 않은 추정 내용은 `(추정)` 표기를 붙인다.
- **status: confirmed** — 사용자가 내용을 검토·확정한 문서. `(추정)` 표기가 남아 있으면 안 된다.
- 문서 간 연결은 위키링크 `[[파일명]]` 또는 상대경로 마크다운 링크를 사용한다.
- 한 파일 = 한 주제. 800줄을 넘으면 분할하고 `index.md`를 갱신한다.
- 저널 파일명은 `journal/YYYY-MM-DD-주제.md` 형식을 따른다.

## 2. 정보 라우팅 테이블 — 어떤 정보를 어디에 쓸 것인가

| 정보 유형 | 대상 파일 |
|---|---|
| 성격, MBTI, 강점/약점, 소통·작업 스타일 | `profile/personality.md` |
| 인생관, 우선순위, 의사결정 원칙 | `profile/values.md` |
| 깊은 고민, 회고, 시기별 생각 | `profile/journal/YYYY-MM-DD-주제.md` |
| 관심 기술·학문·트렌드 정리 | `interest/topics/<주제>.md` |
| 취미 (운동, 독서, 게임, 음악 등) | `interest/hobbies/<취미>.md` |
| 하고 싶은 경험, 사고 싶은 것, 읽을 책 | `interest/wishlist.md` |
| 기술 스택, 숙련도 | `developer/stack.md` |
| 코드 스타일, 컨벤션, 폴더 구조 취향 | `developer/coding-style.md` |
| 개발·아키텍처 철학 | `developer/philosophy.md` |
| 자주 쓰는 패턴, 디버깅 노트 | `developer/snippets/<주제>.md` |
| 이력서 기본 정보, 경력 요약 | `career/resume.md` |
| 프로젝트별 성과·역할·문제 해결 | `career/portfolio/<프로젝트>.md` |
| 커리어 목표, 희망 연봉/이직 조건 | `career/goals.md` |
| 면접 예상 질문과 답변 | `career/interview-qa.md` |

라우팅이 애매하면 새 파일을 만들지 말고 사용자에게 묻는다.

## 3. Read 규칙

1. 항상 `index.md`에서 시작해 필요한 영역만 내려간다. 전체 폴더를 무차별로 읽지 않는다.
2. 질문에 답하기 전, 관련 영역 문서를 읽었는지 확인한다. 문서에 없는 내용은 지어내지 않는다.
3. `status: draft` 문서의 내용을 인용할 때는 "확정되지 않은 초안"임을 밝힌다.

## 4. Write 규칙

1. **수정 우선**: 같은 주제의 문서가 있으면 새 파일 대신 그 문서를 수정한다.
2. **updated 갱신**: 내용을 수정하면 반드시 frontmatter의 `updated`를 오늘 날짜로 바꾼다.
3. **index 동기화**: 새 문서를 만들면 같은 작업 안에서 `index.md`에 링크를 추가한다.
4. **충돌 해결**: 사용자의 현재 발화 > 문서 기록. 충돌 발견 시 문서를 갱신하고 변경 사실을 알린다.
5. **삭제 금지**: 문서·내용 삭제는 사용자가 명시적으로 지시했을 때만 한다. 오래된 정보는 지우는 대신 `~~취소선~~`과 날짜를 남기거나 journal로 옮긴다.
6. **민감 정보 금지**: 비밀번호, API 키, 주민번호 등 크리덴셜은 절대 기록하지 않는다. 연봉 등 민감 수치는 사용자가 직접 요청한 경우에만 기록한다.
7. **저널은 불변**: `journal/`의 과거 날짜 파일은 수정하지 않는다. 생각이 바뀌면 새 날짜의 저널을 쓴다.

## 5. 유지보수

- 월 1회 정도 사용자와 함께 `draft` 문서를 검토해 `confirmed`로 승격하는 것을 제안한다.
- `updated`가 6개월 이상 지난 문서를 발견하면 최신화가 필요한지 사용자에게 확인한다.
