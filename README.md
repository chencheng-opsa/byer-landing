# byer.ai — Landing page

Landing page for Byer, the conversational AI buying agent that wholesalers &amp; resellers chat with on WhatsApp and Telegram.

```
index.html     ← the landing page (chat-style lead capture)
privacy.html   ← privacy policy
vercel.json    ← security headers + hosting config
SETUP.md       ← 10-minute setup (Formspree + Vercel)
```

## Stack (v1)

Lean on purpose — two services, no backend code.

- **Vercel** for static hosting + HTTPS + security headers
- **Formspree** for lead capture (free tier, 50/month)

## Deploy

```bash
npm i -g vercel
vercel --prod
```

Then add `byer.ai` as a custom domain in the Vercel dashboard.

## One placeholder in `index.html` to swap

- `FORMSPREE_ENDPOINT_URL` → your Formspree form URL (`https://formspree.io/f/xxxxxxxx`)

Until it's swapped, the form runs in dev mode — logs submissions to the browser console instead of POSTing.

Full instructions (Formspree signup, Vercel deploy, DNS, verification): see [`SETUP.md`](./SETUP.md).
