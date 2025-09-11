### 15 — Ship to Vercel

Goal: Deploy to Vercel and verify payments + emails work in staging.

Steps
1) Push to GitHub; import the repo into Vercel.
2) Add env vars in Vercel (see step 11) for all providers.
3) Deploy.
4) In Stripe Dashboard → Webhooks: add endpoint `https://<app>/api/webhooks/stripe` and copy signing secret into project env.

Cursor prompt
```
Generate a post-deploy verification checklist (routes to hit, sample order, webhook test, email receipt) tailored to the env vars currently set in Vercel.
```

Outcome
- Staging deploy with working checkout + emails.


