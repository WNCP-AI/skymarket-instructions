# Step 13: Stripe Checkout - Complete Payment Flow

## Objective
Implement a complete Stripe checkout flow for SkyMarket service bookings with secure payment processing, webhook handling, and proper error management. This creates the foundation for all payment transactions in your marketplace.

## What You'll Learn
- How to integrate Stripe Checkout Sessions for secure payments
- Webhook implementation for payment confirmation
- Local development setup with Stripe CLI
- Payment flow error handling and recovery
- Testing payment scenarios with test cards

## Prerequisites
- Completed Step 12 (Context7 MCP configured)
- Stripe account created with test API keys
- SkyMarket app deployed to Vercel with environment variables
- Basic understanding of payment processing concepts

## Understanding Stripe Payment Flow

### SkyMarket Payment Architecture
```
User clicks "Pay" → Checkout Session → Stripe Checkout → Webhook → Order Update → Email
```

1. **Checkout Session Creation:** Server creates secure payment session
2. **User Payment:** Customer completes payment on Stripe's secure form
3. **Webhook Notification:** Stripe notifies your app of payment status
4. **Order Update:** Your app updates booking/order status
5. **Confirmation:** Email sent to customer and provider

### Key Components
- **Frontend:** Payment buttons and loading states
- **API Routes:** Checkout session creation and webhook handling
- **Database:** Payment status tracking
- **Email:** Transaction confirmations

## Instructions

### 1. Environment Setup

Ensure these environment variables are configured in both local and Vercel:

```bash
# Stripe Configuration
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Optional: Product attribution
STRIPE_BOOKING_PRODUCT_ID=prod_...
```

**Security Notes:**
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`: Safe for client-side use
- `STRIPE_SECRET_KEY`: Server-only, never expose to client
- `STRIPE_WEBHOOK_SECRET`: Verifies webhook authenticity

### 2. Set Up Local Development with Stripe CLI

**Install Stripe CLI:**
```bash
# Install via Homebrew (macOS)
brew install stripe/stripe-cli/stripe

# Or download from stripe.com/docs/stripe-cli
```

**Configure Stripe CLI:**
```bash
# Login to your Stripe account
stripe login

# Forward webhooks to your local development server
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

**Important:** Keep this command running during development to receive webhook events.

### 3. Implement Checkout Flow

**Cursor Prompt:**
```
Create a complete Stripe checkout implementation for SkyMarket bookings. I need:

1. API route at app/api/orders/[id]/checkout/route.ts that:
   - Creates a Stripe Checkout Session
   - Includes booking details and pricing
   - Sets success/cancel URLs
   - Uses proper error handling

2. Client-side payment button component that:
   - Triggers the checkout session creation
   - Handles loading states
   - Redirects to Stripe Checkout
   - Shows proper error messages

3. Webhook handler at app/api/webhooks/stripe/route.ts that:
   - Verifies webhook signatures
   - Handles payment_intent.succeeded events
   - Updates booking payment status
   - Includes proper error handling

Use context7 for current Stripe API patterns and security best practices.
```

### 4. Test Your Implementation

**Start Your Development Environment:**
```bash
# Terminal 1: Start Next.js dev server
npm run dev

# Terminal 2: Start Stripe webhook forwarding
stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

**Test Scenarios:**

1. **Successful Payment:**
   - Card: `4242 4242 4242 4242`
   - Expiry: Any future date
   - CVC: Any 3 digits
   - ZIP: Any valid ZIP code

2. **Declined Payment:**
   - Card: `4000 0000 0000 9995` (Insufficient funds)
   - Card: `4000 0000 0000 0002` (Generic decline)

3. **Authentication Required:**
   - Card: `4000 0025 0000 3155` (Requires 3D Secure)

### 5. Understanding the Payment Flow

**Cursor Prompt:**
```
Walk me through the call chain when I click "Pay with Stripe Checkout" on /orders/[id]:
- Where is the server action or client-side handler?
- Which API route is called for checkout session creation?
- How does the redirect back to our app work?
- Which fields on bookings are updated by the webhook?
- What happens if the webhook fails?

Provide the complete technical flow with error handling scenarios.
```

## Code Examples

### Checkout Session Creation

```typescript
// app/api/orders/[id]/checkout/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe/server'
import { createClient } from '@/lib/supabase/server'

export const runtime = 'nodejs' // Required for Stripe
export const dynamic = 'force-dynamic'

export async function POST(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const supabase = createClient()
    
    // Fetch booking details
    const { data: booking, error } = await supabase
      .from('bookings')
      .select('*, listings(title, price)')
      .eq('id', params.id)
      .single()

    if (error || !booking) {
      return NextResponse.json(
        { error: 'Booking not found' },
        { status: 404 }
      )
    }

    // Create Stripe Checkout Session
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      line_items: [
        {
          price_data: {
            currency: 'usd',
            product_data: {
              name: booking.listings.title,
              description: `SkyMarket Service Booking #${booking.id}`,
            },
            unit_amount: Math.round(booking.listings.price * 100), // Convert to cents
          },
          quantity: 1,
        },
      ],
      mode: 'payment',
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/orders/${params.id}?payment=success`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/orders/${params.id}?payment=cancelled`,
      metadata: {
        bookingId: params.id,
      },
    })

    return NextResponse.json({ sessionId: session.id })
  } catch (error) {
    console.error('Checkout session creation failed:', error)
    return NextResponse.json(
      { error: 'Failed to create checkout session' },
      { status: 500 }
    )
  }
}
```

### Client-Side Payment Button

```typescript
// components/payments/CheckoutButton.tsx
'use client'

import { useState } from 'react'
import { loadStripe } from '@stripe/stripe-js'
import { Button } from '@/components/ui/button'

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!)

interface CheckoutButtonProps {
  bookingId: string
  amount: number
}

export function CheckoutButton({ bookingId, amount }: CheckoutButtonProps) {
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const handleCheckout = async () => {
    try {
      setLoading(true)
      setError(null)

      // Create checkout session
      const response = await fetch(`/api/orders/${bookingId}/checkout`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
      })

      if (!response.ok) {
        throw new Error('Failed to create checkout session')
      }

      const { sessionId } = await response.json()
      
      // Redirect to Stripe Checkout
      const stripe = await stripePromise
      const result = await stripe?.redirectToCheckout({ sessionId })

      if (result?.error) {
        setError(result.error.message || 'Payment failed')
      }
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Payment failed')
    } finally {
      setLoading(false)
    }
  }

  return (
    <div>
      <Button 
        onClick={handleCheckout} 
        disabled={loading}
        className="w-full"
      >
        {loading ? 'Processing...' : `Pay $${amount.toFixed(2)}`}
      </Button>
      {error && (
        <p className="mt-2 text-sm text-red-600">
          {error}
        </p>
      )}
    </div>
  )
}
```

### Webhook Handler

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { headers } from 'next/headers'
import { stripe } from '@/lib/stripe/server'
import { createClient } from '@/lib/supabase/server'

export const runtime = 'nodejs'
export const dynamic = 'force-dynamic'

export async function POST(request: NextRequest) {
  try {
    const body = await request.text()
    const signature = headers().get('stripe-signature')

    if (!signature) {
      return NextResponse.json(
        { error: 'Missing signature' },
        { status: 400 }
      )
    }

    // Verify webhook signature
    const event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )

    const supabase = createClient()

    switch (event.type) {
      case 'checkout.session.completed':
        const session = event.data.object
        const bookingId = session.metadata?.bookingId

        if (bookingId) {
          // Update booking payment status
          const { error } = await supabase
            .from('bookings')
            .update({
              payment_status: 'completed',
              payment_intent_id: session.payment_intent,
              updated_at: new Date().toISOString(),
            })
            .eq('id', bookingId)

          if (error) {
            console.error('Failed to update booking:', error)
            return NextResponse.json(
              { error: 'Database update failed' },
              { status: 500 }
            )
          }
        }
        break

      case 'payment_intent.payment_failed':
        const paymentIntent = event.data.object
        // Handle failed payment logic
        break

      default:
        console.log(`Unhandled event type: ${event.type}`)
    }

    return NextResponse.json({ received: true })
  } catch (error) {
    console.error('Webhook error:', error)
    return NextResponse.json(
      { error: 'Webhook handler failed' },
      { status: 400 }
    )
  }
}
```

## Expected Outcome

After completing this step, you should have:

✅ **Working Payment Flow:**
- Checkout button creates Stripe sessions successfully
- Users can complete payments on Stripe's secure forms
- Successful payments redirect back to order page
- Failed payments show appropriate error messages

✅ **Webhook Integration:**
- Webhooks properly verify signatures
- Payment events update booking status in database
- Failed webhook events are logged for debugging
- Local development receives webhook events via CLI

✅ **Error Handling:**
- Network failures show user-friendly messages
- Invalid payments are handled gracefully
- Database errors don't crash the application
- All edge cases have appropriate responses

✅ **Security Implementation:**
- Webhook signatures are verified
- Sensitive operations use server-side API routes
- Payment amounts are validated server-side
- No sensitive data exposed to client

## Troubleshooting Tips

### Common Payment Issues

**❌ "Checkout session creation failed"**
```bash
# Check your Stripe secret key is correct
# Verify the booking exists in your database
# Ensure price is a valid positive number
# Check server console for detailed error messages
```

**❌ Webhooks not received locally**
```bash
# Ensure Stripe CLI is running:
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Check the webhook endpoint URL is correct
# Verify the webhook secret matches CLI output
```

**❌ "Invalid signature" webhook errors**
```bash
# Copy webhook secret from Stripe CLI output
# Ensure STRIPE_WEBHOOK_SECRET matches exactly
# Restart your dev server after updating env vars
```

**❌ Payment completes but order not updated**
```bash
# Check webhook handler logs for errors
# Verify booking ID is passed in session metadata
# Check database permissions and constraints
# Ensure Supabase connection is working
```

### Testing Checklist

Before moving to production:

- [ ] Successful payment updates booking status
- [ ] Failed payment shows error message
- [ ] Webhook signature verification works
- [ ] Local development receives all webhook events
- [ ] Error scenarios don't crash the application
- [ ] Payment amounts are calculated correctly
- [ ] Order page reflects payment status changes
- [ ] Database transactions are properly handled

### Production Deployment Notes

1. **Switch to Live Keys:** Replace test keys with live Stripe keys
2. **Configure Production Webhooks:** Set up webhook endpoints in Stripe Dashboard
3. **Test Webhook Delivery:** Verify production webhooks reach your app
4. **Monitor Payment Errors:** Set up logging and alerting for payment failures
5. **Backup Webhook Processing:** Consider implementing retry logic for failed webhooks

## Next Steps

With payment processing working, you're ready to:
- **Step 14:** Implement email notifications for successful payments
- **Step 15:** Add advanced features like refunds and partial payments
- **Step 16:** Create comprehensive order management dashboards

## References

### Essential Documentation
- [Stripe Checkout](https://stripe.com/docs/checkout) - Complete checkout implementation guide
- [Stripe Webhooks](https://stripe.com/docs/webhooks) - Webhook handling and security
- [Stripe CLI](https://stripe.com/docs/stripe-cli#listen) - Local development setup

### Testing Resources
- [Test Card Numbers](https://stripe.com/docs/testing) - Complete list of test scenarios
- [Webhook Testing](https://stripe.com/docs/webhooks/test) - Local webhook development
- [Payment Error Codes](https://stripe.com/docs/error-codes) - Understanding payment failures

---

💡 **Pro Tip:** Always test your payment flow with various scenarios including network failures, browser back button, and incomplete payments. Real users will encounter these edge cases, so your app should handle them gracefully.


