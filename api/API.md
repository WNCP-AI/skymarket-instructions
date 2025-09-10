# SkyMarket API Documentation

Complete REST API documentation for the SkyMarket drone service marketplace platform.

## API Overview

**Base URL**: `https://skymarket.com/api` (production) | `http://localhost:3000/api` (development)  
**Authentication**: JWT tokens via Supabase Auth  
**Content Type**: `application/json`  
**API Version**: v1  

### Response Format

All API responses follow a consistent format:

```json
{
  "data": {}, // Response data or array
  "error": null, // Error object if request failed
  "message": "Success", // Human-readable message
  "meta": { // Optional metadata
    "pagination": {},
    "total": 0,
    "filters": {}
  }
}
```

### Error Responses

```json
{
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "email",
      "issue": "Email format is invalid"
    }
  },
  "message": "Request failed"
}
```

### Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

## Authentication

### JWT Token Authentication

All authenticated requests must include the JWT token in the Authorization header:

```bash
Authorization: Bearer <jwt_token>
```

### Getting Authentication Token

Authentication is handled by Supabase Auth. Clients should use the Supabase client library:

```typescript
// Client-side authentication
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(url, anonKey)

// Sign in
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'password'
})

// Token is automatically included in subsequent requests
```

## Core Endpoints

## Authentication Endpoints

### POST /api/auth/signup

Create a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "fullName": "John Doe",
  "role": "consumer" // "consumer" | "provider"
}
```

**Response (201):**
```json
{
  "data": {
    "user": {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "email": "user@example.com",
      "role": "consumer"
    },
    "session": {
      "access_token": "jwt_token_here",
      "refresh_token": "refresh_token_here"
    }
  },
  "message": "Account created successfully"
}
```

### POST /api/auth/login

Authenticate existing user.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200):**
```json
{
  "data": {
    "user": {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "email": "user@example.com",
      "role": "consumer",
      "profile": {
        "fullName": "John Doe",
        "avatarUrl": "https://example.com/avatar.jpg"
      }
    },
    "session": {
      "access_token": "jwt_token_here",
      "refresh_token": "refresh_token_here"
    }
  },
  "message": "Login successful"
}
```

### POST /api/auth/logout

Sign out current user.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "data": null,
  "message": "Logout successful"
}
```

## User Profile Endpoints

### GET /api/profile

Get current user's profile.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "email": "user@example.com",
    "fullName": "John Doe",
    "phone": "+1234567890",
    "address": "123 Main St, Detroit, MI 48201",
    "avatarUrl": "https://example.com/avatar.jpg",
    "role": "consumer",
    "detroitZone": "Downtown Detroit",
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-02T00:00:00Z"
  }
}
```

### PATCH /api/profile

Update current user's profile.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "fullName": "John Smith",
  "phone": "+1234567890",
  "address": "456 Oak St, Detroit, MI 48202",
  "detroitZone": "Midtown Detroit"
}
```

**Response (200):**
```json
{
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "fullName": "John Smith",
    "phone": "+1234567890",
    "updatedAt": "2024-01-03T00:00:00Z"
  },
  "message": "Profile updated successfully"
}
```

## Provider Endpoints

### POST /api/providers

Create provider profile (converts consumer to provider).

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "type": "drone", // "courier" | "drone"
  "certifications": ["FAA Part 107", "Drone Insurance"],
  "serviceAreas": ["Downtown Detroit", "Midtown Detroit"]
}
```

**Response (201):**
```json
{
  "data": {
    "id": "provider_uuid",
    "userId": "user_uuid",
    "type": "drone",
    "certifications": ["FAA Part 107", "Drone Insurance"],
    "serviceAreas": ["Downtown Detroit", "Midtown Detroit"],
    "rating": 0.0,
    "completedJobs": 0,
    "verified": false,
    "createdAt": "2024-01-01T00:00:00Z"
  },
  "message": "Provider profile created"
}
```

### GET /api/providers

Get list of providers with filters.

**Query Parameters:**
- `type` - Filter by provider type (`courier` | `drone`)
- `serviceArea` - Filter by service area
- `verified` - Filter by verification status (`true` | `false`)
- `rating` - Minimum rating (0-5)
- `limit` - Results per page (default: 20, max: 100)
- `offset` - Pagination offset

**Example Request:**
```bash
GET /api/providers?type=drone&serviceArea=Downtown%20Detroit&verified=true&rating=4.0&limit=10
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "provider_uuid",
      "type": "drone",
      "rating": 4.8,
      "completedJobs": 157,
      "verified": true,
      "serviceAreas": ["Downtown Detroit", "Midtown Detroit"],
      "user": {
        "fullName": "Mike Johnson",
        "avatarUrl": "https://example.com/avatar.jpg"
      },
      "activeListings": 3,
      "avgResponseTime": 120 // seconds
    }
  ],
  "meta": {
    "total": 45,
    "limit": 10,
    "offset": 0,
    "hasMore": true
  }
}
```

### GET /api/providers/:id

Get specific provider details.

**Response (200):**
```json
{
  "data": {
    "id": "provider_uuid",
    "type": "drone",
    "rating": 4.8,
    "completedJobs": 157,
    "verified": true,
    "certifications": ["FAA Part 107", "Commercial Insurance"],
    "serviceAreas": ["Downtown Detroit", "Midtown Detroit"],
    "user": {
      "fullName": "Mike Johnson",
      "avatarUrl": "https://example.com/avatar.jpg"
    },
    "listings": [
      {
        "id": "listing_uuid",
        "title": "Aerial Photography Service",
        "category": "aerial_imaging",
        "priceBase": 149.99,
        "active": true
      }
    ],
    "reviews": [
      {
        "id": "review_uuid",
        "rating": 5,
        "comment": "Excellent service!",
        "reviewer": {
          "fullName": "Sarah K."
        },
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "stats": {
      "avgResponseTime": 120,
      "onTimeRate": 0.96,
      "avgOrderValue": 87.50
    }
  }
}
```

## Listing Endpoints

### POST /api/listings

Create a new service listing (provider only).

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "title": "Professional Food Delivery",
  "description": "Fast and reliable food delivery service using temperature-controlled drone compartments",
  "category": "food_delivery", // "food_delivery" | "courier" | "aerial_imaging" | "site_mapping"
  "priceBase": 8.99,
  "pricePerMile": 1.50,
  "pricePerMinute": null,
  "serviceRadiusMiles": 15,
  "images": ["https://example.com/image1.jpg", "https://example.com/image2.jpg"]
}
```

**Response (201):**
```json
{
  "data": {
    "id": "listing_uuid",
    "providerId": "provider_uuid",
    "title": "Professional Food Delivery",
    "description": "Fast and reliable food delivery service...",
    "category": "food_delivery",
    "priceBase": 8.99,
    "pricePerMile": 1.50,
    "pricePerMinute": null,
    "serviceRadiusMiles": 15,
    "active": true,
    "images": ["https://example.com/image1.jpg", "https://example.com/image2.jpg"],
    "createdAt": "2024-01-01T00:00:00Z"
  },
  "message": "Listing created successfully"
}
```

### GET /api/listings

Search and filter service listings.

**Query Parameters:**
- `category` - Filter by service category
- `search` - Full-text search query
- `lat` & `lng` - Location coordinates for proximity search
- `radius` - Search radius in miles (default: 25)
- `minRating` - Minimum provider rating
- `maxPrice` - Maximum base price
- `verified` - Only verified providers
- `available` - Only available providers
- `sortBy` - Sort by (`rating` | `price` | `distance` | `created_at`)
- `sortOrder` - Sort direction (`asc` | `desc`)
- `limit` - Results per page
- `offset` - Pagination offset

**Example Request:**
```bash
GET /api/listings?category=food_delivery&lat=42.3314&lng=-83.0458&radius=10&minRating=4.0&sortBy=rating&sortOrder=desc&limit=20
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "listing_uuid",
      "title": "Professional Food Delivery",
      "category": "food_delivery",
      "priceBase": 8.99,
      "pricePerMile": 1.50,
      "serviceRadiusMiles": 15,
      "images": ["https://example.com/image1.jpg"],
      "provider": {
        "id": "provider_uuid",
        "rating": 4.8,
        "completedJobs": 157,
        "verified": true,
        "user": {
          "fullName": "Mike Johnson",
          "avatarUrl": "https://example.com/avatar.jpg"
        }
      },
      "estimatedPrice": 12.49,
      "estimatedTime": 25,
      "distance": 2.3
    }
  ],
  "meta": {
    "total": 24,
    "filters": {
      "category": "food_delivery",
      "location": "42.3314,-83.0458",
      "radius": 10
    },
    "pagination": {
      "limit": 20,
      "offset": 0,
      "hasMore": true
    }
  }
}
```

### GET /api/listings/:id

Get detailed listing information.

**Response (200):**
```json
{
  "data": {
    "id": "listing_uuid",
    "title": "Professional Food Delivery",
    "description": "Fast and reliable food delivery service using temperature-controlled drone compartments. Available 7 days a week with real-time tracking.",
    "category": "food_delivery",
    "priceBase": 8.99,
    "pricePerMile": 1.50,
    "pricePerMinute": null,
    "serviceRadiusMiles": 15,
    "active": true,
    "images": [
      "https://example.com/image1.jpg",
      "https://example.com/image2.jpg"
    ],
    "provider": {
      "id": "provider_uuid",
      "type": "drone",
      "rating": 4.8,
      "completedJobs": 157,
      "verified": true,
      "certifications": ["FAA Part 107", "Commercial Insurance"],
      "user": {
        "fullName": "Mike Johnson",
        "avatarUrl": "https://example.com/avatar.jpg"
      }
    },
    "availability": {
      "monday": { "start": "08:00", "end": "20:00" },
      "tuesday": { "start": "08:00", "end": "20:00" },
      "sunday": null
    },
    "policies": {
      "cancellationPolicy": "Free cancellation up to 1 hour before scheduled time",
      "refundPolicy": "Full refund for service issues"
    },
    "reviews": [
      {
        "id": "review_uuid",
        "rating": 5,
        "comment": "Fast delivery, food was still hot!",
        "reviewer": {
          "fullName": "Sarah K."
        },
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "stats": {
      "avgDeliveryTime": 22,
      "onTimeRate": 0.96,
      "totalBookings": 157
    }
  }
}
```

### PATCH /api/listings/:id

Update listing (provider only).

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "title": "Updated Service Title",
  "priceBase": 9.99,
  "active": false
}
```

**Response (200):**
```json
{
  "data": {
    "id": "listing_uuid",
    "title": "Updated Service Title",
    "priceBase": 9.99,
    "active": false,
    "updatedAt": "2024-01-03T00:00:00Z"
  },
  "message": "Listing updated successfully"
}
```

## Booking Endpoints

### POST /api/bookings

Create a new booking.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "listingId": "listing_uuid",
  "scheduledAt": "2024-01-05T14:30:00Z",
  "pickupAddress": "123 Main St, Detroit, MI 48201", // optional
  "pickupLat": 42.3314, // optional
  "pickupLng": -83.0458, // optional
  "dropoffAddress": "456 Oak St, Detroit, MI 48202",
  "dropoffLat": 42.3500,
  "dropoffLng": -83.0600,
  "specialInstructions": "Leave at front door, ring doorbell"
}
```

**Response (201):**
```json
{
  "data": {
    "id": "booking_uuid",
    "consumerId": "consumer_uuid",
    "providerId": "provider_uuid",
    "listingId": "listing_uuid",
    "status": "pending",
    "scheduledAt": "2024-01-05T14:30:00Z",
    "pickupAddress": "123 Main St, Detroit, MI 48201",
    "dropoffAddress": "456 Oak St, Detroit, MI 48202",
    "specialInstructions": "Leave at front door, ring doorbell",
    "priceTotal": 12.49,
    "paymentStatus": "pending",
    "createdAt": "2024-01-01T00:00:00Z",
    "estimatedDuration": 25,
    "distance": 2.3
  },
  "message": "Booking created successfully"
}
```

### GET /api/bookings

Get user's bookings (consumer and provider views).

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**
- `status` - Filter by status (`pending` | `accepted` | `in_progress` | `completed` | `cancelled`)
- `role` - View as (`consumer` | `provider`) - auto-detected if omitted
- `startDate` - Filter bookings after date
- `endDate` - Filter bookings before date
- `limit` - Results per page
- `offset` - Pagination offset

**Response (200):**
```json
{
  "data": [
    {
      "id": "booking_uuid",
      "status": "accepted",
      "scheduledAt": "2024-01-05T14:30:00Z",
      "dropoffAddress": "456 Oak St, Detroit, MI 48202",
      "priceTotal": 12.49,
      "paymentStatus": "paid",
      "listing": {
        "title": "Professional Food Delivery",
        "category": "food_delivery"
      },
      "provider": {
        "user": {
          "fullName": "Mike Johnson",
          "avatarUrl": "https://example.com/avatar.jpg"
        }
      },
      "consumer": {
        "fullName": "John Doe"
      },
      "createdAt": "2024-01-01T00:00:00Z",
      "estimatedArrival": "2024-01-05T14:55:00Z"
    }
  ],
  "meta": {
    "total": 15,
    "statusCounts": {
      "pending": 2,
      "accepted": 1,
      "in_progress": 0,
      "completed": 10,
      "cancelled": 2
    }
  }
}
```

### GET /api/bookings/:id

Get detailed booking information.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "data": {
    "id": "booking_uuid",
    "status": "in_progress",
    "scheduledAt": "2024-01-05T14:30:00Z",
    "dropoffAddress": "456 Oak St, Detroit, MI 48202",
    "dropoffLat": 42.3500,
    "dropoffLng": -83.0600,
    "specialInstructions": "Leave at front door, ring doorbell",
    "priceTotal": 12.49,
    "paymentStatus": "paid",
    "paymentIntentId": "pi_stripe_id",
    "listing": {
      "id": "listing_uuid",
      "title": "Professional Food Delivery",
      "category": "food_delivery",
      "provider": {
        "id": "provider_uuid",
        "type": "drone",
        "user": {
          "fullName": "Mike Johnson",
          "avatarUrl": "https://example.com/avatar.jpg",
          "phone": "+1234567890"
        }
      }
    },
    "tracking": {
      "currentLocation": {
        "lat": 42.3400,
        "lng": -83.0500
      },
      "eta": "2024-01-05T14:55:00Z",
      "statusHistory": [
        {
          "status": "pending",
          "timestamp": "2024-01-01T00:00:00Z"
        },
        {
          "status": "accepted",
          "timestamp": "2024-01-05T14:00:00Z"
        },
        {
          "status": "in_progress",
          "timestamp": "2024-01-05T14:25:00Z"
        }
      ]
    },
    "messages": [
      {
        "id": "message_uuid",
        "senderId": "provider_user_id",
        "content": "On my way to pickup location",
        "createdAt": "2024-01-05T14:25:00Z"
      }
    ]
  }
}
```

### PATCH /api/bookings/:id

Update booking status (provider and consumer actions).

**Headers:** `Authorization: Bearer <token>`

**Request Body (Provider accepting booking):**
```json
{
  "status": "accepted",
  "estimatedArrival": "2024-01-05T14:55:00Z"
}
```

**Request Body (Provider updating location):**
```json
{
  "tracking": {
    "currentLocation": {
      "lat": 42.3400,
      "lng": -83.0500
    },
    "eta": "2024-01-05T14:50:00Z"
  }
}
```

**Request Body (Consumer cancelling):**
```json
{
  "status": "cancelled",
  "cancellationReason": "No longer needed"
}
```

**Response (200):**
```json
{
  "data": {
    "id": "booking_uuid",
    "status": "accepted",
    "estimatedArrival": "2024-01-05T14:55:00Z",
    "updatedAt": "2024-01-05T14:00:00Z"
  },
  "message": "Booking updated successfully"
}
```

## Messaging Endpoints

### GET /api/bookings/:id/messages

Get messages for a booking.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "data": [
    {
      "id": "message_uuid",
      "senderId": "user_uuid",
      "recipientId": "user_uuid",
      "content": "On my way to pickup location",
      "read": false,
      "createdAt": "2024-01-05T14:25:00Z",
      "sender": {
        "fullName": "Mike Johnson",
        "avatarUrl": "https://example.com/avatar.jpg"
      }
    }
  ]
}
```

### POST /api/bookings/:id/messages

Send message for a booking.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "content": "I'm at the pickup location now",
  "recipientId": "user_uuid"
}
```

**Response (201):**
```json
{
  "data": {
    "id": "message_uuid",
    "senderId": "current_user_id",
    "recipientId": "recipient_user_id",
    "content": "I'm at the pickup location now",
    "read": false,
    "createdAt": "2024-01-05T14:30:00Z"
  },
  "message": "Message sent successfully"
}
```

### PATCH /api/messages/:id/read

Mark message as read.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "data": {
    "id": "message_uuid",
    "read": true,
    "readAt": "2024-01-05T14:35:00Z"
  }
}
```

## Payment Endpoints

### POST /api/orders

Create order and Stripe PaymentIntent (manual capture/escrow).

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "listingId": "listing_uuid",
  "scheduledAt": "2024-01-05T14:30:00Z",
  "pickupAddress": "123 Main St",
  "dropoffAddress": "456 Oak St",
  "specialInstructions": "Leave at door",
  "distanceMiles": 2.3,
  "durationMinutes": 25
}
```

**Response (201):**
```json
{
  "data": {
    "id": "booking_uuid",
    "clientSecret": "pi_client_secret_xxx"
  }
}
```

### GET /api/orders

List orders for current user. Use `role=provider` to view as provider.

### GET /api/orders/:id

Fetch a single order by id.

### PATCH /api/orders/:id

Update order (e.g., `status`). To release escrow on completion, pass `{ "capture": true }` to capture the PaymentIntent.

---

## Webhooks

### POST /api/webhooks/stripe

Stripe webhook for payment lifecycle.

Events handled:
- `payment_intent.succeeded` → set `bookings.payment_status = 'paid'`
- `payment_intent.payment_failed` → set `payment_status = 'failed'`
- `payment_intent.canceled` → set `payment_status = 'cancelled'`
- `charge.captured` → confirm funds captured (escrow release)

Also supports Stripe Checkout:
- `checkout.session.completed` → mark order paid

Additional processing and reliability behaviors:
- If `metadata.booking_id` is present, updates the target booking directly; otherwise falls back to lookup by `payment_intent_id` (for Checkout flows).
- Stores `payment_intent_id` on `checkout.session.completed` when available.
- Handles `charge.succeeded`, `charge.refunded`, and `charge.updated` (when refunded) to keep `payment_status` accurate.

Debugging:
- Set `STRIPE_WEBHOOK_DEBUG=true` to log receipt of events and Supabase errors.
- Webhook verifies signatures using `STRIPE_WEBHOOK_SECRET` against the raw request body.

---

## Checkout

### POST /api/orders/:id/checkout

Creates a Stripe Checkout Session for the order using dynamic `price_data.unit_amount` based on computed booking total.

Response:
```json
{ "data": { "id": "cs_test_...", "url": "https://checkout.stripe.com/c/session_id" } }
```

Client action redirects the user to Stripe Checkout. The server route derives success/cancel URLs from request headers, so it works in dev and prod.

### Order Page Payment State

- When `bookings.payment_status = 'paid'`, the order page shows a Paid badge and a “View receipt” link (Stripe receipt URL) when available.
- If not yet paid and status is `pending`, a “Pay with Stripe Checkout” button is rendered.
- The Cancel action is shown in a top help card (consumer only) explaining it cancels the service booking.

---

## Review Endpoints

### POST /api/reviews

Create review for completed booking.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "bookingId": "booking_uuid",
  "reviewedId": "user_uuid", // ID of user being reviewed
  "rating": 5,
  "comment": "Excellent service, very professional!"
}
```

**Response (201):**
```json
{
  "data": {
    "id": "review_uuid",
    "bookingId": "booking_uuid",
    "reviewerId": "current_user_id",
    "reviewedId": "reviewed_user_id",
    "rating": 5,
    "comment": "Excellent service, very professional!",
    "createdAt": "2024-01-05T16:00:00Z"
  },
  "message": "Review submitted successfully"
}
```

### GET /api/reviews

Get reviews for a user (provider or consumer).

**Query Parameters:**
- `userId` - Get reviews for specific user
- `role` - Filter by reviewer role (`consumer` | `provider`)
- `rating` - Filter by minimum rating
- `limit` - Results per page
- `offset` - Pagination offset

**Response (200):**
```json
{
  "data": [
    {
      "id": "review_uuid",
      "rating": 5,
      "comment": "Excellent service, very professional!",
      "reviewer": {
        "fullName": "John D.",
        "avatarUrl": "https://example.com/avatar.jpg"
      },
      "booking": {
        "listing": {
          "title": "Professional Food Delivery"
        }
      },
      "createdAt": "2024-01-05T16:00:00Z"
    }
  ],
  "meta": {
    "total": 45,
    "averageRating": 4.7,
    "ratingDistribution": {
      "5": 32,
      "4": 10,
      "3": 2,
      "2": 1,
      "1": 0
    }
  }
}
```

## File Upload Endpoints

### POST /api/upload

Upload files (images, documents).

**Headers:** 
- `Authorization: Bearer <token>`
- `Content-Type: multipart/form-data`

**Request Body (FormData):**
```
file: <binary_file_data>
type: "avatar" | "listing" | "document" | "proof"
```

**Response (200):**
```json
{
  "data": {
    "url": "https://storage.supabase.co/object/public/uploads/user_123/image.jpg",
    "filename": "image.jpg",
    "size": 1024000,
    "type": "image/jpeg"
  },
  "message": "File uploaded successfully"
}
```

## WebSocket/Real-time Endpoints

SkyMarket uses Supabase real-time subscriptions for live updates.

### Booking Updates

Subscribe to booking status changes:

```typescript
// Client-side subscription
const subscription = supabase
  .channel(`booking:${bookingId}`)
  .on('postgres_changes', 
    { 
      event: 'UPDATE', 
      schema: 'public', 
      table: 'bookings',
      filter: `id=eq.${bookingId}`
    }, 
    (payload) => {
      console.log('Booking updated:', payload.new)
    }
  )
  .subscribe()
```

### Message Notifications

Subscribe to new messages:

```typescript
const subscription = supabase
  .channel(`messages:${userId}`)
  .on('postgres_changes', 
    { 
      event: 'INSERT', 
      schema: 'public', 
      table: 'messages',
      filter: `recipient_id=eq.${userId}`
    }, 
    (payload) => {
      console.log('New message:', payload.new)
    }
  )
  .subscribe()
```

## Rate Limiting

API endpoints are rate limited to prevent abuse:

| Endpoint Category | Rate Limit | Window |
|------------------|------------|---------|
| Authentication | 5 requests | 1 minute |
| Search/Browse | 100 requests | 1 minute |
| Booking Creation | 10 requests | 1 minute |
| File Upload | 20 requests | 1 minute |
| Messages | 50 requests | 1 minute |
| General API | 200 requests | 1 minute |

Rate limit headers are included in responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1704067200
```

## Error Codes Reference

| Code | Description | Typical Cause |
|------|-------------|---------------|
| `VALIDATION_ERROR` | Request data validation failed | Invalid input format |
| `AUTHENTICATION_REQUIRED` | JWT token missing/invalid | User not logged in |
| `AUTHORIZATION_DENIED` | Insufficient permissions | Role/ownership mismatch |
| `RESOURCE_NOT_FOUND` | Requested resource doesn't exist | Invalid ID |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Client making too many calls |
| `PAYMENT_FAILED` | Payment processing error | Card declined, insufficient funds |
| `BOOKING_CONFLICT` | Booking time/availability conflict | Provider unavailable |
| `SERVICE_UNAVAILABLE` | External service down | Stripe, Mapbox, etc. outage |
| `INTERNAL_ERROR` | Server error | Database, application bug |

## SDK & Client Libraries

### JavaScript/TypeScript

```bash
npm install @skymarket/js-sdk
```

```typescript
import SkyMarket from '@skymarket/js-sdk'

const client = new SkyMarket({
  apiKey: 'your_api_key',
  baseURL: 'https://api.skymarket.com'
})

// Example usage
const listings = await client.listings.search({
  category: 'food_delivery',
  location: { lat: 42.3314, lng: -83.0458 },
  radius: 10
})
```

### cURL Examples

```bash
# Search listings
curl -X GET "https://api.skymarket.com/api/listings?category=food_delivery&lat=42.3314&lng=-83.0458" \
  -H "Authorization: Bearer your_jwt_token"

# Create booking
curl -X POST "https://api.skymarket.com/api/bookings" \
  -H "Authorization: Bearer your_jwt_token" \
  -H "Content-Type: application/json" \
  -d '{
    "listingId": "listing_uuid",
    "scheduledAt": "2024-01-05T14:30:00Z",
    "dropoffAddress": "456 Oak St, Detroit, MI 48202",
    "dropoffLat": 42.3500,
    "dropoffLng": -83.0600
  }'
```

---

This API documentation provides comprehensive coverage of all SkyMarket endpoints with request/response examples, authentication patterns, and real-time capabilities.