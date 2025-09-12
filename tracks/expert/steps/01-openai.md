# Expert Step 01: OpenAI API Integration - Enterprise-Grade Implementation

## Objective
Implement a production-ready OpenAI API integration with enterprise security patterns, proper error handling, and architectural best practices. This establishes the foundation for AI-powered features in SkyMarket with scalable, maintainable patterns.

## What You'll Master
- Enterprise API security patterns for LLM integration
- Server-side AI architecture with proper abstraction layers
- Advanced error handling and rate limiting strategies
- Performance optimization for AI API calls
- Cost monitoring and usage tracking patterns

## Prerequisites
- Expert-level understanding of Next.js API routes
- OpenAI API account with billing configured
- Experience with enterprise security patterns
- Understanding of serverless function limitations

## Enterprise Architecture Principles

### Security-First Design
```
Client → API Route → Service Layer → OpenAI API
         ↓
    Validation & Auth → Rate Limiting → Monitoring
```

**Key Security Principles:**
1. **Server-Only API Access:** Never expose OpenAI keys to client-side
2. **Service Layer Abstraction:** Centralized AI service management
3. **Input Validation:** Comprehensive sanitization and validation
4. **Rate Limiting:** Prevent abuse and cost overruns
5. **Audit Logging:** Complete request/response tracking

### Performance & Cost Optimization
- **Response Streaming:** Real-time user experience
- **Request Batching:** Optimize API call efficiency
- **Caching Strategies:** Reduce redundant API calls
- **Token Management:** Monitor and optimize token usage
- **Fallback Patterns:** Graceful degradation strategies

## Instructions

### 1. Environment Configuration

**Production-Grade Environment Setup:**

```bash
# OpenAI Configuration
OPENAI_API_KEY=sk-proj-your-api-key-here
OPENAI_ORGANIZATION_ID=org-your-org-id  # Optional: Organization control
OPENAI_PROJECT_ID=proj_your-project-id  # Optional: Project-specific billing

# AI Service Configuration
AI_MODEL_PRIMARY=gpt-4o-mini                    # Primary model for production
AI_MODEL_FALLBACK=gpt-3.5-turbo                # Fallback for high availability
AI_MAX_TOKENS=1000                              # Token limit per request
AI_TEMPERATURE=0.7                              # Creativity level
AI_REQUEST_TIMEOUT=30000                        # 30 second timeout

# Monitoring & Rate Limiting
AI_RATE_LIMIT_RPM=60                            # Requests per minute
AI_RATE_LIMIT_TPM=60000                         # Tokens per minute
AI_COST_ALERT_THRESHOLD=100                     # USD cost threshold
```

### 2. Create AI Service Architecture

**Cursor Prompt:**
```
Create an enterprise-grade AI service architecture with these components:

1. Core AI service at lib/ai/openai-service.ts with:
   - OpenAI client initialization with proper configuration
   - Text generation with streaming support
   - Error handling with retry logic and exponential backoff
   - Token counting and cost tracking
   - Request/response logging for debugging
   - Rate limiting integration

2. AI configuration manager at lib/ai/config.ts with:
   - Environment variable validation
   - Model selection logic (primary/fallback)
   - Configuration presets for different use cases
   - Runtime configuration updates

3. Type definitions at lib/ai/types.ts with:
   - Request/response interfaces
   - Configuration types
   - Error handling types
   - Streaming response types

Use context7 to get the latest OpenAI API patterns and Vercel AI SDK best practices.
```

### 3. Implement Rate Limiting & Monitoring

**Cursor Prompt:**
```
Create a comprehensive rate limiting and monitoring system:

1. Rate limiter at lib/ai/rate-limiter.ts using:
   - Redis or in-memory store for production scalability
   - Token bucket algorithm for smooth rate limiting
   - Per-user and global rate limiting
   - Rate limit headers in responses

2. Usage monitoring at lib/ai/monitor.ts with:
   - Token usage tracking per request
   - Cost calculation and monitoring
   - Performance metrics (latency, success rate)
   - Alert triggers for cost/usage thresholds

3. Error handling patterns with:
   - Exponential backoff for API failures
   - Circuit breaker pattern for high availability
   - Graceful degradation when limits exceeded
   - Detailed error logging without exposing secrets
```

### 4. Build Production API Routes

**Cursor Prompt:**
```
Create production-ready API routes for SkyMarket AI features:

1. Chat completion endpoint at app/api/ai/chat/route.ts:
   - Streaming responses with proper error handling
   - Context-aware responses for SkyMarket domain
   - Input validation and sanitization
   - Rate limiting and authentication
   - Request/response logging

2. Service description generation at app/api/ai/describe/route.ts:
   - Generate compelling service descriptions
   - Optimize for SkyMarket marketplace context
   - Handle batch generation efficiently
   - Content moderation integration

3. Smart search enhancement at app/api/ai/search/route.ts:
   - Query understanding and expansion
   - Semantic search capabilities
   - Location-aware results for Detroit Metro
   - Performance optimized responses

Ensure all routes follow Next.js 15 patterns and include proper TypeScript typing.
```

### 5. Implement Security & Compliance

**Advanced Security Patterns:**

```typescript
// lib/ai/security.ts
import { createHash, randomBytes } from 'crypto'

export class AISecurityManager {
  // Sanitize user input to prevent prompt injection
  sanitizeInput(input: string): string {
    // Remove potentially malicious patterns
    return input
      .replace(/\b(ignore|forget|disregard)\s+(previous|above|prior)\b/gi, '')
      .replace(/\b(system|admin|root)\s+(prompt|instruction|command)\b/gi, '')
      .trim()
      .slice(0, 4000) // Limit input length
  }

  // Generate request signatures for audit trails
  generateRequestSignature(request: any): string {
    const data = JSON.stringify({ ...request, timestamp: Date.now() })
    return createHash('sha256').update(data).digest('hex')
  }

  // Validate API key format and permissions
  validateAPIKey(key: string): boolean {
    return key.startsWith('sk-') && key.length > 20
  }
}
```

## Code Examples

### Enterprise AI Service Implementation

```typescript
// lib/ai/openai-service.ts
import OpenAI from 'openai'
import { AIConfig } from './config'
import { RateLimiter } from './rate-limiter'
import { AIMonitor } from './monitor'
import { AISecurityManager } from './security'

export class OpenAIService {
  private client: OpenAI
  private config: AIConfig
  private rateLimiter: RateLimiter
  private monitor: AIMonitor
  private security: AISecurityManager

  constructor() {
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!,
      organization: process.env.OPENAI_ORGANIZATION_ID,
      project: process.env.OPENAI_PROJECT_ID,
      timeout: parseInt(process.env.AI_REQUEST_TIMEOUT || '30000'),
    })
    
    this.config = new AIConfig()
    this.rateLimiter = new RateLimiter()
    this.monitor = new AIMonitor()
    this.security = new AISecurityManager()
  }

  async generateText({
    prompt,
    userId,
    context = 'general',
    stream = false
  }: {
    prompt: string
    userId?: string
    context?: 'chat' | 'description' | 'search'
    stream?: boolean
  }) {
    // Security validation
    const sanitizedPrompt = this.security.sanitizeInput(prompt)
    
    // Rate limiting check
    await this.rateLimiter.checkLimit(userId || 'anonymous')
    
    // Generate request signature for audit
    const requestId = this.security.generateRequestSignature({
      prompt: sanitizedPrompt,
      userId,
      context
    })
    
    try {
      const startTime = Date.now()
      
      const response = await this.client.chat.completions.create({
        model: this.config.getModel(context),
        messages: [
          {
            role: 'system',
            content: this.config.getSystemPrompt(context)
          },
          {
            role: 'user',
            content: sanitizedPrompt
          }
        ],
        max_tokens: this.config.getMaxTokens(context),
        temperature: this.config.getTemperature(context),
        stream,
      })
      
      // Monitor usage and performance
      await this.monitor.recordRequest({
        requestId,
        userId,
        context,
        promptTokens: response.usage?.prompt_tokens || 0,
        completionTokens: response.usage?.completion_tokens || 0,
        latency: Date.now() - startTime,
        model: this.config.getModel(context)
      })
      
      return response
    } catch (error) {
      // Enhanced error logging
      console.error('OpenAI API Error:', {
        requestId,
        error: error.message,
        userId,
        context,
        timestamp: new Date().toISOString()
      })
      
      throw this.handleAPIError(error)
    }
  }

  private handleAPIError(error: any): Error {
    if (error.status === 429) {
      return new Error('Rate limit exceeded. Please try again later.')
    }
    if (error.status === 401) {
      return new Error('Authentication failed. Please check API configuration.')
    }
    if (error.status >= 500) {
      return new Error('OpenAI service unavailable. Please try again later.')
    }
    return new Error('AI service error. Please try again.')
  }
}

// Singleton instance
export const openaiService = new OpenAIService()
```

### Production Chat API Route

```typescript
// app/api/ai/chat/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { openaiService } from '@/lib/ai/openai-service'
import { streamText } from 'ai'
import { openai } from '@ai-sdk/openai'

export const runtime = 'nodejs'
export const dynamic = 'force-dynamic'

export async function POST(request: NextRequest) {
  try {
    const { messages, userId, context = 'chat' } = await request.json()
    
    // Input validation
    if (!messages || !Array.isArray(messages)) {
      return NextResponse.json(
        { error: 'Messages array is required' },
        { status: 400 }
      )
    }
    
    // Extract latest user message
    const userMessage = messages[messages.length - 1]?.content
    if (!userMessage) {
      return NextResponse.json(
        { error: 'User message is required' },
        { status: 400 }
      )
    }
    
    // SkyMarket-specific system prompt
    const systemPrompt = `
      You are a helpful assistant for SkyMarket, a drone service marketplace 
      serving the Detroit Metro area. You help users with:
      - Finding drone services (delivery, imaging, mapping)
      - Understanding service offerings and pricing
      - Connecting with certified drone operators
      - Detroit-area regulations and requirements
      
      Always be helpful, professional, and focused on the SkyMarket ecosystem.
      If asked about areas outside Detroit Metro, politely explain our service area.
    `
    
    // Stream response using Vercel AI SDK
    const result = await streamText({
      model: openai('gpt-4o-mini'),
      system: systemPrompt,
      messages: messages,
      maxTokens: 1000,
      temperature: 0.7,
    })
    
    return result.toTextStreamResponse()
  } catch (error) {
    console.error('Chat API Error:', error)
    
    // Return user-friendly error
    return NextResponse.json(
      { error: 'AI service temporarily unavailable. Please try again.' },
      { status: 500 }
    )
  }
}
```

## Expected Outcome

After completing this step, you should have:

✅ **Enterprise Architecture:**
- Scalable AI service layer with proper abstraction
- Production-ready error handling and monitoring
- Comprehensive security patterns implemented
- Cost-optimized API usage strategies

✅ **Advanced Features:**
- Streaming responses with proper error boundaries
- Context-aware AI responses for SkyMarket domain
- Rate limiting and abuse prevention
- Audit trails and compliance logging

✅ **Performance Optimization:**
- Sub-second response times for AI features
- Efficient token usage and cost monitoring
- Graceful degradation under high load
- Caching strategies for repeated queries

✅ **Production Readiness:**
- Comprehensive monitoring and alerting
- Security hardening against common attacks
- Scalable architecture for growth
- Maintenance and debugging capabilities

## Advanced Patterns

### Cost Optimization Strategies

```typescript
// Implement intelligent caching
class AIResponseCache {
  private cache = new Map<string, { response: string; timestamp: number }>()
  private ttl = 3600000 // 1 hour
  
  getCachedResponse(prompt: string): string | null {
    const key = this.generateKey(prompt)
    const cached = this.cache.get(key)
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.response
    }
    
    return null
  }
}
```

### Circuit Breaker Pattern

```typescript
class AICircuitBreaker {
  private failures = 0
  private lastFailure = 0
  private threshold = 5
  private timeout = 60000 // 1 minute
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.isOpen()) {
      throw new Error('AI service circuit breaker is open')
    }
    
    try {
      const result = await operation()
      this.onSuccess()
      return result
    } catch (error) {
      this.onFailure()
      throw error
    }
  }
}
```

## Security Considerations

### Prompt Injection Prevention
- Input sanitization and validation
- Content filtering and moderation
- System prompt protection
- Output validation and filtering

### Data Privacy
- No PII logging in AI requests
- Request anonymization strategies
- Compliance with data retention policies
- Encrypted storage for sensitive contexts

## Next Steps

With enterprise OpenAI integration complete, you're ready for:
- **Step 02:** Advanced AI SDK patterns with streaming and embeddings
- **Step 03:** AI-powered UI components and user experiences
- **Step 04:** Advanced MCP tooling for enhanced development

## References

### Essential Documentation
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference) - Complete API documentation
- [Vercel AI SDK](https://sdk.vercel.ai/docs) - AI SDK patterns and best practices
- [Next.js API Routes](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) - Server-side implementation

### Security Resources
- [OWASP AI Security](https://owasp.org/www-project-ai-security-and-privacy-guide/) - AI security best practices
- [Prompt Injection Prevention](https://learnprompting.org/docs/prompt_hacking/injection) - Security patterns
- [OpenAI Safety Guidelines](https://platform.openai.com/docs/usage-policies) - Usage policies and safety

---

💡 **Expert Tip:** Always implement comprehensive monitoring and alerting for AI services. Production AI features require careful cost management, performance monitoring, and security oversight to operate successfully at scale.


