# Step 2: AI SDK Integration and Implementation

## What You'll Accomplish

Build upon your OpenAI setup to implement advanced AI SDK features in SkyMarket. You'll analyze the existing business assistant, enhance it with modern AI SDK patterns, and create production-ready AI tools for marketplace operations.

**Key Outcomes**:
- Understanding SkyMarket's current AI implementation architecture
- Enhanced chat API with streaming responses and tool calling
- Marketplace-specific AI tools for service discovery and booking assistance
- Production-ready error handling and analytics integration

## Prerequisites

Before starting, ensure you have:
- Completed [Step 1: OpenAI Setup and Configuration](./01-openai-setup.md)
- OpenAI API key properly configured in environment
- Understanding of Vercel AI SDK core concepts
- Familiarity with SkyMarket's business logic and service categories

## Specification References

This step implements advanced AI patterns described in:
- **AI Specification**: `docs/specs/ai/AI.md` - Core AI requirements and marketplace tools
- **Business Logic**: `docs/specs/business-logic/BUSINESS-LOGIC.md` - Domain context for AI assistant
- **AI SDK Streaming**: [Streaming Text Generation](https://ai-sdk.dev/docs/ai-sdk-core/streaming)
- **Tool Calling**: [Tools and Tool Calling](https://ai-sdk.dev/docs/ai-sdk-core/tools-and-tool-calling)
- **AI Elements**: [Message Components](https://raw.githubusercontent.com/vercel/ai-elements/refs/heads/main/README.md)

## Current Implementation Analysis

SkyMarket implements a focused business assistant using OpenAI's GPT-4o-mini model and the Vercel AI SDK. Let's analyze the existing implementation to understand its architecture and identify enhancement opportunities.

### Chat API Architecture

The implemented chat API (`app/api/chat/route.ts`) demonstrates practical AI SDK patterns:

**Cursor Prompt for Analysis:**
```
Analyze the existing SkyMarket chat API implementation at app/api/chat/route.ts:

1. Examine the AI SDK integration patterns:
   - OpenAI client configuration
   - generateText() usage vs streamText() potential
   - Message conversion and processing
   - Business context loading strategy

2. Identify the business assistant scope:
   - System prompt structure and constraints
   - Document context integration (business-logic, PRD)
   - Service category awareness (food_delivery, courier, aerial_imaging, site_mapping)
   - Detroit Metro geographic constraints

3. Evaluate production readiness:
   - Error handling patterns
   - API key management
   - Fallback mechanisms (echo mode)
   - Token optimization (20k character limit)

Show me the current implementation with analysis of enhancement opportunities.
```

**Key Implementation Insights:**
```typescript
// Current AI SDK integration
import { openai } from '@ai-sdk/openai'
import { generateText, convertToCoreMessages } from 'ai'

// Business-focused system prompt with document context
const systemPrompt = `You are the SkyMarket Business Assistant...`

// Document context loading for business knowledge
let docsContext = ''
const businessLogicPath = resolve(process.cwd(), 'docs/specs/business-logic/BUSINESS-LOGIC.md')
const prdPath = resolve(process.cwd(), 'docs/PRD.md')
```

### Production Patterns Observed

1. **Graceful Fallback**: Echo response when API key is missing
2. **Context Management**: Loads business documentation as context
3. **Token Optimization**: Hard limit of 20k characters for context
4. **Simple Error Handling**: Structured error responses

## Enhanced AI SDK Implementation

### Step 1: Streaming Response Upgrade

**Cursor Prompt for Streaming Enhancement:**
```
Upgrade the existing chat API from generateText() to streamText(). Reference the AI SDK streaming documentation at https://ai-sdk.dev/docs/ai-sdk-core/streaming for implementation patterns. use context7

CURRENT STATE:
- Uses generateText() with plain text response
- Blocks until full response is generated
- No real-time user feedback

ENHANCEMENT REQUIREMENTS:
1. Replace generateText() with streamText()
2. Return streaming response using toDataStreamResponse()
3. Maintain existing business context loading pattern
4. Keep the same system prompt and message processing
5. Preserve error handling and fallback mechanisms
6. Follow the existing app/api/chat/route.ts structure

INTEGRATION WITH EXISTING:
- Maintain compatibility with current chat UI components
- Keep the same API endpoint structure
- Preserve business assistant scope and context
- Reference docs/specs/ai/AI.md for streaming requirements

Show me the complete enhanced implementation with streaming responses.
```

### Step 2: AI Tool Calling Implementation

**Cursor Prompt for Marketplace Tools:**
```
Implement AI SDK tool calling for SkyMarket marketplace operations. Reference the Tool Calling documentation at https://ai-sdk.dev/docs/ai-sdk-core/tools-and-tool-calling for implementation patterns. use context7

BUSINESS CONTEXT:
- SkyMarket drone services marketplace in Detroit Metro
- 4 service categories: food_delivery, courier, aerial_imaging, site_mapping
- Integration with existing Supabase database and auth
- Reference docs/specs/ai/AI.md for tool specifications

REQUIRED TOOLS:
1. searchServices tool:
   - Parameters: category, location, priceRange, availability
   - Query services table with Supabase client
   - Respect Detroit Metro boundaries and RLS policies
   - Return structured service data with provider info

2. estimatePrice tool:
   - Parameters: serviceType, location, requirements
   - Calculate pricing based on category and complexity
   - Include 15% platform fee (from business-logic specs)
   - Return detailed pricing breakdown

3. checkAvailability tool:
   - Parameters: providerId, date, timeSlot
   - Query provider schedules and existing bookings
   - Consider weather and regulatory constraints
   - Return availability with alternative suggestions

INTEGRATION REQUIREMENTS:
- Use existing Supabase patterns from lib/supabase/server.ts
- Follow existing auth middleware and RLS policies
- Maintain compatibility with current chat UI
- Add comprehensive error handling and validation
- Reference components/chat/* for UI integration patterns

Show me the complete tool implementation with proper integration.
```

### Step 3: Production Error Handling

**Cursor Prompt for Robust Error Handling:**
```
Enhance the existing chat API with production-ready error handling following AI SDK best practices:

CURRENT STATE:
- Basic try/catch with generic error responses
- Echo fallback when API key missing (good for development)
- Simple JSON parsing for request validation

ENHANCEMENT REQUIREMENTS:
1. Implement comprehensive error classification:
   - API key validation and quota management
   - Network timeout handling
   - Invalid request validation with Zod schemas
   - Rate limiting and abuse prevention

2. Add structured error responses:
   - User-friendly messages for different error types
   - Proper HTTP status codes
   - Error logging for monitoring and debugging
   - Graceful degradation strategies

3. Integrate with existing patterns:
   - Maintain echo fallback for development
   - Use existing Supabase auth for rate limiting
   - Follow current API route structure
   - Reference error handling patterns from other API routes

4. Security considerations:
   - Never expose API keys or internal errors to client
   - Validate all user input with proper schemas
   - Implement request size limits and timeout handling
   - Add basic DDoS protection patterns

REFERENCE SPECIFICATIONS:
- docs/specs/ai/AI.md for error handling requirements
- Existing API routes for error response patterns
- OpenAI API error handling documentation

Show me the complete enhanced error handling implementation.
```

### Step 4: Analytics and Monitoring Integration

**Cursor Prompt for Analytics Implementation:**
```
Add analytics and monitoring to the chat API following SkyMarket's data patterns:

ANALYTICS REQUIREMENTS:
1. Chat session tracking:
   - Session ID generation and persistence
   - Message count and conversation length
   - Response time and token usage metrics
   - User engagement patterns

2. Performance monitoring:
   - API response time tracking
   - Token consumption analysis
   - Error rate monitoring
   - Tool usage effectiveness

3. Business intelligence:
   - Popular question categories
   - Service discovery patterns through AI
   - Conversion from chat to booking actions
   - User satisfaction indicators

IMPLEMENTATION PATTERNS:
- Use existing Supabase client patterns from lib/supabase/server.ts
- Create analytics tables following existing database schema conventions
- Implement privacy-compliant data collection (no PII storage)
- Add real-time dashboard capabilities for monitoring
- Reference docs/specs/ai/AI.md for analytics requirements

INTEGRATION WITH EXISTING:
- Follow current auth patterns for user identification
- Use existing error logging and monitoring systems
- Maintain performance with async analytics writes
- Integrate with current admin dashboard patterns

Show me the complete analytics implementation with database schema and API integration.
```

## Testing and Validation

### Step 5: Comprehensive Testing Strategy

**Cursor Prompt for Testing Framework:**
```
Create a comprehensive testing strategy for the enhanced chat API:

TESTING SCOPE:
1. Unit Tests:
   - Business context loading and validation
   - AI tool execution and response formatting
   - Error handling and fallback mechanisms
   - Request validation and security measures

2. Integration Tests:
   - End-to-end chat conversations with streaming
   - Tool calling with real Supabase data
   - Authentication and authorization flows
   - Analytics data collection and accuracy

3. Business Logic Tests:
   - Service category recognition and responses
   - Detroit Metro geographic constraint validation
   - Pricing calculation accuracy (15% platform fee)
   - Availability checking with real provider data

4. Performance Tests:
   - Response time under load
   - Token usage optimization
   - Concurrent request handling
   - Analytics write performance impact

TESTING PATTERNS:
- Use existing testing framework and patterns
- Mock external AI API calls for unit tests
- Create test data following database schema
- Implement proper test cleanup and isolation
- Reference docs/specs/ai/AI.md for validation criteria

Show me the complete testing implementation with test cases and validation criteria.
```

### Step 6: Advanced Prompt Engineering

**Cursor Prompt for Business Assistant Optimization:**
```
Optimize the SkyMarket business assistant's prompt engineering for marketplace operations:

CURRENT SYSTEM PROMPT ANALYSIS:
- Basic business scope definition
- Simple document context integration
- Generic tone and format requirements
- Limited interaction patterns

ENHANCEMENT REQUIREMENTS:
1. Marketplace-Specific Personality:
   - Detroit-focused local expertise
   - Drone industry knowledge and terminology
   - Professional but approachable tone
   - Safety and regulatory awareness

2. Advanced Interaction Patterns:
   - Progressive disclosure for complex services
   - Question clarification for better tool usage
   - Contextual follow-up suggestions
   - Error recovery and alternative suggestions

3. Business Intelligence Integration:
   - Service category expertise (food_delivery, courier, aerial_imaging, site_mapping)
   - Pricing transparency and value communication
   - Provider quality indicators and matching
   - Booking flow guidance and optimization

4. Conversational Flow Optimization:
   - Intent recognition for tool activation
   - Multi-turn conversation context
   - Natural transition between discovery and booking
   - Personalization based on user history

REFERENCE SPECIFICATIONS:
- docs/specs/business-logic/BUSINESS-LOGIC.md for business rules
- docs/specs/ai/AI.md for AI assistant requirements
- Existing chat UI components for interaction patterns
- Detroit market research and user feedback data

TESTING REQUIREMENTS:
- A/B test different prompt variations
- Measure conversion rates from chat to booking
- Track user satisfaction and engagement metrics
- Validate business scope adherence

Show me optimized system prompts with conversation flow examples and testing methodology.
```

## Expert Implementation Tasks

### AI SDK Integration Mastery
- [ ] Upgrade to streaming responses with real-time feedback
- [ ] Implement marketplace-specific tool calling
- [ ] Add comprehensive error handling and analytics
- [ ] Optimize prompt engineering for conversions

### Advanced Development Patterns
- [ ] Use spec-driven Cursor prompts for rapid AI feature development
- [ ] Integrate with existing Supabase patterns and auth flows
- [ ] Follow production-ready security and monitoring practices
- [ ] Test AI features with real marketplace scenarios

### Business Logic Integration
- [ ] Map AI capabilities to Detroit drone service operations
- [ ] Design intelligent service discovery and booking assistance
- [ ] Implement pricing transparency and provider matching
- [ ] Create conversion optimization through AI guidance

### Performance and Production Readiness
- [ ] Implement streaming responses for better UX
- [ ] Add analytics dashboard for AI feature monitoring
- [ ] Create comprehensive testing strategy with real data
- [ ] Optimize for cost efficiency and scalability

## Key Development Insights

1. **Spec-Driven Approach**: Use detailed Cursor prompts referencing actual specifications
2. **Business Integration**: AI features must enhance core marketplace operations
3. **Iterative Enhancement**: Build upon existing simple implementation progressively
4. **Production Mindset**: Consider monitoring, analytics, and error handling from start
5. **User Experience**: Focus on conversion optimization through intelligent assistance

## Next Steps

With AI SDK integration mastered, you're ready to implement sophisticated AI Elements UI components that create seamless conversational experiences.

---

**Previous**: [Step 1: OpenAI Setup](./01-openai-setup.md) ← | → **Next**: [Step 3: AI Elements and Chat Interface](./03-ai-elements.md)

**Focus**: Advanced UI patterns for AI-powered marketplace interactions