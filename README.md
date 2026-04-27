# Antinol on Tour 2026 — Trailer Splash

Splash page for the **Antinol on Tour 2026** mobile trailer, scanned via QR codes at twelve K9 sporting events across Australia.

## Preview

Live preview (GitHub Pages, served from `index.html`):

> https://jonnosenior.github.io/antinol-on-tour-2026/

The preview is the iteration target — once it's signed off, the same markup gets ported into the **vetzpetz.com.au** Shopify theme as a custom page template per `HANDOFF.md`.

## Files

- `index.html` — current working copy. Edit this to iterate. Single file, all CSS + JS inlined, GSAP via CDN.
- `antinol-on-tour.html` — frozen v6.4 snapshot from the original handoff. Don't edit; reference only.
- `HANDOFF.md` — Shopify port plan. Read before touching the theme.

## Versioning

Bump the patch version in two places on every change:

1. The HTML comment at the top of `index.html`: `<!-- Antinol on Tour splash · v6.X · YYYY-MM-DD -->`
2. The footer stamp: `<span class="vp-foot__version">Splash v6.X</span>`

## Deploy target

- **Vanity URL:** `vetzpetz.com.au/trailer`
- **Resolves to:** `vetzpetz.com.au/pages/trailer` via a URL redirect in `Online Store → Navigation → URL Redirects`
- **Shopify store:** `vetzpetz-au.myshopify.com`

QR codes on the trailer encode the vanity URL with UTM params:

```
https://vetzpetz.com.au/trailer?utm_source=trailer-exterior&utm_medium=qr&utm_campaign=on-tour-2026
```

For multiple QR placements per event, vary `utm_source`: `trailer-exterior`, `trailer-interior`, `trailer-merch`, etc.

## Roadmap

1. Real event photos (twelve 4:3 placeholders in the "Inside the trailer" section).
2. Real event detail pages — every `.vp-events__row` href is currently `#`.
3. OG image (1200×630) + full favicon set.
4. `prefers-reduced-motion` audit for the video scrub mechanic.
5. Klaviyo wiring (public key + list ID, profile properties for pet name and breed).
6. Port to Shopify per `HANDOFF.md` once the HTML preview is signed off.
