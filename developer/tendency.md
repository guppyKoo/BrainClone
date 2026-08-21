---
title: Tendencies as a developer
area: developer
tags: [tendency, work-style, strengths, weaknesses]
created: 2026-08-03
updated: 2026-08-21
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

## Changelog

- 2026-08-21: strengthened the preference to not invoke `impeccable`
- 2026-08-20: recorded preference for direct execution on small UI changes without `impeccable`
- 2026-08-03: split from profile/personality.md
- 2026-08-03: migrated to English
