### 09 — Connect the App to Supabase

Goal: App reads envs and can auth/query against your project.

Steps
1) Create `.env.local` with your Supabase keys.
2) Start dev server: `npm run dev` → open http://localhost:3000.
3) Verify no Supabase errors in console.

Cursor prompt
```
Scan @lib/supabase/* and confirm env names and SSR usage. If any env is missing, output the exact keys to add to .env.local.
```

Outcome
- App successfully initializes Supabase client (SSR + middleware).


