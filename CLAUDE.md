# BrainClone Management Rules & OKF Spec

This repository is a personal knowledge base that clones the brain of the user (guppy).
AI (Claude) reads and writes here following this spec. Global Read/Write triggers live in `~/.claude/CLAUDE.md`.

## 0. Language

- **All documents are written in English** (token efficiency). Proper nouns stay as-is (goorm, INOS, 뻐끔이 may be romanized).
- The user speaks Korean in conversation; AI translates when writing to BrainClone.
- Korean source quotes may be kept only when the original wording matters.

## 1. OKF Document Spec

Every `.md` document (except this file and `index.md`) starts with this frontmatter:

```markdown
---
title: Document title
area: profile | interest | developer | career
tags: [keyword1, keyword2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: draft | confirmed
---
```

- **status: draft** — written by AI from inference. Unverified claims are marked `(inferred)`.
- **status: confirmed** — reviewed and approved by the user. No `(inferred)` marks may remain.
- Link documents with wikilinks `[[name]]` or relative markdown links.
- One file = one topic. Split at 800 lines and update `index.md`.
- Journal filenames: `journal/YYYY-MM-DD-topic.md`.
- **No "user input needed" placeholders.** If information is missing, leave the section blank —
  AI fills it in later as facts emerge from conversation. AI may ask the user questions naturally,
  and may queue open questions in `now.md` under "Open questions".

## 2. Routing Table — where does information go?

**Boundary principle: "Is it dev-related?" is the first branch.** Anything development-related goes
under `developer/`, whether curiosity-stage or mastered. `profile/` and `interest/` are non-dev (human) areas.

| Information type                                                | Target file                           |
| --------------------------------------------------------------- | ------------------------------------- |
| What's happening now, current status                            | `now.md`                              |
| Personality as a human, MBTI, communication                     | `profile/personality.md`              |
| Life philosophy, priorities, decision principles                | `profile/values.md`                   |
| Deep reflections, periodic thoughts                             | `profile/journal/YYYY-MM-DD-topic.md` |
| Non-dev learning/interest topics (English study, investing, …)  | `interest/topics/<topic>.md`          |
| Hobbies (fitness, reading, games, music, movies, …)             | `interest/hobbies/<hobby>.md`         |
| Experiences/things/books wanted (non-dev)                       | `interest/wishlist.md`                |
| 서비스·제품 아이디어 (공모전·해커톤 포함)             | `idea/service/<idea>.md`                 |
| Dev interests & learning topics (curiosity stage)               | `developer/interest.md`               |
| Tendencies & work style as a developer (observed)               | `developer/tendency.md`               |
| Tech stack, proficiency (used repeatedly at work)               | `developer/stack.md`                  |
| Code style, conventions, structure preferences (concrete rules) | `developer/coding-style.md`           |
| Development/architecture philosophy (principles)                | `developer/philosophy.md`             |
| Recurring patterns, debugging notes                             | `developer/snippets/<topic>.md`       |
| Resume basics, career summary                                   | `career/resume.md`                    |
| Per-project outcomes, roles, problem-solving                    | `career/portfolio/<project>.md`       |
| Career goals, desired salary/job-change conditions              | `career/goals.md`                     |
| Expected interview questions & answers                          | `career/interview-qa.md`              |

If routing is ambiguous, ask the user instead of creating a new file.

서비스·제품 아이디어는 개발 관련 여부와 관계없이 `idea/service/`에 먼저 기록하고,
구현·출품 후 성과가 생기면 `career/portfolio/` 문서와 연결한다.

### Layers inside developer/ and the promotion rule

- `interest.md` (curiosity/learning) → once used repeatedly at work, **promote** to `stack.md`,
  leaving only "want to dig deeper" items in interest. (Moves are preserved via git and `## Changelog`.)
- `tendency.md` (observed traits) / `philosophy.md` (believed principles) / `coding-style.md` (concrete rules) —
  "tends to do X" → tendency, "believes X is right" → philosophy, "writes code like X" → coding-style.

### Boundary with career/

- `developer/` = facts, for working. `career/` = story, for presenting.
- Career documents **reference** developer documents by link and never restate their content.
  When stack or tendencies change, only developer/ needs editing.

## 3. Read Rules

1. Always start from `index.md` and `now.md`, then descend only into needed areas. Never bulk-read every folder.
2. Before answering, verify the relevant area was read. Never invent content that is not in the documents.
3. When citing a `status: draft` document, note that it is an unconfirmed draft.

## 4. Write Rules

1. **Edit over create**: if a document on the topic exists, edit it instead of creating a new file.
2. **Update `updated`**: bump the frontmatter date on every content change.
3. **Index sync**: when creating a document, add its link to `index.md` in the same operation.
4. **Conflict resolution**: the user's current statement > recorded documents. Update the document and mention the change.
5. **No deletion**: delete documents/content only on explicit user instruction. Git preserves history, so keep the body clean and current.
6. **No secrets**: never record passwords, API keys, or government IDs. Sensitive figures (salary, holdings) only on direct user request.
7. **Journals are immutable**: never edit past-dated journal files. New thoughts get a new dated entry.

## 5. Versioning (Git)

**Git owns history.** Filenames stay fixed (always-latest content); never accumulate snapshot files.

1. **Commit after every change**: `docs: <file> — <summary of change>`. Commit messages in English.
   Past versions: `git log -p <file>`.
2. **Changelog section**: meaningful changes (new achievement, changed goal) get one line in the document's
   `## Changelog`: `- YYYY-MM-DD: summary`. Git diff covers _what_; this section covers _why_.
   Skip typo-level edits.
3. **Archive on overhaul**: only when a document is fully rewritten in a new direction (e.g. resume after a job change),
   move the old file to `_archive/YYYY-MM-DD-<filename>.md` and write fresh.
   `_archive/` is **excluded** from default reads — open it only when the user asks about the past.
4. **Journals are the exception**: append-only dated snapshots by design.

## 6. Maintenance

- Roughly monthly, offer to review `draft` documents with the user and promote them to `confirmed`.
- If a document's `updated` is older than 6 months, ask the user whether it needs refreshing.
