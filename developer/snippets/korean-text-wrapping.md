---
title: Korean text wrapping — the break-keep overflow trap
area: developer
tags: [css, tailwind, korean, typography, debugging]
created: 2026-08-24
updated: 2026-08-24
status: confirmed
---

# Korean text wrapping — the break-keep overflow trap

Hit on INOS while long user-written discussion prompts rendered as a single unwrapped line
running off the screen ([[inos]]).

## The trap

`word-break: keep-all` (Tailwind `break-keep`) is the usual choice for Korean because it stops
words from splitting mid-syllable at line ends. But keep-all means **only spaces are break
opportunities**. Korean is frequently typed without spaces, so one long space-less run has
nowhere to break and overflows horizontally instead of wrapping.

Measured on the real page: a space-less Korean sentence in a 600px column produced
`scrollWidth: 3578` against `clientWidth: 600` — one line, everything past the fold invisible.
Without `break-keep` the same text wraps fine, because CJK allows a break between any two
characters by default (UAX #14).

## The trio that works

```
white-space: pre-wrap;      /* keep the newlines the user typed */
word-break: keep-all;       /* prefer breaking at spaces (어절 단위) */
overflow-wrap: break-word;  /* but break anyway when a run cannot fit */
```

Tailwind: `whitespace-pre-wrap break-keep break-words`. Verified: no horizontal overflow,
newlines preserved.

- Without `pre-wrap`, newlines the user typed in a textarea collapse into spaces and a
  multi-line entry renders as one blob. This is a separate bug from the overflow one and both
  showed up together.
- `overflow-wrap: anywhere` also works and breaks more eagerly; `break-word` is enough here.

## Two side lessons

- **Tailwind only generates classes it finds in source files.** Testing a candidate class by
  injecting it into the DOM at runtime silently does nothing — the CSS was never generated.
  Validate the CSS itself with inline styles, then write the class into a source file.
- **`<div>` inside `<p>` is invalid.** A first attempt at sentence-per-line splitting wrapped
  fragments in `div`s inside a `p`; the browser force-closes the paragraph and layout breaks.
  Use `<span className="block">` instead.

## Related

- Sentence-per-line rendering for prompts: split on `.?!` (plus full-width) but not before a
  digit (protects `3.5`), not before another terminator (protects `...`), and not before a
  closing quote. See [[coding-style]] UI taste.
