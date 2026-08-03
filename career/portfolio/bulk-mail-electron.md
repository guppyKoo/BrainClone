---
title: Bulk Mail — Electron 데스크톱 앱
area: career
tags: [포트폴리오, Electron, React, 사이드프로젝트]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# Bulk Mail (Post Office) — Electron 데스크톱 앱

macOS용 대량 메일 발송 데스크톱 앱. 개인 프로젝트, 단독 개발.
저장소: github.com/guppyKoo (원본: Bulk-email → 이식: bulk-mail-electron)

## 문제와 해결 스토리

- Glaze(로우코드 플랫폼)로 먼저 만들었으나 **DMG 빌드 불가** → 2026-07 순정 Electron으로 전면 재구축 결정
- electron-vite + electron-builder 스켈레톤부터 3단계(백엔드 → UI 컴포넌트 → 렌더러)로 이식
- **핵심 목표 달성**: arm64/x64 DMG 빌드 + 패키징 앱 실행 검증
- 이후 v1.1.0: Linear/Notion 스타일 UI 리디자인, 수신자 태그 기능, ad-hoc 코드사인으로 Gatekeeper 이슈 해결

## 기술 구성

- **스택**: Electron 34, React 19, Tailwind v4, TanStack Router/Query, Radix, nodemailer, OpenAI Images API
- **아키텍처**: main 프로세스 서비스 3분할(settings/mail/image) + IPC handlers + 멀티 윈도우 매니저(broadcast)
- **UI**: Radix + CVA 기반 자체 컴포넌트 ~30개 재구현, 디자인 토큰(`rgb(from ...)` 파생) + 다크모드
- **기능**: 리치 텍스트 에디터, 수신자 관리(태그/사이드바), AI 이미지 생성, 발송 결과 리포트, 설정 창(⌘,)

## 어필 포인트

- 플랫폼 한계에 막혔을 때 스택을 통째로 갈아타서 **혼자서 단기간에 전체 이식 + 배포** (2026-07-13 시작)
- 각 단계마다 typecheck/build/dev 검증을 통과시키는 점진적 이식 전략

## TODO

- 스크린샷/데모 링크, 다운로드 수 등 성과 지표
