## Local DB reset & seed

```bash
cd /Users/jdfiscus/dev/wncp-ai/skymarket-subpabase && supabase db reset | cat
```

Notes:
- Uses migrations in `supabase/migrations/`.
- Applies `supabase/seed.sql` automatically.
- Requires Supabase CLI (`brew install supabase/tap/supabase`).
- If no users exist, seed adds only `service_areas`. Create a user via app/auth to auto-create `profiles`, then re-run reset.

