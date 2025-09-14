# FORMS.md - Form Handling Patterns Specification

## Overview

SkyMarket implements comprehensive form handling patterns using Next.js Server Actions, Zod validation, and shadcn/ui components, providing secure and user-friendly form experiences across the application.

## Form Architecture

### Core Technologies
- **Next.js Server Actions**: Server-side form processing with "use server" directive
- **Zod Schemas**: Type-safe validation with runtime type checking
- **React Hook Form**: Client-side form state management and validation
- **shadcn/ui Form Components**: Consistent UI with accessibility features

### Form Processing Flow
```
Client Form → Server Action → Validation → Database Operation → Response/Redirect
```

## Implementation Patterns

### Server Action Forms

#### Basic Server Action Pattern
```typescript
async function serverAction(formData: FormData) {
  "use server"

  // Authentication check
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect("/auth/login")

  // Form data extraction
  const field = String(formData.get("field_name") || "")

  // Database operation
  await supabase.from("table").insert({ field })

  // Redirect or revalidate
  revalidatePath("/path")
  redirect("/success")
}
```

#### Authentication Forms (`app/auth/`)

**Login Form** (`app/auth/login/page.tsx`):
```typescript
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
  })
}
```

**Signup Form** (`app/auth/signup/page.tsx`):
```typescript
async function onSubmit(e: React.FormEvent) {
  e.preventDefault()
  startTransition(async () => {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        data: { full_name: fullName },
        emailRedirectTo: `${window.location.origin}/auth/confirm`,
      },
    })

    // Handle duplicate account detection
    if (data.user?.identities && data.user.identities.length === 0) {
      toast.error("Email already registered")
      return
    }

    toast.success("Check your email to confirm your account")
    router.push("/auth/login")
  })
}
```

#### Provider Application Form (`app/provider/apply/page.tsx`)

**Server Action Implementation:**
```typescript
async function createProvider(formData: FormData) {
  "use server"

  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect("/auth/login")

  // Data extraction with type casting
  const type = String(formData.get("type") || "courier") as "courier" | "drone"
  const certificationsRaw = String(formData.get("certifications") || "").trim()
  const serviceAreasRaw = String(formData.get("service_areas") || "").trim()

  // Array processing
  const certifications = certificationsRaw ?
    certificationsRaw.split(",").map((s) => s.trim()) : []
  const service_areas = serviceAreasRaw ?
    serviceAreasRaw.split(",").map((s) => s.trim()) : []

  // Database operations with type safety
  type ProvidersInsert = Database["public"]["Tables"]["providers"]["Insert"]

  await supabase
    .from("providers")
    .insert({
      user_id: user.id,
      type,
      certifications,
      service_areas
    } as ProvidersInsert)

  // Role update
  await supabase
    .from("profiles")
    .update({ role: "provider" } as Database["public"]["Tables"]["profiles"]["Update"])
    .eq("id", user.id)

  revalidatePath("/dashboard/provider")
  redirect("/dashboard/provider")
}
```

**Form JSX:**
```typescript
<form action={createProvider} className="space-y-6">
  <div className="space-y-2">
    <Label>Provider Type</Label>
    <Select name="type" defaultValue="courier">
      <SelectTrigger className="w-full">
        <SelectValue placeholder="Select type" />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="courier">Courier</SelectItem>
        <SelectItem value="drone">Drone Operator</SelectItem>
      </SelectContent>
    </Select>
  </div>

  <div className="space-y-2">
    <Label htmlFor="certifications">
      Certifications (comma-separated)
    </Label>
    <Input
      id="certifications"
      name="certifications"
      placeholder="Part 107, Insurance"
    />
  </div>

  <Button type="submit">Submit</Button>
</form>
```

#### Booking Form (`app/booking/[listingId]/page.tsx`)

**Complex Server Action with API Integration:**
```typescript
async function createBooking(formData: FormData) {
  "use server"

  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) redirect("/auth/login")

  // Form data extraction
  const listing_id = String(formData.get("listing_id"))
  const scheduledRaw = String(formData.get("scheduled_at") || "")
  const pickup_address = String(formData.get("pickup_address") || "")
  const dropoff_address = String(formData.get("dropoff_address") || "")
  const special_instructions = String(formData.get("special_instructions") || "")

  // Date validation
  const scheduledDate = new Date(scheduledRaw)
  if (Number.isNaN(scheduledDate.valueOf())) {
    const params = new URLSearchParams({ err: 'invalid_date' })
    redirect(`/booking/${listing_id}?${params.toString()}`)
  }

  // API call to create booking with Stripe integration
  const hdrs = await headers()
  const cookieHeader = hdrs.get("cookie") || ""
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000"

  const resp = await fetch(`${baseUrl}/api/orders`, {
    method: "POST",
    headers: {
      "content-type": "application/json",
      cookie: cookieHeader
    },
    body: JSON.stringify({
      listingId: listing_id,
      scheduledAt: scheduled_at,
      pickupAddress: pickup_address || null,
      dropoffAddress: dropoff_address,
      specialInstructions: special_instructions || null,
    }),
  })

  // Error handling
  if (!resp.ok) {
    const { error } = await resp.json()
    const params = new URLSearchParams({
      err: 'api',
      msg: String(error?.message || 'Request failed')
    })
    redirect(`/booking/${listing_id}?${params.toString()}`)
  }

  const json = await resp.json()
  const bookingId = json.data?.id
  if (!bookingId) redirect(`/browse`)

  revalidatePath(`/orders/${bookingId}`)
  redirect(`/orders/${bookingId}`)
}
```

## API Route Forms

### Zod Validation Schema (`app/api/orders/route.ts`)
```typescript
const createOrderSchema = z.object({
  listingId: z.string().uuid(),
  scheduledAt: z.string().datetime(),
  pickupAddress: z.string().optional().nullable(),
  dropoffAddress: z.string().min(1),
  specialInstructions: z.string().optional().nullable(),
  distanceMiles: z.number().nonnegative().optional(),
  durationMinutes: z.number().nonnegative().optional(),
  pickupLat: z.number().optional(),
  pickupLng: z.number().optional(),
  dropoffLat: z.number().optional(),
  dropoffLng: z.number().optional(),
})
```

### API Route Handler
```typescript
export async function POST(request: Request) {
  try {
    const body = await request.json()
    const parsed = createOrderSchema.parse(body)

    // Authentication
    const supabase = await createClient()
    const { data: { user } } = await supabase.auth.getUser()
    if (!user) {
      return NextResponse.json({ error: { message: 'Unauthorized' } }, { status: 401 })
    }

    // Business logic processing
    const result = await processOrder(parsed)

    return NextResponse.json({ data: result })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        error: { message: 'Validation failed', details: error.errors }
      }, { status: 400 })
    }

    return NextResponse.json({
      error: { message: 'Internal server error' }
    }, { status: 500 })
  }
}
```

## React Hook Form Integration

### shadcn/ui Form Components (`components/ui/form.tsx`)
```typescript
const Form = FormProvider

const FormField = <
  TFieldValues extends FieldValues = FieldValues,
  TName extends FieldPath<TFieldValues> = FieldPath<TFieldValues>,
>({
  ...props
}: ControllerProps<TFieldValues, TName>) => {
  return (
    <FormFieldContext.Provider value={{ name: props.name }}>
      <Controller {...props} />
    </FormFieldContext.Provider>
  )
}

const useFormField = () => {
  const fieldContext = React.useContext(FormFieldContext)
  const itemContext = React.useContext(FormItemContext)
  const { getFieldState } = useFormContext()
  const formState = useFormState({ name: fieldContext.name })
  const fieldState = getFieldState(fieldContext.name, formState)

  const { id } = itemContext

  return {
    id,
    name: fieldContext.name,
    formItemId: `${id}-form-item`,
    formDescriptionId: `${id}-form-item-description`,
    formMessageId: `${id}-form-item-message`,
    ...fieldState,
  }
}
```

### Client-Side Form Pattern
```typescript
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const formSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
})

export function ClientForm() {
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      email: "",
      password: "",
    },
  })

  async function onSubmit(values: z.infer<typeof formSchema>) {
    // Submit logic
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input placeholder="email@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Submit</Button>
      </form>
    </Form>
  )
}
```

## Form Components

### Input Components
- **Input** (`components/ui/input.tsx`): Text input with validation states
- **Textarea** (`components/ui/textarea.tsx`): Multi-line text input
- **Select** (`components/ui/select.tsx`): Dropdown selection with keyboard navigation
- **Label** (`components/ui/label.tsx`): Accessible form labels

### Form Layout Components
- **Card** (`components/ui/card.tsx`): Form containers and sections
- **Button** (`components/ui/button.tsx`): Submit buttons with loading states
- **Dialog** (`components/ui/dialog.tsx`): Modal forms

## Validation Patterns

### Server-Side Validation
```typescript
// Basic validation
const email = String(formData.get("email") || "")
if (!email.includes("@")) {
  const params = new URLSearchParams({ err: 'invalid_email' })
  redirect(`/form?${params.toString()}`)
}

// Date validation
const scheduledDate = new Date(scheduledRaw)
if (Number.isNaN(scheduledDate.valueOf())) {
  const params = new URLSearchParams({ err: 'invalid_date' })
  redirect(`/booking/${listing_id}?${params.toString()}`)
}
```

### Client-Side Validation
```typescript
// HTML5 validation attributes
<Input
  type="email"
  required
  minLength={8}
  pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
/>

// React Hook Form with Zod
const schema = z.object({
  email: z.string().email("Invalid email format"),
  password: z.string()
    .min(8, "Password must be at least 8 characters")
    .regex(/[A-Z]/, "Password must contain uppercase letter"),
})
```

## Error Handling

### Server Action Error Patterns
```typescript
// Redirect with error parameters
if (error) {
  const params = new URLSearchParams({
    err: 'validation_failed',
    msg: error.message
  })
  redirect(`/form?${params.toString()}`)
}

// Toast notifications
toast.error(error.message)
toast.success("Form submitted successfully")
```

### API Route Error Handling
```typescript
try {
  const parsed = schema.parse(body)
  // Process form
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
```

## Security Patterns

### Authentication Guards
```typescript
// Server Action authentication
const { data: { user } } = await supabase.auth.getUser()
if (!user) redirect("/auth/login")

// API Route authentication
const { data: { user } } = await supabase.auth.getUser()
if (!user) {
  return NextResponse.json({
    error: { message: 'Unauthorized' }
  }, { status: 401 })
}
```

### CSRF Protection
- Server Actions provide built-in CSRF protection
- Form tokens automatically validated by Next.js
- Cookie-based session management prevents CSRF attacks

### Input Sanitization
```typescript
// String sanitization
const sanitized = String(formData.get("field") || "").trim()

// Array processing with validation
const tags = tagsRaw ?
  tagsRaw.split(",").map((s) => s.trim()).filter(Boolean) : []

// Number validation
const price = Number(formData.get("price"))
if (Number.isNaN(price) || price < 0) {
  // Handle invalid number
}
```

## Detroit-Specific Validation

### Address Validation
```typescript
// Detroit Metro area validation
function validateDetroitAddress(address: string): boolean {
  // Implementation for Detroit Metro boundary checking
  return true // Placeholder
}

// Coordinate validation
function isWithinDetroitBounds(lat: number, lng: number): boolean {
  return (
    lat >= 42.0 && lat <= 42.6 &&
    lng >= -83.5 && lng <= -82.8
  )
}
```

### Service Area Validation
```typescript
// 25-mile radius from Detroit center
const DETROIT_CENTER = { lat: 42.3314, lng: -83.0458 }
const SERVICE_RADIUS_MILES = 25

function isWithinServiceArea(lat: number, lng: number): boolean {
  const distance = haversineMiles(DETROIT_CENTER, { lat, lng })
  return distance <= SERVICE_RADIUS_MILES
}
```

## Business Logic Integration

### Price Calculation
```typescript
function computePriceTotal(
  listing: {
    price_base: number;
    price_per_mile: number | null;
    price_per_minute: number | null
  },
  distanceMiles?: number,
  durationMinutes?: number,
) {
  const base = Number(listing.price_base || 0)
  const perMile = Number(listing.price_per_mile || 0)
  const perMin = Number(listing.price_per_minute || 0)
  const variable = (distanceMiles ?? 0) * perMile + (durationMinutes ?? 0) * perMin
  const total = base + variable
  return Math.max(0, Math.round(total * 100) / 100)
}
```

### Distance Calculation
```typescript
function haversineMiles(
  a: { lat: number; lng: number },
  b: { lat: number; lng: number }
) {
  const toRad = (x: number) => (x * Math.PI) / 180
  const R = 3958.8 // miles
  const dLat = toRad(b.lat - a.lat)
  const dLon = toRad(b.lng - a.lng)
  const lat1 = toRad(a.lat)
  const lat2 = toRad(b.lat)
  const h =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(lat1) * Math.cos(lat2) * Math.sin(dLon / 2) * Math.sin(dLon / 2)
  const c = 2 * Math.atan2(Math.sqrt(h), Math.sqrt(1 - h))
  return R * c
}
```

## Performance Optimizations

### Form Optimization
- `useTransition` for async form submissions
- Optimistic updates for better UX
- Form validation debouncing
- Client-side caching of form state

### Server Action Optimization
- `revalidatePath` for targeted cache invalidation
- Parallel database operations where possible
- Efficient redirect patterns
- Minimal data transfer

## Accessibility Features

### Form Accessibility
- Proper label associations with `htmlFor`
- Required field indicators
- Error message announcements
- Keyboard navigation support
- Focus management and indicators

### ARIA Attributes
```typescript
<Input
  aria-describedby="email-error"
  aria-invalid={!!errors.email}
  aria-required="true"
/>
<div id="email-error" role="alert">
  {errors.email?.message}
</div>
```

## Testing Patterns

### Form Testing
```typescript
// Server Action testing
describe('createProvider', () => {
  it('creates provider with valid data', async () => {
    const formData = new FormData()
    formData.set('type', 'drone')
    formData.set('certifications', 'Part 107, Insurance')

    // Test server action
  })
})

// Client form testing
describe('LoginForm', () => {
  it('submits with valid credentials', async () => {
    render(<LoginForm />)

    fireEvent.change(screen.getByLabelText('Email'), {
      target: { value: 'test@example.com' }
    })

    fireEvent.click(screen.getByRole('button', { name: 'Sign In' }))
    // Assertions
  })
})
```

## Future Enhancements

### Planned Improvements
- Real-time form validation
- Multi-step form wizards
- File upload components
- Auto-save draft functionality
- Form analytics and conversion tracking
- Advanced accessibility features

### Enhanced Validation
- Custom Zod schemas for Detroit addresses
- Real-time availability checking
- Dynamic pricing updates
- Weather condition validation for drone services
- FAA airspace validation integration