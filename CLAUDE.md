# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Digital Monetary Fund (DMF)** website — a Jekyll-based static site hosted on GitHub Pages at `www.digitalmonetary.fund`. DMF is a DAO operating AI-driven stablecoins covering 200+ ISO-standard currencies, deployed on Ethereum and Binance Smart Chain.

## Development Commands

```bash
# Install dependencies
bundle install

# Run local dev server (auto-rebuilds on changes)
bundle exec jekyll serve
# Site available at http://127.0.0.1:4000/

# Build the site without serving
bundle exec jekyll build
```

**Note:** `_config.yml` changes require a server restart — Jekyll does not hot-reload config.

## Architecture

- **Static site generator:** Jekyll 4.2 with the **Minima** theme (v2.5)
- **Hosting:** GitHub Pages, branch `0-Production`, custom domain via `CNAME`
- **Sass pipeline:** `assets/css/style.scss` imports Minima skin + `_sass/minima/initialize.scss`, which chains: `custom-variables.scss` -> `_base.scss` -> `_layout.scss` -> `custom-styles.scss`
- **Layouts:** `_layouts/` contains `default.html` (shell), `page.html`, `post.html`, `home.html`
- **Includes:** `_includes/` has `header.html` (with hardcoded logo URL), `footer.html` (with social links), `head.html`, `social.html`

## Content Structure

Pages use Markdown with YAML front matter (`layout`, `title`, `permalink`, `navigation_weight`). Navigation order is controlled by `header_pages` in `_config.yml`.

Key sections: `/` (home), `/about/`, `/dao/`, `/currencies/`, `/exchanges/`, `/news/`, `/contact`

The `coins/` directory hosts coin/token images and a standalone HTML page (not Jekyll-themed). The `docs/` directory is excluded from the Jekyll build.

## Key Configuration

- `_config.yml` — site metadata, plugins, nav order, color scheme, social links, and background images (`navbar-img`, `footer-img`, `page-img`)
- Plugins: `jekyll-feed`, `jekyll-seo-tag`, `jekyll-paginate`, `jekyll-sitemap`, `jekyll-github-metadata`
- The config has duplicate color blocks (lines ~167 and ~212) — the second block with background images overrides the first

## Deployment

Pushing to `0-Production` branch deploys to GitHub Pages automatically. The `CNAME` file maps to `www.digitalmonetary.fund`.
