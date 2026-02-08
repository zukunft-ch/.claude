# zukunft-ch — GitHub Organisation

## Overview
**Die Zukunft** is a Swiss political party. This org contains all repos for the website at `die-zukunft.ch`.

## Repos

### `die-zukunft.ch` — The Zola website
- **Stack**: Zola (Rust static site generator) + Tera templates
- **Content**: `content/*.md` — Markdown with TOML frontmatter (`+++...+++`)
- **Config**: `config.toml` — base URL, languages, translations, extra vars
- **Static**: `static/` — fonts, OG images, robots.txt, favicon (copied from theme)
- **Theme**: `themes/avenir` — symlink to `../avenir`
- **Build**: `zola build` → `public/`
- **Serve**: `zola serve`

### `avenir` — The Zola theme
- **Templates**: `templates/` — Tera HTML (base.html, page.html, index.html, etc.)
- **Macros**: `templates/macros/` — nav, footer, jsonld, pillar_cards
- **Shortcodes**: `templates/shortcodes/` — faq, callout, newsletter_form
- **Styles**: `sass/` — SCSS (main.scss imports _variables, _reset, _theme, _typography, _layout, _components)
- **Static**: `static/` — favicon.svg, fonts/
- **Config**: `theme.toml`

### `assets` — Brand assets (favicons + OG images)
- `favicons/favicon.svg` — 3D wireframe Swiss cross (SVG)
- `favicons/favicon.png` — 512x512 PNG render
- `og-images/og-{de,fr,it,en}.png` — 1200x630 per-language OG images
- Generated via Python script (perspective projection + rsvg-convert)

## Languages
- **DE** (default, served at `/`)
- **FR** (`/fr/`)
- **IT** (`/it/`)
- **EN** (`/en/`)

## Domain
`die-zukunft.ch` — deployed as static site.

## Brand
- **Tone**: Pragmatic, confident, Swiss-serious. No sci-fi, no culture-war, no climate framing.
- **Brandmark**: "Die Zukunft." / "Le Futur." / "Il Futuro." / "The Future." (always with period)
- **Visual identity**: 3D wireframe Swiss cross, purple→blue→cyan gradient
- **Fonts**: Clash Display (brandmark), Switzer (headlines), Mozilla Headline (body)
