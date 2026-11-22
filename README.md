# StanLite
Link-in-bio storefront for creators. Next.js 15 + Tailwind 4 + SQLite. Core loop: sign up → add products → share `/handle` → track clicks.

## Run locally
```bash
npm install
cp .env.example .env     # fill in values (Next loads .env automatically)
npm run dev              # http://localhost:3000
```
## Environment variables
| Var | Purpose |
|---|---|
| AUTH_SECRET | Long random string signing session cookies (`openssl rand -hex 32`) |
| APP_URL | Public base URL (Stripe redirects) |
| DB_PATH | SQLite file path (point at a persistent volume in prod) |
| ANTHROPIC_API_KEY / ANTHROPIC_MODEL | Powers "Write with AI" (Pro only) |
| STRIPE_SECRET_KEY, STRIPE_PRICE_ID | Test-mode key + a recurring Price (e.g. $19/mo) |
| STRIPE_WEBHOOK_SECRET | From `stripe listen` locally, or the dashboard endpoint in prod |

## Test payments (Stripe test mode)
1. Create a product with a $19/month price in Stripe test mode; put its `price_...` id in `.env`.
2. `stripe listen --forward-to localhost:3000/api/webhook` and copy the `whsec_...` into `.env`.
3. Click **Upgrade to Pro**, pay with card `4242 4242 4242 4242`. The webhook flips your plan to `pro`.
No Stripe yet? Unlock Pro locally: `sqlite3 data.db "update users set plan='pro'"`.

## Deploy
SQLite needs a persistent disk, so use a host with volumes (Railway, Fly.io, Render) — not Vercel.
1. Push to GitHub, create a service from the repo (build `npm run build`, start `npm start`).
2. Attach a volume at `/data`, set `DB_PATH=/data/data.db`, plus all env vars above with `APP_URL` = your live URL.
3. In Stripe, add a webhook endpoint `https://YOUR_URL/api/webhook` for `checkout.session.completed` and `customer.subscription.deleted`.
4. To scale or use Vercel, swap `lib/db.js` for Postgres/Turso.

## Not included yet
Product editing, password reset, email verification, real checkout for creators' products (buyers currently go to the creator's own link).
