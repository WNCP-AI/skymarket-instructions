# SkyMarket Product Requirements Document

**Version**: 2.0  
**Date**: September 1, 2024  
**Tech Stack**: Next.js + Supabase + Tailwind CSS

## Executive Summary

SkyMarket is a multi-modal drone service marketplace serving the greater Detroit metropolitan area. The platform connects consumers and businesses with verified drone operators and couriers for food delivery, package courier services, aerial imaging, and site mapping services.

### Key Value Propositions
- **For Consumers**: Compare providers, transparent pricing, real-time tracking, secure payments
- **For Providers**: Steady income, professional platform, instant payouts, compliance tools
- **For Market**: Unified platform eliminating app fragmentation, verified operators, insured services

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

### Phase 1: Foundation (Weeks 1-2)
- [x] Next.js 14 + TypeScript + Tailwind CSS setup
- [x] Supabase integration (Auth, Database, Storage)
- [x] shadcn/ui component system
- [x] Project structure and environment configuration
- [x] Database schema design
- [] Basic authentication flow

### Phase 2: User Management (Weeks 3-4)
- [ ] User registration and profiles
- [ ] Role-based access (Consumer, Provider, Admin)
- [ ] Profile management and settings
- [ ] Email verification and password reset

### Phase 3: Provider System (Weeks 5-7)
- [ ] Provider onboarding flow
- [ ] Document verification (Part 107, insurance)
- [ ] Service listing creation and management
- [ ] Availability calendar
- [ ] Service area configuration

### Phase 4: Marketplace Core (Weeks 8-10)
- [ ] Service discovery and search
- [ ] Provider comparison and filtering
- [ ] Detailed provider profiles
- [ ] Category-based browsing
- [ ] Location-based service matching

### Phase 5: Booking System (Weeks 11-13)
- [ ] Service booking flow
- [ ] Date/time scheduling
- [ ] Location input and validation
- [ ] Special instructions
- [ ] Booking confirmations

### Phase 6: Payment Processing (Weeks 14-15)
- [ ] Stripe integration
- [ ] Secure checkout flow
- [ ] Escrow payments
- [ ] Fee calculation and display
- [ ] Refund processing

### Phase 7: Real-time Features (Weeks 16-18)
- [ ] Live order tracking
- [ ] In-app messaging
- [ ] Status updates and notifications
- [ ] Map integration with Mapbox
- [ ] ETA calculations

### Phase 8: Reviews & Trust (Weeks 19-20)
- [ ] Two-sided rating system
- [ ] Review collection and display
- [ ] Provider verification badges
- [ ] Dispute resolution system

## Technical Requirements

### Frontend Architecture
- **Framework**: Next.js 14 with App Router
- **Styling**: Tailwind CSS 3 + shadcn/ui components
- **Language**: TypeScript for type safety
- **State Management**: React hooks + Supabase realtime
- **Forms**: react-hook-form with zod validation

### Backend Architecture
- **Database**: Supabase PostgreSQL with RLS policies
- **Authentication**: Supabase Auth with JWT tokens
- **Storage**: Supabase Storage for media files
- **Real-time**: Supabase Realtime subscriptions
- **API**: Next.js API routes + Supabase client

### Core Integrations

#### Supabase
- **Authentication**: Email/password, social login
- **Database**: PostgreSQL with Row Level Security
- **Storage**: Media uploads (images, documents)
- **Realtime**: Live updates for tracking and messaging

#### Stripe
- **Payments**: Secure payment processing
- **Escrow**: Hold funds until service completion
- **Payouts**: Provider instant payouts
- **Subscriptions**: Future B2B plans

#### Mapbox
- **Maps**: Interactive maps with Detroit focus
- **Geocoding**: Address validation and search
- **Routing**: Delivery route optimization
- **Geofencing**: Service area boundaries

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

### Months 1-2: Foundation
- Core platform development (Next.js + Supabase)
- User authentication and basic profiles
- Provider onboarding system
- Payment integration (Stripe)

### Months 3-4: Marketplace
- Service discovery and booking
- Real-time tracking and messaging
- Review and rating system
- Mobile optimization

### Months 5-6: Growth
- Marketing and user acquisition
- Provider recruitment and training
- Performance optimization
- Advanced features (scheduling, batching)

### Months 7-12: Scale
- B2B features and accounts
- Analytics and reporting
- API for third-party integrations
- Expansion planning

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

## API Endpoints (MVP)

Note: Client primarily uses Supabase client SDK. These Next.js route handlers wrap complex server logic or third-party calls.

- `GET /api/listings` — query listings with filters: `category`, `q`, `active`.
- `GET /api/listings/:id` — get listing detail with provider summary and rating.
- `POST /api/bookings` — create booking from listing; validates service area and schedule.
- `GET /api/bookings` — list own bookings (role-aware).
- `PATCH /api/bookings/:id/status` — consumer cancel; provider accept/in_progress/completed.
- `POST /api/messages` — send message within a booking thread.
- `GET /api/messages?bookingId=...` — fetch conversation for booking.
- `POST /api/reviews` — create review post-completion.
- `GET /api/service-areas` — list Detroit zones.
- `POST /api/payments/intent` — create Stripe PaymentIntent (test mode in MVP).

Minimal request/response formats will mirror DB shapes with zod validation on input.

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