### 04 — Cursor Basics (context)

Goal: Use `@` context with files, URLs, and images.

Steps
1) In a chat, type `@` then select project files: `docs/*`, `supabase/schema.sql`.
2) Paste a URL (Stripe/Supabase docs) and a screenshot if helpful; reference them with `@`.
3) Ask Cursor to restate the task using only provided context.

Cursor prompt
```
Using ONLY @docs/PRD.md and @supabase/schema.sql, restate the MVP payment flow in 5 sentences and list the exact tables and statuses involved.
```

Outcome
- You can reliably steer generation with concrete context.


