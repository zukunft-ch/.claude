# Technical Editing Rules

## Build & verify
Always run `zola build` after changes and confirm zero warnings. The build outputs to `public/`.

## Theme vs site
- **Site repo** (`die-zukunft.ch`): content, config, static assets (OG images, PWA icons, service worker)
- **Theme repo** (`avenir`): templates, styles, favicon, fonts, JS
- The theme is a git submodule: `die-zukunft.ch/themes/avenir` → `github.com/zukunft-ch/avenir`
- A standalone clone also exists at `../avenir` — keep both in sync after changes
- Run `zola build` from the `die-zukunft.ch` directory
- **Always edit under `themes/avenir/`** for Zola to pick up changes; edits to the standalone `../avenir/` are NOT seen by `zola build`

## Syncing theme repos
After editing theme files, always sync both copies:
```bash
# If you edited themes/avenir/ (submodule):
cp themes/avenir/path/to/file ../avenir/path/to/file

# Commit submodule first, push, then pull in standalone:
cd themes/avenir && git add ... && git commit && git push
cd ../../avenir && git checkout -- . && git pull
```

## SCSS structure (`avenir/sass/`)
| File | Purpose |
|------|---------|
| `main.scss` | Entry point, CDN font imports, partial imports |
| `_variables.scss` | Colors, spacing, breakpoints, font families, z-indices |
| `_reset.scss` | CSS reset |
| `_theme.scss` | CSS custom properties for light/dark themes, canvas gradient + tint vars |
| `_typography.scss` | @font-face, heading styles, body text |
| `_layout.scss` | Scroll-snap, canvas sections, hex anchors, containers, plasma layer |
| `_components.scss` | All components (topnav, hero, sparkle, cards, carousel, forms, etc.) |

## Key CSS patterns
- **Glassmorphic**: `background: var(--glass-bg); backdrop-filter: blur($glass-blur); border: 1px solid var(--glass-border);`
- **Gradient text**: `@include gradient-text()` mixin
- **Canvas sections**: `.canvas` = full-viewport snap section with rounded corners and gradient/tint background
- **Canvas layout**: `flex-direction: column` with `::before`/`::after` spacers (`flex: 1`) — centers content vertically while preventing upward overflow into hex anchor zone
- **Z-index layers**: `$z-sticky`, `$z-overlay` — topnav uses `$z-overlay + 2`
- **Theme default**: Light (not system preference). Dark mode only activates when user explicitly toggles it.

## Template structure (`avenir/templates/`)
- `base.html` — master layout: head (SwissCross Canvas 2D renderer + Fontshare font links), meta, nav, plasma WebGL, sparkle, progress tracker, to-top, service worker registration, all shared JS
- `index.html` — landing page (extends base): hero, subtitle, carousel, changes, timeline, CTA
- `page.html` — generic content page (extends base)
- Specialized: `vision.html`, `programm.html`, `grundeinkommen.html`, `kontakt.html`, `mitmachen.html`, `info.html`
- `macros/nav.html` — topnav + sparkle button macros
- `macros/footer.html` — site footer
- `macros/jsonld.html` — structured data macros
- `macros/pillar_cards.html` — pillar card rendering

## JavaScript (all in base.html, vanilla, no build step)

### Plasma background (WebGL)
- Single offscreen WebGL canvas renders animated plasma, drawn into per-section 2D canvases
- 4 color presets per theme cycle across sections (light: pastel, dark: rich)
- Shader uniforms: `u_scale` (color range), `u_lift` (brightness floor), `u_saturation`
- Speed: `u_time * 0.08` (~78s per cycle)
- On `body.plasma-active`, sections use semi-transparent `--canvas-tint-*` gradients
- Graceful degradation: no-JS or no-WebGL falls back to opaque CSS gradients
- Respects `prefers-reduced-motion: reduce`
- `powerPreference: 'low-power'`, DPR capped at 1.5

### SwissCross.create() (Canvas 2D — no WebGL)
- Pure Canvas 2D wireframe cross renderer (replaced Three.js in Feb 2026)
- Configurable: `arm`, `thick`, `cameraZ`, `cubeSize`, `fov`, `camX`, `camY`, `opacity`
- Used in hero (large), topnav logo (small), footer (medium) — all devices including mobile
- No WebGL context limits — runs everywhere with 2D canvas
- Defined in `<head>` of `base.html` so it's available before content block scripts
- Perspective projection: `buildPositions()` → cube edge pairs → `rotatePoint()` → `project()` → `ctx.lineTo()`
- Theme-aware: MutationObserver updates color/opacity on `data-theme` change
- IntersectionObserver pauses animation when offscreen
- No external JS dependencies (Three.js CDN removed)

### Other JS systems
- **Sparkle overlay** — open/close with focus trap, search filtering, live region
- **Progress tracker** — IntersectionObserver on sections, glassmorphic dots
- **To-top button** — IntersectionObserver on first section
- **Topnav logo** — IntersectionObserver on hero (show when scrolled past)
- **Hero subline** — scroll-down gesture triggers reveal, scroll locked via `overflow: hidden` + `scroll-snap-type: none` for 800ms during animation (works on iOS Safari)
- **Pillar carousel** — continuous auto-scroll, swipeable by touch/mouse drag, duplicate cards for seamless loop
- **Theme toggle** — removes and re-inserts `<meta name="theme-color">` to force Safari to update browser chrome color

## Gotchas
- **Zola HTML minification** (`minify_html = true`): Zola's minifier aggressively minifies inline JS — renames variables, converts `'\n'` to template literals with literal `\\n` (breaking GLSL shader sources), and may break `//` comments in embedded languages. **Fix**: write GLSL shaders as single-line strings, avoid `array.join('\n')`, avoid `//` comments in shader source.
- **Theme submodule**: `themes/avenir/` is a git submodule with its own working tree — edits to `../avenir/` (standalone clone) are NOT picked up by `zola build`. Always edit files under `themes/avenir/` for Zola to use them.
- `position: fixed` inside a `backdrop-filter` parent positions relative to the parent, not viewport. That's why sparkle button has two elements (desktop fixed + mobile inline).
- Tera `self::` macro references don't work in imported macros — inline the code.
- **Script ordering in base.html**: `SwissCross` must be defined in `<head>` — content block scripts (e.g. index.html hero init) execute before body-end scripts.
- **CSS @import for fonts**: Don't use `@import url(...)` in SCSS for CDN fonts — iOS Safari + service workers can cause double round-trip failures. Use `<link rel="stylesheet">` tags in `<head>` instead.
- **CSP for fonts**: `font-src` and `style-src` must include Fontshare CDN origins (`api.fontshare.com`, `cdn.fontshare.com`) or fonts silently fail.
- Fontshare CDN web fonts have `"false"` as family name locally. If using them for OG image generation, patch with fonttools first.
- **Scroll-snap**: Use `y proximity` (not `mandatory`) + `scroll-behavior: smooth` for natural feel. `mandatory` causes jarring instant snaps on iOS.
- **Canvas sections height**: Use `min-height` (not `height`) so tall sections like footer can exceed viewport without scroll-snap fighting.
- **iOS Safari theme-color**: `setAttribute('content', ...)` on `<meta name="theme-color">` does NOT update Safari browser chrome. Must remove and re-insert the element.
- **iOS Safari scroll-snap + preventDefault**: `e.preventDefault()` on `touchmove` does not reliably block scroll-snap. Use `overflow: hidden` + `scroll-snap-type: none` on `<html>` instead.

## Cloudflare Worker (`worker/`)
- **Name**: `lang-proxy`
- **Config**: `worker/wrangler.toml`
- **Source**: `worker/src/worker.js`
- **Deploy**: `cd worker && npx wrangler deploy`
- Routes `le-futur.ch` → `/fr/` and `il-futuro.ch` → `/it/` with full URL rewriting
- Origin: `die-zukunft-ch.pages.dev` (Cloudflare Pages)
- Handles unquoted HTML attributes from Zola minification in URL rewrites

## Static files for Cloudflare Pages
- `static/_headers` — CSP, HSTS (2 years), X-Frame-Options: DENY, Permissions-Policy
- `static/_redirects` — `www.die-zukunft.ch/* → die-zukunft.ch/:splat 301`
- `static/robots.txt` — static fallback (Worker generates dynamic per-domain version)

## PWA files (`static/`)
- `site.webmanifest` — `display: standalone`, icons (SVG any, 192 PNG, 512 PNG, 512 maskable)
- `sw.js` — service worker: caches shell on install, network-first fetch with cache fallback
- `icon-192.png`, `icon-512.png` — PWA icons (scaled from apple-touch-icon)
- Cache name: `dz-v1` — bump version to invalidate

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
The favicon SVG is generated from the SwissCross projection (arm=1, thick=0, fov=35, cameraZ=7, rotation 0.4/0.6). Coordinates are computed via Python perspective projection matching the Canvas 2D renderer. Source of truth is `avenir/static/favicon.svg`. The site's `static/` should NOT contain a `favicon.svg` override — it falls through to the theme's version.

## Adding a new page
1. Create `content/page-name.md` with TOML frontmatter (set `template = "page.html"`)
2. Create `.fr.md`, `.it.md`, `.en.md` variants
3. Add nav link in `config.toml` translations if it should appear in navigation
4. Add to search index in `macros/nav.html`
5. Add to `hexMap` in `base.html` JS
6. Run `zola build` to verify

## Disabled features
- **Contact form**: Formspree endpoint is placeholder (`your-form-id`). Form inputs are `disabled`, button is `btn--disabled`. To enable: get Formspree ID, update `config.toml`, restore form in `kontakt.html`.
- **Newsletter**: Input disabled + "Coming Soon" button. To enable: integrate provider, update `shortcodes/newsletter_form.html` and `mitmachen.html`.
