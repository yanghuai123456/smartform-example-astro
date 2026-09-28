# Astro contact form — Formspree alternative with AI spam filtering

Wire a contact form to [SmartForm AI](https://usesmartform.com) in an Astro site.

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

## License

MIT.
