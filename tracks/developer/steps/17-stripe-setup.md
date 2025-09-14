# Step 17: Stripe Setup - Cursor & AI Prompting Fundamentals

## What You'll Learn

This step teaches you how to set up Stripe payments through **spec-driven development with Cursor AI**. Instead of copying code, you'll learn to:

- Write effective Cursor prompts for payment system setup
- Use AI to generate secure, production-ready payment code
- Understand prompt patterns for complex integrations
- Validate AI-generated code against specifications

**Core Teaching Objectives**:
- Master Cursor prompting for payment integrations
- Learn to specify requirements clearly for AI code generation
- Understand how to iterate with AI to refine implementations
- Build confidence in AI-assisted development workflows

## Prerequisites

- Completed previous SkyMarket development steps
- Basic understanding of APIs and webhooks
- Cursor IDE installed and configured
- A Stripe account (free to create)

## Cursor Fundamentals for Payment Setup

### Understanding AI-Assisted Development

When setting up complex systems like payments, AI tools like Cursor work best when you:

1. **Specify requirements clearly** - The more specific your prompt, the better the output
2. **Break down complex tasks** - Large features work better as smaller, focused prompts
3. **Iterate and refine** - Use AI feedback loops to improve code quality
4. **Validate against specifications** - Always check AI output against your requirements

### Effective Prompting Patterns

**Pattern 1: Requirements-First Prompting**
```
Instead of: "Create a Stripe integration"
Use: "Create a Stripe PaymentIntent with manual capture that authorizes payment at order creation but only captures funds when service is completed"
```

**Pattern 2: Context-Specific Instructions**
```
Instead of: "Set up environment variables"
Use: "Add these Stripe environment variables to .env.local for a Next.js 15 App Router project: publishable key for client-side, secret key for server-side, webhook secret for signature verification"
```

**Pattern 3: Implementation Constraints**
```
Instead of: "Make it secure"
Use: "Ensure webhook signature verification using stripe.webhooks.constructEvent(), never trust client-provided amounts, and use server-side calculations only"
```

## Step-by-Step Setup with Cursor

### Step 1: Create Your Stripe Account

**Manual Task** (AI can't do this for you):
1. Visit [stripe.com](https://stripe.com) and create an account
2. Complete basic business information
3. Navigate to test mode (toggle in dashboard)
4. Go to Developers > API Keys

**Cursor Learning**: Even with AI assistance, some tasks require manual setup. Understand what AI can and cannot automate.

### Step 2: Environment Configuration with Cursor

**Cursor Prompt Pattern**:
```
Add Stripe configuration to my Next.js project:

1. Update .env.local with these variables:
   - NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY (for client-side)
   - STRIPE_SECRET_KEY (for server-side)
   - STRIPE_WEBHOOK_SECRET (for webhook verification)

2. Update .env.example to show the required variables without actual values

3. Ensure .env.local is in .gitignore

Show me exactly what to add to each file.
```

**Why This Works**: This prompt is specific about files, variable purposes, and security considerations.

**Cursor Follow-up Prompt**:
```
Explain why each environment variable is needed and what security risks exist if they're handled incorrectly.
```

### Step 3: Stripe Client Configuration

**Cursor Prompt for Server-Side Setup**:
```
Create a server-side Stripe configuration file at lib/stripe/server.ts that:

1. Imports Stripe from the stripe package
2. Checks that STRIPE_SECRET_KEY environment variable exists
3. Throws an error if the key is missing
4. Exports a configured stripe instance
5. Uses API version '2024-11-20.acacia'
6. Includes TypeScript types

Also create a helper function to create PaymentIntents with manual capture.
```

**Cursor Prompt for Client-Side Setup**:
```
Create a client-side Stripe configuration at lib/stripe/client.ts that:

1. Imports loadStripe from @stripe/stripe-js
2. Checks for NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY
3. Throws an error if missing
4. Exports a stripePromise for use in React components
5. Include proper TypeScript types

This will be used with Stripe Elements in React components.
```

**Learning Objective**: Notice how each prompt specifies:
- Exact file paths
- Required imports
- Error handling requirements
- TypeScript considerations
- Usage context

### Step 4: Webhook Setup with Iterative Prompting

**Initial Cursor Prompt**:
```
Create a Stripe webhook handler at app/api/webhooks/stripe/route.ts for Next.js App Router that:

1. Accepts POST requests with raw body
2. Verifies webhook signatures using STRIPE_WEBHOOK_SECRET
3. Returns 400 for invalid signatures
4. Returns 200 for successful processing
5. Handles these events: payment_intent.succeeded, payment_intent.payment_failed, charge.refunded

Structure it properly for Next.js 15 App Router with TypeScript.
```

**Follow-up Prompting** (Iterative Refinement):
```
Enhance the webhook handler to:

1. Add proper error logging that doesn't expose sensitive data
2. Use Supabase admin client for database updates
3. Extract order_id from payment intent metadata
4. Update orders table based on event type
5. Never fail webhooks due to database errors (always return 200)

Show me the specific database update logic for each event type.
```

**Advanced Cursor Prompt**:
```
Add webhook security best practices:

1. Validate request size limits
2. Add structured logging for monitoring
3. Include idempotency considerations
4. Add proper TypeScript types for Stripe events
5. Show me how to test this locally with Stripe CLI

Explain why each security measure is important.
```

### Step 5: Testing Setup with AI Guidance

**Cursor Prompt for Testing Strategy**:
```
Create a comprehensive testing approach for Stripe integration:

1. Show me how to install and configure Stripe CLI
2. Create a test endpoint that verifies Stripe configuration
3. Provide test commands for webhook events
4. Include test card numbers for different scenarios
5. Create a validation checklist for each component

Format as a step-by-step guide I can follow.
```

**Cursor Prompt for Debugging**:
```
Create a debugging guide for common Stripe setup issues:

1. Environment variable problems
2. Webhook signature verification failures
3. API key configuration issues
4. Local development connection problems

For each issue, provide:
- How to identify the problem
- Step-by-step fix instructions
- How to prevent it in the future
```

## Advanced Cursor Techniques

### Prompt Chaining for Complex Setup

**Chain 1: Infrastructure**
```
Set up the basic Stripe infrastructure (API clients, environment variables, error handling)
```

**Chain 2: Payment Logic**
```
Now add payment creation logic that uses the infrastructure from step 1
```

**Chain 3: Webhook Processing**
```
Add webhook handling that integrates with the payment logic from step 2
```

**Chain 4: Testing & Validation**
```
Create comprehensive tests for the entire payment system built in previous steps
```

### Specification-Driven Prompting

**Requirement Specification Prompt**:
```
Based on these requirements, generate the Stripe setup:

Requirements:
- Manual capture (authorize now, charge later)
- Order-based payments with metadata tracking
- Webhook synchronization for payment status
- Support for refunds and cancellations
- PCI compliant through Stripe Elements
- Proper error handling and logging

Create a technical specification document first, then generate the implementation.
```

### Error Handling with AI

**Error-First Prompting**:
```
For this Stripe integration, anticipate and handle these error scenarios:

1. Invalid API keys or environment configuration
2. Network failures during API calls
3. Webhook signature verification failures
4. Database connection errors during webhook processing
5. Invalid payment amounts or currency issues

Show me robust error handling for each scenario.
```

## Validating AI-Generated Code

### Quality Checklist for AI Output

When Cursor generates Stripe integration code, verify:

**Security Validation**:
- [ ] Environment variables properly validated
- [ ] Webhook signatures always verified
- [ ] No sensitive data in logs
- [ ] Server-side amount calculations only
- [ ] Proper error messages (no internal exposure)

**Functionality Validation**:
- [ ] PaymentIntents use manual capture
- [ ] Metadata includes necessary order information
- [ ] Webhook events update database correctly
- [ ] Error cases return appropriate status codes
- [ ] TypeScript types are properly defined

**Best Practices Validation**:
- [ ] File structure follows Next.js conventions
- [ ] Import statements are correct and minimal
- [ ] Functions are properly typed
- [ ] Error handling doesn't crash the application
- [ ] Code follows project patterns

### Testing AI-Generated Integration

**Cursor Prompt for Test Creation**:
```
Create a comprehensive test suite for the Stripe integration that validates:

1. Environment variable validation works
2. Payment intent creation with correct parameters
3. Webhook signature verification (valid and invalid)
4. Database updates from webhook events
5. Error handling for each failure scenario

Include both unit tests and integration tests with Stripe CLI.
```

## Common AI Pitfalls and Solutions

### Pitfall 1: Overly Generic Code
**Problem**: AI generates basic examples without project-specific requirements
**Solution**: Include specific project context in prompts

```
Instead of: "Create Stripe integration"
Use: "Create Stripe integration for SkyMarket drone service marketplace using Next.js App Router, Supabase, and manual capture for order-based payments"
```

### Pitfall 2: Missing Error Handling
**Problem**: AI code works in happy path but fails in edge cases
**Solution**: Explicitly request error handling

```
Add to prompts: "Include comprehensive error handling for network failures, invalid data, and database connection issues"
```

### Pitfall 3: Security Vulnerabilities
**Problem**: AI might skip security best practices
**Solution**: Always specify security requirements

```
Add to prompts: "Ensure webhook signature verification, server-side amount calculation, and no sensitive data logging"
```

## Cursor Workflow Best Practices

### 1. Start with Specifications
Always begin with a specification prompt before implementation:
```
"Create a technical specification for Stripe payment integration that includes architecture, security requirements, error handling, and testing strategy"
```

### 2. Build Incrementally
Use small, focused prompts rather than large, complex requests:
```
Step 1: "Create Stripe client configuration"
Step 2: "Add PaymentIntent creation logic"
Step 3: "Implement webhook handling"
```

### 3. Validate and Iterate
After each AI response, validate and refine:
```
"Review this Stripe code for security vulnerabilities and suggest improvements"
```

### 4. Document AI Decisions
Ask AI to explain its choices:
```
"Explain why you chose this approach for webhook error handling and what alternatives exist"
```

## Summary

This step taught you to use Cursor AI effectively for complex integrations like Stripe payments. Key takeaways:

**Effective AI Prompting**:
- Be specific about requirements and constraints
- Include security and error handling explicitly
- Break complex tasks into focused prompts
- Iterate and refine based on output quality

**Validation Skills**:
- Always verify AI-generated code against specifications
- Test security, functionality, and best practices
- Use checklists to ensure comprehensive validation

**AI-Assisted Development Workflow**:
- Specification → Implementation → Validation → Iteration
- Use AI for code generation, explanation, and testing
- Maintain human oversight for security and architecture decisions

With these skills, you can confidently use AI to build production-ready payment systems while understanding exactly what's being created and why.

## Next Steps

In **Step 18: Payment Flow Implementation**, you'll master:
- Advanced Cursor prompting for full-stack payment features
- Multi-component coordination with AI assistance
- Security-first prompting for payment authorization and capture
- Production-ready testing and validation strategies

This advanced step builds on the Cursor fundamentals you've learned to implement complete payment workflows through AI-assisted development.

---

**Previous**: [Step 16: Realtime Communication](./16-realtime-communication.md) ← | → **Next**: [Step 18: Payment Flow Implementation](./18-payment-flow.md)