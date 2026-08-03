---
title: 실시간 동시편집 (CRDT / Yjs)
area: interest
tags: [CRDT, Yjs, hocuspocus, ProseMirror, 동시편집]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# 실시간 동시편집 (CRDT / Yjs)

업무(구름 edu-core 협업 강의 편집기)에서 깊게 다루는 주제이자 기술적 관심사.

## 다뤄본 스택

- **Yjs + hocuspocus**: 협업 편집 백엔드, Y.XmlFragment 시딩 구조
- **y-prosemirror + TipTap/ProseMirror**: 에디터 바인딩, ySyncPlugin
- 2-클라이언트 릴레이로 동기화 버그 재현·검증하는 테스트 방법론

## 배운 것 (하이라이트)

- CRDT 동기화 버그는 데이터 시딩보다 **selection 복원 경로** 같은 바인딩 레이어에서 터지는 경우가 많다
  — 상세: [[y-prosemirror-nodeselection-crash]]
- 라이브러리 버그는 patch-package로 프로젝트 레벨에서 관리 가능 (dist와 src 번들 둘 다 패치해야 하는 함정 포함)

## 더 파보고 싶은 것

- **TODO**: Yjs 내부 구조(아이템 병합, GC), 다른 CRDT(Automerge, Loro) 비교
