# Antinol on Tour 2026 — Shopify Splash Handoff

**For Claude Code.** Read this entire file before starting. Save a copy at the repo root as `CLAUDE.md` once the repo is initialised.

---

## Mission

Convert the existing single-file HTML splash page (v6.4) into a **Shopify page template** for the **antinol.com.au** store, served at the vanity URL **antinol.com.au/trailer**. Splash exists to convert QR-code scans (taken at the trailer's exterior wrap) and direct-typed traffic from people who saw the trailer in person at one of twelve 2026 K9 sporting events.

---

## Source Artifact

- **File:** `antinol-on-tour.html` (v6.4)
- **Size:** ~62 KB single HTML, all CSS + JS inlined, SVG logo as inline `<symbol>`
- **Location:** Jonno will drop it at the repo root before invoking you. If missing, ask before generating anything.
- **Version stamp:** keep the footer `<span class="vp-foot__version">Splash v6.X</span>` and the top-of-file HTML comment current. Bump the patch on every change.

---

## Stack — Shopify Online Store 2.0

This is a Shopify theme deliverable. **No Vite, no npm, no React, no build step.** Files go directly into the live theme via Shopify CLI.

```
Theme:    Existing antinol.com.au theme (Dawn-based, OS 2.0)
Liquid:   Page template + custom layout for the splash
Assets:   trailer.css, trailer.js, vp-logo.svg in /assets
GSAP:     CDN <script> tags (already wired in v6.4)
Video:    Stays on Shopify CDN (current URL is fine)
Forms:    Klaviyo public API POST from trailer.js
Deploy:   Shopify CLI (theme push --unpublished, then publish)
```

If Jonno hasn't said which store: **ask.** Default assumption is `antinol.com.au`. Products linked from the splash live on `vetzpetz.com.au` — leave those external links as full URLs.

---

## URL Strategy

Shopify pages live at `/pages/<handle>`, not at the root. To get **`/trailer`** working:

1. Create a page in admin with handle `trailer` → URL becomes `/pages/trailer`.
2. Add a **URL redirect** in `Online Store → Navigation → URL Redirects`:
   - From: `/trailer`
   - To: `/pages/trailer`
3. The QR code on the trailer wrap should point to `https://antinol.com.au/trailer` — the redirect resolves it.

The redirect is a one-time admin task. Document it in `README.md` so it survives theme migrations.

---

## File Structure (to add to the theme)

```
<theme-root>/
├── layout/
│   └── trailer.liquid              ← custom minimal layout (no header/footer)
├── templates/
│   └── page.trailer.liquid         ← page template, references the layout
├── sections/
│   └── trailer-events.liquid       ← (optional v2) admin-editable events list
├── snippets/
│   └── vp-logo.liquid              ← inline SVG symbol, rendered once per page
└── assets/
    ├── trailer.css                 ← extracted from the inline <style>
    ├── trailer.js                  ← extracted from the inline <script>
    └── vp-logo.svg                 ← (optional) external version of the logo
```

`vp-logo` as a snippet is preferred over an external SVG file — the symbol is referenced twice (nav, footer) and inlining avoids an extra HTTP request.

---

## Custom Layout — `layout/trailer.liquid`

The default `layout/theme.liquid` wraps every page in the store's nav and footer. The splash has its own self-contained nav and footer (`.vp-nav`, `.vp-foot`) and must NOT inherit the theme's chrome. Create a stripped-down layout:

```liquid
<!doctype html>
<html lang="{{ request.locale.iso_code }}" class="no-js">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
  <meta name="theme-color" content="#ffffff">
  <title>{{ page_title }}</title>
  <meta name="description" content="{{ page_description | escape }}">

  {{ content_for_header }}

  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="preload" as="video" href="https://cdn.shopify.com/videos/c/o/v/56c601227c674c70b7b09db80a7b4522.mp4">
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
  {{ 'trailer.css' | asset_url | stylesheet_tag }}
</head>
<body class="trailer-splash">
  {{ content_for_layout }}

  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
  <script src="{{ 'trailer.js' | asset_url }}" defer></script>
</body>
</html>
```

`{{ content_for_header }}` is required by Shopify (analytics, Klaviyo, app pixels). Don't strip it.

---

## Page Template — `templates/page.trailer.liquid`

```liquid
{% layout 'trailer' %}
{% render 'vp-logo' %}

<nav class="vp-nav" id="vp-nav">
  ...
</nav>

<section class="vp-hero">...</section>
<section class="vp-events" id="events">...</section>
<section class="vp-reveal" id="trailer">...</section>
<section class="vp-what" id="about">...</section>
<section class="vp-pillars">...</section>
<section class="vp-starter" id="starter">...</section>
<section class="vp-cta">...</section>
<section class="vp-merch">...</section>
<footer class="vp-foot">...</footer>
```

The body of each section is a near-verbatim copy of the v6.4 markup. **Keep the BEM `vp-` classnames as-is.** They don't collide with Dawn's classes.

`{% layout 'trailer' %}` at the top forces the custom layout instead of `theme.liquid`.

`{% render 'vp-logo' %}` injects the hidden `<svg><symbol>` once near the top of body so both `<use href="#vp-logo"/>` references resolve.

---

## Logo Snippet — `snippets/vp-logo.liquid`

Paste the existing `<svg class="vp-svg-defs">...<symbol id="vp-logo">...</symbol></svg>` block from v6.4 verbatim. No Liquid logic needed inside.

---

## Asset Files

### `assets/trailer.css`

Take everything between `<style>` and `</style>` in v6.4 and save it as a flat CSS file. No changes. The CSS is already self-contained.

### `assets/trailer.js`

Take everything inside the inline `<script>` block (the one after the GSAP CDN scripts). One change: when extracting, the script no longer runs at the position where the body class was added. Move this line to the top:

```js
document.body.classList.add('is-loaded');
```

It already runs at the top of the inline script in v6.4 — just preserve that order. Use `defer` on the `<script>` tag in the layout (already shown above) so it runs after DOM parse without needing `DOMContentLoaded`.

---

## Klaviyo Wiring

The Vetz Petz / Antinol stack already runs Klaviyo. Replace the current mock submission in `trailer.js` with a real subscribe call.

**Approach:** client-side POST to Klaviyo's public subscribe endpoint using the public API key (safe to expose) and a list ID. Pet name + breed save as profile properties.

Required values (ask Jonno):
- `KLAVIYO_PUBLIC_KEY` — 6-character public key, e.g. `XaBcDe`
- `KLAVIYO_LIST_ID` — list ID for the trailer-tour signup audience

Theme settings is the cleanest place for these so marketing can rotate without dev help:

```liquid
{% comment %} settings_schema.json {% endcomment %}
{
  "name": "Trailer Splash",
  "settings": [
    { "type": "text", "id": "klaviyo_public_key", "label": "Klaviyo Public API Key" },
    { "type": "text", "id": "klaviyo_list_id",   "label": "Klaviyo List ID (Trailer Signups)" }
  ]
}
```

Pass to JS via a data attribute on `<body>`:

```liquid
<body class="trailer-splash"
      data-klaviyo-key="{{ settings.klaviyo_public_key }}"
      data-klaviyo-list="{{ settings.klaviyo_list_id }}">
```

Then in `trailer.js`:

```js
const cfg = document.body.dataset;
const KLAVIYO_KEY = cfg.klaviyoKey;
const KLAVIYO_LIST = cfg.klaviyoList;

// POST to https://manage.kmail-lists.com/ajax/subscriptions/subscribe
// with form-encoded body: g=<list_id>, email=<email>, $first_name=..., pet_name=..., pet_breed=...
```

Also fire a Shopify customer subscribe event if the store uses Shopify's customer accounts integration with Klaviyo — coordinate with Jonno before duplicating profile creates.

---

## UTM / QR Source Capture

The existing `captureSource()` IIFE writes `utm_source` to sessionStorage and to a hidden form field. Keep it.

The QR code on the trailer should encode:

```
https://antinol.com.au/trailer?utm_source=trailer-exterior&utm_medium=qr&utm_campaign=on-tour-2026
```

For events with multiple QR placements, use distinct `utm_source` values: `trailer-exterior`, `trailer-interior`, `trailer-merch`, etc. Document the convention in `README.md`.

---

## Deployment Workflow

```bash
# Initial setup
shopify login --store antinol.myshopify.com
shopify theme list

# Pull current live theme into a working copy
shopify theme pull --live -t live -p assets,sections,snippets,templates,layout

# Make changes locally, run dev server with hot-reload
shopify theme dev

# Push to an unpublished theme for QA
shopify theme push --unpublished --theme "Trailer Splash QA"

# Once approved, publish
shopify theme publish --theme "Trailer Splash QA"
```

Don't develop directly against the live theme. Always use an unpublished theme for QA, switch over only after approval.

---

## Bootstrap Tasks (do in this order)

1. **Initialise repo.** `git init`, add `.gitignore` (standard Shopify CLI ignore: `.shopifyignore`, `node_modules`, `.DS_Store`, `*.log`).
2. **Pull the live antinol.com.au theme** with `shopify theme pull` so you're working against the real codebase.
3. **Create `snippets/vp-logo.liquid`** by extracting the SVG symbol block from v6.4.
4. **Create `assets/trailer.css`** from the inline `<style>`.
5. **Create `assets/trailer.js`** from the inline `<script>`. Move `document.body.classList.add('is-loaded')` to the very top.
6. **Create `layout/trailer.liquid`** as shown above.
7. **Create `templates/page.trailer.liquid`** — paste the body markup from v6.4 between layout/render directives.
8. **Add Klaviyo settings** to `config/settings_schema.json` and `data-` attributes to `<body>` in the layout. Wire `trailer.js` to read them.
9. **`shopify theme push --unpublished --theme "Trailer Splash QA"`** to deploy the working version.
10. **In admin:** create a Page with handle `trailer`, body left blank (the template renders everything), template `page.trailer`.
11. **In admin:** create a URL redirect from `/trailer` to `/pages/trailer`.
12. **QA on a real device** — scan a QR pointing to `https://antinol.myshopify.com/trailer?utm_source=qa-test`, confirm the redirect works, the splash renders, the video scrubs, the form submits to Klaviyo, the version stamp shows in the footer.
13. **Publish** the QA theme once approved.
14. **Commit** the working files to the repo with `chore: initial trailer splash deploy v6.4`.

---

## Conventions

### Liquid
- One template per file. Avoid complex logic in templates — push to snippets if a block is reused.
- Asset URLs always via `{{ 'file' | asset_url }}` — never hardcoded `cdn.shopify.com` paths.
- Comments use `{% comment %}` blocks, not HTML `<!-- -->` for Liquid-only notes.

### CSS / JS
- Don't rename existing `vp-` classnames. The class system is intentional and matches the rest of Jonno's Shopify work.
- Keep `assets/trailer.css` as one file. Don't split into Dawn-style component CSS — this is a one-off splash, not part of the design system.
- Don't inline new CSS or JS into the page template. Everything goes into the asset files.

### Markup
- Don't restructure section order: nav → hero → events (primary) → trailer reveal → what is Antinol → pillars → starter packs → CTA stack → merch → footer.
- All interactive controls have `min-height: 48px` for touch targets. Don't shrink below that.

---

## Brand Rules — DO NOT VIOLATE

### Vetz Petz Palette (locked)

```css
--vp-red:        #F50002;
--vp-red-deep:   #C70002;
--vp-navy:       #1A2E3B;  /* Academic Navy */
--vp-ink:        #0d1820;
--vp-charcoal:   #3a4047;
--vp-charcoal-soft: #6a7178;
--vp-white:      #FFFFFF;
--vp-grey:       #F2F2F2;  /* Clinic Grey */
--vp-yellow:     #E5F221;  /* Highlighter — emphasis only */
```

No other colours. No multi-stop gradients. The only gradient permitted is the highlighter on `.vp-mark`.

### Logo

- Always rendered via `{% render 'vp-logo' %}` + `<use href="#vp-logo"/>`.
- Two CSS variables control colours: `--vp-logo-c1` (red), `--vp-logo-c2` (yellow). Footer overrides both to white.
- Don't use `filter: brightness(0) invert(1)` — the variables replace it.

### Typography

- Single typeface: Montserrat (300/400/500/600/700/800/900). Loaded from Google Fonts.
- Headlines `font-weight: 900`, `letter-spacing: -.025em` to `-.03em`.
- Body `font-weight: 500`. Default line-height 1.55.

---

## Pending Integrations

Note these in `README.md` under "Roadmap":

1. **Real event photos** — twelve 4:3 placeholders in the "Inside the trailer" section (`<div class="vp-step__photo"><span>Label</span></div>`). Swap the inner `<span>` for `<img src="{{ 'slatmill-1.jpg' | asset_url }}" alt="...">` when shots are delivered.
2. **Real event detail pages** — every `.vp-events__row` href is `#`. Twelve detail pages (or modal flyouts) follow.
3. **OG image + favicon** — placeholder slot. 1200×630 OG image, full favicon set.
4. **`prefers-reduced-motion` audit** — `.vp-rise` reveals respect the rule, video scrub does not. Decide whether to swap to a poster frame for those users.
5. **Refactor to OS 2.0 sections** — once stable, break the events list and the reveal items into admin-editable sections so marketing can update event details without dev help.
6. **Shopify customer creation** — currently the form only writes to Klaviyo. If Jonno wants attendees in Shopify customers too, add a parallel `customer.create` POST.

---

## Things You Should NOT Do

- Don't migrate to React, Next.js, Astro, Hydrogen, or any framework.
- Don't add Tailwind, UnoCSS, or any utility-first CSS layer.
- Don't add TypeScript.
- Don't add npm dependencies. The CDN GSAP scripts are the only third-party JS allowed.
- Don't replace GSAP with Motion One, Framer Motion, or anything else.
- Don't use Shopify's section schemas to "convert everything to admin-editable" in v1 — that's the v2 follow-up.
- Don't touch the existing antinol.com.au theme's other templates. Splash is additive.
- Don't change the colour palette, typography, or section order.
- Don't remove the version stamp in the footer.
- Don't restructure the `Inside the trailer` scroll mechanic (sticky video + 6 scrolling steps + scrub) — it's been iterated on heavily.
- Don't develop against the live theme. Always use an unpublished QA theme until approved.

---

## Acceptance Criteria

1. `shopify theme dev` starts a local preview with hot-reload pointing at the QA theme.
2. The page renders at `/pages/trailer` and the redirect from `/trailer` resolves correctly.
3. **Visually identical** to the v6.4 single-file source. Pixel-diff if you must.
4. The trailer scrub mechanic works on Chrome desktop, Safari desktop, Chrome Android, and Safari iOS. iOS warm-up (the touchstart `play().then(pause())` trick) is preserved.
5. Form submits to Klaviyo successfully — verify a test email lands in the configured list with `pet_name` and `pet_breed` profile properties populated.
6. UTM source captures correctly — scan a QR pointing to `?utm_source=qa-test`, submit the form, verify the source lands on the Klaviyo profile.
7. Lighthouse mobile (run on the published theme): Performance ≥ 90, Accessibility ≥ 95, Best Practices ≥ 95, SEO ≥ 95.
8. No console errors, no console warnings except font-loading info and Shopify pixel chatter.
9. `CLAUDE.md` exists at repo root with the contents of this handoff.

---

## Quick Context for Future You

- **Audience:** dog-sport competitors and owners scanning a QR on the trailer's exterior wrap at one of twelve Tier-1 K9 events across NSW, VIC, QLD in 2026.
- **Network reality:** rural showgrounds, 4G, sometimes worse. First Contentful Paint matters more than animation polish.
- **Primary content:** the events list. Everything else is supporting.
- **Brand voice:** plain Australian English. Short sentences. No jargon. "Bring the dog. Bring the team." not "Engage with our holistic ecosystem."
- **Two stores:** antinol.com.au hosts the splash, vetzpetz.com.au is the main brand store and where the products are bought. External product links from the splash go to vetzpetz.com.au — leave those as full URLs.

If anything in this handoff conflicts with a fresh request from Jonno, his request wins — but flag the conflict before changing brand-locked items (palette, fonts, section order).
