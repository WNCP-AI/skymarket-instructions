# SkyMarket Build Guide: Step-by-Step Prompts for AI Development

This guide teaches developers how to recreate SkyMarket by prompting an AI assistant. Each step includes the exact prompt to use and what to expect.

## Overview
SkyMarket is a marketplace app built with:
- Next.js 15 (App Router)
- Supabase (Auth + Database)
- TypeScript
- Tailwind CSS + shadcn/ui
- Stripe (Payments)
- Mapbox (Maps)
- OpenAI (Chat Assistant)

## Step-by-Step Prompts

### Step 1: Project Initialization
**Prompt:**
```
Create a new Next.js 15 app with TypeScript, Tailwind CSS, and App Router. 
Name it skymarket. Then install shadcn/ui and set it up with the default theme.
```

**Expected Actions:**
```bash
npx create-next-app@latest skymarket --typescript --tailwind --app --src-dir=false
cd skymarket
npx shadcn@latest init -y
```

### Step 2: Install Core Dependencies
**Prompt:**
```
Install these dependencies for a marketplace app:
- Supabase client libraries (@supabase/ssr and @supabase/supabase-js)
- Form handling (react-hook-form, @hookform/resolvers, zod)
- Stripe (@stripe/stripe-js, stripe)
- Mapbox (mapbox-gl, @types/mapbox-gl)
- UI icons (lucide-react)
- Utilities (clsx, tailwind-merge, date-fns)
```

**Expected Actions:**
```bash
npm install @supabase/ssr @supabase/supabase-js react-hook-form @hookform/resolvers zod @stripe/stripe-js stripe mapbox-gl @types/mapbox-gl lucide-react clsx tailwind-merge date-fns
```

### Step 3: Set Up Supabase Database Schema
**Prompt:**
```
Create a Supabase database schema for a marketplace with these requirements:
1. User roles: consumer, provider, admin
2. Service categories: food_delivery, courier, aerial_imaging, site_mapping
3. Provider types: courier, drone
4. Tables needed:
   - profiles (extends auth.users with role, name, phone, address)
   - providers (user_id, type, certifications, service_areas, rating, verified)
   - listings (provider services with pricing)
   - bookings (orders with status tracking)
   - reviews (two-way rating system)
   - messages (in-app chat)
5. Include proper foreign keys and constraints
```

**Expected File:** `supabase/schema.sql`

### Step 4: Configure Supabase Client
**Prompt:**
```
Set up Supabase client configuration with:
1. Client-side client at lib/supabase/client.ts
2. Server-side client at lib/supabase/server.ts using cookies
3. Middleware at middleware.ts to handle auth and protect /dashboard routes
4. Environment variables in .env.local
```

**Expected Files:**
- `lib/supabase/client.ts`
- `lib/supabase/server.ts`
- `middleware.ts`
- `.env.local`

### Step 5: Create Type Definitions
**Prompt:**
```
Generate TypeScript types from the Supabase schema. Create a types/database.ts 
file with all the table types, enums, and helper types for the marketplace.
```

**Expected File:** `types/database.ts`

### Step 6: Build Authentication Pages
**Prompt:**
```
Create authentication pages with these requirements:
1. Login page at app/auth/login/page.tsx with email/password
2. Signup page at app/auth/signup/page.tsx with role selection (consumer/provider)
3. Use react-hook-form and zod for validation
4. Include proper error handling and loading states
5. Style with shadcn/ui components (Card, Button, Input, Form)
```

**Expected Files:**
- `app/auth/login/page.tsx`
- `app/auth/signup/page.tsx`
- `app/auth/layout.tsx`

### Step 7: Create Landing Page
**Prompt:**
```
Build a landing page at app/page.tsx with:
1. Hero section with headline about drone delivery in Detroit
2. Service category cards (Food Delivery, Courier, Aerial Imaging, Site Mapping)
3. How it works section (Search → Compare → Book → Track)
4. Call-to-action section
5. Footer with links
6. Use brand color #BD1B04 for primary actions
7. Make it responsive with Tailwind
```

**Expected File:** `app/page.tsx`

### Step 8: Implement shadcn/ui Components
**Prompt:**
```
Add these shadcn/ui components needed for the marketplace:
- button, card, form, input, label, select, textarea (forms)
- dialog, sheet, dropdown-menu (overlays)
- badge, avatar (display)
- tabs, navigation-menu (navigation)
- sonner for toast notifications
```

**Expected Actions:**
```bash
npx shadcn@latest add button card form input label select textarea dialog sheet dropdown-menu badge avatar tabs navigation-menu sonner
```

### Step 9: Create Site Header
**Prompt:**
```
Create a global site header component at components/layout/SiteHeader.tsx with:
1. Logo/brand on the left
2. Navigation links (Browse, How it Works)
3. User menu with avatar (login/signup for guests, dashboard/settings/logout for users)
4. Mobile responsive with hamburger menu
5. Use Supabase auth to check user status
```

**Expected File:** `components/layout/SiteHeader.tsx`

### Step 10: Build Browse/Search Functionality
**Prompt:**
```
Create browse functionality:
1. Browse page at app/browse/page.tsx showing all listings
2. Category pages at app/browse/[category]/page.tsx
3. API route at app/api/listings/route.ts for GET (search/filter) and POST (create)
4. Include filters for category, price range, distance
5. Display as cards with provider info, pricing, ratings
```

**Expected Files:**
- `app/browse/page.tsx`
- `app/browse/[category]/page.tsx`
- `app/api/listings/route.ts`

### Step 11: Provider Dashboard
**Prompt:**
```
Build provider dashboard at app/dashboard/provider with:
1. Main page showing stats (earnings, completed jobs, rating)
2. Services page at /services to manage listings
3. New service form at /services/new with:
   - Title, description, category selection
   - Base price, per-mile price, per-minute price
   - Service radius, availability schedule
4. Edit service at /services/[id]/edit
5. Protect routes with middleware
```

**Expected Files:**
- `app/dashboard/provider/page.tsx`
- `app/dashboard/provider/services/page.tsx`
- `app/dashboard/provider/services/new/page.tsx`
- `app/dashboard/provider/services/[id]/edit/page.tsx`

### Step 12: Consumer Dashboard
**Prompt:**
```
Create consumer dashboard at app/dashboard/consumer with:
1. Order history with status badges
2. Saved addresses
3. Favorite providers
4. Settings page for profile updates
5. Link to browse services
```

**Expected Files:**
- `app/dashboard/consumer/page.tsx`
- `app/dashboard/consumer/settings/page.tsx`

### Step 13: Implement Booking Flow
**Prompt:**
```
Build booking flow:
1. Listing detail page at app/provider/[id]/page.tsx
2. Booking page at app/booking/[listingId]/page.tsx with:
   - Date/time selection
   - Pickup address (for delivery services)
   - Dropoff address with Mapbox autocomplete
   - Special instructions
   - Price calculation
3. Stripe payment integration
4. Confirmation page at app/booking/confirmation/page.tsx
```

**Expected Files:**
- `app/provider/[id]/page.tsx`
- `app/booking/[listingId]/page.tsx`
- `app/booking/confirmation/page.tsx`

### Step 14: Add Mapbox Integration
**Prompt:**
```
Create Mapbox map component at components/map/MapboxMap.tsx with:
1. Initialize centered on Detroit (lat: 42.3314, lng: -83.0458)
2. Address search with geocoding
3. Display pickup/dropoff markers
4. Draw route between points
5. Service area visualization
6. Responsive design
```

**Expected File:** `components/map/MapboxMap.tsx`

### Step 15: Order Tracking & Status
**Prompt:**
```
Implement order tracking:
1. Order detail page at app/orders/[id]/page.tsx
2. Real-time status updates using Supabase subscriptions
3. Status progression: pending → accepted → in_progress → completed
4. Live location tracking for delivery
5. In-app messaging between consumer and provider
```

**Expected Files:**
- `app/orders/[id]/page.tsx`
- `components/realtime/MessageSubscriber.tsx`

### Step 16: Review System
**Prompt:**
```
Create review system:
1. Review page at app/orders/[id]/review/page.tsx
2. 5-star rating with comment
3. Two-way reviews (consumer rates provider, provider rates consumer)
4. Update provider rating with database trigger
5. Display reviews on provider profiles
```

**Expected Files:**
- `app/orders/[id]/review/page.tsx`
- `supabase/migrations/provider_rating_trigger.sql`

### Step 17: Stripe Payment Integration
**Prompt:**
```
Set up Stripe payments:
1. Payment utilities in lib/stripe/
2. Create payment intent on booking
3. Webhook handler at app/api/webhooks/stripe/route.ts
4. Handle payment confirmation
5. Implement escrow pattern (hold funds, release on completion)
```

**Expected Files:**
- `lib/stripe/client.ts`
- `lib/stripe/server.ts`
- `app/api/webhooks/stripe/route.ts`

### Step 18: AI Chat Assistant
**Prompt:**
```
Add AI chat assistant using OpenAI:
1. Chat API route at app/api/chat/route.ts using Vercel AI SDK
2. Chat launcher component at components/chat/ChatLauncher.tsx
3. Chat window at components/chat/ChatWindow.tsx with streaming responses
4. System prompt for marketplace assistance
5. Context about services, orders, and FAQ
```

**Expected Files:**
- `app/api/chat/route.ts`
- `components/chat/ChatLauncher.tsx`
- `components/chat/ChatWindow.tsx`

### Step 19: Provider Onboarding
**Prompt:**
```
Create provider onboarding flow:
1. Application page at app/provider/apply/page.tsx
2. Onboarding steps at app/provider/onboarding/page.tsx:
   - Identity verification
   - Service type selection
   - Certifications upload (drone license, insurance)
   - Service area selection
   - Banking details for payouts
```

**Expected Files:**
- `app/provider/apply/page.tsx`
- `app/provider/onboarding/page.tsx`

### Step 20: Final Polish
**Prompt:**
```
Add finishing touches:
1. Loading states with Suspense boundaries
2. Error boundaries for error handling
3. SEO metadata in layout files
4. Toast notifications for user feedback
5. Mobile responsive testing
6. Environment variable validation
7. README with setup instructions
```

## Testing Your Build

After each major step, test that:
1. Pages load without errors
2. Forms validate properly
3. Database operations work
4. Authentication flows correctly
5. Real-time features update
6. Payments process (use Stripe test mode)
7. Maps display properly

## Common Issues & Solutions

**Issue:** Supabase auth not working
**Solution:** Check NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_ANON_KEY in .env.local

**Issue:** TypeScript errors with database types
**Solution:** Regenerate types after schema changes using Supabase CLI

**Issue:** Mapbox not displaying
**Solution:** Ensure NEXT_PUBLIC_MAPBOX_TOKEN is set and valid

**Issue:** Stripe webhooks failing
**Solution:** Use Stripe CLI for local testing: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`

## Deployment Checklist

- [ ] All environment variables set in Vercel
- [ ] Database migrations run on Supabase
- [ ] Stripe webhook endpoint configured
- [ ] Domain configured in Vercel
- [ ] SSL certificates active
- [ ] Error monitoring set up
- [ ] Analytics configured

## Key Learning Points

1. **Server Components First**: Use client components only when needed
2. **Type Safety**: Generate types from database schema
3. **Real-time**: Leverage Supabase subscriptions
4. **Error Handling**: Always use try-catch with proper user feedback
5. **Security**: Use Row Level Security in Supabase
6. **Performance**: Implement proper caching and loading states

---

This guide provides the exact prompts needed to recreate SkyMarket step-by-step with an AI assistant.