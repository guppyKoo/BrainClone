---
title: AI 에이전트 & Claude Code 생태계
area: interest
tags: [AI, 에이전트, Claude Code, MCP, 자동화]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# AI 에이전트 & Claude Code 생태계

가장 활발하게 파고 있는 주제. 단순 사용을 넘어 **개발 워크플로우 자체를 에이전트로 재설계**하는 데 관심.

## 현재 하고 있는 것

- Claude Code에 ECC(Everything Claude Code) 룰셋·스킬·에이전트 대량 설치 및 커스터마이징
- GateGuard 같은 훅 기반 품질 게이트 운용 (Bash/Write 전 사실 강제 제시)
- MCP 서버 다수 연결: Figma, Notion, MongoDB, Chrome DevTools, GitHub 등
- 개인 지식 베이스(BrainClone/OKF)를 AI가 읽고 쓰는 구조로 설계

## 관심 세부 주제

- 멀티 에이전트 오케스트레이션 (병렬 리뷰, 워크플로우 파이프라인)
- 훅(PreToolUse/PostToolUse)으로 AI 행동을 강제하는 패턴
- AI 메모리/컨텍스트 관리 — 세션 간 지식 유지
- OpenAI Images API 등 생성 API의 제품 내 활용 ([[bulk-mail-electron]]에 이미지 생성 기능 내장)

## 다음에 볼 것

- **TODO**: Claude Agent SDK로 커스텀 에이전트 직접 구축
- **TODO**: 에이전트 평가(eval) 체계
