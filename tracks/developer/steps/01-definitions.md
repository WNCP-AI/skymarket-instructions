### 01 — Definition Documents Usage

Goal: Understand what spec docs are and how we use them in this repo.

Steps
1) Open the documentation index.
   - Cursor: type `@docs/README.md` and open.
2) Skim these first:
   - `@docs/PRD.md` (what we’re building)
   - `@docs/architecture/ARCHITECTURE.md` (how it’s built)
   - `@supabase/schema.sql` (data model)
3) Pin them in Cursor so `@` pulls them into context.

Cursor prompt
```
You are my context copilot. Read @docs/PRD.md, @docs/architecture/ARCHITECTURE.md, and @supabase/schema.sql. Summarize the 5 most implementation-relevant constraints I must not violate.
```

Outcome
- You know where specs live and how to keep them in your Cursor context during work.


