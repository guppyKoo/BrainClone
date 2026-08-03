---
title: Electron 실전 패턴
area: developer
tags: [Electron, electron-vite, electron-builder, 패턴]
created: 2026-08-03
updated: 2026-08-03
status: draft
---

# Electron 실전 패턴

[[bulk-mail-electron]] 이식 과정에서 검증된 패턴 모음.

## 프로젝트 골격

- **electron-vite + electron-builder** 조합으로 스캐폴딩 → DMG 빌드까지 가장 매끄러웠음
- 구조: `main/services/*` (도메인 서비스) + `main/handlers` (IPC) + `window-manager` (멀티 윈도우 + broadcast)
- preload는 `window.api` (invoke/on) 브릿지 하나로 통일 — 렌더러가 IPC 채널을 직접 알 필요 없음

## 함정과 해법

- **safeStorage는 동기 API**: 웹 습관으로 `await` 붙이지 말 것 (Electron에서 동기)
- **다크모드 동기화**: `nativeTheme`의 `updated` 이벤트 → 전체 창 broadcast → 렌더러 훅이
  `.dark` 클래스 토글. 테마 설정은 `userData/*.json`에 영속화
- **미서명 macOS 배포**: `identity: null`로 빌드하면 Gatekeeper "손상됨" 오류 발생 →
  ad-hoc 코드사인 추가 + 사용자 안내 필요 (v1.1.x에서 해결)
- **멀티 윈도우 설정 창**: 메뉴 `⌘,` → settings 창, 변경사항은 broadcast로 main 창에 반영

## 빌드

- `npm run build:mac` → arm64 + x64 DMG 동시 생성 (~94–99MB)
- 앱 아이콘: `build/icon.icns` 배치로 적용
