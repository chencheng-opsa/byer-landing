# byer.ai — Setup

A bare-bones, two-service stack: **Vercel** (static hosting) + **Formspree** (lead capture). No database, no serverless functions, no secrets to manage. Budget: ~10 minutes to go live.

---

## What each file does

| File | Purpose |
|---|---|
| `index.html` | Landing page + popup lead-capture modal. Tailwind via CDN. |
| `privacy.html` | Privacy policy linked from the footer. |
| `vercel.json` | Vercel config — clean URLs + HTTP security headers (HSTS, CSP, etc.). |
| `README.md` | One-pager overview. |

---

## Step 1 — Formspree endpoint

Already wired: **`https://formspree.io/f/mojyvkzz`** (set in `index.html`, near the bottom of the `<script>` block as `FORMSPREE_ENDPOINT`).

If you ever need to change it — e.g. new form, new workspace — swap that one string. That's the only place it's used.

### Recommended Formspree settings

- **Allowed domains:** restrict to `byer.ai` (and `localhost` while you develop). This blocks other sites from submitting to your endpoint.
- **Spam filter:** leave on. The form already includes a `_gotcha` honeypot field that Formspree reads automatically — submissions where it's filled are silently dropped.
- **Autoresponder:** optional. It would send a confirmation via email, but since we don't collect email (only WhatsApp/Telegram handles), leave it off for now.

---

## Step 2 — Deploy to Vercel (5 min)

```bash
# From the byer-landing directory
npx vercel --prod
```

Follow the prompts. On first deploy: create a new project named `byer-landing`.

Alternatively, connect the GitHub repo in the Vercel dashboard — every push to `main` auto-deploys.

### Point byer.ai at Vercel

1. In Vercel dashboard → project → **Domains** → add `byer.ai` and `www.byer.ai`.
2. Vercel will show DNS records to add at your registrar:
   - `A` record for `byer.ai` → `76.76.21.21`
   - `CNAME` for `www` → `cname.vercel-dns.com`
3. Wait 5–10 min for DNS + SSL to propagate.

---

## Step 3 — Verify it works

1. Visit `https://byer.ai` (once DNS resolves) or your `*.vercel.app` URL.
2. Click any CTA (nav "Get early access", hero "Hire your agent", or footer "Get access"). The modal should pop up.
3. Fill the form. Submit.
4. Check:
   - Your Formspree inbox — a new entry should appear within seconds.
   - Your notification email — you should receive a message with the lead's `message`, `channel`, and `handle`.
5. The modal auto-closes ~1.8s after a successful submission.
6. Honeypot check: in DevTools, force-set the hidden `_gotcha` input to a value, submit, confirm Formspree logs nothing.

---

## Reading leads

**Formspree dashboard** is the fastest path: log in, open the form, sort by most recent, skim the `message` column, and ping promising leads on their stated WhatsApp/Telegram handle. Export to CSV when you want a batch view.

---

## When to graduate off Formspree

Trigger a migration when any of these hit:

- You're consistently over 50 submissions / month (free tier cap).
- You need to auto-route leads into CRM / Notion / Airtable.
- You need richer per-lead fields (UTM source, referrer, timestamps beyond what Formspree stores).
- Legal / compliance needs fuller audit trail than Formspree provides.

At that point, swap the endpoint for a Supabase Edge Function, Airtable webhook, or your own backend — everything else stays the same (the form already POSTs JSON).

---

## Security posture (v1)

Lean by design. What we have:

- **Honeypot field** (`_gotcha`) — blocks ~95% of naive bots. Formspree recognizes it automatically.
- **Formspree spam filter** — adds a second layer server-side.
- **HTTPS everywhere** (Vercel auto-provisions SSL).
- **Strict CSP + HSTS + X-Frame-Options: DENY** via `vercel.json`.
- **Domain allow-list** on Formspree — only byer.ai can submit.

What we don't have yet:

- No CAPTCHA. Add Cloudflare Turnstile or Formspree reCAPTCHA if bot submissions become an issue once paid traffic turns on.
- No rate limiting beyond Formspree's built-in floor. Good enough until you're running ads at scale.

---

## Troubleshooting

**Form POSTs but no entry in Formspree dashboard.** Check the browser console — you probably forgot to swap `FORMSPREE_ENDPOINT_URL` for your real endpoint. In that fallback mode the form logs the payload to the console and shows "Received locally."

**CSP blocks the POST.** Confirm `vercel.json`'s `connect-src` includes `https://formspree.io`. If you switch to a different provider, update both `connect-src` and `form-action` in the CSP.

**Someone says they submitted but I got nothing.** Check the Formspree spam folder inside the dashboard. If it's there, mark "not spam" to improve their filter.
