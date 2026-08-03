---
title: Expected interview Q&A
area: career
tags: [interview, QA]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# Expected interview Q&A

> Question + answer skeletons drafted from project history. Polish in the user's own words before confirming.

## Technical deep-dives

**Q. Hardest bug you've solved?**
Draft: atom-node deletions not syncing in collaborative editing. Disproved the first hypothesis ("empty-document seeding")
with a test, then found the missing null guard in y-prosemirror's selection-restore code and fixed it via patch-package.
→ Tell the full arc: hypothesis → disproof → root cause → patch. ([[y-prosemirror-nodeselection-crash]])

**Q. How does CRDT/collaborative editing work?**
Draft: explain via the Yjs document model, the hocuspocus server's role, and the editor binding (ySyncPlugin).
Differentiate with real binding-layer bug-hunting experience.

**Q. How do you choose technology?**
Draft: start on proven foundations (Radix, TanStack, Prisma) but own the core layer.
Swap boldly when the platform blocks the goal — Glaze → Electron for DMG shipping. ([[bulk-mail-electron]])

## Projects

**Q. Tell me about your side projects.**
Draft: [[bulk-mail-electron]] (problem → rebuild → ship story) + [[inos]] (own hobby's pain point solved with full-stack + AI).

**Q. How do you use AI tools in development?**
Draft: not just code generation — hook-based quality gates, multi-agent review, and a personal knowledge base (BrainClone)
wired into the workflow. Emphasize the workflow-designer perspective.

## Personality / culture fit

**Q. Introduce yourself / strengths & weaknesses**

## Changelog

- 2026-08-03: migrated to English
