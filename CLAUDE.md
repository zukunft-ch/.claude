# Die Zukunft / Le Futur / Il Futuro — Website Build Context (Claude Code)
> Domain: die-zukunft.ch
> Project: Swiss political party website (Basel-based launch)
> Goal: Credible, calm, Swiss-pragmatic site explaining the party's purpose + pillars, with multilingual support.

## 0) North Star
Build a **trustworthy Swiss political party website** that frames AI/robotics/life-science as an imminent structural shift and proposes an **upgrade of the social contract**:
- reduce fear of job loss via **Basic Income**
- build **digital sovereignty** via Swiss/EU-first procurement and local tech
- support **re-industrialization via robotics** (SME-friendly)
- present life-science modernization as jobs/resilience/health
Avoid sci-fi vibes. Avoid culture-war tone. Make it feel "Swiss, serious, measurable".

## 1) Brand / Tone
- Tone: pragmatic, confident, future-oriented, non-hysterical.
- Visual: minimal, modern, Swiss typography, lots of whitespace, strong hierarchy.
- No "cyberpunk", no neon. No gimmicks.
- Avoid climate-change messaging entirely (user request). Sustainability can mean: resilience, long-term stability, fiscal and social durability.

## 2) Languages (must-have)
- DE / FR / IT.
- Canonical structure:
  - /de/…  (default can be /de/)
  - /fr/…
  - /it/…
- Language switcher always visible (desktop + mobile).
- Content should be equivalent across languages; allow small localization where phrasing differs.

## 3) Information architecture (IA)
### Core pages
1) Home
2) Vision (Why the party exists)
3) Program / Pillars (policy themes)
4) Basic Income (UBI) — explainer + FAQ
5) Digital Sovereignty — explainer + proposals
6) People (founder + board placeholders)
7) Join / Support (newsletter + membership interest + volunteer)
8) News / Updates (optional v1; can be a simple markdown blog)
9) Contact / Press
10) Legal: Impressum, Datenschutz (privacy)

### Home page sections (in order)
- Hero: Party name + one-sentence positioning + 2 CTAs (Join / Read program)
- "What's changing" (AI/robotics/biotech; short, non-alarmist)
- "What we propose" (3–5 pillars cards)
- "How we start" (Basel focus, then national scaling)
- "Get involved" (newsletter, membership interest, volunteer)
- Footer with language links + contact + legal

## 4) Content requirements (key messages)
### One-sentence positioning (base idea; adapt per language)
"A Swiss party upgrading the social contract for the AI/robotics/biotech era: security without stagnation, innovation with dignity, sovereignty by design."

### Pillars (must be present)
1) Social contract for automation (Basic Income)
2) Work without fear (transition, reskilling, mobility)
3) Digital sovereignty (Swiss/EU-first, open standards, anti lock-in)
4) Re-industrialize with robotics (SMEs, shared labs, incentives)
5) Biotech modernization with trust (consent, auditability, no black-box state medicine)

### What to avoid in wording
- No "replace democracy with engineers"
- No "free money" framing; present UBI as simplification + stability + flexibility
- No climate framing

## 5) Functional requirements
- Fast load (Lighthouse focus)
- Fully responsive (mobile-first)
- Accessible (WCAG basics: contrast, focus states, semantic HTML)
- SEO:
  - proper titles/meta descriptions per language/page
  - OpenGraph + Twitter cards
  - sitemap.xml per language
  - robots.txt
  - canonical URLs
- Analytics: privacy-friendly (optional; propose plausible.io or self-hosted minimal; do not add heavy trackers by default)
- Contact:
  - simple contact form OR mailto (form preferred if easy)
  - anti-spam (honeypot + rate limit)
- Newsletter signup:
  - embed-friendly (Mailchimp/Buttondown/etc) OR simple "interest form" storing email
  - If storing emails: minimal data, clear consent text, simple export.

## 6) Tech stack preference (choose one and implement cleanly)
Pick a static-first approach with strong i18n:
- Option A: **Astro + MDX** (recommended for speed + content)
- Option B: **Next.js** (if you want React-heavy; otherwise overkill)
- Option C: **Zola** (if you want Rusty static generator)
Implement whichever is fastest for Claude to deliver with clean i18n routing.

### Content source
- Store page content as markdown/MDX per language:
  - content/de/*.md
  - content/fr/*.md
  - content/it/*.md

## 7) Design system components (must-have)
- Header with:
  - logo/wordmark (text-based for now)
  - nav links
  - language switcher
- Hero block
- Pillar cards
- Callout blocks (for proposals)
- FAQ accordion (UBI page)
- Simple timeline/roadmap block (Basel → national)
- Footer with legal + contact

## 8) Copy blocks (starter copy in EN; translate to DE/FR/IT)
Use these as source meaning; implement localized versions:

### Home hero (meaning)
Title: "Die Zukunft / Le Futur / Il Futuro"
Subtitle: "Switzerland is entering a structural transition. We make it governable."
CTAs: "Join" + "Read the program"

### Vision (meaning)
AI, robotics, and life sciences will reshape productivity, work, and healthcare faster than current politics can absorb. We refuse fear and blind acceleration. We propose a calm Swiss upgrade: basic income as stability, transition support as normal, and sovereignty by design.

### UBI page (meaning)
UBI is the automation dividend paid to society: fewer bureaucratic traps, more freedom to retrain, care, start businesses, and work flexibly—without fear.

## 9) Initial content placeholders
- People page: "Founder" + "Board (coming soon)" + "Advisors (coming soon)"
- News: empty state with "No updates yet" + mailing list CTA
- Contact: address can be "Basel, Switzerland" (no street address unless provided later)

## 10) Deliverables
- A complete website repo with:
  - README (local dev + build + deploy)
  - i18n routing working
  - content in DE/FR/IT for all core pages (concise but non-empty)
  - a clean, professional UI
- Provide deployment guidance:
  - Cloudflare Pages / Netlify / Vercel
  - attach domain die-zukunft.ch
- Ensure the site works without a backend (unless contact/newsletter requires one).

## 11) Non-goals (v1)
- No complex member portal
- No donations/payment processing (can be added later)
- No heavy CMS unless absolutely necessary

## 12) Suggested repo structure (example)
/
  src/
  content/
    de/
    fr/
    it/
  public/
  package.json (if JS stack)
  README.md
  LICENSE

## 13) Success criteria (definition of done)
- Looks credible and "Swiss serious"
- All pages exist in DE/FR/IT with working language switcher
- Mobile looks excellent
- Lighthouse performance strong
- Clear CTAs for joining/supporting
- No climate framing, no sci-fi vibe, no fear-mongering
