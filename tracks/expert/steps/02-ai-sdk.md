# Expert Step 02: Vercel AI SDK - Advanced Patterns & Implementation

## Objective
Master the Vercel AI SDK for production-grade AI features including streaming responses, embeddings, and advanced patterns. Implement sophisticated AI capabilities that enhance the SkyMarket user experience with enterprise-level architecture.

## What You'll Master
- Advanced streaming patterns with real-time UI updates
- Embedding generation and semantic search capabilities
- Tool calling and function execution patterns
- Multi-modal AI interactions (text, images, structured data)
- Performance optimization and caching strategies

## Prerequisites
- Completed Expert Step 01 (OpenAI API integration)
- Advanced understanding of streaming APIs
- Experience with vector databases and embeddings
- Knowledge of advanced TypeScript patterns

## Advanced AI SDK Architecture

### Core SDK Components
```
GenerateText ←→ StreamText ←→ GenerateObject
     ↑                                    ↓
Embeddings ←→ Tool Functions ←→ Multi-Modal
```

**Advanced Patterns:**
1. **Streaming with State:** Real-time UI updates with intermediate results
2. **Tool Integration:** Function calling with external services
3. **Semantic Search:** Vector embeddings for intelligent matching
4. **Multi-Step Reasoning:** Chained AI operations for complex tasks
5. **Structured Generation:** Type-safe AI responses

## Instructions

### 1. Advanced Text Generation Patterns

**Cursor Prompt:**
```
Implement advanced AI SDK patterns for SkyMarket with these components:

1. Streaming text generation at app/api/ai/stream/route.ts:
   - Real-time streaming with intermediate results
   - Token-by-token delivery with minimal latency
   - Error handling and recovery for stream interruptions
   - Progress indicators and completion callbacks
   - Support for different streaming modes (words, sentences, full)

2. Structured generation at app/api/ai/generate/route.ts:
   - Type-safe responses using generateObject
   - Zod schema validation for AI outputs
   - Structured data extraction from unstructured input
   - Error correction and retry logic for malformed responses

3. Multi-step reasoning at app/api/ai/analyze/route.ts:
   - Chain multiple AI calls for complex analysis
   - Context preservation across reasoning steps
   - Intermediate result validation and correction
   - Final synthesis and conclusion generation

Use context7 to get the latest Vercel AI SDK patterns and best practices.
```

### 2. Embedding & Semantic Search Implementation

**Cursor Prompt:**
```
Create a comprehensive semantic search system for SkyMarket:

1. Embedding service at lib/ai/embeddings.ts:
   - Text preprocessing and normalization
   - Batch embedding generation for efficiency
   - Caching strategies for frequently embedded content
   - Vector similarity calculation utilities
   - Integration with vector databases (Pinecone/Supabase Vector)

2. Semantic search API at app/api/ai/search/route.ts:
   - Query understanding and expansion
   - Vector similarity search with filtering
   - Hybrid search combining semantic and keyword matching
   - Result ranking and relevance scoring
   - Detroit Metro area context awareness

3. Content indexing system:
   - Service listing automatic embedding generation
   - Provider profile semantic indexing
   - Review and feedback content analysis
   - Dynamic content updates and re-indexing
```

### 3. Advanced Tool Integration

**Cursor Prompt:**
```
Implement AI tool calling patterns for dynamic SkyMarket features:

1. Service recommendation engine:
   - Dynamic tool calls to search available services
   - Real-time pricing and availability checks
   - Location-based filtering and routing
   - User preference learning and adaptation

2. Booking assistance tools:
   - Calendar availability checking
   - Weather condition validation for drone operations
   - Regulatory compliance verification (FAA, local)
   - Cost estimation and optimization

3. Provider matching tools:
   - Capability matching algorithms
   - Experience and rating analysis
   - Availability optimization
   - Communication preference matching

Use the latest AI SDK tool calling patterns with proper error handling and validation.
```

## Code Examples

### Advanced Streaming Implementation

```typescript
// app/api/ai/stream/route.ts
import { NextRequest } from 'next/server'
import { streamText, convertToCoreMessages } from 'ai'
import { openai } from '@ai-sdk/openai'

export const runtime = 'nodejs'
export const dynamic = 'force-dynamic'

export async function POST(request: NextRequest) {
  try {
    const { 
      messages, 
      userId,
      context = 'general',
      streamingOptions = {}
    } = await request.json()

    // Validate input
    if (!messages || !Array.isArray(messages)) {
      return new Response('Invalid messages format', { status: 400 })
    }

    const {
      temperature = 0.7,
      maxTokens = 1000,
      enableTools = false,
      responseFormat = 'text'
    } = streamingOptions

    // SkyMarket-specific system prompts based on context
    const systemPrompts = {
      general: 'You are a helpful SkyMarket assistant for drone services in Detroit Metro.',
      booking: 'You help users book drone services, focusing on requirements and scheduling.',
      provider: 'You assist service providers with listing optimization and customer communication.',
      technical: 'You provide technical guidance on drone operations, regulations, and safety.'
    }

    const result = await streamText({
      model: openai('gpt-4o-mini'),
      system: systemPrompts[context as keyof typeof systemPrompts],
      messages: convertToCoreMessages(messages),
      temperature,
      maxTokens,
      
      // Advanced streaming configuration
      experimental_streamingApi: true,
      
      // Tool integration if enabled
      tools: enableTools ? {
        searchServices: {
          description: 'Search for available drone services',
          parameters: {
            type: 'object',
            properties: {
              query: { type: 'string' },
              location: { type: 'string' },
              serviceType: { type: 'string', enum: ['delivery', 'imaging', 'mapping'] }
            }
          },
          execute: async (params) => {
            // Implement service search logic
            return await searchAvailableServices(params)
          }
        },
        checkWeather: {
          description: 'Check weather conditions for drone operations',
          parameters: {
            type: 'object',
            properties: {
              location: { type: 'string' },
              date: { type: 'string' }
            }
          },
          execute: async (params) => {
            // Implement weather checking logic
            return await getWeatherConditions(params)
          }
        }
      } : undefined,

      // Custom response processing
      onFinish: async ({ text, toolCalls, usage }) => {
        // Log usage for monitoring
        console.log('Stream completed:', {
          userId,
          context,
          tokenUsage: usage,
          toolCallsCount: toolCalls?.length || 0,
          timestamp: new Date().toISOString()
        })
      }
    })

    return result.toTextStreamResponse({
      headers: {
        'X-User-ID': userId || 'anonymous',
        'X-Context': context,
        'X-Stream-Version': '2.0'
      }
    })
  } catch (error) {
    console.error('Streaming error:', error)
    return new Response(
      JSON.stringify({ error: 'Streaming service unavailable' }),
      { 
        status: 500,
        headers: { 'Content-Type': 'application/json' }
      }
    )
  }
}

// Helper functions
async function searchAvailableServices(params: any) {
  // Implement Supabase query for service search
  // Include location filtering, availability checks, pricing
  return {
    services: [],
    totalFound: 0,
    filters: params
  }
}

async function getWeatherConditions(params: any) {
  // Integrate with weather API
  // Return drone-operation specific conditions
  return {
    suitable: true,
    windSpeed: 10,
    visibility: 'good',
    alerts: []
  }
}
```

### Production Embedding Service

```typescript
// lib/ai/embeddings.ts
import { embed, embedMany } from 'ai'
import { openai } from '@ai-sdk/openai'

export class EmbeddingService {
  private model = openai.textEmbeddingModel('text-embedding-3-small')
  private cache = new Map<string, number[]>()
  private readonly CACHE_TTL = 3600000 // 1 hour
  private readonly MAX_BATCH_SIZE = 100

  async generateEmbedding(text: string): Promise<number[]> {
    // Check cache first
    const cacheKey = this.generateCacheKey(text)
    const cached = this.cache.get(cacheKey)
    
    if (cached) {
      return cached
    }

    try {
      const { embedding } = await embed({
        model: this.model,
        value: this.preprocessText(text)
      })

      // Cache the result
      this.cache.set(cacheKey, embedding)
      
      // Clean up old cache entries periodically
      this.cleanupCache()
      
      return embedding
    } catch (error) {
      console.error('Embedding generation failed:', error)
      throw new Error('Failed to generate embedding')
    }
  }

  async generateBatchEmbeddings(texts: string[]): Promise<number[][]> {
    // Process in batches for efficiency
    const batches = this.chunkArray(texts, this.MAX_BATCH_SIZE)
    const results: number[][] = []

    for (const batch of batches) {
      try {
        const { embeddings } = await embedMany({
          model: this.model,
          values: batch.map(text => this.preprocessText(text))
        })
        
        results.push(...embeddings)
        
        // Cache individual results
        batch.forEach((text, index) => {
          const cacheKey = this.generateCacheKey(text)
          this.cache.set(cacheKey, embeddings[index])
        })
      } catch (error) {
        console.error('Batch embedding failed:', error)
        throw new Error('Failed to generate batch embeddings')
      }
    }

    return results
  }

  calculateSimilarity(embedding1: number[], embedding2: number[]): number {
    // Cosine similarity calculation
    const dotProduct = embedding1.reduce((sum, a, i) => sum + a * embedding2[i], 0)
    const magnitude1 = Math.sqrt(embedding1.reduce((sum, a) => sum + a * a, 0))
    const magnitude2 = Math.sqrt(embedding2.reduce((sum, a) => sum + a * a, 0))
    
    return dotProduct / (magnitude1 * magnitude2)
  }

  private preprocessText(text: string): string {
    return text
      .toLowerCase()
      .replace(/\s+/g, ' ')
      .trim()
      .slice(0, 8000) // Limit to model's context window
  }

  private generateCacheKey(text: string): string {
    // Simple hash function for cache keys
    let hash = 0
    for (let i = 0; i < text.length; i++) {
      const char = text.charCodeAt(i)
      hash = ((hash << 5) - hash) + char
      hash = hash & hash // Convert to 32-bit integer
    }
    return hash.toString(36)
  }

  private chunkArray<T>(array: T[], size: number): T[][] {
    const chunks: T[][] = []
    for (let i = 0; i < array.length; i += size) {
      chunks.push(array.slice(i, i + size))
    }
    return chunks
  }

  private cleanupCache(): void {
    // Simple cleanup - in production, implement LRU or time-based eviction
    if (this.cache.size > 1000) {
      const entries = Array.from(this.cache.entries())
      const half = Math.floor(entries.length / 2)
      this.cache.clear()
      entries.slice(half).forEach(([key, value]) => {
        this.cache.set(key, value)
      })
    }
  }
}

// Singleton instance
export const embeddingService = new EmbeddingService()
```

### Semantic Search API

```typescript
// app/api/ai/search/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { embeddingService } from '@/lib/ai/embeddings'
import { createClient } from '@/lib/supabase/server'

export const runtime = 'nodejs'
export const dynamic = 'force-dynamic'

export async function POST(request: NextRequest) {
  try {
    const { 
      query, 
      filters = {},
      limit = 10,
      threshold = 0.7,
      includeMetadata = true 
    } = await request.json()

    if (!query) {
      return NextResponse.json(
        { error: 'Search query is required' },
        { status: 400 }
      )
    }

    // Generate embedding for search query
    const queryEmbedding = await embeddingService.generateEmbedding(query)
    
    const supabase = createClient()
    
    // Semantic search using Supabase vector similarity
    const { data: results, error } = await supabase.rpc(
      'match_services',
      {
        query_embedding: queryEmbedding,
        match_threshold: threshold,
        match_count: limit,
        filters: filters
      }
    )

    if (error) {
      console.error('Semantic search error:', error)
      return NextResponse.json(
        { error: 'Search service unavailable' },
        { status: 500 }
      )
    }

    // Enhance results with additional context
    const enhancedResults = await Promise.all(
      results.map(async (result: any) => {
        const enhanced = {
          ...result,
          similarity: result.similarity,
          relevanceScore: calculateRelevanceScore(result, query)
        }

        if (includeMetadata) {
          enhanced.metadata = await getServiceMetadata(result.id)
        }

        return enhanced
      })
    )

    return NextResponse.json({
      results: enhancedResults,
      totalFound: results.length,
      query: query,
      searchId: generateSearchId(),
      timestamp: new Date().toISOString()
    })
  } catch (error) {
    console.error('Search API error:', error)
    return NextResponse.json(
      { error: 'Search service error' },
      { status: 500 }
    )
  }
}

function calculateRelevanceScore(result: any, query: string): number {
  // Combine semantic similarity with other relevance factors
  let score = result.similarity * 0.6 // Semantic similarity weight
  
  // Add other factors
  if (result.ratings?.average > 4.0) score += 0.1
  if (result.availability?.immediate) score += 0.1
  if (result.location?.isLocal) score += 0.1
  if (result.verified) score += 0.1
  
  return Math.min(score, 1.0)
}

async function getServiceMetadata(serviceId: string) {
  // Fetch additional service metadata
  return {
    providerInfo: {},
    recentReviews: [],
    availability: {},
    certifications: []
  }
}

function generateSearchId(): string {
  return `search_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`
}
```

## Expected Outcome

After completing this step, you should have:

✅ **Advanced AI SDK Mastery:**
- Production-grade streaming implementations
- Sophisticated embedding and search capabilities
- Tool calling integration with external services
- Multi-modal AI interaction patterns

✅ **Performance Optimization:**
- Efficient caching strategies for embeddings
- Batch processing for large-scale operations
- Real-time streaming with minimal latency
- Resource usage monitoring and optimization

✅ **Enterprise Integration:**
- Semantic search for service discovery
- AI-powered recommendations and matching
- Intelligent query understanding and expansion
- Context-aware responses for SkyMarket domain

✅ **Advanced Patterns:**
- Multi-step reasoning for complex analysis
- Structured data generation with validation
- Error recovery and graceful degradation
- Production monitoring and logging

## Next Steps

With advanced AI SDK patterns implemented:
- **Step 03:** Create AI-powered UI components and experiences
- **Step 04:** Implement advanced MCP tooling and automation
- **Step 05:** Build comprehensive AI analytics and monitoring

## References

### Essential Documentation
- [Vercel AI SDK Core](https://sdk.vercel.ai/docs/ai-sdk-core) - Complete SDK reference
- [Streaming Patterns](https://sdk.vercel.ai/docs/ai-sdk-core/stream-text) - Advanced streaming implementation
- [Tool Calling](https://sdk.vercel.ai/docs/ai-sdk-core/tools-and-tool-calling) - Function integration patterns

### Advanced Topics
- [Vector Databases](https://supabase.com/docs/guides/ai/vector-embeddings) - Embedding storage and search
- [Semantic Search](https://platform.openai.com/docs/guides/embeddings/use-cases) - Search optimization strategies
- [AI Performance](https://vercel.com/docs/functions/edge-functions/ai-sdk) - Edge deployment patterns

---

💡 **Expert Tip:** Focus on creating reusable, composable AI patterns rather than one-off implementations. Well-architected AI services enable rapid feature development and consistent user experiences across your application.


