---
title: 조용히 실패하는 NestJS + Mongoose 함정들
area: developer
tags: [nestjs, mongoose, swagger, migrate-mongo, validation, pitfalls]
created: 2026-08-14
updated: 2026-09-09
status: confirmed
---

# 조용히 실패하는 NestJS + Mongoose 함정들

> Edu Vibe 서버를 만들면서 밟은 것들(2026-08-14 이후). 공통점이 하나 있다: **아무것도 throw하지 않는다.**
> 필드가 하나 없거나, 문서가 비어 있거나, 마이그레이션이 다시 도는 형태로만 드러난다. 그래서 적어둘 가치가 있다.
> 스택 맥락: [[stack]] · 관련 결정 기록: [[mongodb-cascade-strategies]]
> 이 스택을 고른 근거: [[2026-09-09-edu-vibe-architecture]] · 레포 경계 쪽 짝: [[cross-repo-api-contract-drift]]

## 1. `ClassSerializerInterceptor`는 `@Expose`를 빠뜨린 필드를 삭제한다

`excludeExtraneousValues: true`로 전역 인터셉터를 켜는 것은 응답 필드 유출을 막는 올바른 방어다 —
다만 이것은 **필터링이지 검증이 아니다.** `@Expose()`가 없는 응답 DTO 속성은 에러를 내지 않고
그냥 페이로드에서 사라진다.

```ts
app.useGlobalInterceptors(
  new ClassSerializerInterceptor(app.get(Reflector), { excludeExtraneousValues: true }),
)
```

더 나쁜 건 상위의 버그를 *가린다*는 점이다. projection이 필드를 빠뜨리면 값은 `undefined`가 되고
시리얼라이저가 그걸 잘라내므로 응답이 의도된 것처럼 보인다. 저장소에 기준 DTO를 하나 두고
"모든 속성에 `@Expose`가 있는가"를 리뷰 체크리스트 항목으로 다룰 것.

## 2. CLI 플러그인 없이는 `@nestjs/swagger`가 응답 스키마를 만들지 않는다

TypeScript 반환 타입은 런타임에 지워지므로 `async check(): Promise<HealthResponseDto>`는 Swagger에
아무것도 알려주지 않는다. 생성된 `openapi.json`은 `responses.200`이 비어 있고
`components.schemas`가 **완전히 비어** 있었다 — 모든 DTO 속성에 `@ApiProperty()`가 붙어 있었는데도.

둘 중 아무 방법으로나 고칠 수 있다:

```jsonc
// nest-cli.json — 타입을 추론하고, @ApiProperty의 필요를 대부분 없앤다
{ "compilerOptions": { "plugins": [{ "name": "@nestjs/swagger", "options": { "introspectComments": true } }] } }
```

```ts
// 또는 핸들러마다 선언
@ApiOkResponse({ type: HealthResponseDto })
```

프론트엔드가 그 JSON으로 클라이언트를 생성한다면 보이는 것보다 훨씬 중요하다 — 빈 스키마는
코드젠을 실패시키는 대신 조용히 `any`/`void` 타입을 만들어낸다.

## 3. `useFileHash: true`에서 Prettier가 마이그레이션을 재포맷하면 다시 실행된다

`useFileHash: true`인 `migrate-mongo`는 파일을 해싱해서 "이미 적용됐는가"를 판단한다. 포매터가
이미 적용된 마이그레이션을 건드리면 해시가 바뀌고, 그러면 다시 **pending**이 되어 다음
`migrate:up`에서 또 실행된다.

멱등한 연산(`createIndex`)은 살아남지만, 데이터 백필이라면 살아남지 못한다. `.prettierignore`에
`migrations/`를 추가할 것 — 해시 체크 자체는 유지할 가치가 있다. 이미 적용된 마이그레이션을
누군가 수정하는 것을 잡아내는 게 정확히 그 기능이기 때문이다.

## 4. `JwtService.signAsync` — `expiresIn`은 `ms` 리터럴 타입을 원한다

`expiresIn: config.getOrThrow<string>('JWT_TTL')`은 컴파일되지 않는다. 이 옵션의 타입은
`number | StringValue`이고 `StringValue`는 `ms` 패키지의 템플릿 리터럴 유니온이라, 평범한 `string`은
할당할 수 없다.

TTL을 `'12h'` 대신 **초 단위 숫자**로 저장하면 타입 문제가 해결되고, env 설정에서 포맷 오타라는
버그 유형 하나가 통째로 사라진다.

## 5. 전역 `ValidationPipe`는 쿼리 파라미터를 지켜주지 않는다

`whitelist` / `forbidNonWhitelisted`는 핸들러 인자에 바인딩된 **DTO**에 대해 동작한다.
`@Query('limit') limit?: string`을 받는 핸들러에는 DTO가 없으므로 검증할 대상 자체가 없다 —
`?limit=0`, `?limit=999`, 깨진 커서가 전부 200으로 통과한다. 이 파이프의 보호는 전역처럼 느껴지지만
DTO가 기술한 것만 커버한다.

```ts
// 전역 파이프가 뭐라 하든 검증되지 않는다
async list(@Query('limit') limit?: string) {}

// 검증된다
async list(@Query() query: ListQueryDto) {}
```

또한 Express는 반복된 키(`?projectId=a&projectId=b`)를 **배열**로 파싱하는데, 그게 `string` 타입
파라미터에 검사 없이 도달해 Mongoose 쿼리에 그대로 들어간다.

## 6. 기본 거부(default-deny) 전역 가드에서는 `@Public()` 누락이 401 헬스체크가 된다

전역 가드를 "public으로 표시되지 않으면 인증 필요"로 두는 것은 올바른 기본값이다 — 표시를 잊으면
엔드포인트가 노출되는 대신 닫히는 쪽으로 실패한다. 대가는 **토큰 없이 기계가 호출하는 모든 것을
명시적으로 표시해야 한다**는 점이다. `@Public()`이 없는 헬스 엔드포인트는 401을 반환하고,
로드 밸런서와 프로브는 토큰이 없으므로 인스턴스는 영영 healthy가 되지 않는다. 코드만 봐서는
아무 문제도 없어 보이고, 배포 후에야 드러난다 (테스트가 아니라 Postman으로 손수 찾았다).

값싼 방어책: public 엔드포인트마다 토큰 없이 응답하는지 확인하는 테스트를 하나씩.

## 7. `@IsOptional()`이 Swagger 쿼리 파라미터를 반드시 optional로 만들어주진 않는다

class-validator는 런타임 검증을 제어하고, Nest Swagger CLI 플러그인은 주로 TypeScript 속성 형태에서
필수 여부를 도출한다. 아래 DTO는 `@IsOptional()` 덕분에 런타임에서는 optional로 동작하지만,
속성 자체가 optional이 아니라서 OpenAPI에는 `required: true`로 나갈 수 있다:

```ts
@IsOptional()
@Min(1)
@Max(50)
limit: number = 20
```

TypeScript 계약도 optional로 만들고, 런타임 동작이 초기값 적용 변환에만 의존하지 않도록
서비스 폴백을 함께 남긴다:

```ts
limit?: number = 20

const limit = query.limit ?? 20
```

`openapi:gen`이 성공으로 끝났다는 것은 문서가 생성됐다는 사실만 증명한다. API 문서를 GREEN으로
간주하기 전에 생성된 JSON에서 상태 코드, 파라미터의 필수 여부/기본값, 응답 스키마 참조를 직접 확인할 것.

## 8. body 파서 한도가 `@MaxLength`보다 작으면 400이 아니라 413이 나간다

Express의 기본 body 한도는 **100kb**다. 설정한 곳이 없으면 그 값이 걸린다.
DTO의 `@MaxLength`가 그보다 크면 **그 한도는 도달할 수 없고**, 클라이언트는
"검증 실패(400)"가 아니라 **413**을 받는다 — 에러 형태가 달라 프론트가 다른 분기로 떨어진다.

그리고 **한도는 글자가 아니라 바이트다.** 한글은 UTF-8에서 3배로 센다.
`@MaxLength(50000)`은 한글로 채우면 150KB라 한도를 훨씬 먼저 넘는다.

## 9. 「없는 것」과 「남의 것」은 같은 코드로 통일한다

리소스가 없을 때 404, 남의 것일 때 403으로 갈라 놓으면 **응답 코드가 리소스의 존재 여부를 알려준다.**
id를 바꿔가며 찔러 보면 무엇이 있는지 열거된다.

이 레포는 **둘 다 404로 통일**했다. 대신 로그에는 구분해서 남긴다 —
사용자에게 감추는 것이지 운영자에게 감추는 것이 아니다.

## 10. presigned URL의 수명은 `expiresIn`이 아니라 서명한 자격증명이 정한다

S3 서명 URL은 `expiresIn`과 **서명에 쓴 자격증명의 잔여 수명 중 짧은 쪽**에서 죽는다.
EKS의 IRSA 세션이 한 시간 주기라면, **잔여 8분인 자격증명으로 서명한 URL은 8분 뒤 403**이다.
`expiresIn`을 한 시간으로 적어 두면 지키지 못할 약속이 된다.

SDK의 자격증명 갱신 임계(만료 5분 전)를 감안하면 **최악의 경우 5분 안쪽**까지 떨어진다.
URL을 응답 직후에 쓰는 용도라면 짧게(15분) 두고, 만료가 문제되는 지연 로드는 폴백을 둔다.

같은 계열의 함정이 하나 더 있다 — **자격증명을 환경변수로 줬는지 SDK chain에 맡겼는지 부팅 로그에 남긴다.**
role이 붙지 않은 채 chain으로 빠지면 **부팅은 성공하고 첫 업로드에서** 자격증명 오류가 나는데,
그 시점의 스택만으로는 「키를 안 넣었다」와 「role이 안 붙었다」가 구별되지 않는다.

## 11. CORS `allowedHeaders`를 나열하지 않는 것이 더 나을 수 있다

나열하지 않으면 `cors`가 프리플라이트의 `Access-Control-Request-Headers`를
**그대로 `Access-Control-Allow-Headers`에 돌려준다** — 사실상 어떤 헤더든 허용된다.

나열하는 쪽이 좁지만, **나중에 헤더를 추가하고 목록에 넣는 것을 잊으면 브라우저에서만 막히고
서버 로그에는 아무것도 남지 않는다.** CORS의 방어선은 오리진이지 헤더가 아니므로,
허용된 오리진에 헤더를 더 열어 잃는 것보다 **조용한 실패 지점을 늘리는 비용이 크다**고 판단했다.

같은 이유로 오리진에는 `*`를 쓰지 않는다 — 와일드카드는 쿠키 인증으로 옮길 때 바로 막히고,
**운영에 그대로 새어 나가도 아무 증상이 없다.**

## 변경 이력

- 2026-09-09: 아키텍처 결정 문서와 레포 경계 스니펫으로 역링크 추가
- 2026-09-01: 문서를 한국어로 전환
- 2026-08-24: `@IsOptional()`과 TypeScript 속성 형태 사이의 Swagger 필수 여부 불일치 추가
- 2026-08-21: 5~6번 추가 — 쿼리 파라미터는 전역 파이프 밖이고, 기본 거부 가드는 기계 호출 엔드포인트에
  명시적 `@Public()`이 필요하다
- 2026-08-14: Edu Vibe 서버 구축 과정에서 생성
