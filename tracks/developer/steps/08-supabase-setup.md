# Step 08 - Supabase Project Setup

> **Create and configure your Supabase project for the SkyMarket database, authentication, and real-time features**

## Objective
Set up a complete Supabase project with database schema, authentication, Row Level Security (RLS), and real-time subscriptions for the SkyMarket marketplace.

## What You'll Learn
- Supabase project creation and configuration
- PostgreSQL database schema design for marketplaces
- Row Level Security (RLS) implementation
- Authentication setup with custom policies
- Real-time subscriptions configuration

## Step-by-Step Instructions

### 1. Create Supabase Account and Project

1. **Sign up for Supabase**:
   - Visit [supabase.com](https://supabase.com)
   - Click "Start your project" and sign up with GitHub (recommended)

2. **Create New Project**:
   - Click "New Project"
   - Choose organization (or create one)
   - Project details:
     - **Name**: `skymarket-app`
     - **Database Password**: Generate a strong password (save it!)
     - **Region**: Choose closest to Detroit (US East recommended)
   - Click "Create new project"

3. **Wait for Setup**: Project creation takes ~2 minutes

### 2. Get Project Configuration

Once your project is ready:

1. Go to **Settings** → **API**
2. Copy the following values:
   - **Project URL**
   - **Project API Key** (anon, public)
   - **Project API Key** (service_role, secret)

3. Update your `.env.local` file:
```bash
NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_here
```

### 3. Create Database Schema

Navigate to **SQL Editor** in your Supabase dashboard and run these SQL commands:

#### 3.1 Enable Extensions
```sql
-- Enable necessary extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "postgis";
```

#### 3.2 Create Enums
```sql
-- User role types
CREATE TYPE user_role AS ENUM ('consumer', 'provider', 'admin');

-- Provider verification status
CREATE TYPE verification_status AS ENUM ('pending', 'verified', 'rejected');

-- Service categories
CREATE TYPE service_category AS ENUM (
  'food_delivery', 
  'courier', 
  'aerial_imaging', 
  'site_mapping'
);

-- Booking status
CREATE TYPE booking_status AS ENUM (
  'pending', 
  'accepted', 
  'in_progress', 
  'completed', 
  'cancelled'
);

-- Payment status
CREATE TYPE payment_status AS ENUM (
  'pending', 
  'paid', 
  'failed', 
  'refunded'
);
```

#### 3.3 Create Core Tables
```sql
-- User profiles table
CREATE TABLE profiles (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE UNIQUE NOT NULL,
  email TEXT NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  phone TEXT,
  role user_role DEFAULT 'consumer',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Providers table
CREATE TABLE providers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE UNIQUE NOT NULL,
  business_name TEXT,
  description TEXT,
  website_url TEXT,
  verification_status verification_status DEFAULT 'pending',
  part_107_cert_url TEXT,
  insurance_cert_url TEXT,
  service_areas TEXT[], -- Detroit area zones
  rating DECIMAL(3,2) DEFAULT 0.00,
  total_reviews INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Service listings table
CREATE TABLE listings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  provider_id UUID REFERENCES providers(id) ON DELETE CASCADE NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  category service_category NOT NULL,
  base_price DECIMAL(10,2) NOT NULL,
  price_per_hour DECIMAL(10,2),
  price_per_mile DECIMAL(10,2),
  duration_minutes INTEGER,
  service_radius INTEGER, -- miles from Detroit center
  active BOOLEAN DEFAULT true,
  images TEXT[],
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Service areas (Detroit Metro zones)
CREATE TABLE service_areas (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  description TEXT,
  center_lat DECIMAL(10,8) NOT NULL,
  center_lng DECIMAL(11,8) NOT NULL,
  radius_miles INTEGER NOT NULL,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Bookings table
CREATE TABLE bookings (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  consumer_id UUID REFERENCES auth.users(id) NOT NULL,
  listing_id UUID REFERENCES listings(id) NOT NULL,
  provider_id UUID NOT NULL, -- denormalized for queries
  
  -- Service details
  scheduled_at TIMESTAMPTZ NOT NULL,
  duration_minutes INTEGER,
  pickup_address TEXT,
  pickup_lat DECIMAL(10,8),
  pickup_lng DECIMAL(11,8),
  delivery_address TEXT,
  delivery_lat DECIMAL(10,8),
  delivery_lng DECIMAL(11,8),
  special_instructions TEXT,
  
  -- Status and pricing
  status booking_status DEFAULT 'pending',
  total_amount DECIMAL(10,2) NOT NULL,
  service_fee DECIMAL(10,2) NOT NULL,
  
  -- Payment
  stripe_payment_intent_id TEXT,
  payment_status payment_status DEFAULT 'pending',
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Reviews table
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES bookings(id) ON DELETE CASCADE NOT NULL,
  reviewer_id UUID REFERENCES auth.users(id) NOT NULL,
  provider_id UUID REFERENCES providers(id) NOT NULL,
  rating INTEGER CHECK (rating >= 1 AND rating <= 5) NOT NULL,
  comment TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(booking_id, reviewer_id)
);

-- Messages table
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  booking_id UUID REFERENCES bookings(id) ON DELETE CASCADE NOT NULL,
  sender_id UUID REFERENCES auth.users(id) NOT NULL,
  recipient_id UUID REFERENCES auth.users(id) NOT NULL,
  content TEXT NOT NULL,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4. Insert Detroit Service Areas

Add predefined Detroit Metro service areas:

```sql
INSERT INTO service_areas (name, description, center_lat, center_lng, radius_miles) VALUES
('Downtown Detroit', 'Downtown core and surrounding areas', 42.3314, -83.0458, 5),
('Midtown Detroit', 'Cultural center and Wayne State area', 42.3593, -83.0678, 3),
('Corktown', 'Historic neighborhood near downtown', 42.3298, -83.0583, 2),
('Eastern Market', 'Historic market district', 42.3486, -83.0404, 3),
('Belle Isle', 'Detroit River island park', 42.3397, -82.9859, 2),
('Dearborn', 'Adjacent city with large Arab-American population', 42.3223, -83.1763, 8),
('Royal Oak', 'Trendy suburb north of Detroit', 42.4895, -83.1446, 5),
('Birmingham', 'Upscale suburb', 42.5467, -83.2113, 4),
('Ferndale', 'Hip suburb north of Detroit', 42.4609, -83.1349, 3),
('Grosse Pointe', 'Affluent lakefront communities', 42.3803, -82.9118, 6);
```

### 5. Create Indexes for Performance

```sql
-- Performance indexes
CREATE INDEX idx_profiles_user_id ON profiles(user_id);
CREATE INDEX idx_providers_user_id ON providers(user_id);
CREATE INDEX idx_listings_provider_id ON listings(provider_id);
CREATE INDEX idx_listings_category ON listings(category);
CREATE INDEX idx_listings_active ON listings(active);
CREATE INDEX idx_bookings_consumer_id ON bookings(consumer_id);
CREATE INDEX idx_bookings_provider_id ON bookings(provider_id);
CREATE INDEX idx_bookings_status ON bookings(status);
CREATE INDEX idx_reviews_provider_id ON reviews(provider_id);
CREATE INDEX idx_messages_booking_id ON messages(booking_id);

-- Full text search on listings
CREATE INDEX idx_listings_search ON listings USING gin(to_tsvector('english', title || ' ' || description));
```

### 6. Set Up Row Level Security (RLS)

Enable RLS on all tables and create security policies:

```sql
-- Enable RLS on all tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE providers ENABLE ROW LEVEL SECURITY;
ALTER TABLE listings ENABLE ROW LEVEL SECURITY;
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;
ALTER TABLE reviews ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

-- Profiles policies
CREATE POLICY "Public profiles are viewable by everyone" ON profiles
  FOR SELECT USING (true);

CREATE POLICY "Users can update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own profile" ON profiles
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Providers policies
CREATE POLICY "Provider profiles are viewable by everyone" ON providers
  FOR SELECT USING (true);

CREATE POLICY "Users can update own provider profile" ON providers
  FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own provider profile" ON providers
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Listings policies
CREATE POLICY "Active listings are viewable by everyone" ON listings
  FOR SELECT USING (active = true OR (
    SELECT providers.user_id FROM providers 
    WHERE providers.id = listings.provider_id
  ) = auth.uid());

CREATE POLICY "Providers can manage own listings" ON listings
  FOR ALL USING ((
    SELECT providers.user_id FROM providers 
    WHERE providers.id = listings.provider_id
  ) = auth.uid());

-- Bookings policies
CREATE POLICY "Users can view own bookings" ON bookings
  FOR SELECT USING (
    consumer_id = auth.uid() 
    OR provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

CREATE POLICY "Consumers can create bookings" ON bookings
  FOR INSERT WITH CHECK (consumer_id = auth.uid());

CREATE POLICY "Participants can update bookings" ON bookings
  FOR UPDATE USING (
    consumer_id = auth.uid() 
    OR provider_id IN (
      SELECT id FROM providers WHERE user_id = auth.uid()
    )
  );

-- Reviews policies
CREATE POLICY "Reviews are viewable by everyone" ON reviews
  FOR SELECT USING (true);

CREATE POLICY "Users can create reviews for completed bookings" ON reviews
  FOR INSERT WITH CHECK (
    reviewer_id = auth.uid() 
    AND EXISTS (
      SELECT 1 FROM bookings 
      WHERE id = booking_id 
      AND status = 'completed'
      AND consumer_id = auth.uid()
    )
  );

-- Messages policies
CREATE POLICY "Users can view messages in their bookings" ON messages
  FOR SELECT USING (
    sender_id = auth.uid() 
    OR recipient_id = auth.uid()
  );

CREATE POLICY "Users can send messages in their bookings" ON messages
  FOR INSERT WITH CHECK (
    sender_id = auth.uid()
    AND EXISTS (
      SELECT 1 FROM bookings 
      WHERE id = booking_id 
      AND (consumer_id = auth.uid() OR provider_id IN (
        SELECT id FROM providers WHERE user_id = auth.uid()
      ))
    )
  );
```

### 7. Create Database Functions

Add helpful database functions:

```sql
-- Function to create profile after user signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (user_id, email, full_name)
  VALUES (NEW.id, NEW.email, NEW.raw_user_meta_data->>'full_name');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger to create profile on signup
CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();

-- Function to update provider rating
CREATE OR REPLACE FUNCTION update_provider_rating()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE providers 
  SET 
    rating = (
      SELECT AVG(rating)::DECIMAL(3,2) 
      FROM reviews 
      WHERE provider_id = NEW.provider_id
    ),
    total_reviews = (
      SELECT COUNT(*) 
      FROM reviews 
      WHERE provider_id = NEW.provider_id
    )
  WHERE id = NEW.provider_id;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to update rating after review
CREATE TRIGGER update_provider_rating_trigger
  AFTER INSERT ON reviews
  FOR EACH ROW
  EXECUTE FUNCTION update_provider_rating();
```

### 8. Configure Supabase Client

Create Supabase client utilities in your Next.js app:

Create `src/lib/supabase/client.ts`:
```typescript
import { createBrowserClient } from '@supabase/ssr'

export const createClient = () =>
  createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
```

Create `src/lib/supabase/server.ts`:
```typescript
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'

export const createClient = () => {
  const cookieStore = cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch (error) {
            // Handle error appropriately
          }
        },
        remove(name: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value: '', ...options })
          } catch (error) {
            // Handle error appropriately  
          }
        },
      },
    }
  )
}
```

### 9. Create TypeScript Types

Create `src/types/database.ts`:
```typescript
export type Json =
  | string
  | number
  | boolean
  | null
  | { [key: string]: Json | undefined }
  | Json[]

export interface Database {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string
          user_id: string
          email: string
          full_name: string | null
          avatar_url: string | null
          phone: string | null
          role: 'consumer' | 'provider' | 'admin'
          created_at: string
          updated_at: string
        }
        Insert: {
          id?: string
          user_id: string
          email: string
          full_name?: string | null
          avatar_url?: string | null
          phone?: string | null
          role?: 'consumer' | 'provider' | 'admin'
          created_at?: string
          updated_at?: string
        }
        Update: {
          id?: string
          user_id?: string
          email?: string
          full_name?: string | null
          avatar_url?: string | null
          phone?: string | null
          role?: 'consumer' | 'provider' | 'admin'
          created_at?: string
          updated_at?: string
        }
      }
      // Add other table types as needed...
    }
    Views: {
      [_ in never]: never
    }
    Functions: {
      [_ in never]: never
    }
    Enums: {
      user_role: 'consumer' | 'provider' | 'admin'
      verification_status: 'pending' | 'verified' | 'rejected'
      service_category: 'food_delivery' | 'courier' | 'aerial_imaging' | 'site_mapping'
      booking_status: 'pending' | 'accepted' | 'in_progress' | 'completed' | 'cancelled'
      payment_status: 'pending' | 'paid' | 'failed' | 'refunded'
    }
  }
}
```

### 10. Test Database Connection

Create a test component to verify your Supabase setup:

Create `src/components/test-connection.tsx`:
```typescript
'use client'

import { createClient } from '@/lib/supabase/client'
import { useEffect, useState } from 'react'

export function TestConnection() {
  const [isConnected, setIsConnected] = useState(false)
  const [loading, setLoading] = useState(true)
  
  useEffect(() => {
    const testConnection = async () => {
      try {
        const supabase = createClient()
        const { data, error } = await supabase
          .from('service_areas')
          .select('id, name')
          .limit(1)
        
        if (error) throw error
        setIsConnected(true)
      } catch (error) {
        console.error('Connection test failed:', error)
        setIsConnected(false)
      } finally {
        setLoading(false)
      }
    }
    
    testConnection()
  }, [])
  
  if (loading) {
    return <div>Testing database connection...</div>
  }
  
  return (
    <div className={`p-4 rounded-lg ${isConnected ? 'bg-green-50 text-green-800' : 'bg-red-50 text-red-800'}`}>
      {isConnected ? '✅ Supabase connected successfully!' : '❌ Supabase connection failed'}
    </div>
  )
}
```

Add to your `src/app/page.tsx`:
```typescript
import { TestConnection } from '@/components/test-connection'

export default function HomePage() {
  return (
    <main className="container mx-auto px-4 py-8">
      <div className="text-center">
        <h1 className="text-4xl font-bold text-gray-900 mb-4">
          Welcome to SkyMarket
        </h1>
        <p className="text-xl text-gray-600 mb-8">
          Detroit's premier drone service marketplace
        </p>
        <TestConnection />
      </div>
    </main>
  )
}
```

## Expected Outcome

After completing this step, you should have:

✅ **Complete Supabase Project**
- Database with all necessary tables and relationships
- Row Level Security policies protecting user data
- Authentication ready for user signup/login

✅ **Detroit Metro Service Areas**
- Pre-populated service area data
- Geographic boundaries for drone operations
- Ready for location-based matching

✅ **Database Connection**
- Supabase client configured for Next.js
- TypeScript types for database tables
- Test connection working successfully

## Troubleshooting

### Common Issues

**Issue: "Invalid JWT" errors**
```bash
# Check environment variables are set correctly
echo $NEXT_PUBLIC_SUPABASE_URL
echo $NEXT_PUBLIC_SUPABASE_ANON_KEY

# Restart development server
npm run dev
```

**Issue: RLS preventing data access**
```sql
-- Temporarily disable RLS for testing
ALTER TABLE table_name DISABLE ROW LEVEL SECURITY;

-- Remember to re-enable after fixing policies
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;
```

**Issue: Database connection timeout**
- Check your internet connection
- Verify Supabase project is not paused
- Check if you're using the correct region

## Next Steps

Once you've successfully completed this step:

1. **Commit your changes**:
   ```bash
   git add .
   git commit -m "Add Supabase configuration and database schema"
   ```

2. **Move to Step 9**: [Database Schema Design](./09-database-schema.md)

## Key Concepts Learned

- **PostgreSQL Setup**: Creating tables, indexes, and relationships
- **Row Level Security**: Protecting user data with database-level policies  
- **Supabase Client**: Configuring Next.js integration
- **TypeScript Integration**: Type-safe database operations
- **Geographic Data**: Storing and querying location information

---

**🎯 Success Criteria**: Your Supabase database is set up, the test connection shows success, and you can see Detroit service areas in your database.