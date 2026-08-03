---
title: y-prosemirror NodeSelection 크래시 디버깅 노트
area: developer
tags: [y-prosemirror, Yjs, hocuspocus, 동시편집, 디버깅]
created: 2026-08-03
updated: 2026-08-03
status: confirmed
---

# y-prosemirror NodeSelection 크래시

동시편집(hocuspocus + y-prosemirror)에서 **atom 노드 삭제가 다른 클라이언트에 동기화되지 않는 버그**의 근본 원인과 해결.

## 증상

빈 문서에 atom 노드(영상 임베드 등) 삽입 후 삭제 → 다른 클라이언트에 삭제가 반영되지 않음.

## 근본 원인

y-prosemirror 1.3.7 `restoreRelativeSelection`의 `type === 'node'` 분기에 null 가드가 없음.
한 클라이언트가 atom 노드에 NodeSelection을 건 상태에서 다른 클라이언트가 그 노드를 삭제하면,
원격 변경 적용 시 anchor가 null → `NodeSelection.create(doc, null)` 크래시 → yjs↔ProseMirror 동기화 중단.
(바로 아래 TextSelection 분기에는 null 체크가 있는데 node 분기만 누락)

## 오답이었던 가설

"빈 문서가 빈 Y.XmlFragment로 시딩돼서"라고 생각했으나, 빈 fragment여도 insert/delete는
정상 동기화됨을 테스트로 반증. hocuspocus 서버는 ySyncPlugin을 쓰지 않으므로 무관.

## 해결

patch-package로 `patches/y-prosemirror+1.3.7.patch` 생성 — node 분기에
`if (anchor !== null && tr.doc.nodeAt(anchor) !== null)` 가드 추가.
**주의**: `dist/y-prosemirror.cjs`와 `src/plugins/sync-plugin.js` 둘 다 패치해야 함
(webpack이 exports의 import 조건으로 src 번들을 사용).

## 교훈

atom 노드 + 동시편집 + 삭제/선택 동기화 버그가 나오면 **selection 복원 경로부터 의심**.
2-클라이언트 릴레이 테스트로 재현·검증 가능. 관련: [[realtime-collaboration]]
