---
title: 한국어 텍스트 줄바꿈 — break-keep 오버플로 함정
area: developer
tags: [css, tailwind, korean, typography, debugging]
created: 2026-08-24
updated: 2026-09-01
status: confirmed
---

# 한국어 텍스트 줄바꿈 — break-keep 오버플로 함정

INOS에서 사용자가 쓴 긴 토론 발제문이 줄바꿈 없이 한 줄로 렌더링되어 화면 밖으로 흘러나가면서
발견했다 ([[inos]]).

## 함정

`word-break: keep-all`(Tailwind `break-keep`)은 한국어에서 흔히 쓰는 선택이다. 줄 끝에서 단어가
음절 중간에 잘리는 것을 막아주기 때문이다. 하지만 keep-all은 **공백만이 줄바꿈 기회**라는 뜻이다.
한국어는 띄어쓰기 없이 입력되는 경우가 잦고, 그러면 공백 없는 긴 덩어리는 끊을 곳이 없어서
줄바꿈 대신 가로로 넘쳐버린다.

실제 페이지에서 측정한 값: 600px 컬럼 안의 공백 없는 한국어 문장이 `clientWidth: 600`에 대해
`scrollWidth: 3578`을 냈다 — 한 줄, 화면 밖은 전부 안 보임.
`break-keep`이 없으면 같은 텍스트가 잘 감긴다. CJK는 기본적으로 임의의 두 글자 사이에서
줄바꿈이 허용되기 때문이다 (UAX #14).

## 동작하는 3종 세트

```
white-space: pre-wrap;      /* 사용자가 입력한 줄바꿈을 보존 */
word-break: keep-all;       /* 가급적 공백(어절 단위)에서 끊기 */
overflow-wrap: break-word;  /* 그래도 안 들어가면 어쨌든 끊기 */
```

Tailwind: `whitespace-pre-wrap break-keep break-words`. 검증 완료: 가로 오버플로 없음,
줄바꿈 보존됨.

- `pre-wrap`이 없으면 사용자가 textarea에 입력한 줄바꿈이 공백으로 붕괴되어 여러 줄 입력이
  한 덩어리로 렌더링된다. 오버플로와는 별개의 버그인데 둘이 함께 나타났다.
- `overflow-wrap: anywhere`도 동작하고 더 적극적으로 끊는다. 여기서는 `break-word`로 충분하다.

## 부수적으로 배운 두 가지

- **Tailwind는 소스 파일에서 발견한 클래스만 생성한다.** 후보 클래스를 런타임에 DOM에 주입해
  테스트하면 조용히 아무 일도 일어나지 않는다 — CSS 자체가 생성된 적이 없기 때문이다.
  CSS 자체는 인라인 스타일로 검증하고, 그다음 클래스를 소스 파일에 써 넣을 것.
- **`<p>` 안의 `<div>`는 유효하지 않다.** 문장 단위 줄바꿈을 처음 시도할 때 조각들을 `p` 안의
  `div`로 감쌌는데, 브라우저가 문단을 강제로 닫아버려서 레이아웃이 깨졌다.
  대신 `<span className="block">`을 쓸 것.

## 관련

- 발제문의 문장 단위 렌더링: `.?!`(전각 포함)에서 분리하되 숫자 앞에서는 분리하지 않고(`3.5` 보호),
  다른 종결부호 앞에서도 분리하지 않고(`...` 보호), 닫는 따옴표 앞에서도 분리하지 않는다.
  [[coding-style]]의 UI 취향 참고.
