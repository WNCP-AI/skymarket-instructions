### 14 — Resend Email on Payment

Goal: Email consumer and provider when payment succeeds.

Steps
1) Environment
   - `RESEND_API_KEY` (project API key)
   - `RESEND_FROM_EMAIL` (verified domain or sending address)
2) Run dev with Stripe CLI webhooks (see step 13)
3) Complete a test payment; verify two emails are sent (consumer and provider)
4) If emails don’t arrive, check Resend dashboard events and verify domain/sender

Cursor prompt
```
Locate where emails are sent after payment succeeds and list the recipient selection logic. If missing, draft a minimal HTML template with amount, order id, and title.
```

Outcome
- Transactional emails on success.

Where it happens
- `@app/api/webhooks/stripe/route.ts` on `payment_intent.succeeded` and `checkout.session.completed`
- `@lib/resend.ts` wraps the Resend client and sends via `emails.send`

Cursor prompts
```
Inspect @app/api/webhooks/stripe/route.ts and list all event types that trigger email sends. Propose a tiny HTML template function that takes (title, amount, bookingId) and returns an accessible email body.
```

```
If sending fails (Resend returns an error), suggest the minimal retry/log pattern we should apply in this route without introducing a job queue.
```

References
- [Resend Quickstart](https://resend.com/docs)


