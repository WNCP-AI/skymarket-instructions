# Step 14: Resend Email Notifications on Payment Success

## Objective
Implement automated email notifications using Resend when payments are successfully processed. This creates a professional communication system that keeps both customers and service providers informed about booking confirmations and payment status.

## What You'll Learn
- How to integrate Resend for transactional emails
- Creating responsive HTML email templates
- Implementing email notifications in Stripe webhooks
- Error handling and retry patterns for email delivery
- Best practices for transactional email content

## Prerequisites
- Completed Step 13 (Stripe checkout implementation)
- Resend account with API key and verified domain
- Understanding of HTML email best practices
- Basic knowledge of webhook processing

## Understanding Email Notifications in SkyMarket

### Email Flow Architecture
```
Payment Success → Webhook → Database Update → Email Notifications → Recipients
```

1. **Stripe webhook** receives payment confirmation
2. **Database** updates booking status to "paid"
3. **Email service** sends notifications to relevant parties
4. **Error handling** manages delivery failures gracefully

### Email Recipients
- **Consumer:** Booking confirmation with service details
- **Service Provider:** New booking notification with customer contact
- **Admin (optional):** High-value transaction alerts

## Instructions

### 1. Environment Configuration

Ensure these environment variables are set:

```bash
# Resend Configuration
RESEND_API_KEY=re_your_api_key_here
RESEND_FROM_EMAIL=noreply@yourdomain.com  # Must be verified in Resend

# Optional: Email customization
RESEND_REPLY_TO_EMAIL=support@yourdomain.com
NEXT_PUBLIC_APP_NAME=SkyMarket
```

**Domain Verification:**
- Add your domain to Resend dashboard
- Configure DNS records for authentication
- Verify domain status before testing

### 2. Create Email Service Module

**Cursor Prompt:**
```
Create a comprehensive email service module at lib/resend.ts that:

1. Initializes Resend client with proper configuration
2. Includes error handling and logging
3. Has functions for different email types:
   - sendBookingConfirmation (to customer)
   - sendProviderNotification (to service provider)
   - sendPaymentReceipt (transaction details)

4. Uses responsive HTML templates that work across email clients
5. Includes retry logic for failed sends
6. Proper TypeScript types for all email data

Use context7 to get the latest Resend API patterns and best practices.
```

### 3. Design Email Templates

**Cursor Prompt:**
```
Create accessible, responsive HTML email templates for SkyMarket:

1. Booking confirmation template with:
   - SkyMarket branding
   - Service details (title, price, date/time)
   - Provider contact information
   - Booking reference number
   - Clear call-to-action buttons
   - Mobile-responsive design

2. Provider notification template with:
   - New booking alert
   - Customer contact details
   - Service requirements
   - Booking management links
   - Professional formatting

3. Payment receipt template with:
   - Transaction summary
   - Itemized pricing
   - Payment method details
   - Next steps information

Ensure all templates are accessible (WCAG compliant) and work in major email clients.
```

### 4. Integrate with Stripe Webhooks

**Cursor Prompt:**
```
Update the Stripe webhook handler at app/api/webhooks/stripe/route.ts to:

1. Send emails after successful payment processing
2. Extract booking and user details from database
3. Handle email sending errors gracefully
4. Log email delivery status
5. Include retry mechanism for failed emails
6. Process different webhook events appropriately

Focus on the payment_intent.succeeded and checkout.session.completed events.
```

### 5. Test Email Delivery

**Development Testing:**
```bash
# Start your development environment
npm run dev

# In another terminal, start Stripe CLI
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Complete a test payment and verify emails are sent
```

**Test Checklist:**
- [ ] Customer receives booking confirmation email
- [ ] Provider receives new booking notification
- [ ] Payment receipt contains accurate transaction details
- [ ] Email formatting looks good on mobile and desktop
- [ ] Unsubscribe links are included (if applicable)

### 6. Handle Email Delivery Failures

**Cursor Prompt:**
```
Implement robust error handling for email delivery failures in our webhook:

1. Catch and log Resend API errors with context
2. Implement simple retry logic (max 3 attempts)
3. Store failed email attempts for manual review
4. Continue webhook processing even if emails fail
5. Create email delivery status tracking

Don't introduce complex job queues - keep it simple but reliable.
```

## Code Examples

### Resend Service Module

```typescript
// lib/resend.ts
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

interface BookingEmailData {
  customerEmail: string
  customerName: string
  providerEmail: string
  providerName: string
  serviceName: string
  bookingId: string
  amount: number
  scheduledDate: string
}

export async function sendBookingConfirmationEmail(data: BookingEmailData) {
  try {
    const { data: result, error } = await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL!,
      to: [data.customerEmail],
      replyTo: process.env.RESEND_REPLY_TO_EMAIL,
      subject: `Booking Confirmed - ${data.serviceName}`,
      html: generateBookingConfirmationHTML(data),
    })

    if (error) {
      console.error('Email send error:', error)
      return { success: false, error }
    }

    console.log('Booking confirmation sent:', result?.id)
    return { success: true, data: result }
  } catch (error) {
    console.error('Email service error:', error)
    return { success: false, error }
  }
}

export async function sendProviderNotificationEmail(data: BookingEmailData) {
  try {
    const { data: result, error } = await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL!,
      to: [data.providerEmail],
      replyTo: process.env.RESEND_REPLY_TO_EMAIL,
      subject: `New Booking Request - ${data.serviceName}`,
      html: generateProviderNotificationHTML(data),
    })

    if (error) {
      console.error('Provider notification error:', error)
      return { success: false, error }
    }

    console.log('Provider notification sent:', result?.id)
    return { success: true, data: result }
  } catch (error) {
    console.error('Email service error:', error)
    return { success: false, error }
  }
}

// Email template functions
function generateBookingConfirmationHTML(data: BookingEmailData): string {
  return `
    <!DOCTYPE html>
    <html lang=\"en\">
    <head>
      <meta charset=\"UTF-8\">
      <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">
      <title>Booking Confirmation - SkyMarket</title>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #1f2937; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background: #f9fafb; }
        .booking-details { background: white; padding: 20px; margin: 20px 0; border-radius: 8px; }
        .button { display: inline-block; padding: 12px 24px; background: #3b82f6; color: white; text-decoration: none; border-radius: 6px; }
        @media (max-width: 600px) { .container { padding: 10px; } }
      </style>
    </head>
    <body>
      <div class=\"container\">
        <div class=\"header\">
          <h1>🚁 SkyMarket</h1>
          <p>Your booking is confirmed!</p>
        </div>
        <div class=\"content\">
          <p>Hi ${data.customerName},</p>
          <p>Great news! Your booking has been confirmed and payment processed successfully.</p>
          
          <div class=\"booking-details\">
            <h3>Booking Details</h3>
            <p><strong>Service:</strong> ${data.serviceName}</p>
            <p><strong>Provider:</strong> ${data.providerName}</p>
            <p><strong>Scheduled Date:</strong> ${data.scheduledDate}</p>
            <p><strong>Amount Paid:</strong> $${data.amount.toFixed(2)}</p>
            <p><strong>Booking ID:</strong> ${data.bookingId}</p>
          </div>
          
          <p>Your service provider will contact you soon to coordinate the details.</p>
          
          <p style=\"text-align: center;\">
            <a href=\"${process.env.NEXT_PUBLIC_APP_URL}/orders/${data.bookingId}\" class=\"button\">
              View Booking Details
            </a>
          </p>
          
          <p>Questions? Reply to this email or contact our support team.</p>
          <p>Best regards,<br>The SkyMarket Team</p>
        </div>
      </div>
    </body>
    </html>
  `
}

function generateProviderNotificationHTML(data: BookingEmailData): string {
  return `
    <!DOCTYPE html>
    <html lang=\"en\">
    <head>
      <meta charset=\"UTF-8\">
      <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">
      <title>New Booking - SkyMarket</title>
      <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        .header { background: #059669; color: white; padding: 20px; text-align: center; }
        .content { padding: 20px; background: #f0fdf4; }
        .booking-details { background: white; padding: 20px; margin: 20px 0; border-radius: 8px; }
        .button { display: inline-block; padding: 12px 24px; background: #059669; color: white; text-decoration: none; border-radius: 6px; }
      </style>
    </head>
    <body>
      <div class=\"container\">
        <div class=\"header\">
          <h1>🚁 SkyMarket</h1>
          <p>New booking received!</p>
        </div>
        <div class=\"content\">
          <p>Hi ${data.providerName},</p>
          <p>You've received a new paid booking! The customer has completed payment and is ready to proceed.</p>
          
          <div class=\"booking-details\">
            <h3>Booking Information</h3>
            <p><strong>Service:</strong> ${data.serviceName}</p>
            <p><strong>Customer:</strong> ${data.customerName}</p>
            <p><strong>Customer Email:</strong> ${data.customerEmail}</p>
            <p><strong>Scheduled Date:</strong> ${data.scheduledDate}</p>
            <p><strong>Payment Amount:</strong> $${data.amount.toFixed(2)}</p>
            <p><strong>Booking ID:</strong> ${data.bookingId}</p>
          </div>
          
          <p>Please contact the customer within 24 hours to coordinate service delivery.</p>
          
          <p style=\"text-align: center;\">
            <a href=\"${process.env.NEXT_PUBLIC_APP_URL}/dashboard/bookings/${data.bookingId}\" class=\"button\">
              Manage Booking
            </a>
          </p>
          
          <p>Questions? Contact our provider support team.</p>
          <p>Best regards,<br>The SkyMarket Team</p>
        </div>
      </div>
    </body>
    </html>
  `
}
```

### Updated Webhook with Email Integration

```typescript
// app/api/webhooks/stripe/route.ts (updated excerpt)
import { sendBookingConfirmationEmail, sendProviderNotificationEmail } from '@/lib/resend'

export async function POST(request: NextRequest) {
  try {
    // ... webhook verification code ...

    switch (event.type) {
      case 'checkout.session.completed':
        const session = event.data.object
        const bookingId = session.metadata?.bookingId

        if (bookingId) {
          // Update booking status
          const { data: booking, error: updateError } = await supabase
            .from('bookings')
            .update({
              payment_status: 'completed',
              payment_intent_id: session.payment_intent,
              updated_at: new Date().toISOString(),
            })
            .eq('id', bookingId)
            .select(`
              *,
              users:consumer_id(email, name),
              listings(title, price, users:provider_id(email, name))
            `)
            .single()

          if (updateError) {
            console.error('Failed to update booking:', updateError)
            return NextResponse.json({ error: 'Database update failed' }, { status: 500 })
          }

          // Send email notifications (don't fail webhook if emails fail)
          if (booking) {
            try {
              const emailData = {
                customerEmail: booking.users.email,
                customerName: booking.users.name,
                providerEmail: booking.listings.users.email,
                providerName: booking.listings.users.name,
                serviceName: booking.listings.title,
                bookingId: booking.id,
                amount: booking.listings.price,
                scheduledDate: new Date(booking.scheduled_at).toLocaleDateString(),
              }

              // Send emails with retry logic
              await sendEmailsWithRetry(emailData)
            } catch (emailError) {
              console.error('Email sending failed:', emailError)
              // Log but don't fail the webhook
            }
          }
        }
        break

      // ... other webhook events ...
    }

    return NextResponse.json({ received: true })
  } catch (error) {
    console.error('Webhook error:', error)
    return NextResponse.json({ error: 'Webhook handler failed' }, { status: 400 })
  }
}

async function sendEmailsWithRetry(emailData: any, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      // Send customer email
      const customerResult = await sendBookingConfirmationEmail(emailData)
      if (!customerResult.success) {
        throw new Error(`Customer email failed: ${customerResult.error}`)
      }

      // Send provider email
      const providerResult = await sendProviderNotificationEmail(emailData)
      if (!providerResult.success) {
        throw new Error(`Provider email failed: ${providerResult.error}`)
      }

      console.log('All emails sent successfully')
      return
    } catch (error) {
      console.error(`Email attempt ${attempt} failed:`, error)
      
      if (attempt === maxRetries) {
        // Log final failure for manual review
        console.error('All email attempts failed:', {
          bookingId: emailData.bookingId,
          error: error.message,
          timestamp: new Date().toISOString(),
        })
        throw error
      }
      
      // Wait before retry (exponential backoff)
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt))
    }
  }
}
```

## Expected Outcome

After completing this step, you should have:

✅ **Email Service Integration:**
- Resend properly configured with API keys
- Professional email templates for different scenarios
- Robust error handling and logging
- Mobile-responsive email designs

✅ **Automated Notifications:**
- Customers receive booking confirmations automatically
- Providers get notified of new paid bookings
- Payment receipts include transaction details
- Emails trigger immediately after successful payments

✅ **Error Handling:**
- Failed email deliveries don't crash webhooks
- Retry logic handles temporary failures
- Comprehensive logging for debugging issues
- Graceful degradation when email service is down

✅ **Professional Communication:**
- Consistent branding across all emails
- Clear, actionable content for recipients
- Accessible HTML that works in all email clients
- Proper unsubscribe and reply-to configurations

## Troubleshooting Tips

### Common Email Issues

**❌ Emails not sending**
```bash
# Check Resend API key is correct
# Verify from email domain is verified in Resend
# Check webhook is receiving payment events
# Review server logs for specific error messages
```

**❌ Emails going to spam**
```bash
# Verify SPF, DKIM, and DMARC DNS records
# Use verified sending domain, not generic email
# Avoid spam trigger words in subject lines
# Include unsubscribe links and physical address
```

**❌ Email templates broken in some clients**
```bash
# Use table-based layouts for better compatibility
# Inline CSS styles instead of external stylesheets
# Test with Email on Acid or Litmus
# Provide plain-text fallbacks
```

**❌ "From email not verified" error**
```bash
# Add your domain to Resend dashboard
# Configure required DNS records (SPF, DKIM)
# Wait for verification (can take 24-48 hours)
# Use a subdomain like mail.yourdomain.com
```

### Testing Best Practices

1. **Test Across Email Clients:** Gmail, Outlook, Apple Mail, mobile clients
2. **Verify All Scenarios:** Successful payment, failed payment, refunds
3. **Check Accessibility:** Screen reader compatibility, proper alt text
4. **Monitor Delivery Rates:** Track open rates and spam complaints
5. **Validate Links:** Ensure all CTAs and tracking links work properly

## Advanced Features

### Email Personalization

```typescript
// Add dynamic content based on user data
interface EmailPersonalization {
  timezone: string
  preferredLanguage: string
  bookingHistory: number
  loyaltyTier: 'bronze' | 'silver' | 'gold'
}
```

### Email Analytics

```typescript
// Track email engagement
const trackingData = {
  campaign: 'booking-confirmation',
  userId: booking.consumer_id,
  bookingId: booking.id,
  timestamp: new Date().toISOString(),
}
```

### Unsubscribe Management

```typescript
// Handle unsubscribe preferences
export async function handleUnsubscribe(token: string, types: string[]) {
  // Update user preferences in database
  // Respect user choices for different email types
}
```

## Next Steps

With email notifications working, you're ready to:
- **Expert Track:** Implement advanced AI features and tooling
- **Production:** Deploy your complete marketplace application
- **Enhancement:** Add features like email preferences, templates, and analytics

## References

### Essential Documentation
- [Resend Quickstart](https://resend.com/docs) - Complete API reference
- [Email HTML Best Practices](https://templates.mailchimp.com/resources/email-client-css-support/) - Cross-client compatibility
- [Transactional Email Guide](https://postmarkapp.com/guides/transactional-email-templates) - Professional email design

### Testing Tools
- [Email Testing Tools](https://www.emailonacid.com/) - Cross-client testing
- [Spam Testing](https://www.mail-tester.com/) - Deliverability scoring
- [HTML Email Templates](https://github.com/leemunroe/responsive-html-email-template) - Responsive templates

---

💡 **Pro Tip:** Always test your email templates across different email clients and screen sizes. Email rendering varies significantly between Gmail, Outlook, and mobile clients, so thorough testing ensures a professional experience for all users.