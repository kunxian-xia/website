# Build & Deploy a Personal Website with Hugo + GitHub Pages

A complete step-by-step guide — from zero to a live website.

---

## What You'll Need

- A computer with macOS, Windows, or Linux
- A [GitHub account](https://github.com/signup) (free)
- Basic comfort with the terminal / command line
- [Git](https://git-scm.com/downloads) installed on your machine

---

## Step 1: Install Hugo

**macOS (Homebrew):**

```bash
brew install hugo
```

**Windows (Chocolatey or Winget):**

```bash
choco install hugo-extended
# or
winget install Hugo.Hugo.Extended
```

**Linux (Snap):**

```bash
sudo snap install hugo
```

You can also download a prebuilt binary from the [Hugo releases page](https://github.com/gohugoio/hugo/releases). Choose the **extended** edition for full feature support (Sass/SCSS, etc.).

Verify the installation:

```bash
hugo version
```

You should see something like `hugo v0.160.0-extended...`.

---

## Step 2: Create a New Hugo Site

```bash
hugo new site my-website
cd my-website
```

This creates a project folder with the following structure:

```
my-website/
├── archetypes/      # Templates for new content
├── assets/          # Files processed by Hugo Pipes
├── content/         # Your pages and posts (Markdown)
├── data/            # Data files (JSON, YAML, TOML)
├── i18n/            # Internationalization files
├── layouts/         # Custom HTML templates
├── static/          # Static files (images, CSS, JS)
├── themes/          # Installed themes
└── hugo.toml        # Site configuration
```

---

## Step 3: Initialize Git

```bash
git init
```

---

## Step 4: Choose and Install a Theme

Browse themes at [themes.gohugo.io](https://themes.gohugo.io/). Here are some popular ones for personal/academic sites:

| Theme | Style | Good for |
|-------|-------|----------|
| [PaperMod](https://github.com/adityatelange/hugo-PaperMod) | Clean, minimal blog | Blogs, portfolios |
| [Blowfish](https://github.com/nunocoracao/blowfish) | Modern, feature-rich | Personal sites, blogs |
| [Hugo Profile](https://github.com/gurusabarish/hugo-profile) | Single-page portfolio | Developer portfolios |
| [Academic (Wowchemy)](https://github.com/HugoBlox/hugo-blox-builder) | Research-focused | Academic sites like yezhang.io |
| [Ananke](https://github.com/theNewDynamic/gohugo-theme-ananke) | Simple starter | Getting started quickly |

Install a theme as a Git submodule. For example, using PaperMod:

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Then tell Hugo to use it by editing `hugo.toml`:

```toml
baseURL = 'https://YOUR-USERNAME.github.io/'
languageCode = 'en-us'
title = 'Your Name'
theme = 'PaperMod'
```

> **Tip:** Replace `YOUR-USERNAME` with your actual GitHub username. If you're using a project repo (not `username.github.io`), set `baseURL` to `https://YOUR-USERNAME.github.io/REPO-NAME/`.

Each theme has its own configuration options. Check the theme's README or documentation for details on customizing the sidebar, navigation, social links, colors, etc.

---

## Step 5: Add Content

Create your first post:

```bash
hugo new content posts/my-first-post.md
```

This creates `content/posts/my-first-post.md`. Open it in your editor:

```markdown
+++
title = 'My First Post'
date = 2026-04-15T10:00:00+08:00
draft = true
+++

## Hello World

This is my first blog post built with Hugo!
```

**Key notes:**

- The `+++` block at the top is called **front matter** — it holds metadata like title, date, and draft status.
- `draft = true` means this post won't appear in the production build. Set it to `false` when you're ready to publish.
- Write everything below the front matter in standard **Markdown**.

To add standalone pages (like an About page):

```bash
hugo new content about.md
```

Then edit `content/about.md` and set `draft = false`.

---

## Step 6: Preview Locally

```bash
hugo server -D
```

- The `-D` flag includes draft content so you can preview unpublished posts.
- Open your browser to **http://localhost:1313**
- Hugo live-reloads — every time you save a file, the browser updates instantly.
- Press `Ctrl+C` to stop the server.

Spend some time here tweaking your `hugo.toml` config and content until you're happy with how things look.

---

## Step 7: Create a GitHub Repository

1. Go to [github.com/new](https://github.com/new)
2. Name the repository:
   - For a **user site**: `YOUR-USERNAME.github.io` (your site will be at `https://YOUR-USERNAME.github.io/`)
   - For a **project site**: any name you like (your site will be at `https://YOUR-USERNAME.github.io/REPO-NAME/`)
3. Set it to **Public**
4. Do **not** add a README, .gitignore, or license (your local repo already exists)
5. Click **Create repository**

Link your local repo to GitHub:

```bash
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO-NAME.git
git branch -M main
```

---

## Step 8: Configure GitHub Pages Source

1. Go to your repository on GitHub
2. Click **Settings** → **Pages** (in the left sidebar)
3. Under **Source**, change it to **GitHub Actions**

This tells GitHub to use a workflow file (which you'll create next) to build and deploy your site.

---

## Step 9: Add the Image Cache Config

In your `hugo.toml`, add the following so image caching works correctly in CI:

```toml
[caches]
  [caches.images]
    dir = ':cacheDir/images'
```

---

## Step 10: Create the GitHub Actions Workflow

Create the workflow directory and file:

```bash
mkdir -p .github/workflows
```

Create the file `.github/workflows/hugo.yaml` with the following content:

```yaml
name: Build and deploy

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.160.0
    steps:
      - name: Checkout
        uses: actions/checkout@v6
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v6

      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir -p "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"

      - name: Install Node.js dependencies
        run: |
          [[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true

      - name: Build the site
        run: |
          hugo build \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v5
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

> **Note:** This is a simplified version of Hugo's official workflow. If your theme uses Sass, you may also need the Dart Sass and Go steps from the [full official workflow](https://gohugo.io/host-and-deploy/host-on-github-pages/). The version above works for most themes.

---

## Step 11: Create a .gitignore

Create a `.gitignore` file in your project root:

```
# Hugo build output
public/
resources/_gen/

# OS files
.DS_Store
Thumbs.db

# Hugo lock file
.hugo_build.lock
```

---

## Step 12: Commit and Push

Make sure your draft posts are set to `draft = false` for anything you want published, then:

```bash
git add .
git commit -m "Initial Hugo site"
git push -u origin main
```

---

## Step 13: Watch It Deploy

1. Go to your GitHub repo → **Actions** tab
2. You'll see a workflow run called "Build and deploy" in progress
3. Wait for the green checkmark ✅ (usually takes 1–2 minutes)
4. Click on the run → under the **deploy** job → find the link to your live site

Your site is now live at `https://YOUR-USERNAME.github.io/` 🎉

---

## Step 14 (Optional): Add a Custom Domain

If you've purchased a domain (e.g., `yourname.dev`):

1. In your GitHub repo → **Settings** → **Pages** → **Custom domain**, enter your domain
2. At your domain registrar, add DNS records:
   - For an **apex domain** (e.g., `yourname.dev`), add four `A` records pointing to GitHub's IPs:
     ```
     185.199.108.153
     185.199.109.153
     185.199.110.153
     185.199.111.153
     ```
   - For a **www subdomain**, add a `CNAME` record pointing to `YOUR-USERNAME.github.io`
3. Back in GitHub Pages settings, check **Enforce HTTPS**
4. Create a file called `static/CNAME` in your Hugo project containing just your domain:
   ```
   yourname.dev
   ```
5. Commit and push. DNS propagation can take up to 24 hours.

---

## Everyday Workflow

Once everything is set up, your daily workflow is simple:

```bash
# 1. Create a new post
hugo new content posts/new-post-title.md

# 2. Write in your editor, preview with:
hugo server -D

# 3. When ready, set draft = false, then:
git add .
git commit -m "Add new post about XYZ"
git push
```

GitHub Actions takes care of the rest — your site updates automatically within a couple of minutes.

---

## Quick Reference

| Task | Command |
|------|---------|
| Create new site | `hugo new site my-website` |
| Add a theme | `git submodule add <theme-url> themes/<name>` |
| Create a post | `hugo new content posts/my-post.md` |
| Local preview | `hugo server -D` |
| Build for production | `hugo` |
| Check Hugo version | `hugo version` |

---

## Useful Links

- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hugo Themes Gallery](https://themes.gohugo.io/)
- [Hugo Quick Start](https://gohugo.io/getting-started/quick-start/)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
- [Hugo on GitHub Pages (Official)](https://gohugo.io/host-and-deploy/host-on-github-pages/)
- [Hugo Community Forum](https://discourse.gohugo.io/)
