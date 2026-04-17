# cloudflare-connect

Point a subdomain at your hosting platform without clicking through three dashboards and copy-pasting CNAME targets. Invoke it, say where you want the subdomain to go, and it drives the browser through both sides — the platform's custom-domain form AND the Cloudflare DNS record — end-to-end.

## Install

```
/plugin install cloudflare-connect@nope-marketplace
```

Requires the [N-O-P-E marketplace](https://github.com/N-O-P-E/nope-marketplace):
```
/plugin marketplace add n-o-p-e/nope-marketplace
```

## Requirements

- [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) configured
- Chrome browser running
- A Cloudflare account with the target zone already added (the skill does NOT register domains or create zones)
- An account on the hosting platform (Railway, Vercel)

## Supported platforms

- **Railway** — attach subdomain to a Railway service, grab the CNAME target, DNS-only proxy.
- **Vercel** — attach subdomain to a Vercel project, handle TXT verification challenges automatically.

More platforms (Fly, Netlify, Render, Cloud Run custom domains) can be added as new reference playbooks without changing the skill's core flow.

## How it works

```
Discovery → Planning → Execution → Verification
```

**Discovery** — Parses free-form intent ("point `app.example.com` at my Railway service 'glitches-api'"). Derives the Cloudflare zone from the FQDN. Asks only about fields it can't determine.

**Planning** — Presents a numbered list of every step. Flags manual hand-offs (sign-in, 2FA) upfront.

**Execution** — Opens the platform first, adds the custom domain, grabs the CNAME target. Then opens Cloudflare, adds the DNS record (DNS-only by default — both Railway and Vercel need unproxied DNS for their own Let's Encrypt certs). Returns to the platform and waits for the domain to flip from "pending" to "active".

**Verification** — Screenshots the final active state. Writes `.cloudflare-setup.md` to your project root with clickable links to both dashboards and the live subdomain.

## Defaults

- **Proxy mode:** DNS-only (grey cloud). Change to Proxied only if you know your hosting supports Cloudflare-issued certs with Full (strict) SSL.
- **Record type:** CNAME.
- **TTL:** Auto.

## Error handling

- **Zone not in Cloudflare** — stops with a direct link to add the zone first.
- **Subdomain record already exists** — prompts to update / keep / abort. Never silently overwrites.
- **TXT verification challenge** (common on Vercel) — adds the TXT record automatically, waits for the platform to verify, then continues with the CNAME.
- **"Domain already in use"** — stops and reports the conflict.
- **Session expired** — pauses, asks you to re-auth, resumes.

## After setup

Writes `.cloudflare-setup.md` with a verification table:

| What | Verify |
|---|---|
| Cloudflare DNS record | link to the DNS row |
| Platform domain status | link to the platform's domain page |
| Live subdomain | clickable URL |

## Scope

This skill deliberately does NOT do:
- Register domains
- Create Cloudflare zones (add-site flow)
- Configure page rules, redirects, Workers routes
- Multi-record batch operations (one subdomain per run)
