# davidadrian.cc

Personal website and blog for David Adrián Cañones — CTO & Co-Founder at WhiteBox. Built with Hugo + Blowfish theme, deployed via GitHub Pages.

## Tech stack

- **Static site generator**: Hugo extended (v0.155+), installed via conda env `hugo_env`
- **Theme**: [Blowfish](https://github.com/nunocoracao/blowfish) as a git submodule at `themes/blowfish/`
- **Deployment**: GitHub Pages via GitHub Actions (`.github/workflows/gh-pages.yml`)
- **Domain**: `davidadrian.cc` (DNS at Mr. Domain)
- **Color scheme**: `ocean`

## Development

```bash
# Start local dev server (hot reload)
conda run -n hugo_env hugo server --port 1313

# Production build
conda run -n hugo_env hugo --minify

# Build including drafts (for local preview only)
conda run -n hugo_env hugo server -D
```

## Project structure

```
config/_default/          # Split Hugo config (4 files)
  hugo.toml               # Core settings, markup, outputs
  languages.en.toml       # Author profile, social links
  menus.en.toml           # Navigation menu
  params.toml             # Blowfish theme params

content/
  _index.md               # Homepage (profile layout)
  blog/                   # Blog posts as page bundles
    <slug>/
      index.md            # Post content (YAML front matter)
      feature.jpg/png     # Feature image (OG card + hero)
      *.png/jpg           # Other images, referenced relatively
  cv/index.md             # CV page
assets/
  css/custom.css          # Custom CSS overrides
  img/author.jpg          # Profile photo

layouts/
  partials/article-link/
    simple.html           # Overrides Blowfish list item to hide thumbnails globally

static/
  CNAME                   # davidadrian.cc (for GitHub Pages)
  favicon*, site.webmanifest, llms.txt

themes/blowfish/          # Git submodule — do NOT edit directly
```

## Content conventions

### Front matter (published post)
```yaml
---
title: "Post title"
date: 2021-05-31T12:00:00.000Z
lastmod: 2021-06-07T00:00:00.000Z
draft: false
description: "One-sentence SEO description."
tags:
  - "Tutorials"
featureimage: "feature.jpg"
---
```

### Front matter (draft post)
```yaml
draft: true
# No aliases needed for unpublished posts
```

### Images
- Feature image must be named `feature.jpg` or `feature.png` (Blowfish auto-detects)
- All images live alongside `index.md` in the page bundle
- Use descriptive filenames (e.g., `conda-activate.png`, not `image-3.png`)
- Image captions use Hugo's title syntax: `![alt text](image.png "Caption text")`
- Do NOT use the old `![alt](img)\n*caption*` pattern

### URL aliases
All published posts have an alias preserving the old Ghost URL:
```yaml
aliases:
  - /old-ghost-slug/
```

## Key decisions & gotchas

- **Search**: Blowfish uses a modal search overlay (magnifying glass icon). There is no standalone `/search/` page — do not create one.
- **Drafts are gitignored**: The drafts are excluded from the repo via `.gitignore`. Never commit it.
- **Theme submodule**: Run `git submodule update --init --recursive` after cloning.
- **Hugo extended required**: Blowfish uses SCSS. Always use the `extended` build.
- **`unsafe = true`**: Set in goldmark renderer to handle any remaining HTML in content.
- **Thumbnails hidden globally**: The `layouts/partials/article-link/simple.html` override removes thumbnails from all list views. Feature images still work for OG/social cards.
- **Custom CSS**: `assets/css/custom.css` centers `figure` and `figcaption` elements.
