# SkyMarket Product Requirements Document

**Version**: 4.0
**Date**: January 15, 2025
**Tech Stack**: Next.js 15.5.2 + React 19 + Supabase + Tailwind CSS 4 + Stripe Connect + OpenAI + Mapbox + Resend
**Implementation Status**: Core MVP Complete with Payment Processing and AI Integration

## Executive Summary

SkyMarket is a multi-modal drone service marketplace serving the greater Detroit metropolitan area. The platform connects consumers and businesses with verified drone operators and couriers for food delivery, package courier services, aerial imaging, and site mapping services.

### Key Value Propositions
- **For Consumers**: AI-powered service matching, transparent pricing, real-time tracking with live map, secure escrow payments
- **For Providers**: Steady income stream, professional compliance dashboard, instant payouts via Stripe Connect, AI chatbot support
- **For Market**: Unified multi-modal platform, verified operators with FAA compliance, comprehensive insurance coverage, Detroit-optimized service areas

## Business Goals

### Primary Metrics (90 Days)
- **Volume**: 1,000 completed orders/week across Food and Courier services
- **Growth**: Aerial Imaging ≥10% of GMV by month 4
- **Conversion**: Search → booking ≥20%, reorder rate ≥35%
- **Supply**: Fill rate ≥90%, provider acceptance time <2 minutes
- **Quality**: On-time completion ≥95%, dispute rate <2%
- **Financial**: Contribution margin ≥$1/order, take rate 15-20%

### Success Criteria
- **User Satisfaction**: NPS ≥60, refunds <3% of orders
- **Market Position**: Leading drone marketplace in Detroit Metro
- **Operational Excellence**: 99.9% platform uptime, <300ms API response times

## Target Users

### Primary Users

#### Consumers (Detroit Metro Residents)
- **Demographics**: Ages 25-45, household income $50K+, tech-savvy
- **Needs**: Convenience, reliability, transparent pricing, real-time updates
- **Pain Points**: Fragmented apps, hidden fees, unreliable delivery times
- **Usage Patterns**: Food delivery (daily), packages (weekly), aerial services (occasional)

#### Drone Operators
- **Profile**: FAA Part 107 certified, 1-5 years experience, independent contractors
- **Needs**: Steady income, professional platform, compliance support, fast payouts
- **Pain Points**: Finding clients, handling compliance, payment delays
- **Services**: Food delivery, courier, aerial imaging, site mapping

#### Couriers (Bike/Car)
- **Profile**: Independent contractors, flexible schedules, local knowledge
- **Needs**: Reliable income stream, fair compensation, route optimization
- **Services**: Food delivery, package courier services

### Secondary Users

#### Business Customers
- **Profile**: Small-medium businesses, real estate, construction, marketing
- **Needs**: Professional services, bulk ordering, invoicing, SLA guarantees
- **Services**: Aerial imaging, site mapping, courier services

## Core Features

### Phase 1: Foundation ✅ COMPLETE
- [x] Next.js 15.5.2 + React 19 + TypeScript + Tailwind CSS 4 setup
- [x] Supabase integration (Auth, Database, Storage, Realtime)
- [x] shadcn/ui component system with dark mode
- [x] Project structure and environment configuration
- [x] Complete database schema with RLS policies
- [x] Authentication flow with middleware and session management

### Phase 2: Core Marketplace ✅ COMPLETE
- [x] User registration and profile management
- [x] Role-based access (Consumer, Provider, Admin)
- [x] Service listing creation and management
- [x] Category-based browsing (4 service categories)
- [x] Provider profiles and service details
- [x] Geographic restrictions (Detroit Metro area)

### Phase 3: Booking System ✅ COMPLETE
- [x] Complete booking flow with validation
- [x] Date/time scheduling (24-hour advance requirement)
- [x] Location input with Detroit bounds validation
- [x] Special instructions and pickup/dropoff addresses
- [x] Booking confirmations and status management
- [x] Booking status lifecycle (pending → accepted → in_progress → completed)

### Phase 4: Payment Processing ✅ COMPLETE
- [x] Stripe Connect integration with marketplace model
- [x] Secure checkout flow with Stripe Checkout Sessions
- [x] Escrow payments with manual capture for service completion
- [x] Platform fee calculation (15% of service price)
- [x] Comprehensive webhook handling for payment events
- [x] Payment intent creation and management

### Phase 5: AI Integration ✅ COMPLETE
- [x] OpenAI GPT-4o-mini integration for customer support
- [x] AI SDK for streaming chat responses
- [x] Fallback echo mode for development
- [x] Basic chat assistant endpoint (/api/chat)
- [x] Error handling and rate limiting preparation

### Phase 6: Real-time Features 🚧 IN PROGRESS
- [x] Supabase Realtime subscriptions setup
- [x] In-app messaging system structure
- [x] Status update notifications via webhooks
- [ ] Live order tracking with maps
- [ ] Real-time location updates
- [ ] ETA calculations and notifications

### Phase 7: Advanced Features 🔄 PLANNED
- [ ] Two-sided rating and review system
- [ ] Provider verification and compliance dashboard
- [ ] Advanced search and filtering
- [ ] Mapbox integration for interactive maps
- [ ] Automated email notifications (Resend integration)
- [ ] Provider analytics dashboard
- [ ] Customer support ticket system

## Current Implementation Status (January 2025)

### ✅ Completed Core Features
- **Complete Database Schema**: 7 tables with comprehensive RLS policies and relationships
- **Authentication System**: Supabase Auth with JWT, middleware, and session management
- **Service Listings**: Full CRUD for drone and courier services with 4 categories
- **Booking System**: End-to-end booking flow with validation and status management
- **Payment Processing**: Stripe Connect with escrow, webhooks, and marketplace fees
- **AI Assistant**: OpenAI GPT-4o-mini integration with fallback modes
- **Real-time Infrastructure**: Supabase Realtime subscriptions ready

### 🔧 Current API Endpoints
- `POST /api/chat` - AI customer support assistant
- `GET/POST /api/listings` - Service listing management
- `GET /api/listings/[id]` - Individual listing details
- `GET/POST /api/orders` - Booking creation and management
- `PATCH /api/orders/[id]` - Booking status updates
- `POST /api/orders/[id]/checkout` - Stripe Checkout Session creation
- `POST /api/orders/[id]/reset-intent` - Payment intent reset
- `POST /api/webhooks/stripe` - Comprehensive Stripe webhook handler

### 🏗️ Repository Structure
```
/app                    # Next.js 15 App Router
  /(auth)              # Authentication pages
  /api                 # API routes (8 endpoints)
  /booking             # Booking flow pages
  /browse              # Service discovery
  /dashboard           # User dashboards
/components            # React components with shadcn/ui
/lib                   # Core utilities and integrations
  /supabase           # Database clients (server, client, admin)
  /stripe             # Payment processing
/types                 # TypeScript definitions
/supabase             # Database migrations and schema
/docs                 # Complete documentation
```

### 📊 Database Schema (7 Core Tables)
- **profiles**: User management with Detroit zones
- **providers**: Drone operator certifications and profiles
- **listings**: Service offerings (4 categories: food_delivery, courier, aerial_imaging, site_mapping)
- **bookings**: Complete order lifecycle management
- **reviews**: Two-sided rating system structure
- **messages**: In-app communication system
- **service_areas**: Geographic constraints for Detroit Metro

## Technical Requirements

### Frontend Architecture
- **Framework**: Next.js 15 with App Router and React 19 Server Components
- **Styling**: Tailwind CSS 4 with CSS custom properties + shadcn/ui component system
- **Language**: TypeScript with strict mode for comprehensive type safety
- **State Management**: React hooks + Supabase realtime subscriptions + URL state management
- **Forms**: react-hook-form with Zod schema validation and error handling
- **Animations**: Framer Motion + tw-animate-css for transitions and loading states
- **Theme**: next-themes for dark/light mode with system preference detection

### Backend Architecture
- **Database**: Supabase PostgreSQL with comprehensive RLS policies and performance optimization
- **Authentication**: Supabase Auth with JWT tokens, MFA support, and role-based access control
- **Storage**: Supabase Storage with image optimization and secure file handling
- **Real-time**: Supabase Realtime for live booking updates and messaging
- **API**: Next.js 15 API routes with input validation and error handling
- **Edge Functions**: Vercel Edge Functions for geographic restrictions and optimization

### Core Integrations

#### Supabase
- **Authentication**: Email/password with MFA, magic links, social login
- **Database**: PostgreSQL 15+ with RLS, performance indexes, and audit triggers
- **Storage**: Multi-bucket strategy with image transformations and encryption
- **Realtime**: WebSocket subscriptions for live tracking and messaging
- **Edge Functions**: Custom business logic and webhook processing

#### Stripe Connect
- **Payments**: PaymentIntents with 3D Secure authentication
- **Escrow**: Automatic fund holds with 24-hour release window
- **Marketplace**: Express accounts for provider instant payouts
- **Webhooks**: Comprehensive event handling with signature verification
- **Dashboard**: Provider earnings and payout management

#### Mapbox
- **Maps**: Interactive maps with Detroit-centered defaults and geofencing
- **Geocoding**: Address validation within Detroit Metro boundaries
- **Navigation**: Turn-by-turn directions for providers
- **Geofencing**: Service area enforcement and restricted zone management
- **Performance**: Efficient map rendering with caching

#### AI Integration
- **OpenAI**: GPT-4 powered customer support chatbot
- **AI SDK**: Vercel AI SDK for streaming responses and conversation management
- **Embeddings**: Vector search for intelligent service matching
- **Analytics**: AI-powered insights and performance recommendations

### Database Schema

#### Core Tables
- **profiles**: User information and roles
- **providers**: Provider-specific data and certifications
- **listings**: Service offerings and pricing
- **bookings**: Order management and tracking
- **reviews**: Two-sided feedback system
- **messages**: In-app communication

#### Supporting Tables
- **service_areas**: Detroit zone definitions
- **categories**: Service type taxonomy
- **transactions**: Payment and payout tracking
- **notifications**: System alerts and updates

## User Experience Design

### Design Principles
- **Mobile-First**: Responsive design optimized for mobile usage
- **Accessibility**: WCAG 2.1 AA compliance
- **Performance**: Sub-3-second load times, smooth interactions
- **Trust**: Clear pricing, verified providers, secure payments

### Key User Flows

#### Consumer Booking Flow
1. **Discovery**: Browse categories or search services
2. **Selection**: Compare providers, view profiles, check availability
3. **Configuration**: Select date/time, add location, special instructions
4. **Payment**: Review pricing, secure checkout, booking confirmation
5. **Tracking**: Real-time updates, messaging, delivery confirmation
6. **Completion**: Review provider, rate service, rebook option

#### Provider Management Flow
1. **Onboarding**: Profile creation, document verification, service setup
2. **Listing Management**: Create/edit services, pricing, availability
3. **Order Processing**: Accept bookings, navigate to location, update status
4. **Communication**: Message customers, provide updates
5. **Completion**: Confirm delivery, collect payment, get reviews

### UI Components

#### Consumer Interface
- **Homepage**: Category grid, search, featured providers
- **Browse**: Filter sidebar, provider cards, map view
- **Provider Detail**: Service info, pricing, calendar, reviews
- **Booking**: Multi-step form, payment, confirmation
- **Tracking**: Live map, status updates, messaging

#### Provider Interface  
- **Dashboard**: Stats, active bookings, earnings overview
- **Listings**: Service management, pricing tools
- **Calendar**: Availability management, booking queue
- **Orders**: Active job details, navigation, completion
- **Earnings**: Payment history, payout settings

## Service Categories

### Food Delivery
- **Target**: Restaurant meals, groceries, specialty items
- **Pricing**: $2.99-$7.99 delivery fee + distance/time multipliers
- **SLA**: 15-45 minute delivery windows
- **Special Features**: Temperature control, contactless delivery

### Courier Services
- **Target**: Documents, small packages, urgent items
- **Pricing**: $4.99-$12.99 base + distance/weight factors
- **SLA**: Same-day delivery, express options
- **Special Features**: Signature confirmation, fragile handling

### Aerial Imaging
- **Target**: Real estate, events, marketing content
- **Pricing**: $149-$499/hour + editing fees
- **SLA**: Weather-dependent scheduling, 24-48hr delivery
- **Special Features**: 4K video, RAW photos, portfolio galleries

### Site Mapping
- **Target**: Construction, surveying, inspection
- **Pricing**: $299-$799/project + data processing
- **SLA**: Multi-day projects, detailed deliverables
- **Special Features**: 3D mapping, CAD integration, compliance reports

## Compliance & Safety

### Drone Regulations
- **FAA Part 107**: Certificate verification required
- **Airspace**: LAANC integration for controlled airspace
- **Weather**: Real-time weather gating
- **Insurance**: Liability coverage verification

### Food Safety
- **Temperature**: Insulated compartments, monitoring
- **Hygiene**: Sanitization protocols, contactless delivery
- **Traceability**: Order tracking, delivery confirmation

### Data Security
- **Privacy**: GDPR/CCPA compliance, minimal data collection
- **Security**: Encryption in transit and at rest
- **Access**: Role-based permissions, audit logging

## Market Analysis

### Competitive Landscape
- **DoorDash/Uber**: Limited to food, no drone integration
- **TaskRabbit**: General services, no specialized drone focus
- **Direct Drone Operators**: Fragmented, no unified platform

### Market Opportunity
- **Detroit Metro**: 4.3M population, growing tech adoption
- **Drone Market**: $4.4B by 2026, 13.8% CAGR
- **Last-Mile Delivery**: $40B market, efficiency demands

### Competitive Advantages
- **Multi-Modal**: Bikes, cars, and drones on one platform
- **Compliance-First**: Built-in FAA compliance tools
- **Local Focus**: Deep Detroit market knowledge
- **Trust & Safety**: Verified operators, insurance, ratings

## Success Metrics & KPIs

### User Acquisition
- **Consumer**: 10,000 registered users in 6 months
- **Provider**: 500 active providers across categories
- **Retention**: 40% monthly active user rate

### Transaction Metrics
- **GMV**: $500K in first 6 months
- **Order Volume**: 1,000 orders/week by month 3
- **Average Order Value**: $25-$35 across categories
- **Take Rate**: 15-20% platform commission

### Quality Metrics
- **Customer Satisfaction**: 4.5+ average rating
- **Provider Performance**: 95%+ on-time completion
- **Platform Reliability**: 99.9% uptime, <300ms response
- **Support**: <2% contact rate, 24hr response time

### Financial Targets
- **Revenue**: $100K ARR by month 12
- **Unit Economics**: $2+ contribution margin per order
- **Provider Earnings**: $20-25/hour average
- **Customer Acquisition Cost**: <$25 per user

## Risk Assessment

### Technical Risks
- **Scaling**: Database performance with high transaction volume
- **Integration**: Third-party API reliability (Stripe, Mapbox)
- **Real-time**: WebSocket connection stability for tracking
- **Mobile**: Cross-platform compatibility and performance

### Market Risks
- **Competition**: Large platforms entering drone delivery
- **Regulation**: FAA rule changes affecting operations
- **Weather**: Detroit winters impacting drone operations
- **Economy**: Recession affecting discretionary spending

### Operational Risks
- **Supply**: Provider acquisition and retention
- **Quality**: Maintaining service standards at scale
- **Safety**: Incidents affecting platform reputation
- **Compliance**: Regulatory violations or insurance claims

### Mitigation Strategies
- **Technical**: Robust monitoring, fallback systems, performance optimization
- **Market**: Strong provider relationships, regulatory engagement
- **Operational**: Quality processes, insurance coverage, training programs

## Implementation Roadmap

### Phase 1: Foundation ✅ COMPLETED (Dec 2024)
- [x] Next.js 15.5.2 + React 19 + TypeScript + Tailwind CSS 4 setup
- [x] Supabase integration with comprehensive RLS policies
- [x] shadcn/ui component system with dark mode support
- [x] Complete database schema (7 tables) with relationships and triggers
- [x] Authentication flow with JWT, middleware, and role-based access
- [x] User profiles and provider onboarding structure

### Phase 2: Core Marketplace ✅ COMPLETED (Jan 2025)
- [x] Stripe Connect integration with escrow payments and marketplace model
- [x] Complete service listing CRUD with 4 service categories
- [x] End-to-end booking system with comprehensive validation
- [x] Payment processing with Stripe Checkout and webhook handling
- [x] Real-time infrastructure with Supabase subscriptions
- [x] In-app messaging system structure
- [x] AI-powered customer support chatbot with OpenAI GPT-4o-mini
- [x] Geographic restrictions for Detroit Metro area (25-mile radius)
- [x] Business logic implementation for all 4 service categories

### Phase 3: Advanced Features 🔄 IN PROGRESS (Jan-Feb 2025)
- [ ] Two-sided rating and review system implementation
- [ ] Provider verification dashboard with FAA Part 107 compliance
- [ ] Advanced search and filtering with category-specific options
- [ ] Mapbox integration for interactive maps and geofencing
- [ ] Automated email notifications using Resend
- [ ] Provider analytics and earnings dashboard
- [ ] Enhanced customer support with ticket system
- [ ] Advanced search and filtering
- [ ] Provider verification workflow
- [ ] Mobile-optimized responsive design

### Phase 3: Advanced Features (Next)
- [ ] Mapbox integration with Detroit geofencing
- [ ] Real-time location tracking for active bookings
- [ ] Advanced provider dashboard with analytics
- [ ] Review and rating system with moderation
- [ ] Push notifications and email alerts
- [ ] Performance monitoring and optimization

### Phase 4: Business Growth (Future)
- [ ] B2B features and enterprise accounts
- [ ] Advanced analytics and reporting dashboard
- [ ] API for third-party integrations
- [ ] Multi-language support and localization
- [ ] Machine learning for demand prediction
- [ ] Expansion to additional metropolitan areas

## Security & Access Control

### Roles & Permissions
- **consumer**: browse listings, create/manage own bookings, message providers, write reviews post-completion, view public profiles.
- **provider**: create/update own provider profile, create/update/delete own listings, view/manage bookings addressing their provider identity, message consumers, receive reviews.
- **admin**: operational overrides (verification, disputes), read access to most data for support; write via controlled tools only.

### RLS Summary (implemented in Supabase)
- `profiles`
  - Select: public.
  - Update: user can update own row (`auth.uid() = id`).
- `providers`
  - Select: public.
  - Insert/Update: only owner (`auth.uid() = user_id`).
- `listings`
  - Select: public if `active = true`; owners can also see inactive.
  - Insert/Update/Delete: only for listings owned by provider linked to `auth.uid()`.
- `bookings`
  - Select/Update: consumer if `consumer_id = auth.uid()`; provider if booking references their provider id.
  - Insert: only consumer creating for self (`consumer_id = auth.uid()`).
- `reviews`
  - Select: public.
  - Insert: only if booking is `completed` and reviewer is participant.
- `messages`
  - Select: sender or recipient.
  - Insert: sender must be `auth.uid()` and belong to the booking either as consumer or provider.
  - Update: recipient can mark as read.
- `service_areas`
  - Select: public.

### Auth Flows
- Email link/password (Supabase Auth). Post-signup trigger creates `profiles` row.
- Email confirmation view: `app/auth/confirm/page.tsx`.

## API Endpoints (Current Implementation)

The application primarily uses Supabase client SDK for data operations. Next.js API routes handle complex server logic, third-party integrations, and webhook processing.

### Core API Routes
- `GET /api/listings` — Search listings with filters (category, location, rating, active status)
- `GET /api/listings/[id]` — Get detailed listing with provider info and reviews
- `POST /api/orders` — Create booking order with automatic price calculation
- `GET /api/orders` — List user's orders (consumer and provider views)
- `GET /api/orders/[id]` — Get detailed order information with tracking
- `PATCH /api/orders/[id]` — Update order status and capture payments
- `POST /api/orders/[id]/checkout` — Create Stripe Checkout session

### Payment & Webhook Integration
- `POST /api/webhooks/stripe` — Handle Stripe webhook events (payments, payouts)
- `POST /api/orders/[id]/checkout` — Dynamic Stripe Checkout session creation
- Payment escrow management with automatic capture on completion

### AI Integration
- `POST /api/chat` — AI-powered customer support chatbot with streaming responses
- Context-aware responses using OpenAI GPT-4 and conversation history

### Real-time Features
- Supabase Realtime subscriptions for live order updates
- WebSocket connections for instant messaging between users
- Live location tracking during active service delivery

### Security & Validation
- Comprehensive input validation using Zod schemas
- JWT token validation for protected routes
- Role-based access control enforcement
- Rate limiting and CORS protection

## Events & Notifications

System events (emit to Realtime/in-app; email optional later):

- `booking.created` — actor: consumer; recipients: provider(s) candidate; channels: realtime, optional email.
- `booking.accepted` — actor: provider; recipients: consumer; channels: realtime.
- `booking.in_progress` — actor: provider; recipients: consumer; channels: realtime.
- `booking.completed` — actor: provider; recipients: consumer; channels: realtime; follow-up `review.requested` to consumer.
- `booking.cancelled` — actor: consumer or provider; recipients: counterparty.
- `message.sent` — actor: sender; recipients: other participant.
- `review.created` — actor: reviewer; recipients: reviewed user (provider).
- `payment.succeeded|failed|refunded` — actor: system (Stripe webhook); recipients: consumer, provider.
- `provider.verified` — actor: admin; recipients: provider.

Event payload (example):

```json
{
  "type": "booking.accepted",
  "bookingId": "uuid",
  "listingId": "uuid",
  "consumerId": "uuid",
  "providerId": "uuid",
  "timestamp": "2024-09-01T12:00:00Z"
}
```

## Analytics & Metrics

- Acquisition: signups/day, email verification rate, first booking within 7 days.
- Marketplace: search→booking conversion, AOV, completion rate, cancellation rate with reasons.
- Supply: provider acceptance time p50/p90, fill rate, active listings by category.
- Quality: average rating, dispute count, message latency.
- Performance: TTFB, API p95 latency, error rate, realtime disconnects.

Instrumentation: Next.js route handlers log to console in dev, ship to hosted logs in prod; page events via simple analytics wrapper; database-level via Supabase logs and Postgres stats.

## Operations Runbook

- Environments: local (seeded), staging, prod.
- Secrets: managed via `.env.local` in dev and platform secrets in prod.
- Backups: Supabase automated; verify weekly restoration; retention ≥14 days.
- Incident severities: Sev1 (payments down), Sev2 (booking create failing), Sev3 (non-critical).
- On-call: Slack channel alerts from uptime monitor; rotate weekly.
- Routine tasks: review RLS policy changes, check Stripe payouts reconciliation, archive old messages if needed.

## MVP Scope (Dev Preview)

- Categories supported: `food_delivery`, `courier`, `aerial_imaging`, `site_mapping` (listings exist; discovery minimal filters).
- Booking flow: single-provider listing → schedule → create booking → provider accept → complete.
- Payments: Stripe test mode `PaymentIntent` create/confirm; manual refund via dashboard.
- Realtime: in-app updates for booking status and messages.
- Reviews: single per booking, optional comment.
- Out-of-scope for MVP: route optimization, LAANC integration, payouts (use Stripe dashboard), disputes workflow UI, provider availability calendar UI.

Acceptance criteria:
- Consumer can complete E2E booking and see status transitions.
- Provider can accept, progress, and complete bookings; rating updates after review.
- RLS forbids cross-tenant access (validated with negative tests).

## Schema Alignment & Gaps

Implemented tables: `profiles`, `providers`, `listings`, `bookings`, `reviews`, `messages`, `service_areas` with enums for roles, types, categories, booking/payment status and comprehensive RLS.

Gaps vs PRD (proposed additions):
- `categories`: optional if using enum `service_category`; if user-managed taxonomy needed, add table.
- `transactions`: add for escrow/payout tracking even if Stripe is source of truth.
- `notifications`: persist in-app notifications and read-state.
- `provider_documents`: store Part 107, insurance artifacts with verification state.
- `availability`: provider availability calendar and blackout dates.
- `disputes`: basic structure for future resolution workflow.
- `service_areas` geometry: current model is center+radius; Mapbox polygons may be required later.

Indexes present for common queries; FTS on `listings(title, description)` exists in migrations.

## Open Questions & Assumptions

- Payments: escrow flow details and when funds are captured (on accept vs on complete?).
- Stripe Connect: will providers receive instant payouts (Express accounts) at launch?
- Cancellation policy: windows, fees, and who can cancel when.
- Reviews: can providers review consumers too in MVP or later only?
- Aerial jobs: weather gating and reschedule rules.
- Geofencing: how strict must service area matching be (centroid vs poly)?
- Messaging retention: how long to retain, do we support attachments?

## Appendix

### Technical Specifications
- **Performance**: <3s page load, <300ms API response
- **Scalability**: Handle 10,000 concurrent users
- **Security**: SOC 2 Type II compliance target
- **Availability**: 99.9% uptime SLA

### Regulatory Compliance
- **FAA Part 107**: Remote pilot certification
- **State Laws**: Michigan drone operation rules  
- **Local Ordinances**: Detroit city regulations
- **Insurance**: $1M liability minimum

### Business Model
- **Revenue Streams**: Transaction fees (15-20%), subscription plans (future)
- **Cost Structure**: Technology (40%), operations (30%), marketing (20%), admin (10%)
- **Unit Economics**: $2+ contribution margin target
- **Funding**: Seed round for initial development and launch

---

This PRD serves as the definitive source for SkyMarket product development and will be updated as requirements evolve through user feedback and market validation.