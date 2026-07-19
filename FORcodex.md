# MiriLog, Explained for the Next Engineer

MiriLog is Michael Shih's personal technical blog. It is intentionally small: Markdown files go in, Hugo turns them into static HTML, and Netlify publishes the result. There is no database, application server, client-side framework, or content-management system. That simplicity is a feature. A blog post should remain readable in a text editor even if every other part of the site disappears.

The site's personality is closer to a well-kept engineering notebook than a publication. The visual language borrows the useful parts of a terminal—monospace metadata, compact navigation, ISO dates—without turning the page into a fake command prompt. The writing should follow the same rule: technical and precise, but written for a person rather than for a search engine or an analyst template.

## Technical Architecture

The build has a short data path:

1. Hugo reads global settings from `config.toml`.
2. It parses Markdown and front matter from `content/`.
3. It resolves templates from top-level `layouts/` first, then falls back to `themes/mirilog/layouts/`.
4. It processes `themes/mirilog/assets/css/main.css` through Hugo Pipes, minifies it, and adds a content fingerprint.
5. It copies files from `static/` unchanged.
6. It writes the finished site to `public/`.
7. Netlify runs the same production build and serves `public/`.

There are two kinds of output. Normal pages use the HTML templates in the theme. The home page also emits RSS, `llms.txt`, `llms-full.txt`, and a custom robots response. This makes the same source useful to browsers, feed readers, and text-oriented tools without maintaining separate copies of each post.

At runtime, the site is still mostly static. A small amount of plain JavaScript handles the theme preference, sticky-header state, code-copy buttons, entrance animation, table-of-contents scroll tracking, reading progress, and the rotating line in the home-page introduction. None of those scripts is required to read the posts.

## Codebase Structure

### Content and configuration

- `config.toml` is the control panel. It defines the base URL, menus, taxonomies, permalink format, Markdown behavior, theme parameters, and output formats.
- `content/posts/` contains the blog posts. Each file has YAML front matter followed by Markdown.
- `content/about.md` provides the About page.
- `archetypes/default.md` is the starting template used by `hugo new`.
- `static/` contains assets that Hugo copies without processing, primarily images and the PDF.js files used by the PDF shortcode.

The date in a post's front matter controls its position on the home page. The file modification time does not. Production builds exclude drafts and future-dated posts because `buildDrafts` and `buildFuture` are both disabled.

### Theme

`themes/mirilog/` is a local, custom Hugo theme:

- `layouts/_default/baseof.html` provides the shared page shell and the small site-wide JavaScript behaviors.
- `layouts/_default/single.html` renders a post, its metadata, optional table of contents, tags, and back link.
- `layouts/_default/list.html` and `terms.html` render section and taxonomy pages.
- `layouts/index.html` renders the introduction and chronological post list on the home page.
- `layouts/partials/` contains the document head, header, and footer.
- `layouts/shortcodes/image.html` renders the older image-shortcode syntax still used by some posts.
- `assets/css/main.css` contains the complete design system and responsive styles.

Top-level `layouts/` contains site-specific output that should override or extend the theme:

- `index.llms.txt` and `index.llmsfull.txt` generate text representations of the site.
- `robots.txt` and the related index templates control crawler output.
- `shortcodes/embed-pdf.html` embeds PDF documents.

Hugo's lookup order matters here. If a file exists both at the top level and inside the theme, the top-level version wins. This is the safest way to customize a vendored theme because it avoids editing a shared template when only this site needs the change.

### Generated directories

- `public/` is the production output.
- `resources/` contains Hugo's generated asset cache.

Both are build products, not source. They should not be hand-edited. A successful build may change them locally, but the durable change belongs in `content/`, `layouts/`, `static/`, or the theme source.

## Technologies Used

### Hugo

Hugo fits this project because it turns Markdown into a complete site with one fast binary. It provides front matter, taxonomies, permalinks, RSS, syntax highlighting, table-of-contents generation, template inheritance, and asset processing without a JavaScript build system.

The production environment uses Hugo 0.148.0. The extended build is required when Hugo processes SCSS, although the current custom theme uses a plain CSS entry point. Keeping the pinned version in `netlify.toml` makes local and hosted builds easier to compare.

### Goldmark and GitHub-Flavored Markdown

Hugo's Goldmark renderer handles the post body. Tables, task lists, strikethrough, and footnotes are enabled because they are normal building blocks in technical writing. Raw HTML is disabled with `unsafe = false`, which keeps Markdown content safer and makes custom rendering decisions explicit through templates and shortcodes.

### Hugo templates and Pipes

Go templates generate the HTML. Hugo Pipes minifies and fingerprints the theme stylesheet, so browsers can cache it aggressively while a content change automatically produces a new URL.

### Plain CSS and JavaScript

The site deliberately avoids a front-end framework. Its interactions are small enough that plain JavaScript is easier to understand and cheaper to ship. CSS custom properties define the light and dark palettes, with Ubuntu orange as the only accent hue. Geist is used for prose, JetBrains Mono for interface metadata and code, and Instrument Serif is available for selected display treatment.

### Netlify

Netlify runs `hugo --gc --minify` and publishes `public/`. A static host is a natural fit: it provides a CDN, HTTPS, and automatic deployment without adding an application runtime that needs monitoring.

## Important Technical Decisions

### Content is the source of truth

Posts live as Markdown in Git. This makes every change reviewable and keeps the archive portable. It also encourages a useful discipline: citations, uncertainty, and revisions are visible in the source rather than hidden in a publishing tool.

### The home page is the post list

There is no decorative landing page. Readers see the writing immediately. The first post receives a featured class, but the template does not invent a separate hero narrative.

### Terminal values without terminal cosplay

The design uses compact spacing, monospace metadata, `~/posts`, and `cd` language as small structural cues. It avoids fake prompts, green-on-black styling, and interface tricks that would make the site harder for non-engineers to use.

### Light and dark mode are equal

An inline script in the document head reads the saved preference before the stylesheet paints, preventing a flash of the wrong theme. When no preference has been saved, the site follows `prefers-color-scheme`. The toggle stores an explicit choice in `localStorage`.

### Progressive enhancement

JavaScript improves the experience but does not carry the content. A reader can still navigate, read, follow links, and copy code manually if scripts are blocked. Reduced-motion preferences disable the rotating introduction, and the CSS contains responsive behavior for narrow screens.

### Honest technical writing

The recent posts combine engineering mechanisms with market implications. Their strongest sections explain the mechanism directly: what is loaded from memory, what is reused across a batch, or how one agent delegates to another. Labels such as "known unknown," "honest ledger," and "the thesis is already running" obscure that work by sounding like a generic research memo. Prefer a plain claim, the evidence for it, and a clear uncertainty label.

## Lessons Learned

### Check front matter, not file timestamps

The newest post is determined by its `date` field. Files may be copied, restored, or edited later, so modification time is not reliable for publishing order.

### Strong metaphors are useful; stacked metaphors are not

One concrete analogy can help a reader understand a system. A paragraph full of "floors," "costumes," "ledgers," and "trades" makes the reader decode the prose before reaching the mechanism. When an explanation already contains unfamiliar hardware terms, the surrounding sentences should be especially literal.

### Do not promote one source into a consensus

An investor's post can be a useful framing, but it remains one investor's view. State what was found and what was not found. "I found one published argument" is more credible than inventing a proxy for Wall Street.

### Separate facts, estimates, and hypotheses

Technical readers notice when confidence labels drift. A vendor benchmark is not an independent result; a judge subagent score is not a benchmark; a single analyst estimate is not established fact. The posts should name the source and say how much weight the claim carries.

### Explain the causal chain before the conclusion

The MoE discussion works only if the reader can follow this sequence: tokens route to different experts, each expert sees a smaller effective batch, weight reuse declines, and memory traffic stays important. Jumping from "sparse model" to an investment conclusion skips the engineering that makes the post valuable.

### Generated output is not where fixes belong

Editing a file under `public/` may appear to fix the live page locally, but the next Hugo build will erase it. Change the Markdown, template, CSS, or static asset that produced the output, then rebuild.

### Validate the production path

Use `hugo server -D` for drafts and rapid preview, but finish with:

```bash
hugo --gc --minify
```

The production command catches template, shortcode, Markdown, and output-format failures under the same settings Netlify uses. Also run `git diff --check` so whitespace problems do not hide inside a large prose edit.

### Preserve unrelated work

This repository may contain local files and uncommitted changes unrelated to the current task. Inspect `git status` before editing, touch only the files required for the change, and never treat a generated directory or unfamiliar untracked file as disposable without confirmation.

## Everyday Workflow

Create and preview a draft:

```bash
hugo new posts/my-post-title.md
hugo server -D
```

Before publishing:

1. Confirm the title, date, description, tags, and draft status in front matter.
2. Read the post once for technical correctness and once only for prose.
3. Check that every footnote is referenced and every important external claim has a source.
4. Run `hugo --gc --minify`.
5. Review `git diff --check` and `git status --short`.

The final test is simple: a future version of Michael should be able to open the post, recover the mechanism quickly, and tell which parts were measured, inferred, or still uncertain.
