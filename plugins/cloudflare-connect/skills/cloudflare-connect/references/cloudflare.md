# Cloudflare DNS — Playbook

Navigate and drive the Cloudflare dashboard to add DNS records.

---

## URLs

| What | URL |
|---|---|
| Dashboard home (account selector) | `https://dash.cloudflare.com/` |
| Zone overview | `https://dash.cloudflare.com/?to=/:account/ZONE_NAME` |
| DNS records | `https://dash.cloudflare.com/?to=/:account/ZONE_NAME/dns/records` |
| Add site (out of scope — tell user) | `https://dash.cloudflare.com/add-site` |

Cloudflare routes through `?to=` paths; after sign-in the account ID is resolved automatically. If you know the account ID, you can also use `https://dash.cloudflare.com/ACCOUNT_ID/ZONE_NAME/dns/records` directly.

---

## Check Whether a Zone Exists

1. Navigate to `https://dash.cloudflare.com/`.
2. `take_snapshot` the site list.
3. Look for the zone name as a row. If found → proceed. If not → stop and tell the user to add the zone first.

---

## Check Whether a Record Already Exists

1. Navigate to the DNS records page for the zone.
2. `take_snapshot` the records table.
3. Search rows for the subdomain label (e.g. `app` for `app.example.com`) with the record type you plan to add.

If a matching record exists, do NOT overwrite. Use `AskUserQuestion`:

```
question: "A record for 'app.example.com' already exists pointing to 'old-target.com'. What should I do?"
header: "Existing record"
options:
  - label: "Update target"
    description: "Replace the old target with the new one"
  - label: "Keep existing"
    description: "Don't touch it — abort this run"
  - label: "Abort"
    description: "Stop, I'll handle it manually"
```

---

## Add a CNAME Record

1. Click **Add record** (button near the top of the records table).
2. A form appears with fields: Type, Name, Target (CNAME), Proxy status, TTL.
3. Use `fill_form` to set:
   - **Type:** CNAME
   - **Name:** the subdomain label only (`app`, not `app.example.com`). Cloudflare auto-appends the zone.
   - **Target:** the value captured from the platform (e.g. `cname.up.railway.app`).
   - **Proxy status:** per user choice. Default: DNS-only (grey cloud).
   - **TTL:** Auto.
4. Click **Save**.
5. `wait_for` the new record to appear in the table.
6. `take_screenshot`.

### Proxy status nuances

- **DNS-only (grey cloud):** traffic bypasses Cloudflare's proxy. Required when the hosting platform issues its own TLS cert (Railway, Vercel default).
- **Proxied (orange cloud):** traffic goes through Cloudflare. Requires the hosting platform to accept Cloudflare-issued certs, which typically means configuring the platform's SSL mode to "Full (strict)".

If the user chose proxied but the platform is Railway or Vercel, warn them explicitly before saving — those platforms can't issue their own certs for proxied domains and will show "pending" forever.

---

## Add a TXT Record (Verification Challenge)

Some platforms (Vercel in particular) require a TXT record to prove domain ownership before the CNAME activates.

1. On the DNS records page, click **Add record**.
2. Fill:
   - **Type:** TXT
   - **Name:** per the platform's instructions (e.g. `_vercel` for Vercel).
   - **Content:** the exact string the platform provided.
   - **TTL:** Auto.
3. Save.
4. Return to the platform and wait for it to tick "verified" (usually under 60 seconds).
5. Only then add the CNAME.

---

## Common Pitfalls

- **Name field auto-append:** Cloudflare appends the zone to whatever you type in the Name field. Typing `app.example.com` becomes `app.example.com.example.com`. Always use the bare label (`app`).
- **@ for apex:** if the user wants to map the apex domain (`example.com` itself) to a platform, the Name is `@` — but this skill is scoped to subdomains, so if you detect an apex, stop and tell the user this is out of scope.
- **Record type cycling:** the "Type" dropdown defaults to whatever was used last. Always re-select CNAME explicitly even if it looks preselected.
- **Save button disabled:** Cloudflare disables Save until all required fields validate. If Save is greyed out, `take_snapshot` and look for red error text under each field.

---

## After Saving

Cloudflare records propagate within seconds on Cloudflare's own edges. Third-party resolvers may take minutes. The hosting platform's "active" check is the authoritative signal — don't bother using `dig` or external DNS checkers from the skill.
