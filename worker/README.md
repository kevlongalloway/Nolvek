# nolvek-chat

Cloudflare Worker backing the chat assistant on nolvek.online.

It is a **model proxy and nothing else.** It holds the API key — which cannot live in the static
site's JavaScript — enforces the abuse caps, and extracts a proposed lead from the model's reply.
**It never sends email.** The browser does that through FormSubmit, and only after the visitor taps a
confirm card.

## Endpoints

| | |
|---|---|
| `POST /session` | Turnstile token → signed, stateless session token |
| `POST /chat` | one assistant turn |
| `GET /health` | deploy check — booleans for which vars are set, never a value |

`/health` is the only route reachable without an `Origin` header, so it is the only one a browser
address bar can see. Every other path requires an allowlisted origin and returns a plain `forbidden`
otherwise — visiting `chat.nolvek.online` in a browser is *supposed* to say that.

A `405` carries the verb it actually received (`{"error":"method","received":"GET"}`). The widget only
ever sends `POST`, so a 405 means something in front of the Worker — most often a redirect rule on
the zone — rewrote the request; a 301/302 turns a `POST` into a `GET`.

There is no KV and no Durable Object. The turn counter lives inside an HMAC-signed token the client
carries, so it is unforgeable without any storage. The token is also bound to the issuing IP, so it
cannot be lifted and shared.

## Setup

```sh
npm install -g wrangler          # if you don't have it
wrangler login

wrangler secret put SESSION_SECRET      # any long random string, e.g. openssl rand -hex 32
wrangler secret put TURNSTILE_SECRET    # Cloudflare dashboard → Turnstile → your widget
wrangler secret put GROQ_API_KEY
wrangler secret put ANTHROPIC_API_KEY   # optional, but it's the failover — set both
```

Then set `GROQ_MODEL` in `wrangler.toml` from the **live** model list — Groq rotates and deprecates
slugs, and a stale one is a runtime 404:

```sh
curl -H "Authorization: Bearer $GROQ_API_KEY" https://api.groq.com/openai/v1/models
```

Uncomment the `[[routes]]` block, then `wrangler deploy`.

Finally, add the Turnstile **site key** to the widget in `site/index.html` (`CFG.turnstileKey`),
and set `CFG.enabled` to `true`.

## Before it goes live

**Set a spend cap in the provider dashboard.** The caps in this Worker are friction, not a billing
limit — a hard cap at Groq/Anthropic is the actual backstop.

## Variables vs secrets

`[vars]` in `wrangler.toml` is the source of truth. **A deploy replaces the Worker's plain-text
variables with that block**, so editing `GROQ_MODEL`, `PROVIDER` or `CHAT_ENABLED` in the dashboard
lasts only until the next build — change them in the file and deploy.

Secrets are the opposite: they are stored separately and a deploy never touches them. That is why
`SESSION_SECRET` and the API keys stay put while the vars snap back.

## Killing it

Set `CHAT_ENABLED = "false"` in `wrangler.toml` and deploy. `/session` and `/chat` then return 503
and the widget falls back to the contact form; nothing on the Render site is touched.

Flipping it in the dashboard works too and is faster in an emergency — but it is temporary, and the
next build will turn the assistant back on. Follow up with the file change.

## Tests

```sh
npm test
```

No network and no wrangler — the pure logic runs directly and the fetch handler is driven with a fake
env and stubbed `fetch`. Covers sentinel extraction (including the truncated-marker leak that
`max_tokens` can cause), lead validation, history normalisation, origin/method/kill-switch, Turnstile
pass and fail, session tampering, IP rebinding, the turn cap, oversized bodies, upstream 500s, and
rate limiting.

## Logging

One line per turn: session id, provider, latency, turn number, whether a lead was proposed, country,
token counts. **Never message content** — the privacy policy states transcripts are not stored, and
that has to stay true.
