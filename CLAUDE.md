# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
hugo server -D          # local dev server (includes draft posts)
hugo --gc --minify      # production build (output → public/)
```

`/public/` is gitignored — never commit it. Deploy happens automatically via Netlify on push to `main`.

**Netlify caveat**: `netlify.toml` pins `HUGO_VERSION = '0.118.2'` but the local install is 0.146. If a build works locally but fails on Netlify, the version gap is the likely cause.

## Architecture

This is a Hugo static site for a Peruvian labor lawyer (paulparedes.pe), built on the `robjhyndman` theme (a fork of hugo-finite).

**Customization layer** — only two files override the theme:
- `layouts/_default/baseof.html` — master template: SEO meta, Open Graph, Twitter Cards, canonical URL, GA4 (G-JW93WF08Z3), responsive Foundation navbar, MathJax (conditional on `mathjax: true` in front matter)
- `static/css/paulparedes-custom.css` — full visual identity override loaded last; defines all CSS variables

**CSS variables** (use these, not hex values directly):
```css
--pp-primary:    #1E3A5F   /* deep blue — headings, navbar, buttons */
--pp-accent:     #C49332   /* gold — hover states, icons, borders */
--pp-bg:         #F7F5F0   /* page background */
--pp-bg-alt:     #EDEAE4   /* card/blockquote backgrounds */
--pp-border:     #D4CFC6
--pp-text:       #2C2C2C
--pp-text-light: #5A5A5A
--pp-secondary:  #3D7A8A
--pp-contrast:   #9B3B2D
```

Font Awesome 4.6.3 and Academicons 1.7.0 are available via `fa fa-*` and `ai ai-*` classes. The global rule `.fa, .ai { color: var(--pp-primary) !important }` overrides icon colors — to use accent color on icons, add a more specific rule like `.my-class .fa { color: var(--pp-accent) !important }`.

## Content Types

| Type | Directory | Key front matter |
|---|---|---|
| Blog post | `content/blog/` | `date`, `categories`, `tags`, `slug` |
| Publication | `content/publications/` | `pubkind`, `citation`, `link` |
| Seminar | `content/seminars/` | `date`, `link` |
| Standalone page | `content/*.md` | requires `type: "page"` |

Standalone pages at the root of `content/` **must** include `type: "page"` in front matter so Hugo finds `layouts/page/single.html`. Without it the page is counted but not rendered (no output in `public/`).

Markdown renderer has `unsafe = true` — HTML can be embedded directly in `.md` files.

## Navigation Menu

Defined in `config.toml` as `[[menu.main]]` entries with `weight` values. Current weights: Blog=1, Publicaciones=2, Seminarios=4, Curso=6, Servicios=7, Sobre mí=8. Leave gaps to allow insertions.

## Theme Layouts

The theme provides section layouts (`blog/single.html`, `publications/single.html`, `seminars/single.html`) and `page/single.html` for standalone pages. All extend `baseof.html` via `{{ define "main" }}` blocks. To create a new section, add a layout under `layouts/<section>/single.html` that defines `"main"`.
