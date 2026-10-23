# DUO MARINE — Website (Lite)

Static site, ready for Netlify (or any static host). Drag the whole `deploy` folder into Netlify → "Deploy manually", or connect it to a repo with no build command and publish directory = the folder root.

## Contents
- `index.html` — the site
- `support.js` — page runtime (required)
- `assets/` — images (optimised WebP), logos, fonts
- `_redirects` — Netlify rule so `/enquire` serves the enquiry form

## Enquiry page
- Live URL: `https://<your-domain>/enquire` (for QR codes / campaigns). Also opens from every Enquire button, and in-page at `#enquiry-form`.
- `/enquire` relies on `_redirects` (Netlify). On other hosts, add an equivalent rewrite of `/enquire` → `/index.html` (status 200).
- Model Enquire buttons pre-select that model.

## Enquiry form → email (action required)
Submissions are sent via **Web3Forms** (https://web3forms.com) to the inbox the access key is registered to. Each email includes name, email, phone, models of interest, message and source; subject `Website enquiry: <Name> — <Models>`; Reply goes straight to the customer.

1. Create an access key at web3forms.com registered to the receiving inbox (e.g. info@duomarine.mt) and confirm the verification email.
2. In `index.html`, at the top of `<head>`, replace the placeholder:
   ```html
   <script>window.DUO_WEB3FORMS_KEY = 'YOUR_WEB3FORMS_ACCESS_KEY';</script>
   ```
   (The key is meant to be public; it can only deliver to the registered inbox.)
3. Deploy and submit a test enquiry **on the live domain** (check spam). Optionally restrict the key to the domain in the Web3Forms dashboard.

Notes
- Opened locally / on localhost, the form shows success **without sending** — test on the live site.
- If sending fails (e.g. key missing), the visitor's mail app opens pre-filled to info@duomarine.mt, so no lead is lost.

### Optional
- Auto-reply to customers: Web3Forms paid plan, or Resend/Postmark/SendGrid via a Netlify Function.
- Push leads to a CRM / Google Sheet via Web3Forms integrations or webhooks.

## Other
- Host at the domain root (asset paths are relative; `/enquire/` redirects to `/enquire`).
- MV Marine links open in a new tab. Galleries preview six square photos; all photos open in the lightbox.
