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
## Related examples
[Hugo contact form](https://github.com/yanghuai123456/smartform-example-hugo) | [Jekyll contact form](https://github.com/yanghuai123456/smartform-example-jekyll) | [Gatsby contact form](https://github.com/yanghuai123456/smartform-example-gatsby)


## FAQ

### Why use this instead of Formspree?

Both SmartForm and Formspree let you POST a plain HTML form to a hosted
endpoint with no backend. SmartForm adds an AI spam filter (not just
honeypots), AI intent classification (`sales` / `support` / `inquiry`)
and high-value lead detection, with a free tier that includes the spam
filter. Formspree charges per submission; SmartForm's spam filter is
free on every plan.

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

