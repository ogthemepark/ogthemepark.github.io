# Hugo Blog Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Set up a Hugo blog with the Blowfish theme, auto-deployed to `ogthemepark.github.io` via GitHub Actions on every push to `main`.

**Architecture:** Hugo Extended generates static HTML from Markdown posts. Blowfish theme is installed as a git submodule. GitHub Actions builds and deploys to GitHub Pages on every push to `main`. The local workflow is: write `.md` post → `git push` → live in ~1 minute.

**Tech Stack:** Hugo Extended v0.159.1, Blowfish theme (git submodule), GitHub Actions, GitHub Pages

---

## File Map

| File | Action | Responsibility |
|---|---|---|
| `hugo.yaml` | Create | Site-wide config: baseURL, title, theme, Blowfish params |
| `themes/blowfish/` | Submodule | Blowfish theme source |
| `.github/workflows/hugo.yaml` | Create | GitHub Actions: build + deploy on push to main |
| `content/posts/hello-world.md` | Create | First post to verify the pipeline end-to-end |
| `static/` | Exists | Place for future images, favicon, CNAME |
| `.gitmodules` | Auto-created | Submodule registry (created by git submodule add) |

---

## Task 1: Initialize Hugo Site

**Files:**
- Creates: `hugo.yaml`, `archetypes/`, `content/`, `layouts/`, `static/`, `assets/`

- [ ] **Step 1: Install Hugo Extended**

```bash
wget -O /tmp/hugo.deb \
  https://github.com/gohugoio/hugo/releases/download/v0.159.1/hugo_extended_0.159.1_linux-amd64.deb \
&& sudo dpkg -i /tmp/hugo.deb
```

Verify: `hugo version` → should print `hugo v0.159.1+extended`

- [ ] **Step 2: Scaffold the Hugo site**

Run from `/home/allino/blog/`:

```bash
hugo new site . --format yaml --force
```

The `--force` flag is needed because the directory already exists (it has the `docs/` folder).

Expected output:
```
Congratulations! Your new Hugo site was created in /home/allino/blog/
```

- [ ] **Step 3: Verify scaffold**

```bash
ls
```

Expected: `archetypes  assets  content  data  hugo.yaml  i18n  layouts  static  themes  docs`

- [ ] **Step 4: Commit scaffold**

```bash
git add .
git commit -m "chore: scaffold Hugo site"
```

---

## Task 2: Install Blowfish Theme as Git Submodule

**Files:**
- Creates: `themes/blowfish/` (submodule), `.gitmodules`

- [ ] **Step 1: Add Blowfish as a submodule**

```bash
git submodule add https://github.com/nunocoracao/blowfish.git themes/blowfish
cd themes/blowfish && git checkout $(git describe --tags `git rev-list --tags --max-count=1`) && cd ..
git add themes/blowfish
```

Expected: clones Blowfish into `themes/blowfish/`, creates `.gitmodules`

- [ ] **Step 2: Verify submodule**

```bash
ls themes/blowfish/
```

Expected: `assets  exampleSite  i18n  layouts  ...` (theme files present)

- [ ] **Step 3: Commit submodule**

```bash
git add .gitmodules themes/blowfish
git commit -m "chore: add Blowfish theme as git submodule"
```

---

## Task 3: Configure the Site

**Files:**
- Modify: `hugo.yaml`

- [ ] **Step 1: Replace default hugo.yaml with full config**

Overwrite `hugo.yaml` with:

```yaml
baseURL: "https://ogthemepark.github.io/"
languageCode: "en-us"
title: "ogthemepark"
theme: "blowfish"

enableRobotsTXT: true
paginate: 10

params:
  colorScheme: "ocean"
  defaultAppearance: "dark"
  autoSwitchAppearance: true
  enableSearch: true
  enableCodeCopy: true

  author:
    name: "ogthemepark"

menus:
  main:
    - name: Posts
      pageRef: /posts
      weight: 10
    - name: Tags
      pageRef: /tags
      weight: 20
```

- [ ] **Step 2: Verify config parses correctly**

```bash
hugo config | head -20
```

Expected: no errors, prints config values

- [ ] **Step 3: Commit config**

```bash
git add hugo.yaml
git commit -m "feat: configure Hugo site with Blowfish theme"
```

---

## Task 4: Verify Site Builds Locally

**Files:**
- No new files

- [ ] **Step 1: Start local dev server**

```bash
hugo server -D
```

Expected output includes:
```
Web Server is available at http://localhost:1313/
```

- [ ] **Step 2: Open in browser and verify**

Navigate to `http://localhost:1313/` — should show the Blowfish themed homepage (may be empty, that's fine — no posts yet).

- [ ] **Step 3: Stop server**

Press `Ctrl+C`

---

## Task 5: Create First Post

**Files:**
- Create: `content/posts/hello-world.md`

- [ ] **Step 1: Create the post**

```bash
hugo new content posts/hello-world.md
```

- [ ] **Step 2: Edit the post**

Open `content/posts/hello-world.md` and replace its contents with:

```markdown
---
title: "Hello World"
date: 2026-03-29
tags: ["meta", "first-post"]
draft: false
---

Welcome to my blog! I'll be documenting everything I learn here and sharing it freely.

This is the start of something I've wanted to do for a long time. Let's go.
```

- [ ] **Step 3: Verify post appears locally**

```bash
hugo server
```

Navigate to `http://localhost:1313/` — the "Hello World" post should appear. Stop server with `Ctrl+C`.

- [ ] **Step 4: Commit first post**

```bash
git add content/posts/hello-world.md
git commit -m "feat: add first blog post"
```

---

## Task 6: Set Up GitHub Actions Workflow

**Files:**
- Create: `.github/workflows/hugo.yaml`

- [ ] **Step 1: Create the workflow directory**

```bash
mkdir -p .github/workflows
```

- [ ] **Step 2: Create the workflow file**

Create `.github/workflows/hugo.yaml`:

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.159.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Install Dart Sass
        run: sudo snap install dart-sass

      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"

      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

- [ ] **Step 3: Commit the workflow**

```bash
git add .github/workflows/hugo.yaml
git commit -m "ci: add GitHub Actions workflow for Hugo deployment"
```

---

## Task 7: Create GitHub Repo and Push

**Files:**
- No new files

- [ ] **Step 1: Create the GitHub repository**

Go to https://github.com/new and create a repo named exactly `ogthemepark.github.io`
- Set it to **Public**
- Do NOT initialize with README, .gitignore, or license (repo must be empty)

- [ ] **Step 2: Add remote and rename branch to main**

```bash
git remote add origin https://github.com/ogthemepark/ogthemepark.github.io.git
git branch -m master main
```

- [ ] **Step 3: Push to GitHub**

```bash
git push -u origin main
```

- [ ] **Step 4: Enable GitHub Pages source**

In the GitHub repo:
1. Go to **Settings → Pages**
2. Under **Source**, select **"GitHub Actions"**
3. Click **Save**

- [ ] **Step 5: Verify deployment**

Go to **Actions** tab in the GitHub repo. The first workflow run should be in progress or complete. Once green, navigate to `https://ogthemepark.github.io/` — the blog should be live with the Hello World post.

Expected: Blowfish-themed blog, dark mode, Hello World post visible.

---

## Task 8: Verify End-to-End Pipeline

**Files:**
- Create: `content/posts/pipeline-test.md` (temporary, can delete after)

- [ ] **Step 1: Create a test post**

```bash
hugo new content posts/pipeline-test.md
```

Edit `content/posts/pipeline-test.md`:

```markdown
---
title: "Pipeline Test"
date: 2026-03-29
tags: ["test"]
draft: false
---

Testing the push-to-publish pipeline.
```

- [ ] **Step 2: Push and watch deploy**

```bash
git add content/posts/pipeline-test.md
git commit -m "test: verify push-to-publish pipeline"
git push
```

Go to the **Actions** tab — watch the workflow run. When green, check `https://ogthemepark.github.io/` for the new post.

- [ ] **Step 3: Pipeline confirmed — delete test post (optional)**

```bash
git rm content/posts/pipeline-test.md
git commit -m "chore: remove pipeline test post"
git push
```

---

## Writing a New Post (Reference)

For every future post, the workflow is:

```bash
hugo new content posts/YYYY-MM-DD-your-title.md
# write in content/posts/YYYY-MM-DD-your-title.md
# change draft: false when ready
git add content/posts/YYYY-MM-DD-your-title.md
git commit -m "post: your title"
git push
```

Done. Live within ~1 minute.
