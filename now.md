---
title: Now — 현재 스냅샷
area: profile
tags: [now, 스냅샷, 진행중]
created: 2026-08-03
updated: 2026-09-09
last_review: 2026-09-01
status: draft
---

# Now — 현재 스냅샷

> 현재 상태 스냅샷. AI는 어떤 작업이든 `index.md`와 이 파일을 먼저 읽는다.
> 갱신 빈도 최상 — 과거 상태는 git 이력에 있다.

마지막 갱신: 2026-09-09

## 진행 중

### 개발

- **BrainClone**: 구조 확정, 2026-09-01 한국어로 재전환. 같은 날 대규모 문서 확충 —
  `idea/`, `interest/persona/` 신설, 포트폴리오 6건·스니펫 4건 추가. `AGENTS.md` 별칭 생성
  - 다음: `decisions/` 최초 생성, draft → confirmed 승격, 빈 섹션 채우기
  - 큰 그림은 [[second-brain]] 개념 — 최우선 요구사항은 폰/회사 머신/집 머신이 **하나의** 지식베이스를 공유하는 것
- **회사(goorm) — Edu Vibe**: 현재 주력. 두 레포 소유(`edu-vibe-front`, `edu-vibe-server`).
  **아키텍처를 내가 설계했고, MVP 단계지만 이 구조 그대로 정식 서비스로 올라간다.**
  결정·기각 근거는 [[2026-09-09-edu-vibe-architecture]], 성과 서사는 [[edu-vibe]]
  - 서버: NestJS 11 (**Express** 어댑터) + MongoDB + **Mongoose** + `migrate-mongo`
  - 프론트: **워크스페이스가 아닌 단일 패키지** — pnpm + Vite + **React 19** + `react-router-dom` v7 + Tailwind v4 + Vapor-ui + TanStack Query, **Node 24**
  - 학습창은 다른 팀(Devth)에서 `@devth/learn-*` SDK 패키지로 올 것이라 가정 — 아키텍처 전체가 이 가정 위에 서 있음.
    독립 앱으로 나오면 V1의 **Plan B(4앱 완전 분리)**로 회귀
  - 2026-08-19~29 프론트 4화면 완성(커밋 202, PR 25, 78파일 10,712줄), 서버는 spec 327 · e2e 94테스트까지 확장.
    양쪽 개발회고를 2026-08-31에 작성 — 결론이 같다: **만드는 힘은 충분했고 만든 것을 지키는 장치가 약하다**
  - **2026-09-17 학교 시연**이 첫 목표. 배포 준비 진행 중 — OP DB(`edu_vibe`), 도메인, S3(`grm-edu-vibe`),
    Jenkins/ArgoCD 파이프라인, Vault, k8s, Elastic APM. LLM은 사내 **TokenHub** 경유(모델 `gpt-5-mini`)
  - 프론트 테스트 **0건은 결정이다** — 하네스를 걷어냈고(10파일 2,228줄) 서버는 반대로 유지한다
    → [[2026-09-09-edu-vibe-front-no-tests]]
  - **EDU-650**: 학생 id가 URL·S3 경로에 실려 이름이 노출돼 무작위 `stu-*`로 이관.
    신원은 `(classroomProjectId, nickName)` 부분 unique로 옮김 → [[2026-09-09-edu-vibe-student-id-pii]]
  - 남은 부채: `openapi.json`이 실제 동작과 어긋나 아직 계약으로 승격 못 함 → [[cross-repo-api-contract-drift]]
- **회사(goorm) — edu-ai-course**: 2025-11~2026-04 참여분을 2026-09-01 포트폴리오로 정리.
  구조·패턴은 [[edu-ai-course-architecture]], [[spreadsheet-import-stable-ids]],
  [[llm-rate-limit-defense]], [[monaco-model-lifecycle]]에 분리 기록
- **회사(goorm) — edu-core**: 실시간 협업 편집(goorm-hocuspocus, epoch 기반 문서 버저닝)에 더해
  2026-09-01~02 교육과정 편집 V2 저장 버튼 오작동 추적. 왕복 변환 손실 5종 수정(PR #7928),
  공개 기간 검증 정책 확정. 배경은 [[edu-lecture-v2-editor-schema]],
  [[edu-lesson-period-contract]], [[2026-09-02-lesson-period-policy]]
  - 남은 것: v1 `lesson.edit.multi`의 `time_set` 쓰기 누락(별도 PR), 레거시 2,594건 마이그레이션
- **INOS**: 인문학 모임 플랫폼 + Electron 데스크톱 래핑. 2026-08-24 대규모 기능 작업(알림/수신함,
  모임 시각, 게시판, 로컬 인증 + 초대 링크, 발표 모드, 생성 실패 복구). 상세는 [[inos]].
  작업은 `yunchan312/INOS`의 열린 PR에 있고 `master`에는 아직 머지 안 됨
  - 미해결: **토론 프롬프트를 어떤 모델로 생성할 것인가** (GPT / Gemini / Grok / Claude, 소형 모델로 충분한지)
- **Bulk Mail**: v1.1.x — macOS ad-hoc 코드사이닝 처리 중 (`fix/adhoc-signing` 브랜치)
- **Cascade delete 재설계 (탐색 중)**: 선택지 비교와 결정 표는 [[mongodb-cascade-strategies]].
  현재 기본 추천은 다중 문서 트랜잭션, 아직 변경 없음. 제약: 앱이 한 서버에서 다중 인스턴스로 동작

### 아이디어 (공모전·해커톤)

2026-09-01 `idea/service/`에 3건 구체화. 목록은 [[index|idea/service/index.md]]

- **[[color-walking|컬러워킹]]** — 오늘의 색을 따라 걷고 사진·지도로 기록. 동대문 명소·전통시장 연계.
  핵심 흐름과 해커톤 MVP 범위까지 구체화됨
- **[[zip-up|ZIP-UP]]** — AI 매칭 + 공동 리모델링으로 빈집 재생. 의성 파일럿은 **계획·검증 대상**이지 성과가 아님
- **[[uiseong-workation-healing-map|의성 워케이션 힐링맵]]** — 디지털 노마드용 작업 장소 추천.
  필수 조건이 Google Cloud + Gemini 활용

### 비개발

- **OPIc**: 현재 등급 **IM3**, 난이도 5-5로 실전 15문항 연습 중 → [[opic|interest/topics/opic.md]]
- **Divine Embrace**: 중세 정치 판타지 세계관 프로젝트. 2026-09-01 설정집을 요약 정리.
  설정집 자체에 연도·나이·영토 비율 불일치 5건이 남아 있어 정합성 점검 필요 → [[divine-embrace]]
- **Kychann 캐릭터**: 참조 이미지 27장을 비교해 외형·색상·식별 기준 문서화 (`confirmed`) → [[kychann]]
- **글쓰기**: 홍천 여행 에세이 2편 velog 발행(2026-08), 여행 시리즈 누적 6편.
  산문 연습 루틴(필사 + 주간 퇴고 + 낭독) 진행 중 → [[writing|interest/hobbies/writing.md]]
- **인문학 모임**: 4인 월간 모임의 영화 큐레이터. 현재 블록은 두 달에 걸쳐 두 편 —
  *사랑도 통역이 되나요*(구도) → *서브스턴스*(사운드), 한 세션에서 토론 → [[humanities|interest/hobbies/humanities.md]]
- **밴드**: 커버 대신 자작곡 쓰기 시도 중
- **운동**: 주 6일 헬스 루틴 유지

## 파고 있는 것

- AI 에이전트 워크플로 (Claude Code hooks, skills, 멀티 에이전트) → [[interest|developer/interest]]
- MongoDB Change Streams / CDC → [[mongodb-cascade-strategies]]
- Flutter, Playwright — 학습 단계
- 투자 & 거시경제 → [[investing|interest/topics/investing.md]]

## 저널 백로그

<!-- 하루 1건 상한을 넘긴 트리거를 여기 적립. 다음 트리거일 또는 월간 리뷰 때 소진 -->

## 미해결 질문 (AI가 물어볼 것)

- `idea/service/`의 3개 아이디어는 각각 **어느 공모전·해커톤을 대상으로 한 것인가?**
  마감일이 있으면 진행 중으로 승격 필요
  · 등록: 2026-09-01
- 제주 바이오 AX 해커톤 — 2026년 7월 말 접수 마감. 참가했는지? 결과는?
  · 물어봄: 2026-08-14 ×1
- Cascade 재설계: **레거시 Kafka 토픽을 같은 서비스가 소비하는가, 다른 서비스가 소비하는가?**
  같은 서비스면 Kafka 제거, 다른 서비스면 유지하고 Outbox 추가
  · 물어봄: 2026-08-14 ×2
- Cascade 재설계: MongoDB 배포가 레플리카셋인가? 트랜잭션 사용 가능 여부가 여기서 갈림
  · 물어봄: 2026-08-14 ×1
- `profile/personality.md`의 **약점** 섹션이 비어 있음 (CLAUDE.md §1 빈 섹션 규칙)
  · 등록: 2026-09-01
- `developer/tendency.md`의 **약점** 섹션이 비어 있음
  · 등록: 2026-09-01
- `now.md`의 **이번 분기 우선순위** 섹션이 비어 있음
  · 등록: 2026-09-01
- CLAUDE.md §2 라우팅 테이블에 **`idea/`와 `interest/persona/`가 없음** — 명세와 실제 구조 불일치
  · 등록: 2026-09-01

## 이번 분기 우선순위

## 변경 이력

- 2026-09-09: 두 레포 코드에서 기술 보강 — 결정 2건 추가(식별자 PII 이관, 프론트 테스트 정책),
  스니펫 2건 추가(`mongo-migration-safety`, `figma-design-implementation`),
  기존 스니펫 2건에 항목 7개 추가. 아키텍처 문서의 D6을 대체 표시하고
  코드에서 확정된 것(SSE 인증·S3·배포 형태·TokenHub·env 검증) 반영
- 2026-09-09: Edu Vibe 아키텍처를 노션 Tech Spec 트리 전체에서 BrainClone으로 이관 —
  `decisions/2026-09-09-edu-vibe-architecture.md`, `career/portfolio/edu-vibe.md`,
  스니펫 2건(`cross-repo-api-contract-drift`, `notion-jira-reference-links`) 신설.
  `index.md`의 "이관 대기: Prisma 기각 근거, Express 어댑터 선택" 항목 해소
- 2026-09-01: 대규모 구조 변경 반영 — `idea/service/` 3건, `interest/persona/` 2건,
  포트폴리오 6건, 스니펫 4건, `divine-embrace`, `opic` 추가. 아이디어·비개발 섹션 신설.
  라우팅 테이블 누락 2건을 미해결 질문으로 등록
- 2026-09-01: 문서 언어를 한국어로 되돌림(CLAUDE.md §0 개정); `last_review` 필드,
  `## 저널 백로그` 섹션, 미해결 질문 `물어봄 ×N` 카운터 도입; 빈 섹션 3건을 미해결 질문으로 등록
- 2026-08-24: INOS 기능 작업과 모델 선택 질문 등록
- 2026-08-24: 인문학 모임 두 편 블록(사랑도 통역이 되나요 / 서브스턴스), 편별 관람 초점
- 2026-08-18: 홍천 에세이 velog 발행(2편); 여행 시리즈 누적 6편
- 2026-08-18: EDU VIBE/DEVEL 실습 페이지 모듈화 안건 기록; 결정 보류
- 2026-08-19: Edu Vibe 프론트 React 19로 이동 — "18에 고정" 기록은 오류였음
- 2026-08-14: Edu Vibe 정정 — 두 레포 모두 M0 통과; 프론트는 단일 패키지
- 2026-08-14: cascade delete 재설계 스레드와 미해결 질문 2건 추가
- 2026-08-12: 업무 초점 갱신 — Edu Vibe
- 2026-08-03: 프론트매터 추가 (OKF 준수), 업무·비개발 활동으로 확장
- 2026-08-03: 영어로 이관
