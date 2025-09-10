## Skymarket Chat Assistant — PRD

### Summary
- Lower-right floating chat bubble, click-to-open. Loads UI and network only after first click (click-to-load).
- Uses Vercel AI SDK for chat, tools, and streaming; renders with AI Elements components; markdown/stream rendering via Streamdown.
- Persists conversation in `localStorage` with a versioned key; provides a clear/reset action.
- Visible on all screens by mounting the launcher in `app/layout.tsx`.
- Advanced: include a tool call example and a simple embeddings utility.

### Goals
- Provide a lightweight, always-available assistant without interrupting primary flows.
- Fast first interaction: TTFB for first streamed token < 500ms on typical broadband.
- Maintain UX continuity with persisted history; allow explicit reset.

### Non-Goals
- No server-side conversation persistence in first iteration.
- No multi-user or auth-bound chat history.
- No RAG or vector DB; only a simple embedding example is included for future RAG work.

### User Stories
- As a user, I see a chat bubble in the lower-right corner on every page.
- When I click the bubble, the chat panel opens and loads; I can send a message and see streaming responses.
- My chat history remains across page navigations and reloads (localStorage).
- I can clear the chat, which resets UI and storage.

### Functional Requirements
- UI
  - Floating bubble at bottom-right; toggles a panel at a fixed size on desktop, full-height on small screens.
  - Click-to-load: heavy code and network are deferred until first open.
  - Uses AI Elements components for conversation/message/response rendering.
  - Stream rendering with Streamdown.
- Behavior
  - Streamed assistant responses via Vercel AI SDK, cancelable.
  - Persist messages in `localStorage` under a namespaced key; clear action wipes storage and resets state.
  - Available globally by mounting in `app/layout.tsx`.
- Advanced
  - One tool call exposed in the server route (e.g., getWeather).
  - A simple `embed(text)` helper using AI SDK embeddings.

### Technical Requirements
- Next.js App Router (`app/`), TypeScript, Tailwind/shadcn already present.
- Vercel AI SDK packages: `ai`, `@ai-sdk/react`, `@ai-sdk/openai`, `@ai-sdk/ui`.
- AI Elements installed via CLI into `@/components/ai-elements/*`.
- Streamdown for streaming/markdown rendering in the chat body.
- API route: `app/api/chat/route.ts` returning `result.toAIStreamResponse()`.
- Local key: `"skymarket.chat.v1"`.

### Accessibility
- Focus trap when panel open, Escape closes, screen-reader-friendly labels, sufficient contrast.

### Telemetry (optional)
- Track: open/close, message send, stream start/end, clear action.

### Acceptance Criteria
- Bubble is visible on every page; opens panel on click; no network until first open.
- Messages stream token-by-token; cancel works; localStorage persistence works across reloads.
- Clear button removes storage and resets session.
- Tool call path exercised by a prompt like “weather in NYC”.
- Embedding helper returns a numeric vector for a sample input.

### References
- AI Elements: `https://raw.githubusercontent.com/vercel/ai-elements/refs/heads/main/README.md`
- AI SDK embeddings: `https://ai-sdk.dev/docs/ai-sdk-core/embeddings`
- AI SDK Chatbot UI: `https://ai-sdk.dev/docs/ai-sdk-ui/chatbot`
- Streamdown: `https://streamdown.ai/`

