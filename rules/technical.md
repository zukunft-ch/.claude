# Technical Editing Rules

## Build & verify
Always run `mise exec -- zola build` (pinned Zola 0.22.1) after changes and confirm zero warnings. The build outputs to `public/`. A bare `zola` resolves to Homebrew 0.23.x, which fails on this theme (Tera v2: `Unknown tag` on `{% import %}`).

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
- `base.html` — master layout: head (SwissCross Canvas 2D renderer + self-hosted font preloads), meta, nav, plasma WebGL, sparkle, progress tracker, to-top, service worker registration, all shared JS
- `index.html` — landing page (extends base): hero, subtitle, changes (intro + 4 bento cards + CTA), carousel, timeline, CTA
- `page.html` — generic content page (extends base)
- Specialized: `vision.html`, `programm.html`, `grundeinkommen.html`, `kontakt.html`, `mitmachen.html`, `info.html`
- `macros/nav.html` — topnav + sparkle button macros
- `macros/footer.html` — site footer (Verein, Spenden, The Future worldwide, movement pill CTA)
- `macros/jsonld.html` — structured data macros
- `macros/pillar_cards.html` — pillar card rendering

## JavaScript (all in base.html, vanilla, no build step)

### Plasma background (WebGL)
- Single offscreen WebGL canvas renders animated plasma, drawn into per-section 2D canvases
- 4 color presets per theme cycle across sections (light: pastel, dark: rich)
- Shader uniforms: `u_scale` (color range), `u_lift` (brightness floor), `u_saturation`
- Speed: `u_time * 0.12` (~52s per cycle)
- On `body.plasma-active`, sections use `overflow: clip` (NOT `hidden` — `hidden` clips content in taller-than-viewport sections)
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
- **Fonts are self-hosted** (woff2 in `static/fonts/`, `@font-face` in `_typography.scss`) — no third-party CDN, so `font-src 'self'` + `style-src 'self'` suffice. Preload the above-the-fold faces (Mozilla Headline, Switzer, Clash Display 700) in `base.html`. The three Fontshare-origin files (Switzer variable, Clash Display 600/700) are ITF Free Font License, self-hosting permitted.
- **OG image generation** needs TTF/OTF (not the site's woff2): `ClashDisplay-Bold.ttf` + `Switzer-Regular.ttf` are installed in `~/Library/Fonts/`. (If you instead pull raw Fontshare web fonts, they carry `"false"` as the family name — patch with fonttools first.)
- **Scroll-snap**: Use `y proximity` (not `mandatory`) + `scroll-behavior: smooth` for natural feel. `mandatory` causes jarring instant snaps on iOS.
- **Canvas sections height**: Use `min-height` (not `height`) so tall sections like footer can exceed viewport without scroll-snap fighting.
- **iOS Safari theme-color**: `setAttribute('content', ...)` on `<meta name="theme-color">` does NOT update Safari browser chrome. Must remove and re-insert the element.
- **iOS Safari scroll-snap + preventDefault**: `e.preventDefault()` on `touchmove` does not reliably block scroll-snap. Use `overflow: hidden` + `scroll-snap-type: none` on `<html>` instead.
- **Plasma overflow**: Use `overflow: clip` (not `hidden`) on `body.plasma-active .canvas` — `hidden` clips content in sections with 2+ rows of cards that exceed viewport height.
- **Bento grid**: 2x2 layout (`repeat(2, 1fr)`), stacks to 1-col on mobile (`$bp-sm`). No first-child span.
- **SCSS radius variables**: `$radius-sm` (6px), `$radius-md` (12px), `$radius-lg` (20px), `$radius-pill` (100px). There is NO `$radius-full`.

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
- Cache name: `dz-v2` — bump version to invalidate

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
The favicon SVG is generated to match the **hero** logo exactly: the hero is `SwissCross.create(... { cameraZ: 22, cubeSize: 0.88, fov: 35 })` in `index.html` — no `arm`/`thick`, so it uses `buildPositions` defaults `arm=3, thick=1` (the full chunky 3D lattice). (Note: the `cameraZ: 16` call in `base.html` is the **footer** mesh, and `cameraZ: 7, arm:1, thick:0` is the **navbar** logo — don't confuse them with the hero.)

The favicon uses the hero's exact geometry + camera + initial rotation (0.4/0.6), but with **occlusion** (hidden-line removal): interior faces shared between adjacent cubes are culled, back-facing faces are culled, and remaining faces are painter-sorted and drawn as white-filled polygons with gradient-stroked edges. This turns the hero's see-through wireframe tangle (1600+ overlapping edges → an illegible blob at small sizes) into a clean, solid 3D chunky Swiss cross (~99 visible faces) that stays legible down to 32px. A plain wireframe projection (every cube edge, no occlusion) does NOT work at favicon sizes — it must be occluded.

**Tuning workflow**: `assets/scripts/favicon_studio.html` is an interactive tool (open via `python3 -m http.server` in `assets/scripts/`) with live sliders for every dimension — arm, thick, cubeSize, cameraZ, fov, rotX/Y, line width, line color, surface color + opacity (translucent/glass), shading, mode (hidden-line / glass / solid / pure-wireframe) — drag-to-rotate, live 16/32/64px + browser-tab previews, and "Copy params". The studio and the Python generator share the exact same renderer, so a params line copied from the studio reproduces the SVG 1:1.

Generator: `assets/scripts/gen_favicon.py "<params line>"` (no args = the shipped defaults baked into the script). The exact params of the shipped favicon are recorded in `assets/favicons/favicon.params` (regenerate with `python3 gen_favicon.py "$(cat assets/favicons/favicon.params)"`). Current shipped params: `mode=hidden arm=4 thick=1 cubeSize=1 cameraZ=32 fov=60 rotX=1.01 rotY=1.08 stroke=0.3 fill=#ffffff fillOpacity=0.7` (longer-armed, head-on angle, thin gradient lines, translucent-white occluding surfaces). Source of truth is `avenir/static/favicon.svg`; also synced to `themes/avenir/static/favicon.svg` (submodule, what Zola reads) and `assets/favicons/favicon.{svg,png}`. The multi-size (16/32/48) `favicon.ico` fallback lives in the site's `static/`. The site's `static/` should NOT contain a `favicon.svg` override — it falls through to the theme's version.

NB: the favicon is the **wireframe** mark. The PWA/iOS icons (`apple-touch-icon.png`, `icon-192.png`, `icon-512.png`) are intentionally a **solid flat** Swiss cross — a different design language for app tiles — and are NOT regenerated when the wireframe favicon changes.

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

## Revamp (Nov 2026, branch `revamp-2026-11` in site + avenir) — supersedes older notes above
- **Retired**: per-section plasma WebGL, pastel `.canvas` gradients, JS magnetic scroll-snap, arrow-key section hijack (key-hints), scroll-locked hero subline, pillar carousel, standalone sparkle button, side dot progress tracker (replaced by `#read-progress`, a 2px ink line under the top bar on long pages). Docs above that describe them are historical.
- **Ground**: one fixed `#ground-mesh` SwissCross (hero geometry, gradient stroke, `scrollSpin`, `frame()` for camera per viewport) behind every page. JS sets `--ground-p` (0 on home hero → 1) so the lattice drifts right and recedes behind text; sub pages sit at 1.
- **Tokens** (`_theme.scss`): ceramic ground `#eef0ec` / charcoal ink; brand gradient = signal only (brandmark, lattice, primary buttons); Swiss red `--color-signal` = the single accent (pins, "live/now" markers). New partials: `_system.scss` (layout/nav/sec-tags/pins/colophon, loaded after `_components`), `_home.scss`, `_manifest.scss`.
- **Top bar** (`macros/nav.html`): brand · 5 links (≥992px) · Pins · Mitmachen · menu (`[data-menu-toggle]` opens the command overlay).
- **Pins**: `window.DZPins` in `base.html`; localStorage key `dz-pins-v1`, pin id = pathname + `#` + hex. Any `[data-pin]` button (macro `macros/pin.html`) works; every hex section tag gets one automatically (opt out with `data-no-pin`). Shareable sets via `/#pins=<base64url JSON>`. Pins also listed in the command overlay.
- **Homepage** = layers ("Ebenen") with explicit `data-hex` 0x0000–0x0006 (addresses changed from the old slide layout in Nov 2026). Theses are read from the manifest page (`get_page`) — single source.
- **Manifest** uses `manifest.html`; every part has a permanent hex in content (`0x1001` preamble, `0x1002–0x100B` theses, `0x100C` commitments, `0x100D` "not", `0x100E` signature/changelog). Bump `version`/`version_date` and add a `changelog` entry when the text changes.
- **Colophon** in footer measures page weight live (Performance API) and links the public source.
- **Vision** uses `vision.html` (Nov 2026): the argument as a stack — explicit addresses 0x0100 hero, 0x0101 shift (sourced figures), 0x0102 Schweiz 2040, 0x0103 Stack der Abundance, 0x0104 drei Sektoren, 0x0105 Leitplanken, 0x0106 Warum die Schweiz, 0x0107 Warum wir, 0x0108 Messgrössen, 0x0109 Quellen. Every figure on the page carries a source; keep `sources` in sync.
- **Brandtype**: mixin `brandtype($tracking)` in `_system.scss` (gradient fitted to glyphs, optical tracking) + `<span class="brand-dot">.</span>` = the full stop in Swiss red. Use it for every brand/brandmark title.
- **Brand icon (Nov 2026)**: "Würfelkreuz, klar" — `gen_favicon.py` DEFAULTS = `assets/favicons/favicon.params` (arm=1 thick=0, hidden-line, stroke 0.9, Alpenglühen). This supersedes the arm=4 params described in "Favicon generation" above. App icons (apple-touch/192/512), the GitHub org avatar, per-repo social previews and the per-language OG cards are all rendered from it by `assets/scripts/render_brand.mjs` (Playwright, real fonts). PWA/app icons are now this mark on white, no longer the solid flat cross.

## Machine-readable surface / GEO (Nov 2026)
- `scripts/build_llms.py` (run by `mise run build` via the `llms` task; Python 3.13 pinned in mise.toml) generates from content/: `static/llms.txt` (curated EN manifest: summary, how-to-route, 13 pillars, 10 theses, key terms, latest, citation/licence), `static/llms-full.txt` (EN corpus), `static/llms-full-de.txt` (DE corpus), and a Markdown twin of every page under `static/md/` (DE), `static/{en,fr,it}/md/`. Served as die-zukunft.ch/md/<path>.md, die-zukunft.ch/en/md/…, le-futur.ch/md/…, il-futuro.ch/md/… (worker prefixes non-static paths). **Generated files are committed** because Cloudflare Pages only runs `zola build` — always run `mise run build` (or `mise run llms`) before committing content changes.
- Every page links its twin (`<link rel=alternate type=text/markdown>`), `llms.txt` and `rel=license`. `_headers` marks twins + full corpora `X-Robots-Tag: noindex` (AI-readable, no duplicate content in search).
- Policy (user decision): robots fully open (`Content-Signal: search=yes, ai-input=yes, ai-train=yes`), texts CC BY 4.0 (Impressum section "Urheberrecht und Lizenz", 4 languages; name/wordmark/icon excluded). In the Cloudflare dashboard (account cf608a0a…, all three zones) two settings must stay as set on 2026-11-08: **AI Crawl Control → Signals → "Bot Preference Sync" OFF** (it prepends a managed robots.txt with ai-train=no), and **Security → Settings → Bot traffic → "AI Crawler and Agent access policies": Search, Training, Agents all "Allow"** (Training was "Disallow", which hard-blocked ~15 crawlers incl. GPTBot, ClaudeBot, CCBot at the edge). le-futur/il-futuro robots come from the worker.
- JSON-LD: PoliticalParty (slogan, knowsAbout, sameAs), WebSite/WebPage with licence, NewsArticle for Aktuelles posts, FAQPage on pillars. Feeds: atom.xml + rss.xml per language (worker serves per-language feeds on FR/IT domains). `/.well-known/security.txt` (expires 2027-11-07 — renew).
