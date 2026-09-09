---
title: 한글 웹폰트 30벌을 text= 서브셋으로 증분 로딩하기
area: developer
tags: [webfont, google-fonts, 한글, 성능, IntersectionObserver, INOS]
created: 2026-09-09
updated: 2026-09-09
status: draft
---

# 한글 웹폰트 30벌을 text= 서브셋으로 증분 로딩하기

> INOS 서가에서 나온 것. 책등마다 다른 한글 서체를 입히려면 **30벌**이 필요한데,
> 한글 폰트는 한 벌이 수 MB다. 전부 받으면 끝이다.
> 줄바꿈 쪽 함정은 [[korean-text-wrapping]].

## 문제

책등 30종에 각각 다른 서체를 쓰고 싶다. 한글 폰트는 글리프가 수천 자라
라틴 폰트처럼 "그냥 30개 링크" 할 수 없다.

## 해법의 축: `text=`

Google Fonts CSS2는 `text=` 파라미터로 **넘긴 글자만 담은 서브셋**을 만들어 준다.

```
https://fonts.googleapis.com/css2?family=A&family=B&...&text=<필요한 글자들>&display=swap
```

화면에 실제로 있는 제목들의 글자 집합만 넘기면, 30벌이라도 받는 양이 몇 KB로 떨어진다.

## 핵심 발견 — 응답에 `unicode-range`가 실린다

여기서 막히는 지점이 있다. 페이지네이션으로 책이 더 붙으면 새 글자가 생기는데,
**글자 집합을 다시 만들어 통째로 재요청하면** 이미 받은 것까지 또 받는다.

실제 응답을 열어 보니 `@font-face`에 **`unicode-range`가 그대로 실려 있었다.**
이게 결정적이다 — `unicode-range`가 있으면 여러 스타일시트가 **코드포인트 단위로 병합**된다.
같은 family에 대한 두 번째 `<link>`가 첫 번째를 덮어쓰지 않고 **보충**한다.

그래서 증분 요청이 성립한다.

```
// 모듈 레벨: 이미 받은 글자
const loadedChars = new Set<string>();

const missing = charsOf(titles).filter((c) => !loadedChars.has(c));
if (missing.length === 0) return;          // 요청할 것이 없다
missing.forEach((c) => loadedChars.add(c));

const link = document.createElement('link');
link.rel = 'stylesheet';
link.href = buildHref(missing);            // 부족한 글자만
document.head.appendChild(link);
```

**요청량이 «화면의 전체 글자»가 아니라 «새로 등장한 글자»에 비례한다.**

## 언제 쏘는가 — IntersectionObserver + 폴백 타이머

서가가 화면 밖에 있으면 받을 이유가 없다.

- `IntersectionObserver`에 `rootMargin: '400px'` — 스크롤이 닿기 전에 미리
- 관찰이 영영 발화하지 않는 경우(스크롤 컨테이너 구성, 축소 브라우저 등)를 위해
  **3초 폴백 타이머**를 같이 건다. 관찰만 믿으면 폰트가 영원히 안 오는 화면이 생긴다

## 부수적으로 걸린 것들

- **존재하지 않는 family 이름은 HTTP 400.** 조용히 폴백되지 않는다.
  30벌 목록을 손으로 적었다면 한 번은 통째로 400이 난다 — 이름 오타를 먼저 의심할 것
- `display=swap`은 그대로 유지한다. 서브셋이라 작아도 네트워크가 느리면 빈 칸이 생긴다
- 글자 집합에는 제목뿐 아니라 **장식 문자**(별점, 말줄임 등)도 넣어야 한다.
  빠뜨리면 그 글자만 폴백 서체로 튄다 — 30벌 중 한 벌만 어긋나 보여서 원인 찾기가 오래 걸린다

## 곁다리 — 인접 중복은 해시로 못 막는다

책등 스타일을 `hash(seed) % 30`으로 고르면 **정체성은 안정**(같은 책은 늘 같은 서체)이지만
이웃끼리 같은 값이 나오는 걸 못 막는다. 둘 다 원하면 순회하며 한 칸 밀면 된다.

```
let prev = -1;
seeds.map((seed) => {
  let i = hash(seed) % N;
  if (i === prev) i = (i + 1) % N;   // 인접 중복만 회피
  prev = i;
  return STYLES[i];
});
```

랜딩처럼 목록이 고정된 곳은 아예 stride(`(index * 7 + 3) % 30`)가 낫다 — 해시가 필요 없다.

## 관련

- [[korean-text-wrapping]] — 같은 서가에서 나온 한글 줄바꿈 함정
- [[inos]] — 이 작업이 속한 프로젝트
