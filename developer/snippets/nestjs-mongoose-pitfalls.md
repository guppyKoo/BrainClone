---
title: NestJS + Mongoose pitfalls that fail silently
area: developer
tags: [nestjs, mongoose, swagger, migrate-mongo, pitfalls]
created: 2026-08-14
updated: 2026-08-14
status: confirmed
---

# NestJS + Mongoose pitfalls that fail silently

> Hit while building the Edu Vibe server (2026-08-14). All four share one trait: **nothing throws**.
> They surface as a missing field, an empty doc, or a re-run — which is why they are worth writing down.
> Stack context: [[stack]] · related decision record: [[mongodb-cascade-strategies]]

## 1. `ClassSerializerInterceptor` drops fields you forgot to `@Expose`

Turning on the global interceptor with `excludeExtraneousValues: true` is the right defense against
leaking response fields — but it is **filtering, not validation**. A response DTO property without
`@Expose()` does not error; it just disappears from the payload.

```ts
app.useGlobalInterceptors(
  new ClassSerializerInterceptor(app.get(Reflector), { excludeExtraneousValues: true }),
)
```

Worse, it *hides* an upstream bug: if a projection forgot a field, the value is `undefined` and the
serializer strips it, so the response looks intentional. Keep one reference DTO in the repo and treat
"every property has `@Expose`" as a review checklist item.

## 2. `@nestjs/swagger` emits no response schema without the CLI plugin

TypeScript return types are erased at runtime, so `async check(): Promise<HealthResponseDto>` tells
Swagger nothing. The generated `openapi.json` came out with `responses.200` empty and
`components.schemas` **completely empty** — even though every DTO property had `@ApiProperty()`.

Fix either way:

```jsonc
// nest-cli.json — infers types, and mostly removes the need for @ApiProperty
{ "compilerOptions": { "plugins": [{ "name": "@nestjs/swagger", "options": { "introspectComments": true } }] } }
```

```ts
// or declare it per handler
@ApiOkResponse({ type: HealthResponseDto })
```

This matters more than it looks when the frontend generates its client from that JSON — an empty
schema silently produces `any`/`void` types instead of failing the codegen.

## 3. Prettier reformatting a migration re-runs it under `useFileHash: true`

`migrate-mongo` with `useFileHash: true` decides "already applied?" by hashing the file. A formatter
touching an applied migration changes the hash, so it goes back to **pending** and runs again on the
next `migrate:up`.

Idempotent operations (`createIndex`) survive it; a data backfill would not. Add `migrations/` to
`.prettierignore` — the hash check is still worth keeping, since it is exactly what catches someone
editing an already-applied migration.

## 4. `JwtService.signAsync` — `expiresIn` wants an `ms` literal type

`expiresIn: config.getOrThrow<string>('JWT_TTL')` fails to compile: the option is typed
`number | StringValue`, and `StringValue` is a template-literal union from the `ms` package, so a plain
`string` is not assignable.

Storing the TTL as **seconds (number)** instead of `'12h'` fixes the type and removes a whole class of
format typos from env config.

## Changelog

- 2026-08-14: created from the Edu Vibe server build
