# SkyMarket Deployment Documentation

Complete deployment and infrastructure documentation for the SkyMarket drone service marketplace platform.

## Overview

SkyMarket is deployed as a modern serverless application using Vercel for hosting, Supabase for backend services, and multiple third-party integrations. The deployment strategy emphasizes scalability, reliability, and rapid iteration with automated CI/CD pipelines.

**Deployment Architecture**:
- Frontend: Vercel (Next.js 15 with App Router)
- Database: Supabase (PostgreSQL with real-time features)
- Authentication: Supabase Auth
- File Storage: Supabase Storage
- Payments: Stripe
- Email: Resend
- Maps: Mapbox
- AI: OpenAI

## Environment Configuration

### Production Environment Variables

```bash
# Application Configuration
NEXT_PUBLIC_APP_URL=https://skymarket.com
NODE_ENV=production

# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_JWT_SECRET=your-jwt-secret

# Stripe Configuration
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_BOOKING_PRODUCT_ID=prod_...

# Email Configuration
RESEND_API_KEY=re_...

# AI Configuration
OPENAI_API_KEY=sk-...
OPENAI_ORGANIZATION_ID=org-...

# Maps Configuration
NEXT_PUBLIC_MAPBOX_TOKEN=pk....

# Detroit Service Area Configuration
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25

# Monitoring and Analytics
VERCEL_ANALYTICS_ID=prj_...
SENTRY_DSN=https://...@sentry.io/...

# Feature Flags
NEXT_PUBLIC_ENABLE_AI_CHAT=true
NEXT_PUBLIC_ENABLE_REAL_TIME=true
NEXT_PUBLIC_DEBUG_MODE=false
```

### Development Environment

```bash
# Development overrides
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Test keys (use Stripe test keys)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_test_...

# Development features
NEXT_PUBLIC_DEBUG_MODE=true
AI_FALLBACK_ECHO=true  # Enable echo mode for AI without API key
STRIPE_WEBHOOK_DEBUG=true
```

### Environment Validation

```typescript
// Environment validation schema
import { z } from 'zod'

const envSchema = z.object({
  // Application
  NEXT_PUBLIC_APP_URL: z.string().url(),
  NODE_ENV: z.enum(['development', 'test', 'production']),

  // Supabase
  NEXT_PUBLIC_SUPABASE_URL: z.string().url(),
  NEXT_PUBLIC_SUPABASE_ANON_KEY: z.string().min(1),
  SUPABASE_SERVICE_ROLE_KEY: z.string().min(1),

  // Stripe
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().startsWith('pk_'),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),

  // External APIs
  RESEND_API_KEY: z.string().startsWith('re_').optional(),
  OPENAI_API_KEY: z.string().startsWith('sk-').optional(),
  NEXT_PUBLIC_MAPBOX_TOKEN: z.string().startsWith('pk.').optional(),

  // Geographic constraints
  NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT: z.string().regex(/^-?\d+\.?\d*$/),
  NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG: z.string().regex(/^-?\d+\.?\d*$/),
  NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES: z.string().regex(/^\d+$/),
})

// Validate environment on startup
export function validateEnvironment() {
  try {
    envSchema.parse(process.env)
    console.log('✅ Environment validation passed')
  } catch (error) {
    console.error('❌ Environment validation failed:', error)
    process.exit(1)
  }
}

// Call in next.config.js
validateEnvironment()
```

## Vercel Deployment Configuration

### Project Settings

```json
// vercel.json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "functions": {
    "app/api/**/*.ts": {
      "maxDuration": 30
    },
    "app/api/webhooks/**/*.ts": {
      "maxDuration": 60
    }
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, PUT, DELETE, OPTIONS"
        },
        {
          "key": "Access-Control-Allow-Headers",
          "value": "Content-Type, Authorization"
        }
      ]
    }
  ],
  "redirects": [
    {
      "source": "/admin",
      "destination": "/dashboard?tab=admin",
      "permanent": false
    }
  ],
  "rewrites": [
    {
      "source": "/health",
      "destination": "/api/health"
    }
  ]
}
```

### Build Configuration

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Build optimizations
  experimental: {
    ppr: 'incremental', // Partial Prerendering
    reactCompiler: true,
  },

  // Image optimization
  images: {
    domains: [
      'supabase.co',
      'avatars.githubusercontent.com',
      'lh3.googleusercontent.com',
      'res.cloudinary.com'
    ],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },

  // Security headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin'
          },
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-eval' 'unsafe-inline' *.vercel.app *.stripe.com",
              "style-src 'self' 'unsafe-inline' fonts.googleapis.com",
              "img-src 'self' data: blob: *.supabase.co *.stripe.com",
              "font-src 'self' fonts.gstatic.com",
              "connect-src 'self' *.supabase.co *.stripe.com api.openai.com api.mapbox.com",
              "frame-src 'self' *.stripe.com"
            ].join('; ')
          }
        ]
      }
    ]
  },

  // Webpack optimizations
  webpack: (config, { dev, isServer }) => {
    if (!dev && !isServer) {
      // Bundle analyzer in production builds
      if (process.env.ANALYZE === 'true') {
        const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer')
        config.plugins.push(
          new BundleAnalyzerPlugin({
            analyzerMode: 'static',
            openAnalyzer: false,
          })
        )
      }
    }
    return config
  },

  // Environment validation
  env: {
    CUSTOM_BUILD_ID: process.env.VERCEL_GIT_COMMIT_SHA || 'development',
  },

  // Output for serverless deployment
  output: 'standalone',

  // Logging
  logging: {
    fetches: {
      fullUrl: true,
    },
  },
}

module.exports = nextConfig
```

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy to Vercel

on:
  push:
    branches: [main, staging]
  pull_request:
    branches: [main]

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Run type checking
        run: npx tsc --noEmit

      - name: Run tests
        run: npm run test
        env:
          NODE_ENV: test

  build-preview:
    if: github.event_name == 'pull_request'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=preview --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy Project Artifacts to Vercel
        id: deploy
        run: |
          url=$(vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }})
          echo "url=$url" >> $GITHUB_OUTPUT

      - name: Comment PR with preview URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 Preview deployment ready at: ${{ steps.deploy.outputs.url }}`
            })

  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    needs: test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=staging --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy to Staging
        run: vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }} --target=staging

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy to Production
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Run smoke tests
        run: |
          # Wait for deployment to be ready
          sleep 30
          # Run basic health checks
          curl -f https://skymarket.com/api/health || exit 1
```

## Supabase Database Deployment

### Migration Management

```sql
-- Migration file naming convention: YYYYMMDD_HHMM_description.sql
-- Example: 20241201_1200_initial_schema.sql

-- Migration tracking table
CREATE TABLE IF NOT EXISTS _migrations (
  id SERIAL PRIMARY KEY,
  filename TEXT NOT NULL UNIQUE,
  checksum TEXT NOT NULL,
  executed_at TIMESTAMPTZ DEFAULT NOW()
);

-- Migration execution function
CREATE OR REPLACE FUNCTION execute_migration(
  migration_filename TEXT,
  migration_sql TEXT
) RETURNS BOOLEAN AS $$
DECLARE
  migration_checksum TEXT;
BEGIN
  -- Calculate checksum
  migration_checksum := encode(digest(migration_sql, 'sha256'), 'hex');

  -- Check if already executed
  IF EXISTS (
    SELECT 1 FROM _migrations
    WHERE filename = migration_filename
    AND checksum = migration_checksum
  ) THEN
    RAISE NOTICE 'Migration % already executed with same checksum', migration_filename;
    RETURN FALSE;
  END IF;

  -- Execute migration
  EXECUTE migration_sql;

  -- Record migration
  INSERT INTO _migrations (filename, checksum)
  VALUES (migration_filename, migration_checksum)
  ON CONFLICT (filename) DO UPDATE SET
    checksum = migration_checksum,
    executed_at = NOW();

  RAISE NOTICE 'Migration % executed successfully', migration_filename;
  RETURN TRUE;
END;
$$ LANGUAGE plpgsql;
```

### Database Deployment Script

```bash
#!/bin/bash
# deploy-db.sh - Database deployment script

set -e

SUPABASE_PROJECT_ID="your-project-id"
SUPABASE_DB_PASSWORD="your-db-password"

echo "🚀 Starting database deployment..."

# 1. Install Supabase CLI if not present
if ! command -v supabase &> /dev/null; then
    echo "Installing Supabase CLI..."
    npm install -g supabase
fi

# 2. Login to Supabase (requires token)
supabase login

# 3. Link to project
supabase link --project-ref $SUPABASE_PROJECT_ID

# 4. Generate types
echo "📝 Generating TypeScript types..."
supabase gen types typescript --local > types/database.ts

# 5. Run migrations
echo "🔄 Running database migrations..."
supabase db push

# 6. Seed data (production-safe)
if [ "$NODE_ENV" != "production" ]; then
    echo "🌱 Seeding database..."
    supabase db reset --linked
fi

# 7. Verify deployment
echo "✅ Verifying database deployment..."
supabase status

echo "🎉 Database deployment completed!"
```

### RLS Policy Deployment

```sql
-- Row Level Security policy deployment
-- File: supabase/migrations/20241201_1300_rls_policies.sql

-- Enable RLS on all tables
DO $$
DECLARE
    table_name TEXT;
BEGIN
    FOR table_name IN
        SELECT tablename
        FROM pg_tables
        WHERE schemaname = 'public'
        AND tablename NOT LIKE '_%'
    LOOP
        EXECUTE format('ALTER TABLE %I ENABLE ROW LEVEL SECURITY', table_name);
        RAISE NOTICE 'Enabled RLS on table: %', table_name;
    END LOOP;
END $$;

-- Profiles policies
CREATE POLICY IF NOT EXISTS "profiles_select" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY IF NOT EXISTS "profiles_insert" ON profiles
  FOR INSERT WITH CHECK (auth.uid() = id);

CREATE POLICY IF NOT EXISTS "profiles_update" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- Providers policies
CREATE POLICY IF NOT EXISTS "providers_select" ON providers
  FOR SELECT USING (TRUE);

CREATE POLICY IF NOT EXISTS "providers_insert" ON providers
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY IF NOT EXISTS "providers_update" ON providers
  FOR UPDATE USING (auth.uid() = user_id);

-- Listings policies
CREATE POLICY IF NOT EXISTS "listings_select" ON listings
  FOR SELECT USING (
    is_active = TRUE OR
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

CREATE POLICY IF NOT EXISTS "listings_insert" ON listings
  FOR INSERT WITH CHECK (
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

CREATE POLICY IF NOT EXISTS "listings_update" ON listings
  FOR UPDATE USING (
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

-- Bookings policies
CREATE POLICY IF NOT EXISTS "bookings_select" ON bookings
  FOR SELECT USING (
    consumer_id = auth.uid() OR
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

CREATE POLICY IF NOT EXISTS "bookings_insert" ON bookings
  FOR INSERT WITH CHECK (consumer_id = auth.uid());

CREATE POLICY IF NOT EXISTS "bookings_update" ON bookings
  FOR UPDATE USING (
    consumer_id = auth.uid() OR
    provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

-- Reviews policies
CREATE POLICY IF NOT EXISTS "reviews_select" ON reviews
  FOR SELECT USING (TRUE);

CREATE POLICY IF NOT EXISTS "reviews_insert" ON reviews
  FOR INSERT WITH CHECK (
    reviewer_id = auth.uid() AND
    EXISTS (
      SELECT 1 FROM bookings
      WHERE id = booking_id
      AND (
        consumer_id = auth.uid() OR
        provider_id IN (
          SELECT id FROM providers WHERE user_id = auth.uid()
        )
      )
      AND booking_status = 'completed'
    )
  );

-- Messages policies
CREATE POLICY IF NOT EXISTS "messages_select" ON messages
  FOR SELECT USING (
    sender_id = auth.uid() OR receiver_id = auth.uid()
  );

CREATE POLICY IF NOT EXISTS "messages_insert" ON messages
  FOR INSERT WITH CHECK (sender_id = auth.uid());

-- Admin override policy
CREATE POLICY IF NOT EXISTS "admin_full_access" ON profiles
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM profiles
      WHERE id = auth.uid()
      AND user_role = 'admin'
    )
  );
```

## Third-Party Service Configuration

### Stripe Setup

```javascript
// Stripe payment configuration
const stripeConfig = {
  // Production settings
  production: {
    clientId: 'ca_...',
    platformAccountId: 'acct_...',
    webhookEndpoints: [
      {
        url: 'https://skymarket.com/api/webhooks/stripe',
        enabled_events: [
          'checkout.session.completed',
          'payment_intent.succeeded',
          'payment_intent.payment_failed',
          'charge.captured',
          'charge.refunded',
          'account.updated',
          'account.application.deauthorized'
        ]
      }
    ]
  },

  // Development settings
  development: {
    clientId: 'ca_test...',
    platformAccountId: 'acct_test...',
    webhookEndpoints: [
      {
        url: 'https://localhost:3000/api/webhooks/stripe',
        enabled_events: ['*'] // All events for testing
      }
    ]
  }
}

// Stripe CLI for local development
// stripe listen --forward-to localhost:3000/api/webhooks/stripe
```

### Email Service Setup (Resend)

```javascript
// Resend configuration
export const emailConfig = {
  production: {
    domain: 'skymarket.com',
    fromEmail: 'noreply@skymarket.com',
    supportEmail: 'support@skymarket.com',
    templates: {
      bookingConfirmation: 'booking-confirmation-v1',
      paymentReceipt: 'payment-receipt-v1',
      providerNotification: 'provider-notification-v1',
      cancellationNotice: 'cancellation-notice-v1'
    }
  },
  development: {
    domain: 'localhost',
    fromEmail: 'test@localhost',
    supportEmail: 'test@localhost',
    templates: {
      // Use same templates but with test data
    }
  }
}

// Email template deployment
const deployEmailTemplates = async () => {
  const templates = [
    {
      name: 'booking-confirmation-v1',
      subject: 'Booking Confirmation - {{service_title}}',
      html: await fs.readFile('./email-templates/booking-confirmation.html', 'utf8')
    },
    // ... other templates
  ]

  for (const template of templates) {
    await resend.emails.template.create(template)
  }
}
```

## Health Checks and Monitoring

### Health Check Endpoint

```typescript
// app/api/health/route.ts
import { NextResponse } from 'next/server'
import { createClient } from '@/lib/supabase/server'
import { stripe } from '@/lib/stripe/server'

interface HealthCheck {
  status: 'healthy' | 'unhealthy' | 'degraded'
  timestamp: string
  version: string
  services: {
    database: ServiceHealth
    stripe: ServiceHealth
    email: ServiceHealth
    ai: ServiceHealth
  }
}

interface ServiceHealth {
  status: 'up' | 'down' | 'degraded'
  responseTime?: number
  lastCheck: string
  error?: string
}

export async function GET() {
  const startTime = Date.now()
  const health: HealthCheck = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.VERCEL_GIT_COMMIT_SHA || 'development',
    services: {
      database: await checkDatabase(),
      stripe: await checkStripe(),
      email: await checkEmail(),
      ai: await checkAI()
    }
  }

  // Determine overall status
  const serviceStatuses = Object.values(health.services).map(s => s.status)
  if (serviceStatuses.includes('down')) {
    health.status = 'unhealthy'
  } else if (serviceStatuses.includes('degraded')) {
    health.status = 'degraded'
  }

  const responseTime = Date.now() - startTime
  const statusCode = health.status === 'healthy' ? 200 :
                    health.status === 'degraded' ? 200 : 503

  return NextResponse.json(
    { ...health, responseTime },
    { status: statusCode }
  )
}

async function checkDatabase(): Promise<ServiceHealth> {
  const startTime = Date.now()
  try {
    const supabase = await createClient()
    const { error } = await supabase
      .from('profiles')
      .select('id')
      .limit(1)

    if (error) throw error

    return {
      status: 'up',
      responseTime: Date.now() - startTime,
      lastCheck: new Date().toISOString()
    }
  } catch (error) {
    return {
      status: 'down',
      lastCheck: new Date().toISOString(),
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}

async function checkStripe(): Promise<ServiceHealth> {
  const startTime = Date.now()
  try {
    await stripe.accounts.retrieve()
    return {
      status: 'up',
      responseTime: Date.now() - startTime,
      lastCheck: new Date().toISOString()
    }
  } catch (error) {
    return {
      status: 'down',
      lastCheck: new Date().toISOString(),
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}

async function checkEmail(): Promise<ServiceHealth> {
  // Basic API connectivity check for Resend
  const startTime = Date.now()
  try {
    if (!process.env.RESEND_API_KEY) {
      throw new Error('Resend API key not configured')
    }

    // Simple API availability check
    const response = await fetch('https://api.resend.com/domains', {
      headers: {
        'Authorization': `Bearer ${process.env.RESEND_API_KEY}`,
        'Content-Type': 'application/json'
      }
    })

    if (!response.ok) throw new Error(`HTTP ${response.status}`)

    return {
      status: 'up',
      responseTime: Date.now() - startTime,
      lastCheck: new Date().toISOString()
    }
  } catch (error) {
    return {
      status: 'down',
      lastCheck: new Date().toISOString(),
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}

async function checkAI(): Promise<ServiceHealth> {
  const startTime = Date.now()
  try {
    if (!process.env.OPENAI_API_KEY) {
      return {
        status: 'degraded',
        lastCheck: new Date().toISOString(),
        error: 'OpenAI API key not configured - echo mode active'
      }
    }

    // Basic connectivity test
    const response = await fetch('https://api.openai.com/v1/models', {
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json'
      }
    })

    if (!response.ok) throw new Error(`HTTP ${response.status}`)

    return {
      status: 'up',
      responseTime: Date.now() - startTime,
      lastCheck: new Date().toISOString()
    }
  } catch (error) {
    return {
      status: 'down',
      lastCheck: new Date().toISOString(),
      error: error instanceof Error ? error.message : 'Unknown error'
    }
  }
}
```

### Monitoring Setup

```typescript
// Vercel Analytics and monitoring
export const monitoringConfig = {
  vercel: {
    analytics: true,
    webVitals: true,
    speedInsights: true
  },

  customMetrics: {
    // Business metrics
    bookingsCreated: 'increment',
    paymentsProcessed: 'increment',
    userRegistrations: 'increment',

    // Performance metrics
    apiResponseTime: 'histogram',
    databaseQueryTime: 'histogram',
    paymentProcessingTime: 'histogram',

    // Error tracking
    apiErrors: 'increment',
    paymentFailures: 'increment',
    authenticationFailures: 'increment'
  }
}

// Custom monitoring implementation
export async function trackMetric(
  metric: string,
  value: number,
  tags?: Record<string, string>
) {
  // Send to Vercel Analytics
  if (typeof window !== 'undefined' && window.va) {
    window.va.track(metric, { value, ...tags })
  }

  // Log for internal analysis
  console.log(`📊 Metric: ${metric}`, { value, tags, timestamp: new Date().toISOString() })

  // Could integrate with other monitoring services:
  // - Datadog
  // - New Relic
  // - Sentry Performance
}
```

## Backup and Disaster Recovery

### Database Backup Strategy

```bash
#!/bin/bash
# backup-database.sh - Automated database backup

set -e

SUPABASE_PROJECT_ID="your-project-id"
BACKUP_BUCKET="skymarket-backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "🗄️ Starting database backup..."

# 1. Create database dump
supabase db dump --linked > "backup_${TIMESTAMP}.sql"

# 2. Compress backup
gzip "backup_${TIMESTAMP}.sql"

# 3. Upload to cloud storage (example with AWS S3)
aws s3 cp "backup_${TIMESTAMP}.sql.gz" "s3://${BACKUP_BUCKET}/daily/" --storage-class STANDARD_IA

# 4. Verify backup integrity
aws s3 ls "s3://${BACKUP_BUCKET}/daily/backup_${TIMESTAMP}.sql.gz"

# 5. Cleanup local files
rm "backup_${TIMESTAMP}.sql.gz"

# 6. Cleanup old backups (keep 30 days)
CUTOFF_DATE=$(date -d '30 days ago' +%Y%m%d)
aws s3 ls "s3://${BACKUP_BUCKET}/daily/" --recursive | while read -r line; do
    backup_date=$(echo $line | awk '{print $4}' | grep -oP 'backup_\K\d{8}' || echo "")
    if [[ "$backup_date" < "$CUTOFF_DATE" ]]; then
        backup_file=$(echo $line | awk '{print $4}')
        aws s3 rm "s3://${BACKUP_BUCKET}/daily/$backup_file"
        echo "🗑️ Deleted old backup: $backup_file"
    fi
done

echo "✅ Database backup completed: backup_${TIMESTAMP}.sql.gz"
```

### Disaster Recovery Plan

```yaml
# disaster-recovery.yml - Disaster recovery procedures
recovery_plan:
  rto: 4 hours  # Recovery Time Objective
  rpo: 1 hour   # Recovery Point Objective

  scenarios:
    database_failure:
      severity: critical
      steps:
        1. "Verify Supabase status dashboard"
        2. "Switch to read-only mode if partial failure"
        3. "Restore from latest backup if total failure"
        4. "Update DNS if failover required"
        5. "Notify users of estimated recovery time"

    vercel_outage:
      severity: high
      steps:
        1. "Deploy to backup hosting provider (Railway/Render)"
        2. "Update DNS records"
        3. "Verify all integrations work"
        4. "Monitor for issues"

    payment_system_failure:
      severity: high
      steps:
        1. "Enable maintenance mode for checkout"
        2. "Notify active users"
        3. "Process pending payments manually if needed"
        4. "Resume automated processing when resolved"

  contact_tree:
    - primary: "Technical Lead"
    - secondary: "DevOps Engineer"
    - escalation: "CTO"
    - communications: "Support Team"
```

## Performance Optimization

### Build Optimization

```javascript
// Performance optimization configuration
const performanceConfig = {
  // Bundle optimization
  webpack: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
          enforce: true,
        },
        common: {
          name: 'common',
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
        }
      }
    }
  },

  // Image optimization
  images: {
    formats: ['image/avif', 'image/webp'],
    minimumCacheTTL: 31536000, // 1 year
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },

  // Caching strategy
  headers: [
    {
      source: '/static/(.*)',
      headers: [
        {
          key: 'Cache-Control',
          value: 'public, max-age=31536000, immutable'
        }
      ]
    },
    {
      source: '/api/(.*)',
      headers: [
        {
          key: 'Cache-Control',
          value: 'no-store, must-revalidate'
        }
      ]
    }
  ]
}
```

This comprehensive deployment documentation provides everything needed to deploy, monitor, and maintain the SkyMarket platform across all environments while ensuring reliability, security, and optimal performance.