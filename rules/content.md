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
- **Mitmachen**: `mitmachen.html` — join/support with newsletter form
- **Info**: `info.html` — legal/footer content
- **Generic pages**: `page.html` — title + description + markdown body

## Landing page content
The landing page (`_index.md`) is special — all content is in `[extra]` frontmatter fields:
- `hero_title`, `hero_subline`, `hero_subtitle`
- `changes_title`, `changes = [{ icon, title, text }]`
- `pillars_title`, `pillars = [{ icon, title, text }]`
- `timeline_title`, `timeline = [{ phase, title, text }]`
- `cta_title`, `cta_text`

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

## Tone guidelines
- Pragmatic, confident, future-oriented, non-hysterical
- No "free money" framing for UBI — use "stability", "simplification", "foundation"
- No climate framing — use "resilience", "durability", "long-term stability"
- No sci-fi vibes — keep it Swiss-serious and measurable
- No culture-war tone
