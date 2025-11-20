# AI Streaming with Vercel AI SDK

Complete working pattern for Next.js + Vercel AI SDK + Anthropic.

## Backend: API Route

```typescript
// app/api/chat/route.ts
import { anthropic } from '@ai-sdk/anthropic'
import { streamText, convertToModelMessages, UIMessage } from 'ai'

export const maxDuration = 30

export async function POST(req: Request) {
  const { messages } = await req.json()

  const result = streamText({
    model: anthropic('claude-3-5-haiku-20241022'),
    system: 'Your system prompt here.',
    messages: convertToModelMessages(messages),
  })

  return result.toUIMessageStreamResponse()
}
```

## Frontend: Chat Component

```typescript
'use client'

import { useChat } from '@ai-sdk/react'
import { DefaultChatTransport } from 'ai'

export default function ChatBox() {
  const { messages, sendMessage, status } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  })

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    const input = (e.target as HTMLFormElement).elements.namedItem('chat-input') as HTMLInputElement
    const value = input.value.trim()
    
    if (!value || status !== 'ready') return
    
    sendMessage({ text: value })
    input.value = ''
  }

  return (
    <div>
      {/* Messages */}
      <div>
        {messages.map((message) => (
          <div key={message.id}>
            <strong>{message.role}: </strong>
            {message.parts.map((part, index) =>
              part.type === 'text' ? <span key={index}>{part.text}</span> : null
            )}
          </div>
        ))}
      </div>

      {/* Input */}
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          name="chat-input"
          disabled={status !== 'ready'}
        />
        <button type="submit" disabled={status !== 'ready'}>
          Send
        </button>
      </form>
    </div>
  )
}
```

## Passing Additional Data

When you need to send context beyond messages:

```typescript
const [contextId, setContextId] = useState('item-123')

const { messages, sendMessage, status } = useChat({
  key: `chat-${contextId}`, // Reinitialize when context changes
  transport: new DefaultChatTransport({
    api: '/api/chat',
  }),
  body: {
    contextId,
    metadata: { /* your data */ },
  },
})
```

Backend receives it:
```typescript
const { messages, contextId, metadata } = await req.json()
```

## Message Structure

UIMessage format:
```typescript
{
  id: string
  role: "user" | "assistant"
  parts: [
    { type: "text", text: "message content" }
  ]
}
```

Access text content:
```typescript
const text = message.parts
  .filter(part => part.type === 'text')
  .map(part => part.text)
  .join('')
```
