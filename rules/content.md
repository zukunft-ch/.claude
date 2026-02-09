# Content Editing Rules

## File locations
All content lives in `die-zukunft.ch/content/`. Never edit templates to change page text.

## File naming convention
- German (default): `page-name.md`
- French: `page-name.fr.md`
- Italian: `page-name.it.md`
- English: `page-name.en.md`
- Section index: `_index.md` / `_index.fr.md` / etc.

## Frontmatter format
Always use TOML frontmatter delimited by `+++`:
```toml
+++
title = "Page Title"
description = "Meta description for SEO."
template = "page.html"

[extra]
show_cta = true
cta_text = "Call to action text"
+++

Markdown content goes here.
```

**Never** use YAML frontmatter (`---`). Zola uses TOML.

## When editing content
1. Always edit all language variants together (DE, FR, IT, and EN if present)
2. Keep the same structure/sections across languages — only the text differs
3. After editing, run `zola build` to verify zero warnings
4. Check that the page renders by looking at `public/` output

## Templates per page type
- **Landing page**: `index.html` — content is in `_index.md` frontmatter (`[extra]` fields), not in markdown body
- **Vision**: `vision.html` — uses `[[extra.sections]]` arrays in frontmatter
- **Program**: `programm.html` — pillar cards from frontmatter
- **Grundeinkommen**: `grundeinkommen.html` — FAQ via `{% faq %}` shortcode
- **Contact**: `kontakt.html` — form built into template
- **Mitmachen**: `mitmachen.html` — join/support with Verein link, newsletter form, donations
- **Info**: `info.html` — legal/footer content
- **Generic pages**: `page.html` — title + description + markdown body

## Landing page content
The landing page (`_index.md`) is special — all content is in `[extra]` frontmatter fields:
- `hero_title`, `hero_subline`, `hero_subtitle`, `hero_descriptor`
- `changes_title`, `changes_intro`, `changes = [{ icon, title, text }]`, `changes_cta`
- `pillars_title`, `pillars = [{ icon, title, text, link }]`
- `timeline_title`, `timeline = [{ phase, title, text }]`
- `cta_title`, `cta_text`

### "Was sich verändert" cards (4 cards, 2x2 bento grid)
1. Arbeit & Einkommen — productivity gains vs risk distribution
2. Wohnen & Alltag — housing, commutes, comfort per m²
3. Gesundheit & Langlebigkeit — biotech with trust, consent, no black boxes
4. Digitale Sicherheit — payments, identity, admin on foreign platforms = sovereignty

Each card should ground an abstract topic in **everyday experience** (rent, insurance, jobs, commutes) — not read like a policy brief. The `changes_intro` bridge and `changes_cta` closing line frame the cards.

## Shortcodes available
- `{% faq(question="...") %}Answer text{% end %}` — collapsible FAQ item
- `{% callout(type="info") %}Text{% end %}` — styled callout block
- `{{ newsletter_form() }}` — newsletter signup (currently disabled)

## Translation strings
UI text (button labels, form labels, nav items) lives in `config.toml` under:
- `[translations]` — German defaults
- `[languages.fr.translations]` — French
- `[languages.it.translations]` — Italian
- `[languages.en.translations]` — English

Use `{{ trans(key="key_name", lang=lang) }}` in templates.

## External links in content & footer
- **Verein**: `https://collective.die-zukunft.ch` — association/membership site
- **Spenden**: `https://opencollective.com/die-zukunft` — transparent donations
- **The Future**: `https://the-future.world` — worldwide movement site
- All three appear in footer contact column + movement pill CTA above copyright
- Verein also appears as first section on mitmachen pages (all 4 languages)

## Tone guidelines
- Pragmatic, confident, future-oriented, non-hysterical
- No "free money" framing for UBI — use "stability", "simplification", "foundation"
- No climate framing — use "resilience", "durability", "long-term stability"
- No sci-fi vibes — keep it Swiss-serious and measurable
- No culture-war tone
- "Was sich verändert" cards must be **relatable** (everyday examples like rent, insurance, job changes) — not abstract/technocratic
