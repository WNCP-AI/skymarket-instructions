### 08 — Supabase Account & Project Setup

Goal: Create a Supabase project and apply the schema.

Steps
1) Create project at https://supabase.com/dashboard.
2) Copy Project URL, anon key, service role key.
3) Apply schema: open SQL Editor → paste `@supabase/schema.sql` → Run.
4) Configure Auth redirect to `http://localhost:3000` (dev).

Cursor prompt
```
Generate a ready-to-paste `.env.local` with placeholders for NEXT_PUBLIC_SUPABASE_URL, NEXT_PUBLIC_SUPABASE_ANON_KEY, SUPABASE_SERVICE_ROLE_KEY.
```

Outcome
- Supabase is provisioned; schema exists; envs are ready.


