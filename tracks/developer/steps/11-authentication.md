# Step 11: Authentication System

> **Phase 3: Database & Authentication - Complete Auth Implementation**
>
> Build a comprehensive authentication system with Supabase Auth, including signup/login flows, email verification, password reset, role-based access control, and authentication middleware.

## Objective

Learn to implement a complete authentication system by combining the AUTHENTICATION.md specification with official Supabase Auth documentation. You'll build all authentication flows, middleware protection, user profile management, and role-based dashboard routing.

**What you'll build:**
- Complete email/password authentication with verification
- Social authentication integration (GitHub, Google) - configured but disabled
- Password reset flow with secure email handling
- Authentication middleware for route protection
- Role-based access control and dashboard routing
- Profile management with automatic creation triggers
- Authentication utilities for server and client components

## Context Setup

Load these specifications and project documentation into Cursor:

```
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/client-server/CLIENT-SERVER.md
@AGENTS.md
```

**Key points from AUTHENTICATION.md:**
- Server-first authentication with cookie-based sessions
- Email verification gate on login
- Multi-role system (consumer, provider, admin)
- Automatic profile creation on signup
- Comprehensive error handling and user feedback

**Key points from CLIENT-SERVER.md:**
- Server Components for authentication checks by default
- Client Components only for interactive forms
- Server Actions for form handling with built-in security

## Enhanced Context7 Integration

**Powerful Authentication Prompts:**

```
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/architecture/DATABASE.md
@AGENTS.md

Implement comprehensive Supabase authentication system for SkyMarket marketplace:
- Email/password signup with automated profile creation for consumer/provider roles
- Email verification flow with custom templates and error handling
- Multi-role authentication (consumer, provider, admin) with proper middleware protection
- Server-side session management using Next.js 15 App Router patterns
- Social authentication (Google, Apple) with role assignment workflow
- Password reset functionality with secure token handling
- TypeScript integration with generated database types and proper error handling

use context7
```

**Context7 provides:**
- Current Supabase Auth patterns and best practices
- Next.js 15 App Router integration with server/client components
- Middleware configuration for route protection and session management
- Email verification and social auth implementation patterns

## Implementation Prompts

Use these prompts with the loaded context and Context7 documentation:

### 1. Core Authentication Setup

```
Using the Supabase authentication documentation from Context7 and our AUTHENTICATION.md specification, create the core authentication setup:

1. Server and client Supabase configurations with TypeScript types from our database
2. Authentication middleware that handles session updates on all routes
3. Authentication utilities (getUser, requireAuth, getProfile, requireProvider)
4. Email configuration for verification and password reset

Follow the patterns in CLIENT-SERVER.md for server-first architecture and the security requirements in AUTHENTICATION.md.
```

### 2. Authentication Pages

```
Using the authentication flows from AUTHENTICATION.md and Context7 Supabase patterns, create all authentication pages:

1. Login page with email/password only
2. Signup page with email verification flow and profile creation
3. Email confirmation page with supabase functionality
4. Password reset request and confirmation pages
5. All with proper error handling, loading states, and user feedback

Use Client Components for forms but Server Components for layout and static content.
```

### 3. Route Protection and Middleware

```
Create the authentication middleware and route protection system:

1. Middleware that updates sessions on every request
2. Authentication guards for protected routes
3. Role-based redirects (consumer vs provider dashboards)
4. Email verification gates and onboarding flows
5. Automatic profile creation trigger integration

Follow the security patterns in AUTHENTICATION.md and use Server Components for authentication checks where possible.
```

### 4. Profile Management and Role System

```
Implement the profile and role management system:

1. Profile management utilities with role-based access
2. Provider onboarding flow that switches roles
3. Role-based navigation and UI components
4. User profile update functionality
5. Integration with the database schema from Step 10

Use the multi-role system and business rules from AUTHENTICATION.md.
```

## Code Examples

After running the prompts, you should generate:

### Authentication Configuration
```typescript
// lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/database'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (cookies) => cookies.forEach(({ name, value, options }) =>
          cookieStore.set(name, value, options)
        ),
      },
    }
  )
}

// lib/supabase/client.ts
import { createBrowserClient } from '@supabase/ssr'
import type { Database } from '@/types/database'

export function createClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### Authentication Utilities
```typescript
// lib/supabase/auth.ts
import { redirect } from 'next/navigation'
import { createClient } from './server'

export async function getUser() {
  const supabase = await createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  return error || !user ? null : user
}

export async function requireAuth() {
  const user = await getUser()
  if (!user) redirect('/auth/login')
  return user
}

export async function getProfile() {
  const supabase = await createClient()
  const user = await getUser()
  if (!user) return null

  const { data: profile } = await supabase
    .from('profiles')
    .select('*')
    .eq('id', user.id)
    .single()

  return profile
}

export async function requireProvider() {
  const user = await requireAuth()
  const supabase = await createClient()

  const { data: provider } = await supabase
    .from('providers')
    .select('*')
    .eq('user_id', user.id)
    .single()

  if (!provider) redirect('/provider/onboarding')
  return provider
}
```

### Authentication Middleware
```typescript
// middleware.ts
import { updateSession } from '@/lib/supabase/middleware'

export async function middleware(request: NextRequest) {
  return await updateSession(request)
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}

// lib/supabase/middleware.ts
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function updateSession(request: NextRequest) {
  let supabaseResponse = NextResponse.next({
    request,
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => request.cookies.getAll(),
        setAll: (cookies) => {
          cookies.forEach(({ name, value, options }) => request.cookies.set(name, value))
          supabaseResponse = NextResponse.next({
            request,
          })
          cookies.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  // Refresh session if expired
  await supabase.auth.getUser()

  return supabaseResponse
}
```

### Authentication Pages
```typescript
// app/auth/login/page.tsx
"use client"

import { useState, useTransition } from "react"
import { useRouter } from "next/navigation"
import { createClient } from "@/lib/supabase/client"
import { toast } from "sonner"

export default function LoginPage() {
  const router = useRouter()
  const supabase = createClient()
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [isPending, startTransition] = useTransition()

  async function onSubmit(e: React.FormEvent) {
    e.preventDefault()
    startTransition(async () => {
      const { data, error } = await supabase.auth.signInWithPassword({
        email,
        password
      })

      if (error) {
        toast.error(error.message)
        return
      }

      // Email confirmation gate
      const emailConfirmed = !!data.user?.email_confirmed_at
      if (!emailConfirmed) {
        toast.message("Check your email to confirm your account")
        router.push("/auth/confirm")
        return
      }

      toast.success("Signed in")
      router.push("/dashboard")
      router.refresh()
    })
  }

  return (
    <div className="space-y-6">
      <div className="space-y-2 text-center">
        <h1 className="text-3xl font-bold">Sign In</h1>
        <p className="text-muted-foreground">
          Enter your email below to sign in to your account
        </p>
      </div>

      <form onSubmit={onSubmit} className="space-y-4">
        <div className="space-y-2">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
            className="w-full p-2 border rounded"
          />
        </div>
        <div className="space-y-2">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
            className="w-full p-2 border rounded"
          />
        </div>
        <button
          type="submit"
          disabled={isPending}
          className="w-full bg-blue-500 text-white p-2 rounded disabled:opacity-50"
        >
          {isPending ? "Signing in..." : "Sign In"}
        </button>
      </form>

      <div className="text-center">
        <a href="/auth/reset" className="text-blue-500 hover:underline">
          Forgot password?
        </a>
      </div>
    </div>
  )
}
```

### Site Header with Authentication
```typescript
// components/layout/SiteHeader.tsx
import Link from "next/link"
import { createClient } from "@/lib/supabase/server"
import { Button } from "@/components/ui/button"
import { ViewSwitcher } from "./ViewSwitcher"

export default async function SiteHeader() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  let role: "consumer" | "provider" | "admin" | null = null
  if (user) {
    const { data: profile } = await supabase
      .from("profiles")
      .select("role")
      .eq("id", user.id)
      .single()
    role = profile?.role ?? null
  }

  return (
    <header className="border-b">
      <div className="container mx-auto px-4 py-4 flex items-center justify-between">
        <Link href="/" className="text-2xl font-bold">🚁 SkyMarket</Link>

        <nav className="hidden md:flex items-center gap-6">
          <Link href="/browse">Browse Services</Link>
          <Link href="/how-it-works">How It Works</Link>
          <Link href="/provider/apply">Become a Provider</Link>
        </nav>

        <div className="flex items-center gap-3">
          {!user ? (
            <>
              <Link href="/auth/login">
                <Button variant="outline">Sign In</Button>
              </Link>
              <Link href="/auth/signup">
                <Button>Get Started</Button>
              </Link>
            </>
          ) : (
            <>
              <Link href="/dashboard">
                <Button variant="outline">Dashboard</Button>
              </Link>
              <ViewSwitcher role={role} />
              <form action="/auth/logout" method="post">
                <Button variant="ghost" type="submit">Sign Out</Button>
              </form>
            </>
          )}
        </div>
      </div>
    </header>
  )
}
```

## Verification Steps

### 1. Authentication Flow Testing
```bash
# Start development server
npm run dev

# Test authentication flows:
# 1. Visit http://localhost:3000/auth/signup
# 2. Create account and verify email requirement
# 3. Check email for verification link
# 4. Test login with unverified account (should redirect to confirm)
# 5. Verify account and test successful login
# 6. Test password reset flow
```

### 2. Route Protection Verification
```typescript
// Test protected routes
// Visit /dashboard without being logged in - should redirect to login
// Visit /provider/dashboard without provider role - should redirect to onboarding
// Test middleware updates sessions correctly
```

### 3. Role System Testing
```bash
# Test role-based access:
# 1. Sign up as consumer (default role)
# 2. Access consumer dashboard
# 3. Apply to be provider (role switch)
# 4. Access provider dashboard
# 5. Test ViewSwitcher component
```

### 4. Database Integration Check
```sql
-- Verify profile creation trigger works
SELECT * FROM profiles WHERE email = 'test@example.com';

-- Check that roles are assigned correctly
SELECT id, email, role, full_name FROM profiles;

-- Verify provider records are created properly
SELECT * FROM providers WHERE user_id IN (SELECT id FROM profiles WHERE email = 'test@example.com');
```

## Expected Outcome

After completing this step, you'll have:

✅ **Complete Authentication System**: Signup, login, verification, password reset
✅ **Session Management**: Secure cookie-based sessions with automatic refresh
✅ **Route Protection**: Middleware that protects routes and handles auth state
✅ **Role-Based Access**: Multi-role system with consumer, provider, admin roles
✅ **Profile Management**: Automatic profile creation and role switching
✅ **Error Handling**: Comprehensive error handling with user-friendly messages
✅ **Email Verification**: Secure email verification with confirmation gates

**Key achievements:**
- Production-ready authentication with email verification
- Server-first architecture with selective client-side interactivity
- Multi-role system supporting marketplace business model
- Automatic profile creation integrated with database triggers
- Secure session management with HTTP-only cookies
- Role-based navigation and dashboard routing

## Troubleshooting

### Common Issues

**Email Verification Problems:**
- Check Supabase email configuration in dashboard
- Verify redirect URLs are configured correctly
- Ensure confirmation page handles auth state properly

**Session Management Issues:**
```typescript
// Debug session state
const supabase = await createClient()
const { data: { session } } = await supabase.auth.getSession()
console.log('Session:', session)

const { data: { user } } = await supabase.auth.getUser()
console.log('User:', user)
```

**Role-Based Redirect Problems:**
- Verify profile trigger creates profiles correctly
- Check that role field is populated properly
- Ensure dashboard redirect logic handles all role states

**Middleware Not Working:**
```typescript
// Check middleware config matches all routes
export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

### Debug Authentication State

```typescript
// Add to any component to debug auth state
export async function AuthDebug() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  const profile = user ? await supabase
    .from('profiles')
    .select('*')
    .eq('id', user.id)
    .single() : null

  return (
    <pre className="bg-gray-100 p-4 text-xs">
      User: {JSON.stringify(user, null, 2)}
      Profile: {JSON.stringify(profile?.data, null, 2)}
    </pre>
  )
}
```

## Next Steps

In **Step 12: User Dashboards**, you'll:
- Build role-based dashboard layouts using the authentication system
- Create consumer and provider dashboard views
- Implement dashboard navigation and view switching
- Add user profile management interfaces
- Set up the foundation for booking and service management

The authentication system you built provides the security foundation for all user interactions in the marketplace.

---

**Previous**: [Step 10: Database Schema](./10-database-schema.md) ← | → **Next**: [Step 12: User Dashboards](./12-user-dashboards.md)