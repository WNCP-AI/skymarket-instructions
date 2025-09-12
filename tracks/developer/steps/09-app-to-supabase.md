# Step 09: Connect Your App to Supabase

## Objective
Connect your Next.js application to your Supabase project by configuring environment variables and establishing database connections. This step ensures your app can authenticate users and perform database operations.

## What You'll Learn
- How to securely configure Supabase environment variables
- Understanding the difference between client-side and server-side Supabase clients
- How to verify your database connection is working
- Basic troubleshooting for connection issues

## Prerequisites
- Completed Step 08 (Supabase project created)
- Your Supabase project URL and API keys ready
- Next.js development environment running

## Instructions

### 1. Create Environment Variables File

Create a `.env.local` file in your project root:

```bash
# In your terminal
touch .env.local
```

### 2. Add Supabase Configuration

Open `.env.local` and add your Supabase credentials:

```bash
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here

# App Configuration (Detroit Metro area defaults)
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25
```

**⚠️ Important Security Notes:**
- `NEXT_PUBLIC_*` variables are exposed to the browser - only use for non-sensitive data
- `SUPABASE_SERVICE_ROLE_KEY` should never have the `NEXT_PUBLIC_` prefix
- Never commit `.env.local` to version control

### 3. Verify Supabase Client Configuration

Use Cursor to check your Supabase setup:

**Cursor Prompt:**
```
Scan @lib/supabase/* and confirm env names and SSR usage. If any env is missing, output the exact keys to add to .env.local. Also verify the client configurations are correct for Next.js 15.
```

### 4. Test Your Connection

Start your development server:

```bash
npm run dev
```

Open your browser to `http://localhost:3000` and check for:
- No Supabase-related errors in the browser console
- No connection errors in your terminal

### 5. Create a Simple Connection Test

Create a test page to verify your connection:

**Cursor Prompt:**
```
Create a simple test page at app/test-supabase/page.tsx that:
1. Uses the Supabase client to fetch a simple query
2. Displays connection status
3. Shows any error messages clearly
4. Uses proper TypeScript types
```

### 6. Test Authentication Flow

Create a basic sign-up form to test authentication:

**Cursor Prompt:**
```
Create a basic authentication test at app/auth-test/page.tsx that includes:
1. A sign-up form with email and password
2. A sign-in form
3. A sign-out button
4. Display current user state
5. Handle all error states clearly
6. Use proper form validation
```

## Code Snippets

### Basic Supabase Client Usage

```typescript
// Example: Testing your connection
import { createClient } from '@/lib/supabase/client'

export default async function TestPage() {
  const supabase = createClient()
  
  try {
    const { data, error } = await supabase
      .from('users')
      .select('count')
      .single()
    
    if (error) {
      console.error('Supabase connection error:', error)
      return <div>Connection Error: {error.message}</div>
    }
    
    return <div>✅ Supabase Connected Successfully!</div>
  } catch (err) {
    return <div>❌ Failed to connect to Supabase</div>
  }
}
```

### Environment Variable Validation

```typescript
// lib/config.ts - Validate required environment variables
export const config = {
  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    serviceRoleKey: process.env.SUPABASE_SERVICE_ROLE_KEY!,
  },
  app: {
    url: process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000',
    serviceArea: {
      lat: parseFloat(process.env.NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT || '42.3314'),
      lng: parseFloat(process.env.NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG || '-83.0458'),
      radiusMiles: parseInt(process.env.NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES || '25'),
    }
  }
}

// Validate required variables
if (!config.supabase.url) throw new Error('Missing NEXT_PUBLIC_SUPABASE_URL')
if (!config.supabase.anonKey) throw new Error('Missing NEXT_PUBLIC_SUPABASE_ANON_KEY')
```

## Expected Outcome

After completing this step, you should have:

✅ **Environment Configuration:**
- `.env.local` file with correct Supabase credentials
- All required environment variables set
- Secure separation of public and private keys

✅ **Working Connection:**
- Your app successfully initializes Supabase clients
- No connection errors in browser console or terminal
- Ability to make basic database queries

✅ **Authentication Ready:**
- Basic auth flow implemented and tested
- User signup/signin working
- Proper error handling for auth failures

✅ **Development Setup:**
- Development server running without errors
- Test pages created for validation
- Ready for building core application features

## Troubleshooting Tips

### Common Issues and Solutions

**❌ "Invalid API key" Error**
```bash
# Double-check your .env.local file:
# 1. No extra spaces around the = sign
# 2. Keys copied correctly from Supabase dashboard
# 3. File is named .env.local (not .env)
```

**❌ "Network request failed"**
```bash
# Check your Supabase URL:
# 1. Should start with https://
# 2. Should end with .supabase.co
# 3. Project reference should match your dashboard
```

**❌ Environment variables not loading**
```bash
# Restart your development server
npm run dev

# Verify .env.local is in project root
ls -la .env.local

# Check for typos in variable names
grep -n "SUPABASE" .env.local
```

**❌ TypeScript errors with Supabase client**
```bash
# Regenerate types (if you have existing tables)
npx supabase gen types typescript --project-id YOUR_PROJECT_REF > types/database.ts
```

**❌ CORS errors in browser**
```bash
# Check Supabase Authentication settings:
# 1. Go to Authentication > Settings
# 2. Add your domain to "Site URL"
# 3. Add http://localhost:3000 for development
```

### Testing Checklist

Before moving to the next step, verify:

- [ ] Browser console shows no Supabase errors
- [ ] Can sign up a new user account
- [ ] Can sign in with created account
- [ ] Can sign out successfully
- [ ] User state persists across page refreshes
- [ ] Server-side Supabase client works in API routes
- [ ] Environment variables load correctly

### Getting Help

If you're stuck:
1. **Check Supabase Dashboard:** Verify your project is active and keys are correct
2. **Browser DevTools:** Look for specific error messages in Console tab
3. **Network Tab:** Check if requests to Supabase are being made
4. **Supabase Documentation:** [supabase.com/docs/guides/getting-started/quickstarts/nextjs](https://supabase.com/docs/guides/getting-started/quickstarts/nextjs)

## Next Steps

Once your Supabase connection is working, you'll be ready for:
- **Step 10:** Setting up authentication flows and user management
- **Step 11:** Creating your first database queries and mutations
- **Step 12:** Building user dashboards and role management

---

💡 **Pro Tip:** Keep your `.env.local` file organized with comments. As you add more services (Stripe, Mapbox, etc.), good organization will save you time debugging environment issues later.