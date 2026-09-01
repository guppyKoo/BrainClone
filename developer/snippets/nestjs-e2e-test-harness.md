---
title: NestJS e2e 테스트 하네스 — 아무것도 증명하지 않고 통과하는 방법들
area: developer
tags: [nestjs, jest, mongodb-memory-server, migrate-mongo, testing, pitfalls]
created: 2026-08-21
updated: 2026-09-01
status: confirmed
---

# NestJS e2e 테스트 하네스 — 아무것도 증명하지 않고 통과하는 방법들

> Edu Vibe 서버에 테스트를 붙이면서 만들었다(2026-08-21). [[nestjs-mongoose-pitfalls]]의 짝이다:
> 같은 실패 형태, 한 레이어 위. 망가진 하네스는 스위트를 빨갛게 만들지 않는다 —
> **엉뚱한 이유로 초록**으로 만든다. 테스트가 아예 없는 것보다 나쁘다.

## 1. `autoIndex: false`면 테스트에 유니크 제약이 존재하지 않는다

앱이 `autoIndex: false`로 부팅하면(인덱스는 `migrate-mongo`만 생성) 스키마만으로 만든 테스트
데이터베이스에는 **유니크 인덱스가 하나도 없다**. 그러면 중복 방지 테스트가 전부 통과하면서
아무것도 증명하지 않는다: 두 번째 insert가 그냥 성공한다.

해결: 테스트 부트스트랩에서 실제 마이그레이션 파일을 인메모리 데이터베이스에 실행한다.

```ts
const files = readdirSync(MIGRATIONS_DIR).filter((f) => f.endsWith('.js')).sort()
for (const file of files) {
  const migration = requireMigration(join(MIGRATIONS_DIR, file))
  await migration.up(db)
}
```

손으로 쓴 인덱스 목록이 아니라 실제 마이그레이션을 쓰면 두 번째 이득이 있다: 스키마와 마이그레이션의
드리프트(이름이 바뀐 컬렉션, 빠진 인덱스)가 프로덕션이 아니라 테스트에서 터진다.
인덱스가 **존재하는지** 확인하는 테스트를 하나 추가할 가치가 있다. 이 레이어 전체가 조용히 썩는 걸 막는다.

## 2. e2e 테스트에서 파괴적인 데이터베이스 정리를 하지 말 것

`dropDatabase`도, 컬렉션 전체 `deleteMany({})`도 e2e 하네스에 있을 것이 아니다. `dropDatabase`는
마이그레이션이 만든 인덱스를 지우고, `deleteMany({})`는 연결 격리가 한 번이라도 무너지면 개발자의
데이터베이스를 날릴 수 있다. 후자는 스위트가 `mongodb-memory-server`를 쓰려고 했는데도 Edu Vibe에서
실제로 일어났다.

각 spec에 자기만의 일회용 MongoDB 인스턴스를 주고, 픽스처 데이터는 생성된 도메인 ID로 스코프하고,
고정 픽스처는 upsert로 멱등하게 만들고, spec이 끝나면 인스턴스를 폐기한다. 더 강한 격리가 필요한
테스트라면 데이터를 지우는 대신 그 테스트용으로 새 일회용 데이터베이스/애플리케이션을 만든다.

## 3. 전역 설정을 *복제하는* 테스트 앱은 프로덕션과 어긋난다

`main.ts`에 인라인으로 쓴 전역 `ValidationPipe` / `ClassSerializerInterceptor`를 테스트 부트스트랩에
복붙하는 것은 시한폭탄이다. 둘이 갈라지는 날 검증과 직렬화가 테스트에서 한 가지로, 프로덕션에서
다른 가지로 동작한다. 한 번만 추출하고 양쪽이 그것을 호출하게 한다:

```ts
// src/app.setup.ts
export function configureApp(app: INestApplication) { /* pipes + interceptors */ return app }
```

인증도 같은 발상이다: 가드를 오버라이드하지 말고 `app.get(JwtService)`로 **진짜 JWT**를 발급해서
역할 검사, 만료, Bearer 파싱이 프로덕션 경로에 남아 있게 한다.

## 4. 개별 필드가 아니라 응답 키 집합 전체를 단언할 것

`excludeExtraneousValues` 아래에서는 `@Expose()` 누락이 필드를 조용히 지운다
([[nestjs-mongoose-pitfalls]] 1번 참고). 단언 하나가 누락과 과다 노출을 동시에 잡는다:

```ts
expect(Object.keys(body).sort()).toEqual(['entryCode', 'isSkeleton', 'memberCount', 'projectId'])
```

## 5. 도메인 작업보다 시간을 더 잡아먹은 툴링 함정들

- **ts-jest CJS에서 동적 `import()`가 실패한다** — `A dynamic import callback was invoked without
  --experimental-vm-modules`. CommonJS 파일(마이그레이션)을 읽으려면 대신
  `createRequire(__filename)`이 필요하다.
- **`mongodb-memory-server` 병렬 실행은 불안정하다** — `maxWorkers: 2`에서 49개 중 5개쯤이 실행마다
  결과가 뒤집혔다. 단독으로 돌리면 통과하는데도. e2e는 `maxWorkers: 1`로 고정할 것. 유령 실패를
  쫓는 비용에 비하면 벽시계 시간 손해는 작다.
- **jest에는 내장 HTML 리포트가 없다.** 유일한 내장 HTML 출력은 커버리지
  (`coverage/lcov-report/index.html`)뿐이고, Playwright 스타일 대시보드는
  `jest-html-reporters` 같은 리포터가 필요하다. 이 리포터의 `publicPath`는 **`rootDir`이 아니라 cwd**
  기준으로 해석되므로, `test/`에 있는 설정에서 `../reports`를 쓰면 저장소 바깥에 쓴다.

## 6. 밀리초 타임스탬프는 전순서를 주지 않는다

`sort({ createdAt: -1 })`는 문서가 촘촘한 루프에서 생성되면 불안정하다 — 같은 밀리초를 공유하면서
순서가 임의가 된다(어떤 실행에서는 자연 순서, 다른 실행에서는 역순). 한 번에 N개 문서를 만드는
팬아웃이라면 전부 이 문제를 만난다. 목록 순서가 사용자에게 보인다면 `_id`를 최종 타이브레이크로 넣을 것.

## 7. `AppModule`을 정적으로 import한 뒤라면 `compile()` 전에 env를 설정해도 이미 늦었다

`ConfigModule.forRoot()`는 `Test.createTestingModule(...).compile()`이 실행될 때가 아니라 모듈 파일이
**평가될 때** `.env`를 읽는다. 테스트 부트스트랩이 `MONGODB_URI`를 일찍 설정한 것처럼 보여도,
최상위의 `import { AppModule } ...`이 이미 로컬 개발용 URI를 붙잡은 뒤일 수 있다.

Edu Vibe에서 실제로 일어났다: e2e 테스트가 조용히 로컬 `vibe` 데이터베이스에 연결했고, 그 외에는
멀쩡했던 `deleteMany` 정리가 거기에 실행됐다. 발견했을 때는 테스트 픽스처 문서 세 개만 남고 주요
컬렉션이 비어 있었다. 원래 로컬 데이터가 있었는지는 확인할 수 없었다.

애플리케이션 모듈은 테스트 환경을 설치한 *뒤에* 로드하고, 마이그레이션이나 정리를 실행하기 전에
독립적인 두 번째 가드를 추가한다:

```ts
process.env.MONGODB_URI = mongo.getUri(dbName)

const { AppModule } = requireFromTest('../../src/app.module') as typeof import('../../src/app.module')
const moduleRef = await Test.createTestingModule({ imports: [AppModule] }).compile()
const app = configureApp(moduleRef.createNestApplication())
await app.init()

const connection = app.get<Connection>(getConnectionToken())
if (connection.name !== dbName) {
  throw new Error(`Test DB isolation failed: expected=${dbName}, actual=${connection.name}`)
}
```

일반 규칙은 "테스트 env를 설정하라"보다 강하다: **마이그레이션이나 테스트를 실행하기 전에 해석된
연결이 일회용인지 단언하고, 파괴적인 정리는 하네스에서 아예 빼라.** 망가진 테스트 하네스는 잘못된
결과를 내는 데서 그치지 않고 데이터를 손상시킬 수 있다.

## 변경 이력

- 2026-09-01: 문서를 한국어로 전환
- 2026-08-24: 모듈 평가 순서 버그로 `deleteMany({})`가 로컬 데이터베이스를 가리킨 뒤 파괴적 e2e 정리를 금지.
  일회용 데이터베이스, 멱등 픽스처, 닫히는 쪽으로 실패하는 가드를 쓸 것
- 2026-08-21: Edu Vibe 서버에 첫 테스트를 추가하면서 생성
