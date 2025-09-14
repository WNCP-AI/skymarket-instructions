# Step 13: CRUD Operations - Spec-Driven Service and Booking Management

## Objective

Learn to implement comprehensive CRUD operations by combining project specifications with official documentation. You'll build the core service management system and booking infrastructure using BUSINESS-LOGIC.md, API.md, and Supabase patterns.

**Learning Focus**: Specification-driven API development using Context7 for official documentation lookup and TypeScript-first implementation patterns.

## Context Setup

### Required Specifications
Load these project specifications to understand the business requirements:

```bash
# Load core business logic and constraints
@docs/specs/BUSINESS-LOGIC.md

# Load API endpoint specifications
@docs/specs/API.md

# Load data validation patterns
@docs/specs/FORMS.md
```

### Context7 Documentation Integration
Use Context7 to get up-to-date official documentation while implementing. Include "use context7" and reference specific library IDs when known:

**Example Integration Prompts:**

"Create Supabase server actions for Next.js App Router with proper authentication, RLS policies, and error handling for CRUD operations. Reference /supabase/supabase for official patterns. use context7"

"Implement React Hook Form with Zod validation and TypeScript integration for complex form workflows with proper error handling and accessibility. Reference /react-hook-form/react-hook-form documentation. use context7"

"Set up Supabase Row Level Security policies for a multi-tenant application with role-based access control. Reference /supabase/supabase RLS documentation. use context7"

"Configure Supabase real-time subscriptions in TypeScript with proper cleanup and error handling patterns. Reference /supabase/supabase real-time documentation. use context7"

## Implementation Prompts

### Prompt 1: Service CRUD System
"Using BUSINESS-LOGIC.md specifications, implement a complete service management system with:

1. Service creation form supporting all 4 categories (food_delivery, courier, aerial_imaging, site_mapping)
2. Service listing with filtering by category and location
3. Service editing with proper validation
4. Service deletion with soft-delete patterns
5. Detroit Metro geographic constraint validation
6. TypeScript-first approach with proper error handling

Create Supabase server actions with proper authentication, RLS policies, and error handling for all CRUD operations. Reference /supabase/supabase for server actions patterns. Implement React Hook Form with Zod validation for the service management forms. Reference /react-hook-form/react-hook-form documentation. use context7"

### Prompt 2: Booking CRUD Operations
"Using API.md specifications, create the booking management system with:

1. Booking creation with state machine validation
2. Booking status transitions (pending → accepted → in_progress → completed)
3. Geographic validation for Detroit Metro area
4. Price calculations based on service type and distance
5. Proper RLS policies for data security
6. Server Actions for secure mutations

Implement Supabase server actions for booking CRUD operations with proper state machine validation, geographic constraints, and real-time subscriptions for status updates. Reference /supabase/supabase for server actions and real-time patterns. use context7"

## Code Examples

### Service Management Component

```typescript
// app/dashboard/provider/services/page.tsx
import { Suspense } from 'react'
import { createServerClient } from '@/lib/supabase/server'
import { ServicesList } from '@/components/providers/ServicesList'
import { CreateServiceDialog } from '@/components/providers/CreateServiceDialog'
import { Button } from '@/components/ui/button'
import { Plus } from 'lucide-react'

export default async function ServicesPage() {
  const supabase = await createServerClient()

  const { data: services, error } = await supabase
    .from('services')
    .select(`
      *,
      provider:providers(name, avatar_url),
      _count:bookings(count)
    `)
    .eq('provider_id', (await supabase.auth.getUser()).data.user?.id)
    .order('created_at', { ascending: false })

  if (error) {
    console.error('Error fetching services:', error)
    return <div>Error loading services</div>
  }

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-3xl font-bold">My Services</h1>
        <CreateServiceDialog>
          <Button>
            <Plus className="h-4 w-4 mr-2" />
            Add Service
          </Button>
        </CreateServiceDialog>
      </div>

      <Suspense fallback={<div>Loading services...</div>}>
        <ServicesList services={services || []} />
      </Suspense>
    </div>
  )
}
```

### Service Creation Form with Validation

```typescript
// components/providers/CreateServiceForm.tsx
'use client'

import { useState } from 'react'
import { useRouter } from 'next/navigation'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { createService } from '@/app/actions/services'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { useToast } from '@/hooks/use-toast'

// Service categories from BUSINESS-LOGIC.md
const SERVICE_CATEGORIES = [
  { value: 'food_delivery', label: 'Food Delivery' },
  { value: 'courier', label: 'Courier Services' },
  { value: 'aerial_imaging', label: 'Aerial Imaging' },
  { value: 'site_mapping', label: 'Site Mapping' }
] as const

// Validation schema based on BUSINESS-LOGIC.md constraints
const serviceSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters').max(100),
  description: z.string().min(10, 'Description must be at least 10 characters').max(1000),
  category: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
  price_per_hour: z.number().min(25, 'Minimum rate is $25/hour').max(500, 'Maximum rate is $500/hour'),
  service_area_radius: z.number().min(1, 'Minimum radius is 1 mile').max(25, 'Maximum radius is 25 miles (Detroit Metro limit)'),
  max_payload_kg: z.number().min(0.1).max(5, 'Maximum payload is 5kg for consumer drones'),
  flight_time_minutes: z.number().min(10).max(120, 'Maximum flight time is 2 hours'),
  requires_pilot_certification: z.boolean().default(true),
  available_days: z.array(z.number().min(0).max(6)).min(1, 'Select at least one available day'),
  start_time: z.string().regex(/^([01]?[0-9]|2[0-3]):[0-5][0-9]$/, 'Invalid time format'),
  end_time: z.string().regex(/^([01]?[0-9]|2[0-3]):[0-5][0-9]$/, 'Invalid time format')
})

type ServiceFormData = z.infer<typeof serviceSchema>

interface CreateServiceFormProps {
  onSuccess?: () => void
}

export function CreateServiceForm({ onSuccess }: CreateServiceFormProps) {
  const [isSubmitting, setIsSubmitting] = useState(false)
  const router = useRouter()
  const { toast } = useToast()

  const form = useForm<ServiceFormData>({
    resolver: zodResolver(serviceSchema),
    defaultValues: {
      category: 'food_delivery',
      price_per_hour: 50,
      service_area_radius: 10,
      max_payload_kg: 2,
      flight_time_minutes: 30,
      requires_pilot_certification: true,
      available_days: [1, 2, 3, 4, 5], // Monday-Friday
      start_time: '09:00',
      end_time: '17:00'
    }
  })

  const onSubmit = async (data: ServiceFormData) => {
    setIsSubmitting(true)

    try {
      const result = await createService(data)

      if (result.success) {
        toast({
          title: 'Service Created',
          description: 'Your service has been successfully created and is now available for booking.',
        })

        form.reset()
        onSuccess?.()
        router.refresh()
      } else {
        toast({
          title: 'Error',
          description: result.error || 'Failed to create service',
          variant: 'destructive'
        })
      }
    } catch (error) {
      console.error('Service creation error:', error)
      toast({
        title: 'Error',
        description: 'An unexpected error occurred',
        variant: 'destructive'
      })
    } finally {
      setIsSubmitting(false)
    }
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <FormField
            control={form.control}
            name="title"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Service Title</FormLabel>
                <FormControl>
                  <Input placeholder="Professional drone photography..." {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="category"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Service Category</FormLabel>
                <Select onValueChange={field.onChange} defaultValue={field.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue placeholder="Select category" />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    {SERVICE_CATEGORIES.map((category) => (
                      <SelectItem key={category.value} value={category.value}>
                        {category.label}
                      </SelectItem>
                    ))}
                  </SelectContent>
                </Select>
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <FormField
          control={form.control}
          name="description"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Description</FormLabel>
              <FormControl>
                <Textarea
                  placeholder="Describe your service, capabilities, and what clients can expect..."
                  className="min-h-[120px]"
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <FormField
            control={form.control}
            name="price_per_hour"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Hourly Rate ($)</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    min="25"
                    max="500"
                    {...field}
                    onChange={e => field.onChange(parseFloat(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="service_area_radius"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Service Radius (miles)</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    min="1"
                    max="25"
                    {...field}
                    onChange={e => field.onChange(parseFloat(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />

          <FormField
            control={form.control}
            name="max_payload_kg"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Max Payload (kg)</FormLabel>
                <FormControl>
                  <Input
                    type="number"
                    min="0.1"
                    max="5"
                    step="0.1"
                    {...field}
                    onChange={e => field.onChange(parseFloat(e.target.value))}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
        </div>

        <div className="flex justify-end space-x-4">
          <Button type="button" variant="outline" onClick={() => form.reset()}>
            Reset
          </Button>
          <Button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Creating...' : 'Create Service'}
          </Button>
        </div>
      </form>
    </Form>
  )
}
```

### Server Action for Service Creation

```typescript
// app/actions/services.ts
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { createServerClient } from '@/lib/supabase/server'
import type { Database } from '@/types/database'

const serviceSchema = z.object({
  title: z.string().min(3).max(100),
  description: z.string().min(10).max(1000),
  category: z.enum(['food_delivery', 'courier', 'aerial_imaging', 'site_mapping']),
  price_per_hour: z.number().min(25).max(500),
  service_area_radius: z.number().min(1).max(25),
  max_payload_kg: z.number().min(0.1).max(5),
  flight_time_minutes: z.number().min(10).max(120),
  requires_pilot_certification: z.boolean(),
  available_days: z.array(z.number().min(0).max(6)),
  start_time: z.string(),
  end_time: z.string()
})

type ServiceInput = z.infer<typeof serviceSchema>

export async function createService(input: ServiceInput) {
  try {
    // Validate input
    const validatedData = serviceSchema.parse(input)

    // Get authenticated user
    const supabase = await createServerClient()
    const { data: { user }, error: authError } = await supabase.auth.getUser()

    if (authError || !user) {
      return { success: false, error: 'Authentication required' }
    }

    // Verify user is a provider
    const { data: provider, error: providerError } = await supabase
      .from('providers')
      .select('id, part_107_certified')
      .eq('id', user.id)
      .single()

    if (providerError || !provider) {
      return { success: false, error: 'Provider profile required' }
    }

    // Validate Part 107 certification for drone services
    if (validatedData.requires_pilot_certification && !provider.part_107_certified) {
      return {
        success: false,
        error: 'Part 107 certification required for this service type'
      }
    }

    // Create service
    const { data: service, error: serviceError } = await supabase
      .from('services')
      .insert([{
        provider_id: user.id,
        title: validatedData.title,
        description: validatedData.description,
        category: validatedData.category,
        price_per_hour: validatedData.price_per_hour,
        service_area_radius: validatedData.service_area_radius,
        max_payload_kg: validatedData.max_payload_kg,
        flight_time_minutes: validatedData.flight_time_minutes,
        requires_pilot_certification: validatedData.requires_pilot_certification,
        available_days: validatedData.available_days,
        start_time: validatedData.start_time,
        end_time: validatedData.end_time,
        is_active: true
      }])
      .select()
      .single()

    if (serviceError) {
      console.error('Service creation error:', serviceError)
      return { success: false, error: 'Failed to create service' }
    }

    // Revalidate relevant pages
    revalidatePath('/dashboard/provider/services')
    revalidatePath('/browse')

    return { success: true, data: service }

  } catch (error) {
    console.error('Service creation error:', error)

    if (error instanceof z.ZodError) {
      return {
        success: false,
        error: `Validation error: ${error.errors.map(e => e.message).join(', ')}`
      }
    }

    return { success: false, error: 'An unexpected error occurred' }
  }
}

export async function updateService(serviceId: string, input: Partial<ServiceInput>) {
  try {
    const supabase = await createServerClient()
    const { data: { user }, error: authError } = await supabase.auth.getUser()

    if (authError || !user) {
      return { success: false, error: 'Authentication required' }
    }

    // Verify ownership
    const { data: service, error: ownershipError } = await supabase
      .from('services')
      .select('provider_id')
      .eq('id', serviceId)
      .single()

    if (ownershipError || service?.provider_id !== user.id) {
      return { success: false, error: 'Service not found or access denied' }
    }

    // Update service
    const { data: updatedService, error: updateError } = await supabase
      .from('services')
      .update(input)
      .eq('id', serviceId)
      .eq('provider_id', user.id)
      .select()
      .single()

    if (updateError) {
      return { success: false, error: 'Failed to update service' }
    }

    revalidatePath('/dashboard/provider/services')
    revalidatePath('/browse')

    return { success: true, data: updatedService }

  } catch (error) {
    console.error('Service update error:', error)
    return { success: false, error: 'An unexpected error occurred' }
  }
}

export async function deleteService(serviceId: string) {
  try {
    const supabase = await createServerClient()
    const { data: { user }, error: authError } = await supabase.auth.getUser()

    if (authError || !user) {
      return { success: false, error: 'Authentication required' }
    }

    // Check for active bookings
    const { data: activeBookings, error: bookingsError } = await supabase
      .from('bookings')
      .select('id')
      .eq('service_id', serviceId)
      .in('status', ['pending', 'accepted', 'in_progress'])

    if (bookingsError) {
      return { success: false, error: 'Error checking active bookings' }
    }

    if (activeBookings && activeBookings.length > 0) {
      return {
        success: false,
        error: 'Cannot delete service with active bookings. Complete or cancel existing bookings first.'
      }
    }

    // Soft delete service
    const { error: deleteError } = await supabase
      .from('services')
      .update({
        is_active: false,
        deleted_at: new Date().toISOString()
      })
      .eq('id', serviceId)
      .eq('provider_id', user.id)

    if (deleteError) {
      return { success: false, error: 'Failed to delete service' }
    }

    revalidatePath('/dashboard/provider/services')
    revalidatePath('/browse')

    return { success: true }

  } catch (error) {
    console.error('Service deletion error:', error)
    return { success: false, error: 'An unexpected error occurred' }
  }
}
```

### Booking Creation Server Action

```typescript
// app/actions/bookings.ts
'use server'

import { revalidatePath } from 'next/cache'
import { z } from 'zod'
import { createServerClient } from '@/lib/supabase/server'

// Detroit Metro bounds validation from BUSINESS-LOGIC.md
const DETROIT_BOUNDS = {
  north: 42.6,
  south: 42.0,
  east: -82.8,
  west: -83.5,
  center: { lat: 42.3314, lng: -83.0458 },
  maxRadius: 25 // miles
}

const bookingSchema = z.object({
  service_id: z.string().uuid(),
  pickup_address: z.string().min(10),
  pickup_lat: z.number().min(DETROIT_BOUNDS.south).max(DETROIT_BOUNDS.north),
  pickup_lng: z.number().min(DETROIT_BOUNDS.west).max(DETROIT_BOUNDS.east),
  delivery_address: z.string().min(10),
  delivery_lat: z.number().min(DETROIT_BOUNDS.south).max(DETROIT_BOUNDS.north),
  delivery_lng: z.number().min(DETROIT_BOUNDS.west).max(DETROIT_BOUNDS.east),
  scheduled_datetime: z.string().datetime(),
  duration_hours: z.number().min(0.25).max(8),
  special_instructions: z.string().optional(),
  payload_description: z.string().min(3),
  estimated_weight_kg: z.number().min(0.1).max(5)
})

type BookingInput = z.infer<typeof bookingSchema>

// Calculate distance between two coordinates (Haversine formula)
function calculateDistance(lat1: number, lng1: number, lat2: number, lng2: number): number {
  const R = 3959 // Earth's radius in miles
  const dLat = (lat2 - lat1) * Math.PI / 180
  const dLng = (lng2 - lng1) * Math.PI / 180
  const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
    Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
    Math.sin(dLng/2) * Math.sin(dLng/2)
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a))
  return R * c
}

export async function createBooking(input: BookingInput) {
  try {
    // Validate input
    const validatedData = bookingSchema.parse(input)

    // Get authenticated user
    const supabase = await createServerClient()
    const { data: { user }, error: authError } = await supabase.auth.getUser()

    if (authError || !user) {
      return { success: false, error: 'Authentication required' }
    }

    // Get service details with provider info
    const { data: service, error: serviceError } = await supabase
      .from('services')
      .select(`
        *,
        provider:providers(*)
      `)
      .eq('id', validatedData.service_id)
      .eq('is_active', true)
      .single()

    if (serviceError || !service) {
      return { success: false, error: 'Service not found or unavailable' }
    }

    // Verify user is not booking their own service
    if (service.provider_id === user.id) {
      return { success: false, error: 'Cannot book your own service' }
    }

    // Validate locations are within Detroit Metro
    const pickupDistance = calculateDistance(
      DETROIT_BOUNDS.center.lat,
      DETROIT_BOUNDS.center.lng,
      validatedData.pickup_lat,
      validatedData.pickup_lng
    )

    const deliveryDistance = calculateDistance(
      DETROIT_BOUNDS.center.lat,
      DETROIT_BOUNDS.center.lng,
      validatedData.delivery_lat,
      validatedData.delivery_lng
    )

    if (pickupDistance > DETROIT_BOUNDS.maxRadius || deliveryDistance > DETROIT_BOUNDS.maxRadius) {
      return {
        success: false,
        error: 'Locations must be within 25 miles of Detroit Metro area'
      }
    }

    // Check service area radius
    const serviceDistance = Math.max(pickupDistance, deliveryDistance)
    if (serviceDistance > service.service_area_radius) {
      return {
        success: false,
        error: `Location outside service area (${service.service_area_radius} mile radius)`
      }
    }

    // Validate payload weight
    if (validatedData.estimated_weight_kg > service.max_payload_kg) {
      return {
        success: false,
        error: `Payload exceeds maximum capacity (${service.max_payload_kg}kg)`
      }
    }

    // Calculate total distance and price
    const tripDistance = calculateDistance(
      validatedData.pickup_lat,
      validatedData.pickup_lng,
      validatedData.delivery_lat,
      validatedData.delivery_lng
    )

    const totalPrice = (validatedData.duration_hours * service.price_per_hour) +
                      (tripDistance * 2) // $2 per mile

    // Create booking
    const { data: booking, error: bookingError } = await supabase
      .from('bookings')
      .insert([{
        consumer_id: user.id,
        service_id: validatedData.service_id,
        provider_id: service.provider_id,
        pickup_address: validatedData.pickup_address,
        pickup_lat: validatedData.pickup_lat,
        pickup_lng: validatedData.pickup_lng,
        delivery_address: validatedData.delivery_address,
        delivery_lat: validatedData.delivery_lat,
        delivery_lng: validatedData.delivery_lng,
        scheduled_datetime: validatedData.scheduled_datetime,
        duration_hours: validatedData.duration_hours,
        total_price: totalPrice,
        distance_miles: tripDistance,
        status: 'pending',
        special_instructions: validatedData.special_instructions,
        payload_description: validatedData.payload_description,
        estimated_weight_kg: validatedData.estimated_weight_kg
      }])
      .select(`
        *,
        service:services(*),
        provider:providers(*),
        consumer:consumers(*)
      `)
      .single()

    if (bookingError) {
      console.error('Booking creation error:', bookingError)
      return { success: false, error: 'Failed to create booking' }
    }

    // Revalidate relevant pages
    revalidatePath('/dashboard/consumer/bookings')
    revalidatePath('/dashboard/provider/bookings')

    return { success: true, data: booking }

  } catch (error) {
    console.error('Booking creation error:', error)

    if (error instanceof z.ZodError) {
      return {
        success: false,
        error: `Validation error: ${error.errors.map(e => e.message).join(', ')}`
      }
    }

    return { success: false, error: 'An unexpected error occurred' }
  }
}

export async function updateBookingStatus(bookingId: string, status: 'accepted' | 'in_progress' | 'completed' | 'cancelled') {
  try {
    const supabase = await createServerClient()
    const { data: { user }, error: authError } = await supabase.auth.getUser()

    if (authError || !user) {
      return { success: false, error: 'Authentication required' }
    }

    // Get booking with current status
    const { data: booking, error: bookingError } = await supabase
      .from('bookings')
      .select('*')
      .eq('id', bookingId)
      .single()

    if (bookingError || !booking) {
      return { success: false, error: 'Booking not found' }
    }

    // Verify user has permission to update
    if (booking.provider_id !== user.id && booking.consumer_id !== user.id) {
      return { success: false, error: 'Access denied' }
    }

    // Validate status transition based on BUSINESS-LOGIC.md state machine
    const validTransitions: Record<string, string[]> = {
      'pending': ['accepted', 'cancelled'],
      'accepted': ['in_progress', 'cancelled'],
      'in_progress': ['completed', 'cancelled'],
      'completed': [], // Terminal state
      'cancelled': [] // Terminal state
    }

    if (!validTransitions[booking.status]?.includes(status)) {
      return {
        success: false,
        error: `Invalid status transition from ${booking.status} to ${status}`
      }
    }

    // Only provider can accept/start, either party can cancel
    if ((status === 'accepted' || status === 'in_progress') && booking.provider_id !== user.id) {
      return { success: false, error: 'Only provider can accept or start bookings' }
    }

    // Update booking status
    const { data: updatedBooking, error: updateError } = await supabase
      .from('bookings')
      .update({
        status,
        ...(status === 'accepted' && { accepted_at: new Date().toISOString() }),
        ...(status === 'in_progress' && { started_at: new Date().toISOString() }),
        ...(status === 'completed' && { completed_at: new Date().toISOString() }),
        ...(status === 'cancelled' && { cancelled_at: new Date().toISOString() })
      })
      .eq('id', bookingId)
      .select()
      .single()

    if (updateError) {
      return { success: false, error: 'Failed to update booking status' }
    }

    revalidatePath('/dashboard/consumer/bookings')
    revalidatePath('/dashboard/provider/bookings')

    return { success: true, data: updatedBooking }

  } catch (error) {
    console.error('Booking status update error:', error)
    return { success: false, error: 'An unexpected error occurred' }
  }
}
```

## Verification Steps

### 1. Test Service Management
```bash
# Start development server
npm run dev

# Test service creation flow
# 1. Navigate to /dashboard/provider/services
# 2. Click "Add Service" button
# 3. Fill form with all 4 service categories
# 4. Verify validation rules for Detroit constraints
# 5. Test form submission and success handling

# Test service editing
# 1. Click edit button on existing service
# 2. Modify fields and save
# 3. Verify changes persist and pages revalidate

# Test service deletion
# 1. Try deleting service with active bookings (should fail)
# 2. Delete service without bookings (should succeed with soft delete)
```

### 2. Test Booking System
```bash
# Test booking creation
# 1. Browse services at /browse
# 2. Select service and click "Book Now"
# 3. Fill booking form with Detroit Metro addresses
# 4. Test geographic validation (try addresses outside Detroit)
# 5. Verify price calculation accuracy
# 6. Submit booking and verify creation

# Test booking status transitions
# 1. As provider, accept pending booking
# 2. Start accepted booking (in_progress)
# 3. Complete in_progress booking
# 4. Test invalid transitions (should fail)
```

### 3. Validate Business Logic Implementation
```bash
# Run validation commands
npm run lint
npx tsc --noEmit

# Test geographic constraints
# 1. Create service with > 25 mile radius (should fail)
# 2. Create booking outside Detroit Metro (should fail)
# 3. Verify coordinate boundaries are enforced

# Test service categories
# 1. Verify all 4 categories are available in forms
# 2. Test category-specific validation rules
# 3. Ensure proper categorization in listings
```

### 4. Database Integrity Checks
```bash
# In Supabase SQL Editor, verify:

-- Check RLS policies are active
SELECT schemaname, tablename, rowsecurity
FROM pg_tables
WHERE schemaname = 'public'
AND tablename IN ('services', 'bookings');

-- Verify proper foreign key constraints
SELECT
  tc.table_name,
  kcu.column_name,
  ccu.table_name AS foreign_table_name,
  ccu.column_name AS foreign_column_name
FROM
  information_schema.table_constraints AS tc
  JOIN information_schema.key_column_usage AS kcu
    ON tc.constraint_name = kcu.constraint_name
  JOIN information_schema.constraint_column_usage AS ccu
    ON ccu.constraint_name = tc.constraint_name
WHERE constraint_type = 'FOREIGN KEY'
AND tc.table_name IN ('services', 'bookings');

-- Test data integrity
SELECT
  COUNT(*) as total_services,
  COUNT(*) FILTER (WHERE is_active = true) as active_services,
  COUNT(DISTINCT category) as unique_categories
FROM services;
```

## Expected Outcome

After completing this step, you should have:

1. **Complete Service Management System**:
   - Service creation form with all 4 categories
   - Service editing and deletion functionality
   - Proper validation and error handling
   - Geographic constraint enforcement

2. **Robust Booking System**:
   - Booking creation with Detroit Metro validation
   - Status transition state machine
   - Price calculation based on distance and duration
   - Proper access control and security

3. **Production-Ready CRUD Operations**:
   - Server Actions with proper authentication
   - Type-safe database operations
   - Error handling and user feedback
   - Real-time page revalidation

4. **Business Logic Implementation**:
   - All specification requirements enforced
   - Geographic boundaries respected
   - Service category constraints applied
   - Proper state machine transitions

## Troubleshooting

### Common Issues

**Service Creation Fails with "Provider profile required"**
```typescript
// Ensure user has provider profile
const { data: provider } = await supabase
  .from('providers')
  .select('*')
  .eq('id', user.id)
  .single()

if (!provider) {
  // Redirect to provider onboarding
  router.push('/onboarding/provider')
}
```

**Geographic Validation Not Working**
```typescript
// Check coordinate bounds implementation
const isWithinDetroitMetro = (lat: number, lng: number) => {
  return lat >= 42.0 && lat <= 42.6 &&
         lng >= -83.5 && lng <= -82.8
}

// Verify distance calculation is correct
const distance = calculateDistance(centerLat, centerLng, lat, lng)
console.log(`Distance from center: ${distance} miles`)
```

**Database Permissions Issues**
```sql
-- Verify RLS policies in Supabase
SELECT * FROM pg_policies WHERE tablename = 'services';
SELECT * FROM pg_policies WHERE tablename = 'bookings';

-- Test policy with current user
SELECT * FROM services WHERE provider_id = auth.uid();
```

**Form Validation Errors**
```typescript
// Debug Zod schema validation
try {
  const result = serviceSchema.parse(formData)
  console.log('Validation passed:', result)
} catch (error) {
  if (error instanceof z.ZodError) {
    console.error('Validation errors:', error.errors)
  }
}
```

## Next Steps

Step 14 will implement the complete booking system UI with:
- Interactive booking flow (browse → select → configure → book)
- Advanced scheduling and availability management
- Real-time booking status updates
- Enhanced user experience with optimistic updates

The foundation built in this step provides the robust CRUD operations needed for the full booking experience in the next phase.