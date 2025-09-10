## Chat Assistant — Architecture

### Overview
Client-side bubble launcher mounted globally; opens a chat panel that streams responses from a Next.js API route using Vercel AI SDK. UI built with AI Elements, streaming content rendered with Streamdown. Conversation persisted in `localStorage`.

### Key Pieces
- `app/layout.tsx`: mounts `ChatLauncher` so it appears on all screens.
- `components/chat/ChatLauncher.tsx`: small floating button; click toggles `ChatWindow` and defers loading.
- `components/chat/ChatWindow.tsx`: chat UI using AI Elements components and Streamdown for streamed markdown.
- `app/api/chat/route.ts`: server endpoint; `streamText` with a demo tool call; returns `result.toAIStreamResponse()`.
- `lib/chat-storage.ts`: tiny localStorage wrapper (get/save/clear).
- `lib/embedding.ts`: simple `embed(text)` helper using AI SDK.

### Data Flow
1) User clicks bubble → lazy-mount `ChatWindow` (code-split).  
2) `useChat` sends UI messages to `app/api/chat` on submit.  
3) Server streams tokens → client renders incrementally via Streamdown.  
4) `messages` synced to `localStorage`; clear wipes storage and resets `sessionId` to re-init `useChat`.

### API Route (App Router)
```ts
// app/api/chat/route.ts
import { z } from 'zod'
import { openai } from '@ai-sdk/openai'
import { streamText, tool, convertToCoreMessages } from 'ai'

export async function POST(req: Request) {
  const { messages } = await req.json()

  const result = await streamText({
    model: openai('gpt-4.1-mini'),
    system: 'You are a helpful Skymarket assistant.',
    messages: convertToCoreMessages(messages),
    tools: {
      getWeather: tool({
        description: 'Get simple weather reading for a city',
        parameters: z.object({ city: z.string() }),
        execute: async ({ city }) => ({ city, temperatureF: 72, condition: 'Sunny' }),
      }),
    },
  })

  return result.toAIStreamResponse()
}
```

### Chat UI (client)
```tsx
// components/chat/ChatWindow.tsx
'use client'
import { useEffect, useMemo, useState } from 'react'
import { useChat } from '@ai-sdk/react'
import { Streamdown } from 'streamdown'
import { Conversation, ConversationContent } from '@/components/ai-elements/conversation'
import { Message, MessageContent } from '@/components/ai-elements/message'
import { Response } from '@/components/ai-elements/response'

const STORAGE_KEY = 'skymarket.chat.v1'

export function ChatWindow({ onClose }: { onClose: () => void }) {
  const [sessionId, setSessionId] = useState(() => Math.random().toString(36).slice(2))
  const initial = useMemo(() => {
    if (typeof window === 'undefined') return []
    try {
      return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    } catch {
      return []
    }
  }, [])

  const { messages, input, setInput, append, isLoading, stop } = useChat({
    id: sessionId,
    api: '/api/chat',
    initialMessages: initial,
  })

  useEffect(() => {
    try {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(messages))
    } catch {}
  }, [messages])

  const clear = () => {
    try { localStorage.removeItem(STORAGE_KEY) } catch {}
    setSessionId(Math.random().toString(36).slice(2))
  }

  return (
    <div className="flex h-full w-full flex-col">
      <div className="flex items-center justify-between border-b px-3 py-2">
        <div className="font-medium">Assistant</div>
        <div className="flex gap-2">
          {isLoading ? (
            <button onClick={stop} className="text-sm">Stop</button>
          ) : null}
          <button onClick={clear} className="text-sm">Clear</button>
          <button onClick={onClose} className="text-sm">Close</button>
        </div>
      </div>

      <Conversation className="flex-1 overflow-y-auto p-3">
        <ConversationContent>
          {messages.map((m) => (
            <Message key={m.id} role={m.role as any}>
              <MessageContent>
                <Streamdown>{m.content}</Streamdown>
              </MessageContent>
            </Message>
          ))}
        </ConversationContent>
      </Conversation>

      <form
        onSubmit={(e) => {
          e.preventDefault()
          if (!input.trim()) return
          append({ role: 'user', content: input })
        }}
        className="flex gap-2 border-t p-3"
      >
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type a message..."
          className="flex-1 rounded border px-3 py-2"
        />
        <button className="rounded bg-black px-3 py-2 text-white">Send</button>
      </form>
    </div>
  )
}
```

### Launcher and Global Mount
```tsx
// components/chat/ChatLauncher.tsx
'use client'
import dynamic from 'next/dynamic'
import { useState } from 'react'

const ChatWindow = dynamic(() => import('./ChatWindow').then(m => m.ChatWindow), {
  ssr: false,
})

export function ChatLauncher() {
  const [open, setOpen] = useState(false)
  return (
    <div className="fixed bottom-4 right-4 z-50">
      {!open && (
        <button
          onClick={() => setOpen(true)}
          className="h-12 w-12 rounded-full bg-black text-white shadow"
          aria-label="Open chat"
        >
          💬
        </button>
      )}
      {open && (
        <div className="h-[480px] w-[360px] overflow-hidden rounded-xl border bg-white shadow lg:h-[560px] lg:w-[400px]">
          <ChatWindow onClose={() => setOpen(false)} />
        </div>
      )}
    </div>
  )
}
```

```tsx
// app/layout.tsx (excerpt)
import { ChatLauncher } from '@/components/chat/ChatLauncher'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        <ChatLauncher />
      </body>
    </html>
  )
}
```

### Embedding Helper
```ts
// lib/embedding.ts
import { embed } from 'ai'
import { openai } from '@ai-sdk/openai'

export async function createEmbedding(text: string) {
  const { embedding } = await embed({
    model: openai.textEmbeddingModel('text-embedding-3-small'),
    value: text,
  })
  return embedding
}
```

### Notes
- Use AI Elements CLI to install `conversation`, `message`, `response`, etc. into `@/components/ai-elements/*`.
- Stream rendering with `Streamdown` inside message content areas.
- Clearing chat rotates `useChat` `id` to force a new session.

### References
- AI Elements: `https://raw.githubusercontent.com/vercel/ai-elements/refs/heads/main/README.md`
- AI SDK Chatbot UI: `https://ai-sdk.dev/docs/ai-sdk-ui/chatbot`
- AI SDK Embeddings: `https://ai-sdk.dev/docs/ai-sdk-core/embeddings`
- Streamdown: `https://streamdown.ai/`

