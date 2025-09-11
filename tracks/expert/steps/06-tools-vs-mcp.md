### 06 — AI SDK Tools vs MCP: Differences & Use Cases

Goal: Choose the right abstraction for each task.

Rules of thumb
- **AI SDK tools**: in-process, app logic, typed, runs inside your route handlers.
- **MCP**: external tool servers, great for infra/admin, observability, and vendor consoles.

Steps
1) List candidate tasks in your sprint (payments, emails, embeddings, analytics).
2) Assign each to AI SDK tools or MCP using the rules above.

Cursor prompt
```
Given our current codebase, map specific tasks to either AI SDK tools or MCP. Provide one-sentence justification per task and any security caveats.
```

Outcome
- Intentional usage of tools with fewer footguns.


