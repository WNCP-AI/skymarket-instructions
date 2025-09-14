# Webhook Specification

## Overview

This specification defines webhook handling for SkyMarket's payment system. Webhooks provide real-time notifications from Stripe about payment events, ensuring the application stays synchronized with payment status changes.

**Core Requirements**:
- Secure signature verification of all webhook events
- Processing standard Stripe payment events (not Connect events)
- Reliable database updates without causing webhook failures
- Idempotent event processing to prevent duplicate handling

## Specification Structure

This document follows a spec-driven development approach providing:
1. **Requirements** - What the webhook system must accomplish
2. **Cursor Prompts** - Specific instructions for generating webhook handlers
3. **Event Specifications** - Detailed requirements for each event type
4. **Validation Checklists** - How to verify correct implementation

## 1. Webhook Handler Foundation

### Requirements

The webhook endpoint must:
- Accept POST requests with raw body content
- Verify Stripe webhook signatures for security
- Parse webhook events into typed objects
- Route events to appropriate handlers
- Return appropriate HTTP status codes

### Cursor Prompt Pattern

```
Create a webhook handler at app/api/webhooks/stripe/route.ts that:
1. Imports headers from 'next/headers' and Stripe from 'stripe'
2. Accepts POST requests and extracts raw body text
3. Gets the stripe-signature header
4. Uses stripe.webhooks.constructEvent() to verify signature
5. Returns 400 with "Invalid signature" if verification fails
6. Has a try-catch block that processes events
7. Returns { received: true } on success
8. Logs errors but returns 200 to prevent Stripe retries

Structure the handler with:
- Signature verification section
- Event type switch statement
- Database update functions
- Error handling that never fails webhooks

Environment variable required: STRIPE_WEBHOOK_SECRET
```

### Handler Structure

```typescript
POST /api/webhooks/stripe
├── Body extraction (raw text)
├── Signature verification
├── Event parsing
├── Event routing (switch statement)
├── Database operations
└── Response (always 200 for processed events)
```

### Validation Checklist

- [ ] Raw body is extracted correctly
- [ ] Stripe signature header is retrieved
- [ ] Signature verification uses webhook secret
- [ ] Invalid signatures return 400 status
- [ ] Valid events return 200 status
- [ ] Database errors don't fail webhooks

## 2. Event Processing Architecture

### Requirements

Each event type needs:
- Specific handler function
- Metadata extraction for order identification
- Database update operations
- Error handling and logging
- Idempotency considerations

### Cursor Prompt Pattern

```
Create event handler functions that:
1. Extract relevant data from the Stripe event object
2. Get order_id from payment intent metadata
3. Skip processing if no order_id is found
4. Use Supabase admin client for database updates
5. Log success and failure cases
6. Handle database errors gracefully

Create these handler functions:
- handlePaymentIntentSucceeded(paymentIntent, supabase)
- handlePaymentIntentFailed(paymentIntent, supabase)
- handleChargeRefunded(charge, supabase)

Each function should:
- Accept the Stripe object and Supabase client
- Extract order identification from metadata
- Update the orders table appropriately
- Log the operation result
```

### Event Processing Flow

```
Webhook Event → Extract Metadata → Find Order → Update Database → Log Result
                     ↓ (no metadata)
                 Log warning, continue
```

### Validation Checklist

- [ ] Each event type has dedicated handler
- [ ] Metadata extraction handles missing data
- [ ] Database updates use admin client
- [ ] Operations are logged for debugging
- [ ] Handlers don't throw unhandled exceptions

## 3. Payment Intent Succeeded Event

### Requirements

When `payment_intent.succeeded` is received:
- Extract order_id from payment intent metadata
- Update order payment_status to 'authorized'
- Store the stripe_payment_intent_id
- Log the successful authorization

### Cursor Prompt Pattern

```
Create a function handlePaymentIntentSucceeded that:
1. Accepts a Stripe PaymentIntent object and Supabase client
2. Gets order_id from paymentIntent.metadata?.order_id
3. Returns early if no order_id (log warning)
4. Updates the orders table:
   - Set payment_status = 'authorized'
   - Set stripe_payment_intent_id = paymentIntent.id
   - Update updated_at timestamp
5. Uses .eq('id', orderId) to target the specific order
6. Logs success: "Payment authorized for order {orderId}"
7. Logs database errors without throwing

Handle the case where order might not exist.
Use proper error logging.
```

### Event Payload Characteristics

```json
{
  "type": "payment_intent.succeeded",
  "data": {
    "object": {
      "id": "pi_...",
      "status": "requires_capture",
      "capture_method": "manual",
      "metadata": {
        "order_id": "order-uuid-here"
      }
    }
  }
}
```

### Validation Checklist

- [ ] Extracts order_id from metadata
- [ ] Updates payment_status to 'authorized'
- [ ] Stores payment intent ID
- [ ] Handles missing order gracefully
- [ ] Logs operation result

## 4. Payment Intent Failed Event

### Requirements

When `payment_intent.payment_failed` is received:
- Extract order_id from payment intent metadata
- Update order payment_status to 'failed'
- Update order status to 'cancelled'
- Log the payment failure with reason if available

### Cursor Prompt Pattern

```
Create a function handlePaymentIntentFailed that:
1. Accepts a Stripe PaymentIntent object and Supabase client
2. Gets order_id from paymentIntent.metadata?.order_id
3. Returns early if no order_id (log warning)
4. Updates the orders table:
   - Set payment_status = 'failed'
   - Set status = 'cancelled'
   - Update updated_at timestamp
5. Extracts last_payment_error from paymentIntent for logging
6. Logs: "Payment failed for order {orderId}: {errorMessage}"
7. Handles database update errors gracefully

Include error reason in logs for debugging.
```

### Common Failure Scenarios

- Card declined by issuer
- Insufficient funds
- Expired card
- Authentication failures
- Network timeouts

### Validation Checklist

- [ ] Updates payment_status to 'failed'
- [ ] Updates order status to 'cancelled'
- [ ] Extracts and logs error reason
- [ ] Handles missing orders properly
- [ ] Database errors don't cause exceptions

## 5. Charge Refunded Event

### Requirements

When `charge.refunded` is received:
- Use payment_intent ID to find the order
- Determine if refund is full or partial
- Update payment_status appropriately
- Track refunded amount
- Update order status if fully refunded

### Cursor Prompt Pattern

```
Create a function handleChargeRefunded that:
1. Accepts a Stripe Charge object and Supabase client
2. Gets charge.payment_intent to find associated order
3. Returns early if no payment_intent
4. Queries orders table by stripe_payment_intent_id
5. Calculates if refund is full (amount_refunded === amount)
6. Updates the orders table:
   - Set payment_status = 'refunded' (full) or 'partially_refunded' (partial)
   - Set status = 'cancelled' if full refund
   - Set refunded_amount = charge.amount_refunded / 100 (convert from cents)
7. Logs: "Refund processed for order {orderId}: ${amount}"

Handle cases where order lookup fails.
Convert amounts from cents to dollars.
```

### Refund Logic Flow

```
Charge Refunded → Find Order by Payment Intent → Check Refund Amount → Update Status
                           ↓ (not found)
                      Log error, continue
```

### Validation Checklist

- [ ] Finds order using payment_intent ID
- [ ] Distinguishes full vs partial refunds
- [ ] Updates payment_status correctly
- [ ] Tracks refunded amount in dollars
- [ ] Updates order status for full refunds

## 6. Database Integration

### Requirements

Webhook handlers must:
- Use Supabase admin client for elevated permissions
- Handle database connection errors
- Avoid causing webhook failures due to DB issues
- Support concurrent webhook processing

### Cursor Prompt Pattern

```
Set up database integration for webhooks:
1. Import createClient from '@/lib/supabase/admin'
2. Initialize supabase client in the main handler
3. Pass client to event handler functions
4. Wrap database operations in try-catch blocks
5. Log database errors with context
6. Return success even if database update fails

Create error handling that:
- Logs the specific database error
- Includes relevant order/payment IDs
- Uses console.error for webhook debugging
- Never throws errors that would fail the webhook

Example error log: "Database update failed for order {orderId}: {error}"
```

### Database Operations Pattern

```typescript
// Pattern for database updates
try {
  const { error } = await supabase
    .from('orders')
    .update(updateFields)
    .eq('id', orderId);

  if (error) {
    console.error(`DB update failed for order ${orderId}:`, error);
    return; // Don't throw
  }
} catch (err) {
  console.error(`DB operation error for order ${orderId}:`, err);
  return; // Don't throw
}
```

### Validation Checklist

- [ ] Uses admin Supabase client
- [ ] Database errors are caught and logged
- [ ] Webhook never fails due to DB issues
- [ ] Error logs include relevant context
- [ ] Operations handle missing records gracefully

## 7. Security Implementation

### Requirements

Security measures must include:
- Webhook signature verification on every request
- Proper secret management
- Request validation
- Rate limiting considerations
- Secure error handling

### Cursor Prompt Pattern

```
Implement webhook security:
1. Always verify signatures before processing
2. Use stripe.webhooks.constructEvent() with webhook secret
3. Return 400 immediately for invalid signatures
4. Never log sensitive payment data
5. Implement request size limits
6. Add basic rate limiting if needed

Security checklist:
- STRIPE_WEBHOOK_SECRET is properly configured
- Signatures are verified before any processing
- Error messages don't expose internal details
- No sensitive data in logs (card numbers, secrets)
- Raw body is used for signature verification
```

### Security Validation

```typescript
// Never do this - security violation
const event = JSON.parse(body); // UNSAFE!

// Always do this - proper verification
const event = stripe.webhooks.constructEvent(
  body,
  signature,
  webhookSecret
); // SECURE
```

### Validation Checklist

- [ ] All requests verify signatures first
- [ ] Webhook secret is from environment
- [ ] Invalid signatures return 400
- [ ] No sensitive data in logs
- [ ] Raw body used for verification

## 8. Testing Strategy

### Requirements

Testing must cover:
- Signature verification (valid/invalid)
- Each event type processing
- Database update scenarios
- Error handling paths
- Idempotency behavior

### Cursor Prompt Pattern for Testing

```
Create webhook tests that verify:
1. Invalid signatures are rejected with 400
2. Valid signatures are accepted
3. Each event type updates database correctly
4. Missing metadata is handled gracefully
5. Database errors don't fail webhooks

Use Stripe CLI for integration testing:
- stripe listen --forward-to localhost:3000/api/webhooks/stripe
- stripe trigger payment_intent.succeeded
- stripe trigger payment_intent.payment_failed
- stripe trigger charge.refunded

Test webhook with custom metadata:
stripe trigger payment_intent.succeeded \
  --override payment_intent:metadata.order_id=test-order-123
```

### Testing Checklist

- [ ] Signature verification tests pass
- [ ] Each event type has test coverage
- [ ] Error scenarios are tested
- [ ] Database failures don't break webhooks
- [ ] Stripe CLI integration works locally

## 9. Deployment Configuration

### Requirements

Production webhook setup needs:
- Proper endpoint URL configuration
- Environment variable management
- Event subscription configuration
- Monitoring and alerting setup

### Cursor Prompt Pattern for Deployment

```
Configure webhooks for production:
1. In Stripe Dashboard > Webhooks, add endpoint URL
2. Subscribe to these events only:
   - payment_intent.succeeded
   - payment_intent.payment_failed
   - charge.refunded
3. Copy the webhook signing secret
4. Set STRIPE_WEBHOOK_SECRET in production environment
5. Test with a small amount first

Production checklist:
- HTTPS endpoint URL configured
- Only necessary events subscribed
- Webhook secret properly configured
- Test webhook receives events
- Monitor webhook delivery in Stripe Dashboard
```

### Event Subscription List

```
Required Events:
✓ payment_intent.succeeded
✓ payment_intent.payment_failed
✓ charge.refunded

NOT Required (these are Connect-only):
✗ account.updated
✗ transfer.created
✗ application_fee.created
```

### Validation Checklist

- [ ] Production endpoint URL is HTTPS
- [ ] Only required events are subscribed
- [ ] Webhook secret is configured correctly
- [ ] Test events are processed successfully
- [ ] Webhook delivery monitoring is set up

## 10. Monitoring and Debugging

### Requirements

Webhook monitoring must provide:
- Processing success/failure rates
- Event processing latency
- Database update success rates
- Error classification and alerts

### Cursor Prompt Pattern for Monitoring

```
Add monitoring to webhook handler:
1. Log structured data for each webhook event
2. Include event_id, event_type, order_id, processing_result
3. Measure processing time
4. Track database operation success/failure
5. Create alerts for high failure rates

Log format example:
{
  "timestamp": "2024-01-01T00:00:00Z",
  "event_id": "evt_...",
  "event_type": "payment_intent.succeeded",
  "order_id": "order_123",
  "processing_time_ms": 150,
  "status": "success"
}
```

### Debugging Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| 400 Invalid Signature | Wrong webhook secret | Verify STRIPE_WEBHOOK_SECRET |
| Events not processed | Wrong endpoint URL | Check Stripe Dashboard URL |
| Database not updating | RLS policies | Use admin Supabase client |
| Duplicate processing | No idempotency | Check event IDs |

### Validation Checklist

- [ ] Structured logging implemented
- [ ] Processing metrics tracked
- [ ] Error rates monitored
- [ ] Database operation success tracked
- [ ] Alerting configured for failures

## Implementation Workflow

### Phase 1: Basic Handler
1. Create webhook endpoint structure
2. Implement signature verification
3. Add basic event routing
4. Test with Stripe CLI

### Phase 2: Event Processing
1. Add payment_intent.succeeded handler
2. Add payment_intent.payment_failed handler
3. Add charge.refunded handler
4. Test each event type

### Phase 3: Error Handling
1. Add database error handling
2. Implement structured logging
3. Add monitoring capabilities
4. Test failure scenarios

### Phase 4: Production Deployment
1. Configure production webhook
2. Set up monitoring alerts
3. Test with real payments
4. Monitor webhook delivery

## Cursor Workflow Tips

### Effective Webhook Prompting

1. **Specify Event Types Clearly**:
   ```
   Handle exactly these events:
   - payment_intent.succeeded
   - payment_intent.payment_failed
   - charge.refunded
   ```

2. **Define Security Requirements**:
   ```
   Always verify signatures first:
   - Use stripe.webhooks.constructEvent()
   - Return 400 for invalid signatures
   - Never process unverified events
   ```

3. **Database Error Handling**:
   ```
   Database operations must:
   - Log errors but not throw
   - Return 200 even if DB update fails
   - Include order context in error logs
   ```

## Summary

This webhook specification provides a secure, reliable event processing system for payment status synchronization. The spec-driven approach ensures:

- Security through signature verification
- Reliability through proper error handling
- Maintainability through structured code
- Debuggability through comprehensive logging

Follow the Cursor prompts to implement each component, then validate using the provided checklists.