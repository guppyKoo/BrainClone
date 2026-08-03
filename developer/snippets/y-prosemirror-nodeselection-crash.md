---
title: y-prosemirror NodeSelection crash debugging note
area: developer
tags: [y-prosemirror, Yjs, hocuspocus, collab-editing, debugging]
created: 2026-08-03
updated: 2026-08-03
status: confirmed
---

# y-prosemirror NodeSelection crash

Root cause and fix for **atom-node deletions not syncing to other clients** in collaborative editing (hocuspocus + y-prosemirror).

## Symptom

Insert an atom node (e.g. video embed) into an empty document, delete it → the deletion never reaches other clients.

## Root cause

y-prosemirror 1.3.7 `restoreRelativeSelection` lacks a null guard in its `type === 'node'` branch.
When one client holds a NodeSelection on an atom node and another client deletes that node,
applying the remote change yields a null anchor → `NodeSelection.create(doc, null)` crashes → yjs↔ProseMirror sync halts.
(The TextSelection branch right below has the null check; only the node branch is missing it.)

## The wrong hypothesis

Assumed "empty documents seeded as an empty Y.XmlFragment" was the cause — disproved by test:
insert/delete syncs fine even with an empty fragment. The hocuspocus server doesn't use ySyncPlugin, so it's unrelated.

## Fix

patch-package: `patches/y-prosemirror+1.3.7.patch` — add
`if (anchor !== null && tr.doc.nodeAt(anchor) !== null)` to the node branch.
**Caution**: patch both `dist/y-prosemirror.cjs` and `src/plugins/sync-plugin.js`
(webpack resolves the exports `import` condition to the src bundle).

## Lesson

For atom-node + collab-editing + delete/selection sync bugs, **suspect the selection-restore path first**.
Reproducible with a 2-client relay test. Related: [[interest|developer/interest]]

## Changelog

- 2026-08-03: migrated to English
