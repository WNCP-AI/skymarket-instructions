### 10 — Deploy to Vercel (Basics)

Goal: Understand Next.js on Vercel and required configs.

Steps
1) Read: https://vercel.com/docs/frameworks/nextjs
2) Confirm Node runtime for webhooks where needed.

Cursor prompt
```
Check /app/api/webhooks/stripe/route.ts and confirm runtime and dynamic config are set for Vercel. If not, propose the minimal edit.
```

Outcome
- You know the deployment assumptions and function runtime.

Deep dive
- Webhook route sets:
  - `export const runtime = 'nodejs'`
  - `export const dynamic = 'force-dynamic'`
- This ensures raw-body access for Stripe signature verification and disables static optimizations.

References
- [Next.js on Vercel](https://vercel.com/docs/frameworks/nextjs)
- [Functions: Runtime](https://vercel.com/docs/functions/runtimes)


