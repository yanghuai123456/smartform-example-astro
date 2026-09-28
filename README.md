# Astro contact form — Formspree alternative with AI spam filtering

Wire a contact form to [SmartForm AI](https://usesmartform.com) in an Astro site.

## What you're POSTing

The endpoint accepts a standard HTML form POST or JSON via AJAX. Two
kinds of fields:

**Your form fields** — `name`, `email`, `message`, whatever you
want. Every non-reserved field lands in your dashboard as a column in
the submissions table.

**Reserved fields** — names starting with `_` are interpreted by
the API, not stored:

| Field | Purpose |
|---|---|
| ``_gotcha`` | **Honeypot.** Keep it empty. Hidden from humans via CSS; bots fill it automatically. Any non-empty value silently drops the submission. Add this to every form. |
| ``_hp_email`` / ``_website`` / ``_url`` / ``_phone`` | Honeypot aliases for `_gotcha` (WordPress / WPForms / Contact Form 7 migrations). Same drop semantics. |
| ``_next`` | Same-origin URL to redirect to after a successful submission. Browser POST results in a 302 here. AJAX calls (with `Accept: application/json`) get the same value back as `next_url` in the JSON response. Only http(s) and in-site paths allowed. |
| ``_subject`` | Override the AI-generated email subject line. Max 200 chars; control characters stripped. |
| `X-Gotcha` header | Same as `_gotcha` for JSON requests where you can't add a hidden form field. |

Field names are Formspree-compatible — migrating from
`formspree.io/f/{form_id}` requires no renaming.

## Setup

1. Get a form ID at https://usesmartform.com/dashboard (8 chars, looks like `f_abc12345`).
2. Clone and run:
   ```bash
   git clone https://github.com/yanghuai123456/smartform-example-astro.git
   cd smartform-example-astro
   npm install
   cp .env.example .env
   # edit .env → PUBLIC_SMARTFORM_FORM_ID=f_your_real_id
   npm run dev
   ```
3. Open http://localhost:4321, submit, check your SmartForm dashboard.

## The form (static, no JS)

`src/pages/index.astro` posts a native HTML form straight to SmartForm's public endpoint.
The only thing it needs is your `form_id`. No JS, no API route, no server.

```astro
---
// src/pages/index.astro
const formId = import.meta.env.PUBLIC_SMARTFORM_FORM_ID;
---
<form action={`https://api.usesmartform.com/api/v1/f/${formId}`} method="POST">
  <input  name="name"    required />
  <input  name="email"   type="email" required />
  <textarea name="message" required></textarea>
  <input  type="text" name="_gotcha" tabindex="-1" autocomplete="off"
          style="position:absolute;left:-9999px" aria-hidden="true" />
  <button type="submit">Send</button>
</form>
```

The `_gotcha` field is a honeypot. Bots fill it; humans never see it. SmartForm silently discards
those submissions.

## Want JS-enhanced UX?

`src/components/ContactFormJS.astro` shows how to add a small client-side script that
posts as JSON and shows an in-page success/error message instead of navigating away.

## How the API works

- `POST https://api.usesmartform.com/api/v1/f/{form_id}` — JSON or form-data, no API key.
- Browser + no `_next`: 200 JSON.
- Browser + `_next` (same-origin): 302 redirect.
- AJAX (Accept: application/json or X-Requested-With: XMLHttpRequest): always 200 JSON.
- Response shape: `{ success, message, submission_id, is_spam, intent, next_url }`.

For the full contract, see the SmartForm docs at https://usesmartform.com/docs.

## Deploy

```bash
npm run build            # static output in ./dist
npx vercel --prod        # or `netlify deploy --prod`, `wrangler pages deploy ./dist`
```

Set `PUBLIC_SMARTFORM_FORM_ID` as an environment variable in your hosting dashboard.


## FAQ

### Why use this instead of Formspree?

At the basic level, SmartForm and Formspree are very similar: get a
form ID, POST a plain HTML form to a hosted endpoint with `_gotcha`
for spam filtering, and the API delivers the submission. The reserved
fields (`_gotcha`, `_next`, `_subject`, honeypot aliases) are
Formspree-compatible — a migration does not require renaming
anything.

The differences are operational, not API surface:

- **No email confirmation flow.** Formspree requires verifying your
  domain before submissions reach your inbox; SmartForm submissions
  land in your dashboard immediately.
- **AI spam filtering on the free tier.** Formspree's free tier uses
  only a honeypot field, which catches naive bots but lets semantic
  spam through. SmartForm applies AI-based classification by default,
  free of charge.
- **AI intent classification** (`sales` / `support` / `inquiry`
  / `spam`) on the Pro tier, for routing submissions without writing
  rules yourself.
- **No per-submission metering** on the basic plan.

### Is there a free tier?

Yes. AI spam filtering is enabled by default on every plan. AI intent
classification and high-value lead detection require a paid plan (Pro
or Business) — the dashboard enforces this and returns HTTP 402 if
you try to enable them on a free workspace.

### Do I need an API key?

No. The form posts directly to a public endpoint using only an 8-char
form ID, which is non-enumerable. The example also includes a hidden
`_gotcha` honeypot field so naive bots cannot submit.

### Does it work with static output?
Yes. Astro renders the form into static HTML and the form posts straight from the browser to the public endpoint — no Astro runtime, no API route, no server.

## Related examples
[Hugo contact form](https://github.com/yanghuai123456/smartform-example-hugo) | [Jekyll contact form](https://github.com/yanghuai123456/smartform-example-jekyll) | [Gatsby contact form](https://github.com/yanghuai123456/smartform-example-gatsby)


## License

MIT.

