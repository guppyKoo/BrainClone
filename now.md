---
title: Now — 현재 스냅샷
area: profile
tags: [now, 스냅샷, 진행중]
created: 2026-08-03
updated: 2026-09-16
last_review: 2026-09-01
status: draft
---

# Now — 현재 스냅샷

> 현재 상태 스냅샷. AI는 어떤 작업이든 `index.md`와 이 파일을 먼저 읽는다.
> 갱신 빈도 최상 — 과거 상태는 git 이력에 있다.

마지막 갱신: 2026-09-16

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
  - **LLM 품질 평가 체계 구축이 다음 우선순위**: 서비스 코드는 e2e·테스트로 검증하지만 핵심인 LLM 출력은
    현재 프롬프트만으로 관리하고 있어, 개선 전제인 품질 측정·추적 지표를 먼저 마련하기로 함.
    서비스는 턴별 원본과 생성 AX의 L·Ambiguity를 기록한다. 오프라인 처리기의 평가 LLM은 발화·맥락을
    닫힌 실행 스키마 R로 분류하는 Extractor 역할만 맡는다. 코드는 S와 검사항목을 생성하고 브라우저
    증거에서 B·D 및 변경 허용 여부를 계산하며, M/Judge 단계 없이 Q1~Q3를 산출한다. DSL로 옮기지 못한
    요구는 `UNMAPPED_REQUIREMENT`로 분리한다. 중단·정책 차단·코드 없음은 제외하고, 실행 불가 HTML은
    Q1~Q3를 0점으로 기록한다. 다만 검증 가능한 R이 없는 턴과 실행 불가 0점은 정상 품질 그래프에서
    제외하고 Gate 목록에서 별도 추적한다. S/L별 그래프를 두 Ambiguity로 필터링하며 평가 호출은
    TokenHub client tag `edu-vibe-llm-quality-eval-requirements`로 서비스 호출과 분리한다.
    → [[2026-09-15-edu-vibe-llm-eval-pipeline]], [[2026-09-16-edu-vibe-eval-gate-reporting]]
    이론의 출발점은 [[2026-09-11-edu-vibe-llm-evaluation]], 기존 관측 모델은
    [[2026-09-14-edu-vibe-llm-eval-observation-model]]
  - **2026-09-16: 산출물의 주장을 "품질 점수"에서 "계측값"으로 낮췄다.** 좋다/나쁘다는 데이터를 보는
    사람이 판단하고 도구는 수집·계산만 한다. 같은 날 제품 변경 없이 평가기 버전 때문에 Q1 평균이
    0.697 → 0.835로 움직인 것이 계기다. 함께 Extractor 계약을 코드·테스트로 강제하고(결과물 차단,
    모델의 S·checks·scores 무시, 자유 서술 차단), 변경 허용 여부를 R에서 역추론하지 않고
    `changeScope`로 명시 선언하도록 바꿨다. "요구 미충족(FAIL)"과 "측정 불가(NOT_OBSERVABLE)"도 분리했다
    → [[2026-09-16-edu-vibe-eval-instrument-contract]]
  - 계측기로서 남은 선결 조건 3가지: **재현성 통제, 오차 명시, 단위(S) 안정화**.
    모호성·난이도 파이프라인은 26/26 null이라 리포트 축 6개가 비어 있다
- **회사(goorm) — edu-ai-course**: 2025-11~2026-04 참여분을 2026-09-01 포트폴리오로 정리.
  구조·패턴은 [[edu-ai-course-architecture]], [[spreadsheet-import-stable-ids]],
  [[llm-rate-limit-defense]], [[monaco-model-lifecycle]]에 분리 기록
- **회사(goorm) — edu-core**: 실시간 협업 편집(goorm-hocuspocus, epoch 기반 문서 버저닝)에 더해
  2026-09-01~02 교육과정 편집 V2 저장 버튼 오작동 추적. 왕복 변환 손실 5종 수정(PR #7928),
  공개 기간 검증 정책 확정. 배경은 [[edu-lecture-v2-editor-schema]],
  [[edu-lesson-period-contract]], [[2026-09-02-lesson-period-policy]]
  - 남은 것: v1 `lesson.edit.multi`의 `time_set` 쓰기 누락(별도 PR), 레거시 2,594건 마이그레이션
- **INOS**: 인문학 모임 플랫폼 + Electron 데스크톱 래핑. 상세는 [[inos]].
  작업은 `yunchan312/INOS`의 열린 PR(#16 등)에 있고 `master`에는 아직 머지 안 됨
  - 2026-08-24: 알림/수신함, 모임 시각, 게시판, 로컬 인증 + 초대 링크, 발표 모드, 생성 실패 복구
  - 2026-09-09: **서가 책등 스타일을 네 화면에 통일**(`SpineFace` 추출), 도서 검색 자동완성으로
    죽어 있던 `bookIsbn` 경로 연결, 배포 환경변수 구멍 메움, README·DESIGN 재작성,
    회고 + **개발기 블로그 5편** 작성(`docs/blog/`)
  - **모델 질문의 성격이 바뀌었다.** "어떤 모델로 생성할 것인가(GPT/Gemini/Grok/Claude,
    소형으로 충분한가)"는 **지금 답할 수 없는 질문**이다 — 발제문 품질을 재는 수단이 없어서
    무엇을 바꿔도 나아졌는지 알 수 없다. 현재는 `claude-sonnet-4-6` 단일.
    목표는 **"적은 비용에 좋은 품질"**이고, 품질 개선과 비용 절감이 **같은 선결 조건(eval)**을 공유한다.
    → 다음 작업은 모델 비교가 아니라 **평가 기준 구축**
  - 발제문 **슬롯 스키마는 아직 미구현**이다. 실제 프롬프트는 `"발제 질문 5개를 작성해주세요"`
    한 줄 + 정규식 파싱. [[inos]]의 슬롯 표와 `DISCUSSION_GENERATION.md`는 **구현 전 설계안**
  - **다음 단계 수정(2026-09-10)**: INOS의 중심은 별도의 운영 경험 만들기가 아니라 **AI 발제문
    생성 품질을 다듬는 경험**이다. 생성 결과의 평가 지표·데이터셋, 검색 품질·성능 최적화,
    프롬프트·모델·검색 변경의 회귀 비교를 우선한다. 자동화 테스트와 운영 관측은 이를 지지하는 기반이다.
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

- 2026-09-16: Edu Vibe eval의 LLM 역할을 닫힌 R 추출로 제한하고 M/Judge를 제거했다. S·검사 컴파일·B·D·
  Q1~Q3는 코드가 담당하며, DSL 미지원 요구는 부분 채점하지 않고 Gate에서 별도 추적한다.
  (오늘 decisions 1건 작성으로 상한 도달 → 다음 트리거일에 재현성 결정으로 승격)

- 2026-09-16: Edu Vibe eval의 명세량 S를 LLM이 R과 별도로 출력하는 방식을 폐기하고,
  독립 판정 가능한 원자적 R만 생성한 뒤 코드에서 `S = requirements.length`로 계산하도록 변경.
  R과 S의 모순을 구조적으로 막고 각 R의 constraint를 최대 하나로 제한했다.
  (오늘 decisions 1건 작성으로 상한 도달 → 다음 트리거일에 `decisions/`로 승격)
- 2026-09-09: INOS 발제문 — **슬롯 스키마 구현보다 eval이 먼저**라는 우선순위 역전.
  2026-08-25에 확정한 6슬롯 설계는 "무엇을 만들 것인가"를 정했지만 "나아졌는지 어떻게 아는가"를
  비워뒀다. 품질 개선과 비용 절감이 같은 선결 조건을 갖는다는 진단이 그 설계보다 상위다.
  (오늘 decisions 3건으로 상한 초과 → 다음 트리거일에 `decisions/`로 승격)
- 2026-09-10: **AI 시대의 개발자 경력에 대한 고민.** 3년 안에 전통적인 주니어 자리가 크게 줄 수 있고,
  개발자가 되려는 대학생 감소는 경쟁자 감소로 작용할 수 있지만 기업 수와 회사당 개발자 정원도 함께
  줄 수 있다는 양면을 보고 있다. 한국 남성으로서 25세부터 재직해 또래보다 연차를 먼저 쌓은 점이
  좁아지는 진입문을 일찍 통과한 이점인지, 전체 포지션 감소가 그 이점을 상쇄할지 탐색 중이다.
  현재 결론은 "안전하다"가 아니라, 신입 채용이 줄기 전에 진입해 경력자 구간으로 이동 중인 상대적
  이점이 있으며 그 이점을 연차가 아닌 시스템 소유·운영 성과·AI 결과 검증 경험으로 굳혀야 한다는 것.
  (오늘 journal 1건 작성으로 상한 도달 → 다음 트리거일에 `profile/journal/`로 승격)
- 2026-09-14: Edu Vibe LLM eval 구현을 서비스 로그 수집부와 오프라인 평가 처리기로 분리하고,
  1차 범위를 로그 수집·R 추출·전후 브라우저 실행·검사 DSL·결정론적 Q1까지로 확정. 생성 AI는
  Ambiguity만 기록하며 측정 실패를 `false`가 아닌 `null`로 남긴다. Q2·Q3는 B·D·M 증거 구조가
  안정된 뒤 추가한다. (오늘 decisions 1건 작성으로 상한 도달 → 다음 트리거일에 `decisions/`로 승격)
- 2026-09-14: TokenHub/Langfuse export 26건의 `userId`·`sessionId`가 전부 `null`인 원인을 추적.
  서비스 DB에는 `scope.userId`로 대화를 저장하지만 TokenHub에는 메시지 배열만 보내 식별 정보가
  trace로 전달되지 않는다. 누적 메시지 prefix로 6개 대화 체인은 복원할 수 있지만 실제 사용자 간
  귀속은 복원 불가능하다. (오늘 기록 상한 도달 → 다음 트리거일에 `decisions/`로 승격)

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

- 2026-09-16: eval 산출물을 품질 판단이 아닌 계측값으로 재정의. Extractor 계약을 코드·테스트로 강제하고
  변경 허용 범위를 `changeScope` 명시 선언으로 이전. 재현성 백로그 항목을 결정 문서로 승격
- 2026-09-16: 평가 LLM을 닫힌 R Extractor로 제한하고 M/Judge를 제거한 결정론적 채점 구조로 변경
- 2026-09-16: 검증 가능한 R이 없는 턴과 실행 불가 0점을 정상 품질 집계·그래프에서 분리하고 Gate 목록에서 추적
- 2026-09-15: Edu Vibe LLM eval의 Gate, R·D·B/M 단계, 생성·평가 AX 소유권과 평가 전용 TokenHub 태그를 확정
- 2026-09-14: TokenHub 요청에 사용자·세션 식별 정보가 빠져 Langfuse export가 전부 익명화되는 원인 확인
- 2026-09-14: Edu Vibe LLM eval의 구현 방법을 서비스 로그 수집과 오프라인 평가 처리로 분리하고,
  R·브라우저 증거·검사 DSL·Q1까지를 첫 구현 범위로 확정
- 2026-09-14: Edu Vibe LLM eval을 종합점수·과제별 계약 없이 독립 턴의
  Q1~Q4·L·S·`utteranceAmbiguity`·`contextualAmbiguity`를 수집하는 관측 모델로 좁힘
- 2026-09-11: Edu Vibe의 다음 LLM 품질 과제를 프롬프트 개선보다 평가 지표·추적 체계 구축으로 확정.
  이론 문서 작성 완료, 구현 방법론은 미정
- 2026-09-10: AI 시대의 주니어 채용 축소, 개발자 수요 감소, 이른 경력 시작의 상대적 이점에 관한 커리어 고민을 저널 백로그에 추가
- 2026-09-10: INOS 우선순위를 운영 책임에서 AI 발제문 평가·검색 최적화로 수정. 테스트와 운영은 이를 지지하는 기반으로 재배치
- 2026-09-10: INOS의 다음 단계를 운영 책임, 자동화 테스트, AI 발제문 eval 구축으로 확정하고 기존 테스트 경험을 명시
- 2026-09-09: INOS 갱신 — 서가/검색/문서 작업과 개발기 5편 기록. 모델 선택 미해결 질문을
  "평가 기준이 먼저"로 재정의(질문 자체가 답할 수 없는 형태였음). 슬롯 스키마 미구현 명시.
  포트폴리오의 **pgvector 오기재 정정**, 스니펫 1건 신설(`google-fonts-korean-subset`),
  `cross-repo-api-contract-drift`에 모노레포 반례 §13 추가
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
