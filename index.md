# 🧠 BrainClone — 지식 지도

guppy의 뇌를 복제하는 개인 지식베이스. 규칙 & OKF 명세: [CLAUDE.md](CLAUDE.md) (= [AGENTS.md](AGENTS.md)).
**AI는 어떤 작업이든 이 파일과 [now.md](now.md)를 먼저 읽는다.**

> 마지막 갱신: 2026-09-09 · 대부분의 문서가 `draft` 상태 — 검토 후 `confirmed`로 승격 필요
> 경계: **개발 관련은 전부 `developer/`**, `profile/`·`interest/`는 비개발 영역, `idea/`는 미착수 구상
> 언어: 모든 문서 한국어 (CLAUDE.md §0 참조)

## 📍 현재

- [now.md](now.md) — 진행 중인 것, 현재 집중 대상 (갱신 빈도 최상)

## 👤 Profile — 인간으로서의 나 (비개발)

- [personality.md](profile/personality.md) — 성격, MBTI, 커뮤니케이션 스타일
- [values.md](profile/values.md) — 삶의 철학, 우선순위, 의사결정 원칙
- [journal/](profile/journal/) — 깊은 성찰, 간헐적 회고 (append-only)
  - [2026-08-03-brainclone-start.md](profile/journal/2026-08-03-brainclone-start.md) — BrainClone 시작

## 🧭 Decisions — 기술·설계 판단 (append-only)

- `decisions/` — 무엇을 택했고 무엇을 기각했고 왜. 판단이 뒤집히면 새 문서 + 역링크
  - [2026-09-02-lesson-period-policy.md](decisions/2026-09-02-lesson-period-policy.md) — edu-core 강의 공개 기간 검증 정책 (비공개 시 검증 스킵, `time_set` 정규화)
  - [2026-09-09-edu-vibe-architecture.md](decisions/2026-09-09-edu-vibe-architecture.md) — Edu Vibe MVP 아키텍처 확정 (NestJS(Express)·Mongoose 채택, Fastify·Prisma·4앱 분리 기각)

## 💡 Idea — 아직 만들지 않은 것 (공모전·해커톤 구상)

- [service/index.md](idea/service/index.md) — 서비스 아이디어 목록 (구현·출품하면 `career/portfolio/`로 이관)
  - [color-walking.md](idea/service/color-walking.md) — 컬러워킹: 오늘의 색을 따라 걷고 사진·지도로 기록하는 동대문 연계 산책 서비스
  - [zip-up.md](idea/service/zip-up.md) — ZIP-UP: AI 맞춤 매칭과 공동 리모델링으로 빈집을 재생하는 플랫폼 (의성 파일럿)
  - [uiseong-workation-healing-map.md](idea/service/uiseong-workation-healing-map.md) — 의성 워케이션 힐링맵: 디지털 노마드용 작업 장소 추천 (Google Cloud·Gemini)

## ✨ Interest — 에너지와 영감 (비개발: 책, 음악, 영화, 취미 등)

- [topics/](interest/topics/) — 비개발 학습 & 관심 주제
  - [investing.md](interest/topics/investing.md) — 투자 & 거시경제
  - [opic.md](interest/topics/opic.md) — OPIc 영어 말하기 (현재 IM3, 난이도 5-5 연습)
- [hobbies/](interest/hobbies/) — 취미
  - [humanities.md](interest/hobbies/humanities.md) — 인문학 독서/영화 모임 (영화 큐레이터 역할)
  - [pop-culture.md](interest/hobbies/pop-culture.md) — 애니메이션, 영화, 게임
  - [music.md](interest/hobbies/music.md) — 기타, 밴드, 작곡
  - [writing.md](interest/hobbies/writing.md) — velog 블로그(@yunchan312): 기술 시리즈 & 여행 에세이
  - [divine-embrace.md](interest/hobbies/divine-embrace.md) — Divine Embrace: 100년 전쟁 이후를 다루는 중세 정치 판타지 세계관 프로젝트
  - [fitness.md](interest/hobbies/fitness.md) — 운동 루틴
  - [food.md](interest/hobbies/food.md) — 음식과 미식 취향 (노포 판별 기준)
- [persona/](interest/persona/) — 캐릭터·페르소나 설정
  - [kychann.md](interest/persona/kychann.md) — Kychann: 검은 실루엣 + 아이보리 눈의 개발자 마스코트 캐릭터 (외형·식별 기준)
  - [hina.md](interest/persona/hina.md) — 하시모토 히나: 대화용 캐릭터 페르소나 설정
- [wishlist.md](interest/wishlist.md) — 경험, 물건, 책, 영화

## 💻 Developer — 개발 관련 전부

- [stack.md](developer/stack.md) — 주력 기술 스택 & 숙련도 (실무 반복 사용)
- [interest.md](developer/interest.md) — 개발 관심사 & 학습 주제 (호기심 단계, 숙련되면 stack으로 승격)
- [tendency.md](developer/tendency.md) — 개발자로서의 성향 & 업무 스타일 (관찰된 것)
- [philosophy.md](developer/philosophy.md) — 개발/아키텍처 철학 (원칙)
- [coding-style.md](developer/coding-style.md) — 코드 스타일 & 컨벤션 (구체적 규칙)
- [snippets/](developer/snippets/) — 패턴 & 디버깅 노트
  - [edu-lesson-period-contract.md](developer/snippets/edu-lesson-period-contract.md) — edu-core 강의 기간 데이터 계약 (V1·V2 공유 날짜 포맷, `time_set`, `isOpenTime`)
  - [edu-lecture-v2-editor-schema.md](developer/snippets/edu-lecture-v2-editor-schema.md) — tiptap content expression, 왕복 변환 손실, 저장 버튼 오작동
  - [edu-ai-course-architecture.md](developer/snippets/edu-ai-course-architecture.md) — 계약 공유형 모노레포 구조 (oRPC, Inversify, MSW)
  - [spreadsheet-import-stable-ids.md](developer/snippets/spreadsheet-import-stable-ids.md) — 반복 Import에서 DB ID를 보존하는 upsert/멱등성 패턴
  - [llm-rate-limit-defense.md](developer/snippets/llm-rate-limit-defense.md) — 중첩 병렬 LLM 파이프라인의 429 방어 (백오프, BullMQ)
  - [monaco-model-lifecycle.md](developer/snippets/monaco-model-lifecycle.md) — React 화면 전환에서 Monaco 모델 수명·경쟁 상태 관리
  - [mongodb-cascade-strategies.md](developer/snippets/mongodb-cascade-strategies.md) — MongoDB cascade delete 선택지 (트랜잭션 / Change Stream / Kafka+Outbox)
  - [nestjs-mongoose-pitfalls.md](developer/snippets/nestjs-mongoose-pitfalls.md) — 조용히 실패하는 NestJS/Mongoose 함정
  - [nestjs-e2e-test-harness.md](developer/snippets/nestjs-e2e-test-harness.md) — 아무것도 증명하지 못한 채 통과하는 e2e 스위트
  - [y-prosemirror-nodeselection-crash.md](developer/snippets/y-prosemirror-nodeselection-crash.md) — 협업 에디터 크래시 디버깅 노트
  - [electron-patterns.md](developer/snippets/electron-patterns.md) — 실제 프로젝트에서 나온 Electron 패턴
  - [cross-repo-api-contract-drift.md](developer/snippets/cross-repo-api-contract-drift.md) — 레포가 갈린 프론트·서버에서 계약이 조용히 어긋나는 지점
  - [notion-jira-reference-links.md](developer/snippets/notion-jira-reference-links.md) — 노션·피그마·Jira를 코드에서 참조할 때 깨진 것들
  - [korean-text-wrapping.md](developer/snippets/korean-text-wrapping.md) — `break-keep`이 공백 없는 한글을 넘치게 만드는 이유와 세 속성 해법

## 🚀 Career — 사회적 자아와 성과 (표현 계층, developer/를 링크로 참조)

- [resume.md](career/resume.md) — 이력 기본 정보, 커리어 요약
- [portfolio/](career/portfolio/) — 프로젝트별 성과
  - [edu-vibe.md](career/portfolio/edu-vibe.md) — Edu Vibe: K-12 바이브코딩 실습·학습관리 플랫폼 (아키텍처 설계 + 양쪽 레포 구현)
  - [edu-ai-course.md](career/portfolio/edu-ai-course.md) — goorm AI 맞춤형 교육 플랫폼 (구 `new-edu`)
  - [inos.md](career/portfolio/inos.md) — INOS (인문학 모임 플랫폼)
  - [bulk-mail-electron.md](career/portfolio/bulk-mail-electron.md) — Bulk Mail (Electron 데스크톱 앱)
  - [hufs-sinmumgo.md](career/portfolio/hufs-sinmumgo.md) — 한국외대판 청원24 (2025-07~)
  - [wekick.md](career/portfolio/wekick.md) — WeKick 대학 풋살 매칭 플랫폼
  - [rebid.md](career/portfolio/rebid.md) — Re:Bid 입찰 기반 업사이클링 중고 거래 (2024-05)
  - [chepl.md](career/portfolio/chepl.md) — ChePL 체육대회 운영 통합 플랫폼 (2024-03~04)
  - [yellowbook.md](career/portfolio/yellowbook.md) — Yellowbook 소상공인 재고·일정 관리 B2B SaaS (2024-03~04)
  - [challkathon.md](career/portfolio/challkathon.md) — CHALLKATHON 대상 수상 (합의 형성·소통 역할)
- [goals.md](career/goals.md) — 단기/장기 커리어 목표
- [interview-qa.md](career/interview-qa.md) — 예상 면접 Q&A
