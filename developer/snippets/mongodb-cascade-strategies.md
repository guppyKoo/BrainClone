---
title: MongoDB cascade-delete strategies
area: developer
tags: [mongodb, cascade, change-stream, kafka, transactions, cdc]
created: 2026-08-14
updated: 2026-08-14
status: draft
---

# MongoDB cascade-delete strategies

Reference note from a design discussion (2026-08-14) about replacing the legacy
Kafka-based cascade path. Not yet applied to any codebase.

## Context at work

- The in-house legacy codebase publishes to **Kafka** whenever a cascade delete is needed on MongoDB.
- The app runs as **multiple instances on a single server** (PM2-style, not separate K8s deployments).
- Unresolved: whether the Kafka topic's consumer is the **same service** or a **different service**.
  This single fact decides the whole answer — see Open questions in [[now]].

## Decision table

| Situation                                    | Approach                                     | New infra |
| -------------------------------------------- | -------------------------------------------- | --------- |
| 1:1, always read together                    | Embed — delete the child collection entirely  | none      |
| Same DB, 3–5 collections, replica set        | **Multi-document transaction**                | none      |
| Consistency required at response time        | **Multi-document transaction**                | none      |
| Async OK, single service                     | Change Stream + one dedicated worker          | none      |
| Heavy fan-out or retries needed              | Change Stream + BullMQ                        | Redis     |
| Consumer is a different service              | Keep Kafka, add **Transactional Outbox**      | existing  |
| Immediacy not required                       | Soft delete + batch GC                        | none      |

Default answer for the work case is the **transaction** row — it removes the worker,
the queue, the duplicate-execution problem and eventual-consistency reasoning at once.

## Change Stream — what it actually is

- A public API over the **oplog**, not a DB-side trigger. Requires a replica set
  (`rs.initiate()` gives a single-node replica set for local/dev).
- `watch()` returns a **cursor**, held open for the process lifetime. It is application code,
  so **N processes ⇒ N cursors ⇒ N executions**. There is no consumer-group concept.
  Same class of problem as `@Cron` firing once per instance.
- Fix: run `watch()` in **one** process. Either a separate PM2 app entry gated on `ROLE=worker`,
  or `NODE_APP_INSTANCE === '0'` in cluster mode (breaks the moment a second server is added).
- Only **majority-committed** writes are emitted, so a rolled-back delete can never trigger a cascade.
  Lower bound on latency is therefore replication lag.

### Non-obvious details

- Put `$match` **first** — the pipeline is evaluated server-side.
- `$lookup` / `$group` / `$sort` are not allowed (infinite stream).
- `delete` events carry only `_id`. For the deleted body, enable pre-images
  (`changeStreamPreAndPostImages`, MongoDB 6.0+).
- Prefer `startAfter` over `resumeAfter` — only `startAfter` survives an `invalidate`.
- `db.watch()` avoids the `invalidate` that a collection drop causes on `collection.watch()`,
  and saves connections when several collections are watched.
- Persist the resume token **after** handling, else restarts silently drop events. Subscribe to
  `resumeTokenChanged` so quiet collections keep a fresh token.
- Have a reconciliation fallback for `ChangeStreamHistoryLost` (token aged out of the oplog window):
  an aggregate `$lookup` sweep for parentless children.
- Handlers must be idempotent (at-least-once). Cascade deletes naturally are; side effects like
  notifications are not.
- `onModuleDestroy` must `close()` the stream, and Mongoose's `Model.watch()` silently buffers if
  called before the connection is ready.

## Why Kafka is not simply wrong

Kafka's real advantage is not duplicate suppression — it is **crossing service boundaries**
without handing out DB credentials or coupling to another team's schema. Change Stream requires
direct access to the source DB, which is the Shared Database anti-pattern across services.

If the consumer is a different service, the fix is not removing Kafka but adding a
**Transactional Outbox**: write the delete and an outbox row in one transaction, then relay the
outbox to Kafka (Change Stream or Debezium as the relay). Without it, a crash between the delete
and the publish orphans documents permanently, with no record.

## Naive `onSuccess` chaining

Sequential deletes in the request path are fine and common, with two corrections:

1. **Delete children first, parent last.** If it fails mid-way, the parent still exists, so the
   state stays coherent and a retry is trivial. The reverse leaves untraceable orphans.
2. If it is already sequential in one request, wrap it in `withTransaction` — the cost is a few
   lines and the partial-delete state stops existing. `onSuccess` chaining is a transaction
   without atomicity.

Where transactions are impossible (standalone mongod), at minimum log failures to a
`FailedCascade` collection so failures are visible rather than silent.

## Related

- [[now]] — Edu Vibe runs NestJS + MongoDB + Mongoose, so this applies directly there
- [[philosophy]] — preference for engine-enforced guarantees over app-level convention

## Changelog

- 2026-08-14: created from a design discussion on replacing the legacy Kafka cascade path
