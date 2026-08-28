# DUO MARINE — Developer Handover

One-page marketing site for DUO MARINE (Malta), reseller of MV Marine RIBs and
Yamaha WaveRunner jet skis.

## What is in this bundle

| Path | What it is |
|---|---|
| `frontend-spec.md` | Full FE specification — tokens, layout, components, interactions, responsive rules |
| `backend-spec.md` | BE specification — content model, enquiry handling, CMS options, image pipeline |
| `site/` | The working static site, deployable as-is |

## Status of the design files

`site/` is **not a mockup — it is the live, shipping site.** It is a complete
static build: open `site/index.html` in a browser and it runs. It is currently
deployed at duomarine.netlify.app.

That means you have two legitimate paths, and the specs support either:

1. **Keep it static.** Push `site/` to a repo, deploy on Netlify/Vercel/Pages,
   edit content directly in `index.html`. Zero infrastructure. Correct choice
   until content changes get frequent.
2. **Rebuild it in a framework with a CMS.** Use `frontend-spec.md` to
   reproduce the UI exactly, and `backend-spec.md` for the content model and
   enquiry endpoint. Do this when non-technical staff need to edit boats,
   specs, photos and prices without a developer.

**Do not treat path 2 as a given.** The site has four products and a handful of
copy blocks. A CMS is warranted by editing frequency, not by principle.

## Fidelity

High-fidelity and final. Every colour, size, weight, easing curve and piece of
copy in `site/` is the intended production result and has been signed off.
Where the spec and the HTML disagree, **the HTML is correct** — treat the spec
as documentation of it.

## Architecture of the current build

`index.html` is a single self-contained page:

- Markup and all styling **inline** (no stylesheet, no CSS classes except a
  handful of animation helpers in `<style>`)
- `support.js` is a small runtime that mounts the page's logic class
  (tab state, accordions, lightbox, lazy loading, parallax, boot screen)
- Assets in `assets/` — WebP images, TTF fonts, SVG logos
- `assets/gallery/` — 24 product photos, lazily loaded, one per gallery cell

Total first load ≈ 3.8 MB including fonts; gallery photos stream in on scroll.

## Immediate next steps

1. Create the repo, commit `site/` at the root (see `backend-spec.md` §7).
2. Point the custom domain at it and enable HTTPS.
3. Decide on the CMS question above before writing any backend code.
4. Replace the `mailto:` CTAs with a real form **only** if you also build the
   endpoint in `backend-spec.md` §3 — a form that silently fails is worse than
   a mail client opening.

## Contact points in the build

- Enquiry email: `info@duomarine.mt` (every CTA is a `mailto:` to this)
- Instagram: `https://www.instagram.com/duomarine.mt/`

## Open items for the client, not the developer

- Boat prices are currently **not** shown. If they return, they belong in the
  technical card (`frontend-spec.md` §5.3).
- Gallery photos are placeholders in composition terms — real professional
  shoot imagery should replace `assets/gallery/*.webp` at the same aspect.
