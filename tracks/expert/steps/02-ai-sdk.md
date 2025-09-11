### 02 — Vercel AI SDK (text/stream/embeddings)

Goal: Use the SDK for text generation and embeddings.

Steps
1) Non-streaming route
   ```ts
   import { openai } from '@ai-sdk/openai'
   import { generateText, convertToCoreMessages } from 'ai'
   export async function POST(req: Request) {
     const { messages } = await req.json()
     const result = await generateText({
       model: openai('gpt-4o-mini'),
       system: 'You are helpful.',
       messages: convertToCoreMessages(messages),
     })
     return new Response(result.text, { headers: { 'Content-Type': 'text/plain; charset=utf-8' } })
   }
   ```
2) Streaming route
   ```ts
   import { openai } from '@ai-sdk/openai'
   import { streamText, convertToCoreMessages } from 'ai'
   export async function POST(req: Request) {
     const { messages } = await req.json()
     const result = await streamText({ model: openai('gpt-4o-mini'), messages: convertToCoreMessages(messages) })
     return result.toTextStreamResponse()
   }
   ```
3) Embeddings helper (already present)
   ```ts
   import { embed } from 'ai'
   import { openai } from '@ai-sdk/openai'
   export async function createEmbedding(text: string) {
     const { embedding } = await embed({ model: openai.textEmbeddingModel('text-embedding-3-small'), value: text })
     return embedding
   }
   ```

Cursor prompt
```
Add a new API route /api/embed that accepts { text } and returns a JSON array of numbers using @lib/embedding.ts. Ensure it is server-only and typed.
```

Outcome
- Familiarity with AI SDK primitives in routes.

References
- [AI SDK generateText](https://ai-sdk.dev/docs/ai-sdk-core/generate-text)
- [AI SDK streamText](https://ai-sdk.dev/docs/ai-sdk-core/stream-text)
- [AI SDK embeddings](https://ai-sdk.dev/docs/ai-sdk-core/embeddings)


