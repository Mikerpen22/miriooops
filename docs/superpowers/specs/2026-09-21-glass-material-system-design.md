# Glass material system and fluid motion

Date: 2026-09-21. Status: approved.

## Goal

Give MiriLog a consistent "liquid glass" material system and fluid motion
without breaking the terminal-minimal identity. Glass is a material applied
by tier, never sprinkled per element.

## Surface tiers

| Tier | Class | Blur | Members |
|---|---|---|---|
| Flat | `.surface` | none | blockquote, tag-card, table header |
| Raised | `.surface--raised` | 12px | code-block header, tag pills, copy button, theme toggle, GitHub link |
| Floating | `.surface--floating` | 20px | site header when scrolled; future floating controls |

Each tier is four tokens: `--glass-fill-*`, `--glass-blur-*`, `--glass-edge`,
`--glass-shadow-*`. Edge is a 1px inset top highlight. Only tint is the
existing `--accent-tint`; glass itself is grayscale.

Fallbacks: `@supports not (backdrop-filter)` and
`prefers-reduced-transparency` collapse all tiers to solid fills.

## Motion tokens

- Curves: `--ease-out` (layout, fades), `--ease-spring` (controls <= 48px only).
- Durations: `--dur-fast` 150ms, `--dur-base` 250ms, `--dur-slow` 500ms.
- Transitions declared on resting state so every state change is reversible.
- Header morphs to a floating pill via `animation-timeline: scroll()`,
  with the `.scrolled` JS class as fallback.
- Cross-document view transitions via `@view-transition { navigation: auto }`.
- `prefers-reduced-motion` disables all of the above.

## Files

- `themes/mirilog/assets/css/` split into `tokens.css`, `surfaces.css`,
  `motion.css`, `main.css`; concatenated in `head.html`.
- `header.html`, `baseof.html` (code header JS), list/single templates gain
  surface classes.

## Out of scope

Specular sweeps, animated refraction, orange ambient glow.

## Verification

`hugo --gc --minify` builds clean. Screenshots light/dark, top/scrolled,
desktop/phone via Playwright.
