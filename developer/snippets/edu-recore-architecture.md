---
title: RECORE — edu-core 서버 분리 구조와 개발 규칙
area: developer
tags: [goorm, edu-core, edu-server, edu-front, recore, Istio, HTTPRoute, gitops-apps, ArgoCD, monolith-split]
created: 2026-09-23
updated: 2026-09-23
status: confirmed
---

# RECORE — edu-core 서버 분리 구조와 개발 규칙

출처: Notion「RECORE 공유 문서」(EDU Channel 테크스펙, 2026-09-22 기준) · Jira 에픽 EDU-1.
레포 안 문서는 `edu-core/docs/recore/` (edu-server에도 같은 문서가 있다).

## 한 줄 요약

모놀리스 edu-core를 EDU와 DEVTH가 **독립적으로 개발·배포·운영**할 수 있도록 점진적으로 나눈다.
기능은 추가하지 않고 구조만 바꾼다. 기준은 **응답 호환**이다. 프론트는 어느 서버가 응답했는지 알 수 없어야 한다.

- 문제: API·SSR·프론트 빌드·소켓·Kafka 컨슈머가 Express 프로세스 하나에 들어 있다.
  두 팀이 같은 코드를 고치니 영향 범위를 예측하기 어렵고, 한쪽 장애가 다른 쪽으로 퍼진다.
- 담당 원칙: **그릇은 edu, 알맹이는 devth.**
  - 그릇(edu): 강좌 개설, 수강생, 공지, 초대, 알림, 채널, 커뮤니티, 시스템
  - 알맹이(devth): 문제, 시험, 응시, 채점, 성적, 실시간 모니터링, AISA
  - 강좌 종류(일반/평가)로 나누면 애매해지지만, 그릇과 알맹이로 나누면 기준이 일관된다.
- ⚠️ 과도기에는 위 원칙이 아니라 **기존에 담당하던 기능**을 기준으로 나눈다
  (예: 평가 관리의 목표 담당은 edu지만 당분간 edu-core에 남는다). 이관 단계마다 넘어가는 기능·API·컬렉션 목록을 적는다.

## 서버 구성

| 서버 | 레포 | 역할 | 받는 트래픽 |
| --- | --- | --- | --- |
| edu-core | goorm-dev/edu-core | 원본 모놀리스. 사실상 devth 서버. web/SSR·프론트 서빙 계속 담당 | HTTPRoute에 없는 모든 요청 (catch-all) |
| edu-server | goorm-dev/edu-server (edu-core fork) | edu API 전용. `USE_DIST=false`로 프론트를 서빙하지 않는다 | HTTPRoute에 등록된 (경로, 메서드)만 |
| edu-front | goorm-dev/edu-front | EDU 페이지 서빙. React Router v7 앱들을 Express 호스트 하나가 URL prefix로 마운트 | PathPrefix `/admin`, `/front-assets` |

```text
브라우저 (단일 origin: <channel>.dev.goorm.io)
  └─ Istio istio-alb-gateway  ← 라우팅 원본 = gitops-apps HTTPRoute (path + method)
       ├─ 등록된 (path, method)      → edu-server (API only)
       ├─ /admin, /front-assets      → edu-front
       └─ catch-all ^/.*$            → edu-core (web/SSR, devth API)
edu-server ──S2S (k8s service DNS, /api/integration/*, apiKey)──> edu-core
두 서버 공유: MongoDB 1개 · Redis(세션·dbcache) · Kafka
```

- 서버 간 호출은 Gateway를 거치지 않고 k8s service DNS로 직접 간다.
- edu-front는 인프라에 접속하지 않는다. 백엔드로는 edu-core API만 호출한다. 브라우저의 `/api/*`는 edu-front를 거치지 않는다.
- **front web 서빙, Kafka consumer, socket은 아직 edu-core가 담당한다.**
- BETA도 같은 구성이다 (beta edu-core → edu-server로 API 요청).

## 라우팅: gitops-apps HTTPRoute

- **단일 원본**: `kubernetes/manifests/goorm/edu-server/overlays/<env>/<cluster>/edu/httproutes/*.yaml`.
  인프라팀을 거치지 않고 개발자가 직접 rule PR을 올린다. 머지 후 **ArgoCD에서 sync해야 반영된다**.
- 매치는 `Exact` 또는 `RegularExpression` 경로에 `method`를 조합한다. Express `:param`은 `[^/]+`로 바꾼다.
- 함정 3가지:
  1. `method`를 생략하면 그 경로의 **모든 메서드**가 함께 넘어간다. 일부만 옮길 때는 반드시 명시한다.
  2. `Exact /api/foo`는 `/api/foo/bar`나 `/api/foo/`(trailing slash)를 잡지 않는다.
     PathPrefix를 쓰려면 그 아래에 아직 안 옮긴 API가 없는지 먼저 확인한다.
  3. rule을 빠뜨려도 **에러가 나지 않는다.** 요청이 조용히 catch-all을 타고 edu-core로 간다.
- 상한: HTTPRoute 하나에 **rule 16개**, rule 하나에 **match 8개**. 넘으면 파일을 나눈다(`lecture-1`~`lecture-10`, `misc-1`~`misc-9`).
  parentRef·hostname이 같으면 Gateway가 병합하므로 나누는 위치는 동작에 영향을 주지 않는다.
- env별로 PR을 따로 올린다. env는 dev(goorm-seoul-dev-beta), beta(goorm-seoul-prod-beta), prod(goorm-seoul-prod-goorm)다.
- 이관된 (경로, 메서드)에 edu-core 핸들러를 새로 추가하면 요청이 edu-core에 닿지 않는다.

## 데이터 분리

- **컬렉션 write는 한 서버만 한다.** 추후 DB 계정 권한도 나눌 계획이다.
  - 예외: `identitycounters` 등은 양쪽에서 write한다. 각 서버는 자기 컬렉션의 카운터만 갱신한다.
- read도 API로 하는 것이 목표지만, 당분간은 직접 read를 허용한다.
- **세션 구조는 바꾸지 않는다.** 두 서버가 같은 Redis 세션을 읽기 때문이다. 필드 추가·삭제는 금지하고, 기존 필드 값 갱신만 허용한다.

### edu-server 쪽 (consumer): port / adapter

`repositories/devth/<domain>/` 아래에 devth 소유 컬렉션 접근을 모았다.

- `index.js` = **port**: 유스케이스 단위 named method. 어느 어댑터를 쓸지 고른다.
- `<domain>.mongo.js` / `<domain>.api.js` = **adapter**: 같은 port를 Mongo 직접 조회와 edu-core API 호출로 각각 구현한다. 호출하는 쪽은 어떤 어댑터가 쓰이는지 모른다.
- `_lib/apiAdapter.js` = api 어댑터의 반환값을 mongo 어댑터와 똑같이 맞추는 호환 계층이다. 바인딩을 바꿔도 호출 코드를 건드리지 않게 하는 **임시 비계이므로 가급적 쓰지 않는다.**
  write-seal → mongo 어댑터·스키마 제거가 끝나면 이 계층도 없애고, 그때 port 계약을 도메인 기준으로 다시 정한다.
- 이름 관계: repository(디렉터리) 안에 port(`index.js`)와 adapter(`*.mongo.js`/`*.api.js`)가 있다.

### edu-core 쪽 (provider): integration API

- `routes/api/integration/{exam,quiz,supervisionGroup,executeLog,history}.js`
- 인증: `useInternalServiceMiddleware()` (goorm-service-client apiKey). 채널 범위는 `x-channel-index` 헤더로 전달한다.

## integration 조회 3계층

| 계층 | 성격 | 예 |
| --- | --- | --- |
| T1 범용 목록·단건 | 조건 조회 후 문서를 그대로 반환. 판단은 edu-server가 한다 | `GET /exams`, `/exams/:index`, `/quizzes`, `/quizzes/:index` |
| T2 비즈니스 조회 | 유스케이스 조건(접두어·category·활성 여부)을 서버 안에서 처리한다. edu-server는 조건을 몰라도 된다 | `/exams/copy-targets`, `/exams/score-targets` |
| T3 판정·계산 | 응시 데이터(`quiz_answers`, `exam_answers`)를 합산·판정한 결과만 반환. 응시 컬렉션에 범용 read를 열지 않으려고 둔 층 | `/exams/in-progress`, `/exams/has-submission`, `/exams/scores`, `/quizzes/achievement`, `/exams/:index/recommendation-context`, `/quizzes/:index/{collaboration,completion}-validity` |

- 잘못 고른 경우: T1로 충분한 것을 T2나 T3로 만들면 endpoint만 늘어난다. 응시 데이터를 T1로 열면 devth 스키마가 계약에 드러난다.

## 새 devth 데이터 접근 추가 순서

1. **edu-server**: `#repositories/devth/<domain>` port에 유스케이스 단위 메서드를 추가한다 (devth 컬렉션에 직접 접근 금지).
2. **edu-core**: `routes/api/integration/<domain>.js`에 핸들러를 추가하고 `integration/index.js`에 마운트한다.
   **기존 서비스 함수를 그대로 호출한다.** 다시 구현하면 edu-server 계약 테스트와 결과가 어긋난다. 먼저 T1/T2/T3 중 어디에 넣을지 정한다.
3. **edu-server**: port 바인딩에 반영한다.

edu-core 작업을 edu-server로 옮길 때는 필요한 기능만 **cherry-pick**한다.

## 로컬 개발환경

운영과 같은 HTTPRoute 파일을 로컬 Envoy가 그대로 읽어 라우팅을 재현한다.
자세한 내용은 gitops-apps `scripts/edu-server-local-dev/README.md`에 있다.

```text
브라우저 → nginx :443 (TLS 종료, Host 보존, WS Upgrade)
        → envoy :18080 (Exact > PathPrefix > Regex)
            ├ rule 일치            → edu-server
            ├ /admin, /front-assets → edu-front (EDU_API_ORIGIN으로 edu-core 호출)
            └ catch-all            → edu-core
```

환경 변수 요지 (값은 기록하지 않음):
- edu-core: `GOORM_SERVICE_CLIENT_API_KEY` — goorm-service-client apiKey
- edu-server: `INTEGRATIONS.DEVTH.HOST`(edu-core 주소), `INTEGRATIONS.DEVTH.TOKEN`(= edu-core의 위 키), `APM_AGENT_SERVER.serviceName: edu-server`, `KAFKA.groupId`(추후 변경)
- edu-front `.env`: `DEV_APP`, `SERVE_APPS`, `EDU_API_ORIGIN`(edu-core origin), `PORT`(dev 53000 / prod 53100), `CDN_HOST`

## gitops-apps 구조

```text
kubernetes/manifests/goorm/
├── edu-server/base/                         # Deployment(USE_DIST=false, /ping probe), Service, SA
├── edu-server/overlays/<env>/<cluster>/edu/
│   ├── kustomization.yaml                   # httproutes 등록, Vault config.json, 로깅 사이드카, edufiles 마운트
│   └── httproutes/{httproute.yaml(catch-all), <module>[-N].yaml}
├── edu-front/overlays/<env>/<cluster>/edu/  # PathPrefix /admin, /front-assets (현재 dev만), Vault 없음
└── edu-core/edu-core/{base, overlays/<env>/<cluster>/{edu,devth}}
argocd/goorm/application/overlays/<env>/<cluster>/edu/edu/{edu-core, edu-server, edu-front(예정)}
```

- edufiles(NFS): edu-server도 edu-core와 같은 스토리지를 `/edufiles`에 마운트한다.

## 배포

- k8s namespace: 기존 **goorm NS**의 edu-core는 단계적으로 없앤다. 신규 **edu NS**에는 edu-core, edu-server, edu-front가 있다.
- edu-core: Jenkins `goorm-edu/edu-core/edu-core`는 기존과 같다. goorm NS와 edu NS가 **같은 이미지**를 쓰므로 **edu NS 쪽도 ArgoCD sync가 필요하다.**
  Vault 경로는 edu NS에서 `/v1/goorm/edu-core/data/config.json`이다 (goorm NS는 appconfig 사용).
- edu-server: Jenkins `goorm-edu/edu-server/{DEV|BETA|OP}`. 프론트는 빌드하지 않는다. Vault: `/v1/goorm/edu-server/data/config.json`
- edu-front: Jenkins `goorm-edu/edu-front/{DEV|BETA|OP}`. env는 Dockerfile과 gitops-apps에 직접 주입한다.
- 신규·이관 API: 코드 배포와 **별개로** HTTPRoute PR → 머지 → ArgoCD sync를 해야 트래픽이 넘어간다.

## 모니터링

- Istio: Grafana(istio ambient mesh 대시보드), **Kiali**(메시 그래프, prod/beta/dev 각각), **Tempo**(세부 trace, TraceQL `{status=error}`)
- Elastic: APM(`edu-server`, `edu-front`), access_log(Discover)
- OpenTelemetry로 trace 관측 체계를 바꾸는 작업이 진행 중이다.

## 트러블슈팅 체크리스트

- **edu-core를 고쳤는데 dev에 반영되지 않음** → 그 (경로, 메서드)가 이미 edu-server로 넘어갔을 수 있다. HTTPRoute를 먼저 본다.
- **내 API가 여전히 edu-core로 감** → rule이 빠졌거나 ArgoCD sync를 안 했다. 에러 없이 catch-all로 빠진다.
- **하위 경로가 안 잡힘** → `Exact`의 한계다. trailing slash도 잡지 않는다.
- **GET만 옮기려 했는데 POST까지 넘어감** → rule에 `method`가 빠졌다.

## 참고 문서

- Notion: RECORE 공유 문서, edu-core 서버 분리 가이드 v2(fork 기반 설계 원본), edu / devth 기능 분담 정리(담당표), EDU 프론트엔드 분리 전략, gitops-apps 구조 가이드
- GitHub: `edu-core/docs/recore`, gitops-apps `scripts/edu-server-local-dev/README.md`, gitops-apps `kubernetes/manifests/goorm/edu-server/overlays`

관련: [[cross-repo-api-contract-drift]] · [[edu-lesson-period-contract]]
