# SkyMarket AI Integration Documentation

Complete AI integration documentation for the SkyMarket drone service marketplace platform using OpenAI GPT-4 and advanced AI features.

## Overview

SkyMarket integrates artificial intelligence to enhance user experience through intelligent customer support, service matching, and operational automation. The AI system is built on OpenAI's GPT-4 models with plans for advanced features including embeddings, tool calling, and multi-modal capabilities.

**Current Implementation**:
- Basic chat assistant for customer support
- Fallback echo mode for development without API keys
- Foundation for advanced AI features

**Planned Features**:
- Service recommendation engine
- Intelligent matching between consumers and providers
- Automated customer support with tool calling
- Multi-modal content analysis (images, documents)
- Vector search for service discovery

## Architecture

### Core AI Components

```typescript
// AI SDK Integration
import { openai } from '@ai-sdk/openai'
import { generateText, convertToCoreMessages } from 'ai'

// Current model configuration
const MODEL = 'gpt-4o-mini'  // Cost-optimized for basic chat
const SYSTEM_PROMPT = 'You are a helpful Skymarket assistant.'
```

### Environment Configuration

```bash
# OpenAI Configuration
OPENAI_API_KEY=sk-...

# Future AI Features
OPENAI_ORGANIZATION_ID=org-...  # Optional organization ID
AI_MODEL_TEMPERATURE=0.7        # Response creativity
AI_MAX_TOKENS=500              # Response length limit
AI_CONTEXT_WINDOW=8192         # Context size

# Development
AI_DEBUG_MODE=true             # Enable debug logging
AI_FALLBACK_ECHO=true          # Echo mode without API key
```

## Current Implementation

### Chat Assistant Endpoint

**Location**: `/app/api/chat/route.ts`

```typescript
export async function POST(req: Request) {
  try {
    const { messages } = await req.json()

    // Fallback echo mode for development
    if (!process.env.OPENAI_API_KEY) {
      const last: unknown = Array.isArray(messages) ? messages[messages.length - 1] : null
      const parts = (last as { parts?: Array<{ type?: string; text?: string }> } | null)?.parts || []
      const text = (parts.find((p) => p?.type === 'text')?.text as string | undefined) || 'Hello!'

      return new Response(`Echo: ${text}`, {
        headers: { 'Content-Type': 'text/plain; charset=utf-8' },
      })
    }

    // Generate AI response
    const result = await generateText({
      model: openai('gpt-4o-mini'),
      system: 'You are a helpful Skymarket assistant.',
      messages: convertToCoreMessages(messages),
    })

    return new Response(result.text, {
      headers: { 'Content-Type': 'text/plain; charset=utf-8' },
    })
  } catch (err) {
    const message = err instanceof Error ? err.message : 'Unknown error'
    return new Response(`Error: ${message}`, { status: 500 })
  }
}
```

### Message Format

The chat endpoint expects messages in AI SDK format:

```typescript
interface ChatMessage {
  role: 'system' | 'user' | 'assistant'
  content: string | Array<{
    type: 'text' | 'image'
    text?: string
    image?: string
  }>
}

// Example request
{
  "messages": [
    {
      "role": "user",
      "content": "I need help booking a drone delivery service"
    }
  ]
}
```

### Error Handling

```typescript
// API Key missing - development mode
if (!process.env.OPENAI_API_KEY) {
  return new Response(`Echo: ${userInput}`, {
    headers: { 'Content-Type': 'text/plain; charset=utf-8' },
  })
}

// OpenAI API errors
catch (err) {
  console.error('[ai] Chat generation failed:', err)

  if (err instanceof OpenAIError) {
    switch (err.type) {
      case 'insufficient_quota':
        return new Response('AI service temporarily unavailable', { status: 503 })
      case 'rate_limit_exceeded':
        return new Response('Please wait a moment before trying again', { status: 429 })
      case 'invalid_request_error':
        return new Response('Invalid request format', { status: 400 })
      default:
        return new Response('AI processing error', { status: 500 })
    }
  }

  return new Response(`Error: ${err.message}`, { status: 500 })
}
```

## Advanced AI Features (Planned)

### 1. Service Recommendation Engine

**Purpose**: Intelligent matching of consumers with relevant drone services

```typescript
interface ServiceRecommendation {
  listingId: string
  providerId: string
  matchScore: number
  reasons: string[]
  estimatedPrice: number
  estimatedDuration: number
}

export async function getServiceRecommendations(
  userId: string,
  requirements: {
    category: string
    location: { lat: number; lng: number }
    budget?: number
    urgency?: 'low' | 'medium' | 'high'
    description?: string
  }
): Promise<ServiceRecommendation[]> {
  // Use embeddings to match requirements with service descriptions
  const requirementEmbedding = await generateEmbedding(requirements.description || '')

  // Vector search for similar services
  const { data: listings } = await supabase
    .from('listings')
    .select(`
      id, title, description, base_price, provider_id,
      embedding, providers(user_id, rating, total_jobs)
    `)
    .eq('service_category', requirements.category)
    .eq('is_active', true)

  // Calculate similarity scores and rank
  const recommendations = listings
    .map(listing => ({
      listingId: listing.id,
      providerId: listing.provider_id,
      matchScore: cosineSimilarity(requirementEmbedding, listing.embedding),
      reasons: generateMatchReasons(requirements, listing),
      estimatedPrice: calculateDynamicPrice(listing.base_price, requirements),
      estimatedDuration: estimateServiceDuration(listing, requirements)
    }))
    .sort((a, b) => b.matchScore - a.matchScore)
    .slice(0, 10)

  return recommendations
}
```

### 2. Intelligent Customer Support

**Purpose**: AI assistant with tool calling capabilities for booking and support

```typescript
import { generateObject } from 'ai'
import { z } from 'zod'

const CustomerSupportTools = {
  searchServices: z.object({
    category: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
    location: z.string(),
    maxPrice: z.number().optional()
  }),

  createBooking: z.object({
    listingId: z.string(),
    scheduledFor: z.string(),
    specialInstructions: z.string().optional()
  }),

  checkBookingStatus: z.object({
    bookingId: z.string()
  }),

  cancelBooking: z.object({
    bookingId: z.string(),
    reason: z.string()
  })
}

export async function handleCustomerSupportChat(
  messages: ChatMessage[],
  userId?: string
) {
  const result = await generateObject({
    model: openai('gpt-4o'),
    system: `You are a helpful SkyMarket customer support assistant.

    You can help customers:
    - Find and book drone services in Detroit Metro area
    - Check booking status and manage orders
    - Provide information about services and pricing
    - Handle cancellations and refunds

    Detroit service categories:
    - food_delivery: Restaurant delivery via drone
    - courier: Package and document delivery
    - aerial_imaging: Photography, videography, inspections
    - site_mapping: Surveying, construction mapping, real estate

    Always verify user identity before accessing order information.`,

    messages: convertToCoreMessages(messages),
    tools: CustomerSupportTools,
  })

  // Execute tools based on AI decisions
  if (result.toolCalls?.length > 0) {
    for (const toolCall of result.toolCalls) {
      switch (toolCall.toolName) {
        case 'searchServices':
          const services = await searchServicesWithAI(toolCall.args)
          // Add service results to conversation context
          break
        case 'createBooking':
          if (userId) {
            const booking = await createBookingWithValidation(userId, toolCall.args)
            // Return booking confirmation
          }
          break
        // Handle other tool calls...
      }
    }
  }

  return result
}
```

### 3. Multi-Modal Content Analysis

**Purpose**: Analyze images and documents for service requirements

```typescript
export async function analyzeImageRequirements(imageUrl: string) {
  const result = await generateObject({
    model: openai('gpt-4o'),
    schema: z.object({
      serviceCategory: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
      complexity: z.enum(['simple', 'moderate', 'complex']),
      estimatedDuration: z.number(),
      requiredEquipment: z.array(z.string()),
      safetyConsiderations: z.array(z.string()),
      recommendations: z.string()
    }),
    messages: [
      {
        role: 'user',
        content: [
          {
            type: 'text',
            text: 'Analyze this image to determine what drone service would be most appropriate and estimate the requirements.'
          },
          {
            type: 'image',
            image: imageUrl
          }
        ]
      }
    ]
  })

  return result.object
}

// Usage in booking flow
export async function enhanceBookingWithImageAnalysis(
  bookingData: CreateBookingData,
  attachments: string[]
) {
  const analyses = await Promise.all(
    attachments.map(url => analyzeImageRequirements(url))
  )

  // Merge AI insights with booking data
  const enhancedBooking = {
    ...bookingData,
    aiInsights: {
      complexity: Math.max(...analyses.map(a =>
        a.complexity === 'complex' ? 3 : a.complexity === 'moderate' ? 2 : 1
      )),
      estimatedDuration: Math.max(...analyses.map(a => a.estimatedDuration)),
      requiredEquipment: [...new Set(analyses.flatMap(a => a.requiredEquipment))],
      safetyNotes: [...new Set(analyses.flatMap(a => a.safetyConsiderations))]
    }
  }

  return enhancedBooking
}
```

### 4. Vector Search and Embeddings

**Purpose**: Semantic search for services and content recommendations

```typescript
import { embed } from 'ai'

export async function generateEmbedding(text: string): Promise<number[]> {
  const { embedding } = await embed({
    model: openai.embedding('text-embedding-3-small'),
    value: text,
  })

  return embedding
}

export async function setupVectorSearch() {
  // Create extension and vector column in Supabase
  await supabaseAdmin.rpc('create_vector_extension')

  // Add embedding column to listings table
  await supabaseAdmin.rpc('add_vector_column', {
    table_name: 'listings',
    column_name: 'embedding',
    dimensions: 1536 // text-embedding-3-small dimensions
  })

  // Create vector index for performance
  await supabaseAdmin.rpc('create_vector_index', {
    table_name: 'listings',
    column_name: 'embedding'
  })
}

export async function searchServicesBySemantics(
  query: string,
  filters?: {
    category?: string
    maxPrice?: number
    location?: { lat: number; lng: number; radius: number }
  }
): Promise<SearchResult[]> {
  // Generate query embedding
  const queryEmbedding = await generateEmbedding(query)

  // Vector similarity search
  const { data: results } = await supabase
    .from('listings')
    .select(`
      id, title, description, base_price, service_category,
      providers!inner(user_id, business_name, rating),
      profiles!inner(full_name, avatar_url)
    `)
    .eq('is_active', true)
    .order('embedding <-> $1', { ascending: true }) // Vector distance
    .limit(20)

  return results?.map(result => ({
    ...result,
    relevanceScore: calculateRelevanceScore(queryEmbedding, result.embedding)
  })) || []
}
```

### 5. Automated Content Generation

**Purpose**: Generate service descriptions, emails, and marketing content

```typescript
export async function generateServiceDescription(
  serviceType: string,
  providerInfo: {
    businessName: string
    specialties: string[]
    experience: string
  }
): Promise<string> {
  const result = await generateText({
    model: openai('gpt-4o'),
    system: `You are a professional copywriter specializing in drone service marketing.

    Create compelling service descriptions that:
    - Highlight unique value propositions
    - Include relevant technical capabilities
    - Emphasize safety and compliance
    - Appeal to Detroit-area customers
    - Follow professional tone suitable for business clients`,

    prompt: `Create a service description for ${serviceType} provided by ${providerInfo.businessName}.

    Specialties: ${providerInfo.specialties.join(', ')}
    Experience: ${providerInfo.experience}

    Focus on Detroit market and regulatory compliance.`
  })

  return result.text
}

export async function generateBookingConfirmationEmail(
  booking: BookingData,
  personalization: {
    customerName: string
    providerName: string
    serviceDetails: string
  }
): Promise<{ subject: string; html: string; text: string }> {
  const result = await generateObject({
    model: openai('gpt-4o'),
    schema: z.object({
      subject: z.string(),
      html: z.string(),
      text: z.string()
    }),
    prompt: `Generate a professional booking confirmation email for a drone service.

    Customer: ${personalization.customerName}
    Provider: ${personalization.providerName}
    Service: ${personalization.serviceDetails}
    Booking ID: ${booking.id}
    Scheduled: ${booking.scheduled_for}
    Total: $${booking.price_total}

    Include:
    - Warm, professional greeting
    - Booking details and next steps
    - Provider contact information
    - Cancellation policy
    - SkyMarket branding
    - Detroit-specific regulations if applicable`
  })

  return result.object
}
```

## Testing and Validation

### AI Response Quality Testing

```typescript
describe('AI Chat Assistant', () => {
  test('handles basic customer service inquiries', async () => {
    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        messages: [
          { role: 'user', content: 'I need help booking a drone delivery' }
        ]
      })
    })

    const text = await response.text()
    expect(text).toContain('drone')
    expect(text).toContain('delivery')
    expect(response.status).toBe(200)
  })

  test('fallback mode works without API key', async () => {
    // Temporarily remove API key
    delete process.env.OPENAI_API_KEY

    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        messages: [{ role: 'user', content: 'Hello test' }]
      })
    })

    const text = await response.text()
    expect(text).toBe('Echo: Hello test')
  })

  test('handles rate limiting gracefully', async () => {
    // Mock rate limit error
    jest.spyOn(openai, 'generateText').mockRejectedValue(
      new OpenAIError('Rate limit exceeded', 'rate_limit_exceeded')
    )

    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        messages: [{ role: 'user', content: 'Test message' }]
      })
    })

    expect(response.status).toBe(429)
  })
})

// Performance testing for embeddings
describe('Vector Search Performance', () => {
  test('embedding generation under 500ms', async () => {
    const start = Date.now()
    await generateEmbedding('Test drone photography service in Detroit')
    const duration = Date.now() - start

    expect(duration).toBeLessThan(500)
  })

  test('vector search returns results under 200ms', async () => {
    const start = Date.now()
    const results = await searchServicesBySemantics('aerial photography')
    const duration = Date.now() - start

    expect(duration).toBeLessThan(200)
    expect(results.length).toBeGreaterThan(0)
  })
})
```

### Content Quality Validation

```typescript
interface ContentQualityMetrics {
  readabilityScore: number  // Flesch reading ease
  sentimentScore: number    // Positive/negative sentiment
  professionalismScore: number // Business appropriate language
  accuracyScore: number     // Factual correctness
}

export async function validateGeneratedContent(
  content: string,
  context: {
    type: 'service_description' | 'email' | 'support_response'
    audience: 'consumer' | 'provider' | 'business'
  }
): Promise<ContentQualityMetrics> {
  const validation = await generateObject({
    model: openai('gpt-4o'),
    schema: z.object({
      readabilityScore: z.number().min(0).max(100),
      sentimentScore: z.number().min(-1).max(1),
      professionalismScore: z.number().min(0).max(100),
      accuracyScore: z.number().min(0).max(100),
      improvements: z.array(z.string()),
      flaggedIssues: z.array(z.string())
    }),
    messages: [
      {
        role: 'system',
        content: `You are a content quality analyst. Evaluate the given content for:
        1. Readability (0-100, higher = easier to read)
        2. Sentiment (-1 to 1, -1 = negative, 1 = positive)
        3. Professionalism (0-100, business appropriate language)
        4. Accuracy (0-100, factual correctness for drone services)`
      },
      {
        role: 'user',
        content: `Evaluate this ${context.type} content for ${context.audience} audience:\n\n${content}`
      }
    ]
  })

  return validation.object
}
```

## Monitoring and Analytics

### AI Usage Metrics

```typescript
interface AIMetrics {
  totalChatRequests: number
  averageResponseTime: number
  errorRate: number
  costPerRequest: number
  userSatisfactionScore: number
  toolCallSuccessRate: number
}

export async function trackAIUsage(
  endpoint: string,
  responseTime: number,
  tokenUsage: { prompt: number; completion: number },
  success: boolean
) {
  await supabaseAdmin
    .from('ai_usage_logs')
    .insert({
      endpoint,
      response_time_ms: responseTime,
      prompt_tokens: tokenUsage.prompt,
      completion_tokens: tokenUsage.completion,
      total_tokens: tokenUsage.prompt + tokenUsage.completion,
      success,
      cost_usd: calculateOpenAICost(tokenUsage),
      created_at: new Date().toISOString()
    })
}

function calculateOpenAICost(usage: { prompt: number; completion: number }): number {
  // GPT-4o-mini pricing (as of 2024)
  const promptCostPer1k = 0.00015   // $0.15 per 1k prompt tokens
  const completionCostPer1k = 0.0006 // $0.60 per 1k completion tokens

  return (usage.prompt * promptCostPer1k / 1000) +
         (usage.completion * completionCostPer1k / 1000)
}
```

### Performance Monitoring

```sql
-- AI usage analytics
CREATE VIEW ai_performance_metrics AS
SELECT
  DATE_TRUNC('day', created_at) as date,
  COUNT(*) as total_requests,
  AVG(response_time_ms) as avg_response_time,
  COUNT(*) FILTER (WHERE success = false) * 100.0 / COUNT(*) as error_rate,
  SUM(cost_usd) as total_cost,
  AVG(total_tokens) as avg_tokens_per_request
FROM ai_usage_logs
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY date DESC;

-- Token usage by endpoint
SELECT
  endpoint,
  COUNT(*) as requests,
  AVG(prompt_tokens) as avg_prompt_tokens,
  AVG(completion_tokens) as avg_completion_tokens,
  SUM(cost_usd) as total_cost
FROM ai_usage_logs
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY endpoint
ORDER BY total_cost DESC;
```

## Security and Privacy

### Data Protection

```typescript
// Sanitize sensitive information from AI requests
function sanitizeForAI(content: string): string {
  return content
    .replace(/\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/g, '[CARD_NUMBER]')
    .replace(/\b\d{3}-\d{2}-\d{4}\b/g, '[SSN]')
    .replace(/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g, '[EMAIL]')
    .replace(/\b\d{10,}\b/g, '[PHONE]')
}

// Content filtering for AI responses
function filterAIResponse(response: string): string {
  const prohibitedContent = [
    'unsafe flight practices',
    'regulatory violations',
    'unauthorized airspace',
    'privacy violations'
  ]

  for (const prohibited of prohibitedContent) {
    if (response.toLowerCase().includes(prohibited)) {
      throw new Error('AI response contains prohibited content')
    }
  }

  return response
}
```

### Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(20, '1 m'), // 20 requests per minute
})

export async function rateLimitAIRequest(userId: string) {
  const { success, limit, remaining, reset } = await ratelimit.limit(
    `ai:${userId}`
  )

  if (!success) {
    throw new Error('AI rate limit exceeded')
  }

  return { remaining, reset }
}
```

## Deployment and Scaling

### Production Configuration

```bash
# Production AI settings
OPENAI_API_KEY=sk-prod-...
AI_MODEL_TEMPERATURE=0.3        # Lower for more consistent responses
AI_MAX_TOKENS=300               # Reasonable limit for customer support
AI_CONTEXT_WINDOW=4096          # Sufficient for most conversations

# Monitoring
AI_USAGE_ANALYTICS=true
AI_RESPONSE_LOGGING=false       # Don't log responses in production
AI_ERROR_TRACKING=true

# Performance
AI_CACHE_TTL=3600              # Cache common responses for 1 hour
AI_CONCURRENT_REQUESTS=5       # Limit concurrent OpenAI requests
```

### Scaling Considerations

```typescript
// Connection pooling for high-volume AI requests
import { OpenAI } from 'openai'

class AIServicePool {
  private clients: OpenAI[] = []
  private currentIndex = 0

  constructor(poolSize: number = 3) {
    for (let i = 0; i < poolSize; i++) {
      this.clients.push(new OpenAI({
        apiKey: process.env.OPENAI_API_KEY,
        timeout: 30000,
        maxRetries: 2
      }))
    }
  }

  getClient(): OpenAI {
    const client = this.clients[this.currentIndex]
    this.currentIndex = (this.currentIndex + 1) % this.clients.length
    return client
  }
}

export const aiPool = new AIServicePool()
```

This AI integration provides a solid foundation for intelligent features in SkyMarket while maintaining flexibility for future enhancements and ensuring proper security and performance considerations.