---
title: Edu Vibe LLM eval의 Gate와 생성·평가 AX 경계를 확정한다
area: developer
tags: [edu-vibe, LLM, eval, gate, TokenHub, Langfuse]
created: 2026-09-15
updated: 2026-09-15
status: confirmed
---

# 결정: Gate 뒤에서 R·D·B와 M을 분리해 평가한다

이 결정은 [[2026-09-14-edu-vibe-llm-eval-observation-model]]의 관측 모델 가운데 현재 구현 범위와
축의 소유권을 구체화한다. 현재 결과 범위는 Q1~Q3이며, `L`, `utteranceAmbiguity`,
`contextualAmbiguity`는 평가 AX가 재판정하지 않고 생성 AX가 서비스 로그에 기록한 값을 사용한다.

## 처리 흐름

1. 로그의 사용자 발화와 AI 답변을 1:1로 연결한다. 사용자 식별자는 필요하지 않지만 같은 대화의 이전
   발화와 작업물은 보존한다.
2. Gate에서 사용자 중단으로 잘렸거나 완성되지 않은 답변, 정책으로 끊긴 답변, HTML 코드가 없는
   답변을 평가 대상에서 제외한다.
3. 완결된 HTML이 실행되지 않으면 후속 브라우저 검사는 하지 않고 Q1~Q3를 0점으로 기록한다.
4. Step1에서 현재 발화의 `R·S`, 전후 증거의 `B·D`를 추출한다.
5. Step2에서 R과 D의 의미 관계 `M`을 추출한다.
6. Step3에서 고정식으로 Q1~Q3를 계산하고 S별·L별 Q1~Q3 그래프 데이터를 만든다. 두 Ambiguity를
   각각 필터로 사용한다.

EDU-VIBE 생성 결과물은 현재 HTML만 지원하므로 다른 artifact 형식은 고려하지 않는다.

## 호출 추적 경계

평가 호출이 서비스 생성 호출과 같은 이름으로 Langfuse에 중복 적재되지 않도록 TokenHub client tag를
단계별로 분리한다.

- R·S와 검사 계획: `edu-vibe-llm-quality-eval-requirements`
- B·D: `edu-vibe-llm-quality-eval-semantics`
- M: `edu-vibe-llm-quality-eval-matching`

평가 로그 importer는 위 접두어의 client tag가 붙은 호출을 서비스 원본 로그로 다시 가져오지 않는다.

## 실패와 0점의 구분

- `SKIPPED`: 중단, 정책 차단, 코드 없음. 점수 집계와 그래프에서 제외한다.
- `ZERO_SCORED`: 평가 대상 응답이지만 HTML이 실행되지 않음. Q1~Q3를 모두 0으로 집계한다.
- `FAILED`: 평가기 자체가 증거 또는 모델 결과를 얻지 못함. 0점이 아니라 `null`이다.

과거 Langfuse export에는 명시적 중단 여부와 생성 AX의 L·Ambiguity가 없다. 닫히지 않은 HTML은
`INTERRUPTED_INFERRED`로 보수적으로 분류하고, 없는 L·Ambiguity는 추정하지 않고 `null`로 둔다.

## 기각한 대안

- **평가 AX가 L과 Ambiguity를 다시 판단**: 생성 당시 판단과 사후 판단이 섞여 필터 축의 의미가
  달라지므로 기각했다.
- **B·D·M을 한 번의 의미 분석 호출에서 함께 생성**: Step1의 관측 단위와 Step2의 관계 판단을
  재실행·추적하기 어려워 분리했다.
- **코드 없는 답변에 이전 HTML을 이어 붙여 정상 평가**: 실제 AI 답변에 코드가 없었다는 Gate 사실을
  숨기므로 기각했다. 이전 HTML은 대화 이력 복원용으로만 보존한다.
- **평가 호출에 서비스와 같은 client tag 사용**: Langfuse에서 서비스 생성과 평가 호출이 섞이고
  재수집될 수 있어 기각했다.

## 대체 관계

[[2026-09-14-edu-vibe-llm-eval-observation-model]]의 Q4와 평가 AX가 L을 추출하는 부분은 현재 구현에
적용하지 않는다. 종합 점수를 만들지 않고 각 Q를 독립 관측한다는 원칙은 유지한다.
