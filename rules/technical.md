# Technical Editing Rules

## Build & verify
Always run `zola build` after changes and confirm zero warnings. The build outputs to `public/`.

## Theme vs site
- **Site repo** (`die-zukunft.ch`): content, config, static assets (OG images)
- **Theme repo** (`avenir`): templates, styles, favicon, fonts, JS
- The theme is symlinked: `die-zukunft.ch/themes/avenir` → `../avenir`
- Run `zola build` from the `die-zukunft.ch` directory

## SCSS structure (`avenir/sass/`)
| File | Purpose |
|------|---------|
| `main.scss` | Entry point, CDN font imports, partial imports |
| `_variables.scss` | Colors, spacing, breakpoints, font families, z-indices |
| `_reset.scss` | CSS reset |
| `_theme.scss` | CSS custom properties for light/dark themes |
| `_typography.scss` | @font-face, heading styles, body text |
| `_layout.scss` | Scroll-snap, canvas sections, containers |
| `_components.scss` | All components (topnav, hero, sparkle, cards, forms, etc.) |

## Key CSS patterns
- **Glassmorphic**: `background: var(--glass-bg); backdrop-filter: blur($glass-blur); border: 1px solid var(--glass-border);`
- **Gradient text**: `@include gradient-text()` mixin
- **Canvas sections**: `.canvas` = full-viewport snap section with rounded corners and gradient background
- **Z-index layers**: `$z-sticky`, `$z-overlay` — topnav uses `$z-overlay + 2`

## Template structure (`avenir/templates/`)
- `base.html` — master layout: head, meta, nav, sparkle, progress tracker, to-top, Three.js, all shared JS
- `index.html` — landing page (extends base)
- `page.html` — generic content page (extends base)
- Specialized: `vision.html`, `programm.html`, `grundeinkommen.html`, `kontakt.html`, `mitmachen.html`, `info.html`
- `macros/nav.html` — topnav + sparkle button macros
- `macros/footer.html` — site footer
- `macros/jsonld.html` — structured data macros

## JavaScript (all in base.html)
All JS is vanilla, no build step. Key systems:
- **SwissCross.create()** — Three.js wireframe cross renderer with configurable `arm`, `thick`, `cameraZ`, `cubeSize`, `fov`, `camX`, `camY`, `opacity`
- **Sparkle overlay** — open/close with focus trap, search filtering, live region
- **Progress tracker** — IntersectionObserver on sections, glassmorphic dots
- **To-top button** — IntersectionObserver on first section
- **Topnav logo** — IntersectionObserver on hero (show when scrolled past)
- **Hero subline** — scroll-gesture detection with 800ms blocking window

## Gotchas
- `position: fixed` inside a `backdrop-filter` parent positions relative to the parent, not viewport. That's why sparkle button has two elements (desktop fixed + mobile inline).
- Tera `self::` macro references don't work in imported macros — inline the code.
- Three.js canvas needs non-zero dimensions at init time. Use `transform: scale(0)` instead of `width: 0` to hide elements that contain canvases.
- Fontshare CDN web fonts have `"false"` as family name. If using them locally (e.g., for OG image generation), patch with fonttools first.

## OG image generation
Requires: `rsvg-convert` (librsvg), Python 3, fonttools + brotli.
Fonts must be installed in `~/Library/Fonts/` with patched family names:
```bash
python3 -c "
from fontTools.ttLib import TTFont
f = TTFont('input.ttf')
for r in f['name'].names:
    if r.nameID == 1: r.string = 'Clash Display'
    if r.nameID == 4: r.string = 'Clash Display'
    if r.nameID == 6: r.string = 'ClashDisplay'
f.save('output.ttf')
"
```
Then generate: create SVG with wireframe lines from favicon.svg, add text, convert via `rsvg-convert -w 1200 -h 630`.

## Favicon generation
The favicon SVG is generated from the Three.js SwissCross projection (arm=1, thick=0, fov=35, cameraZ=7, rotation 0.4/0.6). Coordinates are computed via Python perspective projection matching the JS renderer. Source of truth is `avenir/static/favicon.svg`.

## Adding a new page
1. Create `content/page-name.md` with TOML frontmatter (set `template = "page.html"`)
2. Create `.fr.md`, `.it.md`, `.en.md` variants
3. Add nav link in `config.toml` translations if it should appear in navigation
4. Run `zola build` to verify

## Disabled features
- **Contact form**: Formspree endpoint is placeholder (`your-form-id`). Form inputs are `disabled`, button is `btn--disabled`. To enable: get Formspree ID, update `config.toml`, restore form in `kontakt.html`.
- **Newsletter**: Input disabled + "Coming Soon" button. To enable: integrate provider, update `shortcodes/newsletter_form.html` and `mitmachen.html`.
