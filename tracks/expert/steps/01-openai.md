### 01 — OpenAI API (server-only)

Goal: Configure model access and keep secrets server-side.

Steps
1) Add `OPENAI_API_KEY` to `.env.local` or Vercel envs.
2) Use server routes for generation. Avoid client model calls.
3) Verify `/api/chat` returns text.

Cursor prompt
```
Review @app/api/chat/route.ts and suggest the minimal change to stream responses using AI SDK's streamText, keeping the response `toTextStreamResponse()`.
```

Outcome
- Secure, server-only LLM calls.


