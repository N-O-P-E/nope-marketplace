# Vercel — Platform Playbook

Attach a custom domain to a Vercel project, handle TXT verification, and capture the CNAME target.

---

## URLs

| What | URL |
|---|---|
| Dashboard | `https://vercel.com/dashboard` |
| Project domains | `https://vercel.com/TEAM_SLUG/PROJECT_NAME/settings/domains` |
| Account domains | `https://vercel.com/dashboard/domains` |

Vercel URLs use human-readable team and project slugs. If you know them, navigate directly. Otherwise click through the dashboard.

---

## Attach a Custom Domain

1. Navigate to `https://vercel.com/dashboard`.
2. If not signed in: pause for manual sign-in, then confirm the dashboard loaded.
3. `take_snapshot` — find the project card. If ambiguous, ask the user which team + project.
4. Click into the project.
5. Click **Settings** (top nav).
6. Click **Domains** in the left sidebar.
7. In the **Add Domain** input, enter the full FQDN (`app.example.com`).
8. Click **Add**.

Vercel responds with one of two flows depending on how the domain is resolved today:

### Flow A: CNAME only

A banner shows:
```
Add a CNAME record with the following:
  Name: app
  Value: cname.vercel-dns.com
```

Capture the target (`cname.vercel-dns.com` — this is static for Vercel, but still read it rather than hard-coding). Go to Cloudflare, add the CNAME, return and wait for "Valid Configuration".

### Flow B: TXT verification required

If Vercel can't verify the domain via the CNAME alone (common when the apex also needs to be owned, or the domain has existing conflicting records), it asks for a TXT record:
```
Add the following TXT record:
  Name: _vercel
  Value: vc-domain-verify=<random string>
```

Handle in this order:
1. Capture the TXT name and value.
2. Switch to Cloudflare and add the TXT record (`references/cloudflare.md`).
3. Return to Vercel and click **Refresh** on the domain row.
4. Wait for the TXT check to pass.
5. Vercel then reveals the CNAME target — capture it.
6. Add the CNAME in Cloudflare.
7. Return and wait for "Valid Configuration".

---

## Capture the CNAME Target

Vercel's target is typically `cname.vercel-dns.com` or `cname-china.vercel-dns.com` (rare). Read it from the page rather than assuming.

Steps:
1. `take_snapshot` the domain detail panel.
2. Locate the displayed CNAME target.
3. Store it for the Cloudflare step.

---

## Verify the Domain Goes Valid

After the CNAME is in Cloudflare:

1. Return to the Vercel Domains page.
2. The domain row shows a status — first **Invalid Configuration**, then **Valid Configuration** once DNS resolves.
3. `wait_for` text "Valid Configuration" or a green check.
4. Timeout: 2 minutes. Vercel usually verifies in 10-30 seconds for fresh attaches.
5. `take_screenshot`.

Certificate issuance happens automatically after validation. There's no separate cert step for the user.

---

## Sign-In Flow

Vercel supports GitHub, GitLab, Bitbucket, and email.

1. Navigate to `https://vercel.com/login`.
2. Stop automation. Tell the user:
   > Please sign in to Vercel in the browser. Let me know when you're on the dashboard.
3. Wait for confirmation.
4. `take_screenshot` to verify you're on `/dashboard`.

---

## Pitfalls

- **Team scope:** Vercel accounts have personal scope + team scopes. The domain belongs to a specific project in a specific team/scope. If the user has multiple teams, list them and ask.
- **Existing domain on another account:** if the domain is already attached to another Vercel account (even a personal one from years ago), Vercel refuses with a conflict. The user must remove it from the other account first. Report the exact error and stop.
- **Apex vs subdomain:** if the user's FQDN is the apex (`example.com`), Vercel uses an A record (`76.76.21.21`) rather than a CNAME. This skill is scoped to subdomains — if apex is requested, stop.
- **Proxied Cloudflare incompatibility:** Vercel issues its own Let's Encrypt cert on verified domains. Proxied Cloudflare will break the cert handshake. Warn the user if they selected proxied.
- **Wildcard subdomains:** `*.example.com` requires a special flow on Vercel Pro+. Stop if requested — out of scope.
