# Hugo Blog Design Spec
**Date:** 2026-03-29
**Status:** Approved

## Overview

A personal learning-in-public blog hosted on GitHub Pages using Hugo and the Blowfish theme. The author documents things they learn and shares them freely with the world. Markdown-based writing workflow: write a post, push to `main`, it auto-deploys.

## Goals

- Write posts in Markdown locally
- Push to GitHub → auto-publish via GitHub Actions
- Clean, modern aesthetic with dark/light mode
- Start with `username.github.io`, migrate to custom domain later (no rework needed)

## Architecture

```
username/username.github.io  (GitHub repo)
├── .github/
│   └── workflows/
│       └── hugo.yaml          ← GitHub Actions: build + deploy on push to main
├── content/
│   └── posts/                 ← Markdown blog posts live here
├── static/                    ← Images, favicon, CNAME (added later for custom domain)
├── hugo.yaml                  ← Site configuration
└── themes/
    └── blowfish/              ← Theme as a git submodule
```

## Tech Stack

| Layer | Choice | Notes |
|---|---|---|
| Static site generator | Hugo Extended v0.159.1 | Extended edition required for Blowfish (Tailwind CSS / SCSS) |
| Theme | Blowfish | Tailwind CSS 3, dark/light mode, multiple color schemes, search |
| Hosting | GitHub Pages | Free, supports custom domain (HTTPS enforced) |
| CI/CD | GitHub Actions (official workflow) | Triggers on push to `main`, uses `actions/deploy-pages@v5` |
| Theme management | Git submodule | Keeps theme version-pinned and updatable |

## GitHub Actions Workflow

File: `.github/workflows/hugo.yaml`

- Triggers on push to `main` and manual dispatch
- Installs Hugo Extended + Dart Sass
- Checks out repo with `submodules: recursive` (required for Blowfish submodule)
- Builds with `--gc --minify`, injects `baseURL` dynamically from `configure-pages`
- Uploads artifact and deploys via `actions/deploy-pages@v5`

Hugo version is pinned in the workflow `env:` block for reproducible builds.

## Theme: Blowfish

Key features included out of the box:
- Auto dark/light mode (follows OS preference + manual toggle)
- Multiple color schemes (configured in `hugo.yaml`)
- Fuse.js full-text search
- Table of contents on posts
- Tags and categories
- Reading time indicator
- Code syntax highlighting + copy button
- Responsive / mobile-friendly
- Mermaid diagrams and KaTeX math (available when needed)

## Content Structure

```
content/
└── posts/
    └── YYYY-MM-DD-post-title.md
```

Each post uses Hugo front matter:
```yaml
---
title: "Post Title"
date: 2026-03-29
tags: ["tag1", "tag2"]
draft: false
---
```

## Custom Domain (Future)

When a custom domain is purchased:
1. Add `static/CNAME` with the domain (e.g. `www.yourdomain.com`)
2. Update `baseURL` in `hugo.yaml`
3. Add DNS records at registrar (A records for apex, CNAME for www)
4. Set custom domain in GitHub repo Settings → Pages → enforce HTTPS

No structural changes needed — the setup is designed to support this from day one.

## Out of Scope (for now)

- Comments system (Giscus, Disqus)
- Analytics
- Newsletter / email subscription
- Custom domain (deferred)
