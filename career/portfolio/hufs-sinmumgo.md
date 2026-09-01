---
title: HUFS sinmumgo — 한국외대판 청원24
area: career
tags: [portfolio, frontend, React, petition, security]
created: 2026-09-01
updated: 2026-09-01
status: confirmed
---

# HUFS sinmumgo — 한국외대판 청원24

2025년 7월부터 진행한 교내 청원 웹 서비스다. 학생과 학교 사이의 소통을 늘리는 것을 목표로 사용자 권한, 청원 목록 탐색, 작성 경험과 보안 통신을 구현했다.

## 기술 스택

- React
- TypeScript
- TailwindCSS
- Axios
- Recoil
- TanStack Query
- Quill

## 주요 역할과 성과

- 비회원·학생·관리자의 인증 상태와 권한에 따라 접근 가능한 기능과 화면을 동적으로 분기했다.
- 쿼리 스트링 기반 페이지네이션으로 대규모 청원 목록을 관리하고, TanStack Query의 캐싱과 로딩 상태를 활용해 페이지 전환을 최적화했다.
- Axios Interceptor로 인증 상태 연장 흐름을 구현했다.
- 검색과 미리보기에 debounce를 적용해 불필요한 API 호출을 줄였다.
- 역할과 기능별 컴포넌트 모듈화, 상태 관리와 API 호출의 분리를 통해 재사용성과 유지보수성을 높였다.
- 모바일과 데스크톱에 대응하는 반응형 디자인을 구현했다.
- access token과 refresh token을 적용했다.
- 교내 클러스터에서 SSL 인증서를 발급해 HTTPS 통신을 적용했다.
- 브라우저별 특성을 고려해 사용자 경험을 최적화했다.

## 변경 이력

- 2026-09-01: 이력 프로젝트 자료를 바탕으로 최초 작성
