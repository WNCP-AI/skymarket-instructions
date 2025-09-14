# AUTHENTICATION.md - Authentication System Specification

## Overview

SkyMarket implements a comprehensive authentication system built on Supabase Auth, providing secure user management, session handling, and role-based access control for the drone service marketplace.

## Authentication Architecture

### Core Components
- **Supabase Auth**: Authentication service with JWT tokens
- **Server-Side Auth**: Cookie-based sessions with middleware
- **Client-Side Auth**: Browser authentication for interactive components
- **Role-Based Access**: Consumer, Provider, and Admin role management

### Authentication Flow
```
Registration → Email Verification → Profile Creation → Role Assignment → Dashboard Access
```

## Implementation Structure

### Client Configuration (`lib/supabase/client.ts`)
```typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### Server Configuration (`lib/supabase/server.ts`)
```typescript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
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
```

### Middleware Integration (`middleware.ts`)
```typescript
export async function middleware(request: NextRequest) {
  return await updateSession(request)
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

## Authentication Pages

### Login Page (`app/auth/login/page.tsx`)
**Features:**
- Email/password authentication
- OAuth provider integration (GitHub, Google) - currently disabled
- Email confirmation validation
- Password reset link
- Automatic dashboard redirect

**Key Implementation:**
```typescript
async function onSubmit(e: React.FormEvent) {
  const { data, error } = await supabase.auth.signInWithPassword({ email, password })

  // Email confirmation gate
  const emailConfirmed = !!data.user?.email_confirmed_at
  if (!emailConfirmed) {
    router.push("/auth/confirm")
    return
  }

  router.push("/dashboard")
}
```

### Signup Page (`app/auth/signup/page.tsx`)
**Features:**
- User registration with email/password
- Full name collection
- Email confirmation flow
- Identity validation (prevents duplicate accounts)

**Key Implementation:**
```typescript
const { data, error } = await supabase.auth.signUp({
  email,
  password,
  options: {
    data: { full_name: fullName },
    emailRedirectTo: `${window.location.origin}/auth/confirm`,
  },
})

// Check for existing account
if (data.user?.identities && data.user.identities.length === 0) {
  toast.error("Email already registered")
}
```

### Password Reset Flow
1. **Reset Request** (`app/auth/reset/page.tsx`): Email collection for reset link
2. **Reset Confirmation** (`app/auth/reset/confirm/page.tsx`): New password setting

### Email Confirmation (`app/auth/confirm/page.tsx`)
- Confirmation status display
- Resend confirmation email option
- Automatic redirect after confirmation

## Authentication Utilities (`lib/supabase/auth.ts`)

### Core Functions

**User Retrieval:**
```typescript
export async function getUser() {
  const supabase = await createClient()
  const { data: { user }, error } = await supabase.auth.getUser()
  return error || !user ? null : user
}
```

**Authentication Guard:**
```typescript
export async function requireAuth() {
  const user = await getUser()
  if (!user) redirect('/auth/login')
  return user
}
```

**Profile Management:**
```typescript
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
```

**Role-Specific Guards:**
```typescript
export async function requireProvider() {
  const user = await requireAuth()
  const { data: provider } = await supabase
    .from('providers')
    .select('*')
    .eq('user_id', user.id)
    .single()

  if (!provider) redirect('/provider/onboarding')
  return provider
}
```

## Session Management

### Server-Side Session Updates (`lib/supabase/middleware.ts`)
```typescript
export async function updateSession(request: NextRequest) {
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => request.cookies.getAll(),
        setAll: (cookies) => {
          // Set cookies in request and response
        },
      },
    }
  )

  // Refresh session if expired
  await supabase.auth.getUser()
  return supabaseResponse
}
```

### Cookie Handling
- HTTP-only cookies for security
- Automatic session refresh
- Cross-request session persistence
- Secure cookie attributes in production

## Role-Based Access Control

### User Roles
```typescript
type UserRole = 'consumer' | 'provider' | 'admin'
```

### Profile Schema
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  role user_role NOT NULL DEFAULT 'consumer',
  full_name TEXT NOT NULL,
  email TEXT NOT NULL,
  phone TEXT,
  address TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Role Assignment Flow
1. User signs up with default 'consumer' role
2. Provider application process changes role to 'provider'
3. Admin assignment through database or admin interface
4. Role validation on protected routes

## Security Features

### Row Level Security (RLS)
```sql
-- Profiles table policy
CREATE POLICY "Users can view own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);
```

### Authentication Middleware
- Automatic session refresh on every request
- Protected route enforcement
- Cookie security with HTTP-only flags
- CSRF protection through Supabase's built-in security

### Email Verification
- Required email confirmation for account activation
- Email confirmation gate on login
- Resend confirmation email functionality
- Secure redirect URLs with domain validation

## Navigation Integration

### Header Authentication State (`components/layout/SiteHeader.tsx`)
```typescript
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
    <header>
      {!user ? (
        <>
          <Link href="/auth/login">Sign In</Link>
          <Link href="/auth/signup">Get Started</Link>
        </>
      ) : (
        <>
          <Link href="/dashboard">Dashboard</Link>
          <ViewSwitcher role={role} />
          <SignOutButton />
        </>
      )}
    </header>
  )
}
```

### View Switching (`components/layout/ViewSwitcher.tsx`)
- Dynamic dashboard view switching between Consumer and Provider
- Role-based menu item visibility
- Current view indication with visual feedback

## Error Handling

### Authentication Errors
```typescript
// Login error handling
const { data, error } = await supabase.auth.signInWithPassword({ email, password })
if (error) {
  toast.error(error.message)
  return
}

// Signup error handling
if (data.user?.identities && data.user.identities.length === 0) {
  toast.error("Email already registered")
  return
}
```

### Session Validation
- Automatic retry on network failures
- Graceful degradation when auth service unavailable
- Clear error messages for user guidance
- Redirect to login on authentication failures

## Testing Patterns

### Authentication Testing
```typescript
// Mock authenticated user
const mockUser = {
  id: 'uuid',
  email: 'test@example.com',
  role: 'consumer'
}

// Test protected routes
describe('Protected Routes', () => {
  it('redirects unauthenticated users to login', async () => {
    // Test implementation
  })
})
```

## Environment Configuration

### Required Environment Variables
```bash
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Security Configuration
- JWT secret rotation
- Session timeout configuration
- Multi-factor authentication settings
- OAuth provider configurations

## Database Integration

### Auth Schema Extensions
```sql
-- Custom user metadata
ALTER TABLE auth.users
ADD COLUMN full_name TEXT,
ADD COLUMN phone TEXT;

-- Profile trigger for new users
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.profiles (id, email, full_name, role)
  VALUES (new.id, new.email, new.raw_user_meta_data->>'full_name', 'consumer');
  RETURN new;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE PROCEDURE public.handle_new_user();
```

## Performance Optimizations

### Server Components
- Authentication checks in Server Components for better performance
- Reduced client-side JavaScript bundle
- Automatic session handling without client-side code

### Caching Strategy
- Profile data caching in Server Components
- Session state caching with automatic invalidation
- Optimistic UI updates for better user experience

## Future Enhancements

### Planned Features
- Multi-factor authentication (MFA)
- Social login providers (Google, Apple, Facebook)
- Biometric authentication for mobile apps
- Single Sign-On (SSO) for enterprise customers
- Password complexity requirements
- Account lockout after failed attempts

### Security Improvements
- Rate limiting on authentication endpoints
- Device fingerprinting for fraud detection
- Session management dashboard
- Audit logging for security events
- Advanced threat detection and prevention

## Migration and Deployment

### Database Migrations
```sql
-- Authentication-related migrations
-- Migration files in supabase/migrations/
-- Profile creation and RLS policy setup
```

### Environment Setup
- Development environment configuration
- Production security hardening
- SSL certificate requirements
- Domain verification for email redirects