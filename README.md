# MiriLog

Personal blog by Michael Shih — physicist turned engineer.

**Live**: [mirilog.netlify.app](https://mirilog.netlify.app/)

## Stack

- [Hugo](https://gohugo.io/) static site generator
- Custom `mirilog` theme (Geist + JetBrains Mono, Ubuntu orange accent)
- Deployed on [Netlify](https://netlify.com/)

## Local development

```bash
# Install Hugo (macOS)
brew install hugo

# Dev server with drafts
hugo server -D

# Production build
hugo --gc --minify

# Create new post
hugo new posts/my-post-title.md
```

## Theme

The `mirilog` theme is a custom terminal-minimal theme built for this blog. It supports:

- Light/dark mode via `prefers-color-scheme`
- GitHub Flavored Markdown (tables, task lists, strikethrough, footnotes)
- Tag taxonomy with tinted pill components
- Table of contents sidebar with scroll spy and progress bar
- Syntax highlighting (Dracula theme)
- Staggered entrance animations
- Responsive layout with fluid spacing
- RSS feed + [llms.txt](https://llmstxt.org/) output

## Writing a new post

### 1. Create the file

```bash
hugo new posts/my-post-title.md
```

This generates `content/posts/my-post-title.md` from the archetype with `draft: true` so it won't appear in production until you're ready.

### 2. Edit the front matter

```yaml
---
title: "My Post Title"
date: 2026-04-12
draft: true
tags: [kubernetes, networking]
---
```

| Field | Required | Notes |
|-------|----------|-------|
| `title` | Yes | Displayed as the post title and in the browser tab |
| `date` | Yes | ISO format (`YYYY-MM-DD`). Controls sort order on the home page |
| `draft` | Yes | Set to `true` while writing, `false` to publish |
| `tags` | No | Array of lowercase tags. Each tag gets a pill and its own `/tags/{name}/` page |

### 3. Write content

The blog supports full [GitHub Flavored Markdown](https://github.github.com/gfm/):

```markdown
## Headings generate the table of contents

Body text in **bold**, *italic*, and ~~strikethrough~~.

Inline `code` and fenced code blocks with syntax highlighting:

​```go
func main() {
    fmt.Println("hello")
}
​```

| Tables | work |
|--------|------|
| like   | this |

- [x] Task lists
- [ ] are supported

Footnotes[^1] work too.

[^1]: Like this one.
```

For images, place them in `static/images/` and reference them:

```markdown
![Alt text](/images/my-diagram.png)
```

Or use the image shortcode for positioning:

```markdown
{{</* image src="/images/my-diagram.png" alt="Description" */>}}
```

### 4. Preview locally

```bash
hugo server -D
```

Open `http://localhost:1313`. The `-D` flag includes drafts. Hugo live-reloads on save.

Posts with 2+ headings will show a table of contents sidebar on wide screens (>1200px).

### 5. Publish

Set `draft: false` in the front matter, commit, and push:

```bash
git add content/posts/my-post-title.md
git commit -m "feat: add post on my-post-title"
git push
```

Netlify auto-deploys from `main`. The post will be live at `mirilog.netlify.app/posts/YYYY/MM/my-post-title/`.

## License

Content is [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). Theme code is MIT.
