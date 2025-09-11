### 04 — MCP as Tool (Cursor)

Goal: Use MCP servers as external tools when appropriate.

Steps
1) Decide which servers to enable (Stripe, Supabase, Vercel).
2) Scope permissions; prefer read-only on prod.
3) Use MCP for ops/admin tasks; keep app logic inside the repo.

Cursor prompt
```
Propose 3 tasks that are safer as MCP actions (read-only on prod) and 3 tasks that should stay as Next.js routes. Justify each.
```

Outcome
- Clear division of concerns between MCP and app code.


