# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Slidev presentation about Claude Code features (Skills, Rules, Hooks, MCPs, SubAgents) for real-world team workflows. 15 slides, 15-minute talk.

## Commands

```bash
npm run dev      # Start Slidev dev server (live reload)
npm run build    # Build static SPA for deployment
npm run export   # Export slides to PDF
```

## Architecture

- **`slides.md`** — Main presentation file. All 15 slides live here, separated by `---`. Uses `seriph` theme with per-slide frontmatter for layouts (`cover`, `default`, `two-cols-header`, `center`, `end`).
- **`style.css`** — Custom CSS: card system (`.card`, `.card-skill`, `.card-rule`, `.card-hook`), feature color coding (Skills=blue `#60a5fa`, Rules=green `#34d399`, Hooks=amber `#fbbf24`), gradient headings, Claude icon watermark.
- **`slides-content.md`** — Original plain-text content (reference only, not used by Slidev).
- **`public/`** — Static assets served at root (`/backgrounds/`, `/claude-icon.svg`).

## Slidev Conventions

- Speaker notes go in `<!-- -->` HTML comments after slide content.
- `two-cols-header` layout requires **both** `::left::` and `::right::` slot markers. The heading goes above columns, then `::left::` starts left column content.
- Progressive reveal uses `<v-clicks>` wrapper or `v-click` directive.
- Code block line highlighting: ` ```jsonc {5,8} `.
- Background images referenced as `/backgrounds/filename.jpg` (resolved from `public/`).

## Style

- Professional tone — no emojis in slides.
- Cards used sparingly (Main Features overview, Why Claude Code value props).
- 2-space indentation, LF line endings (see `.editorconfig`).
