### 13 — Stripe: Basic Purchase Flow

Goal: Trigger Stripe Checkout from an order and return to the order page.

Steps
1) Environment
   - `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...`
   - `STRIPE_SECRET_KEY=sk_test_...`
   - `STRIPE_WEBHOOK_SECRET=whsec_...` (from CLI or Dashboard)
   - Optional: `STRIPE_BOOKING_PRODUCT_ID=prod_...` to attribute a product in Checkout
2) Start dev: `npm run dev`
3) Create or open a booking → on `orders/[id]` click "Pay with Stripe Checkout"
4) For local webhooks, run Stripe CLI forwarding

Local commands
```bash
stripe login
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

Test cards
- 4242 4242 4242 4242, any future expiry, any CVC, any ZIP
- Insufficient funds: 4000 0000 0000 9995

What code runs
- Server action `startCheckout()` posts to `@app/api/orders/[id]/checkout/route.ts`
- That route creates a Checkout Session via `@/lib/stripe/server`
- Stripe redirects back to `/orders/[id]` (success/cancel)
- Webhook `@app/api/webhooks/stripe/route.ts` receives events and marks `bookings.payment_status`
- Resend emails are sent from the webhook on success

Cursor prompt
```
Walk me through the call chain when I click "Pay with Stripe Checkout" on /orders/[id]: where is the server action, which API route is called, and which fields on bookings are updated by the webhook?
```

Outcome
- Checkout launches, returns to order, and payment state updates.

References
- [Checkout Sessions](https://stripe.com/docs/checkout)
- [Webhook signing](https://stripe.com/docs/webhooks)
- [Stripe CLI listen](https://stripe.com/docs/stripe-cli#listen)


