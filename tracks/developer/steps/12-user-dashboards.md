# Step 12: User Dashboards

> **Phase 3: Database & Authentication - Role-Based Dashboard System**
>
> Build comprehensive user dashboards with role-based views, user management interfaces, and protected navigation using the authentication system and database schema from previous steps.

## Objective

Learn to create role-based dashboard experiences by combining CLIENT-SERVER.md patterns with COMPONENTS.md specifications. You'll build consumer and provider dashboards, implement role-based navigation, create user profile management, and establish the foundation for booking and service management.

**What you'll build:**
- Role-based dashboard routing and layouts
- Consumer dashboard with booking history and service discovery
- Provider dashboard with service management and earnings overview
- User profile management with role switching capabilities
- Protected route navigation with authentication guards
- ViewSwitcher component for seamless role transitions
- Dashboard navigation with contextual menu items

## Context Setup

Load these specifications and project documentation into Cursor:

```
@docs/specs/client-server/CLIENT-SERVER.md
@docs/specs/components/COMPONENTS.md
@docs/specs/authentication/AUTHENTICATION.md
@AGENTS.md
```

**Key points from CLIENT-SERVER.md:**
- Server Components for data fetching and authentication checks
- Client Components only for interactive UI elements
- Server Actions for form handling and data mutations
- Role-based redirect patterns and authentication guards

**Key points from COMPONENTS.md:**
- shadcn/ui component system for consistent design
- Dashboard layouts with navigation and content areas
- Form components with validation and error handling
- Data display components for tables and cards

**Key points from AUTHENTICATION.md:**
- Multi-role system (consumer, provider, admin)
- Profile management with role transitions
- Authentication utilities and route protection

## Enhanced Context7 Integration

**Comprehensive Dashboard Prompts:**

```
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/components/COMPONENTS.md
@docs/specs/authentication/AUTHENTICATION.md
@AGENTS.md

Create production-ready user dashboards for SkyMarket drone service marketplace:

**Consumer Dashboard:**
- Booking management with real-time status updates and service tracking
- Payment history with detailed receipts and refund handling
- Service discovery with location-based filtering (Detroit Metro area)
- Profile management with preferences and notification settings

**Provider Dashboard:**
- Service listing management with pricing and availability controls
- Booking requests with accept/reject workflow and calendar integration
- Earnings tracking with payout history and tax document access
- Stripe payment integration status and transaction management
- Performance analytics with booking completion rates and customer ratings

Use Next.js 15 App Router, React Server Components, and shadcn/ui components.

use context7
```

**Context7 provides:**
- Current React Server Component and data fetching patterns
- Next.js 15 App Router layout and nested routing strategies
- Form handling with React Hook Form and Zod validation patterns
- shadcn/ui component implementations with accessibility features

## Implementation Prompts

Use these prompts with the loaded context and Context7 documentation:

### 1. Dashboard Routing and Layouts

```
Using the CLIENT-SERVER.md patterns and Next.js documentation from Context7, create the dashboard routing system:

1. Dashboard redirect page that routes users based on their role
2. Consumer and provider dashboard layouts with navigation
3. Protected route guards using the authentication utilities from Step 11
4. Role-based navigation menus with contextual items
5. Responsive layouts that work on mobile and desktop

Follow the server-first approach with authentication checks in Server Components.
```

### 2. Consumer Dashboard Experience

```
Create the consumer dashboard using COMPONENTS.md specifications and React patterns from Context7:

1. Consumer dashboard overview with recent bookings and quick actions
2. Booking history table with status indicators and actions
3. Service discovery with favorite providers and categories
4. Profile management form with validation and updates
5. Quick booking shortcuts for frequently used services

Use Server Components for data fetching and Client Components for interactive elements.
```

### 3. Provider Dashboard Experience

```
Build the provider dashboard using the patterns from CLIENT-SERVER.md and Context7 documentation:

1. Provider dashboard overview with earnings, active bookings, and performance metrics
2. Service listings management with create, edit, and delete capabilities
3. Booking management interface for accepting/declining requests
4. Provider profile management with certification and service area updates
5. Onboarding flow for new providers to complete their setup

Implement proper form handling with Server Actions and client-side validation.
```

### 4. Profile Management and Role Switching

```
Create the profile management system with role switching capabilities:

1. User profile form with all fields from the profiles table
2. Provider onboarding flow that creates provider records and switches roles
3. ViewSwitcher component for seamless dashboard navigation
4. Profile photo upload and management
5. Account settings and preferences management

Use the authentication utilities and database relationships from previous steps.
```

## Code Examples

After running the prompts, you should generate:

### Dashboard Routing System
```typescript
// app/dashboard/page.tsx - Smart redirect based on role
export default async function DashboardRedirectPage() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect("/auth/login")
  }

  const { data: profile } = await supabase
    .from("profiles")
    .select("role")
    .eq("id", user.id)
    .single()

  // Redirect based on user role
  if (profile?.role === "provider") {
    redirect("/dashboard/provider")
  }

  redirect("/dashboard/consumer")
}

// app/dashboard/layout.tsx - Shared dashboard layout
export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const user = await requireAuth()
  const profile = await getProfile()

  return (
    <div className="flex h-screen">
      <DashboardSidebar user={user} profile={profile} />
      <main className="flex-1 overflow-auto">
        <DashboardHeader user={user} profile={profile} />
        <div className="p-6">
          {children}
        </div>
      </main>
    </div>
  )
}
```

### Consumer Dashboard
```typescript
// app/dashboard/consumer/page.tsx
import { createClient } from '@/lib/supabase/server'
import { requireAuth } from '@/lib/supabase/auth'
import { BookingHistoryTable } from '@/components/dashboard/BookingHistoryTable'
import { QuickActions } from '@/components/dashboard/QuickActions'
import { RecentActivity } from '@/components/dashboard/RecentActivity'

export default async function ConsumerDashboard() {
  const user = await requireAuth()
  const supabase = await createClient()

  // Fetch consumer-specific data
  const [bookingsResult, favouriteProvidersResult] = await Promise.all([
    supabase
      .from('bookings')
      .select(`
        *,
        listings (
          title,
          category,
          providers (
            profiles (full_name, avatar_url)
          )
        )
      `)
      .eq('consumer_id', user.id)
      .order('created_at', { ascending: false })
      .limit(10),

    supabase
      .from('providers')
      .select(`
        *,
        profiles (full_name, avatar_url),
        listings (count)
      `)
      .eq('verified', true)
      .order('rating', { ascending: false })
      .limit(5)
  ])

  return (
    <div className="space-y-8">
      <div>
        <h1 className="text-3xl font-bold">Consumer Dashboard</h1>
        <p className="text-muted-foreground">
          Manage your bookings and discover new services
        </p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <QuickActions />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <div className="lg:col-span-2">
          <BookingHistoryTable bookings={bookingsResult.data || []} />
        </div>
        <div>
          <RecentActivity providers={favouriteProvidersResult.data || []} />
        </div>
      </div>
    </div>
  )
}
```

### Provider Dashboard
```typescript
// app/dashboard/provider/page.tsx
import { requireProvider } from '@/lib/supabase/auth'
import { createClient } from '@/lib/supabase/server'
import { EarningsOverview } from '@/components/dashboard/EarningsOverview'
import { ActiveBookings } from '@/components/dashboard/ActiveBookings'
import { ServiceListingsTable } from '@/components/dashboard/ServiceListingsTable'

export default async function ProviderDashboard() {
  const provider = await requireProvider()
  const supabase = await createClient()

  // Fetch provider-specific data
  const [bookingsResult, listingsResult, earningsResult] = await Promise.all([
    supabase
      .from('bookings')
      .select(`
        *,
        listings (title, category),
        profiles!consumer_id (full_name, avatar_url)
      `)
      .eq('provider_id', provider.id)
      .in('status', ['pending', 'accepted', 'in_progress'])
      .order('scheduled_at', { ascending: true }),

    supabase
      .from('listings')
      .select('*')
      .eq('provider_id', provider.id)
      .order('created_at', { ascending: false }),

    supabase
      .from('bookings')
      .select('price_total, created_at, status')
      .eq('provider_id', provider.id)
      .eq('status', 'completed')
      .gte('created_at', new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString())
  ])

  return (
    <div className="space-y-8">
      <div>
        <h1 className="text-3xl font-bold">Provider Dashboard</h1>
        <p className="text-muted-foreground">
          Manage your services and track your earnings
        </p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <EarningsOverview earnings={earningsResult.data || []} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <ActiveBookings bookings={bookingsResult.data || []} />
        <ServiceListingsTable
          listings={listingsResult.data || []}
          providerId={provider.id}
        />
      </div>
    </div>
  )
}
```

### ViewSwitcher Component
```typescript
// components/layout/ViewSwitcher.tsx
'use client'

import { usePathname } from 'next/navigation'
import Link from 'next/link'
import { Check, ChevronDown } from 'lucide-react'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { Button } from '@/components/ui/button'

type Role = "consumer" | "provider" | "admin" | null

export function ViewSwitcher({ role }: { role: Role }) {
  const pathname = usePathname() || ''
  const isProviderView = pathname.startsWith('/dashboard/provider') || pathname.startsWith('/provider')
  const activeView: 'consumer' | 'provider' = isProviderView ? 'provider' : 'consumer'

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost" className="flex items-center gap-2">
          <span className="capitalize">
            {activeView}
          </span>
          <ChevronDown className="h-4 w-4" />
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-48">
        <DropdownMenuLabel>Dashboard View</DropdownMenuLabel>
        <DropdownMenuSeparator />

        <DropdownMenuItem asChild>
          <Link
            href="/dashboard/consumer"
            className="flex w-full items-center justify-between"
          >
            <span>Consumer</span>
            {activeView === 'consumer' && <Check className="h-4 w-4" />}
          </Link>
        </DropdownMenuItem>

        {role === 'provider' && (
          <DropdownMenuItem asChild>
            <Link
              href="/dashboard/provider"
              className="flex w-full items-center justify-between"
            >
              <span>Provider</span>
              {activeView === 'provider' && <Check className="h-4 w-4" />}
            </Link>
          </DropdownMenuItem>
        )}

        {role !== 'provider' && (
          <DropdownMenuItem asChild>
            <Link href="/provider/apply">
              <span className="text-muted-foreground">Become a Provider</span>
            </Link>
          </DropdownMenuItem>
        )}
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

### Profile Management
```typescript
// app/dashboard/profile/page.tsx
'use client'

import { useState, useTransition } from 'react'
import { useRouter } from 'next/navigation'
import { createClient } from '@/lib/supabase/client'
import { toast } from 'sonner'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Textarea } from '@/components/ui/textarea'

interface ProfileFormProps {
  profile: {
    id: string
    email: string
    full_name: string | null
    phone: string | null
    address: string | null
    avatar_url: string | null
    role: string
  }
}

export function ProfileForm({ profile }: ProfileFormProps) {
  const router = useRouter()
  const [isPending, startTransition] = useTransition()
  const [formData, setFormData] = useState({
    full_name: profile.full_name || '',
    phone: profile.phone || '',
    address: profile.address || '',
  })

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    startTransition(async () => {
      const supabase = createClient()

      const { error } = await supabase
        .from('profiles')
        .update(formData)
        .eq('id', profile.id)

      if (error) {
        toast.error('Failed to update profile')
        return
      }

      toast.success('Profile updated successfully')
      router.refresh()
    })
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      <div className="space-y-4">
        <div>
          <Label htmlFor="email">Email</Label>
          <Input
            id="email"
            type="email"
            value={profile.email}
            disabled
            className="bg-muted"
          />
          <p className="text-sm text-muted-foreground">
            Email cannot be changed
          </p>
        </div>

        <div>
          <Label htmlFor="full_name">Full Name</Label>
          <Input
            id="full_name"
            value={formData.full_name}
            onChange={(e) => setFormData({ ...formData, full_name: e.target.value })}
            placeholder="Enter your full name"
          />
        </div>

        <div>
          <Label htmlFor="phone">Phone</Label>
          <Input
            id="phone"
            type="tel"
            value={formData.phone}
            onChange={(e) => setFormData({ ...formData, phone: e.target.value })}
            placeholder="Enter your phone number"
          />
        </div>

        <div>
          <Label htmlFor="address">Address</Label>
          <Textarea
            id="address"
            value={formData.address}
            onChange={(e) => setFormData({ ...formData, address: e.target.value })}
            placeholder="Enter your address"
          />
        </div>
      </div>

      <Button type="submit" disabled={isPending}>
        {isPending ? 'Updating...' : 'Update Profile'}
      </Button>
    </form>
  )
}
```

## Verification Steps

### 1. Dashboard Navigation Testing
```bash
# Test role-based dashboard access:
# 1. Login as consumer - should redirect to /dashboard/consumer
# 2. Apply to be provider - role should switch to provider
# 3. Access /dashboard - should redirect to /dashboard/provider
# 4. Use ViewSwitcher to navigate between views
# 5. Test that unauthorized users get redirected to login
```

### 2. Data Display Verification
```typescript
// Check that dashboard data loads correctly
// Consumer dashboard should show:
// - Recent bookings with provider information
// - Quick action buttons for common tasks
// - Service discovery with top providers

// Provider dashboard should show:
// - Active bookings requiring attention
// - Service listings management
// - Earnings overview and metrics
```

### 3. Form Functionality Testing
```bash
# Test profile management:
# 1. Update profile information (name, phone, address)
# 2. Verify changes are saved to database
# 3. Test form validation and error handling
# 4. Check that profile data persists across sessions
```

### 4. Role-Based Feature Access
```typescript
// Verify role-based features:
// - Consumers can't access provider-only features
// - Providers see provider-specific navigation items
// - Unauthenticated users are redirected appropriately
// - ViewSwitcher only shows available roles
```

## Expected Outcome

After completing this step, you'll have:

✅ **Role-Based Dashboard System**: Separate experiences for consumers and providers
✅ **Protected Navigation**: Authentication guards on all dashboard routes
✅ **User Profile Management**: Complete profile editing with form validation
✅ **ViewSwitcher Integration**: Seamless navigation between dashboard views
✅ **Data-Driven Interfaces**: Dashboards populated with real user data
✅ **Responsive Layouts**: Mobile-friendly dashboard experiences
✅ **Provider Onboarding**: Complete flow for users to become providers

**Key achievements:**
- Multi-role dashboard system with contextual navigation
- Server Component data fetching with Client Component interactivity
- Protected routes with proper authentication checks
- User profile management with real-time updates
- Foundation for booking and service management features
- Role switching capabilities for multi-role users

## Troubleshooting

### Common Issues

**Dashboard Redirect Problems:**
```typescript
// Debug role-based redirects
export default async function DebugDashboard() {
  const user = await getUser()
  const profile = await getProfile()

  return (
    <pre>
      User: {JSON.stringify(user, null, 2)}
      Profile: {JSON.stringify(profile, null, 2)}
      Expected redirect: {profile?.role === 'provider' ? '/dashboard/provider' : '/dashboard/consumer'}
    </pre>
  )
}
```

**Data Not Loading:**
- Check that database queries match your schema
- Verify RLS policies allow users to access their own data
- Ensure foreign key relationships are properly set up
- Check that user IDs match between auth.users and profiles

**Form Submission Issues:**
```typescript
// Add error logging to forms
const { error } = await supabase
  .from('profiles')
  .update(formData)
  .eq('id', profile.id)

if (error) {
  console.error('Database error:', error)
  toast.error(`Failed to update: ${error.message}`)
  return
}
```

**ViewSwitcher Not Working:**
- Verify role is passed correctly from Server Component
- Check that provider records exist for provider role
- Ensure navigation links are correct
- Test that pathname detection works properly

### Database Query Debugging

```sql
-- Check user profiles and roles
SELECT p.id, p.email, p.full_name, p.role, pr.type, pr.verified
FROM profiles p
LEFT JOIN providers pr ON p.id = pr.user_id;

-- Verify booking data relationships
SELECT b.id, b.status, l.title, pc.full_name as consumer, pp.full_name as provider
FROM bookings b
JOIN listings l ON b.listing_id = l.id
JOIN profiles pc ON b.consumer_id = pc.id
JOIN providers pr ON b.provider_id = pr.id
JOIN profiles pp ON pr.user_id = pp.id;
```

## Next Steps

In **Step 13: CRUD Operations**, you'll:
- Build service listing creation and management
- Implement booking creation and status management
- Add search and filtering capabilities
- Create data tables with sorting and pagination
- Implement real-time updates for booking status changes

The dashboard foundation you built provides the user interface for all marketplace operations in the upcoming steps.

---

**Previous**: [Step 11: Authentication System](./11-authentication.md) ← | → **Next**: [Step 13: CRUD Operations](./13-crud-operations.md)