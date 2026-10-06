---
title: 이력 기본 정보
area: career
tags: [resume, career]
created: 2026-08-03
updated: 2026-10-06
status: confirmed
---

# 이력 기본 정보

## 인적 사항

- 이름: (핸들: guppy / guppy.koo)
- 이메일: yunchan0339@gmail.com
- GitHub: https://github.com/guppyKoo
- 2001년생 / 수지 거주

## 현재

- 소속: **goorm** — 판교 오피스 (goorm 교육 제품 라인: Edu Vibe, edu-ai-course, edu-core)
- 역할: JavaScript/TypeScript **풀스택 개발자** — NestJS 백엔드, 실시간 협업 편집(Yjs/Hocuspocus), 사내 k8s 플랫폼 기반 배포
- 고용: **2025-09 인턴 입사 → 2026-03 정규직 전환**

## 핵심 역량

- **서비스 아키텍처 설계 → 정식 서비스 배포**: Edu Vibe(K-12 바이브코딩 플랫폼) 아키텍처를 설계하고
  프론트·서버 두 레포를 구현. MVP 구조 그대로 학교 시연과 배포까지 완료. 대안(Fastify·Prisma)을
  실험·검토 후 기각 근거까지 문서화 → [[2026-09-09-edu-vibe-architecture]]
- **LLM 출력 품질 계측**: LLM은 요구 추출만, 판정은 결정론적 코드가 맡는 eval 파이프라인 설계.
  측정 재현성(실행 분산 0/47)과 추출 분산을 분리해 수치로 입증 → [[2026-09-16-edu-vibe-eval-instrument-contract]]
- **서버 spec-first TDD**: spec 327 · e2e 94, 실제 마이그레이션을 테스트 DB에도 적용하는 하네스
- **실시간 협업 편집 심층 디버깅**: Yjs/hocuspocus/ProseMirror — 오픈소스 라이브러리 패치, epoch 기반 문서 버저닝
- **LLM 서비스 연동**: SSE 스트리밍, 멀티 provider 구성, 중첩 병렬 파이프라인의 429 방어
- **TypeScript/React 제품 개발**: 실무 에디터 시스템 + 출시한 개인 프로젝트 (Electron 앱 DMG 배포까지 단독 완성)
- **AI 에이전트 워크플로**: Claude Code, MCP, hooks 운영 — 사내 LiteLLM 프록시 연동 포함
- **배포**: 사내 EKS 기반 k8s 플랫폼(Jenkins, ArgoCD) 위에서 서비스 배포·운영

## 이전 경험 & 리더십

- **UMC**: 멤버에서 스터디 리더, 다시 지부 리더로 성장했다. 멤버의 이야기를 듣고 그들의 요구를 대변하고, 필요할 때 가르치고, 조직이 합의에 이르도록 돕는 소통의 다리 역할로 신뢰를 얻었다.
- **CHALLKATHON**: 해커톤 팀에서 소통과 합의 형성을 주도했고 팀이 대상을 받았다. 개인의 기술력보다 적극적인 소통이 더 중요할 수 있다는 것을 확인한 경험. [[challkathon]] 참고.
- **WeKick**: 대학생 풋살 매칭 플랫폼의 기획과 프론트엔드 개발을 주도했다. 실제 사업자 등록과 3개 기업 스폰서십 유치까지 진행했다. 서비스는 결국 리텐션에서 고전했지만, 그 과정에서 제품·운영·사용자 피드백에 대한 실전 경험을 쌓았다. [[wekick]] 참고.
- **HUFS 청원 플랫폼**: 총학생회의 지원을 받아 교내 학생 청원 서비스 개발을 주도적으로 이끌었다. 권한 기반 화면, 쿼리 스트링 페이지네이션, debounce 검색, access/refresh token 인증과 HTTPS를 구현했다. [[hufs-sinmumgo]] 참고.
- **Yellowbook**: 소상공인 팀을 위한 재고·발주 일정 관리 B2B SaaS에서 팀 단위 데이터 분리와 역할별 권한·라우팅을 구현했다. [[yellowbook]] 참고.
- **Re:Bid**: 업사이클링 중고 거래 서비스에서 JWT 인증, polling 기반 실시간 입찰, 입찰 유효성 검증과 예외 처리를 구현했다. [[rebid]] 참고.
- **ChePL**: 학교 웹메일 인증 기반 체육대회 운영 플랫폼에서 Firestore 실시간 경기 반영, 운영진 상태 관리와 재사용 가능한 대진표 알고리즘을 구현했다. [[chepl]] 참고.
- **초기 풀스택 인턴/프로젝트 경험**: 에듀테크 MVP에서 REST API와 페이지를 개발하고, 디자인과 협업하고, 테스트를 작성하고, CI/CD에 GitHub Actions 타입 체크 단계를 추가했다.

## 경력 사항

### goorm — 풀스택 개발자 (2025-09 인턴 → 2026-03 정규직 ~ 현재)

- **Edu Vibe** (2026-08~): K-12 바이브코딩 실습·학습관리 플랫폼. 아키텍처 설계 + 프론트·서버 레포 구현,
  2026-09 학교 시연·배포, LLM 품질 eval 설계 → [[edu-vibe]]
- **edu-ai-course** (2025-11 ~ 2026-04): AI 맞춤형 교육 플랫폼. oRPC 계약 공유 모노레포, LLM 파이프라인
  → [[edu-ai-course]]
- **edu-core**: 실시간 협업 편집(goorm-hocuspocus, epoch 기반 문서 버저닝), 교육과정 편집 V2 저장 오작동
  추적과 공개 기간 검증 정책 → [[2026-09-02-lesson-period-policy]]

## 학력 / 자격증

- **한국외국어대학교** — 컴퓨터공학 전공, **2026-02 졸업**

## 변경 이력

- 2026-10-06: 리뷰 — 소속에서 gem·mist-blocks 제거, 인턴→정규직 전환 명시, 핵심 역량 재작성(Edu Vibe 설계·LLM eval
  추가, pgvector 제거), 경력 사항 채움, 학력을 2026-02 졸업으로 정정. 사용자 확인 후 confirmed 승격
- 2026-09-01: CHALLKATHON 대상 수상 경험을 독립 포트폴리오 문서로 연결
- 2026-09-01: 한국외국어대학교 컴퓨터공학 전공 재학 정보 추가
- 2026-09-01: Yellowbook, Re:Bid, ChePL, HUFS sinmumgo 프로젝트 역할과 포트폴리오 링크 추가
- 2026-09-01: 문서를 한국어로 전환
- 2026-08-18: UMC 리더십, 협업, WeKick, HUFS 청원, 초기 풀스택 경험 추가
- 2026-08-03: 소속(goorm, 판교), 역할, 인프라 경험을 추론에서 확정으로 변경
- 2026-08-03: 영어로 전환
