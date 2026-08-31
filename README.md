# Nolvek

Agency site for [nolvek.online](https://nolvek.online), plus the Cloudflare Worker that backs the
chat assistant.

## Layout

```
site/      the website — this, and only this, is served to the public
worker/    Cloudflare Worker (model proxy for the chat widget) — never served
```

**`site/` is the published directory.** Render's static site is configured to publish `site`, not
the repo root. That separation is deliberate: anything outside `site/` is unreachable over HTTP, so
the Worker's source and system prompt never appear on the public site.

If the site ever 404s after a deploy, check that setting first — publishing the repo root would both
break the URLs and expose `worker/`.

## site/

Hand-written static HTML. No build step, no dependencies, no bundler — `site/index.html` carries its
own inline `<style>` and `<script>`. Open it in a browser, or serve the directory:

```sh
cd site && python3 -m http.server 8099
```

Asset paths are root-relative (`/favicon.ico`, `/wordmark.png`), which resolve correctly because
`site/` *is* the domain root once published.

Lead delivery goes to FormSubmit from the browser; there is no server involved in sending email.

## worker/

The chat widget cannot call a model provider directly — an API key in client-side JavaScript on a
public page is harvestable. The Worker holds the key and proxies the call. It is a model proxy only:
it never sends email. Leads go out through FormSubmit from the browser, on the same path the contact
form uses, and only after the visitor taps a confirm card.

Setup, deployment, the kill switch and tests are in [`worker/README.md`](worker/README.md).

Secrets are set with `wrangler secret put`, never committed. `.dev.vars` is gitignored for the same
reason.

### Turning the chat on

The widget ships inert. It only starts talking once two values are filled in:

1. `worker/wrangler.toml` → `GROQ_MODEL`, plus the secrets, then `wrangler deploy`.
2. `site/index.html` → `CFG.turnstileKey` in the chat module (the Turnstile **site** key, which is
   public and safe to commit), and `CFG.endpoint` if the Worker is not at `chat.nolvek.online`.

Until then the launcher is hidden and the page behaves exactly as it did before — the contact form
is untouched and remains the primary path.
