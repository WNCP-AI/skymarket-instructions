# Step 14: Booking System - Complete Flow Implementation

## Objective

Build a comprehensive booking system combining BUSINESS-LOGIC.md specifications with official documentation patterns. You'll implement the complete booking flow from service discovery to completion, including state management, scheduling, and real-time updates.

**Learning Focus**: Full-stack booking system implementation using Context7 for form libraries, UI patterns, and state management best practices.

## Context Setup

### Required Specifications
Load these project specifications for complete booking system requirements:

```bash
# Load booking business logic and state machine
@docs/specs/BUSINESS-LOGIC.md

# Load form validation and user input patterns
@docs/specs/FORMS.md

# Load UI component specifications
@docs/specs/UI-COMPONENTS.md

# Load API endpoint specifications for booking flow
@docs/specs/API.md
```

### Context7 Documentation Integration
Use Context7 to get up-to-date official documentation while implementing. Include "use context7" and reference specific library IDs when known:

**Example Integration Prompts:**

"Create multi-step React Hook Form with conditional fields, validation, and progress tracking for complex booking workflows. Reference /react-hook-form/react-hook-form for form patterns. use context7"

"Set up Supabase real-time subscriptions with React hooks for live booking status updates with proper cleanup and error handling. Reference /supabase/supabase for real-time patterns. use context7"

"Implement date/time handling with React components including timezone support and business hour validation. Reference /date-fns/date-fns for date utilities. use context7"

"Create React useReducer state machine patterns for booking flow management with proper state transitions. Reference React documentation for useReducer patterns. use context7"

"Build responsive booking interfaces using shadcn/ui dialog, sheet, and form components with accessibility features. Reference /shadcn-ui/ui for component patterns. use context7"

## Implementation Prompts

### Prompt 1: Multi-Step Booking Flow
"Using BUSINESS-LOGIC.md specifications, create a multi-step booking interface with:

1. Service selection with filtering by category and location
2. Scheduling component with provider availability checking
3. Location selection with Detroit Metro validation
4. Booking details and special instructions
5. Price calculation and confirmation
6. Real-time form validation and error handling

Create a multi-step React Hook Form with conditional fields, progress tracking, and real-time validation for complex booking workflows. Implement responsive booking interfaces using shadcn/ui components with accessibility features. use context7"

### Prompt 2: Booking State Management
"Using BUSINESS-LOGIC.md specifications, implement:

1. Complete booking status lifecycle (pending → accepted → in_progress → completed)
2. Real-time status updates using Supabase subscriptions
3. Optimistic UI updates with rollback on failure
4. Provider and consumer role-specific actions
5. Automated notifications for status changes
6. Proper error handling and retry mechanisms

Implement React useReducer state machine patterns for booking flow management with proper state transitions. Set up Supabase real-time subscriptions with React hooks for live booking status updates with proper cleanup and error handling. use context7"

## Code Examples

### Multi-Step Booking Flow Component

```typescript
// components/bookings/BookingWizard.tsx
'use client'

import { useState, useReducer, useCallback } from 'react'
import { useRouter } from 'next/navigation'
import { useForm, FormProvider } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { createBooking } from '@/app/actions/bookings'
import { Button } from '@/components/ui/button'
import { Progress } from '@/components/ui/progress'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { useToast } from '@/hooks/use-toast'
import { ServiceSelectionStep } from './steps/ServiceSelectionStep'
import { SchedulingStep } from './steps/SchedulingStep'
import { LocationStep } from './steps/LocationStep'
import { DetailsStep } from './steps/DetailsStep'
import { ConfirmationStep } from './steps/ConfirmationStep'
import type { Database } from '@/types/database'

type Service = Database['public']['Tables']['services']['Row'] & {
  provider: Database['public']['Tables']['providers']['Row']
}

// Booking flow state management
type BookingState = {
  currentStep: number
  selectedService: Service | null
  scheduledDateTime: string | null
  duration: number
  pickupLocation: {
    address: string
    lat: number
    lng: number
  } | null
  deliveryLocation: {
    address: string
    lat: number
    lng: number
  } | null
  payloadDetails: {
    description: string
    weight: number
  } | null
  specialInstructions: string
  totalPrice: number
}

type BookingAction =
  | { type: 'NEXT_STEP' }
  | { type: 'PREV_STEP' }
  | { type: 'SELECT_SERVICE'; service: Service }
  | { type: 'SET_SCHEDULE'; dateTime: string; duration: number }
  | { type: 'SET_PICKUP'; location: BookingState['pickupLocation'] }
  | { type: 'SET_DELIVERY'; location: BookingState['deliveryLocation'] }
  | { type: 'SET_PAYLOAD'; payload: BookingState['payloadDetails'] }
  | { type: 'SET_INSTRUCTIONS'; instructions: string }
  | { type: 'CALCULATE_PRICE'; price: number }
  | { type: 'RESET' }

const initialBookingState: BookingState = {
  currentStep: 0,
  selectedService: null,
  scheduledDateTime: null,
  duration: 1,
  pickupLocation: null,
  deliveryLocation: null,
  payloadDetails: null,
  specialInstructions: '',
  totalPrice: 0
}

function bookingReducer(state: BookingState, action: BookingAction): BookingState {
  switch (action.type) {
    case 'NEXT_STEP':
      return { ...state, currentStep: Math.min(state.currentStep + 1, 4) }
    case 'PREV_STEP':
      return { ...state, currentStep: Math.max(state.currentStep - 1, 0) }
    case 'SELECT_SERVICE':
      return { ...state, selectedService: action.service }
    case 'SET_SCHEDULE':
      return { ...state, scheduledDateTime: action.dateTime, duration: action.duration }
    case 'SET_PICKUP':
      return { ...state, pickupLocation: action.location }
    case 'SET_DELIVERY':
      return { ...state, deliveryLocation: action.location }
    case 'SET_PAYLOAD':
      return { ...state, payloadDetails: action.payload }
    case 'SET_INSTRUCTIONS':
      return { ...state, specialInstructions: action.instructions }
    case 'CALCULATE_PRICE':
      return { ...state, totalPrice: action.price }
    case 'RESET':
      return initialBookingState
    default:
      return state
  }
}

// Form validation schema
const bookingFormSchema = z.object({
  serviceId: z.string().uuid(),
  scheduledDateTime: z.string().datetime(),
  duration: z.number().min(0.25).max(8),
  pickupAddress: z.string().min(5),
  pickupLat: z.number().min(42.0).max(42.6),
  pickupLng: z.number().min(-83.5).max(-82.8),
  deliveryAddress: z.string().min(5),
  deliveryLat: z.number().min(42.0).max(42.6),
  deliveryLng: z.number().min(-83.5).max(-82.8),
  payloadDescription: z.string().min(3),
  estimatedWeight: z.number().min(0.1).max(5),
  specialInstructions: z.string().optional()
})

type BookingFormData = z.infer<typeof bookingFormSchema>

interface BookingWizardProps {
  services: Service[]
  onSuccess?: (bookingId: string) => void
}

export function BookingWizard({ services, onSuccess }: BookingWizardProps) {
  const [state, dispatch] = useReducer(bookingReducer, initialBookingState)
  const [isSubmitting, setIsSubmitting] = useState(false)
  const router = useRouter()
  const { toast } = useToast()

  const form = useForm<BookingFormData>({
    resolver: zodResolver(bookingFormSchema),
    mode: 'onChange'
  })

  const steps = [
    { title: 'Select Service', component: ServiceSelectionStep },
    { title: 'Schedule', component: SchedulingStep },
    { title: 'Locations', component: LocationStep },
    { title: 'Details', component: DetailsStep },
    { title: 'Confirm', component: ConfirmationStep }
  ]

  const currentStepComponent = steps[state.currentStep]?.component
  const progress = ((state.currentStep + 1) / steps.length) * 100

  const handleNext = useCallback(async () => {
    const isValid = await form.trigger()
    if (isValid) {
      dispatch({ type: 'NEXT_STEP' })
    }
  }, [form])

  const handlePrevious = useCallback(() => {
    dispatch({ type: 'PREV_STEP' })
  }, [])

  const handleSubmit = async (data: BookingFormData) => {
    if (!state.selectedService || !state.pickupLocation || !state.deliveryLocation) {
      toast({
        title: 'Error',
        description: 'Please complete all required steps',
        variant: 'destructive'
      })
      return
    }

    setIsSubmitting(true)

    try {
      const result = await createBooking({
        service_id: data.serviceId,
        pickup_address: data.pickupAddress,
        pickup_lat: data.pickupLat,
        pickup_lng: data.pickupLng,
        delivery_address: data.deliveryAddress,
        delivery_lat: data.deliveryLat,
        delivery_lng: data.deliveryLng,
        scheduled_datetime: data.scheduledDateTime,
        duration_hours: data.duration,
        special_instructions: data.specialInstructions,
        payload_description: data.payloadDescription,
        estimated_weight_kg: data.estimatedWeight
      })

      if (result.success) {
        toast({
          title: 'Booking Created',
          description: 'Your booking has been submitted and is awaiting provider acceptance.'
        })

        onSuccess?.(result.data.id)
        router.push('/dashboard/consumer/bookings')
      } else {
        toast({
          title: 'Booking Failed',
          description: result.error || 'Unable to create booking',
          variant: 'destructive'
        })
      }
    } catch (error) {
      console.error('Booking submission error:', error)
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
    <Card className="w-full max-w-4xl mx-auto">
      <CardHeader>
        <CardTitle>Book a Drone Service</CardTitle>
        <div className="space-y-2">
          <div className="flex justify-between text-sm text-muted-foreground">
            <span>Step {state.currentStep + 1} of {steps.length}</span>
            <span>{currentStepComponent ? steps[state.currentStep].title : ''}</span>
          </div>
          <Progress value={progress} />
        </div>
      </CardHeader>

      <CardContent>
        <FormProvider {...form}>
          <form onSubmit={form.handleSubmit(handleSubmit)} className="space-y-6">
            {currentStepComponent && (
              <currentStepComponent
                services={services}
                state={state}
                dispatch={dispatch}
                form={form}
              />
            )}

            <div className="flex justify-between pt-6">
              <Button
                type="button"
                variant="outline"
                onClick={handlePrevious}
                disabled={state.currentStep === 0}
              >
                Previous
              </Button>

              {state.currentStep < steps.length - 1 ? (
                <Button type="button" onClick={handleNext}>
                  Next
                </Button>
              ) : (
                <Button
                  type="submit"
                  disabled={isSubmitting || !form.formState.isValid}
                >
                  {isSubmitting ? 'Creating Booking...' : 'Confirm Booking'}
                </Button>
              )}
            </div>
          </form>
        </FormProvider>
      </CardContent>
    </Card>
  )
}
```

### Service Selection Step

```typescript
// components/bookings/steps/ServiceSelectionStep.tsx
'use client'

import { useCallback } from 'react'
import { UseFormReturn } from 'react-hook-form'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'
import { MapPin, Clock, DollarSign, Package } from 'lucide-react'
import type { BookingState, BookingAction } from '../BookingWizard'
import type { Database } from '@/types/database'

type Service = Database['public']['Tables']['services']['Row'] & {
  provider: Database['public']['Tables']['providers']['Row']
}

interface ServiceSelectionStepProps {
  services: Service[]
  state: BookingState
  dispatch: React.Dispatch<BookingAction>
  form: UseFormReturn<any>
}

const SERVICE_CATEGORIES = [
  { value: 'all', label: 'All Services' },
  { value: 'food_delivery', label: 'Food Delivery' },
  { value: 'courier', label: 'Courier Services' },
  { value: 'aerial_imaging', label: 'Aerial Imaging' },
  { value: 'site_mapping', label: 'Site Mapping' }
]

export function ServiceSelectionStep({ services, state, dispatch, form }: ServiceSelectionStepProps) {
  const [searchTerm, setSearchTerm] = useState('')
  const [selectedCategory, setSelectedCategory] = useState('all')

  // Filter services based on search and category
  const filteredServices = services.filter(service => {
    const matchesSearch = service.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
                         service.description.toLowerCase().includes(searchTerm.toLowerCase())
    const matchesCategory = selectedCategory === 'all' || service.category === selectedCategory
    return matchesSearch && matchesCategory && service.is_active
  })

  const handleServiceSelect = useCallback((service: Service) => {
    dispatch({ type: 'SELECT_SERVICE', service })
    form.setValue('serviceId', service.id)
  }, [dispatch, form])

  return (
    <div className="space-y-6">
      {/* Search and Filter */}
      <div className="flex flex-col sm:flex-row gap-4">
        <div className="flex-1">
          <Input
            placeholder="Search services..."
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
          />
        </div>
        <div className="sm:w-48">
          <Select value={selectedCategory} onValueChange={setSelectedCategory}>
            <SelectTrigger>
              <SelectValue placeholder="Filter by category" />
            </SelectTrigger>
            <SelectContent>
              {SERVICE_CATEGORIES.map((category) => (
                <SelectItem key={category.value} value={category.value}>
                  {category.label}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>
      </div>

      {/* Services Grid */}
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {filteredServices.map((service) => (
          <Card
            key={service.id}
            className={`cursor-pointer transition-all duration-200 hover:shadow-md ${
              state.selectedService?.id === service.id
                ? 'ring-2 ring-primary border-primary'
                : ''
            }`}
            onClick={() => handleServiceSelect(service)}
          >
            <CardHeader className="pb-3">
              <div className="flex items-start justify-between">
                <CardTitle className="text-lg line-clamp-2">{service.title}</CardTitle>
                <Badge variant="outline" className="ml-2">
                  {service.category.replace('_', ' ')}
                </Badge>
              </div>
            </CardHeader>

            <CardContent className="space-y-4">
              <p className="text-sm text-muted-foreground line-clamp-3">
                {service.description}
              </p>

              <div className="space-y-2">
                <div className="flex items-center text-sm">
                  <DollarSign className="h-4 w-4 mr-2 text-green-600" />
                  <span className="font-medium">${service.price_per_hour}/hour</span>
                </div>

                <div className="flex items-center text-sm">
                  <MapPin className="h-4 w-4 mr-2 text-blue-600" />
                  <span>{service.service_area_radius} mile radius</span>
                </div>

                <div className="flex items-center text-sm">
                  <Package className="h-4 w-4 mr-2 text-orange-600" />
                  <span>Max {service.max_payload_kg}kg payload</span>
                </div>

                <div className="flex items-center text-sm">
                  <Clock className="h-4 w-4 mr-2 text-purple-600" />
                  <span>Up to {service.flight_time_minutes}min flight time</span>
                </div>
              </div>

              {/* Provider Info */}
              <div className="pt-3 border-t">
                <div className="flex items-center space-x-3">
                  {service.provider.avatar_url ? (
                    <img
                      src={service.provider.avatar_url}
                      alt={service.provider.name}
                      className="h-8 w-8 rounded-full"
                    />
                  ) : (
                    <div className="h-8 w-8 rounded-full bg-muted flex items-center justify-center">
                      <span className="text-xs font-medium">
                        {service.provider.name.charAt(0)}
                      </span>
                    </div>
                  )}
                  <div>
                    <p className="text-sm font-medium">{service.provider.name}</p>
                    <p className="text-xs text-muted-foreground">
                      {service.provider.part_107_certified ? 'Part 107 Certified' : 'Hobbyist'}
                    </p>
                  </div>
                </div>
              </div>

              {state.selectedService?.id === service.id && (
                <Button className="w-full mt-4" size="sm">
                  Selected
                </Button>
              )}
            </CardContent>
          </Card>
        ))}
      </div>

      {filteredServices.length === 0 && (
        <div className="text-center py-12">
          <p className="text-muted-foreground">
            No services found matching your criteria.
          </p>
          <Button
            variant="outline"
            onClick={() => {
              setSearchTerm('')
              setSelectedCategory('all')
            }}
            className="mt-4"
          >
            Clear Filters
          </Button>
        </div>
      )}
    </div>
  )
}
```

### Real-Time Booking Status Component

```typescript
// components/bookings/BookingStatusCard.tsx
'use client'

import { useState, useEffect, useCallback } from 'react'
import { useRouter } from 'next/navigation'
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { Progress } from '@/components/ui/progress'
import { useToast } from '@/hooks/use-toast'
import { updateBookingStatus } from '@/app/actions/bookings'
import { MapPin, Clock, Package, User, MessageCircle } from 'lucide-react'
import type { Database } from '@/types/database'

type Booking = Database['public']['Tables']['bookings']['Row'] & {
  service: Database['public']['Tables']['services']['Row']
  provider: Database['public']['Tables']['providers']['Row']
  consumer: Database['public']['Tables']['consumers']['Row']
}

interface BookingStatusCardProps {
  booking: Booking
  currentUserId: string
  userRole: 'provider' | 'consumer'
}

// Status configuration
const STATUS_CONFIG = {
  pending: {
    label: 'Pending',
    color: 'bg-yellow-500',
    variant: 'secondary' as const,
    progress: 25,
    description: 'Waiting for provider acceptance'
  },
  accepted: {
    label: 'Accepted',
    color: 'bg-blue-500',
    variant: 'default' as const,
    progress: 50,
    description: 'Service confirmed, preparing for start'
  },
  in_progress: {
    label: 'In Progress',
    color: 'bg-green-500',
    variant: 'default' as const,
    progress: 75,
    description: 'Service is currently being performed'
  },
  completed: {
    label: 'Completed',
    color: 'bg-emerald-500',
    variant: 'default' as const,
    progress: 100,
    description: 'Service completed successfully'
  },
  cancelled: {
    label: 'Cancelled',
    color: 'bg-red-500',
    variant: 'destructive' as const,
    progress: 0,
    description: 'Booking was cancelled'
  }
}

export function BookingStatusCard({ booking: initialBooking, currentUserId, userRole }: BookingStatusCardProps) {
  const [booking, setBooking] = useState(initialBooking)
  const [isUpdating, setIsUpdating] = useState(false)
  const [connectionStatus, setConnectionStatus] = useState<'connecting' | 'connected' | 'error'>('connecting')

  const router = useRouter()
  const { toast } = useToast()
  const supabase = createClientComponentClient<Database>()

  // Real-time subscription to booking updates
  useEffect(() => {
    const channel = supabase
      .channel(`booking_${booking.id}`)
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'bookings',
          filter: `id=eq.${booking.id}`
        },
        (payload) => {
          console.log('Booking update received:', payload)
          setBooking(prev => ({
            ...prev,
            ...payload.new
          }))

          // Show notification for status changes
          if (payload.old.status !== payload.new.status) {
            const statusConfig = STATUS_CONFIG[payload.new.status as keyof typeof STATUS_CONFIG]
            toast({
              title: 'Booking Update',
              description: `Status changed to: ${statusConfig.label}`,
            })
          }
        }
      )
      .subscribe((status) => {
        if (status === 'SUBSCRIBED') {
          setConnectionStatus('connected')
        } else if (status === 'CHANNEL_ERROR') {
          setConnectionStatus('error')
        }
      })

    return () => {
      supabase.removeChannel(channel)
    }
  }, [booking.id, supabase, toast])

  const handleStatusUpdate = useCallback(async (newStatus: 'accepted' | 'in_progress' | 'completed' | 'cancelled') => {
    setIsUpdating(true)

    try {
      const result = await updateBookingStatus(booking.id, newStatus)

      if (result.success) {
        // Optimistic update - real-time subscription will handle the actual update
        setBooking(prev => ({
          ...prev,
          status: newStatus,
          ...(newStatus === 'accepted' && { accepted_at: new Date().toISOString() }),
          ...(newStatus === 'in_progress' && { started_at: new Date().toISOString() }),
          ...(newStatus === 'completed' && { completed_at: new Date().toISOString() }),
          ...(newStatus === 'cancelled' && { cancelled_at: new Date().toISOString() })
        }))

        toast({
          title: 'Status Updated',
          description: `Booking ${newStatus.replace('_', ' ')}`,
        })

        // Refresh the page to ensure data consistency
        router.refresh()
      } else {
        toast({
          title: 'Update Failed',
          description: result.error || 'Unable to update booking status',
          variant: 'destructive'
        })
      }
    } catch (error) {
      console.error('Status update error:', error)
      toast({
        title: 'Error',
        description: 'An unexpected error occurred',
        variant: 'destructive'
      })
    } finally {
      setIsUpdating(false)
    }
  }, [booking.id, router, toast])

  const statusConfig = STATUS_CONFIG[booking.status as keyof typeof STATUS_CONFIG]
  const canUpdateStatus = userRole === 'provider' && booking.provider_id === currentUserId
  const canCancel = booking.consumer_id === currentUserId || booking.provider_id === currentUserId

  // Available actions based on current status and user role
  const getAvailableActions = () => {
    const actions: Array<{ label: string; status: string; variant: 'default' | 'destructive' }> = []

    if (booking.status === 'pending' && canUpdateStatus) {
      actions.push({ label: 'Accept', status: 'accepted', variant: 'default' })
    }

    if (booking.status === 'accepted' && canUpdateStatus) {
      actions.push({ label: 'Start Service', status: 'in_progress', variant: 'default' })
    }

    if (booking.status === 'in_progress' && canUpdateStatus) {
      actions.push({ label: 'Mark Complete', status: 'completed', variant: 'default' })
    }

    if (['pending', 'accepted', 'in_progress'].includes(booking.status) && canCancel) {
      actions.push({ label: 'Cancel', status: 'cancelled', variant: 'destructive' })
    }

    return actions
  }

  const availableActions = getAvailableActions()

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle className="flex items-center space-x-3">
            <span>{booking.service.title}</span>
            <Badge variant={statusConfig.variant}>
              {statusConfig.label}
            </Badge>
          </CardTitle>

          {connectionStatus === 'connected' && (
            <div className="flex items-center text-xs text-green-600">
              <div className="w-2 h-2 bg-green-500 rounded-full mr-2 animate-pulse" />
              Live
            </div>
          )}
        </div>

        <div className="space-y-2">
          <p className="text-sm text-muted-foreground">{statusConfig.description}</p>
          <Progress value={statusConfig.progress} className="h-2" />
        </div>
      </CardHeader>

      <CardContent className="space-y-6">
        {/* Booking Details */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div className="space-y-3">
            <div className="flex items-center text-sm">
              <Clock className="h-4 w-4 mr-2 text-muted-foreground" />
              <span>
                {new Date(booking.scheduled_datetime).toLocaleString()}
              </span>
            </div>

            <div className="flex items-center text-sm">
              <Package className="h-4 w-4 mr-2 text-muted-foreground" />
              <span>{booking.payload_description} ({booking.estimated_weight_kg}kg)</span>
            </div>

            <div className="flex items-center text-sm">
              <User className="h-4 w-4 mr-2 text-muted-foreground" />
              <span>
                {userRole === 'provider' ? booking.consumer.name : booking.provider.name}
              </span>
            </div>
          </div>

          <div className="space-y-3">
            <div className="flex items-start text-sm">
              <MapPin className="h-4 w-4 mr-2 mt-0.5 text-muted-foreground" />
              <div className="space-y-1">
                <p className="font-medium">Pickup:</p>
                <p className="text-muted-foreground">{booking.pickup_address}</p>
              </div>
            </div>

            <div className="flex items-start text-sm">
              <MapPin className="h-4 w-4 mr-2 mt-0.5 text-muted-foreground" />
              <div className="space-y-1">
                <p className="font-medium">Delivery:</p>
                <p className="text-muted-foreground">{booking.delivery_address}</p>
              </div>
            </div>
          </div>
        </div>

        {/* Pricing */}
        <div className="flex justify-between items-center py-3 border-t">
          <span className="font-medium">Total Price:</span>
          <span className="text-lg font-bold">${booking.total_price}</span>
        </div>

        {/* Special Instructions */}
        {booking.special_instructions && (
          <div className="p-3 bg-muted rounded-lg">
            <p className="text-sm font-medium mb-1">Special Instructions:</p>
            <p className="text-sm text-muted-foreground">{booking.special_instructions}</p>
          </div>
        )}

        {/* Action Buttons */}
        {availableActions.length > 0 && (
          <div className="flex flex-wrap gap-3 pt-4 border-t">
            {availableActions.map((action) => (
              <Button
                key={action.status}
                variant={action.variant}
                size="sm"
                disabled={isUpdating}
                onClick={() => handleStatusUpdate(action.status as any)}
              >
                {isUpdating ? 'Updating...' : action.label}
              </Button>
            ))}

            <Button variant="outline" size="sm">
              <MessageCircle className="h-4 w-4 mr-2" />
              Message
            </Button>
          </div>
        )}

        {/* Timestamps */}
        <div className="text-xs text-muted-foreground space-y-1 pt-4 border-t">
          <p>Created: {new Date(booking.created_at).toLocaleString()}</p>
          {booking.accepted_at && (
            <p>Accepted: {new Date(booking.accepted_at).toLocaleString()}</p>
          )}
          {booking.started_at && (
            <p>Started: {new Date(booking.started_at).toLocaleString()}</p>
          )}
          {booking.completed_at && (
            <p>Completed: {new Date(booking.completed_at).toLocaleString()}</p>
          )}
          {booking.cancelled_at && (
            <p>Cancelled: {new Date(booking.cancelled_at).toLocaleString()}</p>
          )}
        </div>
      </CardContent>
    </Card>
  )
}
```

### Booking Management Hook

```typescript
// hooks/useBookings.ts
'use client'

import { useState, useEffect, useCallback } from 'react'
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import type { Database } from '@/types/database'

type Booking = Database['public']['Tables']['bookings']['Row'] & {
  service: Database['public']['Tables']['services']['Row']
  provider: Database['public']['Tables']['providers']['Row']
  consumer: Database['public']['Tables']['consumers']['Row']
}

export function useBookings(userRole: 'provider' | 'consumer', userId: string) {
  const [bookings, setBookings] = useState<Booking[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  const supabase = createClientComponentClient<Database>()

  // Fetch bookings based on user role
  const fetchBookings = useCallback(async () => {
    try {
      setLoading(true)
      setError(null)

      const query = supabase
        .from('bookings')
        .select(`
          *,
          service:services(*),
          provider:providers(*),
          consumer:consumers(*)
        `)

      // Filter based on user role
      if (userRole === 'provider') {
        query.eq('provider_id', userId)
      } else {
        query.eq('consumer_id', userId)
      }

      const { data, error: fetchError } = await query
        .order('created_at', { ascending: false })

      if (fetchError) throw fetchError

      setBookings(data || [])
    } catch (err) {
      console.error('Error fetching bookings:', err)
      setError(err instanceof Error ? err.message : 'Failed to fetch bookings')
    } finally {
      setLoading(false)
    }
  }, [supabase, userRole, userId])

  // Set up real-time subscription
  useEffect(() => {
    fetchBookings()

    // Subscribe to booking changes
    const channel = supabase
      .channel('bookings_changes')
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'bookings',
          filter: userRole === 'provider' ? `provider_id=eq.${userId}` : `consumer_id=eq.${userId}`
        },
        (payload) => {
          console.log('Booking change:', payload)

          if (payload.eventType === 'INSERT') {
            // Fetch the complete booking data with relationships
            fetchBookings()
          } else if (payload.eventType === 'UPDATE') {
            setBookings(prev => prev.map(booking =>
              booking.id === payload.new.id
                ? { ...booking, ...payload.new }
                : booking
            ))
          } else if (payload.eventType === 'DELETE') {
            setBookings(prev => prev.filter(booking => booking.id !== payload.old.id))
          }
        }
      )
      .subscribe()

    return () => {
      supabase.removeChannel(channel)
    }
  }, [fetchBookings, supabase, userId, userRole])

  // Helper functions
  const getBookingsByStatus = useCallback((status: string) => {
    return bookings.filter(booking => booking.status === status)
  }, [bookings])

  const getActiveBookings = useCallback(() => {
    return bookings.filter(booking =>
      ['pending', 'accepted', 'in_progress'].includes(booking.status)
    )
  }, [bookings])

  const getCompletedBookings = useCallback(() => {
    return bookings.filter(booking => booking.status === 'completed')
  }, [bookings])

  const refresh = useCallback(() => {
    fetchBookings()
  }, [fetchBookings])

  return {
    bookings,
    loading,
    error,
    getBookingsByStatus,
    getActiveBookings,
    getCompletedBookings,
    refresh
  }
}
```

## Verification Steps

### 1. Test Complete Booking Flow
```bash
# Start development server
npm run dev

# Test booking creation flow
# 1. Navigate to /browse or /book
# 2. Go through complete booking wizard:
#    - Select service from filtered list
#    - Choose date/time scheduling
#    - Set pickup and delivery locations
#    - Add payload details and instructions
#    - Confirm booking and price calculation

# Test form validation
# 1. Try submitting incomplete forms (should show validation errors)
# 2. Test Detroit Metro location validation
# 3. Verify payload weight constraints
# 4. Test special instructions handling
```

### 2. Test Booking State Management
```bash
# Test booking status transitions
# 1. Create booking as consumer (starts as 'pending')
# 2. Switch to provider account and accept booking
# 3. Start the service (change to 'in_progress')
# 4. Complete the service
# 5. Verify all state transitions work correctly

# Test invalid transitions
# 1. Try to complete a pending booking (should fail)
# 2. Try to accept a completed booking (should fail)
# 3. Verify proper error handling
```

### 3. Test Real-Time Features
```bash
# Test real-time updates
# 1. Open booking in two browser windows (provider and consumer)
# 2. Update status in one window
# 3. Verify immediate update in other window
# 4. Check toast notifications appear
# 5. Test connection status indicator

# Test subscription cleanup
# 1. Navigate between pages
# 2. Verify no memory leaks or duplicate subscriptions
# 3. Check browser console for WebSocket connections
```

### 4. Validate Business Logic Integration
```bash
# Run validation commands
npm run lint
npx tsc --noEmit

# Test geographic constraints
# 1. Create booking with locations outside Detroit Metro
# 2. Verify proper error messages and validation
# 3. Test service area radius enforcement

# Test pricing calculations
# 1. Verify hourly rate calculations
# 2. Test distance-based pricing
# 3. Check special fee handling
```

## Expected Outcome

After completing this step, you should have:

1. **Complete Booking System**:
   - Multi-step booking wizard with intuitive flow
   - Service selection with filtering and search
   - Scheduling component with availability checking
   - Location selection with map integration
   - Real-time price calculation and confirmation

2. **Advanced State Management**:
   - Proper booking state machine implementation
   - Real-time status updates via Supabase subscriptions
   - Optimistic UI updates with error handling
   - Role-based action permissions

3. **Production-Ready Components**:
   - Responsive design for all screen sizes
   - Accessible form controls and interactions
   - Comprehensive error handling and user feedback
   - Loading states and connection indicators

4. **Business Logic Integration**:
   - All BUSINESS-LOGIC.md requirements implemented
   - Geographic constraints properly enforced
   - Service category validation working
   - Proper state transition validation

## Troubleshooting

### Common Issues

**Real-Time Updates Not Working**
```typescript
// Check WebSocket connection status
const channel = supabase.channel('test')
  .subscribe((status) => {
    console.log('Channel status:', status)
  })

// Verify RLS policies allow real-time access
// In Supabase SQL Editor:
SELECT * FROM pg_policies WHERE tablename = 'bookings';
```

**Form Validation Errors**
```typescript
// Debug React Hook Form validation
const { errors, isValid } = form.formState
console.log('Form errors:', errors)
console.log('Form is valid:', isValid)

// Check Zod schema validation
try {
  bookingSchema.parse(formData)
} catch (error) {
  console.error('Validation failed:', error.errors)
}
```

**Booking State Transitions Failing**
```typescript
// Verify state machine logic
const validTransitions: Record<string, string[]> = {
  'pending': ['accepted', 'cancelled'],
  'accepted': ['in_progress', 'cancelled'],
  'in_progress': ['completed', 'cancelled'],
  'completed': [],
  'cancelled': []
}

console.log('Valid next states:', validTransitions[currentStatus])
```

**Geographic Validation Issues**
```typescript
// Test coordinate bounds
const DETROIT_BOUNDS = {
  north: 42.6, south: 42.0,
  east: -82.8, west: -83.5
}

function isInDetroitMetro(lat: number, lng: number): boolean {
  return lat >= DETROIT_BOUNDS.south && lat <= DETROIT_BOUNDS.north &&
         lng >= DETROIT_BOUNDS.west && lng <= DETROIT_BOUNDS.east
}
```

## Next Steps

Step 15 will integrate interactive maps with:
- Visual service area representation
- Interactive location selection
- Real-time tracking display
- Geographic constraint visualization
- Enhanced user experience with map-based booking

The booking system foundation built here provides the data flow and state management needed for the visual map experience in the next step.