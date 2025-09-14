# Expert Step 05: Cursor AI Development Mastery - Production AI Patterns

## Objective
Master advanced Cursor prompting techniques for AI-powered development workflows. Learn spec-driven development, context-aware prompting, and production deployment patterns for AI features.

## What You'll Master
- Expert-level Cursor prompting strategies for complex AI implementations
- Spec-driven development workflows for marketplace features
- Context-aware AI assistance and prompt engineering
- Production deployment patterns for AI-powered applications
- Monitoring, analytics, and performance optimization for AI features

## Prerequisites
- Completed Expert Steps 01-04 (Full AI implementation analysis and tools)
- Experience with the SkyMarket codebase and business logic
- Understanding of production deployment and monitoring concepts
- Familiarity with advanced development workflows

## Expert Cursor Prompting Patterns

### 1. Spec-Driven AI Development

**Context-Rich Development Prompts:**
```typescript
/**
 * @cursor-prompt: Enhance the SkyMarket chat API with advanced features:
 *
 * CONTEXT:
 * - Current implementation: app/api/chat/route.ts (simple generateText)
 * - Business focus: Detroit drone services marketplace
 * - Four categories: food_delivery, courier, aerial_imaging, site_mapping
 * - Document context loading from business-logic and PRD
 *
 * REQUIREMENTS:
 * 1. Upgrade to streamText() for better UX
 * 2. Add comprehensive tool calling:
 *    - Service search with Supabase integration
 *    - Price estimation with 15% platform fee
 *    - Weather validation for drone operations
 *    - Availability checking with provider schedules
 * 3. Implement proper error handling and fallbacks
 * 4. Add request validation with Zod schemas
 * 5. Include usage analytics to Supabase
 *
 * CONSTRAINTS:
 * - Maintain existing business context loading pattern
 * - Keep the professional, concise response style
 * - Ensure Detroit Metro geographic boundaries
 * - Respect existing RLS policies and permissions
 * - Must handle missing API key gracefully (echo fallback)
 *
 * OUTPUT: Complete enhanced implementation with detailed comments
 */
```

### 2. Component Enhancement Workflows

**Advanced Component Prompting:**
```typescript
/**
 * @cursor-prompt: Create an enhanced ChatWindow component:
 *
 * ANALYZE CURRENT:
 * - components/chat/ChatWindow.tsx uses manual streaming
 * - localStorage persistence with session management
 * - Manual message state management
 * - Basic error handling and abort control
 *
 * ENHANCE WITH:
 * 1. Tool result display components:
 *    - ServiceCard for search results with booking actions
 *    - WeatherCard for drone operation conditions
 *    - PriceBreakdown for cost estimations
 * 2. Advanced UX features:
 *    - Typing indicators during streaming
 *    - Message timestamps and reactions
 *    - Quick action buttons for common queries
 * 3. Better error handling:
 *    - Graceful tool failure recovery
 *    - Retry mechanisms with exponential backoff
 *    - User-friendly error messages
 *
 * MAINTAIN:
 * - Existing localStorage pattern
 * - Dynamic loading optimization
 * - Accessibility features
 * - Mobile responsiveness
 *
 * INTEGRATION:
 * - Work with existing AI Elements components
 * - Follow SkyMarket design system
 * - Ensure type safety throughout
 */
```

### 3. Database Integration Patterns

**Supabase-Aware Development:**
```typescript
/**
 * @cursor-prompt: Create a comprehensive service search tool:
 *
 * BUSINESS CONTEXT:
 * - SkyMarket drone services marketplace in Detroit Metro
 * - Four service categories with specific pricing models
 * - Provider verification and rating systems
 * - Geographic constraints and regulatory compliance
 *
 * DATABASE SCHEMA (infer from existing):
 * - services table: id, title, description, price, category, provider_id
 * - profiles table: business_name, rating, verified, location
 * - bookings table: status, schedule, requirements
 *
 * TOOL IMPLEMENTATION:
 * 1. Service search with advanced filtering:
 *    - Category, location, price range, availability
 *    - Provider ratings and verification status
 *    - Distance-based sorting from user location
 * 2. Real-time availability checking:
 *    - Cross-reference booking schedules
 *    - Weather condition validation
 *    - Regulatory compliance checks
 * 3. Intelligent recommendations:
 *    - Similar services if exact match unavailable
 *    - Alternative time slots for better pricing
 *    - Provider suggestions based on history
 *
 * OUTPUT: Complete tool implementation with Zod schemas, error handling, and TypeScript types
 */
```

## Production Deployment Patterns

### 1. Environment Configuration

**Production-Ready Setup:**
```bash
# Environment variables for production AI features
OPENAI_API_KEY=sk-proj-production-key
OPENAI_ORGANIZATION=org-skymarket-prod

# AI feature flags
NEXT_PUBLIC_AI_CHAT_ENABLED=true
NEXT_PUBLIC_AI_TOOLS_ENABLED=true
AI_RATE_LIMIT_PER_USER=20
AI_RATE_LIMIT_WINDOW=300

# Monitoring and analytics
AI_ANALYTICS_ENABLED=true
AI_ERROR_TRACKING=true
```

### 2. Performance Optimization

**Cursor Prompt for Optimization:**
```typescript
/**
 * @cursor-prompt: Optimize AI feature performance:
 *
 * CURRENT PERFORMANCE ISSUES:
 * - Document context loading on every request (20k chars)
 * - No caching for repeated queries
 * - Tool execution without timeout handling
 * - Large bundle size from AI components
 *
 * OPTIMIZATION STRATEGIES:
 * 1. Document context caching:
 *    - Cache business documents in memory/Redis
 *    - Implement versioning for cache invalidation
 *    - Compress context while preserving key information
 * 2. Response caching:
 *    - Cache common AI responses (FAQ patterns)
 *    - Implement semantic similarity for cache hits
 *    - Respect user-specific vs general cacheable content
 * 3. Bundle optimization:
 *    - Dynamic imports for AI components
 *    - Code splitting for tool-specific functionality
 *    - Lazy loading of heavy dependencies
 * 4. Request optimization:
 *    - Implement request deduplication
 *    - Add timeout handling for tool execution
 *    - Batch similar requests when possible
 *
 * OUTPUT: Enhanced implementation with performance metrics and monitoring
 */
```

### 3. Monitoring and Analytics

**Production Monitoring Setup:**
```typescript
/**
 * @cursor-prompt: Create AI feature analytics system:
 *
 * ANALYTICS REQUIREMENTS:
 * 1. Usage tracking:
 *    - Chat sessions started/completed
 *    - Tool invocation frequency and success rates
 *    - User satisfaction and feedback
 * 2. Performance metrics:
 *    - Response times by tool and complexity
 *    - Token usage and cost tracking
 *    - Error rates and failure patterns
 * 3. Business metrics:
 *    - Conversion from chat to booking
 *    - Most requested service types
 *    - Geographic usage patterns
 *
 * IMPLEMENTATION:
 * 1. Supabase analytics tables:
 *    - ai_chat_sessions: session tracking
 *    - ai_tool_usage: tool invocation logs
 *    - ai_performance_metrics: response times, errors
 * 2. Real-time dashboards:
 *    - Usage overview and trends
 *    - Error monitoring and alerts
 *    - Cost optimization insights
 * 3. User experience tracking:
 *    - Conversation flow analysis
 *    - Drop-off points identification
 *    - Feature adoption rates
 *
 * OUTPUT: Complete analytics system with dashboard components
 */
```

## Advanced Development Workflows

### 1. Iterative AI Feature Development

**Development Cycle Pattern:**
1. **Analyze**: Study existing implementations and user feedback
2. **Spec**: Create detailed specifications with business context
3. **Prompt**: Use Cursor for initial implementation
4. **Refine**: Iterate with targeted enhancement prompts
5. **Test**: Validate with real marketplace scenarios
6. **Deploy**: Monitor and optimize based on usage data

### 2. Context-Aware Prompting Strategies

**Progressive Enhancement Approach:**
```typescript
/**
 * Phase 1: Basic functionality with business context
 * Phase 2: Enhanced UX with tool integration
 * Phase 3: Advanced features with analytics
 * Phase 4: Production optimization and scaling
 */
```

### 3. Cross-Feature Integration

**Marketplace-Wide AI Enhancement:**
```typescript
/**
 * @cursor-prompt: Integrate AI assistance across SkyMarket:
 *
 * SERVICE DISCOVERY (browse pages):
 * - AI-powered search suggestions
 * - Natural language filtering
 * - Personalized recommendations
 *
 * BOOKING FLOW (booking pages):
 * - Intelligent form completion
 * - Schedule optimization
 * - Requirement validation
 *
 * PROVIDER DASHBOARD:
 * - Service optimization suggestions
 * - Pricing recommendations
 * - Customer communication assistance
 *
 * CUSTOMER SUPPORT:
 * - Automated issue resolution
 * - FAQ enhancement
 * - Escalation routing
 *
 * Create a unified AI assistance system with consistent UX
 */
```

## Expert Implementation Tasks

### Production Readiness
- [ ] Implement comprehensive error handling and monitoring
- [ ] Add performance optimization and caching strategies
- [ ] Create analytics dashboard for AI feature usage
- [ ] Set up automated testing for AI tool functionality

### Advanced Features
- [ ] Build context-aware AI assistance across the platform
- [ ] Implement user preference learning and personalization
- [ ] Create advanced tool chaining for complex workflows
- [ ] Add voice interface support for accessibility

### Scaling and Optimization
- [ ] Implement rate limiting and quota management
- [ ] Add A/B testing framework for AI features
- [ ] Create cost optimization and token usage monitoring
- [ ] Build automated failover and disaster recovery

### Developer Experience
- [ ] Create AI feature development guidelines and patterns
- [ ] Build testing utilities for AI functionality
- [ ] Document advanced Cursor prompting strategies
- [ ] Establish code review processes for AI features

## Key Insights for Expert Development

1. **Spec-First Approach**: Always start with detailed specifications that include business context, technical constraints, and user experience requirements

2. **Iterative Enhancement**: Use Cursor's context awareness to build upon existing implementations rather than starting from scratch

3. **Production Thinking**: Consider monitoring, analytics, error handling, and performance optimization from the beginning

4. **Business Integration**: AI features should enhance core marketplace operations, not replace them

5. **User Experience Focus**: AI should feel natural and helpful, not intrusive or confusing

## Expected Outcome

After mastering these advanced Cursor workflows, you'll be able to:
- Rapidly prototype and implement sophisticated AI features
- Create production-ready AI systems with proper monitoring and optimization
- Build context-aware AI assistance that enhances user experience
- Develop and maintain complex AI integrations efficiently

---

**Congratulations!** You've completed the Expert Track for AI implementation in SkyMarket. You now have the skills to build, deploy, and maintain sophisticated AI-powered marketplace features using modern development tools and practices.

**Next Steps**: Apply these patterns to build advanced AI features specific to your marketplace needs, always focusing on user value and business impact.