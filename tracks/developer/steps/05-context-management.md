# Step 5: Context Management

## Objective

Master Cursor's context system to effectively combine project specifications with MCP Context7 documentation for optimal AI-assisted development.

## Understanding Context in Cursor

Context is everything the AI knows about your request. In spec-driven development, you carefully curate context to include:

1. **Project Specifications** - What to build (requirements, business rules)
2. **Official Documentation** - How to build (current patterns, best practices)
3. **Project Files** - Current implementation state
4. **Conversation History** - Previous decisions and context

### The Context Window

Cursor has a limited context window (similar to human short-term memory). Managing this efficiently is crucial for effective spec-driven development.

**Context Window Sizes**:
- **Claude 3.5 Sonnet**: ~200K tokens (~150K words)
- **Typical spec file**: 2-5K tokens
- **Context7 response**: 3-8K tokens
- **Conversation**: Grows over time

## The @ Mention System

### Basic @ Mentions

**File References**:
```
@filename.ext           # Single file
@folder/file.ext        # File in subfolder
@docs/specs/API.md      # Specific specification
```

**Folder References**:
```
@docs/specs/            # Entire specs folder (use carefully)
@components/ui/         # Component folder
@lib/supabase/          # Library utilities
```

### Advanced @ Mention Patterns

**Multiple File Loading**:
```
@docs/specs/authentication/AUTHENTICATION.md @docs/specs/architecture/DATABASE.md @AGENTS.md

Based on these three specifications, implement user signup...
```

**Selective Context Loading**:
```
# Don't do this (too much context):
@docs/specs/

# Do this (specific files):
@docs/specs/authentication/AUTHENTICATION.md @docs/specs/payment/PAYMENT.md
```

**Project Configuration Context**:
```
@AGENTS.md @package.json @tsconfig.json

Using our project configuration, set up the development environment...
```

## Context7 Integration Patterns

### Natural Language + Specifications

**Step 1: Load Specifications**
```
@docs/specs/authentication/AUTHENTICATION.md
```

**Step 2: Create Combined Implementation Prompt**
```
@docs/specs/authentication/AUTHENTICATION.md
@AGENTS.md

Using the authentication specification, implement email/password signup with Supabase for Next.js 15 App Router:

1. Implement email/password signup with the exact flow from our spec
2. Use Supabase's current best practices for Next.js 15 App Router
3. Include proper TypeScript types and error handling
4. Follow our project conventions from AGENTS.md

use context7
```

**The Power of "use context7"**:
- Context7 automatically fetches current official documentation
- Natural language only - no commands needed
- Seamlessly integrates with project specifications
- Always provides up-to-date library patterns and best practices
- Perfect for complex payment implementations like Stripe processing

### Effective Context7 Prompt Patterns

**Basic Pattern**:
```
@docs/specs/[feature]/FEATURE.md
@AGENTS.md

[Your specific implementation request with project context]

use context7
```

**Advanced Marketplace Integration Pattern**:
```
@docs/specs/payment/PAYMENT.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/architecture/DATABASE.md
@AGENTS.md

Implement comprehensive Stripe payment infrastructure for SkyMarket drone services:

**Core Payment System Requirements:**
1. Secure payment processing with comprehensive verification for drone operators
2. Payment intent creation with dynamic pricing calculation and secure handling
3. Webhook endpoints processing payment_intent.succeeded and status update events
4. Consumer checkout flow with secure authentication and multiple payment methods
5. Automatic payment confirmation triggered by booking completion (status: completed)

**Provider Integration Requirements:**
6. Provider earnings dashboard with transaction history and status tracking
7. Payment analytics with detailed transaction reporting features
8. Transaction status management and monitoring capabilities

**Business Logic Integration:**
9. Geographic validation for Detroit Metro service area compliance
10. Service category-specific pricing for 4 drone service types
11. Integration with booking state machine and real-time status updates
12. Database transaction handling for payment and booking status synchronization

Use current Stripe payment processing patterns, Next.js 15 App Router, and Supabase integration.

use context7
```

## Context Management Strategies

### 1. Layer Your Context

**Foundation Layer**: Always load these
- `@AGENTS.md` - Project conventions
- `@package.json` - Dependencies and scripts
- Primary specification for current feature

**Feature Layer**: Add as needed
- Related specification files
- Relevant Context7 documentation
- Current implementation files

**Implementation Layer**: For specific tasks
- Target files you're modifying
- Related utility functions
- Test files

### 2. Context Lifecycle Management

**Start of Feature**:
```
# Clean context - start new chat
@docs/specs/[feature]/FEATURE.md
@AGENTS.md

Plan the implementation of [feature] following our specification and current best practices.

use context7
```

**During Implementation**:
```
# Add specific files as you work on them
@components/auth/LoginForm.tsx
@lib/supabase/auth.ts

Update these components to handle the new authentication flow using current patterns.

use context7
```

**Review and Refinement**:
```
# Include test files and validation
@__tests__/auth.test.ts
@docs/specs/authentication/AUTHENTICATION.md

Verify this implementation matches all specification requirements and follows current best practices.

use context7
```

### 3. Context Window Optimization

**Efficient Loading**:
```
# Good - specific and focused
@docs/specs/payment/PAYMENT.md @lib/stripe/server.ts

Implement payment intent creation using current Stripe patterns.

use context7

# Bad - too broad
@docs/specs/ @lib/ @components/

Implement payments.

use context7
```

**Context Pruning**:
- Start fresh chat when switching to different features
- Remove irrelevant files from context as you progress
- Use specific file sections rather than entire large files

## Effective Prompt Engineering

### 1. Structured Prompt Template

```
[Context Loading]
@relevant-spec.md
@AGENTS.md

[Implementation Request]
Based on [specification], implement [feature] with [library]:

[Specific Requirements]
1. Requirement from specification
2. Project convention from AGENTS.md
3. Business logic constraint
4. Performance/security requirement

[Output Format]
- Complete implementation with TypeScript
- Proper error handling and validation
- Tests where applicable
- Documentation comments

[Verification Criteria]
- Matches specification requirements exactly
- Follows current best practices
- Includes proper TypeScript types
- Handles all edge cases from spec

use context7
```

### 2. Progressive Refinement

**Initial Implementation**:
```
@docs/specs/payment/PAYMENT.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@AGENTS.md

Create the core Stripe payment processing system outlined in our specifications:
- Express account creation for drone service providers
- Payment intent creation with 15% platform fee structure
- Basic webhook handling for payment events
- Consumer checkout flow with escrow functionality

Use Stripe's current payment patterns for Next.js 15 App Router.

use context7
```

**Refinement Round 1**:
```
Enhance the payment system with the advanced features from our specifications:
1. KYC verification status tracking and provider dashboard updates
2. 3D Secure authentication support for consumer payments
3. Automatic payout handling when bookings reach 'completed' status
4. Comprehensive webhook event processing (account updates, payout events)
5. Error handling for failed payments and retry logic

Implement current Stripe best practices for payment processing.

use context7
```

**Refinement Round 2**:
```
@docs/specs/security/SECURITY.md
@docs/specs/api/WEBHOOKS.md

Add security and reliability measures from our specifications:
1. Webhook signature verification with proper error handling
2. Idempotency handling for duplicate webhook events
3. Rate limiting for payment API endpoints
4. PCI compliance patterns for payment data handling
5. Audit logging for all financial transactions

Apply current security and webhook best practices for financial applications.

use context7
```

### 3. Verification Prompts

```
@docs/specs/authentication/AUTHENTICATION.md

Review the authentication implementation and verify:
1. All flows from the spec are implemented
2. Error handling covers all spec requirements
3. TypeScript types match our database schema
4. Security requirements are met

Provide a checklist of verified items and any missing pieces.
```

## Common Context Patterns for SkyMarket

### Authentication Features
```
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/architecture/DATABASE.md
@AGENTS.md

Implement complete Supabase authentication system for Next.js 15 App Router with:
- Multi-role signup/login (consumer, provider, admin)
- Email verification with custom templates
- Server-side session management with middleware
- Social auth integration (Google, Apple)
- Profile creation with role-specific fields
- TypeScript integration with generated database types

use context7
```

### Stripe Payment Processing
```
@docs/specs/payment/PAYMENT.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@AGENTS.md

Implement comprehensive Stripe payment system for drone services:
- Secure payment processing with verification
- Payment intent creation with variable pricing structure
- Secure transaction handling until service completion
- Webhook handling for payment events and status updates
- Provider dashboard with earnings and transaction tracking
- Consumer checkout with multiple payment methods and secure authentication
- Automatic payment confirmation upon booking completion
- Refund handling for cancelled services

use context7
```

### Booking System Integration
```
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/architecture/DATABASE.md
@AGENTS.md

Create complete booking management system for drone services:
- Service discovery with location-based filtering (Detroit Metro area)
- Real-time availability checking and booking creation
- Booking state management (pending → accepted → in_progress → completed)
- Integration with payment system for status synchronization
- Provider acceptance/rejection workflow with notifications
- Consumer tracking and updates throughout service lifecycle
- Automated booking confirmation and reminder emails

use context7
```

### UI Components with shadcn/ui
```
@docs/specs/components/COMPONENTS.md
@AGENTS.md

Create production-ready UI components using React, TypeScript, Tailwind CSS, and shadcn/ui:
- Responsive booking forms with date/time selection and location input
- Payment checkout components with loading states and error handling
- Provider dashboard cards showing verification status and earnings
- Interactive service listings with filtering and search
- Real-time notification components with proper accessibility
- Mobile-first design patterns with proper touch interactions

use context7
```

## Troubleshooting Context Issues

### Common Problems

**Problem**: Context too large, responses truncated
**Solution**:
```
# Remove unnecessary files
# Start fresh chat
# Load only essential context for current task
```

**Problem**: AI doesn't understand specifications
**Solution**:
```
# Load specification first
# Ask AI to summarize the spec
# Then proceed with implementation requests
```

**Problem**: Generated code doesn't follow project patterns
**Solution**:
```
# Always include @AGENTS.md in context
# Reference project conventions explicitly
# Ask AI to explain its approach before generating code
```

**Problem**: Context7 responses conflict with specifications
**Solution**:
```
# Explicitly state precedence in prompts:
"Follow our SkyMarket specifications exactly, then apply current best practices from Context7 documentation where they enhance but don't conflict with our requirements."

# Example with clear precedence:
@docs/specs/payment/PAYMENT.md

Implement our variable pricing structure and secure payment handling as specified, using Stripe's current payment processing patterns for the technical implementation.

use context7
```

### Performance Optimization

**Slow Responses**:
- Reduce context size
- Use more specific files/sections
- Clear conversation history periodically
- Avoid loading large folders

**Incomplete Responses**:
- Split complex requests into smaller parts
- Use follow-up prompts for details
- Ask for specific sections/functions individually

**Inconsistent Results**:
- Use consistent context loading patterns
- Include project conventions (@AGENTS.md) always
- Reference specifications explicitly in prompts

## Expected Outcome

After completing this step, you should have:

### Context Management Skills
- [ ] Understand Cursor's context window limitations
- [ ] Can effectively use @ mentions for file loading
- [ ] Know when to start fresh chats vs continue existing ones
- [ ] Can optimize context for performance

### Integration Mastery
- [ ] Can combine specifications with Context7 documentation
- [ ] Use structured prompt templates effectively
- [ ] Apply progressive refinement for complex features
- [ ] Verify implementations against requirements

### Troubleshooting Abilities
- [ ] Recognize context-related issues
- [ ] Apply appropriate solutions for common problems
- [ ] Optimize prompts for best results
- [ ] Manage context lifecycle effectively

## Pro Tips

### Context Strategy
- **Start simple**: Load essential specs (@AGENTS.md + feature spec), add related files as needed
- **Be specific**: Use exact file paths and detailed implementation requirements
- **Stay organized**: Follow consistent spec-driven patterns for all features
- **Verify understanding**: Ask AI to summarize specs and approach before implementing

### Performance Tips
- **Fresh starts**: New chat for major features (authentication, payments, booking system)
- **Context awareness**: Monitor context size - SkyMarket specs are detailed
- **Batch strategically**: Load related specs together (payment + business-logic)
- **Prune regularly**: Remove irrelevant context, keep @AGENTS.md always

### Quality Assurance
- **Always include**: @AGENTS.md for project conventions and architecture patterns
- **Spec precedence**: Explicitly state "follow specifications first" in complex prompts
- **Test Context7 integration**: Verify current patterns work with SkyMarket requirements
- **Review systematically**: Use verification prompts with both specs and Context7
- **Payment focus**: Prioritize Stripe payments, Supabase, and Next.js 15 patterns

## Next Steps

Excellent! You now understand how to manage context effectively for spec-driven development. Next, we'll explore the project specifications structure and learn to navigate them efficiently.

### What's Coming in Step 6
- Navigate the docs/specs/ folder structure
- Understand how specifications interconnect
- Learn to choose the right specs for each feature
- Practice combining specifications effectively

---

**Next Step**: [Step 6: Specification Navigation](./06-spec-navigation.md) →

**Quick Links**:
- [Previous: MCP Context7 Setup](./04-mcp-context7.md)
- [Track Overview](../README.md)
- [Project Specifications](../../../specs/)