# CLAUDE.md

Project rules for the Upstream Relay marketing site. Read this before making any change.

## What this is

A single-file static site. The entire site lives in `index.html` — inline CSS, inline SVG hero, no separate stylesheet or script file. No framework, no build step, no external dependencies, no JavaScript unless the user explicitly asks for it.

## Hard rules

- **No prices anywhere.** Never add pricing to UI copy, meta tags, or structured data. This applies everywhere, always — not a style preference.
- **Preserve both JSON-LD blocks** (`ProfessionalService` and `FAQPage`) byte-for-byte unless the user explicitly asks to change them.
- **Preserve the `_headers` security configuration** (HSTS, CSP, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, X-Frame-Options) in any edit. Don't loosen, remove, or "simplify" headers without being explicitly asked.
- **No new dependencies.** No CSS/JS frameworks, no CDN scripts, no analytics or trackers of any kind — the site is cookie-free and tracker-free by design.
- **Forms use Netlify Forms.** Don't change the `<form>` attributes (`data-netlify`, `netlify-honeypot`, the hidden `form-name` input, honeypot field, `action="/thanks.html"`) without being explicitly asked.
- **Keep total deployed weight under 300 KB.** Check current weight before adding any asset (fonts, images, etc.).

## Design system (NYC transit-inspired)

- Fonts are self-hosted in `/fonts/` as woff2 and loaded via `@font-face` — never reintroduce Google Fonts `<link>` tags.
  - **Bricolage Grotesque** — display/headings only (weights 700, 800 — audit the CSS before adding other weights; unused weights get dropped)
  - **Public Sans** — body text (weights 400, 500, 600, 700)
  - **IBM Plex Mono** — labels, eyebrows, mono UI text (weights 400, 500, 600)
- MTA line colors (`--l-yellow`, `--l-orange`, `--l-green`, `--l-gray`, `--l-red`, `--l-blue`) are for the service bullets and subway-line motifs only — don't repurpose them decoratively elsewhere.
- Taxi-yellow (`--signal`, `#FCCC0A`) is reserved for CTAs. Don't use it for anything else.
- If a color pair fails contrast (WCAG AA), fix it by adding a darker "-deep" variant of the existing color (see `--primary-deep`, `--l-green-deep`) scoped to that one selector — don't change the shared base variable, and don't introduce an unrelated new hue.

## Structure conventions

- Section eyebrow labels (`01 · STRATEGY SPRINT`, `02 · STRATEGY RETAINER`, etc.) must stay numerically sequential. If you reorder sections, renumber them.
- Nav anchor links (`#services`, `#process`, `#faq`, `#commitments`) must stay in sync with the actual section `id`s — never rename a section `id` without updating the nav to match.

## Repo layout

- `index.html` — the entire site; both the source and the deploy artifact, no separate build step
- `fonts/` — self-hosted woff2 files
- `404.html`, `thanks.html` — standalone branded pages, kept visually consistent with the main palette
- `favicon.svg`, `apple-touch-icon.png`, `og-image.png` — brand assets
- `robots.txt`, `sitemap.xml`, `_headers`, `_redirects` — SEO/security/Netlify config
- `upstream-relay-website.html`, `upstream-relay-deployment-requirements.md` — original source and spec docs, kept for provenance, not part of the deployed site

## Deploy

Netlify, continuous deploy from `main`, no build command, publish directory = repo root.
