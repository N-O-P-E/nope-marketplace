---
name: cloudflare-connect
description: Adds a subdomain to a Cloudflare-managed zone and wires it to a hosting platform (Railway, Vercel) via browser automation. Use when the user wants to point a subdomain at a Railway service, Vercel project, or other hosting platform. Also use when the user says "add a subdomain", "connect domain to Railway/Vercel", "point domain at my app", "create CNAME", "set up custom domain", or needs to attach a domain to a deployed service. Handles both sides end-to-end — the platform's custom-domain form AND the Cloudflare DNS record. Requires the Cloudflare zone to already exist (does not register domains or create zones). Defaults to DNS-only (grey cloud) proxy mode because Railway and Vercel issue their own Let's Encrypt certs.
allowed-tools:
  - AskUserQuestion
  - Read
  - Write
  - Bash
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__navigate_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__click
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__fill
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__fill_form
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_screenshot
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__take_snapshot
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__wait_for
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__list_pages
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__select_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__evaluate_script
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__new_page
  - mcp__plugin_chrome-devtools-mcp_chrome-devtools__press_key
---

# Cloudflare Connect — Subdomain + Hosting-Platform Wiring

Points a subdomain at a hosting platform (Railway, Vercel, more) by driving two browser dashboards: the platform's custom-domain form, and Cloudflare's DNS editor. End-to-end, click-by-click.

---

## Before You Start

Check these before opening the browser. If anything is missing, tell the user what to do first.

**You need:**
- [ ] Chrome browser running
- [ ] Chrome DevTools MCP extension active and connected (verify with `list_pages`)
- [ ] A Cloudflare account with the target zone already added
- [ ] An account on the hosting platform (Railway, Vercel, etc.)

**Check for an existing summary:** look for `.cloudflare-setup.md` in the project root. If it exists, read it — previously configured subdomains may already be listed there.

---

## What This Skill Does — And What It Doesn't

**In scope:**
- Attaching a subdomain to a Railway service, Vercel project, or other supported platform.
- Creating the matching CNAME (and optional TXT) record in Cloudflare DNS.
- Handling platform-side verification challenges (e.g. Vercel `_vercel` TXT).
- One subdomain per run.

**Out of scope (stop and tell the user):**
- Registering domains.
- Creating Cloudflare zones (add-site flow).
- Page rules, redirects, Workers routes, Cloudflare Tunnels.
- Bulk or batch operations — one subdomain per run.

---

## How This Works

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Discovery  │ ──▶ │   Planning   │ ──▶ │  Execution   │ ──▶ │ Verification │
│  (no browser)│     │  (no browser)│     │  (browser)   │     │ (screenshot) │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
       │                    │                    │                      │
  Parse intent         Build step list      Drive two          Wait for platform
  fill gaps            confirm with user    dashboards         to flip to "active"
```

Two strict phases: **Discovery** (no browser) and **Execution** (drive the browser). Never open the browser until the plan is confirmed.

---

## Phase 1: Discovery

### Step 1: Parse Intent

The user may invoke bare (`/cloudflare-connect`) or with free-form intent (`/cloudflare-connect point app.example.com at my Railway service "glitches-api"`). Extract whatever you can:

- **Subdomain FQDN** (e.g. `app.example.com`)
- **Platform** (railway / vercel / other)
- **Service or project identifier** on that platform

### Step 2: Ask for Missing Fields

Use `AskUserQuestion` for each missing input.

**Subdomain:**
```
question: "What subdomain should I set up?"
header: "Subdomain"
(free-form text — expect fully qualified like app.example.com)
```

**Platform:**
```
question: "Which hosting platform does it point to?"
header: "Platform"
options:
  - label: "Railway"
    description: "Railway service — cname.up.railway.app target"
  - label: "Vercel"
    description: "Vercel project — cname.vercel-dns.com target"
  - label: "Other"
    description: "Describe the platform — I'll adapt"
```

**Service/project identifier:** ask by name. If you can't see it in the user's words, ask after sign-in is done and you can list services from the dashboard.

**Proxy mode:**
```
question: "Proxy this record through Cloudflare (orange cloud), or leave it as DNS-only (grey cloud)?"
header: "Cloudflare proxy"
options:
  - label: "DNS-only (recommended)"
    description: "Grey cloud. Required for Railway + Vercel Let's Encrypt certs."
  - label: "Proxied"
    description: "Orange cloud. Only works if your platform supports Cloudflare Full (strict) SSL."
```

### Step 3: Derive the Zone

The registrable root of the FQDN is the Cloudflare zone. For `app.example.com` → `example.com`. For `api.staging.example.co.uk` → `example.co.uk` (handle multi-level TLDs via the Public Suffix List — when in doubt, ask the user).

### Step 4: Check Zone Existence

Navigate to `https://dash.cloudflare.com/` → check if the derived zone appears in the account's site list.

**If missing:** stop. Tell the user:
> The zone `example.com` isn't in your Cloudflare account. Add it first at https://dash.cloudflare.com/add-site, then re-run this skill.

### Step 5: Present the Plan

```
Here's what I'll do:

Platform side (Railway):
  1. Open Railway dashboard → service "glitches-api" → Settings → Domains
  2. Click "Add Custom Domain" and enter "app.example.com"
  3. Capture the CNAME target Railway shows

Cloudflare side:
  4. Open dash.cloudflare.com → zone "example.com" → DNS records
  5. Add CNAME:
     - Name: app
     - Target: (captured from step 3)
     - Proxy: DNS-only
  6. Save

Verify:
  7. Return to Railway, wait for "app.example.com" to show as "active"
  8. Take a final screenshot

Manual steps (you'll need to do these yourself):
  - Sign in to Railway (first run only)
  - Sign in to Cloudflare (first run only)
  - Any 2FA prompts

Ready to start?
```

Wait for explicit confirmation.

---

## Phase 2: Execution

For per-platform step details, read the relevant reference file(s):

| Platform | Reference file |
|---|---|
| Railway | `references/railway.md` |
| Vercel | `references/vercel.md` |
| Cloudflare DNS | `references/cloudflare.md` |

### The Step Loop

```
For each step:
  1. Navigate to the correct page
  2. take_snapshot for element UIDs
  3. Perform the action (click, fill)
  4. wait_for confirmation or take_snapshot to verify
  5. take_screenshot
  6. Report progress
  
  If something looks wrong:
  → take_snapshot, diagnose, retry once
  → If still wrong, stop and ask the user
```

### Manual Steps — CRITICAL

**Always stop automation for:**
- Sign-in screens on any dashboard.
- 2FA / security challenges.
- Any "Verify it's you" prompts.

Tell the user what screen they should see, what to do, then wait and `take_screenshot` to confirm before continuing.

### Execution Order

**Platform FIRST, then Cloudflare.** You need the CNAME target from the platform before you can create the DNS record. Don't try to predict the target — always read it from what the platform shows after you click "Add".

### Capturing the CNAME Target

After the platform accepts the subdomain, it reveals a target like `cname.up.railway.app` or `cname.vercel-dns.com`. Use `take_snapshot` to locate the string in the page, then extract it. If there are multiple acceptable targets shown (some platforms show both CNAME and A options), prefer the CNAME.

If the platform also shows a TXT verification challenge (common on Vercel), note the TXT record name and value — you'll add it to Cloudflare before the CNAME.

### Error Handling

| Condition | What to do |
|---|---|
| Zone not in Cloudflare | Stop. Link to `https://dash.cloudflare.com/add-site`. |
| Subdomain record already exists in Cloudflare | Don't overwrite. Use `AskUserQuestion` with options: (a) update target, (b) keep existing, (c) abort. |
| Platform shows TXT verification challenge | Add the TXT record to Cloudflare first. Wait for the platform to tick "verified". Then continue with the CNAME. |
| Platform reports "domain already in use" | Stop. Report the conflict verbatim. Ask the user how to proceed. |
| Auth session expired mid-run | Pause. Ask user to re-auth. Resume from the current step. |
| Platform status still "pending" after 2 minutes | Stop waiting. Note it in the summary. Link the user to the platform's domain page so they can check later. |

### Progress Reporting

After each major step:
```
[2/7] Added "app.example.com" to Railway service "glitches-api"
       → Target captured: cname.up.railway.app
       → Moving to: Open Cloudflare DNS records
```

---

## Phase 3: Verification & Summary

### Verify

After the CNAME is saved in Cloudflare:

1. Return to the platform's domain page.
2. `wait_for` text indicating the domain is active (e.g. "Active", "Verified", or the green check icon).
3. Timeout: 2 minutes. If still pending, note it and continue — don't block forever on cert issuance.
4. `take_screenshot` of the final state.

### Write the Summary

Write `.cloudflare-setup.md` in the project root. If the file exists, append a new entry; don't overwrite prior runs.

```markdown
# Cloudflare Connect Summary
Generated: YYYY-MM-DD

## app.example.com

- **FQDN:** app.example.com
- **Zone:** example.com
- **Platform:** Railway — service "glitches-api"
- **CNAME target:** cname.up.railway.app
- **Proxy:** DNS-only
- **Status at completion:** Active

### Verify

| What | Link |
|---|---|
| Cloudflare DNS record | https://dash.cloudflare.com/?to=/:account/example.com/dns/records |
| Railway domain | https://railway.app/project/.../service/.../settings/domains |
| Live | https://app.example.com |
```

Tell the user:
> Written `.cloudflare-setup.md` to project root. The live URL is https://app.example.com — TLS cert may take a couple more minutes to finalize if it's a fresh attach.

---

## Browser Session Management

- Pages may close between runs. Call `list_pages` before navigating an existing page. If none, use `new_page`.
- Cloudflare and the platform dashboards are separate origins — sign-in to each is independent.
- The Chrome DevTools MCP session persists cookies across runs, so sign-in is only needed on first run.

---

## Important Principles

1. **Never guess the CNAME target.** Always read it from the platform's "Add custom domain" result page.
2. **Never silently overwrite** an existing DNS record. Always prompt.
3. **Platform first, Cloudflare second** — order matters because the target is captured from the platform.
4. **Default to DNS-only.** Both Railway and Vercel need unproxied DNS for their Let's Encrypt certs. Only go proxied if the user explicitly asks.
5. **Screenshot every major step.** If you can't see it worked, it didn't work.
6. **One subdomain per run.** If the user wants multiple, run the skill multiple times — don't batch.
7. **Write the summary file always.** Even on partial completion — note which step failed and what state the resource is in.
