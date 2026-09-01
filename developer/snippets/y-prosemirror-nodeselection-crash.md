---
title: y-prosemirror NodeSelection 크래시 디버깅 노트
area: developer
tags: [y-prosemirror, Yjs, hocuspocus, collab-editing, debugging]
created: 2026-08-03
updated: 2026-09-01
status: confirmed
---

# y-prosemirror NodeSelection 크래시

협업 편집(hocuspocus + y-prosemirror)에서 **atom 노드 삭제가 다른 클라이언트로 동기화되지 않던** 문제의 근본 원인과 해결.

## 증상

빈 문서에 atom 노드(예: 비디오 임베드)를 삽입하고 삭제하면 → 그 삭제가 다른 클라이언트에 영영 도달하지 않는다.

## 근본 원인

y-prosemirror 1.3.7의 `restoreRelativeSelection`은 `type === 'node'` 분기에 null 가드가 없다.
한 클라이언트가 atom 노드에 NodeSelection을 잡고 있는 상태에서 다른 클라이언트가 그 노드를 삭제하면,
원격 변경을 적용할 때 anchor가 null이 되고 → `NodeSelection.create(doc, null)`이 크래시하고 → yjs↔ProseMirror 동기화가 멈춘다.
(바로 아래 TextSelection 분기에는 null 체크가 있다. node 분기에만 빠져 있다.)

## 틀렸던 가설

"빈 문서를 빈 Y.XmlFragment로 시딩하는 것"이 원인이라고 가정했다 — 테스트로 반증했다:
빈 fragment여도 삽입/삭제는 정상 동기화된다. hocuspocus 서버는 ySyncPlugin을 쓰지 않으므로 무관하다.

## 해결

patch-package: `patches/y-prosemirror+1.3.7.patch` — node 분기에
`if (anchor !== null && tr.doc.nodeAt(anchor) !== null)`를 추가한다.
**주의**: `dist/y-prosemirror.cjs`와 `src/plugins/sync-plugin.js`를 **둘 다** 패치할 것
(webpack이 exports의 `import` 조건을 src 번들로 해석한다).

## 교훈

atom 노드 + 협업 편집 + 삭제/선택 동기화 버그라면 **selection 복원 경로를 먼저 의심할 것**.
2-클라이언트 릴레이 테스트로 재현 가능하다. 관련: [[interest|developer/interest]]

## 변경 이력

- 2026-09-01: 문서를 한국어로 전환
- 2026-08-03: 영어로 전환
