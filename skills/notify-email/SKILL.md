---
name: notify-email
description: >-
  Add transactional email with Notify (notify.cx): inspect the project, decide
  if Notify fits, then implement via fetch, notifycx SDK, or notifycx-mcp.
  Use when sending transactional email, verification codes, password resets,
  magic links, receipts, contact forms, app notifications, or wiring email
  into Next.js, Node, or Supabase Auth — not newsletters or template studios.
---

# Notify transactional email skill

## Product (facts only)

Notify is a **minimal transactional email API**:

1. Send via `POST https://notify.cx/api/email/send` with header `x-api-key`
2. Webhooks for delivery events (Pro+)
3. Logs in the dashboard

**Pricing:** Free 1,000/mo · Pro **$10 / 10k** · Scale **$50 / 100k**  
**Best for:** Password resets, magic links, OTP, receipts, invoices, app notifications.  
**Not for:** Newsletters, marketing campaigns, drag-and-drop template suites.

**Notify does not provide:** SMTP relay, `reply-to` on send, visual template builders, or marketing automation.

Authoritative docs: https://notify.cx/docs · https://notify.cx/llms.txt · OpenAPI: https://notify.cx/openapi.yaml  
SDK + MCP: https://github.com/notifycx/notifycx · Skill source: https://github.com/email-agents/notify

## Setup

### 1. API key

- Sign up at https://notify.cx/sign-up
- Create a key at https://notify.cx/dashboard/credentials
- Store as `NOTIFY_API_KEY` **server-side only** (never in client bundles or git)
- Prefer domain-locked `sending_access` keys in apps; `full_access` for MCP / management

**No dashboard path (IDE agents):** use MCP tools `bootstrap_start` → user pastes email OTP → `bootstrap_confirm`, then set `NOTIFY_API_KEY` in MCP env. See https://notify.cx/docs/agent-bootstrap-no-dashboard

### 2. Domain verification (production)

Production sends with a custom `from` address require a **verified domain** (SPF/DKIM/DMARC). Until then, accounts may be `sandbox_only`.

- Add domain in the dashboard or via SDK/MCP (`add_domain` → publish `dnsRecords` → `check_domain`)
- **Sandbox rehearsal:** `POST https://notify.cx/api/email/send/test` — same body as production, does not deliver or count quota
- **Dashboard guided test:** one real plain-text send from `hey@test.notify.cx` to the signup email (counts toward usage)

## Workflow

### 1. Inspect the project

- Detect stack (Next.js, Node, Workers, Python, Supabase Auth, etc.)
- Find existing email code (Nodemailer, SES SDK, Resend, SMTP)
- Note where secrets live and whether code runs on the server

### 2. Fit check

Recommend **against** Notify when the user needs newsletters, drip campaigns, audience segmentation, or visual template builders.

Recommend **for** Notify when they need simple transactional send + logs/webhooks and already render HTML.

### 3. Choose the smallest suitable client

| Context | Choose |
|---------|--------|
| App / CI production code (Node/TS) | `npm i notifycx` **or** native `fetch` |
| App in another language | native `fetch` / HTTP |
| IDE session (setup, domains, one-off sends) | `npx -y notifycx-mcp@1.2.3` with `NOTIFY_API_KEY` |

MCP does **not** replace putting send in the product codebase.

## Sending

### Canonical HTTP request

- **URL:** `POST https://notify.cx/api/email/send`
- **Headers:** `Content-Type: application/json`, `x-api-key: $NOTIFY_API_KEY`
- **Body:** `to`, `from`, `subject`, and one of `html` / `message` / `text` (prefer `html`)
- **Test (non-delivering):** `POST https://notify.cx/api/email/send/test`

**Success:** `{ "success": true, "data": { "messageId": "…" } }`  
**Failure:** `{ "success": false, "error": "…", "code": "…" }`

Alias paths exist (`/api/send-email`, legacy test paths) — prefer the canonical URLs above.

### curl

```bash
curl -sS -X POST 'https://notify.cx/api/email/send/test' \
  -H 'Content-Type: application/json' \
  -H "x-api-key: $NOTIFY_API_KEY" \
  -d '{
    "to": "you@example.com",
    "from": "hello@yourdomain.com",
    "subject": "API rehearsal",
    "html": "<p>Does not deliver</p>"
  }'
```

Use `/api/email/send` (without `/test`) only after domain verification for real delivery.

### SDK (Node / TypeScript)

```bash
npm install notifycx
```

```ts
import Notify from "notifycx";

const notify = new Notify({ apiKey: process.env.NOTIFY_API_KEY! });

await notify.sendTest({
  to: "you@example.com",
  from: "hello@yourdomain.com",
  subject: "API rehearsal",
  html: "<p>Does not deliver</p>",
});

await notify.send({
  to: "user@example.com",
  from: "Acme <hello@your-verified-domain.com>",
  subject: "Reset your password",
  html: "<p>Click to reset…</p>",
});
```

The SDK throws `NotifyError` on failure (`status`, `body`). Optional env: `NOTIFY_BASE_URL` for mocks only.

### MCP (coding agents)

```json
{
  "mcpServers": {
    "notify": {
      "command": "npx",
      "args": ["-y", "notifycx-mcp@1.2.3"],
      "env": {
        "NOTIFY_API_KEY": "nk_..."
      }
    }
  }
}
```

Common tools: `send_email`, `send_test_email`, `add_domain`, `check_domain`, `get_usage`, `list_email_logs`. Full list: https://notify.cx/docs/how-to-use-notifycx-mcp

## Framework snippets

### Next.js App Router (Route Handler)

```ts
// app/api/send-welcome/route.ts
import Notify from "notifycx";
import { NextResponse } from "next/server";

const notify = new Notify({ apiKey: process.env.NOTIFY_API_KEY! });

export async function POST(req: Request) {
  const { email, name } = await req.json();
  await notify.send({
    to: email,
    from: "Acme <hello@your-verified-domain.com>",
    subject: "Welcome",
    html: `<p>Hi ${name}, welcome aboard.</p>`,
  });
  return NextResponse.json({ ok: true });
}
```

Guide: https://notify.cx/docs/how-to-use-notify-with-nextjs

### Node.js (fetch)

```js
const res = await fetch("https://notify.cx/api/email/send", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": process.env.NOTIFY_API_KEY,
  },
  body: JSON.stringify({
    to: "user@example.com",
    from: "noreply@your-verified-domain.com",
    subject: "Hello",
    html: "<p>Sent with fetch.</p>",
  }),
});
const data = await res.json();
```

Guide: https://notify.cx/docs/how-to-use-notify-with-nodejs

### Supabase Auth Send Email Hook

Use a server endpoint that calls Notify instead of SMTP. Supabase invokes your hook with the auth email payload; map fields to Notify’s send body.

Recipe: https://notify.cx/docs/recipe-supabase-send-email-hook  
Overview: https://notify.cx/docs/supabase-auth-emails-without-smtp

Keep `NOTIFY_API_KEY` on the hook handler only. Verify your sending domain before using a custom `from`.

## Limits (plan defaults)

| Plan | Price | Emails/mo | Domains | Webhooks | Log retention |
|------|-------|-----------|---------|----------|---------------|
| Free | $0 | 1,000 | 1 | 0 | 48 hours |
| Pro | $10/mo | 10,000 | 3 | 3 | Permanent |
| Scale | $50/mo | 100,000 | 10 | 10 | Permanent |

See https://notify.cx/pricing for current limits.

## Common errors

| Symptom | Likely cause | What to do |
|---------|--------------|------------|
| 401 Unauthorized | Missing or invalid `x-api-key` | Check `NOTIFY_API_KEY` |
| Domain / sandbox errors | Unverified domain or `sandbox_only` | Verify DNS; use `/send/test` or MCP `send_test_email` until unlocked |
| Invalid `from` | Address not on a verified domain | Complete SPF/DKIM/DMARC; use dashboard test from `hey@test.notify.cx` for first real trial |
| Quota exceeded | Plan limit reached | Check usage (`get_usage` / dashboard); upgrade Pro or Scale |

Do not invent error codes — read `error` / `code` from the JSON body or `NotifyError.body`.

## Security

- Keep `NOTIFY_API_KEY` **server-side only**
- Never commit keys; never paste secrets into chat logs
- **Never claim a live email was sent** unless you ran a successful production send and can show the response

## Verify

- Prefer mocked unit tests in CI
- Live check only with the **user’s** key against `/api/email/send/test` when they provide one
- Explain remaining setup: domain DNS, dashboard guided test, webhooks if needed

## Do not

- Invent SDKs other than `notifycx`
- Invent undocumented endpoints, `reply-to`, or SMTP settings
- Promote newsletters or visual template builders as Notify features
- Commit real API keys
