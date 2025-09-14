# SkyMarket Business Logic Documentation

Business logic and domain rules documentation for the SkyMarket drone service marketplace platform.

## Overview

SkyMarket is a Detroit-focused drone ecommerce marketplace with 5 core domains: **Listings**, **Bookings**, **Orders**, **Auth**, and **Messaging**. The platform connects consumers with drone service providers across 4 service categories.

**Core Business Model**:
- Two-sided marketplace: Consumers and Providers
- Stripe Connect payment processing with escrow
- Detroit Metro area geographic focus
- Simple CRUD operations with minimal business rules

## Core Domains

### 1. Listings Domain

**Purpose**: Service catalog management for drone providers

**API Endpoints**:
- `GET /api/listings` - Browse/search active listings
- `POST /api/listings` - Create new listing (providers only)
- `GET /api/listings/[id]` - Get listing details
- `PUT/DELETE /api/listings/[id]` - Update/delete listing (owner only)

**Data Model**:
```sql
CREATE TABLE listings (
  id UUID PRIMARY KEY,
  provider_id UUID REFERENCES providers(id),
  title TEXT NOT NULL,
  description TEXT,
  category service_category NOT NULL, -- 'food_delivery' | 'courier' | 'aerial_imaging' | 'site_mapping'
  price_base DECIMAL(10,2) NOT NULL,
  price_per_mile DECIMAL(10,2),
  price_per_minute DECIMAL(10,2),
  service_radius_miles INTEGER DEFAULT 10,
  active BOOLEAN DEFAULT TRUE,
  images TEXT[]
);
```

**Business Rules**:
- Only verified providers can create listings
- Listings must have valid service category
- Base price must be positive
- Optional distance/time-based pricing

### 2. Bookings/Orders Domain

**Purpose**: Order lifecycle management from creation to completion

**API Endpoints**:
- `GET /api/orders` - List user's orders (consumer/provider view)
- `POST /api/orders` - Create new booking
- `GET /api/orders/[id]` - Get order details
- `PUT /api/orders/[id]` - Update order status
- `POST /api/orders/[id]/checkout` - Complete payment

**Data Model**:
```sql
CREATE TABLE bookings (
  id UUID PRIMARY KEY,
  consumer_id UUID REFERENCES profiles(id),
  provider_id UUID REFERENCES providers(id),
  listing_id UUID REFERENCES listings(id),
  status booking_status DEFAULT 'pending', -- 'pending' | 'accepted' | 'in_progress' | 'completed' | 'cancelled'
  scheduled_at TIMESTAMPTZ NOT NULL,
  pickup_address TEXT,
  pickup_lat/pickup_lng DECIMAL,
  dropoff_address TEXT NOT NULL,
  dropoff_lat/dropoff_lng DECIMAL NOT NULL,
  special_instructions TEXT,
  price_total DECIMAL(10,2) NOT NULL,
  payment_intent_id TEXT,
  payment_status payment_status DEFAULT 'pending' -- 'pending' | 'paid' | 'failed' | 'refunded'
);
```

**Business Rules**:
- Price calculation: `base + (distance * price_per_mile) + (duration * price_per_minute)`
- Stripe PaymentIntent created with manual capture
- Status flow: `pending → accepted → in_progress → completed`
- Cancellation allowed before `in_progress`

### 3. Authentication Domain

**Purpose**: User management and role-based access

**Data Model**:
```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  email TEXT UNIQUE NOT NULL,
  full_name TEXT,
  role user_role DEFAULT 'consumer', -- 'consumer' | 'provider' | 'admin'
  phone TEXT,
  address TEXT,
  detroit_zone TEXT
);

CREATE TABLE providers (
  id UUID PRIMARY KEY,
  user_id UUID UNIQUE REFERENCES profiles(id),
  type provider_type NOT NULL, -- 'courier' | 'drone'
  certifications TEXT[],
  verified BOOLEAN DEFAULT FALSE,
  rating DECIMAL(3,2) DEFAULT 0.00
);
```

**Business Rules**:
- Auto-create profile on Supabase auth signup
- Providers require separate verification process
- Role-based access via RLS policies

### 4. Messaging Domain

**Purpose**: Communication between consumers and providers

**API Endpoints**:
- `GET /api/chat` - AI chat for general inquiries
- Real-time messaging via Supabase Realtime

**Data Model**:
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY,
  booking_id UUID REFERENCES bookings(id),
  sender_id UUID REFERENCES profiles(id),
  recipient_id UUID REFERENCES profiles(id),
  content TEXT NOT NULL,
  read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Business Rules**:
- Messages scoped to specific bookings
- Users can only message within their bookings
- Real-time updates via Supabase subscriptions

## Service Categories

4 fixed service categories with minimal business rules:

```sql
CREATE TYPE service_category AS ENUM (
  'food_delivery',   -- Restaurant and food delivery
  'courier',         -- Package and document delivery
  'aerial_imaging',  -- Photography and videography
  'site_mapping'     -- Surveying and construction mapping
);
```

**Category Usage**: Categories are informational only - no complex validation rules or restrictions per category.

## Geographic Focus

**Detroit Metro Area**: Platform focuses on Detroit Metro area but has minimal geographic restrictions:

```sql
-- Default service areas for reference
INSERT INTO service_areas (name, center_lat, center_lng, radius_miles) VALUES
  ('Downtown Detroit', 42.3314, -83.0458, 5),
  ('Midtown Detroit', 42.3564, -83.0666, 3),
  ('Royal Oak', 42.4895, -83.1446, 4),
  ('Dearborn', 42.3223, -83.1763, 5);
```

**Geographic Rules**:
- No enforced geographic restrictions in API
- Detroit coordinates used as defaults in UI
- Individual providers set their own `service_radius_miles`

## Payment Processing

**Stripe Integration**: Simple escrow system using Stripe Connect

```typescript
// Price calculation (from /api/orders/route.ts)
function computePriceTotal(
  listing: { price_base: number; price_per_mile: number | null; price_per_minute: number | null },
  distanceMiles?: number,
  durationMinutes?: number
) {
  const base = Number(listing.price_base || 0)
  const perMile = Number(listing.price_per_mile || 0)
  const perMin = Number(listing.price_per_minute || 0)
  const variable = (distanceMiles ?? 0) * perMile + (durationMinutes ?? 0) * perMin
  const total = base + variable
  return Math.max(0, Math.round(total * 100) / 100)
}
```

**Payment Flow**:
1. Order created → PaymentIntent created with manual capture
2. Consumer completes payment → Payment authorized (held in escrow)
3. Service completed → Payment captured and released to provider
4. Cancellation → Payment refunded based on timing

## Order Lifecycle

**Simple Status Flow**: `pending → accepted → in_progress → completed` (with `cancelled` from any state)

**Status Updates**: Manual status updates via API - no automated workflows or complex state machine

## Reviews System

**Simple Review Model**: One review per completed booking

```sql
CREATE TABLE reviews (
  id UUID PRIMARY KEY,
  booking_id UUID UNIQUE REFERENCES bookings(id), -- One review per booking
  reviewer_id UUID REFERENCES profiles(id),
  reviewed_id UUID REFERENCES profiles(id),
  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  comment TEXT,
  CHECK (reviewer_id != reviewed_id)
);
```

**Review Rules**:
- Only available after booking status = `completed`
- Both consumer and provider can review each other
- 1-5 star rating with optional comment

## Implementation Notes

**Simplicity Focus**:
- Minimal business rules enforcement in API
- Basic CRUD operations with standard validations
- Complex workflows handled manually by users
- Focus on core ecommerce functionality

**Key Patterns**:
- Zod validation for API inputs
- Supabase RLS for data access control
- Stripe Connect for payment processing
- TypeScript for type safety
- Next.js App Router for API structure

This simplified business logic documentation reflects SkyMarket's actual implementation as a straightforward drone ecommerce CRUD application focused on core marketplace functionality without complex regulatory or workflow automation.