---
title: 개발 관련 관심사
area: developer
tags: [관심사, AI, 에이전트, CRDT, 학습]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# 개발 관련 관심사

호기심·학습 단계의 개발 주제 모음. **실무에서 반복 사용하는 수준이 되면 [[stack]]으로 승격**하고
여기에는 "더 파보고 싶은 것"만 남긴다. (비개발 관심사는 `interest/`에)

## AI 에이전트 & Claude Code 생태계

가장 활발하게 파는 주제. 단순 사용을 넘어 **개발 워크플로우 자체를 에이전트로 재설계**하는 데 관심.

- 현재: ECC 룰셋·스킬·에이전트 대량 운용, GateGuard 등 훅 기반 품질 게이트, MCP 다수 연결(Figma/Notion/MongoDB/GitHub 등), BrainClone을 AI가 읽고 쓰는 구조로 설계
- 관심 세부: 멀티 에이전트 오케스트레이션, PreToolUse/PostToolUse 훅으로 행동 강제, 세션 간 메모리/컨텍스트 관리
- 다음에 볼 것: **TODO** — Claude Agent SDK 커스텀 에이전트, 에이전트 평가(eval) 체계

## 실시간 동시편집 (CRDT / Yjs)

실무 숙련 영역이라 스택 자체는 [[stack]]에 정리됨. 남은 호기심:

- **TODO**: Yjs 내부 구조(아이템 병합, GC), 다른 CRDT(Automerge, Loro) 비교

## 만들어보고 싶은 것

- (추정) Claude Agent SDK 기반 개인 에이전트 — BrainClone을 읽는 "디지털 분신"
- (추정) Bulk Mail 정식 배포 — 코드사인/공증까지 마친 릴리즈
- (추정) INOS 실서비스 오픈

## 변경 이력

- 2026-08-03: interest/topics/의 ai-agent-tooling·realtime-collaboration + wishlist 개발 항목을 병합해 생성
