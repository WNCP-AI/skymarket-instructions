# Expert Step 1: OpenAI API - Key Configuration and Server-Only Usage Patterns

**Objective**: Configure OpenAI API with proper security patterns, understand server-only usage, and establish production-ready integration foundations.

## Overview

This step focuses on setting up OpenAI API integration with enterprise-grade security patterns, proper key management, and server-side architecture that's suitable for production deployment.

## OpenAI API Account Setup

### Advanced Account Configuration

1. **API Key Generation**
   - Navigate to [platform.openai.com](https://platform.openai.com)
   - Create project-specific API keys
   - Set usage limits and monitoring

2. **Usage Limits & Monitoring**
   ```bash
   # Set monthly usage limits
   curl -X POST "https://api.openai.com/v1/organizations/[org-id]/usage_limits" \
     -H "Authorization: Bearer $OPENAI_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "max_usage_usd": 100.00,
       "alert_threshold": 0.8
     }'
   ```

3. **Rate Limit Understanding**
   - Tier 1: 3 RPM, 200 RPD (default)
   - Tier 2: 3,500 RPM, 10,000 RPD (requires $5 spent)
   - Enterprise: Custom limits available

## Server-Only Usage Patterns

### Why Server-Side Only?

**Security**
- API keys never exposed to client
- Request/response logging and monitoring
- Rate limiting and abuse prevention
- Content filtering and moderation

**Architecture**
- Centralized AI service management
- Cost control and monitoring
- Model switching without client updates
- Response caching and optimization

**Compliance**
- Data privacy protection
- Audit trail maintenance
- Request/response validation
- Content policy enforcement

### Implementation Architecture

```typescript
// lib/ai/openai-client.ts
import OpenAI from 'openai'
import { z } from 'zod'

// Validate environment configuration
const openAIConfig = z.object({
  apiKey: z.string().min(1),
  organization: z.string().optional(),
  maxRetries: z.number().default(3),
  timeout: z.number().default(60000),
}).parse({
  apiKey: process.env.OPENAI_API_KEY,
  organization: process.env.OPENAI_ORGANIZATION,
  maxRetries: parseInt(process.env.OPENAI_MAX_RETRIES || '3'),
  timeout: parseInt(process.env.OPENAI_TIMEOUT || '60000'),
})

// Create singleton client with configuration
export const openai = new OpenAI({
  apiKey: openAIConfig.apiKey,
  organization: openAIConfig.organization,
  maxRetries: openAIConfig.maxRetries,
  timeout: openAIConfig.timeout,
})

// Request/Response types
export interface AIRequest {
  prompt: string
  model?: string
  maxTokens?: number
  temperature?: number
  userId?: string
  context?: Record<string, unknown>
}

export interface AIResponse {
  content: string
  model: string
  usage: {
    promptTokens: number
    completionTokens: number
    totalTokens: number
  }
  metadata: {
    requestId: string
    latency: number
    timestamp: string
  }
}
```

### Advanced Service Layer

```typescript
// lib/ai/ai-service.ts
import { openai, type AIRequest, type AIResponse } from './openai-client'
import { createHash } from 'crypto'
import { Redis } from 'ioredis'

export class AIService {
  private redis: Redis
  private metrics: AIMetrics

  constructor() {
    this.redis = new Redis(process.env.REDIS_URL!)
    this.metrics = new AIMetrics()
  }

  async generateText(request: AIRequest): Promise<AIResponse> {
    const startTime = Date.now()
    const requestId = this.generateRequestId(request)

    try {
      // Check cache first
      const cached = await this.getCachedResponse(request)
      if (cached) {
        this.metrics.recordCacheHit()
        return cached
      }

      // Rate limiting check
      await this.checkRateLimit(request.userId)

      // Make OpenAI request
      const completion = await openai.chat.completions.create({
        model: request.model || 'gpt-4-turbo-preview',
        messages: [
          { role: 'user', content: request.prompt }
        ],
        max_tokens: request.maxTokens || 1000,
        temperature: request.temperature || 0.1,
        user: request.userId, // For OpenAI's usage tracking
      })

      const response: AIResponse = {
        content: completion.choices[0]?.message?.content || '',
        model: completion.model,
        usage: {
          promptTokens: completion.usage?.prompt_tokens || 0,
          completionTokens: completion.usage?.completion_tokens || 0,
          totalTokens: completion.usage?.total_tokens || 0,
        },
        metadata: {
          requestId,
          latency: Date.now() - startTime,
          timestamp: new Date().toISOString(),
        },
      }

      // Cache response
      await this.cacheResponse(request, response)

      // Record metrics
      this.metrics.recordRequest(response)

      return response
    } catch (error) {
      this.metrics.recordError(error)
      throw new AIServiceError('Failed to generate text', error)
    }
  }

  private generateRequestId(request: AIRequest): string {
    return createHash('sha256')
      .update(JSON.stringify(request))
      .digest('hex')
      .substring(0, 16)
  }

  private async getCachedResponse(request: AIRequest): Promise<AIResponse | null> {
    const cacheKey = this.getCacheKey(request)
    const cached = await this.redis.get(cacheKey)
    return cached ? JSON.parse(cached) : null
  }

  private async cacheResponse(request: AIRequest, response: AIResponse): Promise<void> {
    const cacheKey = this.getCacheKey(request)
    const ttl = 3600 // 1 hour
    await this.redis.setex(cacheKey, ttl, JSON.stringify(response))
  }

  private getCacheKey(request: AIRequest): string {
    return `ai:${createHash('md5').update(JSON.stringify(request)).digest('hex')}`
  }

  private async checkRateLimit(userId?: string): Promise<void> {
    if (!userId) return

    const key = `rate_limit:${userId}`
    const current = await this.redis.incr(key)
    
    if (current === 1) {
      await this.redis.expire(key, 60) // 1 minute window
    }

    if (current > 10) { // 10 requests per minute per user
      throw new RateLimitError('Too many requests')
    }
  }
}

// Custom error classes
export class AIServiceError extends Error {
  constructor(message: string, public cause?: unknown) {
    super(message)
    this.name = 'AIServiceError'
  }
}

export class RateLimitError extends Error {
  constructor(message: string) {
    super(message)
    this.name = 'RateLimitError'
  }
}
```

## Production Environment Configuration

### Environment Variables

```bash
# .env.local (for development)
OPENAI_API_KEY=sk-proj-your-development-key
OPENAI_ORGANIZATION=org-your-org-id
OPENAI_MAX_RETRIES=3
OPENAI_TIMEOUT=60000
REDIS_URL=redis://localhost:6379

# Production considerations
NODE_ENV=production
OPENAI_LOG_LEVEL=error
AI_CACHE_TTL=3600
AI_RATE_LIMIT_PER_USER=10
AI_RATE_LIMIT_WINDOW=60
```

### Security Configuration

```typescript
// lib/ai/security.ts
import { headers } from 'next/headers'
import { createHash, timingSafeEqual } from 'crypto'

export class AISecurityManager {
  private static readonly ALLOWED_ORIGINS = [
    process.env.NEXT_PUBLIC_APP_URL!,
    'https://skymarket-production.vercel.app',
  ]

  static validateRequest(request: Request): void {
    // Validate origin
    const origin = request.headers.get('origin')
    if (origin && !this.ALLOWED_ORIGINS.includes(origin)) {
      throw new Error('Invalid origin')
    }

    // Validate content type for POST requests
    if (request.method === 'POST') {
      const contentType = request.headers.get('content-type')
      if (!contentType?.includes('application/json')) {
        throw new Error('Invalid content type')
      }
    }

    // Check for required headers
    const authHeader = request.headers.get('authorization')
    if (!authHeader) {
      throw new Error('Missing authorization header')
    }
  }

  static sanitizeInput(input: string): string {
    // Remove potentially harmful content
    return input
      .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
      .replace(/javascript:/gi, '')
      .trim()
  }

  static validateTokenUsage(usage: number, limit: number): void {
    if (usage > limit) {
      throw new Error(`Token usage ${usage} exceeds limit ${limit}`)
    }
  }
}
```

## API Route Implementation

### Structured API Endpoint

```typescript
// app/api/ai/generate/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { AIService, AIServiceError, RateLimitError } from '@/lib/ai/ai-service'
import { AISecurityManager } from '@/lib/ai/security'
import { createServerClient } from '@/lib/supabase/server'

// Request validation schema
const generateRequestSchema = z.object({
  prompt: z.string().min(1).max(10000),
  model: z.enum(['gpt-4-turbo-preview', 'gpt-3.5-turbo']).optional(),
  maxTokens: z.number().min(1).max(4000).optional(),
  temperature: z.number().min(0).max(2).optional(),
  context: z.record(z.unknown()).optional(),
})

export async function POST(request: NextRequest) {
  try {
    // Security validation
    AISecurityManager.validateRequest(request)

    // Parse and validate request body
    const body = await request.json()
    const validatedRequest = generateRequestSchema.parse(body)

    // Get authenticated user
    const supabase = createServerClient()
    const { data: { user }, error: authError } = await supabase.auth.getUser()
    
    if (authError || !user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    }

    // Sanitize input
    const sanitizedPrompt = AISecurityManager.sanitizeInput(validatedRequest.prompt)

    // Initialize AI service and generate response
    const aiService = new AIService()
    const aiResponse = await aiService.generateText({
      ...validatedRequest,
      prompt: sanitizedPrompt,
      userId: user.id,
    })

    // Log request for audit purposes
    await logAIRequest(user.id, validatedRequest, aiResponse)

    return NextResponse.json({
      success: true,
      data: aiResponse,
    })

  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        error: 'Invalid request',
        details: error.errors,
      }, { status: 400 })
    }

    if (error instanceof RateLimitError) {
      return NextResponse.json({
        error: 'Rate limit exceeded',
      }, { status: 429 })
    }

    if (error instanceof AIServiceError) {
      console.error('AI Service Error:', error)
      return NextResponse.json({
        error: 'AI service unavailable',
      }, { status: 503 })
    }

    console.error('Unexpected error:', error)
    return NextResponse.json({
      error: 'Internal server error',
    }, { status: 500 })
  }
}

async function logAIRequest(
  userId: string, 
  request: z.infer<typeof generateRequestSchema>, 
  response: any
) {
  // Implementation for audit logging
  // Could store in database, send to logging service, etc.
  const supabase = createServerClient()
  
  await supabase.from('ai_requests').insert({
    user_id: userId,
    prompt_length: request.prompt.length,
    model_used: response.model,
    tokens_used: response.usage.totalTokens,
    latency_ms: response.metadata.latency,
    created_at: new Date().toISOString(),
  })
}
```

## Testing and Monitoring

### Unit Tests

```typescript
// __tests__/lib/ai/ai-service.test.ts
import { AIService } from '@/lib/ai/ai-service'
import { jest } from '@jest/globals'

jest.mock('openai')

describe('AIService', () => {
  let aiService: AIService

  beforeEach(() => {
    aiService = new AIService()
    jest.clearAllMocks()
  })

  test('should generate text successfully', async () => {
    const mockRequest = {
      prompt: 'Test prompt',
      model: 'gpt-3.5-turbo',
      userId: 'test-user',
    }

    const response = await aiService.generateText(mockRequest)

    expect(response).toHaveProperty('content')
    expect(response).toHaveProperty('usage')
    expect(response.metadata).toHaveProperty('requestId')
    expect(response.metadata).toHaveProperty('latency')
  })

  test('should respect rate limits', async () => {
    const mockRequest = {
      prompt: 'Test prompt',
      userId: 'test-user',
    }

    // Simulate multiple rapid requests
    const promises = Array.from({ length: 15 }, () => 
      aiService.generateText(mockRequest)
    )

    await expect(Promise.all(promises)).rejects.toThrow('Too many requests')
  })
})
```

### Performance Monitoring

```typescript
// lib/ai/metrics.ts
export class AIMetrics {
  async recordRequest(response: AIResponse): Promise<void> {
    const metrics = {
      model: response.model,
      tokens: response.usage.totalTokens,
      latency: response.metadata.latency,
      timestamp: response.metadata.timestamp,
    }

    // Send to monitoring service (e.g., DataDog, New Relic)
    await this.sendMetrics('ai.request', metrics)
  }

  async recordCacheHit(): Promise<void> {
    await this.sendMetrics('ai.cache.hit', { count: 1 })
  }

  async recordError(error: unknown): Promise<void> {
    await this.sendMetrics('ai.error', {
      error: error instanceof Error ? error.message : 'Unknown error',
      count: 1,
    })
  }

  private async sendMetrics(metric: string, data: Record<string, unknown>): Promise<void> {
    // Implementation depends on your monitoring service
    if (process.env.NODE_ENV === 'production') {
      // Send to your monitoring service
      console.log(`Metric: ${metric}`, data)
    }
  }
}
```

## Expected Outcome

After completing this step, you should have:

### Production-Ready Configuration
- [ ] OpenAI API key properly secured
- [ ] Server-side only architecture implemented
- [ ] Request/response validation in place
- [ ] Error handling and logging configured

### Advanced Features
- [ ] Response caching implementation
- [ ] Rate limiting per user
- [ ] Metrics and monitoring setup
- [ ] Security validation layers

### Testing & Monitoring
- [ ] Unit tests for AI service
- [ ] Performance metrics collection
- [ ] Audit logging for compliance
- [ ] Error tracking and alerting

### Architecture Foundation
- [ ] Scalable service layer design
- [ ] Proper separation of concerns
- [ ] TypeScript type safety throughout
- [ ] Production deployment ready

## Next Steps

Excellent! You now have a secure, scalable OpenAI integration. Next, we'll explore Vercel AI Elements and decide whether UI primitives are needed for your advanced AI features.

---

**Next Step**: [Step 2: Vercel AI Elements](./02-vercel-ai-elements.md)

**Architecture Review**: Consider how this AI service integrates with your existing SkyMarket architecture and what additional patterns might be needed for your specific use cases.