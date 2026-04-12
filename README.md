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

## Content

Posts live in `content/posts/` as Markdown files. Front matter:

```yaml
---
title: "Post Title"
date: 2026-01-01
draft: false
tags: [tag1, tag2]
---
```

## License

Content is [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). Theme code is MIT.
