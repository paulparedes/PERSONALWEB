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
- `layouts/_default/baseof.html` — master template: SEO meta, Open Graph, Twitter Cards, og:image/twitter:image, canonical URL, GA4 (G-JW93WF08Z3), JSON-LD schemas (Person global; BlogPosting, ScholarlyArticle, conditional by section), responsive Foundation navbar, MathJax (conditional on `mathjax: true` in front matter)
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

## JSON-LD Schemas

Three schemas live in `baseof.html`; one is inline in content:

| Schema | Location | Condition |
|---|---|---|
| `Person` | `baseof.html` | all pages |
| `BlogPosting` | `baseof.html` | `eq .Section "blog"` |
| `ScholarlyArticle` | `baseof.html` | `eq .Section "publications"` |
| `LegalService` | `content/servicios.md` | hardcoded inline |

**Critical gotchas** — do not skip these:

1. **`<script>` JS context double-escaping**: Hugo's `html/template` treats `<script>` content as JavaScript. Inline template vars like `{{ .Title | jsonify }}` get double-escaped. Always build schemas as a `dict` and output with `jsonify | safeJS`:
   ```go-html-template
   {{ $s := dict "@type" "BlogPosting" "headline" .Title ... }}
   <script type="application/ld+json">{{ $s | jsonify | safeJS }}</script>
   ```

2. **`og:image` type guard**: some posts have `image` in front matter as a map/struct (not a string). Using `.Params.image` directly renders as `map[caption: ...]`. Always guard with:
   ```go-html-template
   {{ $ogImage := "img/pgpp.png" }}
   {{ if and .Params.image (eq (printf "%T" .Params.image) "string") }}{{ $ogImage = .Params.image }}{{ end }}
   ```
   Note: `kindIs` does not exist in Hugo — use `printf "%T"`.

3. **`@id` for author disambiguation**: the `Person` block uses `"@id": "https://paulparedes.pe/#person"`. The `author` object in `BlogPosting` and `ScholarlyArticle` repeats this same `@id` so Google links them as the same entity across pages.

## Theme Layouts

The theme provides section layouts (`blog/single.html`, `publications/single.html`, `seminars/single.html`) and `page/single.html` for standalone pages. All extend `baseof.html` via `{{ define "main" }}` blocks. To create a new section, add a layout under `layouts/<section>/single.html` that defines `"main"`.
