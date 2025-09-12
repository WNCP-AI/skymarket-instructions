# Step 10: Deploy to Vercel - Basics

## Objective
Understand how Next.js applications work on Vercel's serverless platform and prepare your SkyMarket app for deployment. Learn about function runtimes, edge configurations, and deployment best practices.

## What You'll Learn
- How Vercel's serverless functions work with Next.js
- The difference between Edge Runtime and Node.js Runtime
- How to configure API routes for production deployment
- Vercel's automatic deployment pipeline with Git integration

## Prerequisites
- Your SkyMarket app connected to Supabase (Step 09 completed)
- Basic understanding of API routes and webhooks
- Git repository set up for your project

## Understanding Vercel + Next.js

### The Vercel Advantage for Next.js
Vercel was created by the same team that builds Next.js, making it the optimal hosting platform:

- **Zero Configuration:** Next.js apps deploy instantly
- **Automatic Optimization:** Built-in performance optimizations
- **Edge Network:** Global CDN with 100+ locations worldwide
- **Serverless Functions:** API routes become serverless functions automatically
- **Preview Deployments:** Every push gets a unique preview URL

### How It Works
```
Your Code → Git Push → Vercel Build → Global Edge Network
```

1. You push code to your Git repository
2. Vercel automatically builds your Next.js app
3. Static pages are distributed globally via CDN
4. API routes become serverless functions
5. Your app is instantly available worldwide

## Instructions

### 1. Read Vercel Documentation

First, understand the fundamentals:

**📖 Required Reading:** [Next.js on Vercel](https://vercel.com/docs/frameworks/nextjs)

**Key Concepts to Understand:**
- **Static Generation:** Pages built at deploy time
- **Server-Side Rendering:** Pages rendered on-demand
- **API Routes:** Become serverless functions
- **Edge Functions:** Ultra-fast, lightweight functions
- **Build Process:** Automatic optimization and deployment

### 2. Understand Function Runtimes

Vercel offers two runtime options for your API routes:

#### Edge Runtime (Default)
```javascript
export const runtime = 'edge'
```
- **Ultra-fast cold starts** (~10ms)
- **Limited APIs** (no Node.js APIs)
- **Best for:** Simple API routes, middleware
- **Geographic distribution:** Runs at edge locations

#### Node.js Runtime
```javascript
export const runtime = 'nodejs'
```
- **Full Node.js compatibility** 
- **Slower cold starts** (~100ms)
- **Best for:** Complex logic, database connections, webhooks
- **Central execution:** Runs in specific regions

### 3. Configure Your API Routes for Production

Check your existing API routes and configure them properly:

**Cursor Prompt:**
```
Check /app/api/webhooks/stripe/route.ts and confirm runtime and dynamic config are set for Vercel. If the file doesn't exist, create it with proper Stripe webhook configuration. Include:
1. export const runtime = 'nodejs' 
2. export const dynamic = 'force-dynamic'
3. Basic webhook handler structure
4. Proper error handling
Also check any other API routes in /app/api/ and suggest runtime configurations.
```

### 4. Review Build Configuration

Verify your Next.js configuration is Vercel-ready:

**Cursor Prompt:**
```
Review next.config.ts and suggest any Vercel-specific optimizations. Check for:
1. Image optimization settings
2. Build output configuration  
3. Any custom webpack configs that might affect deployment
4. Environment variable requirements
```

### 5. Prepare for Environment Variables

Understand how environment variables work on Vercel:

**Development vs Production:**
```bash
# Development (.env.local)
NEXT_PUBLIC_SUPABASE_URL=https://abc123.supabase.co
SUPABASE_SERVICE_ROLE_KEY=secret_key_here

# Production (Vercel Dashboard)
# These will be configured in Step 11
```

## Code Examples

### Proper API Route Configuration

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server'

// Required for Vercel deployment
export const runtime = 'nodejs'        // Use Node.js for webhook processing
export const dynamic = 'force-dynamic' // Disable static optimization

export async function POST(request: NextRequest) {
  try {
    // Webhook processing logic here
    const body = await request.text() // Raw body needed for signature verification
    
    // Process Stripe webhook
    // ... webhook logic
    
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

### API Route Runtime Selection Guide

```typescript
// Use Edge Runtime for simple operations
// app/api/health/route.ts
export const runtime = 'edge'

export async function GET() {
  return new Response(JSON.stringify({ status: 'healthy' }), {
    headers: { 'content-type': 'application/json' }
  })
}

// Use Node.js Runtime for complex operations
// app/api/bookings/route.ts
export const runtime = 'nodejs'
export const dynamic = 'force-dynamic'

import { createClient } from '@/lib/supabase/server'

export async function POST(request: NextRequest) {
  const supabase = createClient()
  // Complex database operations, payment processing, etc.
}
```

### Next.js Configuration for Vercel

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Vercel optimizations
  experimental: {
    // Enable experimental features if needed
    serverComponentsExternalPackages: ['@supabase/supabase-js']
  },
  
  // Image optimization (Vercel handles this automatically)
  images: {
    domains: ['your-project-ref.supabase.co'], // Add Supabase storage domain if using
  },
  
  // Optional: Configure redirects
  async redirects() {
    return [
      {
        source: '/home',
        destination: '/',
        permanent: true,
      },
    ]
  },
};

export default nextConfig;
```

## Expected Outcome

After completing this step, you should understand:

✅ **Vercel Deployment Model:**
- How Next.js apps are optimized for Vercel
- The difference between static generation and server-side rendering
- How API routes become serverless functions

✅ **Function Runtime Configuration:**
- When to use Edge Runtime vs Node.js Runtime
- How to configure API routes for production
- Proper webhook configuration for payment processing

✅ **Build Preparation:**
- Your Next.js config is Vercel-ready
- API routes are properly configured
- Understanding of how environment variables work

✅ **Performance Considerations:**
- Global CDN distribution for static assets
- Serverless function cold start implications
- Edge location optimization strategies

## Key Configuration Requirements

### For Webhook Routes (Stripe, etc.)
```typescript
export const runtime = 'nodejs'        // Required for raw request body
export const dynamic = 'force-dynamic' // Prevents static optimization
```

**Why this configuration?**
- **Raw Body Access:** Webhooks need to verify signatures using the raw request body
- **Node.js APIs:** Webhook processing often requires full Node.js capabilities
- **Dynamic Rendering:** Webhooks must process each request individually

### For Simple API Routes
```typescript
export const runtime = 'edge' // Default - ultra-fast
```

**Why Edge Runtime?**
- **Speed:** ~10ms cold start vs ~100ms for Node.js
- **Global Distribution:** Runs at edge locations near users
- **Cost Effective:** More efficient resource usage

## Troubleshooting Tips

### Common Deployment Issues

**❌ "Runtime is not supported"**
```bash
# Check your API route configuration:
# - Make sure you're using 'nodejs' or 'edge' (not 'node')
# - Verify the export syntax is correct
```

**❌ Build failures with dependencies**
```bash
# Some packages don't work with Edge Runtime
# Move them to Node.js runtime or use alternatives
export const runtime = 'nodejs' // If using problematic packages
```

**❌ Environment variable issues**
```bash
# Remember: Vercel needs environment variables configured separately
# .env.local only works in development
```

**❌ Function timeout issues**
```bash
# Vercel has timeout limits:
# - Hobby plan: 10 seconds
# - Pro plan: 15 seconds  
# - Enterprise: 900 seconds
```

### Performance Optimization Tips

1. **Use Edge Runtime when possible** for maximum speed
2. **Minimize serverless function size** by keeping API routes focused
3. **Leverage static generation** for pages that don't change often
4. **Optimize images** using Next.js Image component
5. **Configure caching headers** for API responses where appropriate

## Deep Dive: Why These Configurations Matter

### Webhook Routes Configuration Explained

```typescript
export const runtime = 'nodejs'
export const dynamic = 'force-dynamic'
```

**Why Node.js Runtime?**
- Stripe webhooks require access to the raw request body for signature verification
- Edge Runtime has limitations with certain Node.js APIs
- Webhook processing often involves complex business logic

**Why Force Dynamic?**
- Webhooks must be processed individually (no caching)
- Each webhook payload is unique and time-sensitive
- Prevents Vercel from trying to optimize these routes statically

## Next Steps

You're now ready for the actual deployment process:
- **Step 11:** Configure environment variables in Vercel dashboard
- **Step 12:** Connect your Git repository and deploy
- **Step 13:** Set up custom domains and production configuration

## References

### Essential Reading
- [Next.js on Vercel](https://vercel.com/docs/frameworks/nextjs) - Complete deployment guide
- [Vercel Functions Runtime](https://vercel.com/docs/functions/runtimes) - Runtime options explained
- [Vercel Environment Variables](https://vercel.com/docs/concepts/projects/environment-variables) - Production configuration

### Advanced Topics
- [Edge Runtime](https://vercel.com/docs/functions/edge-functions) - Ultra-fast serverless functions
- [Build Step](https://vercel.com/docs/concepts/builds/build-step) - Understanding the build process
- [Preview Deployments](https://vercel.com/docs/concepts/deployments/preview-deployments) - Testing before production

---

💡 **Pro Tip:** Always test your API routes locally before deployment. Use `vercel dev` command to simulate the Vercel environment on your local machine and catch configuration issues early.