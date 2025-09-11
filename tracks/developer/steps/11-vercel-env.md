### 11 — Vercel Account & Environment

Goal: Import repo and add environment variables.

Steps
1) Create/log into Vercel → “Add New…” → Import Git Repository.
2) On the Environment Variables screen, add:
   - Supabase: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
   - Stripe: `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, optional `STRIPE_BOOKING_PRODUCT_ID`
   - Resend: `RESEND_API_KEY`, `RESEND_FROM_EMAIL`
   - OpenAI: `OPENAI_API_KEY`
   - Mapbox: `NEXT_PUBLIC_MAPBOX_TOKEN`
3) After deploy, go to Project → Settings → Environment Variables to add/update and redeploy.
4) For webhooks, add `STRIPE_WEBHOOK_SECRET` from Dashboard → Developers → Webhooks.

Cursor prompt
```
Generate a one-screen checklist of envs grouped by provider with brief purpose for each, based on scanning process.env usage.
```

Outcome
- Project imported and envs configured.

Cursor prompt
```
Scan process.env usage across @lib and /app/api/* and output a table of (KEY, file usage, purpose, required) with recommendations for Development vs Production values.
```


