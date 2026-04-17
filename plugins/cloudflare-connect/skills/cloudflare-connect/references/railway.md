# Railway — Platform Playbook

Attach a custom domain to a Railway service and capture its CNAME target.

---

## URLs

| What | URL |
|---|---|
| Dashboard | `https://railway.app/dashboard` |
| Project | `https://railway.app/project/PROJECT_ID` |
| Service settings (domains section) | `https://railway.app/project/PROJECT_ID/service/SERVICE_ID/settings/domains` |

Railway's URLs use opaque IDs, not human names. You'll usually navigate by clicking into the project from the dashboard rather than typing the URL.

---

## Attach a Custom Domain

1. Navigate to `https://railway.app/dashboard`.
2. If not signed in: pause for manual sign-in, then `take_screenshot` to confirm you're on the dashboard.
3. `take_snapshot` — find the project card for the user's project. If the user didn't name the project, list the visible projects and ask.
4. Click into the project.
5. Click the specific **service** (Railway projects contain multiple services — pick the one the user named).
6. Click the **Settings** tab.
7. Scroll to or navigate to the **Domains / Networking** section.
8. Under **Custom Domain**, click **+ Custom Domain** (label may be slightly different — look for a button that opens a text field).
9. Enter the full FQDN (`app.example.com`).
10. Confirm. Railway validates the format and responds with a CNAME target.

---

## Capture the CNAME Target

After submitting, Railway shows a banner / row with:
```
Please add a CNAME record pointing to <generated>.up.railway.app
```

The target is usually something like `abc123.up.railway.app` — specific to that project/service. **Always read it from the page** — never guess.

Steps:
1. `take_snapshot` the domains section.
2. Locate the new row for the subdomain.
3. Extract the `*.up.railway.app` value. Regex: `[a-z0-9-]+\.up\.railway\.app` (or similar).
4. Store this value — it's the `Target` for the Cloudflare CNAME.

---

## Verify the Domain Goes Active

After the Cloudflare record is saved:

1. Return to the same Railway settings → Domains page.
2. The row for the subdomain shows a status — initially **Pending**, then **Active** (green check) once Railway has verified DNS and issued a Let's Encrypt cert.
3. `wait_for` text "Active" or similar confirmation.
4. Timeout: 2 minutes. Most Railway activations are under 30 seconds.
5. `take_screenshot` of the active state.

If still pending after 2 minutes: note it and link the user to the domains page. Let's Encrypt issuance can occasionally take longer; it's not a skill failure.

---

## Sign-In Flow

Railway supports Google, GitHub, and email/password. First-run:

1. Navigate to `https://railway.app/login`.
2. Stop automation. Tell the user:
   > Please sign in to Railway in the browser. Let me know when you're on the dashboard.
3. Wait for the user to confirm.
4. `take_screenshot` to verify you're on `/dashboard`.

Subsequent runs reuse the session — skip straight to the dashboard.

---

## Pitfalls

- **Project vs service:** Railway projects contain multiple services (web app, database, cron worker). The custom domain goes on a specific **service**, not the project. If the user didn't specify, list services and ask.
- **Domain limits:** Free tier has limits on custom domains per project. If Railway refuses with a quota error, report it and stop — the user must upgrade.
- **Proxied Cloudflare incompatibility:** Railway issues its own Let's Encrypt cert and expects direct DNS. If the user selected "Proxied" in Cloudflare, warn them that Railway's cert issuance will fail and the domain will stay pending.
- **Wildcard domains:** Railway supports `*.example.com` wildcards but they require a special TXT verification flow. If the user's FQDN starts with `*`, stop — this skill is scoped to concrete subdomains.
