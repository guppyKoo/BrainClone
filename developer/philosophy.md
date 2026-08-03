---
title: 개발 철학
area: developer
tags: [철학, 아키텍처, AI협업]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# 개발 철학

> 작업 방식에서 역추론한 초안. 본인의 언어로 다듬기 권장.

## 관찰된 철학 (추정)

1. **끝까지 배포한다**: "빌드가 된다"가 아니라 "패키징된 앱이 실행된다"까지가 완료.
   Bulk Mail은 typecheck → build → dev → DMG → 패키징 앱 실행까지 전부 검증 후 완료 선언.
2. **근본 원인까지 판다**: 증상 우회 대신 라이브러리 소스(y-prosemirror)까지 내려가 null 가드를 패치.
   오답 가설도 테스트로 반증하고 기록해 둔다.
3. **규칙은 문서가 아니라 시스템으로**: 컨벤션을 머리로 기억하지 않고 훅(GateGuard)·룰셋(ECC)·
   프레임워크(OKF)로 강제한다.
4. **검증된 것 위에서, 그러나 소유한다**: Radix/TanStack 같은 검증된 기반을 쓰되,
   UI 컴포넌트 30개를 직접 재구현하는 등 핵심 레이어는 직접 소유·이해한다.
5. **AI는 위임 대상, 사람은 구조 설계자**: 폴더 구조와 규칙을 먼저 정의하고 AI에게 실행을 맡기는 패턴.

## 아키텍처 성향 (추정)

- 모노레포 + 공유 패키지(prisma/types/utils)로 타입 일관성 확보
- 서비스 분리 (INOS: API 서버 / AI 서버 분리, Bulk Mail: settings/mail/image 서비스 분리)
- IPC·API 경계에 명시적 브릿지 (`window.api` preload 브릿지)

## TODO

- 본인이 동의하는/아닌 부분 표시, 자주 인용하는 원칙(Clean Code? YAGNI?) 직접 기술
