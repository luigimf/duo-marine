# DUO MARINE — Backend Specification

## 0. Current state, stated plainly

**There is no backend.** The site is static HTML: no server, no database, no
build step, no runtime dependencies. Enquiries are `mailto:` links that open the
visitor's mail client.

This is not an oversight — for a four-product brochure site it is the right
architecture. Everything below is **conditional**: build the section that
answers a real need, not the whole document.

| If the client needs… | Build § |
|---|---|
| Staff editing boats/photos without a developer | §2 content model + §4 CMS |
| Enquiries in an inbox/CRM instead of the visitor's mail app | §3 |
| More products, or a second market | §2 + §4 |
| Nothing to change monthly | Nothing. Ship §7 and stop. |

---

## 1. Recommended architecture (if you go dynamic)

Static-first, not server-rendered-per-request. The content changes rarely and
the page must stay fast on Maltese mobile connections.

```
Headless CMS  ──build hook──▶  SSG (Astro / Next static / Nuxt generate)
                                    │
                              CDN (Netlify / Vercel)
                                    │
                          Serverless function ──▶  Email / CRM
                              (enquiry endpoint only)
```

Rules:

- **Keep the output static.** No SSR, no client-side data fetching for content.
- The only runtime code is the enquiry endpoint.
- Images processed at build time, never on request.

---

## 2. Content model

Four collections and one singleton. Field names map to what the current HTML
hard-codes.

### 2.1 `product`

| Field | Type | Notes |
|---|---|---|
| `id` | slug | `mv-23gt`, `mv-27gt`, `gp-svho`, `fx-limited-svho` |
| `category` | enum | `rib` \| `jetski` — drives which section it renders in |
| `name` | string | "MV 23GT" — displays in `#F47F4B` |
| `tabLabel` | string | Short form for the tab pill ("FX Limited") |
| `eyebrow` | string | "Entry to the GT range" |
| `subtitle` | string | "Compact soul, uncompromising quality" |
| `description` | text | One paragraph |
| `order` | int | Tab order within its category |
| `heroImage` | image | Product cutout, transparent background |
| `engineImage` | image | Optional, offset below the hero cutout |
| `supplier` | ref | → `supplier` |
| `specs` | list\<spec\> | §2.2 — technical card |
| `specGroups` | list\<specGroup\> | §2.3 — accordions |
| `gallery` | list\<image\> | §2.4 |
| `price` | money, nullable | **Currently hidden.** Keep nullable; do not render when empty |

### 2.2 `spec` (technical card cell)

`{ label: string, value: string, order: int }`

Values carry their own units as strings (`"6.99 m"`, `"1049 cc"`) — do not model
units separately. The card is a lattice grid, so any count works, but 6–9 reads
best.

### 2.3 `specGroup` (accordion)

`{ title: string, order: int, entries: list<{ label, value? }> }`

- The **count badge is derived** from `entries.length` — never a stored field.
- RIBs currently: Console & seating (5), Deck & structure (6), Systems (4),
  Equipment & accessories (9), Finishes (4).
- Jet skis: Technical, Features.
- All groups render closed.

### 2.4 Gallery images

`{ file: image, alt: string, order: int }`

Six per product today (`g23-1…6` etc.). The grid is 6/3/2 columns by viewport,
so **multiples of 6 keep every row complete** — validate that the count is a
multiple of 6, or accept a ragged final row.

Each image needs two derivatives:

| Derivative | Spec | Used by |
|---|---|---|
| Grid | 1:1 crop, ~800px, WebP q≈85 | Gallery cell (`object-fit: cover`) |
| Full | Original aspect, max 2600px long edge, WebP q≈90 | Lightbox (`object-fit: contain`) |

The current build ships a single derivative used for both, which is why the grid
crop and the lightbox frame differ visually. Generating both properly is a real
improvement.

### 2.5 `careTier`

`{ title, body, icon, order }` — six entries: Delivery, Setup, Upkeep,
Maintenance, Technical support, Lifetime care. Icons render `#F47F4B`.

### 2.6 `faq`

`{ question, answer, order }` — five entries, all closed by default.

### 2.7 `siteSettings` (singleton)

```
enquiryEmail      "info@duomarine.mt"
instagramUrl      "https://www.instagram.com/duomarine.mt/"
heroHeadline      "The waters, on your own terms"
heroImageDesktop  image
heroImageMobile   image        # separate crop, more sea, less cliff
lifestyleImage    image        # parallax band above the FAQ
footerNote        string
```

### 2.8 `supplier`

`{ name, logoLight, logoDark, url }` — MV Marine and Yamaha. `logoDark` (pure
white) is required for the blue jet-ski section.

---

## 3. Enquiry endpoint

Only build this if you are replacing the `mailto:` CTAs with a form. A form
that fails silently is worse than opening a mail client.

`POST /api/enquiry`

```json
{
  "name": "string, required, 2..80",
  "email": "string, required, RFC-validated",
  "phone": "string, optional",
  "productId": "slug, optional",
  "message": "string, required, 10..2000",
  "consent": "boolean, must be true",
  "token": "captcha token, required"
}
```

Responses: `202` accepted · `400` validation (field-level errors) ·
`429` rate-limited · `5xx` never surfaced raw to the visitor.

Behaviour:

1. Validate server-side. Never trust the client.
2. Verify the captcha (Turnstile or hCaptcha — invisible, no puzzle).
3. Rate-limit: 3/min and 20/hour per IP.
4. Send via a transactional provider (Resend, Postmark, SES) — **not** SMTP from
   the function. `From` must be an authenticated domain address; put the
   visitor's address in `Reply-To`.
5. Send the visitor an acknowledgement naming the product they asked about.
6. Persist the enquiry (or forward to a CRM). An enquiry that exists only in an
   inbox will be lost.
7. Log failures with the payload so nothing is silently dropped.

DNS: SPF, DKIM and DMARC on `duomarine.mt` before launch, or acknowledgements
land in spam.

GDPR (Malta / EU): explicit consent checkbox, linked privacy notice, stated
retention period, and a documented deletion path.

---

## 4. CMS options

| Option | Fit | Trade-off |
|---|---|---|
| **Sanity** | Best structured-content fit; handles the nested spec groups cleanly; strong image pipeline | Schema is code; needs a developer for model changes |
| **Payload** | Self-hosted, Postgres, full control | You operate it |
| **Contentful / Storyblok** | Non-technical friendly, visual preview | Cost scales; nested repeaters are clumsier |
| **Git-based (Decap, TinaCMS)** | Content in the repo, no external service, free | Editing UX is weakest for image-heavy galleries |
| **Webflow** | Client can edit visually | Would mean abandoning this build entirely — the motion and lightbox behaviour would have to be rebuilt within Webflow's limits, and some of it can't be |

Recommendation: **Sanity + Astro**. Astro ships almost no JavaScript by default,
which suits a page whose interactivity is four tab groups, some accordions and a
lightbox. The current inline-style approach ports to Astro components directly.

---

## 5. Build pipeline

1. CMS webhook triggers a rebuild on publish.
2. Generate image derivatives (§2.4) — Sharp, or the CMS's own CDN transforms.
3. Emit WebP with dimensions inlined to prevent layout shift.
4. Keep the boot-screen script inline in `<head>`, unbundled — it must run before
   anything else paints (see `frontend-spec.md` §3).
5. Fingerprint asset filenames; long-cache them (`_headers` in `site/` already
   sets `max-age=31536000, immutable` for `/assets/*` and no-cache for the page).

Budget to hold: **< 1.2 MB** initial transfer excluding lazy gallery images;
LCP < 2.5s on 4G. The current static build is ~3.8 MB first load, dominated by
self-hosted font weights — subsetting Barlow to the weights actually used is the
single biggest win available.

---

## 6. What must not regress

Non-negotiables carried over from the signed-off build:

- First painted frame is `#071B2B`. No white flash, ever.
- Boot screen waits for fonts and above-fold image **decode**, min 820ms,
  hard cap 3000ms.
- Gallery images lazy-load; only the visible product's photos are fetched.
- Lightbox opens the photo that was clicked, uncropped.
- Accordions closed on load; counts derived from entry length.
- Tab state is per-section.
- Every hover state is `#F47F4B`.

Any rebuild should be checked against `site/index.html` on these points
specifically — each one was a reported defect at some stage.

---

## 7. Repository setup

```
duo-marine/
├── README.md
├── frontend-spec.md
├── backend-spec.md
└── site/
    ├── index.html
    ├── support.js
    ├── _headers
    └── assets/
        ├── fonts/          Barlow + Funnel Sans (TTF)
        ├── gallery/        24 product photos (WebP)
        ├── *.webp          hero + product cutouts
        └── *.svg           logos, wave mark
```

**Deploying the static site as-is:** publish directory `site`, no build command.
On Netlify, drag the *contents* of `site/` or set the publish dir; `_headers` is
picked up automatically.

Suggested `.gitignore`: `node_modules/`, `.DS_Store`, `dist/`, `.env*`.

Branching: `main` deploys production, PR previews for changes. Keep the specs
in the repo and update them with the code — a spec that drifts from the build is
worse than none.
