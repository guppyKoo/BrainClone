---
title: 실제 프로젝트에서 얻은 Electron 패턴
area: developer
tags: [Electron, electron-vite, electron-builder, patterns]
created: 2026-08-03
updated: 2026-09-01
status: confirmed
---

# Electron 패턴

[[bulk-mail-electron]] 마이그레이션에서 검증된 패턴들.

## 프로젝트 골격

- **electron-vite + electron-builder**가 스캐폴드부터 DMG까지 가장 매끄러운 경로였다
- 구조: `main/services/*` (도메인 서비스) + `main/handlers` (IPC) + `window-manager` (멀티 윈도우 + 브로드캐스트)
- preload는 단일 `window.api`(invoke/on) 브리지만 노출한다 — 렌더러는 IPC 채널을 직접 건드리지 않는다

## 함정 & 해결

- **safeStorage는 동기다**: 웹 습관대로 `await`하지 말 것 (Electron에서는 동기 API)
- **다크 모드 동기화**: `nativeTheme`의 `updated` 이벤트 → 모든 윈도우로 브로드캐스트 → 렌더러 훅이 `.dark` 토글.
  테마 설정은 `userData/*.json`에 영속화
- **서명 없는 macOS 배포**: `identity: null`로 빌드하면 Gatekeeper "손상됨" 오류가 뜬다 →
  ad-hoc 코드사인 + 사용자 안내 추가 (v1.1.x에서 수정)
- **설정 윈도우**: 메뉴 `⌘,` → 설정 윈도우. 변경 사항은 브로드캐스트로 메인 윈도우에 전달

## 빌드

- `npm run build:mac` → arm64 + x64 DMG (약 94~99MB)
- 앱 아이콘: `build/icon.icns`에 배치

## 변경 이력

- 2026-09-01: 문서를 한국어로 전환
- 2026-08-03: 영어로 전환
