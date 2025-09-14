# Step 18: Payment Flow Implementation - Advanced Cursor Prompting Patterns

## What You'll Master

This step teaches **advanced Cursor prompting techniques** for building complete payment systems. You'll learn to:

- Use complex prompting patterns for full-stack features
- Coordinate frontend and backend development with AI
- Implement secure payment flows through specification-driven prompts
- Master iterative development with AI feedback loops

**Core Teaching Objectives**:
- Advanced prompt engineering for complex features
- Multi-component coordination with AI assistance
- Security-first prompting for payment systems
- Validation and testing strategies with AI

## Prerequisites

- Completed [Step 17: Stripe Setup](./17-stripe-setup.md)
- Understanding of Cursor prompting fundamentals
- Basic React and API development knowledge
- Stripe configuration completed

## Advanced Prompting Architecture

### Understanding Complex Feature Development

Payment flows involve multiple interconnected components:
- Backend API endpoints for order creation
- Frontend payment collection forms
- Webhook processing for status updates
- State management across the application

**Key Insight**: Use **component-driven prompting** - break complex features into focused, interconnected prompts that build on each other.

### Master Prompting Patterns

**Pattern 1: Dependency-Aware Prompting**
```
Create component A that will be used by component B, ensuring the interface is compatible with requirement C.
```

**Pattern 2: Security-First Prompting**
```
Implement feature X with these security constraints: Y, Z. Validate that the implementation doesn't introduce vulnerabilities A, B.
```

**Pattern 3: Integration-Focused Prompting**
```
Build component X that integrates with existing components Y and Z, following the established patterns in the codebase.
```

## Payment Flow Development with Cursor

### Phase 1: Orders API with Specification-Driven Prompting

**Master Cursor Prompt**:
```
Create an orders API endpoint at app/api/orders/route.ts that implements this specification:

BUSINESS LOGIC:
- Consumer places order for a specific listing
- Payment is authorized but not captured (hold-and-release pattern)
- Order is created in database with pending status
- PaymentIntent metadata tracks order for webhook processing

TECHNICAL REQUIREMENTS:
- Next.js 15 App Router POST handler
- Supabase authentication and database operations
- Stripe PaymentIntent with capture_method: 'manual'
- Server-side amount calculation (never trust client)
- Comprehensive error handling with appropriate HTTP status codes

SECURITY CONSTRAINTS:
- Validate authenticated user before order creation
- Verify listing exists and is active
- Calculate total amount server-side using database values
- Include order/consumer/listing IDs in PaymentIntent metadata
- Return only client secret and order ID (no sensitive data)

INPUT VALIDATION:
- listingId (required, valid UUID)
- quantity (required, positive integer)
- deliveryAddress (required, non-empty string)

RESPONSE FORMAT:
Success: { orderId, clientSecret, amount }
Error: { error: string } with appropriate status code

Show me the complete implementation with TypeScript types.
```

**Why This Prompt Works**:
- Separates concerns (business logic, technical, security)
- Specifies exact file path and framework version
- Includes input validation and output format
- Explicitly mentions security constraints
- Requests TypeScript types for better code quality

**Follow-up Refinement Prompt**:
```
Review the orders API for these potential issues:
1. Race conditions in order creation
2. Duplicate order prevention
3. Listing availability checking
4. Database transaction handling
5. Error message security (no internal details exposed)

Suggest improvements for each area.
```

### Phase 2: Frontend Payment Collection with Component Architecture

**Cursor Prompt for Payment Form**:
```
Create a payment collection component with this architecture:

COMPONENT STRUCTURE:
CheckoutForm (components/checkout/CheckoutForm.tsx)
├── Props: { orderId: string, amount: number }
├── State: error, processing, success
├── Stripe Integration: useStripe, useElements, PaymentElement
└── Submission: confirmPayment with return URL

BUSINESS REQUIREMENTS:
- Display order total prominently
- Show "authorize vs charge" messaging clearly
- Handle payment confirmation with proper loading states
- Redirect to confirmation page on success
- Display user-friendly error messages

TECHNICAL IMPLEMENTATION:
- Use Stripe's PaymentElement for PCI compliance
- Form validation before submission
- Loading state during payment processing
- Error boundaries for Stripe failures
- TypeScript props and state interfaces

UX REQUIREMENTS:
- Clear visual hierarchy (amount → form → submit)
- Disabled state management (can't submit while processing)
- Progress indicators during payment
- Accessible form labels and error messages
- Mobile-responsive design

INTEGRATION POINTS:
- Accepts orderId and amount from parent
- Uses stripe.confirmPayment with configured return URL
- Handles both success and error states
- Integrates with existing UI component library

Show me the complete component with proper TypeScript types and error handling.
```

**Cursor Prompt for Checkout Page**:
```
Create a checkout page that wraps the payment form with Stripe Elements provider:

PAGE ARCHITECTURE:
app/checkout/[orderId]/page.tsx
├── Dynamic route parameter handling
├── Order data fetching and validation
├── Stripe Elements provider configuration
├── Loading and error state management
└── CheckoutForm integration

IMPLEMENTATION REQUIREMENTS:
- Fetch order details and client secret from API
- Handle loading state while fetching order data
- Configure Stripe Elements with appearance customization
- Error handling for invalid order IDs
- Proper TypeScript types for page props

ERROR SCENARIOS TO HANDLE:
- Order not found
- Order already paid
- Network failures during data fetching
- Invalid order states
- Missing client secret

Show me the complete implementation with proper error boundaries.
```

### Phase 3: Payment Capture with Authorization Patterns

**Cursor Prompt for Capture Endpoint**:
```
Create a payment capture endpoint that implements provider authorization:

AUTHORIZATION LOGIC:
- Only the assigned provider can capture payment for their orders
- Verify order exists and belongs to authenticated user
- Check payment status is 'authorized' before capture
- Prevent duplicate capture attempts

ENDPOINT SPECIFICATION:
File: app/api/orders/[orderId]/capture/route.ts
Method: POST
Route: /api/orders/[order-id]/capture

BUSINESS FLOW:
1. Authenticate user (Supabase auth)
2. Fetch order with provider relationship
3. Verify user is the assigned provider
4. Check payment status allows capture
5. Call Stripe capture API
6. Update order status to completed
7. Return capture confirmation

ERROR HANDLING:
- 401: User not authenticated
- 403: User not authorized for this order
- 404: Order not found
- 400: Payment not in capturable state
- 500: Stripe API or database errors

SECURITY CONSIDERATIONS:
- Never allow capturing others' orders
- Validate order status transitions
- Log capture attempts for audit
- Don't expose internal error details

Show me the implementation with comprehensive error handling and authorization checks.
```

### Phase 4: Refund Processing with Partial Support

**Advanced Cursor Prompt for Refunds**:
```
Implement a flexible refund system that supports both full and partial refunds:

REFUND BUSINESS LOGIC:
- Full refund: Cancel entire order, refund complete amount
- Partial refund: Refund specified amount, keep order active
- Status tracking: Track total refunded amount over time
- Audit trail: Log all refund activities

ENDPOINT SPECIFICATION:
File: app/api/orders/[orderId]/refund/route.ts
Input: { amount?: number, reason?: string }
Output: { refundId, amount, status, remainingAmount }

REFUND CALCULATION:
- If no amount specified: refund full payment
- If amount specified: validate <= (total - previously refunded)
- Update order status based on refund completeness
- Track cumulative refunded amount

COMPLEX SCENARIOS:
- Multiple partial refunds
- Refund amount validation
- Currency precision handling
- Failed refund recovery

AUTHORIZATION:
- Admin users can refund any order
- Providers can refund their completed orders
- Consumers can request refunds (different endpoint)

Show me the implementation that handles all these scenarios with proper validation.
```

## Advanced Integration Patterns

### Pattern: Multi-Component Coordination

**Cursor Prompt for Status Management**:
```
Create a cohesive order status management system that coordinates between:

COMPONENTS TO COORDINATE:
- Orders API (creation and updates)
- Webhook handler (status synchronization)
- Payment capture (provider actions)
- Refund processing (cancellation handling)

STATUS FLOW SPECIFICATION:
Order Status: pending → confirmed → in_progress → completed → [cancelled]
Payment Status: pending → authorized → paid → [refunded/failed]

COORDINATION REQUIREMENTS:
- Webhook updates must never conflict with API updates
- Status transitions must be atomic and validated
- All components use consistent status values
- Race condition prevention

IMPLEMENTATION STRATEGY:
- Create shared status validation functions
- Implement optimistic locking for status updates
- Add status transition logging
- Create status query helpers

Show me the implementation that ensures all components stay synchronized.
```

### Pattern: Error Boundary Integration

**Cursor Prompt for Comprehensive Error Handling**:
```
Create an error handling system that works across the entire payment flow:

ERROR CATEGORIES:
1. User Input Errors (validation failures)
2. Business Logic Errors (invalid state transitions)
3. External Service Errors (Stripe API failures)
4. System Errors (database connectivity)
5. Authorization Errors (permission denied)

ERROR HANDLING STRATEGY:
- Client-side: User-friendly messages, retry options
- Server-side: Detailed logging, proper HTTP status codes
- Webhook: Always return 200, log failures
- Database: Transaction rollback, consistency checks

ERROR RECOVERY:
- Payment failures: Allow retry with different payment method
- Capture failures: Retry mechanism with exponential backoff
- Webhook failures: Idempotent processing
- Database failures: Graceful degradation

Show me error handling utilities that can be used across all payment components.
```

## Testing Strategy with AI

### Cursor Prompt for Comprehensive Testing

**Testing Architecture Prompt**:
```
Create a comprehensive testing strategy for the payment flow:

TESTING LAYERS:
1. Unit Tests: Individual function logic
2. Integration Tests: API endpoint testing
3. E2E Tests: Complete payment flow
4. Security Tests: Authorization and validation
5. Error Tests: Failure scenario handling

TEST SCENARIOS:
- Happy path: Order → Payment → Capture → Complete
- Cancellation: Order → Payment → Refund → Cancel
- Failure: Order → Payment Failure → Retry
- Security: Unauthorized access attempts
- Edge cases: Duplicate orders, race conditions

TESTING TOOLS:
- Jest for unit tests
- Supertest for API testing
- Stripe CLI for webhook testing
- Test database for integration tests

MOCK STRATEGY:
- Mock Stripe API calls in unit tests
- Use Stripe test mode for integration tests
- Mock database operations for isolated tests

Show me test setup and key test cases for each component.
```

### Validation Prompts

**Security Validation Prompt**:
```
Audit the payment system implementation for these security concerns:

SECURITY CHECKLIST:
- [ ] All amounts calculated server-side
- [ ] Webhook signatures verified
- [ ] Authorization checked on all mutations
- [ ] No sensitive data in logs or responses
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention
- [ ] Rate limiting on payment endpoints
- [ ] Idempotency key usage

For each item, show me how to verify it's properly implemented and suggest improvements if needed.
```

## Advanced Cursor Techniques

### Technique 1: Context-Aware Prompting

**Building on Previous Components**:
```
Using the orders API created earlier, now implement the webhook handler that integrates with the same database schema and status management patterns.
```

### Technique 2: Specification Evolution

**Iterative Requirements**:
```
Enhance the payment capture endpoint to support these new requirements:
- Partial captures (capture less than authorized amount)
- Capture scheduling (capture at specific time)
- Multi-provider orders (split captures)

Maintain backward compatibility with existing implementation.
```

### Technique 3: Cross-Component Validation

**System Integration Check**:
```
Review all payment system components (orders API, webhook handler, capture endpoint, refund processor) for:
1. Data consistency across components
2. Status synchronization issues
3. Race condition possibilities
4. Error handling completeness
5. Security gap analysis

Provide a system-wide improvement plan.
```

## Production Readiness with AI

### Cursor Prompt for Production Checklist

**Production Validation Prompt**:
```
Create a production readiness checklist for the payment system:

PERFORMANCE REQUIREMENTS:
- API response times under 500ms
- Database query optimization
- Stripe API timeout handling
- Memory usage optimization

MONITORING REQUIREMENTS:
- Payment success/failure rates
- Webhook processing metrics
- Error rate alerting
- Performance monitoring

SECURITY REQUIREMENTS:
- Environment variable validation
- API rate limiting
- Request size limits
- Security header configuration

RELIABILITY REQUIREMENTS:
- Database connection pooling
- Graceful error degradation
- Automatic retry mechanisms
- Circuit breaker patterns

For each requirement, show me how to implement and validate it.
```

## Common Advanced Pitfalls

### Pitfall 1: Over-Engineering
**Problem**: Asking AI to build overly complex solutions
**Solution**: Start simple, then iterate with enhancement prompts

```
Instead of: "Build a complete payment system with all features"
Use: "Build basic payment creation, then I'll ask for enhancements"
```

### Pitfall 2: Missing Integration Points
**Problem**: Components that don't work together
**Solution**: Explicitly mention integration requirements in prompts

```
Add to prompts: "Ensure this component integrates properly with the existing orders API and webhook handler"
```

### Pitfall 3: Security Afterthoughts
**Problem**: Adding security as an afterthought
**Solution**: Lead with security in every prompt

```
Start prompts with: "Implement [feature] with these security requirements first: [list requirements]"
```

## Summary

This step taught you advanced Cursor prompting for complex full-stack features. Key mastery areas:

**Advanced Prompting Patterns**:
- Specification-driven development with AI
- Multi-component coordination prompts
- Security-first prompt engineering
- Iterative enhancement through AI feedback

**Production-Ready Development**:
- Comprehensive error handling strategies
- Testing automation with AI assistance
- Security validation and audit practices
- Performance optimization techniques

**AI-Assisted Architecture**:
- Breaking complex features into manageable prompts
- Ensuring component integration through prompt design
- Using AI for system validation and improvement
- Building production-ready systems with AI assistance

With these advanced techniques, you can use AI to build sophisticated, secure, and maintainable payment systems while maintaining full understanding and control over the architecture.

## Next Steps

With your payment system mastered through AI-assisted development, you're ready to explore:
- **Advanced UI Components**: Use Cursor to build complex user interfaces
- **Performance Optimization**: AI-assisted performance profiling and optimization
- **Production Deployment**: Cursor prompts for CI/CD and infrastructure
- **Additional Integrations**: Email notifications, SMS updates, and analytics

You've now mastered both fundamental and advanced AI-assisted development patterns that you can apply to any complex software project.

---

**Previous**: [Step 17: Stripe Setup](./17-stripe-setup.md) ← | → **Next**: [Advanced Topics](../advanced/)