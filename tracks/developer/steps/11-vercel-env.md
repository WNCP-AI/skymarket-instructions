# Step 11: Vercel Account & Environment Variables

## Objective
Import your SkyMarket repository to Vercel and configure all required environment variables for production deployment. This step ensures your app has all the necessary API keys and configurations to run in production.

## What You'll Learn
- How to import a Git repository to Vercel
- The difference between development and production environment variables
- How to securely configure API keys and secrets
- How to update environment variables after deployment

## Prerequisites
- Completed Step 10 (Understanding Vercel basics)
- A Vercel account (free tier available)
- Your SkyMarket repository pushed to GitHub/GitLab/Bitbucket
- API keys from all services (Supabase, Stripe, etc.)

## Understanding Environment Variables on Vercel

### Development vs Production
```bash
# Development (.env.local) - Local only
NEXT_PUBLIC_SUPABASE_URL=https://abc123.supabase.co

# Production (Vercel Dashboard) - Deployed app
NEXT_PUBLIC_SUPABASE_URL=https://abc123.supabase.co
```

**Key Differences:**
- **Local:** `.env.local` file in your project
- **Production:** Vercel dashboard configuration
- **Security:** Production variables are encrypted and secure
- **Scope:** Can be set per environment (Preview, Production)

## Instructions

### 1. Create/Access Your Vercel Account

1. **Sign up for Vercel:** Go to [vercel.com](https://vercel.com)
2. **Choose sign-in method:** GitHub, GitLab, or Bitbucket (recommended)
3. **Verify your account:** Complete any required verification steps

### 2. Import Your Repository

**Step-by-Step Import Process:**

1. **Click "Add New..."** in your Vercel dashboard
2. **Select "Project"** from the dropdown
3. **Import Git Repository:** Find and select your SkyMarket repo
4. **Configure Project:**
   - **Framework Preset:** Next.js (should auto-detect)
   - **Root Directory:** Leave as `.` (root)
   - **Build Command:** `npm run build` (default)
   - **Output Directory:** `.next` (default)

### 3. Configure Environment Variables

During import, you'll see an "Environment Variables" section. Add these variables:

**Cursor Prompt:**
```
Generate a one-screen checklist of envs grouped by provider with brief purpose for each, based on scanning process.env usage. Include all variables needed for SkyMarket to function properly.
```

## Required Environment Variables

### 🗄️ Supabase (Database & Auth)
```bash
# Public variables (exposed to browser)
NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key_here

# Private variables (server-only)
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_here
```

**Purpose:**
- `NEXT_PUBLIC_SUPABASE_URL`: Supabase project endpoint
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Public key for client-side operations
- `SUPABASE_SERVICE_ROLE_KEY`: Private key for admin operations

### 💳 Stripe (Payments)
```bash
# Public variables
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...

# Private variables
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Optional
STRIPE_BOOKING_PRODUCT_ID=prod_...
```

**Purpose:**
- `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`: Client-side Stripe integration
- `STRIPE_SECRET_KEY`: Server-side payment processing
- `STRIPE_WEBHOOK_SECRET`: Webhook signature verification
- `STRIPE_BOOKING_PRODUCT_ID`: Default product for bookings

### 📧 Resend (Email Service)
```bash
# Private variables
RESEND_API_KEY=re_...
RESEND_FROM_EMAIL=noreply@yourdomain.com
```

**Purpose:**
- `RESEND_API_KEY`: Email service authentication
- `RESEND_FROM_EMAIL`: Default sender email address

### 🤖 OpenAI (AI Features)
```bash
# Private variables
OPENAI_API_KEY=sk-...
```

**Purpose:**
- `OPENAI_API_KEY`: AI content generation and chat features

### 🗺️ Mapbox (Maps & Geocoding)
```bash
# Public variables
NEXT_PUBLIC_MAPBOX_TOKEN=pk.eyJ1...
```

**Purpose:**
- `NEXT_PUBLIC_MAPBOX_TOKEN`: Map rendering and geocoding

### 🏙️ App Configuration (Detroit Metro)
```bash
# Public variables
NEXT_PUBLIC_APP_URL=https://your-app.vercel.app
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25
```

**Purpose:**
- `NEXT_PUBLIC_APP_URL`: Your production domain
- Service area coordinates for Detroit Metro area validation

## Step-by-Step Configuration

### 1. During Initial Import

When importing your repo, Vercel will show an "Environment Variables" screen:

1. **Add each variable:** Name and Value
2. **Select environments:** Production, Preview, Development
3. **Verify sensitive variables:** Don't expose secrets as `NEXT_PUBLIC_`

### 2. After Deployment

To add or update variables after deployment:

1. **Go to your project dashboard**
2. **Click "Settings" tab**
3. **Select "Environment Variables"**
4. **Add/Edit variables**
5. **Redeploy your app** (automatic or manual)

### 3. Webhook Configuration (Important!)

**Stripe webhooks require special setup:**

1. **Deploy your app first** (to get your production URL)
2. **Go to Stripe Dashboard** → Developers → Webhooks
3. **Create new webhook endpoint:**
   ```
   https://your-app.vercel.app/api/webhooks/stripe
   ```
4. **Select events to send:**
   - `payment_intent.succeeded`
   - `payment_intent.payment_failed`
   - `customer.subscription.updated`
5. **Copy webhook signing secret**
6. **Add to Vercel:** `STRIPE_WEBHOOK_SECRET=whsec_...`
7. **Redeploy your app**

## Advanced Configuration

### Environment-Specific Values

**Cursor Prompt:**
```
Scan process.env usage across @lib and /app/api/* and output a table of (KEY, file usage, purpose, required) with recommendations for Development vs Production values.
```

### Variable Scoping

**Production Only:**
- Live API keys
- Production database URLs
- Real webhook secrets

**Preview & Development:**
- Test API keys
- Development database URLs
- Test webhook endpoints

## Expected Outcome

After completing this step, you should have:

✅ **Vercel Project Setup:**
- Repository successfully imported to Vercel
- Automatic deployments configured with Git
- Build and deployment settings optimized

✅ **Environment Variables Configured:**
- All required variables added to Vercel dashboard
- Proper separation of public vs private variables
- Environment-specific configurations set

✅ **Webhook Integration:**
- Stripe webhook endpoint configured
- Webhook signing secret properly set
- Test webhook delivery working

✅ **Production Deployment:**
- App successfully deployed to production
- All integrations working (database, payments, etc.)
- No environment-related errors in logs

## Configuration Checklist

### Pre-Deployment Checklist
- [ ] All API keys obtained and tested locally
- [ ] Repository pushed to Git provider
- [ ] Environment variables documented
- [ ] Webhook endpoints identified

### During Import Checklist
- [ ] Repository imported successfully
- [ ] Framework detected as Next.js
- [ ] Build settings are correct
- [ ] Initial environment variables added

### Post-Deployment Checklist
- [ ] App deploys without errors
- [ ] Database connections working
- [ ] Authentication flow functional
- [ ] Payment integration tested
- [ ] Email sending operational
- [ ] Map integration working
- [ ] Webhook endpoints responding

## Troubleshooting Tips

### Common Issues and Solutions

**❌ "Missing environment variable" errors**
```bash
# Check variable names for typos
# Ensure NEXT_PUBLIC_ prefix for client-side variables
# Verify all required variables are added in Vercel dashboard
```

**❌ Build fails during deployment**
```bash
# Check build logs in Vercel dashboard
# Verify all dependencies are in package.json
# Ensure TypeScript errors are resolved
```

**❌ Webhook signature verification fails**
```bash
# Confirm webhook endpoint URL is correct
# Verify STRIPE_WEBHOOK_SECRET matches Stripe dashboard
# Check that webhook is configured for correct events
```

**❌ Database connection fails**
```bash
# Verify Supabase URL and keys are correct
# Check Supabase project is active
# Confirm RLS policies allow operations
```

**❌ Map not loading**
```bash
# Verify Mapbox token is valid and public
# Check domain restrictions in Mapbox account
# Ensure token has required permissions
```

### Environment Variable Validation

**Add this to your app for debugging:**

```typescript
// lib/validate-env.ts
export function validateEnvironment() {
  const required = {
    'NEXT_PUBLIC_SUPABASE_URL': process.env.NEXT_PUBLIC_SUPABASE_URL,
    'NEXT_PUBLIC_SUPABASE_ANON_KEY': process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
    'SUPABASE_SERVICE_ROLE_KEY': process.env.SUPABASE_SERVICE_ROLE_KEY,
    'NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY': process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY,
    'STRIPE_SECRET_KEY': process.env.STRIPE_SECRET_KEY,
  }
  
  const missing = Object.entries(required)
    .filter(([key, value]) => !value)
    .map(([key]) => key)
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`)
  }
}
```

### Quick Test After Deployment

**Create a simple health check:**

```typescript
// app/api/health/route.ts
import { NextResponse } from 'next/server'

export const runtime = 'edge'

export async function GET() {
  const checks = {
    supabase_url: !!process.env.NEXT_PUBLIC_SUPABASE_URL,
    stripe_key: !!process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY,
    mapbox_token: !!process.env.NEXT_PUBLIC_MAPBOX_TOKEN,
    timestamp: new Date().toISOString(),
  }
  
  return NextResponse.json(checks)
}
```

Visit `https://your-app.vercel.app/api/health` to verify configuration.

## Security Best Practices

### ✅ Do's
- Use `NEXT_PUBLIC_` prefix only for client-safe variables
- Keep sensitive keys in server-only variables
- Use different API keys for development vs production
- Regularly rotate API keys
- Monitor usage and set up alerts

### ❌ Don'ts
- Never commit `.env.local` to version control
- Don't use production keys in development
- Don't expose sensitive data in `NEXT_PUBLIC_` variables
- Don't share API keys in plain text
- Don't use the same webhook secret across environments

## Next Steps

With your environment properly configured, you're ready to:
- **Step 12:** Use Context7 MCP for enhanced development workflow
- **Step 13:** Implement Stripe checkout and payment processing
- **Step 14:** Set up email notifications with Resend
- **Step 15:** Add advanced features like real-time updates

---

💡 **Pro Tip:** Keep a secure document with all your API keys and their purposes. This makes it easier to update configurations and onboard team members later.