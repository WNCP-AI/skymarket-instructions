# Step 20: Email Automation System

## Objective

Implement a comprehensive email automation system using React Email and Resend to handle all transactional communications in the SkyMarket platform. This step demonstrates production-ready email infrastructure with professional templates, automated triggers, and robust error handling.

## Context Setup

This step integrates multiple specifications and official documentation:

- **Primary Spec**: @docs/specs/email/EMAIL.md - Complete email system architecture
- **Project Config**: @AGENTS.md - SkyMarket business logic and technical patterns
- **Official Documentation**: Context7 integration for React Email and Resend best practices
- **Integration Points**: Authentication system, booking lifecycle, payment events, user preferences

## Context7 Integration

Use Context7 to get up-to-date official documentation while implementing. Simply include "use context7" in your development prompts:

**Example Integration Prompts:**

"Create professional email templates using React Email components with responsive design and accessibility features for transactional emails. use context7"

"Set up Resend API SDK with TypeScript for reliable email delivery including error handling, retry logic, and delivery tracking. use context7"

"Configure Resend webhooks for delivery tracking, bounce handling, and email analytics with proper security validation. use context7"

"Implement React Email template personalization with dynamic content and localization support. use context7"

"Set up email testing and preview development workflow with React Email for debugging and validation. use context7"

## Implementation Prompts

### Phase 1: Email Infrastructure Setup

**Prompt 1: Enhanced Email Service Layer**
```
Based on @docs/specs/email/EMAIL.md specifications, enhance the existing email service at lib/resend.ts with:

1. Professional email template system using React Email components
2. Email type definitions and template management
3. Enhanced error handling with retry logic and fallback strategies
4. Email analytics and delivery tracking integration
5. User preference management for notification settings
6. Template rendering with personalization and localization support

Set up Resend API SDK with TypeScript for reliable email delivery including error handling, retry logic, and delivery tracking. Create professional email templates using React Email components with responsive design. use context7
```

**Prompt 2: React Email Template Library**
```
Create a comprehensive email template library in emails/ directory:

1. Base template with SkyMarket branding (logo, colors, typography)
2. Booking confirmation templates (consumer and provider versions)
3. Payment confirmation and receipt templates
4. Booking status update templates (accepted, in_progress, completed, cancelled)
5. Authentication templates (welcome, verification, password reset)
6. Provider notification templates (new booking, payment received)
7. Responsive design optimized for mobile and desktop email clients

Create professional email templates using React Email components with responsive design and accessibility features for transactional emails. Implement React Email template personalization with dynamic content and branding. use context7
```

### Phase 2: Automated Email Triggers

**Prompt 3: Booking Lifecycle Email Automation**
```
Implement automated email triggers for the complete booking lifecycle by enhancing existing API routes:

1. Update app/api/orders/route.ts to send professional booking confirmation emails
2. Enhance booking status update endpoints to trigger appropriate notification emails
3. Add email notifications for booking cancellations with cancellation policies
4. Implement reminder emails for upcoming bookings (24 hours before)
5. Add completion confirmation emails with review request prompts

Ensure all email triggers are wrapped in try-catch blocks to prevent disruption of core functionality, with proper logging for monitoring and debugging.
```

**Prompt 4: Payment and Authentication Email Integration**
```
Based on the Stripe webhook system and Supabase Auth integration, implement:

1. Enhanced payment confirmation emails with detailed receipts and tax information
2. Payout notification emails for providers when payments are released
3. Payment failure and retry notification emails
4. Welcome email series for new user onboarding
5. Email verification and password reset using custom templates
6. Provider verification completion emails with next steps

Integrate with existing webhook handlers and auth flows without disrupting current functionality.
```

### Phase 3: Email Management and Analytics

**Prompt 5: User Email Preferences**
```
Create a comprehensive email preference management system:

1. Database schema for user email preferences and notification settings
2. User preference UI in dashboard for managing email notifications
3. Unsubscribe handling with granular preferences (not just all-or-nothing)
4. Email preference enforcement in all email sending functions
5. Compliance with email marketing regulations (CAN-SPAM, GDPR)
6. Preference sync across consumer and provider roles

Include migration scripts and update existing email functions to respect user preferences.
```

**Prompt 6: Email Analytics and Monitoring**
```
Implement email analytics and monitoring capabilities:

1. Email delivery tracking and status monitoring
2. Open rate and click tracking for transactional emails (where appropriate)
3. Bounce and complaint handling with automatic list management
4. Email sending rate monitoring and alerting
5. Template performance analytics and A/B testing framework
6. Integration with health check system for email service monitoring
7. Dashboard for email analytics in admin interface

Configure Resend webhooks for delivery tracking, bounce handling, and email analytics with proper security validation and monitoring integration. use context7
```

## Code Examples

### Enhanced Email Service

```typescript
// lib/email/service.ts
import { Resend } from 'resend'
import { render } from '@react-email/render'
import { z } from 'zod'
import type { ReactElement } from 'react'

// Email types and schemas
const emailSchema = z.object({
  to: z.string().email(),
  subject: z.string().min(1).max(200),
  template: z.string(),
  data: z.record(z.any()),
  tags?: z.array(z.string()).optional(),
  scheduledFor?: z.date().optional()
})

export type EmailRequest = z.infer<typeof emailSchema>

export interface EmailTemplate {
  component: ReactElement
  subject: string
  previewText?: string
}

// Enhanced email service with retry and monitoring
export class EmailService {
  private resend: Resend | null = null
  private retryAttempts = 3
  private retryDelay = 1000

  constructor() {
    const apiKey = process.env.RESEND_API_KEY
    if (apiKey) {
      this.resend = new Resend(apiKey)
    }
  }

  async sendEmail(request: EmailRequest): Promise<boolean> {
    const validation = emailSchema.safeParse(request)
    if (!validation.success) {
      console.error('[EMAIL] Invalid request:', validation.error)
      return false
    }

    // Check user preferences
    const canSend = await this.checkUserPreferences(request.to, request.template)
    if (!canSend) {
      console.log('[EMAIL] User opted out:', { to: request.to, template: request.template })
      return false
    }

    return this.sendWithRetry(validation.data)
  }

  private async sendWithRetry(request: EmailRequest, attempt = 1): Promise<boolean> {
    if (!this.resend) {
      console.log('[EMAIL] Service not configured')
      return false
    }

    try {
      const template = await this.renderTemplate(request.template, request.data)
      const result = await this.resend.emails.send({
        from: process.env.RESEND_FROM_EMAIL || 'noreply@skymarket.com',
        to: request.to,
        subject: template.subject,
        html: render(template.component),
        text: render(template.component, { plainText: true }),
        tags: request.tags,
        scheduledAt: request.scheduledFor?.toISOString()
      })

      // Track success metrics
      await this.trackEmailMetrics('sent', request.template, true)
      console.log('[EMAIL] Sent successfully:', result.data?.id)
      return true

    } catch (error) {
      console.error(`[EMAIL] Attempt ${attempt} failed:`, error)

      if (attempt < this.retryAttempts) {
        await this.delay(this.retryDelay * attempt)
        return this.sendWithRetry(request, attempt + 1)
      }

      await this.trackEmailMetrics('failed', request.template, false)
      return false
    }
  }

  private async checkUserPreferences(email: string, template: string): Promise<boolean> {
    // Implementation for checking user email preferences
    // Return true if user wants to receive this type of email
    return true // Simplified for example
  }

  private async renderTemplate(templateName: string, data: any): Promise<EmailTemplate> {
    // Dynamic template loading and rendering
    const templates = await import(`../emails/${templateName}`)
    return templates.default(data)
  }

  private async trackEmailMetrics(event: string, template: string, success: boolean) {
    // Track email metrics for analytics
    console.log(`📊 [EMAIL] ${event}:`, { template, success, timestamp: new Date().toISOString() })
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
}

export const emailService = new EmailService()
```

### React Email Templates

```tsx
// emails/booking-confirmation.tsx
import {
  Body,
  Container,
  Head,
  Heading,
  Html,
  Img,
  Link,
  Preview,
  Section,
  Text,
  Button,
  Row,
  Column
} from '@react-email/components'

interface BookingConfirmationProps {
  customerName: string
  orderId: string
  serviceTitle: string
  providerName: string
  scheduledAt: string
  dropoffAddress: string
  total: number
  isProvider: boolean
}

export default function BookingConfirmation({
  customerName,
  orderId,
  serviceTitle,
  providerName,
  scheduledAt,
  dropoffAddress,
  total,
  isProvider = false
}: BookingConfirmationProps) {
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL || 'https://skymarket.com'

  return (
    <Html>
      <Head />
      <Preview>
        {isProvider ? 'New booking received' : 'Booking confirmed'} - {serviceTitle}
      </Preview>
      <Body style={main}>
        <Container style={container}>
          {/* Header */}
          <Section style={header}>
            <Img
              src={`${baseUrl}/logo-email.png`}
              width="120"
              height="40"
              alt="SkyMarket"
              style={logo}
            />
          </Section>

          {/* Main Content */}
          <Section style={content}>
            <Heading style={h1}>
              {isProvider ? '🚁 New Booking Received!' : '✅ Booking Confirmed!'}
            </Heading>

            <Text style={text}>
              Hi {customerName},
            </Text>

            <Text style={text}>
              {isProvider
                ? `You have a new booking for your ${serviceTitle} service.`
                : `Your booking has been confirmed! Here are the details:`
              }
            </Text>

            {/* Booking Details */}
            <Section style={bookingCard}>
              <Row>
                <Column>
                  <Text style={label}>Order ID</Text>
                  <Text style={value}>#{orderId.slice(0, 8).toUpperCase()}</Text>
                </Column>
                <Column style={textRight}>
                  <Text style={label}>Total</Text>
                  <Text style={price}>${total.toFixed(2)}</Text>
                </Column>
              </Row>

              <hr style={divider} />

              <Row>
                <Column>
                  <Text style={label}>Service</Text>
                  <Text style={value}>{serviceTitle}</Text>
                </Column>
              </Row>

              <Row>
                <Column>
                  <Text style={label}>{isProvider ? 'Customer' : 'Provider'}</Text>
                  <Text style={value}>{providerName}</Text>
                </Column>
              </Row>

              <Row>
                <Column>
                  <Text style={label}>Scheduled</Text>
                  <Text style={value}>{new Date(scheduledAt).toLocaleString()}</Text>
                </Column>
              </Row>

              <Row>
                <Column>
                  <Text style={label}>Location</Text>
                  <Text style={value}>{dropoffAddress}</Text>
                </Column>
              </Row>
            </Section>

            {/* CTA Button */}
            <Section style={buttonSection}>
              <Button style={button} href={`${baseUrl}/dashboard/bookings/${orderId}`}>
                {isProvider ? 'Manage Booking' : 'View Booking Details'}
              </Button>
            </Section>

            {/* Next Steps */}
            <Section>
              <Text style={text}>
                <strong>What's next?</strong>
              </Text>
              {isProvider ? (
                <Text style={text}>
                  Please review the booking details and accept or decline within 24 hours.
                  The customer will be notified once you respond.
                </Text>
              ) : (
                <Text style={text}>
                  Your provider will review and confirm your booking within 24 hours.
                  You'll receive another email once your booking is accepted.
                </Text>
              )}
            </Section>
          </Section>

          {/* Footer */}
          <Section style={footer}>
            <Text style={footerText}>
              Need help? Contact us at{' '}
              <Link href="mailto:support@skymarket.com" style={link}>
                support@skymarket.com
              </Link>
            </Text>
            <Text style={footerText}>
              <Link href={`${baseUrl}/preferences`} style={link}>
                Email Preferences
              </Link>
              {' • '}
              <Link href={`${baseUrl}/help`} style={link}>
                Help Center
              </Link>
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  )
}

// Styles
const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif',
}

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  maxWidth: '600px',
  marginTop: '20px',
  marginBottom: '20px',
  borderRadius: '8px',
  overflow: 'hidden',
  boxShadow: '0 4px 12px rgba(0, 0, 0, 0.05)',
}

const header = {
  backgroundColor: '#1e40af',
  padding: '20px',
  textAlign: 'center' as const,
}

const logo = {
  margin: '0 auto',
  filter: 'brightness(0) invert(1)',
}

const content = {
  padding: '32px',
}

const h1 = {
  color: '#1f2937',
  fontSize: '24px',
  fontWeight: '600',
  lineHeight: '1.25',
  margin: '0 0 20px 0',
}

const text = {
  color: '#374151',
  fontSize: '16px',
  lineHeight: '1.5',
  margin: '0 0 16px 0',
}

const bookingCard = {
  backgroundColor: '#f9fafb',
  border: '1px solid #e5e7eb',
  borderRadius: '8px',
  padding: '24px',
  margin: '24px 0',
}

const label = {
  color: '#6b7280',
  fontSize: '14px',
  fontWeight: '500',
  margin: '0 0 4px 0',
  textTransform: 'uppercase' as const,
  letterSpacing: '0.5px',
}

const value = {
  color: '#111827',
  fontSize: '16px',
  fontWeight: '600',
  margin: '0 0 16px 0',
}

const price = {
  color: '#059669',
  fontSize: '18px',
  fontWeight: '700',
  margin: '0 0 16px 0',
}

const textRight = {
  textAlign: 'right' as const,
}

const divider = {
  borderColor: '#e5e7eb',
  margin: '16px 0',
}

const buttonSection = {
  textAlign: 'center' as const,
  margin: '32px 0',
}

const button = {
  backgroundColor: '#1e40af',
  borderRadius: '6px',
  color: '#ffffff',
  fontSize: '16px',
  fontWeight: '600',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'inline-block',
  padding: '12px 24px',
}

const footer = {
  backgroundColor: '#f9fafb',
  padding: '24px 32px',
  borderTop: '1px solid #e5e7eb',
  textAlign: 'center' as const,
}

const footerText = {
  color: '#6b7280',
  fontSize: '14px',
  lineHeight: '1.5',
  margin: '0 0 8px 0',
}

const link = {
  color: '#1e40af',
  textDecoration: 'none',
}

// Export template configuration
export const template = {
  subject: (data: BookingConfirmationProps) =>
    `${data.isProvider ? 'New Booking' : 'Booking Confirmed'}: ${data.serviceTitle} (#${data.orderId.slice(0, 8).toUpperCase()})`,

  previewText: (data: BookingConfirmationProps) =>
    `${data.isProvider ? 'New booking received' : 'Booking confirmed'} for ${data.serviceTitle}`,
}
```

## Verification Steps

### 1. Email Template Testing
```bash
# Install React Email CLI
npm install -g @react-email/cli

# Preview email templates
npx react-email preview

# Test template rendering
npm run test:emails
```

### 2. Email Delivery Testing
```bash
# Test email sending in development
curl -X POST http://localhost:3000/api/test/email \
  -H "Content-Type: application/json" \
  -d '{
    "template": "booking-confirmation",
    "to": "test@example.com",
    "data": {
      "customerName": "John Doe",
      "orderId": "test-order-123",
      "serviceTitle": "Drone Food Delivery",
      "providerName": "Sky Foods",
      "scheduledAt": "2024-01-15T14:30:00Z",
      "dropoffAddress": "123 Main St, Detroit, MI",
      "total": 25.99,
      "isProvider": false
    }
  }'

# Verify email preferences
curl -X GET http://localhost:3000/api/user/email-preferences \
  -H "Authorization: Bearer <token>"
```

### 3. Integration Verification
```typescript
// Test email automation triggers
describe('Email Automation', () => {
  it('should send booking confirmation when order created', async () => {
    const mockEmailService = jest.spyOn(emailService, 'sendEmail')

    const response = await fetch('/api/orders', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(mockBookingData)
    })

    expect(response.ok).toBe(true)
    expect(mockEmailService).toHaveBeenCalledWith({
      template: 'booking-confirmation',
      to: 'customer@example.com',
      data: expect.objectContaining({
        orderId: expect.any(String),
        serviceTitle: 'Drone Food Delivery'
      })
    })
  })

  it('should respect user email preferences', async () => {
    // Test that users who opted out don't receive emails
    await updateUserPreferences('user@example.com', {
      bookingConfirmations: false
    })

    const mockEmailService = jest.spyOn(emailService, 'sendEmail')

    // Trigger email
    await emailService.sendEmail({
      template: 'booking-confirmation',
      to: 'user@example.com',
      data: mockData
    })

    // Verify email was not sent due to preferences
    expect(mockEmailService).toHaveReturnedWith(false)
  })
})
```

### 4. Production Readiness Check
```bash
# Verify email deliverability setup
dig TXT skymarket.com | grep -i spf
dig TXT _dmarc.skymarket.com
dig TXT mail._domainkey.skymarket.com

# Test email service health
curl https://skymarket.com/api/health | jq '.services.email'

# Monitor email metrics
curl -X GET https://api.resend.com/emails \
  -H "Authorization: Bearer $RESEND_API_KEY" | jq '.data[]'
```

## Expected Outcome

After completing this step, you will have:

1. **Professional Email Infrastructure**:
   - React Email template library with SkyMarket branding
   - Type-safe email service with retry logic and error handling
   - Template personalization and responsive design

2. **Complete Email Automation**:
   - Automated booking lifecycle emails (confirmation, status updates, completion)
   - Payment and receipt notifications with detailed transaction information
   - Authentication emails (welcome, verification, password reset)
   - Provider notifications for new bookings and payments

3. **User Email Management**:
   - Granular email preference settings in user dashboard
   - Unsubscribe handling with preference persistence
   - GDPR and CAN-SPAM compliance features

4. **Email Analytics and Monitoring**:
   - Delivery tracking and bounce handling
   - Email performance metrics and monitoring
   - Integration with health check system
   - Administrative email analytics dashboard

5. **Production-Ready Features**:
   - Email template versioning and A/B testing capability
   - Rate limiting and delivery optimization
   - Comprehensive error handling and fallback strategies
   - Security and compliance measures

## Troubleshooting

### Common Email Issues

**Email Templates Not Rendering**
```bash
# Check React Email setup
npx react-email preview
npm run build:emails

# Debug template rendering
console.log(await render(BookingConfirmation(mockData)))
```

**Delivery Issues**
```bash
# Check Resend configuration
curl -X GET https://api.resend.com/domains \
  -H "Authorization: Bearer $RESEND_API_KEY"

# Verify SPF/DKIM records
dig TXT skymarket.com
```

**High Bounce Rates**
```typescript
// Monitor and handle bounces
export async function handleBounce(email: string, reason: string) {
  // Mark email as bounced
  await updateEmailStatus(email, 'bounced')

  // Remove from active list if hard bounce
  if (reason.includes('permanent')) {
    await removeFromEmailList(email)
  }
}
```

**Email Preferences Not Working**
```sql
-- Check email preferences schema
SELECT * FROM user_email_preferences WHERE user_id = 'user-id';

-- Verify preference checks in email service
SELECT email, notification_type, enabled
FROM email_preferences
WHERE email = 'user@example.com';
```

## Next Steps

- **Step 21**: Complete Vercel deployment with domain configuration and performance optimization
- **Advanced Features**: Implement email scheduling, drip campaigns, and advanced analytics
- **Compliance**: Add GDPR data export and deletion capabilities for email data
- **Optimization**: Implement email template caching and CDN delivery for images

This email automation system provides production-ready transactional email capabilities that enhance user experience while maintaining compliance and reliability standards essential for a marketplace platform.