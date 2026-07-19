# Upstream Relay — Deployment Requirements

Requirements for taking `upstream-relay-site.html` (single-file static site, v4) live at **upstreamrelay.com**. Written to be executed by Claude Code.

---

## 1. Goal & guardrails

Deploy the existing single-page site as a fast, secure, production static website with a working contact form. **Do not over-engineer:**

- No framework, no build pipeline beyond minification, no JavaScript beyond what's already in the file (which is none), no CMS, no database.
- The site is one hand-coded HTML file with inline CSS and inline SVG. Keep it that way. Do not convert to React/Next/Astro/etc.
- Preserve the existing design, copy, structured data, and the "no prices in the UI" rule exactly. Do not rewrite content.
- Preserve the constraint stated in the site footer: no trackers. Any analytics must be cookie-free and privacy-first, or omitted.

## 2. Tech stack (decided — do not substitute)

| Concern | Choice | Why |
|---|---|---|
| Hosting | **Netlify** (free tier) | Global CDN, automatic HTTPS, and built-in form handling — eliminates the only piece that would otherwise need backend code. |
| Forms | **Netlify Forms** with honeypot | Zero backend, spam filtering, email notifications. Replaces the current `mailto:` action, which is the one launch blocker. |
| Source control | **GitHub repo** → Netlify continuous deploy from `main` | Standard, simple, reviewable. |
| Fonts | **Self-hosted** woff2 (see §4) | Removes third-party requests to Google, faster first paint, no font-related privacy exposure. |
| Analytics | **Netlify Analytics** (server-side) *or none* | Cookie-free, no client script, keeps the "no trackers" claim true. Do not add GA4/Meta pixels. |

Acceptable alternative if the owner prefers: Cloudflare Pages + Formspree. Only switch if instructed.

## 3. Repository structure

```
upstream-relay/
├── index.html          # from upstream-relay-site.html (modified per below)
├── 404.html            # simple branded not-found page, links home
├── fonts/              # self-hosted woff2 files
├── og-image.png        # 1200×630 social card (see §5)
├── favicon.svg         # reuse the nav logo SVG (blue merge arrow + yellow dot)
├── apple-touch-icon.png# 180×180 raster of the same mark
├── robots.txt
├── sitemap.xml
├── _headers            # Netlify security/caching headers (see §6)
├── _redirects          # www → apex 301
└── netlify.toml        # build config, HTML minification via plugin or none
```

## 4. Performance requirements

Target: **Lighthouse ≥ 95 on all four categories (mobile)**; LCP < 1.5s, CLS < 0.05, total page weight < 300 KB.

1. **Self-host fonts.** Download woff2 subsets (latin) of Bricolage Grotesque (400/600/700/800), Public Sans (400/500/600/700), IBM Plex Mono (400/500/600). Serve from `/fonts/` with `@font-face`, `font-display: swap`. Remove the three Google Fonts `<link>` tags and both `preconnect`s. Add `<link rel="preload" as="font">` for the two fonts used above the fold (Bricolage Grotesque 800, Public Sans 400).
2. **Trim font weights if unused.** Audit the CSS; drop any weight not actually referenced.
3. **Minify** `index.html` for production (single file: this is the entire "build"). Keep an unminified copy in the repo as the source of truth if minification is done at deploy time.
4. **Caching:** immutable, 1-year `Cache-Control` for `/fonts/*` and images; `max-age=0, must-revalidate` for HTML (see `_headers` in §6).
5. No additional JS, no external CSS, no third-party embeds. The inline SVG hero stays inline.
6. Keep the existing `prefers-reduced-motion` handling untouched.

## 5. SEO requirements

1. **Canonical & domain:** primary domain is `https://upstreamrelay.com` (apex). 301 `www.upstreamrelay.com` → apex via `_redirects`. Confirm the `<link rel="canonical">` matches the final URL exactly.
2. **sitemap.xml** listing the single URL; **robots.txt** allowing all and pointing to the sitemap.
3. **OG image:** create `og-image.png` (1200×630) — chalk background, the wordmark, the subway-map motif from the hero, tagline "Fractional Developer Relations · NYC". No prices. Add `og:image`, `og:image:width/height`, `og:image:alt`, and `twitter:image` meta tags; switch `twitter:card` to `summary_large_image`.
4. **Preserve existing structured data** (ProfessionalService + FAQPage JSON-LD) byte-for-byte except: no changes needed. Validate both blocks with Google's Rich Results test after deploy.
5. **404 page** with a link back home (Netlify serves `404.html` automatically).
6. After DNS cutover: submit the domain to Google Search Console and Bing Webmaster Tools, submit the sitemap, and verify the FAQ rich result renders.

## 6. Security requirements

1. **HTTPS everywhere:** Netlify-managed TLS cert; enable "Force HTTPS" (HTTP → HTTPS 301).
2. **`_headers` file** applying to `/*`:
   - `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`
   - `Content-Security-Policy: default-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; form-action 'self'; frame-ancestors 'none'; base-uri 'self'` — note `'unsafe-inline'` for styles is required because the CSS is inline; that is an accepted trade-off for this architecture. Tighten with a hash/nonce only if trivial.
   - `X-Content-Type-Options: nosniff`
   - `Referrer-Policy: strict-origin-when-cross-origin`
   - `Permissions-Policy: camera=(), microphone=(), geolocation=()`
   - `X-Frame-Options: DENY`
3. **Form hardening:** convert the contact form to Netlify Forms (`data-netlify="true"`, hidden `form-name` input) with a honeypot field (`netlify-honeypot`). Keep it three visible fields. On submit, redirect to a small inline success state or `/thanks.html` (create it: brief "A partner will reply within one business day" message, link home). Set form notification email to `hello@upstreamrelay.com`.
4. **Email authenticity for the domain** (flag for the owner; requires DNS access): set up SPF, DKIM, and DMARC records once mailbox hosting for `hello@upstreamrelay.com` is chosen. Mailbox hosting itself is out of scope for this deploy — flag it as a launch dependency, since form notifications must go somewhere real.
5. No cookies are set by the site; confirm after deploy (keeps the no-tracker claim and avoids any consent-banner requirement).
6. Enable DNSSEC at the registrar if supported.

## 7. Deployment steps (in order)

1. Create the GitHub repo with the structure in §3; commit `index.html` derived from `upstream-relay-site.html` with the §4–§6 modifications.
2. Connect the repo to Netlify; no build command; publish directory = repo root.
3. Verify the deploy preview: fonts load locally, form submits to Netlify Forms, headers present (check with `curl -I`).
4. Add the custom domain in Netlify; update DNS at the registrar (apex A/ALIAS + `www` CNAME per Netlify's instructions); enable Force HTTPS after cert issuance.
5. Confirm `_redirects` (www→apex) and 404 behavior on the live domain.
6. Run Lighthouse (mobile + desktop) and the Rich Results test; fix anything below target.
7. Submit sitemap to Search Console/Bing.

## 8. Acceptance criteria

- [ ] Live at `https://upstreamrelay.com`; `www` and `http` both 301 to it
- [ ] Lighthouse mobile ≥ 95 across Performance / Accessibility / Best Practices / SEO
- [ ] Contact form delivers a test submission to `hello@upstreamrelay.com` and blocks a honeypot submission
- [ ] All §6 headers present on every response; SSL Labs grade A or better
- [ ] No requests to any third-party domain on page load (verify in DevTools network tab)
- [ ] No cookies set; no prices anywhere in rendered UI, meta tags, or structured data
- [ ] ProfessionalService and FAQPage JSON-LD both validate with zero errors
- [ ] Visual output is pixel-equivalent to the approved v4 design (spot-check hero map, station-sign strip, line bullets, taxi-yellow CTAs, reduced-motion mode)

## 9. Explicitly out of scope

Blog, multiple pages, CMS, A/B testing tooling, chat widgets, marketing pixels, newsletter capture, booking/calendar embeds (may be added later — if a scheduling link is adopted, it replaces the form's success-page copy, not the form). Do not add dependencies "while we're here."
