---
title: Tendencies as a developer
area: developer
tags: [tendency, work-style, strengths, weaknesses]
created: 2026-08-03
updated: 2026-08-24
status: draft
---

# Tendencies as a developer

**Observed** developer traits from work history. (Believed principles → [[philosophy]], concrete rules → [[coding-style]],
human personality → `profile/personality.md`.)

## Work style (inferred)

- **Serious about tools & automation**: hundreds of skills/agents/hooks (ECC, GateGuard) configured in Claude Code;
  spends real energy tuning the dev workflow itself — early adopter, meta-productivity oriented
- **Rebuilds from the root when blocked**: Glaze couldn't produce a DMG → rebuilt the whole app on vanilla Electron and shipped
- **Verifies to the end**: only "done" after typecheck/build/dev/packaged-app-run all pass
- **Structure first, delegate execution**: defines folder structure and rules first, then hands execution to AI

## Strengths (inferred)

- **Finisher**: absorbs a new stack quickly and reaches shipping (DMG)
- **Root-cause tracking** — dug [[y-prosemirror-nodeselection-crash]] down to a library patch; disproves wrong hypotheses with tests and records them
- **Systems thinking** — designs frameworks like OKF and the ECC ruleset himself

## Weaknesses

## Collaboration style

- Korean commit messages + Conventional Commits, feature branch → PR flow
- Do not invoke `impeccable`; use direct, scoped implementation unless the user explicitly reverses this preference.
- **Separates diagnosis from repair**: asks for the analysis of a failure first and explicitly holds
  off the fix ("코드 수정은 아직 하지 마"). Wants to see the reasoning before code moves.
- **Defines metrics by computation, not by term**: corrected an ambiguous spec word ("class capacity")
  by restating it as the exact rule — split `_id` on `-`, match the lesson's entry code, count
  `role: 'student'`. Write aggregation decisions at query level for him, not as business vocabulary.
- **Wants deliverables in full, not summarized**: when a document is meant to be handed to someone
  else, he asks for the whole text rather than a digest of it.

## Changelog

- 2026-08-24: added three collaboration traits — diagnosis before repair, metrics defined as
  computations, deliverables in full text
- 2026-08-21: strengthened the preference to not invoke `impeccable`
- 2026-08-20: recorded preference for direct execution on small UI changes without `impeccable`
- 2026-08-03: split from profile/personality.md
- 2026-08-03: migrated to English
