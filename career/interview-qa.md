---
title: 면접 예상 질문 & 답변
area: career
tags: [면접, QA]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# 면접 예상 질문 & 답변

> 프로젝트 이력 기반으로 "받을 법한 질문 + 답변 골격"을 초안으로 작성. 본인 언어로 다듬어 확정할 것.

## 기술 심화

**Q. 가장 어려웠던 버그를 해결한 경험은?**
초안: 동시편집에서 atom 노드 삭제가 동기화되지 않는 버그. "빈 문서 시딩 문제"라는 첫 가설을
테스트로 반증한 뒤, y-prosemirror의 selection 복원 코드에서 누락된 null 가드를 찾아
patch-package로 해결. → 가설-반증-근본원인-패치의 전 과정 서술. ([[y-prosemirror-nodeselection-crash]])

**Q. CRDT/동시편집은 어떻게 동작하나요?**
초안: Yjs 문서 모델, hocuspocus 서버 역할, 에디터 바인딩(ySyncPlugin) 구조로 설명.
실무에서 바인딩 레이어 버그를 잡아본 경험으로 차별화. — **TODO: 이론 정리 보강**

**Q. 기술 선택은 어떤 기준으로 하나요?**
초안: 검증된 기반(Radix, TanStack, Prisma) 위에서 시작하되 핵심 레이어는 직접 소유.
Glaze → Electron 전환처럼 플랫폼이 목표(DMG 배포)를 막으면 과감히 교체. ([[bulk-mail-electron]])

## 프로젝트

**Q. 사이드 프로젝트 소개해주세요.**
초안: [[bulk-mail-electron]] (문제→재구축→배포 스토리) + [[inos]] (취미의 페인포인트를 풀스택+AI로 해결).

**Q. AI 도구를 개발에 어떻게 활용하나요?**
초안: 단순 코드 생성이 아니라 훅 기반 품질 게이트, 멀티 에이전트 리뷰, 개인 지식 베이스(BrainClone)
연동까지 — 워크플로우 설계자로서의 관점 강조.

## 인성/컬처핏

**Q. 자기소개 / 강점과 약점**
- **TODO — [[personality]] 확정 후 작성**
