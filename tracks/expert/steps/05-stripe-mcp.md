### 05 — Stripe MCP (Cursor)

Goal: Evaluate using Stripe MCP during development.

Steps
1) Watch a quick demo of Stripe MCP in Cursor (any demo works).
2) If enabling, configure it to your Stripe test account.
3) Use MCP to inspect objects (intents, charges) while testing checkout.

Cursor prompt
```
After completing a test checkout, fetch the latest PaymentIntent by metadata.booking_id and show status, amount, and latest charge receipt_url.
```

Outcome
- Faster debugging of payment flows without changing app code.


