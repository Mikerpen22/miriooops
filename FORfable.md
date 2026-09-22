# FORfable.md — MiriLog, as seen from the glass pass

This is a companion to `FORcodex.md`, which already covers the Hugo
architecture, the content workflow, and the deploy path. Read that one first.
This file covers what changed when the theme gained a material system and
fluid motion, and what we learned doing it.

## Technical architecture

Nothing moved at the Hugo level. The site is still Markdown in `content/`,
a custom theme in `themes/mirilog/`, and Netlify running `hugo --gc --minify`.

What changed is how the stylesheet is organised. Think of it like a kitchen:
before, everything lived in one drawer. Now there is a drawer for ingredients
(`tokens.css`), one for the plates things are served on (`surfaces.css`), one
for how things move between the counter and the table (`motion.css`), and the
recipes themselves stay in `main.css`. Hugo's asset pipeline concatenates the
four into one fingerprinted file, so the browser still downloads a single
stylesheet. The split is for humans, not for the network.

## Codebase structure

```
themes/mirilog/assets/css/
  tokens.css     colors (light + dark), type scale, weights, tracking,
                 radii, spacing, glass material tokens, motion tokens
  surfaces.css   the three glass tiers and their fallbacks
  motion.css     entrance fades, header morph, view transitions,
                 reduced-motion overrides
  main.css       every component, layout, responsive and print rule
```

The order in `partials/head.html` matters. Tokens must come first because
every later file reads them. Motion comes before main so component rules can
still override a transition if they need to.

The theme toggle and GitHub link carry `surface--raised surface--quiet` in
markup. Tag pills look raised (rim plus border) but skip the blur on purpose:
their fill is nearly opaque so blur is invisible, and a page can show dozens
of them. Code-block chrome has its own always-dark material because the code
surface is dark in both themes and page-colored glass over it was unreadable
in light mode. Elements that come from Markdown (blockquote, tables) cannot
carry a class, so they reference the same tokens by element selector.

## Technologies used

- **CSS custom properties with `color-mix()`**. Every glass fill is the page
  background mixed with transparency, so it adapts to light and dark for free.
- **`backdrop-filter`**. The blur that makes glass read as glass. Wrapped in
  `@supports` so a browser without it gets a solid fill.
- **Scroll-driven animations** (`animation-timeline: scroll()`). The header
  morphs from a flat bar into a floating pill as a function of scroll
  position, not time. Safari lacks it, so the older JavaScript `.scrolled`
  class stays as the fallback and both paths land on the same final state.
- **Cross-document view transitions** (`@view-transition`). One rule, and
  navigating between posts crossfades instead of flashing. Browsers that do
  not support it navigate normally.
- **`clamp()` for type**. The four largest sizes scale with the viewport, so
  the mobile media query no longer needs its own font-size overrides.

We chose plain CSS over Sass because the theme already used plain CSS and
custom properties cover everything a mixin would have. Fewer build steps on
Netlify, fewer things to go wrong.

## Technical decisions

**Three tiers, not per-element styling.** The brief was "adopt liquid glass",
and the easy failure mode is sprinkling blur on whatever looks nice. Instead
every surface was classified: flat (in-flow content boxes), raised (chrome
sitting on content), floating (chrome hovering over scrolling content). Glass
strength follows the tier. If a new component appears, the first question is
"which tier is it", not "how much blur do I want".

**Quiet variant for header buttons.** Permanent frosted squares in the header
looked like buttons on a microwave. The quiet modifier keeps the material
invisible at rest and reveals it on hover, focus, and press.

**The rim is two inset shadows, not a border.** A 1px light line on top and a
1px dark line on the bottom is what makes a surface read as lit glass with
thickness. In dark mode the top rim drops to 9% white; a bright rim on a dark
surface reads as an outline instead of a highlight.

**Transitions live on the resting state.** Declaring `transition` on `.btn`
rather than `.btn:hover` means leaving mid-animation reverses smoothly. This
one change is most of what people mean by "fluid".

**Spring easing only on small controls.** Overshoot on a 32px button feels
alive. Overshoot on a 700px column looks like a wobble.

**One scale per property.** Before the audit the sheet had fifteen font sizes,
six tracking values, seven radii, and post titles at weight 400 next to
featured cards at 600 and h1 at 700. Now there is one type scale, three
weights, four tracking values, four radii, all tokens. The post title stays at
400 on purpose: Instrument Serif italic only ships in that weight.

## Lessons learned

**The `animation` shorthand resets `animation-timeline`.** If you write the
shorthand and then set the timeline, the order matters: timeline must come
after the shorthand or it silently reverts to `auto`, and the header morph
would just not run. `motion.css` sets the timeline after the shorthand for
this reason; keep that order if you ever touch it.

**Reduced motion needs a separate rule for scroll timelines.** The usual trick
of forcing `animation-duration` to near zero does nothing to a scroll-driven
animation because duration is not a concept there. It needs `animation: none`
explicitly, at which point the JavaScript fallback takes over.

**Animations win the cascade.** While a fill-mode `both` animation is active,
its values beat ordinary declarations. That is why the `.scrolled` class is
harmless in browsers that support scroll timelines: the animation simply
overrides it. Useful when you want a progressive enhancement layered over a
fallback without `@supports` gymnastics around every property.

**Bind the dev server to the host you will test against.** Hugo rewrote asset
URLs to `localhost` while the test browser opened `127.0.0.1`, and the
stylesheet was blocked by CORS. Ten minutes lost to a hostname.

**Audit before you polish.** The user asked for glass. The audit found the
inconsistencies underneath, and fixing those did more for how the site feels
than the blur did. Good engineers look at the tally of values first; the
`grep | sort | uniq -c` pattern on a stylesheet is a cheap x-ray.

**Do not trust "it looks fine" from one viewport.** The pill header was checked
at desktop and phone width, in both colour schemes, at top and scrolled, and
with reduced motion emulated. Each of those caught nothing this time, and that
is the point: the check is cheap and the alternative is a user finding it.
