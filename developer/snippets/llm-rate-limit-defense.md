---
title: 중첩 병렬 LLM 파이프라인의 429 방어
area: developer
tags: [LLM, rate-limit, retry, backoff, LangChain, BullMQ]
created: 2026-09-01
updated: 2026-09-01
status: draft
---

# 중첩 병렬 LLM 파이프라인의 429 방어

AI 코스 생성처럼 `course → chapter → lesson → step`으로 fan-out하는 작업에서는 개별 호출의 재시도만으로
429를 해결할 수 없다. 병렬도와 재시도가 곱해져 제한 회복 전에 더 많은 요청을 보낼 수 있기 때문이다.

## 먼저 호출 폭을 계산한다

`edu-ai-course` contents server의 현재 콘텐츠 생성 경로는 다음 두 배치를 중첩한다.

- chapter batch: `maxConcurrency: 3`
- 각 chapter의 lesson batch: `maxConcurrency: 5`

따라서 LLM node가 추가 병렬화를 하지 않아도 한 job에서 이론상 최대 15개 lesson 처리가 겹친다.
BullMQ worker도 여러 job을 동시에 처리한다. 429를 볼 때는 모델 한 번 호출이 아니라
`worker concurrency × chapter concurrency × lesson concurrency × node별 호출 수`를 봐야 한다.

## 현재 적용된 방어층

### 1. SDK 호출 재시도

LLM factory가 OpenAI/Anthropic client의 `maxRetries`를 받을 수 있다. 네트워크 오류나 짧은 제한은
provider SDK가 가장 가까운 위치에서 처리한다.

### 2. 작업 단위 재시도

LangChain `Runnable.withRetry()`를 chapter와 lesson 경계에 둔다.

- chapter: 최대 2회 시도, 429이면 30초 base의 지수 backoff
- lesson: 최대 3회 시도, 429이면 10초 base의 지수 backoff
- backoff: `base × 2^attempt + random(0, base)`, 최대 2분

에러 감지는 SDK마다 다른 표현을 흡수한다.

- 메시지에 `429` 또는 `rate_limit`
- SDK error의 `status === 429`
- 도메인 error의 `statusCode === 429`
- wrapper 안에 숨은 `cause` 재귀 탐색

### 3. 실행 폭 제한과 관측

batch마다 `maxConcurrency`를 명시하고, 재시도 횟수·실패 레벨·course/user 식별자를 APM에 남긴다.
worker 실패 로그에는 BullMQ `attemptsMade`와 내부 `retryAttempts`를 함께 남길 수 있어 어느 층에서
소진됐는지 구분한다.

## 중요한 함정

- **재시도 층을 곱하지 않는다.** SDK, lesson, chapter, queue가 모두 같은 호출을 재시도하면 최악의 호출 수가
  빠르게 커진다. 각 층이 어떤 실패 단위를 복구하는지 먼저 정한다.
- **현재 `withRetry`는 429만 재시도하는 것이 아니다.** 모든 오류를 재시도하되, `onFailedAttempt`가 429일 때만
  기다린다. validation처럼 영구적인 오류까지 재실행하지 않으려면 재시도 조건을 별도로 제한해야 한다.
- **공유 retry counter의 의미를 분명히 한다.** 현재 batch 안의 여러 item이 counter 하나를 공유한다.
  동시에 여러 429가 나면 대기 시간이 빠르게 늘어 글로벌 제한 회복에는 유리하지만, item별 시도 횟수로
  해석할 수 없고 실행 순서에 따라 backoff가 달라진다.
- **jitter 없는 동시 재시도는 thundering herd를 만든다.** 여러 worker가 같은 시각에 깨어나지 않도록 한다.
- **queue 재시도에는 idempotency가 선행한다.** DB 저장·상태 전환·후속 job 생성 뒤 전체 job을 다시 돌리면
  중복 쓰기가 생긴다. 영구 실패나 이미 반영된 main-server 응답은 `UnrecoverableError`로 끊는다.
- **상태는 반드시 실패로 종결한다.** 생성 시작 시 `in_progress`, 최종 예외 시 `failed`로 바꿔 무한 로딩을 막는다.

## 적용 순서

1. fan-out 전체의 동시 호출 상한을 계산한다.
2. provider별 rate-limit header와 error shape를 정규화한다.
3. 가장 작은 재시도 단위와 가장 큰 재시도 단위를 구분한다.
4. exponential backoff + jitter를 적용한다.
5. queue 재시도 전 모든 side effect의 idempotency를 검증한다.
6. 성공률만 보지 말고 provider/model, latency, token, retry layer와 횟수를 기록한다.
7. backoff만으로 부족하면 전역 semaphore 또는 provider별 rate limiter를 둔다.

## 연결 문서

- [[edu-ai-course-architecture]] — 이 패턴이 동작하는 앱/queue 경계
- [[edu-ai-course]] — 프로젝트 성과 서사

## 변경 이력

- 2026-09-01: 중첩 병렬도 계산, SDK/Runnable/worker 방어층, 재시도 증폭과 idempotency 함정을 추출
