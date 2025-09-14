# Step 1: OpenAI Setup and Configuration

## What You'll Accomplish

In this step, you'll establish a **production-ready OpenAI integration** for SkyMarket. This includes secure API key management, billing optimization, and Vercel AI SDK configuration designed for marketplace AI features.

**Key Outcomes**:
- Secure OpenAI API key setup with environment management
- Billing and usage monitoring configuration
- Vercel AI SDK integration with SkyMarket architecture
- Basic AI endpoint validation and testing

## Prerequisites

Before starting, ensure you have:
- Completed the [Expert Track Introduction](./00-introduction.md)
- SkyMarket development environment running
- Basic understanding of AI API integration
- Production mindset for security and cost management

## Specification References

This step implements the AI foundation described in:
- **AI Specification**: `docs/specs/ai/AI.md` - Core AI requirements and architecture
- **Vercel AI Elements**: [AI Elements README](https://raw.githubusercontent.com/vercel/ai-elements/refs/heads/main/README.md)
- **AI SDK Core**: [Text Generation Guide](https://ai-sdk.dev/docs/ai-sdk-core/generating-text)
- **AI SDK UI**: [Chatbot Implementation](https://ai-sdk.dev/docs/ai-sdk-ui/chatbot)

## OpenAI Account Setup

### Step 1: Create OpenAI API Account

**Manual Setup** (Required):
1. Visit [platform.openai.com](https://platform.openai.com)
2. Create account or sign in
3. Navigate to **API Keys** section
4. Set up billing with a payment method
5. Review pricing and set usage limits

**Expert Insight**: Set conservative usage limits initially. AI features can consume tokens quickly, especially during development and testing.

### Step 2: Generate Production-Ready API Key

**Security Best Practices**:
```
API Key Configuration:
- Name: "skymarket-production" or "skymarket-development"
- Permissions: Restricted to necessary models only
- Usage Limits: Set monthly spending caps
- IP Restrictions: Add if deploying to known infrastructure
```

**Advanced Configuration**:
- Enable usage tracking and alerts
- Set up separate keys for development and production
- Configure model access (GPT-4, GPT-4 Turbo, future GPT-5)
- Review rate limits for your use case

## Environment Configuration

### Step 3: Secure Environment Setup

**Cursor Prompt for Environment Management**:
```
Configure OpenAI API integration for a Next.js marketplace application:

1. Add environment variables to .env.local:
   - OPENAI_API_KEY (for server-side AI calls)
   - OPENAI_ORG_ID (optional, for organization tracking)
   - AI_MODEL_PREFERENCE (default model selection)
   - AI_MAX_TOKENS (default token limits)

2. Update .env.example with placeholder values
3. Ensure .env.local is properly gitignored
4. Add environment validation for production deployment

SECURITY REQUIREMENTS:
- Never expose API keys to client-side code
- Use environment validation to prevent deployment without keys
- Include fallback configurations for different environments
- Add logging for API usage monitoring

Show me the complete environment configuration.
```

**Advanced Environment Pattern**:
```typescript
// lib/ai/config.ts
const validateEnv = () => {
  if (!process.env.OPENAI_API_KEY) {
    throw new Error('OPENAI_API_KEY is required');
  }

  return {
    apiKey: process.env.OPENAI_API_KEY,
    orgId: process.env.OPENAI_ORG_ID,
    defaultModel: process.env.AI_MODEL_PREFERENCE || 'gpt-4-turbo-preview',
    maxTokens: parseInt(process.env.AI_MAX_TOKENS || '1000'),
    environment: process.env.NODE_ENV,
  };
};

export const aiConfig = validateEnv();
```

## Vercel AI SDK Integration

### Step 4: Install and Configure AI SDK

**Installation Command**:
```bash
npm install ai @ai-sdk/openai @ai-sdk/ui
```

**Cursor Prompt for AI SDK Setup**:
```
Set up Vercel AI SDK integration for SkyMarket marketplace. Reference the AI SDK Core documentation at https://ai-sdk.dev/docs/ai-sdk-core/generating-text for official patterns. use context7

1. Create AI configuration at lib/ai/client.ts:
   - Configure OpenAI client with API key using @ai-sdk/openai
   - Set up default model preferences (GPT-4, GPT-5 when available)
   - Include error handling and retries as per AI SDK patterns
   - Add usage tracking capabilities

2. Create AI utilities at lib/ai/utils.ts:
   - Token counting functions using AI SDK utilities
   - Cost estimation utilities
   - Request/response logging compatible with AI SDK
   - Error classification and handling

3. Follow AI Elements patterns. Reference https://raw.githubusercontent.com/vercel/ai-elements/refs/heads/main/README.md for component patterns. use context7
4. Ensure integration with existing Supabase auth
5. Add TypeScript types for AI responses matching AI SDK types
6. Include rate limiting considerations per docs/specs/ai/AI.md

INTEGRATION REQUIREMENTS:
- Work with existing authentication system
- Support marketplace-specific prompts per AI specification
- Handle concurrent requests efficiently using AI SDK streaming
- Include proper error boundaries following AI SDK best practices
- Reference actual components/chat/* and app/api/chat/route.ts patterns

Show me the complete setup with TypeScript types matching AI SDK standards.
```

### Step 5: Basic AI Endpoint Creation

**Cursor Prompt for Test Endpoint**:
```
Create a test AI endpoint at app/api/ai/test/route.ts following the existing app/api/chat/route.ts patterns. Reference AI SDK documentation at https://ai-sdk.dev/docs/ai-sdk-core/generating-text for implementation patterns. use context7

PURPOSE:
- Validate OpenAI API connection using AI SDK
- Test Vercel AI SDK integration with streaming responses
- Verify authentication and environment setup
- Provide debugging information for development

ENDPOINT SPECIFICATION:
- Follow patterns from existing app/api/chat/route.ts
- Accept POST requests with test prompts
- Use AI SDK streaming response for real-time feedback (streamText from @ai-sdk/core)
- Include request logging and usage tracking per docs/specs/ai/AI.md
- Return structured responses with metadata matching AI SDK types
- Handle authentication with existing Supabase system

RESPONSE FORMAT:
- Stream AI response content using AI SDK streamText
- Include token usage statistics from AI SDK response
- Add request timing information
- Provide error details for debugging
- Follow AI Elements patterns for consistent UI integration

SECURITY:
- Validate user authentication using existing Supabase patterns
- Rate limit requests per user following AI specification
- Log all interactions for monitoring
- Never expose API keys or sensitive config
- Implement tool calling security if needed. Reference https://ai-sdk.dev/docs/ai-sdk-core/tools-and-tool-calling for security patterns. use context7

INTEGRATION REQUIREMENTS:
- Reference existing components/chat/* for UI patterns
- Use components/ai-elements/* if available
- Follow GPT-5 best practices. Reference https://ai-sdk.dev/cookbook/guides/gpt-5#openai-gpt-5 for optimization patterns. use context7
- Ensure compatibility with AI chatbot patterns. Reference https://ai-sdk.dev/docs/ai-sdk-ui/chatbot for UI patterns. use context7

Show me the complete implementation with proper error handling and AI SDK integration.
```

## Production Readiness

### Step 6: Monitoring and Optimization

**Cursor Prompt for Usage Monitoring**:
```
Create AI usage monitoring and optimization utilities:

1. Usage Tracking (lib/ai/monitoring.ts):
   - Track token consumption per request
   - Monitor API response times
   - Log usage patterns by feature
   - Calculate cost per interaction

2. Cost Optimization (lib/ai/optimization.ts):
   - Token-efficient prompt templates
   - Response caching for repeated queries
   - Model selection based on complexity
   - Batch processing for multiple requests

3. Error Monitoring (lib/ai/errors.ts):
   - Classify API error types
   - Implement retry strategies with backoff
   - Track error rates and patterns
   - Alert on usage spikes or failures

4. Integration with existing analytics
5. Dashboard for tracking AI feature performance

Include comprehensive TypeScript types and validation.
```

### Step 7: Security and Rate Limiting

**Advanced Security Configuration**:
```typescript
// lib/ai/security.ts
export const aiSecurityConfig = {
  // Rate limiting per user/IP
  rateLimits: {
    perUser: { requests: 100, window: '1h' },
    perIP: { requests: 200, window: '1h' },
    perEndpoint: { requests: 1000, window: '1h' }
  },

  // Input validation and sanitization
  inputValidation: {
    maxPromptLength: 2000,
    allowedContentTypes: ['text/plain', 'application/json'],
    sanitizeUserInput: true
  },

  // Response filtering
  outputFiltering: {
    preventDataLeakage: true,
    filterPII: true,
    contentModeration: true
  }
};
```

## Testing and Validation

### Step 8: Comprehensive Testing Strategy

**Cursor Prompt for AI Testing Framework**:
```
Create a comprehensive testing framework for AI integration:

1. Unit Tests (tests/ai/unit/):
   - API client configuration
   - Prompt template rendering
   - Response parsing and validation
   - Error handling scenarios

2. Integration Tests (tests/ai/integration/):
   - End-to-end API calls with real responses
   - Authentication flow validation
   - Rate limiting behavior
   - Database integration testing

3. Performance Tests (tests/ai/performance/):
   - Response time benchmarks
   - Token usage optimization
   - Concurrent request handling
   - Memory usage monitoring

4. Security Tests (tests/ai/security/):
   - API key protection validation
   - Input sanitization testing
   - Rate limiting verification
   - Authorization boundary testing

Include Jest configuration and test utilities.
```

**Validation Checklist**:
```
AI Integration Validation:
□ OpenAI API connection successful
□ Environment variables properly configured
□ API key security verified (not exposed client-side)
□ Rate limiting functional
□ Error handling comprehensive
□ Usage monitoring operational
□ Authentication integration working
□ TypeScript types complete
□ Test coverage >80%
□ Performance benchmarks established
```

## Development Best Practices

### Expert-Level AI Development Patterns

**Prompt Engineering for Marketplace**:
```typescript
// lib/ai/prompts/marketplace.ts
export const marketplacePrompts = {
  serviceRecommendation: (context: UserContext) => `
    You are an expert drone service advisor for SkyMarket.

    User Context:
    - Location: ${context.location}
    - Previous Orders: ${context.orderHistory}
    - Preferences: ${context.preferences}

    Recommend 3 relevant services with specific reasoning.
    Format as JSON with service IDs and explanations.
  `,

  providerAssistance: (issue: string) => `
    You are helping a drone service provider on SkyMarket.

    Issue: ${issue}

    Provide practical, actionable advice specific to drone operations.
    Include safety considerations and regulatory compliance.
  `
};
```

**Error Handling Patterns**:
```typescript
// lib/ai/errors.ts
export class AIServiceError extends Error {
  constructor(
    message: string,
    public code: string,
    public retryable: boolean = false
  ) {
    super(message);
    this.name = 'AIServiceError';
  }
}

export const handleAIError = (error: unknown): AIServiceError => {
  if (error instanceof OpenAIError) {
    switch (error.status) {
      case 429:
        return new AIServiceError('Rate limit exceeded', 'RATE_LIMIT', true);
      case 401:
        return new AIServiceError('Invalid API key', 'AUTH_ERROR', false);
      default:
        return new AIServiceError('AI service error', 'SERVICE_ERROR', true);
    }
  }

  return new AIServiceError('Unknown error', 'UNKNOWN_ERROR', false);
};
```

## Common Setup Issues and Solutions

### Issue 1: API Key Configuration
**Problem**: API keys not working in production
**Solution**: Verify environment variable names and deployment configuration

### Issue 2: High Token Usage
**Problem**: Unexpected API costs during development
**Solution**: Implement token counting and usage alerts

### Issue 3: Rate Limiting
**Problem**: API requests being throttled
**Solution**: Implement proper rate limiting and retry strategies

### Issue 4: Authentication Integration
**Problem**: AI endpoints not respecting user authentication
**Solution**: Ensure all AI routes use existing auth middleware

## Next Steps

With OpenAI properly configured, you're ready to implement AI-powered features:

In **Step 2: AI SDK Integration and Implementation**, you'll:
- Build your first marketplace-specific AI endpoint
- Implement streaming responses for better UX
- Create reusable AI components and utilities
- Integrate with existing SkyMarket authentication and database

---

**Previous**: [Step 0: Introduction](./00-introduction.md) ← | → **Next**: [Step 2: AI SDK Integration](./02-ai-sdk-integration.md)