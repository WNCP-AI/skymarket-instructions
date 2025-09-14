# Developer Track - Build SkyMarket with Spec-Driven Development

> **Master modern AI-assisted development using Cursor IDE, MCP Context7, and comprehensive specifications to build a production-ready drone service marketplace**

## What You'll Learn

This track teaches you **spec-driven development** - a modern approach where you leverage AI tools (Cursor IDE + Claude) with official library documentation (MCP Context7) and project specifications to rapidly build production applications.

Instead of manually coding everything, you'll learn to:
- 📚 Load comprehensive specifications into AI context
- 🤖 Use MCP Context7 to access official library documentation
- 💬 Write effective prompts that reference both specs and docs
- ⚡ Generate implementation code that matches requirements
- ✅ Verify output against specifications and best practices

## What You'll Build

**SkyMarket**: A full-stack drone service marketplace for Detroit Metro featuring:

- **Multi-modal Services**: Food delivery, courier, aerial imaging, site mapping
- **User Authentication**: Supabase Auth with role-based access control
- **Service Management**: Provider dashboard with FAA Part 107 compliance
- **Booking System**: Complete flow with scheduling and Detroit geofencing
- **Payment Processing**: Stripe payment processing with variable pricing
- **Real-time Features**: Live updates via Supabase subscriptions
- **Email Automation**: Transactional emails with Resend
- **Interactive Maps**: Mapbox with service area boundaries
- **Production Deployment**: Vercel with environment configuration

## Tech Stack

- **Frontend**: Next.js 15.5.2, React 19, TypeScript 5
- **Backend**: Supabase (PostgreSQL, Auth, Realtime)
- **UI**: Tailwind CSS 4 + shadcn/ui components
- **Payments**: Stripe payment processing
- **Maps**: Mapbox GL JS
- **Email**: Resend
- **AI Tools**: Cursor IDE + MCP Context7
- **Deployment**: Vercel

## Prerequisites

- ✅ Basic Next.js/React knowledge
- ✅ Understanding of TypeScript basics
- ✅ Cursor IDE installed (or willingness to install)
- ✅ GitHub account for repository access
- ✅ 12-16 hours available for completion

## The Spec-Driven Development Method

### Traditional Development
```
Idea → Manual Coding → Debug → Test → Deploy
(Slow, error-prone, inconsistent)
```

### Spec-Driven Development
```
Specs → Load Context → AI Generation → Verify → Deploy
(Fast, consistent, best practices built-in)
```

### How It Works

1. **Load Specifications**: Add project specs to Cursor context using `@docs/specs/`
2. **Access Documentation**: Use MCP Context7 for official library docs
3. **Prompt with Context**: Write prompts referencing loaded specifications
4. **Generate Code**: AI creates implementation matching requirements
5. **Verify & Iterate**: Check output against specs and refine as needed

## Track Structure - 21 Steps

### Phase 1: Foundation & Tool Setup (Steps 1-6)
Learn spec-driven development fundamentals and set up your AI-powered development environment.

1. **[Intro to Spec-Driven Development](./steps/01-intro.md)** - Understanding the methodology
2. **[Environment Setup](./steps/02-environment-setup.md)** - Accounts and prerequisites
3. **[Cursor IDE Installation](./steps/03-cursor-install.md)** - IDE setup and configuration
4. **[MCP Context7 Setup](./steps/04-mcp-context7.md)** - Library documentation server
5. **[Context Management](./steps/05-context-management.md)** - Mastering @ mentions and context
6. **[Specification Navigation](./steps/06-spec-navigation.md)** - Understanding project specs

### Phase 2: Project Setup with Specs (Steps 7-9)
Initialize the project using specifications and official documentation.

7. **[Project Initialization](./steps/07-project-init.md)** - Next.js setup with Context7
8. **[Styling System](./steps/08-styling-system.md)** - Tailwind + shadcn/ui configuration
9. **[Landing Pages](./steps/09-landing-pages.md)** - Homepage and supporting pages

### Phase 3: Database & Authentication (Steps 10-12)
Build the data layer and authentication using Supabase specifications.

10. **[Database Schema](./steps/10-database-schema.md)** - PostgreSQL with Supabase
11. **[Authentication System](./steps/11-authentication.md)** - Supabase Auth implementation
12. **[User Dashboards](./steps/12-user-dashboards.md)** - Role-based dashboard views

### Phase 4: Core Functionality (Steps 13-16)
Implement business logic and core features using multiple specifications.

13. **[CRUD Operations](./steps/13-crud-operations.md)** - Services and bookings
14. **[Booking System](./steps/14-booking-system.md)** - Complete booking flow
15. **[Maps Integration](./steps/15-maps-integration.md)** - Mapbox with geofencing
16. **[Real-time Features](./steps/16-realtime-features.md)** - Live updates and messaging

### Phase 5: Payment Integration (Steps 17-19)
Add Stripe payment processing with secure transactions.

17. **[Stripe Setup](./steps/17-stripe-setup.md)** - Payment system configuration
18. **[Payment Flow](./steps/18-payment-flow.md)** - Checkout and webhooks
19. **[Order Management](./steps/19-order-management.md)** - Order tracking and fulfillment

### Phase 6: Production Deployment (Steps 20-21)
Complete the application with email automation and deploy to production.

20. **[Email Automation](./steps/20-email-automation.md)** - Resend integration
21. **[Vercel Deployment](./steps/21-vercel-deployment.md)** - Production deployment

## Key Learning Outcomes

### Technical Skills
- 🎯 **AI-Assisted Development**: Master Cursor IDE for rapid development
- 📖 **MCP Context7**: Access and use official library documentation
- 🏗️ **Specification-Based Architecture**: Build from comprehensive specs
- 🔄 **Modern Stack Proficiency**: Next.js 15, Supabase, Stripe, TypeScript
- 🚀 **Production Deployment**: Ship real applications to Vercel

### Development Practices
- 📋 **Spec-Driven Workflow**: Requirements → Context → Implementation
- 🤖 **Prompt Engineering**: Write effective AI prompts for code generation
- ✅ **Verification Methods**: Ensure code matches specifications
- 🔍 **Documentation Navigation**: Find and use technical documentation
- 🎨 **Pattern Recognition**: Apply consistent patterns across the codebase

## Project Specifications

The power of this approach comes from comprehensive specifications. Explore these key documents:

### Core Specifications
- **[Planning Guide](../../specs/PLANNING-GUIDE.md)** - Spec-driven development methodology
- **[Product Requirements](../../PRD.md)** - Complete business requirements
- **[CLAUDE.md](../../../CLAUDE.md)** - AI assistant instructions

### Technical Specifications (13 Domains)
- **[API Endpoints](../../specs/api/API.md)** - REST API specification
- **[Authentication](../../specs/authentication/AUTHENTICATION.md)** - Auth flows and patterns
- **[Business Logic](../../specs/business-logic/BUSINESS-LOGIC.md)** - Domain rules
- **[Client-Server](../../specs/client-server/CLIENT-SERVER.md)** - Component architecture
- **[Components](../../specs/components/COMPONENTS.md)** - UI component library
- **[Database](../../specs/architecture/DATABASE.md)** - Schema and relationships
- **[Email](../../specs/email/EMAIL.md)** - Transactional email patterns
- **[Forms](../../specs/forms/FORMS.md)** - Form handling and validation
- **[Maps](../../specs/maps/MAPBOX.md)** - Mapbox integration
- **[Payment](../../specs/payment/PAYMENT.md)** - Stripe payment setup
- **[Real-time](../../specs/realtime/REALTIME.md)** - WebSocket features
- **[Security](../../specs/security/SECURITY.md)** - Auth and compliance
- **[Webhooks](../../specs/webhooks/WEBHOOKS.md)** - Event handling

## Example: Spec-Driven Development in Action

Here's how you'll build features using this methodology:

### Step 1: Load Specifications
```
# In Cursor chat (Cmd+L)
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/architecture/DATABASE.md
@CLAUDE.md
```

### Step 2: Get Official Documentation
```
/context7 get-library-docs supabase authentication
/context7 get-library-docs supabase auth-helpers-nextjs
```

### Step 3: Generate Implementation
```
Using the Supabase authentication documentation from Context7 and our
AUTHENTICATION.md specification, create the authentication system with:
- Email/password signup with verification
- Social auth (Google, GitHub)
- Password reset flow
- Profile creation on signup
Follow Supabase best practices for Next.js 15 App Router.
```

### Step 4: Verify Against Specs
```
Verify this implementation:
1. Matches all flows in AUTHENTICATION.md
2. Uses current Supabase patterns from Context7
3. Follows project conventions from CLAUDE.md
```

## Time Commitment

**Total Duration**: 12-16 hours

- **Phase 1** (Steps 1-6): 2-3 hours - Setup and learning methodology
- **Phase 2** (Steps 7-9): 2 hours - Project initialization
- **Phase 3** (Steps 10-12): 2-3 hours - Database and auth
- **Phase 4** (Steps 13-16): 3-4 hours - Core features
- **Phase 5** (Steps 17-19): 2-3 hours - Payment integration
- **Phase 6** (Steps 20-21): 1-2 hours - Production deployment

## Why This Approach?

### Traditional Tutorial Problems
- ❌ Copy-paste without understanding
- ❌ Outdated code examples
- ❌ No connection to real requirements
- ❌ Missing production considerations
- ❌ Slow, manual implementation

### Spec-Driven Development Benefits
- ✅ Understand requirements before coding
- ✅ Always use current best practices via Context7
- ✅ Generate consistent, production-ready code
- ✅ Learn transferable AI-assisted development skills
- ✅ Build faster with higher quality

## Support Resources

### Getting Help
- **Specifications**: All answers are in `docs/specs/`
- **MCP Context7**: Official library documentation
- **Troubleshooting**: Each step includes common issues
- **Verification**: Checklists ensure correctness

### Quick Links
- [Prerequisites Checklist](./prerequisites.md)
- [Resources & References](./resources.md)
- [Start Tutorial →](./steps/01-intro.md)

## Ready to Start?

You're about to learn the future of software development - where AI amplifies your capabilities and specifications drive implementation. This isn't just about building an app; it's about mastering a methodology that will transform how you develop software.

**Let's begin your journey into spec-driven development!**

---

**Next Step**: [Step 1: Introduction to Spec-Driven Development](./steps/01-intro.md) →