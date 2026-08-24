---
title: NestJS e2e test harness — the ways a suite passes without proving anything
area: developer
tags: [nestjs, jest, mongodb-memory-server, migrate-mongo, testing, pitfalls]
created: 2026-08-21
updated: 2026-08-21
status: confirmed
---

# NestJS e2e test harness — the ways a suite passes without proving anything

> Built while putting the Edu Vibe server under test (2026-08-21). Companion to
> [[nestjs-mongoose-pitfalls]]: same failure shape, one layer up. A broken harness does not turn the
> suite red — it turns it **green for the wrong reason**, which is worse than having no tests.

## 1. `autoIndex: false` means unique constraints do not exist in tests

When the app boots with `autoIndex: false` (indexes created only by `migrate-mongo`), a test database
built from the schemas alone has **no unique indexes at all**. Every duplicate-prevention test then
passes while proving nothing: the second insert simply succeeds.

Fix: run the real migration files against the in-memory database in the test bootstrap.

```ts
const files = readdirSync(MIGRATIONS_DIR).filter((f) => f.endsWith('.js')).sort()
for (const file of files) {
  const migration = requireMigration(join(MIGRATIONS_DIR, file))
  await migration.up(db)
}
```

Using the actual migrations (not a hand-written index list) has a second payoff: schema-vs-migration
drift — a renamed collection, a missing index — fails in tests instead of in production.
Worth adding one test that asserts the indexes **exist**, so this whole layer cannot silently rot.

## 2. Cleaning with `dropDatabase` undoes item 1

Per-test isolation by dropping the database also drops the migration-created indexes, so the suite
degrades back to case 1 after the first test. Delete documents, keep collections:

```ts
const collections = await connection.db.collections()
await Promise.all(collections.map((c) => c.deleteMany({})))
```

## 3. A test app that *replicates* global config will drift from production

Global `ValidationPipe` / `ClassSerializerInterceptor` written inline in `main.ts` and copy-pasted into
the test bootstrap is a time bomb: the day they diverge, validation and serialization behave one way in
tests and another in production. Extract once and have both call it:

```ts
// src/app.setup.ts
export function configureApp(app: INestApplication) { /* pipes + interceptors */ return app }
```

Same idea for auth: mint **real JWTs** via `app.get(JwtService)` instead of overriding the guard, so
role checks, expiry, and Bearer parsing stay on the production path.

## 4. Assert the whole response key set, not individual fields

Under `excludeExtraneousValues`, a missing `@Expose()` deletes a field silently (see
[[nestjs-mongoose-pitfalls]] item 1). One assertion catches both omission and over-exposure:

```ts
expect(Object.keys(body).sort()).toEqual(['entryCode', 'isSkeleton', 'memberCount', 'projectId'])
```

## 5. Tooling traps that cost more time than the domain work

- **Dynamic `import()` fails under ts-jest CJS** — `A dynamic import callback was invoked without
  --experimental-vm-modules`. Reading CommonJS files (migrations) needs
  `createRequire(__filename)` instead.
- **Parallel `mongodb-memory-server` is flaky** — with `maxWorkers: 2`, ~5 of 49 tests flipped between
  runs while passing in isolation. Pin e2e to `maxWorkers: 1`; the wall-clock cost is small compared to
  chasing phantom failures.
- **jest has no built-in HTML report.** The only built-in HTML output is coverage
  (`coverage/lcov-report/index.html`); a Playwright-style dashboard needs a reporter such as
  `jest-html-reporters`. Its `publicPath` resolves from **cwd, not `rootDir`**, so `../reports` from a
  config living in `test/` writes outside the repo.

## 6. Millisecond timestamps do not give a total order

`sort({ createdAt: -1 })` is unstable when documents are created in a tight loop — they share a
millisecond and the order becomes arbitrary (natural order in one run, reverse in another). Any
fan-out that creates N documents at once hits this. Add the `_id` as a final tie-break whenever a list
order is user-visible.

## Changelog

- 2026-08-21: created while adding the first tests to the Edu Vibe server
