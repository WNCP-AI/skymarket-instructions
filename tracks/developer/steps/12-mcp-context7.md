### 12 — MCP / Context7 in Cursor

Goal: Use Cursor’s context and Context7 MCP to inject authoritative, versioned docs into every task.

Install Context7 MCP (global)
```bash
npx -y @upstash/context7-mcp@latest
```

Configure Cursor MCP
Add to `~/.cursor/mcp.json` → “Add new global MCP server”.
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

How to use it
- Append "use context7" in prompts that need latest docs.
- Ask for specific libraries by name: Next.js, Supabase, Stripe Node, Stripe.js, Resend, Vercel AI SDK.

Cursor prompts (copy/paste)
```
Scan our code and pull the exact docs for these used APIs (use context7):
- Next.js App Router route handlers
- Supabase SSR auth for Next.js
- Stripe Checkout sessions + webhook signature verification
- Resend Node send API
- Vercel AI SDK generateText/streamText/embed
Return links and 3-line usage reminders for each.
```

```
For the task "wire checkout then send email on success", only use:
@app/api/orders/[id]/checkout/route.ts and @app/api/webhooks/stripe/route.ts.
Pull current Stripe docs (use context7) and validate our webhook logic against them; propose deltas.
```

Acceptance
- Context7 MCP installed and visible in Cursor Settings → MCP.
- At least one task completed with "use context7" pulls and cited links.

References
- [Model Context Protocol](https://modelcontextprotocol.io)
- [Context7 MCP](https://github.com/upstash/context7)


