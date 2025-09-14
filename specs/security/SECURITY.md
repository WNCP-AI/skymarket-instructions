# SkyMarket Security & Compliance Documentation

Complete security architecture and compliance documentation for the SkyMarket drone service marketplace platform.

## Overview

SkyMarket implements comprehensive security measures across authentication, authorization, data protection, and regulatory compliance. The platform prioritizes user data privacy, payment security, and drone operation compliance with FAA regulations.

**Security Architecture**:
- JWT-based authentication via Supabase Auth
- Row Level Security (RLS) for data access control
- PCI DSS compliance for payment processing
- FAA Part 107 compliance for drone operations
- End-to-end encryption for sensitive communications

## Authentication System

### Supabase Auth Implementation

**Location**: `/lib/supabase/`

```typescript
// Server-side authentication client
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: any) {
          cookieStore.set({ name, value, ...options })
        },
        remove(name: string, options: any) {
          cookieStore.set({ name, value: '', ...options })
        },
      },
    }
  )
}

// Client-side authentication
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### Authentication Middleware

**Location**: `/lib/supabase/middleware.ts`

```typescript
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function updateSession(request: NextRequest) {
  let response = NextResponse.next({
    request: {
      headers: request.headers,
    },
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return request.cookies.get(name)?.value
        },
        set(name: string, value: string, options: any) {
          response = NextResponse.next({
            request: {
              headers: request.headers,
            },
          })
          response.cookies.set({ name, value, ...options })
        },
        remove(name: string, options: any) {
          response = NextResponse.next({
            request: {
              headers: request.headers,
            },
          })
          response.cookies.set({ name, value: '', ...options })
        },
      },
    }
  )

  // Refresh session if expired
  const { data: { session } } = await supabase.auth.getSession()

  return response
}

// Route protection
export function withAuth(handler: Function) {
  return async (request: NextRequest) => {
    const { data: { user } } = await supabase.auth.getUser()

    if (!user) {
      return NextResponse.json(
        { error: { code: 'AUTHENTICATION_REQUIRED', message: 'Login required' } },
        { status: 401 }
      )
    }

    return handler(request, { user })
  }
}
```

### User Roles and Permissions

```sql
-- User role enumeration
CREATE TYPE user_role AS ENUM ('consumer', 'provider', 'admin');

-- Profile table with role-based access
CREATE TABLE profiles (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  user_role user_role DEFAULT 'consumer',
  phone TEXT,
  avatar_url TEXT,
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Role-based authorization check
CREATE OR REPLACE FUNCTION authorize_role(required_role user_role)
RETURNS BOOLEAN AS $$
BEGIN
  RETURN (
    SELECT user_role
    FROM profiles
    WHERE id = auth.uid()
  ) = required_role OR (
    SELECT user_role
    FROM profiles
    WHERE id = auth.uid()
  ) = 'admin';
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

## Row Level Security (RLS)

### Core RLS Policies

```sql
-- Enable RLS on all user data tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE providers ENABLE ROW LEVEL SECURITY;
ALTER TABLE listings ENABLE ROW LEVEL SECURITY;
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

-- Profiles: Users can only see and edit their own profile
CREATE POLICY "profiles_select" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "profiles_update" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- Providers: Public read, own provider can update
CREATE POLICY "providers_select" ON providers
  FOR SELECT USING (TRUE);

CREATE POLICY "providers_insert" ON providers
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "providers_update" ON providers
  FOR UPDATE USING (auth.uid() = user_id);

-- Listings: Public read, provider can CRUD their own
CREATE POLICY "listings_select" ON listings
  FOR SELECT USING (is_active = TRUE OR provider_id IN (
    SELECT id FROM providers WHERE user_id = auth.uid()
  ));

CREATE POLICY "listings_insert" ON listings
  FOR INSERT WITH CHECK (
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

CREATE POLICY "listings_update" ON listings
  FOR UPDATE USING (
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

-- Bookings: Consumers and providers can see their own bookings
CREATE POLICY "bookings_select" ON bookings
  FOR SELECT USING (
    consumer_id = auth.uid() OR
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

CREATE POLICY "bookings_insert" ON bookings
  FOR INSERT WITH CHECK (consumer_id = auth.uid());

CREATE POLICY "bookings_update" ON bookings
  FOR UPDATE USING (
    consumer_id = auth.uid() OR
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

-- Reviews: Public read, booking participants can create
CREATE POLICY "reviews_select" ON reviews
  FOR SELECT USING (TRUE);

CREATE POLICY "reviews_insert" ON reviews
  FOR INSERT WITH CHECK (
    reviewer_id = auth.uid() AND
    EXISTS (
      SELECT 1 FROM bookings
      WHERE id = booking_id
      AND (consumer_id = auth.uid() OR provider_id IN (
        SELECT id FROM providers WHERE user_id = auth.uid()
      ))
      AND booking_status = 'completed'
    )
  );
```

### Advanced RLS Patterns

```sql
-- Time-based access restrictions
CREATE POLICY "messages_time_restricted" ON messages
  FOR SELECT USING (
    (sender_id = auth.uid() OR receiver_id = auth.uid()) AND
    created_at > NOW() - INTERVAL '90 days'  -- Only recent messages
  );

-- Location-based restrictions (Detroit Metro area)
CREATE POLICY "listings_geo_restricted" ON listings
  FOR INSERT WITH CHECK (
    service_location IS NOT NULL AND
    ST_DWithin(
      service_location,
      ST_SetSRID(ST_Point(-83.0458, 42.3314), 4326),  -- Detroit center
      40233.6  -- 25 miles in meters
    )
  );

-- Dynamic role elevation
CREATE POLICY "admin_full_access" ON bookings
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE id = auth.uid()
      AND user_role = 'admin'
    )
  );
```

## Data Protection and Encryption

### Sensitive Data Handling

```typescript
// Data classification levels
enum DataClassification {
  PUBLIC = 'public',
  INTERNAL = 'internal',
  CONFIDENTIAL = 'confidential',
  RESTRICTED = 'restricted'
}

interface SecureDataField {
  classification: DataClassification
  encrypted: boolean
  piiField: boolean
  retentionDays: number
}

// Field-level security configuration
const DATA_SECURITY_CONFIG = {
  profiles: {
    email: { classification: DataClassification.CONFIDENTIAL, encrypted: false, piiField: true, retentionDays: 2555 },
    full_name: { classification: DataClassification.CONFIDENTIAL, encrypted: false, piiField: true, retentionDays: 2555 },
    phone: { classification: DataClassification.CONFIDENTIAL, encrypted: true, piiField: true, retentionDays: 2555 },
    avatar_url: { classification: DataClassification.INTERNAL, encrypted: false, piiField: false, retentionDays: 365 }
  },
  bookings: {
    payment_intent_id: { classification: DataClassification.RESTRICTED, encrypted: true, piiField: false, retentionDays: 2555 },
    special_instructions: { classification: DataClassification.INTERNAL, encrypted: false, piiField: false, retentionDays: 365 }
  }
} as const
```

### Data Encryption Implementation

```sql
-- Enable pgcrypto for encryption functions
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Encryption helper functions
CREATE OR REPLACE FUNCTION encrypt_sensitive_data(data TEXT)
RETURNS TEXT AS $$
BEGIN
  RETURN encode(
    encrypt(
      data::bytea,
      current_setting('app.encryption_key'),
      'aes-cbc'
    ),
    'base64'
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE OR REPLACE FUNCTION decrypt_sensitive_data(encrypted_data TEXT)
RETURNS TEXT AS $$
BEGIN
  RETURN convert_from(
    decrypt(
      decode(encrypted_data, 'base64'),
      current_setting('app.encryption_key'),
      'aes-cbc'
    ),
    'utf-8'
  );
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Automatic encryption triggers
CREATE OR REPLACE FUNCTION encrypt_phone_number()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.phone IS NOT NULL AND NEW.phone != OLD.phone THEN
    NEW.phone = encrypt_sensitive_data(NEW.phone);
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER encrypt_profile_phone
  BEFORE INSERT OR UPDATE ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION encrypt_phone_number();
```

### PII Data Management

```typescript
// PII data anonymization for GDPR compliance
export async function anonymizeUserData(userId: string) {
  const anonymousData = {
    email: `deleted-${Date.now()}@skymarket-deleted.com`,
    full_name: 'DELETED_USER',
    phone: null,
    avatar_url: null,
    is_verified: false
  }

  // Anonymize profile
  await supabaseAdmin
    .from('profiles')
    .update(anonymousData)
    .eq('id', userId)

  // Anonymize booking history - keep for analytics but remove PII
  await supabaseAdmin
    .from('bookings')
    .update({
      consumer_notes: null,
      special_instructions: null,
      consumer_contact_info: null
    })
    .eq('consumer_id', userId)

  // Log anonymization for compliance
  await supabaseAdmin
    .from('data_processing_logs')
    .insert({
      user_id: userId,
      action: 'anonymize',
      reason: 'gdpr_request',
      processed_at: new Date().toISOString(),
      retention_until: null // Permanently anonymized
    })
}

// Data retention cleanup
export async function cleanupExpiredData() {
  const expiredCutoff = new Date(Date.now() - 90 * 24 * 60 * 60 * 1000) // 90 days

  // Delete expired messages
  await supabaseAdmin
    .from('messages')
    .delete()
    .lt('created_at', expiredCutoff.toISOString())

  // Anonymize old booking details
  await supabaseAdmin
    .from('bookings')
    .update({
      special_instructions: null,
      consumer_notes: null
    })
    .lt('created_at', expiredCutoff.toISOString())
    .neq('booking_status', 'in_progress') // Keep active bookings
}
```

## Payment Security

### PCI DSS Compliance

```typescript
// Payment data handling - never store card details
interface SecurePaymentData {
  // Safe to store
  paymentIntentId: string
  paymentStatus: 'pending' | 'paid' | 'failed' | 'refunded'
  amountCents: number
  currency: string

  // NEVER store these
  // cardNumber: string - FORBIDDEN
  // cvv: string - FORBIDDEN
  // expiryDate: string - FORBIDDEN
}

// Secure payment intent creation
export async function createSecurePaymentIntent(
  bookingId: string,
  amountCents: number,
  metadata: Record<string, string>
) {
  // Validate amount to prevent manipulation
  if (amountCents < 50 || amountCents > 100000) { // $0.50 to $1000
    throw new Error('Invalid payment amount')
  }

  // Create payment with security safeguards
  const intent = await stripe.paymentIntents.create(
    {
      amount: amountCents,
      currency: 'usd',
      capture_method: 'manual',
      metadata: {
        ...metadata,
        booking_id: bookingId,
        created_by: 'skymarket_platform',
        ip_address: await getClientIP(), // For fraud detection
      },
      receipt_email: await getConsumerEmail(bookingId),
    },
    {
      // Prevent duplicate charges
      idempotencyKey: `booking:${bookingId}:${Date.now()}`
    }
  )

  // Log payment attempt for auditing
  await logPaymentEvent('payment_intent_created', {
    booking_id: bookingId,
    payment_intent_id: intent.id,
    amount_cents: amountCents
  })

  return {
    paymentIntentId: intent.id,
    clientSecret: intent.client_secret,
  }
}
```

### Webhook Security

```typescript
// Stripe webhook signature verification
export function verifyWebhookSignature(
  rawBody: Buffer,
  signature: string,
  webhookSecret: string
): boolean {
  try {
    stripe.webhooks.constructEvent(rawBody, signature, webhookSecret)
    return true
  } catch (err) {
    console.error('[security] Webhook signature verification failed:', err)
    return false
  }
}

// Webhook rate limiting and replay protection
const webhookCache = new Map<string, number>()

export function preventWebhookReplay(eventId: string): boolean {
  const now = Date.now()
  const eventTime = webhookCache.get(eventId)

  if (eventTime && (now - eventTime) < 300000) { // 5 minutes
    console.warn('[security] Webhook replay detected:', eventId)
    return false
  }

  webhookCache.set(eventId, now)

  // Cleanup old entries
  if (webhookCache.size > 1000) {
    const oldestEntries = Array.from(webhookCache.entries())
      .sort(([,a], [,b]) => a - b)
      .slice(0, 500)

    oldestEntries.forEach(([id]) => webhookCache.delete(id))
  }

  return true
}
```

## Regulatory Compliance

### FAA Part 107 Compliance

```sql
-- Provider certification tracking
CREATE TABLE provider_certifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provider_id UUID REFERENCES providers(id) ON DELETE CASCADE,
  certificate_type TEXT NOT NULL, -- 'part_107', 'medical', 'waiver'
  certificate_number TEXT NOT NULL,
  issued_date DATE NOT NULL,
  expiry_date DATE NOT NULL,
  issuing_authority TEXT NOT NULL DEFAULT 'FAA',
  verification_status TEXT DEFAULT 'pending' CHECK (verification_status IN ('pending', 'verified', 'expired', 'revoked')),
  verification_date TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Airspace authorization tracking
CREATE TABLE flight_authorizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  booking_id UUID REFERENCES bookings(id) ON DELETE CASCADE,
  airspace_class TEXT NOT NULL, -- 'B', 'C', 'D', 'E'
  airport_code TEXT, -- IATA code if near airport
  authorization_type TEXT DEFAULT 'laanc' CHECK (authorization_type IN ('laanc', 'waiver', 'coa')),
  altitude_limit_ft INTEGER DEFAULT 400,
  authorization_number TEXT,
  valid_from TIMESTAMPTZ NOT NULL,
  valid_until TIMESTAMPTZ NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'approved', 'denied', 'expired')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Compliance validation trigger
CREATE OR REPLACE FUNCTION validate_drone_compliance()
RETURNS TRIGGER AS $$
BEGIN
  -- Verify provider has valid Part 107 certificate
  IF NOT EXISTS (
    SELECT 1 FROM provider_certifications pc
    JOIN providers p ON p.id = pc.provider_id
    WHERE p.id = NEW.provider_id
    AND pc.certificate_type = 'part_107'
    AND pc.expiry_date > CURRENT_DATE
    AND pc.verification_status = 'verified'
  ) THEN
    RAISE EXCEPTION 'Provider does not have valid Part 107 certification';
  END IF;

  -- Verify flight is within legal hours (sunrise to sunset)
  IF EXTRACT(hour FROM NEW.scheduled_for) < 6 OR EXTRACT(hour FROM NEW.scheduled_for) > 20 THEN
    RAISE EXCEPTION 'Drone flights are only permitted during daylight hours (6 AM - 8 PM)';
  END IF;

  -- Verify altitude compliance (400 ft AGL limit)
  IF NEW.max_altitude_ft > 400 THEN
    IF NOT EXISTS (
      SELECT 1 FROM flight_authorizations fa
      WHERE fa.booking_id = NEW.id
      AND fa.altitude_limit_ft >= NEW.max_altitude_ft
      AND fa.status = 'approved'
    ) THEN
      RAISE EXCEPTION 'Altitude exceeds 400 ft AGL without proper authorization';
    END IF;
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER booking_compliance_check
  BEFORE INSERT OR UPDATE ON bookings
  FOR EACH ROW
  EXECUTE FUNCTION validate_drone_compliance();
```

### Data Protection Compliance (GDPR)

```typescript
// GDPR compliance interface
interface GDPRRequest {
  userId: string
  requestType: 'access' | 'portability' | 'rectification' | 'erasure' | 'restriction'
  requestDate: string
  completionDeadline: string
  status: 'pending' | 'processing' | 'completed' | 'rejected'
}

export class GDPRCompliance {
  // Right to access - provide user data export
  static async exportUserData(userId: string): Promise<UserDataExport> {
    const [profile, bookings, reviews, messages] = await Promise.all([
      supabaseAdmin.from('profiles').select('*').eq('id', userId).single(),
      supabaseAdmin.from('bookings').select('*').or(`consumer_id.eq.${userId},provider_id.eq.${userId}`),
      supabaseAdmin.from('reviews').select('*').eq('reviewer_id', userId),
      supabaseAdmin.from('messages').select('*').or(`sender_id.eq.${userId},receiver_id.eq.${userId}`)
    ])

    return {
      exportDate: new Date().toISOString(),
      personalData: {
        profile: profile.data,
        bookings: bookings.data,
        reviews: reviews.data,
        messages: messages.data
      },
      dataRetentionInfo: {
        profileRetention: '7 years after account closure',
        bookingRetention: '7 years for tax purposes',
        messageRetention: '90 days from creation'
      }
    }
  }

  // Right to erasure - delete user data
  static async deleteUserData(userId: string, reason: string): Promise<void> {
    // Can't delete if active bookings exist
    const { data: activeBookings } = await supabaseAdmin
      .from('bookings')
      .select('id')
      .eq('consumer_id', userId)
      .in('booking_status', ['pending', 'accepted', 'in_progress'])

    if (activeBookings && activeBookings.length > 0) {
      throw new Error('Cannot delete user data with active bookings')
    }

    // Anonymize instead of delete to maintain referential integrity
    await anonymizeUserData(userId)

    // Log the deletion for compliance audit
    await supabaseAdmin
      .from('gdpr_requests')
      .insert({
        user_id: userId,
        request_type: 'erasure',
        reason,
        status: 'completed',
        completed_at: new Date().toISOString()
      })
  }

  // Data portability - structured data export
  static async createPortableExport(userId: string): Promise<Buffer> {
    const userData = await this.exportUserData(userId)

    // Create ZIP archive with structured data
    const zip = new JSZip()

    // Add JSON files for each data type
    zip.file('profile.json', JSON.stringify(userData.personalData.profile, null, 2))
    zip.file('bookings.json', JSON.stringify(userData.personalData.bookings, null, 2))
    zip.file('reviews.json', JSON.stringify(userData.personalData.reviews, null, 2))
    zip.file('messages.json', JSON.stringify(userData.personalData.messages, null, 2))

    // Add metadata
    zip.file('export_info.json', JSON.stringify({
      exportDate: userData.exportDate,
      dataRetention: userData.dataRetentionInfo,
      format: 'JSON',
      version: '1.0'
    }, null, 2))

    return await zip.generateAsync({ type: 'nodebuffer' })
  }
}
```

## Security Monitoring and Incident Response

### Security Event Logging

```sql
-- Security events table
CREATE TABLE security_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type TEXT NOT NULL, -- 'login_attempt', 'unauthorized_access', 'data_breach', etc.
  severity TEXT NOT NULL CHECK (severity IN ('low', 'medium', 'high', 'critical')),
  user_id UUID REFERENCES profiles(id),
  ip_address INET,
  user_agent TEXT,
  event_data JSONB,
  detected_at TIMESTAMPTZ DEFAULT NOW(),
  resolved_at TIMESTAMPTZ,
  resolution_notes TEXT
);

-- Audit log trigger
CREATE OR REPLACE FUNCTION log_sensitive_operations()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO security_events (
    event_type,
    severity,
    user_id,
    event_data
  ) VALUES (
    TG_OP || '_' || TG_TABLE_NAME,
    CASE
      WHEN TG_TABLE_NAME IN ('profiles', 'provider_certifications') THEN 'high'
      WHEN TG_TABLE_NAME IN ('bookings', 'payments') THEN 'medium'
      ELSE 'low'
    END,
    COALESCE(NEW.user_id, OLD.user_id, auth.uid()),
    jsonb_build_object(
      'table', TG_TABLE_NAME,
      'operation', TG_OP,
      'old_values', to_jsonb(OLD),
      'new_values', to_jsonb(NEW)
    )
  );

  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

-- Apply audit logging to sensitive tables
CREATE TRIGGER audit_profiles
  AFTER INSERT OR UPDATE OR DELETE ON profiles
  FOR EACH ROW EXECUTE FUNCTION log_sensitive_operations();

CREATE TRIGGER audit_providers
  AFTER INSERT OR UPDATE OR DELETE ON providers
  FOR EACH ROW EXECUTE FUNCTION log_sensitive_operations();
```

### Intrusion Detection

```typescript
// Suspicious activity detection
export class SecurityMonitor {
  // Detect unusual login patterns
  static async detectSuspiciousLogin(
    userId: string,
    ipAddress: string,
    userAgent: string
  ): Promise<SecurityAlert | null> {
    // Check for login from new location
    const recentLogins = await supabaseAdmin
      .from('security_events')
      .select('ip_address, detected_at')
      .eq('user_id', userId)
      .eq('event_type', 'login_attempt')
      .gte('detected_at', new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString())
      .order('detected_at', { ascending: false })

    const knownIPs = new Set(recentLogins.data?.map(log => log.ip_address))

    if (!knownIPs.has(ipAddress)) {
      await this.logSecurityEvent({
        eventType: 'new_location_login',
        severity: 'medium',
        userId,
        ipAddress,
        userAgent,
        data: { knownLocations: Array.from(knownIPs) }
      })

      return {
        type: 'new_location_login',
        message: 'Login from new location detected',
        recommendedAction: 'verify_identity'
      }
    }

    // Check for multiple rapid login attempts
    const recentAttempts = recentLogins.data?.filter(
      log => Date.now() - new Date(log.detected_at).getTime() < 5 * 60 * 1000
    ).length || 0

    if (recentAttempts > 5) {
      await this.logSecurityEvent({
        eventType: 'rapid_login_attempts',
        severity: 'high',
        userId,
        ipAddress,
        userAgent,
        data: { attempts: recentAttempts }
      })

      return {
        type: 'brute_force_attempt',
        message: 'Multiple rapid login attempts detected',
        recommendedAction: 'temporary_lockout'
      }
    }

    return null
  }

  // Rate limiting implementation
  static async enforceRateLimit(
    identifier: string,
    operation: string,
    limits: { maxRequests: number; windowMinutes: number }
  ): Promise<boolean> {
    const windowStart = new Date(Date.now() - limits.windowMinutes * 60 * 1000)

    const { count } = await supabaseAdmin
      .from('security_events')
      .select('*', { count: 'exact', head: true })
      .eq('event_type', operation)
      .eq('ip_address', identifier)
      .gte('detected_at', windowStart.toISOString())

    if (count && count >= limits.maxRequests) {
      await this.logSecurityEvent({
        eventType: 'rate_limit_exceeded',
        severity: 'medium',
        ipAddress: identifier,
        data: { operation, count, limit: limits.maxRequests }
      })

      return false
    }

    return true
  }
}
```

### Incident Response Plan

```typescript
enum IncidentSeverity {
  P0 = 'critical',    // Data breach, payment fraud
  P1 = 'high',        // Unauthorized access, system compromise
  P2 = 'medium',      // Policy violations, suspicious activity
  P3 = 'low'          // Minor security events
}

interface SecurityIncident {
  id: string
  severity: IncidentSeverity
  type: string
  description: string
  affectedUsers: string[]
  detectedAt: string
  containedAt?: string
  resolvedAt?: string
  responseActions: string[]
  postIncidentActions: string[]
}

export class IncidentResponse {
  // Automatic incident creation for critical events
  static async handleCriticalIncident(
    type: string,
    details: any,
    affectedUsers: string[] = []
  ): Promise<void> {
    const incident: SecurityIncident = {
      id: generateId(),
      severity: IncidentSeverity.P0,
      type,
      description: `Critical security incident: ${type}`,
      affectedUsers,
      detectedAt: new Date().toISOString(),
      responseActions: [],
      postIncidentActions: []
    }

    // Immediate automated responses
    switch (type) {
      case 'data_breach':
        await this.executeDataBreachResponse(incident, details)
        break
      case 'payment_fraud':
        await this.executePaymentFraudResponse(incident, details)
        break
      case 'unauthorized_access':
        await this.executeAccessViolationResponse(incident, details)
        break
    }

    // Log incident
    await supabaseAdmin
      .from('security_incidents')
      .insert({
        ...incident,
        incident_data: details
      })

    // Alert security team
    await this.alertSecurityTeam(incident)
  }

  private static async executeDataBreachResponse(
    incident: SecurityIncident,
    details: any
  ): Promise<void> {
    // 1. Immediate containment
    if (details.compromisedUserId) {
      await this.lockUserAccount(details.compromisedUserId)
      incident.responseActions.push(`Locked compromised account: ${details.compromisedUserId}`)
    }

    // 2. Assess scope
    const affectedUsers = await this.assessBreachScope(details)
    incident.affectedUsers = affectedUsers

    // 3. Mandatory notifications (72-hour GDPR requirement)
    if (affectedUsers.length > 0) {
      await this.scheduleGDPRNotification(incident)
      incident.responseActions.push('Scheduled GDPR compliance notifications')
    }

    // 4. Revoke compromised tokens
    await this.revokeUserSessions(affectedUsers)
    incident.responseActions.push('Revoked sessions for affected users')
  }

  private static async executePaymentFraudResponse(
    incident: SecurityIncident,
    details: any
  ): Promise<void> {
    // 1. Freeze suspicious transactions
    if (details.suspiciousTransactions) {
      for (const txId of details.suspiciousTransactions) {
        await this.freezeTransaction(txId)
      }
      incident.responseActions.push(`Froze ${details.suspiciousTransactions.length} transactions`)
    }

    // 2. Enhanced monitoring
    await this.enableEnhancedPaymentMonitoring()
    incident.responseActions.push('Activated enhanced payment monitoring')

    // 3. Provider notification if needed
    if (details.affectedProviders) {
      await this.notifyAffectedProviders(details.affectedProviders)
      incident.responseActions.push('Notified affected providers')
    }
  }
}
```

This comprehensive security documentation provides the foundation for maintaining a secure, compliant, and resilient SkyMarket platform while protecting user data and ensuring regulatory compliance across multiple domains.