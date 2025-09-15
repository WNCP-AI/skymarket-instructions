# Payment Specification

## Overview

This specification defines the payment system for SkyMarket using standard Stripe PaymentIntents with manual capture. The system implements a hold-and-release pattern suitable for service marketplaces.

**Core Requirements**:
- Authorization of payment at order placement
- Hold funds without immediate capture
- Manual capture upon service completion
- Refund capability for cancellations

## Specification Structure

This document follows a spec-driven development approach. Each section provides:
1. **Requirements** - What needs to be built
2. **Cursor Prompts** - How to instruct Cursor to generate the code
3. **Validation** - How to verify the implementation

## 1. Payment Intent Creation

### Requirements

When a consumer creates an order, the system must:
- Create an order record in the database
- Initialize a Stripe PaymentIntent with manual capture
- Return a client secret for frontend payment collection

### Cursor Prompt Pattern

```
Create an API endpoint at app/api/orders/route.ts that:
1. Accepts POST requests with listingId, quantity, and deliveryAddress
2. Validates the authenticated user using Supabase auth
3. Fetches the listing details and calculates total amount
4. Creates an order in the orders table with status 'pending'
5. Creates a Stripe PaymentIntent with:
   - amount in cents
   - capture_method: 'manual'
   - metadata containing order_id, consumer_id, listing_id
6. Updates the order with stripe_payment_intent_id
7. Returns orderId, clientSecret, and amount

Use these imports:
- NextResponse from 'next/server'
- createClient from '@/lib/supabase/server'
- stripe from '@/lib/stripe/server'

Include proper error handling for each step.
```

### Expected Structure

```typescript
// The endpoint should follow this pattern:
// 1. Input validation
// 2. Authentication check
// 3. Resource fetching
// 4. Database transaction
// 5. Payment intent creation
// 6. Response formatting
```

### Validation Checklist

- [ ] Endpoint accepts POST requests
- [ ] Authentication is verified
- [ ] Amount is calculated server-side
- [ ] PaymentIntent uses manual capture
- [ ] Metadata includes necessary IDs
- [ ] Error responses have appropriate status codes

## 2. Client-Side Payment Collection

### Requirements

The checkout interface must:
- Display the order amount clearly
- Use Stripe Elements for secure card collection
- Show that payment will be held, not charged immediately
- Handle errors gracefully

### Cursor Prompt Pattern

```
Create a React component at components/checkout/CheckoutForm.tsx that:
1. Uses Stripe Elements (PaymentElement) for card collection or simply the stripe checkout process
2. Accepts orderId and amount as props
3. Shows the total amount prominently
4. Includes a submit button that says "Pay $[amount]"
5. Displays a note: "Your card will be authorized but not charged until service is complete"
6. Handles form submission with stripe.confirmPayment
7. Shows loading state during processing
8. Displays errors using Alert component

Use these Stripe hooks:
- useStripe
- useElements
- PaymentElement component

Style with Tailwind classes.
```

### Component Architecture

```
CheckoutPage (Elements Provider)
  └── CheckoutForm
      ├── Amount Display
      ├── PaymentElement
      ├── Error Display
      └── Submit Button
```

### Validation Checklist

- [ ] PaymentElement renders correctly
- [ ] Amount is clearly displayed
- [ ] Loading states work properly
- [ ] Errors display to user
- [ ] Hold vs charge messaging is clear

## 3. Webhook Event Processing

### Requirements

The webhook handler must:
- Verify Stripe signatures for security
- Process payment_intent.succeeded events
- Process payment_intent.payment_failed events
- Process charge.refunded events
- Update order status accordingly

### Cursor Prompt Pattern

```
Create a webhook handler at app/api/webhooks/stripe/route.ts that:
1. Receives POST requests with raw body text
2. Verifies the stripe-signature header using stripe.webhooks.constructEvent
3. Returns 400 if signature verification fails
4. Processes these event types:
   - payment_intent.succeeded: Update order payment_status to 'authorized'
   - payment_intent.payment_failed: Update order payment_status to 'failed'
   - charge.refunded: Update order payment_status to 'refunded'
5. Uses the order_id from payment intent metadata
6. Returns { received: true } on success
7. Logs but doesn't fail on database errors (return 200)

Use Supabase admin client for database updates.
Include proper error logging.
```

### Event Flow

```
Stripe Event → Signature Verification → Event Router → Database Update
                    ↓ (fail)
                 Return 400
```

### Validation Checklist

- [ ] Signature verification works
- [ ] Each event type updates database correctly
- [ ] Invalid signatures return 400
- [ ] Database errors don't cause webhook failures
- [ ] Events are logged for debugging

## 4. Payment Capture

### Requirements

When a service is completed, the provider must be able to:
- Capture the previously authorized payment
- Update the order status to completed
- Only allow the assigned provider to capture

### Cursor Prompt Pattern

```
Create an API endpoint at app/api/orders/[orderId]/capture/route.ts that:
1. Accepts POST requests with orderId as a route parameter
2. Verifies the authenticated user is the provider for this order
3. Checks that payment_status is 'authorized'
4. Calls stripe.paymentIntents.capture with the stripe_payment_intent_id
5. Updates the order:
   - payment_status to 'paid'
   - status to 'completed'
   - completed_at timestamp
6. Returns success with payment intent ID and captured amount

Include authorization checks and error handling.
Use dynamic route parameters in Next.js App Router.
```

### Authorization Flow

```
User → Provider Check → Payment Status Check → Capture → Update Order
         ↓ (fail)           ↓ (fail)
      Return 403         Return 400
```

### Validation Checklist

- [ ] Only provider can capture their orders
- [ ] Can only capture authorized payments
- [ ] Order status updates correctly
- [ ] Stripe capture succeeds
- [ ] Proper error messages returned

## 5. Refund Processing

### Requirements

The system must support:
- Full refunds for cancelled orders
- Partial refunds when specified
- Proper status updates after refund

### Cursor Prompt Pattern

```
Create an API endpoint at app/api/orders/[orderId]/refund/route.ts that:
1. Accepts POST requests with optional amount in body
2. Fetches the order with stripe_payment_intent_id
3. Creates a Stripe refund:
   - Full refund if no amount specified
   - Partial refund if amount provided
4. Updates order:
   - payment_status to 'refunded' or 'partially_refunded'
   - status to 'cancelled'
   - refunded_amount field
5. Returns refund ID, amount, and status

Handle cases where payment intent doesn't exist.
Use proper error responses.
```

### Refund Logic

```
if (amount) {
  // Partial refund
  payment_status = 'partially_refunded'
} else {
  // Full refund
  payment_status = 'refunded'
}
```

### Validation Checklist

- [ ] Full refunds work correctly
- [ ] Partial refunds work with amount
- [ ] Order status updates appropriately
- [ ] Refund amount is tracked
- [ ] Error cases handled properly

## 6. Database Schema

### Requirements

The orders table must track:
- Order details and relationships
- Payment status separately from order status
- Stripe payment intent ID for reference

### Cursor Prompt Pattern

```
Create a SQL migration for the orders table with:
- id (UUID primary key)
- consumer_id (references auth.users)
- listing_id (references listings)
- provider_id (UUID)
- quantity (integer)
- total_amount (decimal)
- delivery_address (text)
- status (enum: pending, confirmed, in_progress, completed, cancelled)
- payment_status (enum: pending, authorized, paid, failed, refunded, partially_refunded)
- stripe_payment_intent_id (text, unique)
- refunded_amount (decimal, nullable)
- completed_at (timestamp, nullable)
- created_at and updated_at timestamps

Add indexes for:
- stripe_payment_intent_id
- consumer_id
- provider_id
- status and payment_status combination
```

### State Transitions

```
Order Status:       pending → confirmed → in_progress → completed
                        ↓         ↓           ↓
                    cancelled  cancelled  cancelled

Payment Status:     pending → authorized → paid
                        ↓         ↓
                     failed   refunded
```

### Validation Checklist

- [ ] All required fields present
- [ ] Proper foreign key constraints
- [ ] Indexes for query performance
- [ ] Enum constraints work correctly
- [ ] Timestamps auto-update

## Testing Strategy

### Unit Testing Prompts

```
Create unit tests that verify:
1. Payment intent creation with correct parameters
2. Webhook signature verification
3. Payment capture authorization logic
4. Refund amount calculations
5. Order status transitions
```

### Integration Testing with Stripe CLI

```bash
# Test webhook locally
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Trigger test events
stripe trigger payment_intent.succeeded
stripe trigger payment_intent.payment_failed
stripe trigger charge.refunded
```

### End-to-End Testing Checklist

1. **Happy Path**:
   - [ ] Create order → Payment authorized → Service complete → Payment captured

2. **Cancellation Path**:
   - [ ] Create order → Payment authorized → Order cancelled → Payment refunded

3. **Failure Path**:
   - [ ] Create order → Payment fails → Order cancelled

## Security Requirements

### Cursor Prompts for Security

```
Ensure all payment endpoints:
1. Verify webhook signatures before processing
2. Calculate amounts server-side only
3. Check user authorization before captures/refunds
4. Use idempotency keys for payment operations
5. Never log sensitive payment data
6. Return appropriate error codes without exposing internals
```

### Security Checklist

- [ ] No client-side amount calculations
- [ ] Webhook signatures always verified
- [ ] Authorization checks on all mutations
- [ ] Idempotency prevents duplicate charges
- [ ] PCI compliance through Stripe Elements
- [ ] Rate limiting on payment endpoints

## Implementation Order

1. **Phase 1: Core Payment Flow**
   - Orders API endpoint
   - Payment intent creation
   - Basic webhook handler

2. **Phase 2: Payment Collection**
   - Checkout form component
   - Stripe Elements integration
   - Confirmation page

3. **Phase 3: Payment Operations**
   - Payment capture endpoint
   - Refund processing
   - Order status management

4. **Phase 4: Testing & Security**
   - Webhook testing with CLI
   - Security hardening
   - Error handling improvements

## Cursor Workflow Tips

### Effective Prompting Patterns

1. **Be Specific About Imports**:
   ```
   Use these exact imports:
   - stripe from '@/lib/stripe/server'
   - createClient from '@/lib/supabase/server'
   ```

2. **Define Error Handling**:
   ```
   Return these error responses:
   - 400 for invalid input
   - 401 for unauthenticated
   - 403 for unauthorized
   - 404 for not found
   - 500 for server errors
   ```

3. **Specify Validation Rules**:
   ```
   Validate that:
   - amount is positive
   - user owns the resource
   - status allows the operation
   ```

### Common Pitfalls to Avoid

1. **Don't Trust Client Data**: Always recalculate amounts server-side
2. **Don't Skip Webhook Signatures**: Security depends on verification
3. **Don't Capture Immediately**: Use manual capture for marketplaces
4. **Don't Expose Internal Errors**: Log errors, return generic messages

## Summary

This specification provides a complete payment system using standard Stripe with manual capture. The spec-driven approach ensures:

- Clear requirements before implementation
- Consistent patterns across the codebase
- Proper security and error handling
- Testable and maintainable code

Follow the cursor prompts to generate each component, then validate against the checklists to ensure correctness.