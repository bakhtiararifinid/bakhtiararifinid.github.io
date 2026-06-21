---
title: "Create a Hugo Site and Deploy to GitHub Pages"
date: 2026-06-21
draft: false
tags: ["hugo", "github-pages", "tutorial"]
summary: "A step-by-step guide to building a Hugo blog with the PaperMod theme and deploying it automatically to GitHub Pages with GitHub Actions."
---

This blog is built with [Hugo](https://gohugo.io/), themed with
[PaperMod](https://github.com/adityatelange/hugo-PaperMod), and deployed
automatically to [GitHub Pages](https://pages.github.com/). Here's exactly how
to set it up from scratch.

## 1. Install Hugo (extended)

The extended edition is required for the PaperMod theme because it processes
SCSS. On macOS:

```bash
brew install hugo
hugo version
```

Make sure the output contains `extended`.

## 2. Create the site

```bash
hugo new site bakhtiararifinid
cd bakhtiararifinid
git init
```

## 3. Add the PaperMod theme as a submodule

Using a Git submodule keeps the theme separate and easy to update:

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --init --recursive
```

## 4. Configure the site

Replace the generated config with a `hugo.toml` that selects the theme and turns
on the features you want:

```toml
baseURL = "https://<username>.github.io/"
title = "Bakhtiar Arifin"
theme = "PaperMod"

enableInlineShortcodes = true
enableRobotsTXT = true
enableEmoji = true

[outputs]
  home = ["HTML", "RSS", "JSON"]

[params]
  env = "production"
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowCodeCopyButtons = true
  ShowToc = true
```

The `JSON` output on the home page powers PaperMod's built-in search.

## 5. Write your first post

```bash
hugo new posts/hello-world.md
```

Open the file, set `draft: false`, and add some content. Preview locally with:

```bash
hugo server -D
```

Then visit `http://localhost:1313/`.

## 6. Deploy with GitHub Actions

Create `.github/workflows/hugo.yml`. This builds the site on every push to
`main` and publishes it to Pages:

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
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.163.3
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
        run: hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
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
        uses: actions/deploy-pages@v4
```

Two details that matter: `submodules: recursive` pulls in the theme, and
`HUGO_VERSION` should match a version that ships the extended binary.

## 7. Turn on Pages and push

In your repository on GitHub, go to **Settings → Pages** and set the source to
**GitHub Actions**. Then push:

```bash
git add .
git commit -m "Initial Hugo site"
git branch -M main
git remote add origin https://github.com/<username>/<username>.github.io.git
git push -u origin main
```

The workflow runs, builds the site, and your blog goes live at
`https://<username>.github.io/`. Every push to `main` from now on redeploys
automatically.
