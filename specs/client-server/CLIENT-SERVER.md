# CLIENT-SERVER.md - Next.js Client-Server Patterns Specification

## Overview

SkyMarket implements Next.js 15 with React 19 Server Components architecture, leveraging server-first patterns with selective client-side interactivity for optimal performance and user experience.

## Architecture Philosophy

### Server-First Approach
- **Default**: Server Components for data fetching and static rendering
- **Selective**: Client Components only when interactivity is required
- **Progressive**: Enhanced with client-side features where needed
- **Performance**: Minimized JavaScript bundle size and improved load times

### Component Strategy
```
Server Components (Default) → Client Components ('use client') → Interactive Features
```

## Server Component Patterns

### Root Layout (`app/layout.tsx`)
```typescript
// Server Component - handles authentication and global setup
export default async function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  return (
    <html lang="en">
      <body className={inter.className}>
        <SiteHeader />
        {user ? (
          <MessageSubscriber userId={user.id} />
        ) : null}
        {children}
        <Toaster />
        <ChatLauncher />
      </body>
    </html>
  )
}
```

### Page Components with Data Fetching
```typescript
// Server Component with async data fetching
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

  if (profile?.role === "provider") {
    redirect("/dashboard/provider")
  }

  redirect("/dashboard/consumer")
}
```

### Authentication Integration (`components/layout/SiteHeader.tsx`)
```typescript
// Server Component with auth state
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
              <Link href="/auth/login"><Button variant="outline">Sign In</Button></Link>
              <Link href="/auth/signup"><Button>Get Started</Button></Link>
            </>
          ) : (
            <>
              <Link href="/dashboard"><Button variant="outline">Dashboard</Button></Link>
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

## Client Component Patterns

### Interactive Components with 'use client'

**ViewSwitcher** (`components/layout/ViewSwitcher.tsx`):
```typescript
'use client'

import { usePathname } from 'next/navigation'

export function ViewSwitcher({ role }: { role: Role }) {
  const pathname = usePathname() || ''
  const isProviderView = pathname.startsWith('/dashboard/provider') || pathname.startsWith('/provider')
  const activeView: 'consumer' | 'provider' = isProviderView ? 'provider' : 'consumer'

  return (
    <DropdownMenu>
      <DropdownMenuTrigger asChild>
        <Button variant="ghost">
          {activeView.charAt(0).toUpperCase() + activeView.slice(1)}
        </Button>
      </DropdownMenuTrigger>
      <DropdownMenuContent align="end" className="w-48">
        <DropdownMenuLabel>Dashboard View</DropdownMenuLabel>
        <DropdownMenuSeparator />
        <DropdownMenuItem asChild>
          <Link href="/dashboard/consumer" className="flex w-full items-center justify-between">
            <span>Consumer</span>
            {activeView === 'consumer' && <Check className="h-4 w-4" />}
          </Link>
        </DropdownMenuItem>
        {role === 'provider' && (
          <DropdownMenuItem asChild>
            <Link href="/dashboard/provider" className="flex w-full items-center justify-between">
              <span>Provider</span>
              {activeView === 'provider' && <Check className="h-4 w-4" />}
            </Link>
          </DropdownMenuItem>
        )}
      </DropdownMenuContent>
    </DropdownMenu>
  )
}
```

**ChatLauncher** (`components/chat/ChatLauncher.tsx`):
```typescript
'use client'

import dynamic from 'next/dynamic'
import { useState } from 'react'

// Dynamic import with SSR disabled for client-only components
const ChatWindow = dynamic(() => import('./ChatWindow').then(m => m.ChatWindow), { ssr: false })

export function ChatLauncher() {
  const [open, setOpen] = useState(false)

  return (
    <div className="fixed bottom-4 right-4 z-50">
      {!open && (
        <button
          onClick={() => setOpen(true)}
          className="flex h-12 w-12 items-center justify-center rounded-full bg-black text-white shadow"
          aria-label="Open chat"
        >
          <MessageCircle className="h-6 w-6" />
        </button>
      )}
      {open && (
        <div className="h-[480px] w-[360px] overflow-hidden rounded-xl border bg-white shadow lg:h-[560px] lg:w-[400px]">
          <ChatWindow onClose={() => setOpen(false)} />
        </div>
      )}
    </div>
  )
}
```

### Authentication Forms with Transitions

**Login Page** (`app/auth/login/page.tsx`):
```typescript
"use client"

import { useState, useTransition } from "react"
import { useRouter } from "next/navigation"

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
    <form onSubmit={onSubmit} className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
      </div>
      <div className="space-y-2">
        <Label htmlFor="password">Password</Label>
        <Input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
      </div>
      <Button className="w-full" disabled={isPending}>
        Sign In
      </Button>
    </form>
  )
}
```

## Data Fetching Patterns

### Server-Side Data Fetching
```typescript
// Direct database queries in Server Components
export default async function ServerPage() {
  const supabase = await createClient()

  const { data: listings } = await supabase
    .from('listings')
    .select('*')
    .eq('active', true)
    .order('created_at', { ascending: false })

  return (
    <div>
      {listings?.map(listing => (
        <ListingCard key={listing.id} listing={listing} />
      ))}
    </div>
  )
}
```

### Client-Side State Management
```typescript
'use client'

import { useState, useEffect } from 'react'
import { createClient } from '@/lib/supabase/client'

export function ClientDataComponent() {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const supabase = createClient()

  useEffect(() => {
    async function fetchData() {
      const { data: result } = await supabase
        .from('table')
        .select('*')

      setData(result)
      setLoading(false)
    }

    fetchData()
  }, [supabase])

  if (loading) return <div>Loading...</div>

  return <div>{/* Render data */}</div>
}
```

## Real-time Integration

### Server Component with Real-time Subscription
```typescript
'use client'

import { useEffect } from 'react'
import { createClient } from '@/lib/supabase/client'
import { toast } from 'sonner'

export default function MessageSubscriber({ userId }: { userId: string }) {
  useEffect(() => {
    const supabase = createClient()
    const channel = supabase
      .channel("messages-realtime")
      .on(
        "postgres_changes",
        {
          event: "INSERT",
          schema: "public",
          table: "messages",
          filter: `recipient_id=eq.${userId}`
        },
        (payload) => {
          const content = payload.new.content
          toast.message("New message", { description: content })
        }
      )
      .subscribe()

    return () => {
      supabase.removeChannel(channel)
    }
  }, [userId])

  return null // No UI, just subscription logic
}
```

## Route Layout Patterns

### Nested Layouts
```typescript
// app/auth/layout.tsx - Authentication pages layout
export default function AuthLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return (
    <div className="min-h-[calc(100vh-0px)] flex items-center justify-center px-4 py-12 bg-background">
      <div className="w-full max-w-md">
        {children}
      </div>
    </div>
  )
}
```

### Route Groups and Parallel Routes
```
app/
├── layout.tsx                    # Root layout
├── page.tsx                      # Home page
├── (auth)/                       # Route group
│   ├── layout.tsx               # Auth layout
│   ├── login/page.tsx           # Login page
│   ├── signup/page.tsx          # Signup page
│   └── reset/page.tsx           # Password reset
├── dashboard/
│   ├── page.tsx                 # Dashboard redirect
│   ├── consumer/page.tsx        # Consumer dashboard
│   └── provider/page.tsx        # Provider dashboard
└── api/                         # API routes
    ├── orders/route.ts          # Orders endpoint
    └── webhooks/stripe/route.ts # Webhook handler
```

## Server Actions Integration

### Form Handling with Server Actions
```typescript
// Server Action in Server Component
async function createProvider(formData: FormData) {
  "use server"

  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect("/auth/login")

  const type = String(formData.get("type") || "courier") as "courier" | "drone"
  const certifications = String(formData.get("certifications") || "")
    .split(",")
    .map((s) => s.trim())

  await supabase
    .from("providers")
    .insert({
      user_id: user.id,
      type,
      certifications,
    })

  revalidatePath("/dashboard/provider")
  redirect("/dashboard/provider")
}

// Server Component with form
export default async function ProviderApplyPage() {
  return (
    <form action={createProvider} className="space-y-6">
      <Select name="type" defaultValue="courier">
        <SelectTrigger>
          <SelectValue placeholder="Select type" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="courier">Courier</SelectItem>
          <SelectItem value="drone">Drone Operator</SelectItem>
        </SelectContent>
      </Select>
      <Button type="submit">Submit</Button>
    </form>
  )
}
```

## API Route Patterns

### REST API with Validation
```typescript
// app/api/orders/route.ts
import { NextResponse } from 'next/server'
import { z } from 'zod'
import { createClient } from '@/lib/supabase/server'

const createOrderSchema = z.object({
  listingId: z.string().uuid(),
  scheduledAt: z.string().datetime(),
  dropoffAddress: z.string().min(1),
  specialInstructions: z.string().optional().nullable(),
})

export async function POST(request: Request) {
  try {
    const body = await request.json()
    const parsed = createOrderSchema.parse(body)

    const supabase = await createClient()
    const { data: { user } } = await supabase.auth.getUser()

    if (!user) {
      return NextResponse.json(
        { error: { message: 'Unauthorized' } },
        { status: 401 }
      )
    }

    // Process order creation
    const result = await createOrder(parsed, user.id)

    return NextResponse.json({ data: result })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        error: {
          message: 'Validation failed',
          details: error.errors
        }
      }, { status: 400 })
    }

    return NextResponse.json({
      error: { message: 'Internal server error' }
    }, { status: 500 })
  }
}
```

## Performance Optimization Patterns

### Dynamic Imports for Code Splitting
```typescript
// Lazy load heavy components
const ChatWindow = dynamic(
  () => import('./ChatWindow').then(m => m.ChatWindow),
  { ssr: false }
)

// Route-based code splitting
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <p>Loading...</p>,
})
```

### Streaming and Suspense
```typescript
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<div>Loading data...</div>}>
        <AsyncDataComponent />
      </Suspense>
    </div>
  )
}

async function AsyncDataComponent() {
  const data = await fetchData()
  return <div>{/* Render data */}</div>
}
```

### Cache Optimization
```typescript
// Server Component with caching
export default async function CachedPage() {
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 } // Cache for 60 seconds
  })

  return <div>{/* Render data */}</div>
}

// Revalidation patterns
import { revalidatePath, revalidateTag } from 'next/cache'

// Revalidate specific path
revalidatePath('/dashboard')

// Revalidate by tag
revalidateTag('orders')
```

## Error Handling Patterns

### Error Boundaries
```typescript
'use client'

import { useEffect } from 'react'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    console.error(error)
  }, [error])

  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Something went wrong!</h2>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        Try again
      </button>
    </div>
  )
}
```

### Not Found Pages
```typescript
// app/not-found.tsx
export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Not Found</h2>
      <p>Could not find requested resource</p>
      <Link href="/">Return Home</Link>
    </div>
  )
}
```

## Environment and Configuration

### Environment Variables Access
```typescript
// Server-side environment access
const baseUrl = process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000"
const serviceRoleKey = process.env.SUPABASE_SERVICE_ROLE_KEY!

// Client-side environment access (NEXT_PUBLIC_ prefix)
const publicUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const anonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
```

### Type Safety Integration
```typescript
// Database types integration
import type { Database } from '@/types/database'

type ListingsRow = Database["public"]["Tables"]["listings"]["Row"]
type ProvidersInsert = Database["public"]["Tables"]["providers"]["Insert"]

// Component props with database types
interface Props {
  listing: ListingsRow
  onUpdate: (update: ProvidersInsert) => void
}
```

## Security Patterns

### Authentication Guards
```typescript
// Server Component auth guard
export default async function ProtectedPage() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect('/auth/login')
  }

  return <div>Protected content</div>
}

// Client Component auth guard
'use client'

export function ClientProtectedComponent() {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    async function getUser() {
      const supabase = createClient()
      const { data: { user } } = await supabase.auth.getUser()

      if (!user) {
        router.push('/auth/login')
        return
      }

      setUser(user)
      setLoading(false)
    }

    getUser()
  }, [])

  if (loading) return <div>Loading...</div>

  return <div>Protected content</div>
}
```

### CSRF Protection
```typescript
// Server Actions provide built-in CSRF protection
async function secureAction(formData: FormData) {
  "use server"
  // Automatically protected against CSRF
}

// API Routes with custom headers
export async function POST(request: Request) {
  const origin = request.headers.get('origin')
  const referer = request.headers.get('referer')

  // Validate origin/referer as needed
  if (!origin || !allowedOrigins.includes(origin)) {
    return new Response('Forbidden', { status: 403 })
  }
}
```

## Future Enhancement Patterns

### Planned Improvements
- Progressive Web App (PWA) integration
- Advanced caching strategies with Vercel's Edge Runtime
- Real-time collaboration features with Supabase channels
- Enhanced error reporting with Sentry integration
- Performance monitoring with Web Vitals tracking

### Scalability Patterns
- Micro-frontends for large feature areas
- Edge computing for geolocation-based services
- CDN optimization for Detroit Metro area
- Database connection pooling optimization
- Advanced state management for complex interactions