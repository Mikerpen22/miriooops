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
- **Stable header geometry**. JavaScript toggles `.scrolled` to change the
  glass fill, border, and shadow. Its dimensions stay fixed while scrolling;
  mobile uses two rows with 44px navigation controls.
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

**Shared timing for interactions.** Small controls use the 150ms token;
colour changes and navigation crossfades use 250ms; content fades use 320ms.
Reduced-motion preferences disable animations and smooth scrolling.

**One scale per property.** Before the audit the sheet had fifteen font sizes,
six tracking values, seven radii, and post titles at weight 400 next to
featured cards at 600 and h1 at 700. Now there is one type scale, three
weights, four tracking values, four radii, all tokens. Geist headings and
article-card titles use 600, prose uses 400, and metadata and code use
JetBrains Mono. Home and archive cards share one template.

## Lessons learned

**Reserve space for rotating text.** The home introduction measures every
phrase in an invisible grid layer so changing its text cannot move the post
list below it. Reduced motion keeps the first phrase visible.

**Keep reveal timing in CSS.** JavaScript only toggles reveal classes and a
bounded delay. No-JavaScript and reduced-motion visitors see every card,
and changing the motion preference clears pending reveals immediately.

**Sort explicitly when chronology matters.** Hugo's default ordering uses
front-matter weights before dates. Home, archive, taxonomy, RSS, and LLM
indexes explicitly use date descending so older weighted posts cannot bury
new articles.

**Bind the dev server to the host you will test against.** Hugo rewrote asset
URLs to `localhost` while the test browser opened `127.0.0.1`, and the
stylesheet was blocked by CORS. Ten minutes lost to a hostname.

**Audit before you polish.** The user asked for glass. The audit found the
inconsistencies underneath, and fixing those did more for how the site feels
than the blur did. Good engineers look at the tally of values first; the
`grep | sort | uniq -c` pattern on a stylesheet is a cheap x-ray.

**Do not trust "it looks fine" from one viewport.** The responsive layout was checked
from 320px to 1440px, in both colour schemes, at top and scrolled, with
reduced motion, storage blocked, and JavaScript disabled. Check interaction
behaviour as well as screenshots: focus, clipboard, overflow, and sorting
all affect whether the site is usable.
