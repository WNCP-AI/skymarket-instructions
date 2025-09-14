# SkyMarket Email Integration Documentation

Complete email system documentation for the SkyMarket drone service marketplace platform using Resend.

## Overview

SkyMarket implements transactional email notifications using Resend to keep users informed throughout the booking and payment lifecycle. The email system is designed to be lightweight, reliable, and failure-resistant with graceful error handling.

**Email Integration Features**:
- Booking confirmation notifications for consumers and providers
- Payment success notifications via Stripe webhooks
- Fallback-safe implementation (email failures don't break core functionality)
- HTML email templates with booking details

## Architecture

### Resend Integration

**Location**: `/lib/resend.ts`

```typescript
import { Resend } from 'resend'

let resendClient: Resend | null = null
const apiKey = process.env.RESEND_API_KEY
if (apiKey) {
  resendClient = new Resend(apiKey)
}

type BookingEmail = {
  to: string
  subject: string
  html: string
}

export async function sendBookingEmail(params: BookingEmail) {
  const from = process.env.RESEND_FROM_EMAIL || 'no-reply@skymarket.app'
  if (!resendClient) return
  return resendClient.emails.send({
    from,
    to: params.to,
    subject: params.subject,
    html: params.html
  })
}
```

### Environment Configuration

```bash
# Resend Configuration
RESEND_API_KEY=re_...                    # Required for email functionality
RESEND_FROM_EMAIL=no-reply@skymarket.com # Optional: defaults to no-reply@skymarket.app

# Development/Testing
RESEND_API_KEY=                          # Leave empty to disable emails in development
```

### Error Handling Strategy

The email system is designed with **graceful degradation**:

```typescript
// Email failures never break core functionality
try {
  if (consumerProfile?.email) {
    await sendBookingEmail({ to: consumerProfile.email, subject, html })
  }
  if (providerProfile?.email) {
    await sendBookingEmail({ to: providerProfile.email, subject, html })
  }
} catch {
  // Do not fail the request on email errors
  // Core booking/payment functionality continues normally
}
```

## Current Email Templates

### 1. Booking Confirmation Email

**Trigger**: New booking created via `/api/orders`
**Recipients**: Consumer and Provider
**Template Location**: Inline HTML in `/app/api/orders/route.ts`

```typescript
const subject = `New SkyMarket Order: ${listingDetail?.title ?? 'Service'} (#${created.id.slice(0, 8)})`
const html = `
  <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
    <h2>Order Created</h2>
    <p>Order ID: <strong>${created.id}</strong></p>
    <p>Service: ${listingDetail?.title ?? 'Service'}</p>
    <p>Scheduled At: ${new Date(scheduledAt).toLocaleString()}</p>
    <p>Dropoff: ${dropoffAddress}</p>
    <p>Total: $${priceTotal.toFixed(2)}</p>
  </div>
`
```

**Email Details**:
- **Subject Format**: `New SkyMarket Order: {service_title} (#{booking_id_short})`
- **Content**: Order ID, service details, scheduling info, total price
- **Styling**: Inline CSS for email client compatibility

### 2. Payment Confirmation Email

**Trigger**: Payment succeeded via Stripe webhook `/api/webhooks/stripe`
**Recipients**: Consumer and Provider
**Template Location**: Inline HTML in `/app/api/webhooks/stripe/route.ts`

```typescript
const subject = `Payment received: ${listing?.title ?? 'Service'} (#${booking.id.slice(0, 8)})`
const html = `
  <div>
    <h2>Payment received</h2>
    <p>Order ${booking.id} has been paid.</p>
    <p>Total: $${Number(booking.price_total).toFixed(2)}</p>
  </div>
`
```

**Email Details**:
- **Subject Format**: `Payment received: {service_title} (#{booking_id_short})`
- **Content**: Payment confirmation, order reference, total amount
- **Trigger**: Stripe `payment_intent.succeeded` webhook event

## Email Sending Patterns

### 1. Booking Creation Flow

**Location**: `/app/api/orders/route.ts` (lines 175-196)

```typescript
// After successful booking creation
const subject = `New SkyMarket Order: ${listingDetail?.title ?? 'Service'} (#${created.id.slice(0, 8)})`
const html = `
  <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
    <h2>Order Created</h2>
    <p>Order ID: <strong>${created.id}</strong></p>
    <p>Service: ${listingDetail?.title ?? 'Service'}</p>
    <p>Scheduled At: ${new Date(p.scheduledAt).toLocaleString()}</p>
    <p>Dropoff: ${p.dropoffAddress}</p>
    <p>Total: $${priceTotal.toFixed(2)}</p>
  </div>
`

// Send to both parties
if (consumerProfile?.email) {
  await sendBookingEmail({ to: consumerProfile.email, subject, html })
}
if (providerProfile?.email) {
  await sendBookingEmail({ to: providerProfile.email, subject, html })
}
```

### 2. Payment Success Flow

**Location**: `/app/api/webhooks/stripe/route.ts` (lines 67-101)

```typescript
case 'payment_intent.succeeded': {
  const intent = event.data.object as Stripe.PaymentIntent
  const bookingId = intent.metadata?.booking_id

  if (bookingId) {
    // Update payment status first
    await supabaseAdmin
      .from('bookings')
      .update({ payment_status: 'paid' })
      .eq('id', bookingId)

    // Fetch booking and related data
    const { data: booking } = await supabaseAdmin
      .from('bookings')
      .select('id, listing_id, provider_id, consumer_id, price_total')
      .eq('id', bookingId)
      .single()

    if (booking) {
      // Get additional context for email
      const [{ data: listing }, { data: consumer }, { data: providerOwner }] = await Promise.all([
        supabaseAdmin.from('listings').select('title').eq('id', booking.listing_id).single(),
        supabaseAdmin.from('profiles').select('email, full_name').eq('id', booking.consumer_id).single(),
        supabaseAdmin
          .from('providers')
          .select('user_id')
          .eq('id', booking.provider_id)
          .single(),
      ])

      // Generate email content
      const subject = `Payment received: ${listing?.title ?? 'Service'} (#${booking.id.slice(0, 8)})`
      const html = `<div><h2>Payment received</h2><p>Order ${booking.id} has been paid.</p><p>Total: $${Number(booking.price_total).toFixed(2)}</p></div>`

      // Send notifications
      if (consumer?.email) await sendBookingEmail({ to: consumer.email, subject, html })
      if (providerProfile?.email) await sendBookingEmail({ to: providerProfile.email, subject, html })
    }
  }
  break
}
```

## Email Client Compatibility

### HTML Email Best Practices

The current implementation uses basic HTML with inline styles for maximum compatibility:

```html
<div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
  <h2>Order Created</h2>
  <p>Order ID: <strong>${orderId}</strong></p>
  <p>Service: ${serviceTitle}</p>
  <p>Total: $${amount}</p>
</div>
```

**Design Principles**:
- **Inline CSS**: All styles inline for email client compatibility
- **System Fonts**: Arial/sans-serif for universal support
- **Max Width**: 600px for mobile and desktop readability
- **Simple Structure**: Basic HTML elements (div, h2, p, strong)
- **No External Assets**: No images or external CSS dependencies

### Responsive Design

```html
<!-- Mobile-friendly responsive pattern -->
<div style="
  font-family: Arial, sans-serif;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  line-height: 1.6;
">
  <h2 style="color: #333; margin-bottom: 20px;">Email Title</h2>
  <p style="margin-bottom: 15px;">Email content...</p>
</div>
```

## Testing and Development

### Local Development

```typescript
// Development mode - emails disabled
if (!process.env.RESEND_API_KEY) {
  console.log('[EMAIL] Would send:', { to, subject, html })
  return // No email sent in development
}
```

### Testing Email Templates

```typescript
// Test email template rendering
export function generateBookingConfirmationEmail(booking: BookingData): EmailTemplate {
  const subject = `New SkyMarket Order: ${booking.listing.title} (#${booking.id.slice(0, 8)})`
  const html = `
    <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
      <h2>Order Created</h2>
      <p>Order ID: <strong>${booking.id}</strong></p>
      <p>Service: ${booking.listing.title}</p>
      <p>Scheduled At: ${new Date(booking.scheduled_at).toLocaleString()}</p>
      <p>Dropoff: ${booking.dropoff_address}</p>
      <p>Total: $${booking.price_total.toFixed(2)}</p>
    </div>
  `
  return { subject, html }
}

// Test in development
const testEmail = generateBookingConfirmationEmail(mockBooking)
console.log('Email Preview:', testEmail)
```

### Email Delivery Testing

```bash
# Test with Resend CLI (if available)
resend emails send \
  --from "no-reply@skymarket.com" \
  --to "test@example.com" \
  --subject "Test SkyMarket Email" \
  --html "<h1>Test Email</h1>"
```

## Error Handling and Monitoring

### Error Handling Strategy

```typescript
export async function sendBookingEmail(params: BookingEmail) {
  const from = process.env.RESEND_FROM_EMAIL || 'no-reply@skymarket.app'

  // Graceful failure if no client configured
  if (!resendClient) {
    console.log('[EMAIL] Resend not configured, skipping email')
    return null
  }

  try {
    const result = await resendClient.emails.send({
      from,
      to: params.to,
      subject: params.subject,
      html: params.html
    })
    console.log('[EMAIL] Sent successfully:', result.data?.id)
    return result
  } catch (error) {
    console.error('[EMAIL] Send failed:', error)
    // Don't throw - email failures should not break core functionality
    return null
  }
}
```

### Email Metrics and Monitoring

```typescript
// Enhanced email logging for monitoring
export async function sendBookingEmailWithMetrics(params: BookingEmail & {
  bookingId: string,
  emailType: 'booking_created' | 'payment_received'
}) {
  const startTime = Date.now()

  try {
    const result = await sendBookingEmail(params)

    // Log success metrics
    console.log('[EMAIL] Success:', {
      bookingId: params.bookingId,
      emailType: params.emailType,
      recipient: params.to,
      duration: Date.now() - startTime,
      messageId: result?.data?.id
    })

    return result
  } catch (error) {
    // Log failure metrics
    console.error('[EMAIL] Failure:', {
      bookingId: params.bookingId,
      emailType: params.emailType,
      recipient: params.to,
      duration: Date.now() - startTime,
      error: error.message
    })

    return null
  }
}
```

## Future Enhancements

### Planned Email Features

1. **Email Templates**:
   - Move from inline HTML to Resend template system
   - Create branded email templates with SkyMarket styling
   - Support for multiple template variants

2. **Additional Email Types**:
   - Order status updates (accepted, in_progress, completed)
   - Booking cancellation notifications
   - Provider verification emails
   - Marketing and promotional emails

3. **Enhanced Personalization**:
   - Dynamic content based on user preferences
   - Localization support for Detroit area
   - A/B testing for email effectiveness

4. **Advanced Features**:
   - Email scheduling for time-sensitive notifications
   - Delivery tracking and read receipts
   - Unsubscribe management
   - Email bounce handling

### Template System Migration

```typescript
// Future: Resend template-based emails
export async function sendBookingEmailFromTemplate(params: {
  to: string
  templateId: string
  variables: Record<string, any>
  bookingId: string
}) {
  if (!resendClient) return null

  return resendClient.emails.send({
    from: 'no-reply@skymarket.com',
    to: params.to,
    template: params.templateId,
    variables: params.variables
  })
}

// Usage
await sendBookingEmailFromTemplate({
  to: consumer.email,
  templateId: 'booking-confirmation-v1',
  variables: {
    customerName: consumer.full_name,
    orderId: booking.id.slice(0, 8),
    serviceTitle: listing.title,
    scheduledAt: booking.scheduled_at,
    total: booking.price_total
  },
  bookingId: booking.id
})
```

### Email Analytics Integration

```typescript
// Future: Email engagement tracking
interface EmailMetrics {
  emailsSent: number
  deliveryRate: number
  openRate: number
  clickRate: number
  bounceRate: number
}

export async function getEmailMetrics(dateRange: { from: string, to: string }): Promise<EmailMetrics> {
  // Integration with Resend analytics API
  // Track email performance and engagement
}
```

## Production Configuration

### Domain Setup

1. **Domain Verification**: Configure sending domain in Resend dashboard
2. **DNS Records**: Set up DKIM, SPF, and DMARC for deliverability
3. **From Address**: Use verified domain email address

```bash
# Production environment variables
RESEND_API_KEY=re_production_key_here
RESEND_FROM_EMAIL=noreply@skymarket.com

# Sending domain should be verified in Resend dashboard
```

### Deliverability Best Practices

1. **Sender Reputation**: Use consistent from address and domain
2. **Content Quality**: Avoid spam trigger words and formatting
3. **List Hygiene**: Handle bounces and unsubscribes properly
4. **Volume Management**: Gradual sending volume increases

### Security Considerations

1. **API Key Security**: Store Resend API key securely in environment variables
2. **Email Validation**: Validate recipient email addresses before sending
3. **Content Sanitization**: Ensure user-generated content is sanitized in emails
4. **Rate Limiting**: Implement rate limiting for email sending endpoints

This email system provides a solid foundation for SkyMarket's notification requirements while maintaining simplicity and reliability. The graceful error handling ensures that email delivery issues never impact core marketplace functionality.