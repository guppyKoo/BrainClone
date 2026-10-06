---
title: Edu Vibe LLM eval에서 M/Judge 단계를 없애고 LLM을 닫힌 R 추출기로 제한한다
area: developer
tags: [edu-vibe, LLM, eval, extractor, judge, 결정론]
created: 2026-09-16
updated: 2026-10-06
status: confirmed
---

# 결정: 평가 LLM은 R만 뽑는다. 판정은 전부 코드가 한다

> 2026-09-16 저널 백로그 2건을 2026-10-06 월간 리뷰에서 승격한 문서다.
> Extractor 계약의 코드 강제와 계측기 프레이밍은 같은 날의
> [[2026-09-16-edu-vibe-eval-instrument-contract]]가 다룬다. 이 문서는 그 전제가 된
> **단계 구조의 변경**만 기록한다.

## 결정

1. **M(R과 D의 의미 관계) 추출 단계와 Judge 역할을 제거한다.**
   평가 LLM은 발화·맥락을 닫힌 실행 스키마 R로 분류하는 Extractor 역할만 맡는다.
2. **S·검사 컴파일·B·D·Q1~Q3는 코드가 담당한다.**
3. **S를 LLM이 R과 별도로 출력하지 않는다.** 독립 판정 가능한 원자적 R만 생성하고
   코드에서 `S = requirements.length`로 계산한다. 각 R의 constraint는 **최대 하나**로 제한한다.
4. **DSL로 옮기지 못한 요구는 부분 채점하지 않는다.** `UNMAPPED_REQUIREMENT`로 분리해
   Gate에서 별도 추적한다.

## 기각한 대안

- **M/Judge LLM 단계 유지**: 점수에 LLM 판정이 직접 들어온다. 판정 단계가 남아 있는 한
  점수 변동이 제품 때문인지 평가기 때문인지 분리할 수 없다.
- **LLM이 R과 S를 함께 출력**: R 개수와 S가 어긋나는 모순이 생길 수 있다.
  S를 R에서 계산하면 이 모순이 구조적으로 불가능해진다.
- **DSL 미지원 요구를 부분 점수로 채점**: 측정하지 못한 것을 측정한 것처럼 섞는다.

## 대체 관계

- [[2026-09-15-edu-vibe-llm-eval-pipeline]]의 **Step2(M 추출)**와 M용 TokenHub client tag
  `edu-vibe-llm-quality-eval-matching`을 대체한다. Gate, 생성·평가 AX 축 소유권,
  R·S용 tag `edu-vibe-llm-quality-eval-requirements` 분리는 유지한다.
- 같은 문서의 "R·S를 Step1에서 추출" 중 **S를 추출하는 부분**도 대체한다.

## 무엇이 바뀌면 이 판단이 무효인가

- 닫힌 스키마 R로 표현할 수 없는 요구(`UNMAPPED_REQUIREMENT`) 비중이 커져서
  측정 범위가 의미 없을 만큼 좁아지면, 의미 판정 단계를 다시 검토해야 한다.

## 변경 이력

- 2026-10-06: 2026-09-16 저널 백로그 2건(M/Judge 제거, S 코드 계산)을 병합해 승격
