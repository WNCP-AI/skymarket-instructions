# Step 15 - Booking System Implementation

> **Build the core booking system that allows consumers to book services and providers to manage orders**

## Objective
Implement a comprehensive booking system with service selection, scheduling, location input, and status management that forms the heart of the SkyMarket marketplace.

## What You'll Learn
- Complex form handling with multi-step workflows
- Date and time scheduling with availability checking
- Location-based service matching and validation
- Real-time booking status management
- Integration between multiple database tables

## Step-by-Step Instructions

### 1. Create Booking Types and Utilities

Create `src/types/booking.ts`:
```typescript
export type BookingStatus = 'pending' | 'accepted' | 'in_progress' | 'completed' | 'cancelled'
export type PaymentStatus = 'pending' | 'paid' | 'failed' | 'refunded'
export type ServiceCategory = 'food_delivery' | 'courier' | 'aerial_imaging' | 'site_mapping'

export interface Booking {
  id: string
  consumer_id: string
  listing_id: string
  provider_id: string
  
  // Service details
  scheduled_at: string
  duration_minutes?: number
  pickup_address?: string
  pickup_lat?: number
  pickup_lng?: number
  delivery_address?: string
  delivery_lat?: number
  delivery_lng?: number
  special_instructions?: string
  
  // Status and pricing
  status: BookingStatus
  total_amount: number
  service_fee: number
  
  // Payment
  stripe_payment_intent_id?: string
  payment_status: PaymentStatus
  
  created_at: string
  updated_at: string
}

export interface BookingFormData {
  listingId: string
  scheduledAt: Date
  pickupAddress: string
  deliveryAddress: string
  specialInstructions: string
  estimatedDuration: number
}

export interface LocationCoordinates {
  lat: number
  lng: number
  address: string
}
```

### 2. Create Location Services

Create `src/lib/location.ts`:
```typescript
// Detroit Metro coordinates for service area validation
export const DETROIT_CENTER = { lat: 42.3314, lng: -83.0458 }
export const MAX_SERVICE_RADIUS = 25 // miles

export interface GeocodeResult {
  lat: number
  lng: number
  address: string
}

// Calculate distance between two coordinates using Haversine formula
export function calculateDistance(
  coord1: { lat: number; lng: number },
  coord2: { lat: number; lng: number }
): number {
  const R = 3959 // Earth's radius in miles
  const dLat = toRadians(coord2.lat - coord1.lat)
  const dLng = toRadians(coord2.lng - coord1.lng)
  
  const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRadians(coord1.lat)) * Math.cos(toRadians(coord2.lat)) *
    Math.sin(dLng / 2) * Math.sin(dLng / 2)
  
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a))
  return R * c
}

function toRadians(degrees: number): number {
  return degrees * (Math.PI / 180)
}

export function isWithinServiceArea(coordinates: { lat: number; lng: number }): boolean {
  const distance = calculateDistance(DETROIT_CENTER, coordinates)
  return distance <= MAX_SERVICE_RADIUS
}

// Mock geocoding function - in production, use Mapbox Geocoding API
export async function geocodeAddress(address: string): Promise<GeocodeResult | null> {
  try {
    // This would make a real API call to Mapbox in production
    const response = await fetch(`https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(address)}.json?access_token=${process.env.NEXT_PUBLIC_MAPBOX_TOKEN}&country=US&proximity=${DETROIT_CENTER.lng},${DETROIT_CENTER.lat}`)
    
    if (!response.ok) return null
    
    const data = await response.json()
    if (!data.features || data.features.length === 0) return null
    
    const [lng, lat] = data.features[0].center
    const formattedAddress = data.features[0].place_name
    
    return {
      lat,
      lng,
      address: formattedAddress
    }
  } catch (error) {
    console.error('Geocoding error:', error)
    return null
  }
}

export function calculateBookingPrice(
  listing: { base_price: number; price_per_hour?: number; price_per_mile?: number },
  distance: number,
  duration: number // in minutes
): { subtotal: number; serviceFee: number; total: number } {
  let subtotal = listing.base_price
  
  // Add hourly rate if applicable
  if (listing.price_per_hour && duration > 0) {
    const hours = duration / 60
    subtotal += listing.price_per_hour * hours
  }
  
  // Add mileage rate if applicable
  if (listing.price_per_mile && distance > 0) {
    subtotal += listing.price_per_mile * distance
  }
  
  // Calculate service fee (15% of subtotal, minimum $2)
  const serviceFee = Math.max(subtotal * 0.15, 2)
  const total = subtotal + serviceFee
  
  return {
    subtotal: Math.round(subtotal * 100) / 100,
    serviceFee: Math.round(serviceFee * 100) / 100,
    total: Math.round(total * 100) / 100
  }
}
```

### 3. Create Booking Database Operations

Create `src/lib/supabase/bookings.ts`:
```typescript
import { createClient } from './client'
import type { Booking, BookingFormData } from '@/types/booking'
import { calculateBookingPrice, geocodeAddress, isWithinServiceArea } from '@/lib/location'

export async function createBooking(
  formData: BookingFormData,
  userId: string
): Promise<{ booking?: Booking; error?: string }> {
  try {
    const supabase = createClient()
    
    // Get listing details
    const { data: listing, error: listingError } = await supabase
      .from('listings')
      .select(`
        *,
        providers (
          id,
          user_id,
          business_name
        )
      `)
      .eq('id', formData.listingId)
      .eq('active', true)
      .single()
    
    if (listingError || !listing) {
      return { error: 'Service listing not found' }
    }
    
    // Geocode pickup and delivery addresses
    const pickupCoords = await geocodeAddress(formData.pickupAddress)
    const deliveryCoords = await geocodeAddress(formData.deliveryAddress)
    
    if (!pickupCoords || !deliveryCoords) {
      return { error: 'Could not locate provided addresses' }
    }
    
    // Validate service area
    if (!isWithinServiceArea(pickupCoords) || !isWithinServiceArea(deliveryCoords)) {
      return { error: 'Service area is limited to Detroit Metro (25 mile radius)' }
    }
    
    // Calculate distance and pricing
    const distance = calculateDistance(pickupCoords, deliveryCoords)
    const pricing = calculateBookingPrice(listing, distance, formData.estimatedDuration)
    
    // Create booking
    const bookingData = {
      consumer_id: userId,
      listing_id: formData.listingId,
      provider_id: listing.providers.id,
      scheduled_at: formData.scheduledAt.toISOString(),
      duration_minutes: formData.estimatedDuration,
      pickup_address: pickupCoords.address,
      pickup_lat: pickupCoords.lat,
      pickup_lng: pickupCoords.lng,
      delivery_address: deliveryCoords.address,
      delivery_lat: deliveryCoords.lat,
      delivery_lng: deliveryCoords.lng,
      special_instructions: formData.specialInstructions || null,
      status: 'pending' as const,
      total_amount: pricing.total,
      service_fee: pricing.serviceFee,
      payment_status: 'pending' as const
    }
    
    const { data: booking, error: bookingError } = await supabase
      .from('bookings')
      .insert([bookingData])
      .select()
      .single()
    
    if (bookingError) {
      console.error('Booking creation error:', bookingError)
      return { error: 'Failed to create booking' }
    }
    
    return { booking }
    
  } catch (error) {
    console.error('Create booking error:', error)
    return { error: 'Failed to create booking' }
  }
}

export async function getUserBookings(userId: string, role: 'consumer' | 'provider') {
  const supabase = createClient()
  
  let query = supabase
    .from('bookings')
    .select(`
      *,
      listings (
        title,
        category,
        providers (
          business_name,
          profiles (
            full_name
          )
        )
      )
    `)
  
  if (role === 'consumer') {
    query = query.eq('consumer_id', userId)
  } else {
    // For providers, join through their provider profile
    query = query.in('provider_id', 
      supabase.from('providers').select('id').eq('user_id', userId)
    )
  }
  
  return query.order('created_at', { ascending: false })
}

export async function updateBookingStatus(
  bookingId: string,
  status: BookingStatus,
  userId: string
): Promise<{ success: boolean; error?: string }> {
  try {
    const supabase = createClient()
    
    // Verify user can update this booking
    const { data: booking } = await supabase
      .from('bookings')
      .select(`
        *,
        providers (user_id)
      `)
      .eq('id', bookingId)
      .single()
    
    if (!booking) {
      return { success: false, error: 'Booking not found' }
    }
    
    const isConsumer = booking.consumer_id === userId
    const isProvider = booking.providers.user_id === userId
    
    if (!isConsumer && !isProvider) {
      return { success: false, error: 'Unauthorized' }
    }
    
    // Validate status transitions
    const validTransitions = {
      pending: ['accepted', 'cancelled'],
      accepted: ['in_progress', 'cancelled'],
      in_progress: ['completed', 'cancelled'],
      completed: [],
      cancelled: []
    }
    
    if (!validTransitions[booking.status].includes(status)) {
      return { success: false, error: 'Invalid status transition' }
    }
    
    // Only providers can accept/start bookings, consumers can cancel
    if (status === 'accepted' || status === 'in_progress') {
      if (!isProvider) {
        return { success: false, error: 'Only providers can update booking status' }
      }
    }
    
    const { error } = await supabase
      .from('bookings')
      .update({ status, updated_at: new Date().toISOString() })
      .eq('id', bookingId)
    
    if (error) {
      return { success: false, error: 'Failed to update booking' }
    }
    
    return { success: true }
    
  } catch (error) {
    console.error('Update booking status error:', error)
    return { success: false, error: 'Failed to update booking' }
  }
}
```

### 4. Create Booking Form Components

Create `src/components/booking/booking-form.tsx`:
```typescript
'use client'

import { useState, useEffect } from 'react'
import { useRouter } from 'next/navigation'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { createClient } from '@/lib/supabase/client'
import { createBooking } from '@/lib/supabase/bookings'
import { calculateBookingPrice, geocodeAddress } from '@/lib/location'
import type { BookingFormData } from '@/types/booking'

const bookingSchema = z.object({
  scheduledAt: z.date().min(new Date(), 'Booking must be in the future'),
  pickupAddress: z.string().min(5, 'Pickup address is required'),
  deliveryAddress: z.string().min(5, 'Delivery address is required'),
  specialInstructions: z.string().optional(),
  estimatedDuration: z.number().min(15).max(480) // 15 minutes to 8 hours
})

interface BookingFormProps {
  listingId: string
  listing: {
    title: string
    base_price: number
    price_per_hour?: number
    price_per_mile?: number
    duration_minutes?: number
  }
}

export function BookingForm({ listingId, listing }: BookingFormProps) {
  const router = useRouter()
  const [isSubmitting, setIsSubmitting] = useState(false)
  const [pricingEstimate, setPricingEstimate] = useState<{
    subtotal: number
    serviceFee: number
    total: number
  } | null>(null)
  
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors }
  } = useForm<z.infer<typeof bookingSchema>>({
    resolver: zodResolver(bookingSchema),
    defaultValues: {
      estimatedDuration: listing.duration_minutes || 60
    }
  })
  
  const pickupAddress = watch('pickupAddress')
  const deliveryAddress = watch('deliveryAddress')
  const estimatedDuration = watch('estimatedDuration')
  
  // Calculate pricing estimate when addresses change
  useEffect(() => {
    const calculateEstimate = async () => {
      if (pickupAddress && deliveryAddress) {
        try {
          const pickupCoords = await geocodeAddress(pickupAddress)
          const deliveryCoords = await geocodeAddress(deliveryAddress)
          
          if (pickupCoords && deliveryCoords) {
            const distance = calculateDistance(pickupCoords, deliveryCoords)
            const pricing = calculateBookingPrice(listing, distance, estimatedDuration)
            setPricingEstimate(pricing)
          }
        } catch (error) {
          console.error('Pricing calculation error:', error)
        }
      }
    }
    
    const timeoutId = setTimeout(calculateEstimate, 500)
    return () => clearTimeout(timeoutId)
  }, [pickupAddress, deliveryAddress, estimatedDuration, listing])
  
  const onSubmit = async (data: z.infer<typeof bookingSchema>) => {
    setIsSubmitting(true)
    
    try {
      const supabase = createClient()
      const { data: { user } } = await supabase.auth.getUser()
      
      if (!user) {
        router.push('/auth/login')
        return
      }
      
      const formData: BookingFormData = {
        listingId,
        ...data
      }
      
      const result = await createBooking(formData, user.id)
      
      if (result.error) {
        alert(result.error)
      } else if (result.booking) {
        // Redirect to payment page
        router.push(`/booking/${result.booking.id}/payment`)
      }
      
    } catch (error) {
      console.error('Booking submission error:', error)
      alert('Failed to create booking. Please try again.')
    } finally {
      setIsSubmitting(false)
    }
  }
  
  return (
    <div className="max-w-2xl mx-auto p-6">
      <div className="bg-white rounded-lg shadow-md p-6">
        <h2 className="text-2xl font-bold text-gray-900 mb-6">Book Service</h2>
        
        <div className="mb-6 p-4 bg-blue-50 rounded-lg">
          <h3 className="font-semibold text-blue-900">{listing.title}</h3>
          <p className="text-blue-700">Base Price: ${listing.base_price}</p>
          {listing.price_per_hour && (
            <p className="text-blue-700">+ ${listing.price_per_hour}/hour</p>
          )}
          {listing.price_per_mile && (
            <p className="text-blue-700">+ ${listing.price_per_mile}/mile</p>
          )}
        </div>
        
        <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Scheduled Date & Time
            </label>
            <input
              type="datetime-local"
              {...register('scheduledAt', { valueAsDate: true })}
              className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              min={new Date().toISOString().slice(0, 16)}
            />
            {errors.scheduledAt && (
              <p className="mt-1 text-sm text-red-600">{errors.scheduledAt.message}</p>
            )}
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Pickup Address
            </label>
            <input
              type="text"
              {...register('pickupAddress')}
              placeholder="Enter pickup address in Detroit Metro area"
              className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            {errors.pickupAddress && (
              <p className="mt-1 text-sm text-red-600">{errors.pickupAddress.message}</p>
            )}
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Delivery Address
            </label>
            <input
              type="text"
              {...register('deliveryAddress')}
              placeholder="Enter delivery address in Detroit Metro area"
              className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            {errors.deliveryAddress && (
              <p className="mt-1 text-sm text-red-600">{errors.deliveryAddress.message}</p>
            )}
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Estimated Duration (minutes)
            </label>
            <input
              type="number"
              {...register('estimatedDuration', { valueAsNumber: true })}
              min="15"
              max="480"
              className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
            {errors.estimatedDuration && (
              <p className="mt-1 text-sm text-red-600">{errors.estimatedDuration.message}</p>
            )}
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">
              Special Instructions (Optional)
            </label>
            <textarea
              {...register('specialInstructions')}
              placeholder="Any special instructions for the provider..."
              rows={4}
              className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
          
          {pricingEstimate && (
            <div className="p-4 bg-green-50 rounded-lg">
              <h4 className="font-semibold text-green-900 mb-2">Pricing Estimate</h4>
              <div className="space-y-1 text-sm text-green-800">
                <div className="flex justify-between">
                  <span>Subtotal:</span>
                  <span>${pricingEstimate.subtotal}</span>
                </div>
                <div className="flex justify-between">
                  <span>Service Fee:</span>
                  <span>${pricingEstimate.serviceFee}</span>
                </div>
                <div className="flex justify-between font-semibold text-base text-green-900 border-t border-green-200 pt-1">
                  <span>Total:</span>
                  <span>${pricingEstimate.total}</span>
                </div>
              </div>
            </div>
          )}
          
          <button
            type="submit"
            disabled={isSubmitting}
            className="w-full bg-blue-600 text-white py-3 px-4 rounded-md font-semibold hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {isSubmitting ? 'Creating Booking...' : 'Continue to Payment'}
          </button>
        </form>
      </div>
    </div>
  )
}
```

### 5. Create Booking Status Components

Create `src/components/booking/booking-status.tsx`:
```typescript
'use client'

import { useState } from 'react'
import { updateBookingStatus } from '@/lib/supabase/bookings'
import type { Booking, BookingStatus } from '@/types/booking'

interface BookingStatusProps {
  booking: Booking
  userRole: 'consumer' | 'provider'
  onStatusUpdate: (newStatus: BookingStatus) => void
}

export function BookingStatus({ booking, userRole, onStatusUpdate }: BookingStatusProps) {
  const [isUpdating, setIsUpdating] = useState(false)
  
  const getStatusColor = (status: BookingStatus) => {
    switch (status) {
      case 'pending': return 'bg-yellow-100 text-yellow-800'
      case 'accepted': return 'bg-blue-100 text-blue-800'
      case 'in_progress': return 'bg-purple-100 text-purple-800'
      case 'completed': return 'bg-green-100 text-green-800'
      case 'cancelled': return 'bg-red-100 text-red-800'
      default: return 'bg-gray-100 text-gray-800'
    }
  }
  
  const getStatusLabel = (status: BookingStatus) => {
    switch (status) {
      case 'pending': return 'Pending'
      case 'accepted': return 'Accepted'
      case 'in_progress': return 'In Progress'
      case 'completed': return 'Completed'
      case 'cancelled': return 'Cancelled'
      default: return status
    }
  }
  
  const getAvailableActions = (): { status: BookingStatus; label: string; variant: string }[] => {
    if (booking.status === 'completed' || booking.status === 'cancelled') {
      return []
    }
    
    const actions: { status: BookingStatus; label: string; variant: string }[] = []
    
    if (userRole === 'provider') {
      if (booking.status === 'pending') {
        actions.push({ status: 'accepted', label: 'Accept Booking', variant: 'success' })
      } else if (booking.status === 'accepted') {
        actions.push({ status: 'in_progress', label: 'Start Service', variant: 'primary' })
      } else if (booking.status === 'in_progress') {
        actions.push({ status: 'completed', label: 'Mark Complete', variant: 'success' })
      }
    }
    
    // Both consumer and provider can cancel (with restrictions)
    if (booking.status === 'pending' || booking.status === 'accepted') {
      actions.push({ status: 'cancelled', label: 'Cancel Booking', variant: 'danger' })
    }
    
    return actions
  }
  
  const handleStatusUpdate = async (newStatus: BookingStatus) => {
    if (isUpdating) return
    
    setIsUpdating(true)
    
    try {
      const result = await updateBookingStatus(booking.id, newStatus, booking.consumer_id)
      
      if (result.success) {
        onStatusUpdate(newStatus)
      } else {
        alert(result.error || 'Failed to update booking status')
      }
    } catch (error) {
      console.error('Status update error:', error)
      alert('Failed to update booking status')
    } finally {
      setIsUpdating(false)
    }
  }
  
  const actions = getAvailableActions()
  
  return (
    <div className="space-y-4">
      <div className="flex items-center gap-3">
        <span className={`px-3 py-1 rounded-full text-sm font-medium ${getStatusColor(booking.status)}`}>
          {getStatusLabel(booking.status)}
        </span>
        <span className="text-sm text-gray-500">
          Updated {new Date(booking.updated_at).toLocaleDateString()}
        </span>
      </div>
      
      {actions.length > 0 && (
        <div className="flex gap-2 flex-wrap">
          {actions.map((action) => (
            <button
              key={action.status}
              onClick={() => handleStatusUpdate(action.status)}
              disabled={isUpdating}
              className={`px-4 py-2 rounded-md text-sm font-medium disabled:opacity-50 disabled:cursor-not-allowed ${
                action.variant === 'success' 
                  ? 'bg-green-600 text-white hover:bg-green-700'
                  : action.variant === 'primary'
                  ? 'bg-blue-600 text-white hover:bg-blue-700'
                  : action.variant === 'danger'
                  ? 'bg-red-600 text-white hover:bg-red-700'
                  : 'bg-gray-600 text-white hover:bg-gray-700'
              }`}
            >
              {isUpdating ? 'Updating...' : action.label}
            </button>
          ))}
        </div>
      )}
    </div>
  )
}
```

### 6. Create Booking Detail Page

Create `src/app/booking/[id]/page.tsx`:
```typescript
import { createClient } from '@/lib/supabase/server'
import { BookingStatus } from '@/components/booking/booking-status'
import { notFound } from 'next/navigation'

interface BookingDetailPageProps {
  params: {
    id: string
  }
}

export default async function BookingDetailPage({ params }: BookingDetailPageProps) {
  const supabase = createClient()
  
  const { data: booking, error } = await supabase
    .from('bookings')
    .select(`
      *,
      listings (
        title,
        category,
        providers (
          business_name,
          profiles (
            full_name,
            phone
          )
        )
      )
    `)
    .eq('id', params.id)
    .single()
  
  if (error || !booking) {
    notFound()
  }
  
  // Get current user to determine role
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) {
    return <div>Please log in to view this booking</div>
  }
  
  const isConsumer = booking.consumer_id === user.id
  const userRole = isConsumer ? 'consumer' : 'provider'
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <div className="bg-white rounded-lg shadow-md p-6">
        <div className="mb-6">
          <h1 className="text-3xl font-bold text-gray-900">Booking Details</h1>
          <p className="text-gray-600">Booking ID: {booking.id.slice(0, 8)}</p>
        </div>
        
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div className="space-y-4">
            <div>
              <h2 className="text-xl font-semibold text-gray-900 mb-2">Service Information</h2>
              <div className="bg-gray-50 p-4 rounded-lg space-y-2">
                <p><strong>Service:</strong> {booking.listings.title}</p>
                <p><strong>Category:</strong> {booking.listings.category.replace('_', ' ').toUpperCase()}</p>
                <p><strong>Provider:</strong> {booking.listings.providers.business_name}</p>
                <p><strong>Scheduled:</strong> {new Date(booking.scheduled_at).toLocaleString()}</p>
                {booking.duration_minutes && (
                  <p><strong>Duration:</strong> {booking.duration_minutes} minutes</p>
                )}
              </div>
            </div>
            
            <div>
              <h2 className="text-xl font-semibold text-gray-900 mb-2">Location Details</h2>
              <div className="bg-gray-50 p-4 rounded-lg space-y-2">
                {booking.pickup_address && (
                  <p><strong>Pickup:</strong> {booking.pickup_address}</p>
                )}
                {booking.delivery_address && (
                  <p><strong>Delivery:</strong> {booking.delivery_address}</p>
                )}
                {booking.special_instructions && (
                  <div>
                    <strong>Special Instructions:</strong>
                    <p className="text-gray-700 mt-1">{booking.special_instructions}</p>
                  </div>
                )}
              </div>
            </div>
          </div>
          
          <div className="space-y-4">
            <div>
              <h2 className="text-xl font-semibold text-gray-900 mb-2">Status</h2>
              <BookingStatus
                booking={booking}
                userRole={userRole}
                onStatusUpdate={() => window.location.reload()}
              />
            </div>
            
            <div>
              <h2 className="text-xl font-semibold text-gray-900 mb-2">Pricing</h2>
              <div className="bg-gray-50 p-4 rounded-lg space-y-2">
                <div className="flex justify-between">
                  <span>Total Amount:</span>
                  <span className="font-semibold">${booking.total_amount}</span>
                </div>
                <div className="flex justify-between">
                  <span>Service Fee:</span>
                  <span>${booking.service_fee}</span>
                </div>
                <div className="flex justify-between text-sm text-gray-600">
                  <span>Payment Status:</span>
                  <span className="capitalize">{booking.payment_status}</span>
                </div>
              </div>
            </div>
            
            {userRole === 'consumer' && booking.listings.providers.profiles?.phone && (
              <div>
                <h2 className="text-xl font-semibold text-gray-900 mb-2">Contact</h2>
                <div className="bg-gray-50 p-4 rounded-lg">
                  <p><strong>Provider:</strong> {booking.listings.providers.profiles.full_name}</p>
                  <p><strong>Phone:</strong> {booking.listings.providers.profiles.phone}</p>
                </div>
              </div>
            )}
          </div>
        </div>
        
        <div className="mt-8 pt-6 border-t border-gray-200 text-sm text-gray-500">
          <p>Created: {new Date(booking.created_at).toLocaleString()}</p>
          <p>Last Updated: {new Date(booking.updated_at).toLocaleString()}</p>
        </div>
      </div>
    </div>
  )
}
```

### 7. Create Booking List Component

Create `src/components/booking/booking-list.tsx`:
```typescript
'use client'

import { useEffect, useState } from 'react'
import Link from 'next/link'
import { createClient } from '@/lib/supabase/client'
import { getUserBookings } from '@/lib/supabase/bookings'
import type { Booking } from '@/types/booking'

interface BookingWithDetails extends Booking {
  listings: {
    title: string
    category: string
    providers: {
      business_name: string
      profiles: {
        full_name: string
      }
    }
  }
}

interface BookingListProps {
  userRole: 'consumer' | 'provider'
}

export function BookingList({ userRole }: BookingListProps) {
  const [bookings, setBookings] = useState<BookingWithDetails[]>([])
  const [loading, setLoading] = useState(true)
  const [filter, setFilter] = useState<'all' | 'pending' | 'accepted' | 'in_progress' | 'completed'>('all')
  
  useEffect(() => {
    loadBookings()
  }, [userRole])
  
  const loadBookings = async () => {
    try {
      const supabase = createClient()
      const { data: { user } } = await supabase.auth.getUser()
      
      if (!user) return
      
      const { data, error } = await getUserBookings(user.id, userRole)
      
      if (error) {
        console.error('Error loading bookings:', error)
      } else {
        setBookings(data || [])
      }
    } catch (error) {
      console.error('Error loading bookings:', error)
    } finally {
      setLoading(false)
    }
  }
  
  const filteredBookings = filter === 'all' 
    ? bookings 
    : bookings.filter(booking => booking.status === filter)
  
  const getStatusColor = (status: string) => {
    switch (status) {
      case 'pending': return 'bg-yellow-100 text-yellow-800'
      case 'accepted': return 'bg-blue-100 text-blue-800'
      case 'in_progress': return 'bg-purple-100 text-purple-800'
      case 'completed': return 'bg-green-100 text-green-800'
      case 'cancelled': return 'bg-red-100 text-red-800'
      default: return 'bg-gray-100 text-gray-800'
    }
  }
  
  if (loading) {
    return <div className="text-center py-8">Loading bookings...</div>
  }
  
  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h2 className="text-2xl font-bold text-gray-900">
          My {userRole === 'consumer' ? 'Bookings' : 'Jobs'}
        </h2>
        
        <select
          value={filter}
          onChange={(e) => setFilter(e.target.value as any)}
          className="px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          <option value="all">All Status</option>
          <option value="pending">Pending</option>
          <option value="accepted">Accepted</option>
          <option value="in_progress">In Progress</option>
          <option value="completed">Completed</option>
        </select>
      </div>
      
      {filteredBookings.length === 0 ? (
        <div className="text-center py-8 text-gray-500">
          No bookings found.
        </div>
      ) : (
        <div className="space-y-4">
          {filteredBookings.map((booking) => (
            <Link
              key={booking.id}
              href={`/booking/${booking.id}`}
              className="block bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow p-6"
            >
              <div className="flex items-start justify-between">
                <div className="flex-1">
                  <h3 className="text-xl font-semibold text-gray-900 mb-2">
                    {booking.listings.title}
                  </h3>
                  
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm text-gray-600">
                    <div>
                      <p><strong>Provider:</strong> {booking.listings.providers.business_name}</p>
                      <p><strong>Category:</strong> {booking.listings.category.replace('_', ' ')}</p>
                      <p><strong>Scheduled:</strong> {new Date(booking.scheduled_at).toLocaleDateString()}</p>
                    </div>
                    
                    <div>
                      {booking.pickup_address && (
                        <p><strong>Pickup:</strong> {booking.pickup_address.split(',')[0]}</p>
                      )}
                      {booking.delivery_address && (
                        <p><strong>Delivery:</strong> {booking.delivery_address.split(',')[0]}</p>
                      )}
                      <p><strong>Total:</strong> ${booking.total_amount}</p>
                    </div>
                  </div>
                </div>
                
                <div className="ml-4 text-right">
                  <span className={`px-3 py-1 rounded-full text-sm font-medium ${getStatusColor(booking.status)}`}>
                    {booking.status.replace('_', ' ')}
                  </span>
                  <p className="text-xs text-gray-500 mt-2">
                    {new Date(booking.created_at).toLocaleDateString()}
                  </p>
                </div>
              </div>
            </Link>
          ))}
        </div>
      )}
    </div>
  )
}
```

## Expected Outcome

After completing this step, you should have:

✅ **Complete Booking System**
- Multi-step booking form with validation
- Address geocoding and service area validation  
- Dynamic pricing calculation
- Status management workflow

✅ **Location Services**
- Detroit Metro area validation
- Distance calculation between addresses
- Geocoding integration ready for Mapbox

✅ **Database Integration**
- Secure booking creation with RLS
- Status update functionality
- User role-based permissions

## Troubleshooting

### Common Issues

**Issue: Geocoding fails**
```typescript
// Add error handling for geocoding
export async function geocodeAddress(address: string): Promise<GeocodeResult | null> {
  try {
    // For development, you can use mock coordinates
    if (process.env.NODE_ENV === 'development') {
      return {
        lat: 42.3314,
        lng: -83.0458,
        address: `${address}, Detroit, MI`
      }
    }
    // ... rest of implementation
  } catch (error) {
    console.error('Geocoding error:', error)
    return null
  }
}
```

**Issue: Status updates not working**
- Check RLS policies allow updates for both consumers and providers
- Verify user authentication state
- Check booking ownership in update function

**Issue: Pricing calculation errors**
- Validate all numeric inputs
- Handle edge cases (zero distance, duration)
- Add proper error boundaries around calculations

## Next Steps

Once you've successfully completed this step:

1. **Test the booking flow**:
   ```bash
   # Create a test booking
   # Update booking status
   # Verify RLS policies work
   ```

2. **Move to Step 17**: [Real-time Features](./17-realtime-features.md)

## Key Concepts Learned

- **Complex Form Handling**: Multi-step workflows with validation
- **Geospatial Calculations**: Distance calculation and area validation
- **Business Logic**: Pricing calculation and service area management
- **Database Relationships**: Managing data across multiple tables
- **Real-time Status**: Status management with role-based permissions

---

**🎯 Success Criteria**: Users can successfully create bookings, providers can update status, and the system validates Detroit Metro service area restrictions.