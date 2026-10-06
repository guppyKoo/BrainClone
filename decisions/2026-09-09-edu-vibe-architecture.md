---
title: Edu Vibe MVP 아키텍처 확정 (V2 Tech Spec)
area: developer
tags: [edu-vibe, goorm, architecture, NestJS, Mongoose, Prisma, Fastify, SSE, JWT, MongoDB]
created: 2026-09-09
updated: 2026-10-06
status: confirmed
---

# 결정: Edu Vibe MVP 아키텍처

내가 설계한 아키텍처다. MVP 단계에서 정했지만 **이 구조 그대로 정식 서비스로 올라간다.**
그래서 "MVP니까 나중에 바꾼다"를 근거로 삼은 결정이 하나도 없다 — 감수한 것은 감수한 것으로,
되돌릴 조건은 조건으로 아래에 명시했다.

원천 문서는 노션이다. 상위 컨테이너 [\[Edu Vibe\] Tech Spec 계획](https://app.notion.com/p/3b34e6997fb0809a818adca20a6424dd)이
**첫 번째 결정(V1)**이고, 그 아래 [\[V2\] Edu Vibe Tech Spec](https://app.notion.com/p/3b94e6997fb080a58e4ceffec1f5250b)이
**확정된 아키텍처**다. V1은 폐기가 아니라 **Plan B로 내려간 상태**다(D1 참고).

관련 문서: [[nestjs-mongoose-pitfalls]] · [[nestjs-e2e-test-harness]] · [[cross-repo-api-contract-drift]] · [[edu-vibe]]

## 맥락 — 무엇 위에서 설계했나

제품은 국내 K-12(1차 타겟 중등) 대상 **바이브코딩 실습 + 학습관리 플랫폼**이다.
아키텍처 판단을 지배한 전제는 기술이 아니라 **기획의 전제 조건 두 개**였다.

1. **이번 MVP는 교사용 서비스 중심이다.** 학생은 계정을 갖지 않고, 교사가 발급한 학습 코드로
   실습창에만 들어온다. → 무계정 식별(D6)과 라우트 접근 제어의 근거가 전부 여기서 나온다.
2. **만 14세 미만 가입 허들이 제품 도입 자체를 막는다.** → "학생 회원가입"이라는 선택지가
   기술적으로 가능해도 제품적으로 존재하지 않는다.

여기에 조직 전제가 하나 더 붙는다. **실습창(학습창)은 다른 팀(Devth)이 만든다.**
그 모듈이 어떤 형태로 오느냐가 레포 구조 전체를 갈랐다.

추가로 **추후 전자정부표준프레임워크(Spring) 이관 가능성**이 전제로 들어와 있다. D2가 여기서 나왔다.

## 결정

| # | 결정 | 근거 한 줄 |
|---|---|---|
| D1 | 프론트 1레포 + 백엔드 1레포 (`edu-vibe-front` / `edu-vibe-server`), Edu가 둘 다 소유 | Devth 학습창이 `@devth/learn-*` **SDK 패키지**로 온다는 전제. 책임 분리는 레포가 아니라 패키지·모듈 경계로 |
| D2 | 서버는 **NestJS + Express 어댑터** | Spring 이관 시 코드 변경이 가장 적다. DI·데코레이터·컨트롤러/서비스/모듈이 Bean·어노테이션·계층과 1:1에 가깝다 |
| D3 | **MongoDB + Mongoose + `migrate-mongo`** | 새 DB를 띄우고 검증할 시간이 없다. 표준프레임워크 v5부터 MongoDB 공식 지원이라 이관 전제와 충돌하지 않는다 |
| D4 | 계약은 코드 공유가 아니라 **OpenAPI → FE codegen** | `@nestjs/swagger`가 스펙 원천. 레포가 갈려도 드리프트를 codegen diff가 잡는 구조 |
| D5 | 실시간은 **SSE**, 교사 패널은 폴링으로 시작 | 학교망/프록시가 WebSocket Upgrade를 자주 차단한다 — 이 제품 최대의 인프라 리스크 |
| D6 | 학생은 **무계정**, `id = {entryCode}-{nickName}` upsert + 단기 JWT ⚠️**대체됨** | 동명이인 방지를 DB unique 제약으로 보장. 이어하기(UC04)가 깨지지 않는 최소선 |
| D7 | 문서 id는 ObjectId가 아니라 **접두사 붙은 base36 문자열** (`class-`, `proj-`, `arti-`, `clpj-`, `chat-`) | 값만 보고 무엇의 id인지 알 수 있고, 잘못된 id를 DTO 경계에서 거를 수 있다 |
| D8 | 서버 경로를 **프론트 라우팅과 동일한 3중 키**로 (`/classrooms/:classroomId/projects/:projectId/artifacts/:userId`) | 프론트가 `/learn/:classId/:studentId/:projectId`를 쓰므로 id 변환 계층이 아예 생기지 않는다 |
| D9 | **Redis · 메시지 큐 미도입** | 공유 상태가 필요한 지점이 없다. JWT는 stateless, rate limit은 Mongo `$inc`, SSE는 요청 스코프, 교사 패널은 폴링 |
| D10 | 인가 경계는 **전적으로 서버**. 프론트 라우트 가드는 UX 장치 | URL의 `studentId`를 프론트에서 대조하지 않는다 — 보안처럼 보이는 코드가 서버 검증 누락을 가린다 |

### D1 — Plan A / Plan B는 지금도 살아 있다

V1의 원안은 **워크스페이스 4개 완전 분리**였다: `edu-vibe-learn`(Devth FE) · `edu-vibe-manage`(Edu FE) ·
`devth-server`(Devth BE) · `edu-server`(Edu BE). 공유 패키지 없음, Turborepo 없음, 팀 간 공통분모는
OpenAPI 계약과 JWT 시크릿뿐. 이 안에는 **서버–DB 경계 1안/2안**(DB 1개 컬렉션 분리 vs 논리 DB 2개 + 서버 간 API)까지
비교표로 정리해 뒀고, MVP 속도 기준으로는 1안을 권장했다.

이 구조가 Plan B로 내려간 이유는 설계가 틀려서가 아니라 **전제가 바뀌어서**다 —
Devth 모듈이 독립 앱/서비스가 아니라 SDK 패키지로 온다면, 레포를 넷으로 가르는 비용만 남는다.
**Devth 모듈이 독립 앱으로 나오는 순간 Plan B로 회귀한다.** 그때 필요한 비교표는 이미 V1 문서에 있다.

관련해서 실습 페이지 분담은 별도 안건으로 정리했고, 내가 선호한 방향은
**"코드 실습 + AI 채팅을 API 서버로 제공"**(데이터는 사용처인 Edu Vibe가 소유)이었다.
차선은 UI까지 Devth가 만들고 iframe으로 띄우는 안 — UI/UX 확장성이 나빠 차선으로 뒀다.

### D2 — Fastify를 실험까지 하고 채택하지 않았다

**이 결정은 뒤집힌 결정이다.** V1에서는 Fastify 5 + TypeBox를 제안했고,
그냥 제안이 아니라 [별도 실험 레포](https://github.com/guppyKoo/Fastify-vs-Express-test)로
동일 API를 Express/Fastify 양쪽에 구현해 4종 실험을 돌렸다.

실험이 실제로 증명한 것:

- **성능은 도입 근거가 아니다.** 순수 프레임워크 오버헤드(`/hello`)에서는 1.6배 차이(34,555 → 53,761 req/s)가
  나지만, **30ms 지연 하나만 넣어도 +1.8%로 붕괴**하고 실제 Atlas 조회에서는 +2.1%다. 꼬리 지연(p99)도 사실상 동일.
- **진짜 이득은 응답 스키마의 구조적 방어였다.** 같은 실수(도큐먼트를 그대로 return)를 양쪽에 심었더니
  Express는 `passwordHash`·`internalMemo`를 그대로 유출했고 Fastify는 화이트리스트로 걸렀다.
  응답 크기도 32.7KB → 14.1KB로 절반이 됐는데, 이건 성능이 아니라 **내부 필드가 빠진 부산물**이다.
- **선언 지점이 줄었다.** "email은 이메일 형식"을 Express는 2곳(zod + swagger 주석), Fastify는 1곳(TypeBox)에 쓴다.
  실제 커밋 diff로 +2/−2 vs +1/−1을 확인했다.

그럼에도 NestJS로 간 이유는 **표준프레임워크 이관 전제**가 나중에 들어왔기 때문이다.
그리고 NestJS(Fastify)를 쓰지 않은 이유는 따로 있다 — NestJS는 DTO 중심으로 검증·직렬화·문서를 처리하므로
**Fastify 고유의 스키마 이점이 희석**된다. 어댑터 계층에서는 생태계 호환이 넓은 Express가 실리적이다.

중요한 것은 **실험이 증명한 원칙은 스택이 바뀌어도 보존했다**는 점이다.
"경계에서 선언한 스키마가 실수를 막는다"를 NestJS 장치로 옮겼다.

| 실험이 확인한 가치 | Fastify | NestJS(Express) |
|---|---|---|
| 요청 검증 | TypeBox + ajv | DTO + class-validator (`ValidationPipe`) |
| mass assignment 차단 | `additionalProperties: false` | `whitelist` + `forbidNonWhitelisted` 전역 |
| 응답 필드 유출 차단 | 응답 스키마 화이트리스트 | `ClassSerializerInterceptor` + `@Expose` |
| 문서 자동 생성 | `@fastify/swagger` | `@nestjs/swagger` |
| 스키마-실데이터 일치 | (앱 계층에서만) | **Mongoose `strict: true`가 데이터 계층에서 강제** |

⚠️ 대가는 명확하다. **NestJS는 이 방어선이 기본값이 아니라 설정이다.** 초기 셋업에서 켜지 않으면
나중에 소급 적용이 어렵다. 그래서 M0 스캐폴딩 체크리스트에 못 박았다.

### D3 · Prisma를 검토하고 기각했다

Prisma를 1차 후보로 검토했다. 우위는 실재한다 — `schema.prisma` 단일 명세, `select`/`include`에 따른
정확한 타입 추론, 스키마리스 DB에 정형 구조 강제, Prisma Studio.

**결정을 뒤집은 것은 Prisma의 결함이 아니라 전제의 변화다.** 7개 근거 중 **결정을 주도한 것은 5번**이다 —
유일하게 시간에 민감했기 때문이다.

1. 이관 대상이 Spring Data MongoDB면 "관계형 이관의 다리"라는 Prisma의 1번 강점이 성립하지 않는다.
2. **집계 쿼리가 타입 밖으로 나간다.** `$dateTrunc`·`$unwind`·조건부 카운트는 전부 `aggregateRaw`로 내려가
   `Prisma.JsonValue`가 되고, raw 파이프라인은 `@map`된 DB 필드명을 쓰므로 **필드명을 바꿔도 컴파일 에러 없이
   조용히 빈 결과**를 낸다. 타입 추론이 가장 필요한 곳에서 적용되지 않는다.
3. change streams API가 없어 네이티브 드라이버 병행이 필요하다 — 단 현 기획(SSE 요청 스코프 + 폴링)에서는 조건부 근거.
4. **레플리카셋 없이는 평범한 CRUD조차 돌지 않는다.** Prisma는 중첩 쓰기 부분 저장을 막으려 일반 쓰기 경로에서도
   내부적으로 트랜잭션을 쓴다. Mongoose는 트랜잭션·change streams를 쓸 때만 레플리카셋이 필요하다.
5. **마이그레이션 이력을 확보할 수 없다.** Prisma Migrate는 MongoDB 미지원이고 추가 계획도 없다. `db push`뿐이라
   파일도 이력도 남지 않는다. **DB가 비어 있는 지금이 깨끗한 이력을 시작할 유일한 타이밍이었다.**
6. Prisma 7.x는 MongoDB 미지원이라 v6.19에 묶인다. 향후 v7 지원 시 또 한 번 마이그레이션이 발생한다.
7. `@nestjs/mongoose` 공식 통합을 쓸 수 있다.

**정직하게 감수한 손실**은 CRUD 정적 타입 추론의 약화다. 다만 NestJS DTO 계층이 상당 부분을 덮는데,
**덮는 방향이 비대칭**이다 — 인바운드는 `ValidationPipe`가 **검증**하고, 아웃바운드는 `ClassSerializerInterceptor`가
**필터링만** 한다(응답 DTO는 검증되지 않는다).

남는 잔여 리스크는 한 줄로 좁혀진다 — **쿼리 구성 오타와 프로젝션/DTO 불일치가 조용히 실패한다.**
`FilterQuery<T>`가 임의 키를 허용하고, `.select('name')`이 반환 타입을 좁히지 않아
`doc.email`이 컴파일을 통과한 뒤 런타임 `undefined`가 되고, 시리얼라이저가 그걸 걷어내면서 **오히려 숨긴다.**

그래서 M0에 세 가지를 넣었다.

1. **Zod를 레포지토리 경계에** — DTO와 중복이 아니다. Zod는 "DB에서 실제로 나온 모양", DTO는 "내보내겠다고 약속한 모양".
2. **레포지토리 메서드는 `FilterQuery<T>`를 받지 않는다** — `findByClassId(classId: string)`처럼 도메인 파라미터만.
3. **레포지토리 메서드당 `mongodb-memory-server` 왕복 테스트** — Prisma가 타입으로 공짜로 주던 것을 테스트로 사는 정직한 거래.

### D6 · D10 — 무계정 학생 식별과 인증 경계

> ⚠️ **D6의 식별자 부분은 [[2026-09-09-edu-vibe-student-id-pii]]로 대체됐다.**
> 학생 id가 URL과 S3 경로에 실려 **주소만 봐도 이름이 읽혔기 때문**이다(EDU-650).
> 지금은 무작위 `stu-*`이고 신원은 `(classroomProjectId, nickName)` 부분 unique 인덱스가 보장한다.
> 아래는 그 전환의 출발점이 된 원래 판단이므로 그대로 둔다 — 무계정·upsert·정규화·JWT는 지금도 유효하다.

V1의 초안은 **localStorage `deviceId`(UUID v4)** 기반이었다. 최종은 다르다.

- 학생 식별자는 **`{entryCode}-{nickName}`**이고, 입장 시 User와 Artifact를 **upsert 한 연산**으로 만든다.
  이전 초안의 "member 확인 → 분기 → 두 번 쓰기"를 합친 것이다 — **단일 노드 mongod라 트랜잭션이 없어서**,
  쓰기가 두 번이면 중간에 깨졌을 때 "멤버엔 있는데 작업물은 없는" 상태가 조용히 남는다.
- **`entryCode`는 재발급하지 않는다.** 재발급하면 기존 학생 id가 전부 무효가 되고 UC04(이어하기)가 끊긴다.
  재발급이 필요해지면 `id`를 자체 값으로 두고 `(entryCode, nickName)` 복합 unique로 바꾼다.
- **이름은 정규화 후 저장한다**(trim + 연속 공백 축약). 안 하면 `"김민준"`과 `"김민준 "`이 다른 학생이 되어
  unique 제약이 공백 하나로 뚫린다.
- 토큰 저장은 **`sessionStorage`에 액세스 토큰 하나만.** 역할·이름·범위는 저장하지 않고 `GET /auth/me`로 조회한다.
  저장된 role로 접근을 제어하면 DevTools에서 `student`를 `teacher`로 바꾸는 것만으로 교사 화면이 열린다.
  교실 공용 PC라 탭을 닫으면 토큰이 사라져야 한다는 요구도 여기서 만난다.
- 인가는 두 겹이다. 전역 `JwtAuthGuard`(기본 거부, `@Public()`만 예외) + `@Roles`, 그 위에
  학생은 `assertArtifactScope`(토큰의 userId·수업 범위 대조), 교사는 `assertAccess`(학급 소유권 DB 조회).
  **둘 중 하나라도 빠지면 학생이 URL의 `studentId`만 바꿔 남의 작업물에 접근한다.**

**감수한 리스크가 하나 있다.** 이미 쓰인 이름을 치면 그 사람의 작업으로 들어간다.
MVP 단계 임시 로그인으로 수용했다 — 교실에서 교사 감독이 통제 수단이고, 막는 비용(PIN·명단 등록)이 교사 부담을 늘린다.

### D9 — 인프라를 더하지 않은 근거

- **Redis**: 공유 상태가 필요한 지점이 없다. 세션 인증을 택하면 Redis가 필요해지는데, 그건
  **팀 경계를 가로지르는 상태 인프라를 인증 계층에서 부활시키는 것**이라 배제했다.
- **MQ(Kafka/BullMQ)**: 큐가 정당화되는 조건(요청보다 오래 사는 작업, 스파이크 배압, 다수 소비자 팬아웃,
  보장 재시도/DLQ) 중 해당하는 것이 없다. LLM 호출은 SSE 대화형 스트림이라 큐와 상극이고, BullMQ는 Redis 의존이라 위 결정과 충돌한다.
- **금지선은 하나다** — 큐를 나중에 각 팀이 자기 안에 들이는 것은 자유지만,
  **두 서버가 공유하는 브로커(이벤트 버스)**는 팀 간 계약이라 공동 결정이 필요하다.
- MongoDB 병목: MVP 최대 추정이 초당 수십 ops라 용량 계획 불필요. 단 쿼리 위생 3가지는 처음부터 —
  ① 폴링·조회 경로 인덱스(+TTL) ② 화면당 쿼리 한 방(N+1 금지 — Atlas 왕복 ~50ms는 병목이 아니라 상수라 **왕복 횟수가 UX를 결정**한다) ③ 커넥션 수 = 풀 크기 × 파드 수.

## 기각한 대안

**(A) 워크스페이스 4개 완전 분리 (V1 원안)** — 기각이 아니라 **Plan B로 보류**. Devth가 SDK를 준다는 전제 위에서만
Plan A가 이긴다. 전제가 바뀌면 되돌린다. → D1

**(B) Fastify 5 + TypeBox** — 실험으로 이점을 증명하고도 기각. 표준프레임워크 이관 전제가 이겼다.
실험이 증명한 **원칙**은 NestJS 장치로 이식했다. → D2

**(C) Prisma + MongoDB** — 5번(마이그레이션 이력 부재)이 결정을 주도. 나머지 6개는 결정을 지지하되 조건부다. → D3

**(D) WebSocket** — 이 제품의 실시간 요구는 전부 서버→클라이언트 단방향이고, 학교 필터링 장비가
Upgrade 핸드셰이크를 자주 차단한다. 양방향 UC(실시간 협업 편집)는 기획상 Out of Scope. → D5

**(E) 세션 기반 인증 + Redis** — 서버 2개 + 멀티 파드 환경에서 공유 세션 저장소를 요구한다.
JWT는 서명 시크릿만 있으면 어느 파드든 동일하게 동작한다. 대가(만료 전 철회 불가)는 TTL을 짧게 가져가 흡수한다. → D6, D9

**(F) `artifactId` 단일 키** — 프론트 라우트가 3중 키를 들고 있으므로 단일 키를 쓰면
프론트↔서버 id 변환 계층이 하나 생긴다. → D8

**(G) ObjectId를 도메인 id로** — 전부 24자 hex라 형식으로 구분되지 않아,
잘못된 id가 예외가 아니라 "빈 결과"로만 드러난다. → D7

**(H) 프론트에서 `studentId`를 세션 사용자와 대조** — URL 값은 사용자가 바꿀 수 있어 인가는 전적으로 서버 일이다.
프론트에서 먼저 막으면 **보안 기능처럼 보이는 코드만 늘고 서버의 인가 누락을 가린다.** → D10

## 설계 후 코드에서 확정된 것 (2026-09-09 확인)

노션 문서가 「미결」로 두었던 것들이 구현에서 닫혔다. 문서만 읽으면 아직 열려 있는 것처럼 보인다.

- **SSE 인증 토큰 전달 방식(V1 §9 미결: 쿼리 파라미터 vs 쿠키)이 사라졌다.**
  **EventSource를 쓰지 않기 때문**이다 — `POST` + `fetch` + `response.body.getReader()`로
  스트림을 읽으므로 `Authorization` 헤더가 그대로 실린다. EventSource가 헤더를 못 싣는다는 제약 자체가 적용되지 않는다.
  서버도 대칭으로, **헤더를 열기 전에 인가를 끝내** 실패가 SSE 프레임이 아니라 평범한 403/404로 나가게 했다.
  프레임은 `data: {type:"user"|"delta"|"done"|"error"}` + 종료 시 `data: [DONE]`.
- **S3가 붙었다.** 쓰기는 서버만 하고(브라우저에 PUT 서명을 주지 않는다) 읽기는 15분 서명 URL이다.
  버킷은 비공개, `ACL` 없음, `CacheControl`은 짧게(같은 키를 덮어쓰므로 `immutable` 금지).
  **설정이 없으면 미리보기만 꺼지고 나머지는 뜬다** — 필수로 만들면 AWS 접근 없이 로컬·e2e가 아예 못 뜬다.
- **배포는 k8s + nginx**로 갔다. 프론트는 `/api`를 백엔드로 프록시하고, 환경별 값은
  배포 매니페스트(`gitops-apps`) overlay가 만드는 `/config.js`를 런타임에 읽는다 —
  **한 이미지를 dev·stage·운영에 그대로 쓴다.** 정적 에셋은 CDN 도메인도 런타임 값이다.
- **LLM은 사내 TokenHub를 OpenAI SDK로 부른다**(`baseURL`만 바꾸고 `apiKey`는 검사되지 않아 더미).
  시스템 프롬프트는 Langfuse에 두고 `prompt_name`으로 참조하는데, **비어 있으면 아예 안 보내
  프롬프트 없이 개발이 진행된다.** 스트리밍(채팅)과 논스트리밍(추천 질문 생성) 두 경로가 있다.
- **환경변수는 부팅 시점에 세운다.** `class-validator`로 형태까지 검증하고
  (`MONGODB_URI`는 스킴, `JWT_SECRET`은 16자 이상, `CORS_ORIGINS`는 오리진 목록 정규식),
  **기본값을 두는 것은 `PORT` 하나뿐**이다 — 틀려도 「안 붙는다」로 즉시 드러나고 부팅 로그에 남기 때문이다.
  나머지는 잘못된 값으로 뜬 서버가 조용히 위험하므로 부팅을 세운다.

## 실제로 물린 곳 (설계가 예측하지 못한 것)

- **`User`만 도메인 id를 `_id`가 아니라 별도 `id` 필드에 둔다**(다른 다섯 컬렉션은 `_id`에 직접 넣는다).
  학생 id를 `{entryCode}-{nickName}`으로 두는 보장을 지키려는 예외인데, 서버 코드가 `_id`로 조회하는 바람에
  **교사 로그인이 401만 냈다.** 예외도 아니고 404도 아니고, 아무 일도 일어나지 않는 형태였다.
- **`autoIndex: false`라 스키마의 `unique: true`는 인덱스를 만들지 않는다.** 방어선은 마이그레이션이다.
  `pnpm migrate:up`을 빼면 서버는 정상으로 뜨지만 중복 입장코드·중복 학생이 오류 없이 통과한다.
- **`(classroomId, projectId)` unique 인덱스가 없었다.** 3중 키 경로 설계는 학급×교안 조합에 프로젝트가
  하나뿐이라는 전제 위에 서 있는데, 제약이 없으면 URL이 어느 문서를 가리키는지 모호해진다.
- **결정을 확정하고도 아무 데도 넣지 않은 것이 있었다.** `정렬 파라미터`는 확정 표시까지 됐는데 테스트에도
  구현에도 들어가지 않았고, 그 상태로 GREEN·lint·OpenAPI가 전부 통과했다. 사람이 일주일 뒤 발견했다.
  **어떤 검증도 "빠진 것"은 잡지 못한다.**
- **결정을 개별 항목으로 물으면 상호작용이 드러나지 않는다.** 「CSV는 프론트가 만든다」와
  「무한스크롤은 cursor로」는 각각 타당했지만 **함께 채택하면 서로를 비싸게 만드는 조합**이었고,
  5일 뒤 CSV가 서버로 되돌아왔다.

## 무엇이 바뀌면 이 판단이 무효가 되나

- **Devth 학습창이 독립 앱/서비스로 온다** → D1이 Plan B(4앱 분리)로 회귀하고, 서버–DB 경계 1안/2안 결정이 다시 열린다.
- **데이터스토어가 RDB로 바뀔 가능성이 생긴다** → D3의 Prisma 기각 근거 1·5·6이 한꺼번에 해소된다.
  이때 성립하는 것은 Prisma 회복분(Migrate, v7, `$queryRaw<T>`, JOIN, 트랜잭션)뿐 아니라
  RDB 자체의 이득(View로 집계 이관, FK로 고아 참조 차단, JSONB, `LISTEN/NOTIFY`)까지다.
- **MongoDB가 레플리카셋으로 구성된다** → 트랜잭션·change streams가 열리고, D6의 "단일 노드라 upsert 한 방" 근거가 약해진다.
- **실시간 요구가 양방향으로 확대된다** → D5(SSE)와 D3의 3번(change streams) 근거가 동시에 재평가 대상이 된다.
- **"학생 즉시 차단" 요구가 생긴다** → D6의 JWT가 denylist를 요구하고, 그건 상태 재도입이다.
- **전자정부표준프레임워크 이관 전제가 사라진다** → D2의 결정 근거가 통째로 사라지고 Fastify 실험 결과가 다시 유효해진다.

## 남은 것

- ~~`openapi.json`을 "참고 문서"에서 "계약"으로 승격~~ — **2026-10-06 승격 완료** (사용자 확인).
  경위는 [[cross-repo-api-contract-drift]]
- 프론트 회귀 테스트 0건 — **보류가 아니라 결정이다.** [[2026-09-09-edu-vibe-front-no-tests]]

## 변경 이력

- 2026-10-06: confirmed 승격. "남은 것"에서 `openapi.json` 계약 승격 완료 표시, 한글 정렬 불일치·spec 병렬 흔들림·`--forceExit` 항목은
  사용자 지시로 삭제
- 2026-09-09: D6의 식별자 부분이 [[2026-09-09-edu-vibe-student-id-pii]]로 대체됨.
  코드에서 확정된 것(SSE 인증·S3·배포 형태·TokenHub·env 검증)을 절로 추가
- 2026-09-09: 최초 작성. 노션 V1/V2 Tech Spec, DB 스키마 문서, 설계 원칙 문서, 서버·프론트 개발회고,
  MVP 배포 준비, Fastify 도입 제안, 양쪽 레포 `docs/references.md`에서 결정과 기각 근거를 이관.
  `index.md`에 "git 이력에만 존재"로 남아 있던 **Prisma-MongoDB 기각 근거**와 **Express 어댑터 선택**을 여기서 해소했다
