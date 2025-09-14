# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **SkyMarket Documentation Hub** - comprehensive specifications and learning tracks for building a production-ready drone service marketplace platform serving the Detroit Metropolitan Area.

**Key Purpose**: This repository contains complete documentation and learning tracks for SkyMarket, a multi-modal marketplace connecting consumers with drone operators and couriers for food delivery, package courier services, aerial imaging, and site mapping services.

## Project Architecture

### Documentation Structure

```
├── README.md                      # Main documentation index
├── PRD.md                         # Product Requirements Document (MVP Complete)
└── specs/                         # Technical Specifications (13 domains)
    ├── PLANNING-GUIDE.md          # Spec-driven development methodology
    ├── api/API.md                 # REST API endpoints (8 implemented)
    ├── architecture/              # System design and database schema
    ├── authentication/            # Supabase Auth patterns
    ├── business-logic/            # Detroit-specific rules and constraints
    ├── client-server/             # Next.js component patterns
    ├── components/                # shadcn/ui component library
    ├── deployment/                # Vercel deployment guide
    ├── email/                     # Resend integration
    ├── forms/                     # Form handling patterns
    ├── maps/                      # Mapbox integration
    ├── payment/                   # Stripe payment processing
    ├── realtime/                  # Supabase subscriptions
    ├── security/                  # Authentication & compliance
    └── webhooks/                  # Stripe webhook handling
```

### Learning Tracks

```
tracks/
├── developer/                     # Spec-driven development (21 steps)
│   ├── README.md                 # Next.js 15 + Supabase + AI tools
│   └── steps/                    # Step-by-step implementation
└── expert/                       # Advanced AI integration (5 steps)
    ├── README.md                 # OpenAI + Cursor mastery
    └── steps/                    # Expert patterns and tools
```

## Technology Stack

- **Frontend**: Next.js 15.5.2, React 19, TypeScript 5, Tailwind CSS 4, shadcn/ui
- **Backend**: Supabase (PostgreSQL, Auth, Storage, Realtime)
- **Payments**: Stripe payment processing for services
- **Maps**: Mapbox GL JS with Detroit Metro geofencing
- **Email**: Resend for transactional emails
- **AI Tools**: Cursor IDE, MCP Context7, OpenAI GPT-4o-mini
- **Deployment**: Vercel

## Core Business Context

### Service Categories (Exactly 4)
1. **Food Delivery** (`food_delivery`) - Restaurant/grocery delivery
2. **Courier Services** (`courier`) - Document/package delivery
3. **Aerial Imaging** (`aerial_imaging`) - Real estate/event photography
4. **Site Mapping** (`site_mapping`) - Construction surveys/3D mapping

### Geographic Constraints
- **Service Area**: Detroit Metropolitan Area only
- **Radius**: 25-mile radius from downtown Detroit (42.3314°N, 83.0458°W)
- **Compliance**: FAA Part 107 certification required for drone operators

### Implementation Status (January 2025)
✅ **COMPLETED MVP Features**:
- Complete database schema (7 tables with RLS policies)
- Authentication & authorization system
- Service listing management
- End-to-end booking flow
- Stripe payment processing with secure checkout
- AI customer support (OpenAI GPT-4o-mini)
- Real-time infrastructure
- 8 implemented API endpoints

## Development Workflows

### Spec-Driven Development Method

This repository demonstrates a **specification-driven development approach** where comprehensive documentation guides AI-assisted implementation:

1. **Load Context**: Reference relevant specifications using `@` mentions
2. **Use Official Docs**: Leverage MCP Context7 for current best practices
3. **Generate Code**: Create implementations matching specifications
4. **Verify**: Check output against documented requirements

### Key Development Commands

Since this is a documentation repository, there are no build/test commands. However, when implementing the actual SkyMarket application, use these patterns:

```bash
# Project initialization (from specs)
npx create-next-app@latest skymarket --typescript --tailwind --app

# Database setup (Supabase)
npx supabase start
npx supabase db reset  # Uses migrations in specs/

# Development
npm run dev

# Code quality
npm run lint
npx tsc --noEmit  # TypeScript checking

# Build verification
npm run build
```

## High-Level Architectural Patterns

### Database Design (PostgreSQL + Supabase)
- **Core Tables**: profiles, providers, listings, bookings, reviews, messages, service_areas
- **Security**: Row Level Security (RLS) on all user data
- **Performance**: Strategic indexing for marketplace queries
- **Relationships**: Normalized with foreign keys, proper constraints

### API Architecture (Next.js 15 App Router)
```
api/
├── chat/route.ts              # AI customer support
├── listings/                  # Service discovery
├── orders/                    # Booking management
├── orders/[id]/checkout/      # Stripe Checkout
└── webhooks/stripe/           # Payment processing
```

### Authentication Flow
- **Supabase Auth**: Email/password, social login
- **Role-based Access**: consumer, provider, admin
- **Middleware**: JWT validation on protected routes
- **RLS Policies**: Database-level access control

### Payment Architecture
- **Stripe Payments**: Secure checkout for services
- **Payment Flow**: Direct payment processing
- **Platform Fee**: 15% transaction fee
- **Webhooks**: Comprehensive event handling

## Common Implementation Patterns

### Loading Specifications
When working with implementation tasks, always load relevant specs:

```
# For authentication work
@specs/authentication/AUTHENTICATION.md
@specs/security/SECURITY.md
@specs/architecture/DATABASE.md

# For payment integration
@specs/payment/PAYMENT.md
@specs/webhooks/WEBHOOKS.md
@specs/business-logic/BUSINESS-LOGIC.md

# For UI components
@specs/components/COMPONENTS.md
@specs/client-server/CLIENT-SERVER.md
@specs/forms/FORMS.md
```

### Business Logic Validation
Always validate implementations against business rules:
- Detroit Metro area service boundaries (25-mile radius)
- Service category constraints (exactly 4 types)
- Booking requirements (24-hour advance scheduling)
- Payment processing (15% platform fee, escrow handling)
- Provider verification (FAA Part 107 compliance)

### Error Handling Patterns
- **Client-side**: React Error Boundaries, form validation with Zod
- **Server-side**: Structured error responses, proper HTTP status codes
- **Database**: RLS policy violations, constraint errors
- **External APIs**: Stripe, Mapbox, Resend error handling

## File Organization Guidelines

### Specification Management
Each specification domain includes:
- **Implementation details**: How features are currently built
- **Integration patterns**: How components connect
- **Business rules**: Domain-specific constraints
- **Future enhancements**: Planned improvements

### Documentation Standards
- Use clear, actionable language
- Include code examples where relevant
- Reference other specifications for context
- Update with implementation changes

## Quality and Testing

### Validation Approach
- **Specification Compliance**: Verify against documented requirements
- **Pattern Consistency**: Follow established architectural patterns
- **Security**: Validate RLS policies and auth flows
- **Performance**: Check database queries and API response times

### Code Standards
- **TypeScript**: Strict mode, comprehensive type definitions
- **Component Patterns**: Server/Client component separation
- **Error Handling**: Comprehensive error boundaries and validation
- **Accessibility**: WCAG 2.1 AA compliance

## MCP Context7 Integration

This repository is optimized for use with MCP Context7 server for accessing official library documentation:

### Common Context7 Queries
```
# Supabase patterns
/context7 get-library-docs supabase auth-helpers-nextjs
/context7 get-library-docs supabase realtime-js

# Next.js best practices
/context7 get-library-docs nextjs app-router
/context7 get-library-docs nextjs middleware

# Stripe integration
/context7 get-library-docs stripe node
/context7 get-library-docs stripe webhooks
```

## Important Notes

### Documentation Repository
This is a **documentation and learning repository**, not the actual SkyMarket application codebase. It contains:
- Complete specifications for building SkyMarket
- Learning tracks for spec-driven development
- Architectural guidance and patterns

### Implementation Context
When working with these specifications:
- Reference multiple related specs for comprehensive context
- Use MCP Context7 for current best practices
- Follow the spec-driven development methodology
- Validate against business rules and constraints

### Geographic Focus
All features and business logic are specifically designed for the **Detroit Metropolitan Area** with strict geographic constraints and local compliance requirements.