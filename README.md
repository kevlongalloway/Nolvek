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
public page is harvestable. The Worker holds the key and proxies the call.

Secrets are set with `wrangler secret put`, never committed. `.dev.vars` is gitignored for the same
reason.
