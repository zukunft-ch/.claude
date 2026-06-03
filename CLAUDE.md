# zukunft-ch — GitHub Organisation

## Overview
**Die Zukunft** is a Swiss political party. This org contains all repos for the website at `die-zukunft.ch`.

## Repos

### `die-zukunft.ch` — The Zola website
- **Stack**: Zola 0.19.2 (Rust static site generator) + Tera templates
- **Content**: `content/*.md` — Markdown with TOML frontmatter (`+++...+++`)
- **Config**: `config.toml` — base URL, languages, translations, extra vars
- **Static**: `static/` — fonts, OG images, robots.txt, favicon, PWA icons, service worker
- **Theme**: `themes/avenir` — git submodule pointing to `github.com/zukunft-ch/avenir`
- **Worker**: `worker/` — Cloudflare Worker for multi-domain language proxy
- **Build**: `zola build` → `public/` (or `mise run build`)
- **Serve**: `zola serve` (or `mise run dev`)
- **Task runner**: mise (optional, see `mise.toml`)

### `avenir` — The Zola theme
- **Templates**: `templates/` — Tera HTML (base.html, page.html, index.html, etc.)
- **Macros**: `templates/macros/` — nav, footer, jsonld, pillar_cards
- **Shortcodes**: `templates/shortcodes/` — faq, callout, newsletter_form
- **Styles**: `sass/` — SCSS (main.scss imports _variables, _reset, _theme, _typography, _layout, _components)
- **Static**: `static/` — favicon.svg (3D wireframe Swiss cross), fonts/
- **Config**: `theme.toml`

### `assets` — Brand assets (favicons + OG images)
- `favicons/favicon.svg` — 3D wireframe Swiss cross (SVG)
- `favicons/favicon.png` — 512x512 PNG render
- `og-images/og-{de,fr,it,en}.png` — 1200x630 per-language OG images
- Generated via Python script (perspective projection + rsvg-convert)

## Languages & Domains
- **DE** (default) — `die-zukunft.ch/` (served directly from Cloudflare Pages)
- **FR** (`/fr/`) — `le-futur.ch` (proxied via Cloudflare Worker)
- **IT** (`/it/`) — `il-futuro.ch` (proxied via Cloudflare Worker)
- **EN** (`/en/`) — `die-zukunft.ch/en/` (no separate domain)
- **Worldwide movement** — `the-future.world` (separate static app)
- **Verein (association)** — `collective.die-zukunft.ch` (separate static app)

## Deployment — Cloudflare Pages + Workers

### Architecture
```
Browser → Cloudflare DNS → Worker (lang proxy) → Pages origin → static files
```

1. **Cloudflare Pages** hosts the built Zola site at `die-zukunft-ch.pages.dev`
2. **Cloudflare Worker** (`worker/src/worker.js`, name: `lang-proxy`) handles:
   - Multi-domain routing: `le-futur.ch` → `/fr/`, `il-futuro.ch` → `/it/`
   - `www` → apex redirects (301)
   - Origin masking: fetches from `die-zukunft-ch.pages.dev` to avoid 522 errors
   - URL rewriting in HTML/XML responses (rewrites absolute URLs, hreflang, sitemaps)
   - Dynamic `robots.txt` with per-domain Sitemap URL
   - Handles Zola's minified unquoted HTML attributes

### Worker routes (`worker/wrangler.toml`)
- `le-futur.ch/*` → zone: le-futur.ch
- `www.le-futur.ch/*` → zone: le-futur.ch
- `il-futuro.ch/*` → zone: il-futuro.ch
- `www.il-futuro.ch/*` → zone: il-futuro.ch

The main domain `die-zukunft.ch` is served directly by Pages (no Worker).

### Static headers & redirects
- `static/_headers` — security headers (CSP, HSTS, X-Frame-Options, Permissions-Policy)
- `static/_redirects` — `www.die-zukunft.ch` → `die-zukunft.ch` (301)
- CSP allows: `api.fontshare.com` (font CSS), `cdn.fontshare.com` (font files), `formspree.io` (contact form)
- **HTTPS redirect**: Enable "Always Use HTTPS" in Cloudflare dashboard for each zone (die-zukunft.ch, le-futur.ch, il-futuro.ch)

### DNS records (Cloudflare-managed)
All three domains (`die-zukunft.ch`, `le-futur.ch`, `il-futuro.ch`) are on Cloudflare DNS. Pages custom domain is `die-zukunft.ch`. Worker routes handle `le-futur.ch` and `il-futuro.ch`.

### Deploying
- Push to `main` triggers Cloudflare Pages build (`zola build`)
- Worker changes: `cd worker && npx wrangler deploy`

## PWA
The site is installable as a Progressive Web App:
- `static/site.webmanifest` — app manifest with 192/512 icons + maskable icon
- `static/sw.js` — service worker (network-first with offline cache fallback)
- `static/icon-192.png`, `static/icon-512.png` — PWA icons
- iOS meta tags: `apple-mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style`
- Service worker registered in `base.html`

## Hex Address System
Every page section gets a unique hex anchor (`0xPPSS`) displayed in the top-left corner. `PP` = page byte, `SS` = section byte. The mapping lives in `base.html` (JS `hexMap`). The `data-hex-page` attribute on `<main>` is derived from the content file's base name (e.g. `programm.fr.md` → `programm`).

### Address map

| Range | Category | Pages |
|---|---|---|
| `0x00__`–`0x0F__` | Core nav | `0x00` Home, `0x01` Vision, `0x02` Programm, `0x03` Mitmachen |
| `0x10__`–`0x1F__` | Framing | `0x10` Manifest, `0x11` Kein links/rechts |
| `0x20__`–`0x2F__` | Pillars | `0x20` Digitale Souveränität, `0x21` Robotik, `0x22` Übergänge, `0x23` Life Science, `0x24` Grundeinkommen, `0x25` Städte & Wohnen, `0x26` Research & Innovation, `0x27` Energy & Resilience, `0x28` Offener Staat, `0x29` Neutralität & Frieden, `0x2A` Talent & Integration, `0x2B` Sprache & Verständigung |
| `0x30__`–`0x3F__` | Zukunftslabor | `0x30` Zukunftslabor, `0x31` Notizen, `0x32` Methodik |
| `0x40__`–`0x4F__` | People & community | `0x40` Menschen, `0x41` Kontakt, `0x42` Abstimmungen |
| `0x50__`–`0x5F__` | Aktuelles articles | `0x50` Platz 1 ist kein Plan, `0x51` Abstimmung 14. Juni 2026 |
| `0xF0__`–`0xFF__` | Legal & meta | `0xF0` Impressum, `0xF1` Datenschutz, `0xF2` Info, `0xF3` Aktuelles (index) |

**Section-level meta slots**: within any page, `0xPPF0`–`0xPPFF` is reserved for cross-cutting auxiliary sections (banners, overlays, ephemera) so adding/removing one does not shift the addresses of the page's primary sections. Mark such sections with a `data-hex="0xPPFn"` attribute; the hex-anchor JS skips them in the auto-counter so primary sections keep stable addresses.

**Stability rule**: once assigned, a page or section hex address must not change. New pages take the next free index in the appropriate range; if a page is removed, leave its slot retired (don't reuse).

**Adding a new page**: Append to the appropriate category in `hexMap` (next free index). Each page gets 256 section slots.

## Brand
- **Tone**: Pragmatic, confident, Swiss-serious. No sci-fi, no culture-war, no climate framing.
- **Brandmark**: "Die Zukunft." / "Le Futur." / "Il Futuro." / "The Future." (always with period)
- **Visual identity**: 3D wireframe Swiss cross, purple→blue→cyan gradient
- **Fonts**: Clash Display (brandmark), Switzer (headlines) — loaded via `<link>` tags from Fontshare CDN (NOT CSS `@import`)
- **Brand colors**: Purple `#a855f7`, Blue `#3b82f6`, Teal `#06b6d4`
