# Skill: AI SaaS Builder

## Purpose
Build, integrate, and productionize AI features into SaaS applications — covering LLM integration, streaming, prompt engineering, token management, billing by usage, rate limiting, and evaluation.

## When to Use
- Adding AI features (chat, summarization, code generation, classification) to an app
- Integrating OpenAI, Anthropic, Google Gemini, or open-source models
- Building a multi-tenant AI SaaS with per-user API key management
- Implementing streaming responses in a Next.js / Node.js app
- Managing AI costs and implementing usage-based billing

---

## Rules
- Never expose API keys to the frontend — all AI calls go through your backend.
- Always stream long AI responses — don't make users wait for the full response.
- Implement rate limiting per user/org before enabling AI features publicly.
- Store only what you need: if you don't need conversation history, don't persist it.
- Every AI feature needs a fallback when the model API is unavailable.
- Track token usage per user for billing and abuse prevention.

---

## Step-by-Step Workflow

### Step 1 — Model & Provider Selection
```
Choose model based on task:
- Complex reasoning / code generation: Claude Sonnet/Opus, GPT-4o
- Fast, cheap, simple tasks: Claude Haiku, GPT-4o-mini, Gemini Flash
- Embeddings: text-embedding-3-small (OpenAI), voyage-3 (Anthropic)
- Image understanding: GPT-4o, Claude 3.5 Sonnet
- Local / private data: Ollama with Llama 3.1, Mistral

Anthropic SDK (recommended for Claude):
- Latest: claude-sonnet-4-6 (balanced), claude-opus-4-7 (most capable), claude-haiku-4-5-20251001 (fast)
```

### Step 2 — Backend AI Route Setup

#### Next.js API Route with Streaming (Anthropic)
```typescript
// app/api/chat/route.ts
import Anthropic from '@anthropic-ai/sdk'

const client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })

export async function POST(req: Request) {
  const { messages, userId } = await req.json()

  // Rate limit check
  const allowed = await checkRateLimit(userId)
  if (!allowed) return new Response('Rate limit exceeded', { status: 429 })

  const stream = client.messages.stream({
    model: 'claude-sonnet-4-6',
    max_tokens: 1024,
    system: 'You are a helpful assistant.',
    messages,
  })

  // Track usage for billing
  stream.on('message', async (msg) => {
    await trackTokenUsage(userId, msg.usage.input_tokens, msg.usage.output_tokens)
  })

  return new Response(stream.toReadableStream())
}
```

#### OpenAI Streaming
```typescript
import OpenAI from 'openai'
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })

export async function POST(req: Request) {
  const { messages } = await req.json()

  const stream = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  })

  const readable = new ReadableStream({
    async start(controller) {
      for await (const chunk of stream) {
        const text = chunk.choices[0]?.delta?.content || ''
        controller.enqueue(new TextEncoder().encode(`data: ${JSON.stringify({ text })}\n\n`))
      }
      controller.close()
    },
  })

  return new Response(readable, {
    headers: { 'Content-Type': 'text/event-stream', 'Cache-Control': 'no-cache' },
  })
}
```

### Step 3 — Frontend Streaming Consumer

```typescript
// React hook for streaming AI responses
function useAIStream() {
  const [content, setContent] = useState('')
  const [isStreaming, setIsStreaming] = useState(false)

  const stream = async (messages: Message[]) => {
    setIsStreaming(true)
    setContent('')

    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ messages }),
    })

    const reader = response.body!.getReader()
    const decoder = new TextDecoder()

    while (true) {
      const { done, value } = await reader.read()
      if (done) break
      const text = decoder.decode(value)
      setContent(prev => prev + text)
    }

    setIsStreaming(false)
  }

  return { content, isStreaming, stream }
}
```

### Step 4 — Prompt Engineering
```
System prompt structure:
1. Role definition: "You are a [role] that helps users [task]."
2. Constraints: "Always respond in [format]. Never [boundary]."
3. Output format: Specify JSON schema, markdown structure, or plain text.
4. Examples (few-shot): Include 1-3 input/output examples for complex tasks.

Prompt optimization:
- Be specific about output format — it reduces post-processing.
- Use XML tags to separate sections in long prompts: <context>, <task>, <format>.
- For JSON output: "Respond with valid JSON only. No markdown fences."
- Temperature: 0 for deterministic tasks (classification, extraction), 0.7 for creative.
```

### Step 5 — Token & Cost Management
```typescript
// Estimate cost before calling
function estimateCost(inputTokens: number, model: string): number {
  const pricing: Record<string, { input: number; output: number }> = {
    'claude-sonnet-4-6': { input: 0.000003, output: 0.000015 },
    'claude-haiku-4-5': { input: 0.00000025, output: 0.00000125 },
    'gpt-4o': { input: 0.0000025, output: 0.00001 },
  }
  return inputTokens * (pricing[model]?.input ?? 0)
}

// Track usage in DB
async function trackTokenUsage(userId: string, inputTokens: number, outputTokens: number) {
  await prisma.aiUsage.create({
    data: { userId, inputTokens, outputTokens, model: 'claude-sonnet-4-6' }
  })
}
```

### Step 6 — Rate Limiting
```typescript
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(20, '1 m'), // 20 requests per minute per user
})

async function checkRateLimit(userId: string): Promise<boolean> {
  const { success } = await ratelimit.limit(userId)
  return success
}
```

### Step 7 — Multi-tenant API Key Management
```typescript
// Store encrypted customer API keys in DB (if letting users bring their own key)
import { encrypt, decrypt } from '@/lib/crypto'

// Resolve which API key to use
async function getApiKey(userId: string): Promise<string> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { encryptedApiKey: true }
  })

  if (user?.encryptedApiKey) {
    return decrypt(user.encryptedApiKey) // user's own key
  }
  return process.env.ANTHROPIC_API_KEY! // platform key (bill to platform)
}
```

---

## Commands to Run

```bash
# Install Anthropic SDK
npm install @anthropic-ai/sdk

# Install OpenAI SDK
npm install openai

# Test API connectivity
node -e "const a = new (require('@anthropic-ai/sdk').default)(); a.messages.create({model:'claude-haiku-4-5-20251001',max_tokens:10,messages:[{role:'user',content:'hi'}]}).then(r=>console.log(r.content[0].text))"

# Estimate token count (Anthropic)
# Use tiktoken for OpenAI models
npm install tiktoken
node -e "const {encoding_for_model}=require('tiktoken'); const e=encoding_for_model('gpt-4o'); console.log(e.encode('your prompt here').length)"

# Load test AI endpoint
wrk -t4 -c10 -d10s -s post.lua http://localhost:3000/api/chat
```

---

## Validation Checklist
- [ ] API keys are server-side only, never in frontend bundles
- [ ] Streaming is implemented for responses > 1 sentence
- [ ] Rate limiting per user is in place
- [ ] Token usage tracked per user per request
- [ ] Error handling for model API failures (503, rate limit, timeout)
- [ ] Prompt injection protection: user input is clearly demarcated
- [ ] Conversation history has a max token budget (prevent infinite context growth)
- [ ] Costs estimated and monitored
- [ ] No PII sent to model unless explicitly required and disclosed to user

---

## Final Response Format

```
## AI Feature Implementation Summary

**Model Used:** [model name + version]
**Feature:** [what AI capability was built]

**Architecture:**
- Frontend: [streaming UI component]
- Backend: [API route + SDK call]
- Storage: [what's persisted and why]

**Cost Estimate:**
- Per request: ~$X (at X input + X output tokens avg)
- Monthly at 1000 req/day: ~$X

**Rate Limits Configured:** [X req/min per user]

**Files Created/Modified:**
- [file] — [purpose]

**Open Issues / Next Steps:**
- [anything not yet implemented]
```
