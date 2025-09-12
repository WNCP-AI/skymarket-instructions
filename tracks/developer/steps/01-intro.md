# Step 1: Intro - Project Overview and Objectives

**Objective**: Understand the SkyMarket project, its goals, and what you'll build during this hackathon.

## What is SkyMarket?

SkyMarket is a multi-modal marketplace for the Detroit Metro area that connects consumers with service providers offering:

- **Drone Services**: Aerial photography, site mapping, inspections
- **Courier Services**: Package delivery, document transport
- **Food Delivery**: Restaurant and grocery delivery

### Key Features You'll Build
- User authentication with consumer/provider roles
- Service listing creation and management
- Booking system with real-time status updates
- Secure payment processing with Stripe
- Email notifications
- Modern, responsive UI

## Project Context

### Target Market
- **Location**: Detroit Metropolitan area
- **Primary Users**: Consumers needing services, service providers (drone operators, couriers)
- **Business Model**: Commission-based marketplace

### Technical Goals
- Build a production-ready application
- Learn modern web development patterns
- Understand full-stack development
- Deploy to production on Vercel

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Services      │
│   Next.js 15    │────│   Supabase      │────│   Stripe        │
│   React         │    │   PostgreSQL    │    │   Resend        │
│   TypeScript    │    │   Auth          │    │   Vercel        │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Tech Stack Rationale
- **Next.js 15**: Modern React framework with excellent DX
- **Supabase**: PostgreSQL database with built-in auth and real-time features
- **TypeScript**: Type safety for better development experience
- **Stripe**: Industry-standard payment processing
- **Vercel**: Seamless deployment and hosting

## Learning Objectives

By completing this track, you will:

### Technical Skills
- Master Next.js 15 App Router patterns
- Implement secure authentication systems
- Build real-time features with Supabase
- Integrate payment processing with Stripe
- Deploy production applications

### Development Practices
- Use modern development tools (Cursor IDE)
- Write maintainable, typed code
- Understand database design principles
- Implement secure coding practices
- Follow industry best practices

### Business Understanding
- Understand marketplace business models
- Learn about payment flow design
- Understand user role management
- Learn about service-based platforms

## Project Phases Overview

### Phase 1: Foundation (Steps 1-6)
Set up development environment and understand the project structure.

### Phase 2: Project Setup (Steps 7-9)
Navigate the codebase and create initial pages with styling.

### Phase 3: Database & Auth (Steps 10-12)
Set up Supabase, implement authentication, and create user dashboards.

### Phase 4: Core Features (Steps 13-16)
Build CRUD operations, set up deployment, and integrate development tools.

### Phase 5: Payments (Steps 17-19)
Integrate Stripe, create order management, and implement advanced UI features.

### Phase 6: Production (Steps 20-21)
Add email notifications and complete production deployment.

## Success Criteria

You'll know you've succeeded when you have:

### Functional Application
- [ ] Users can register and authenticate
- [ ] Providers can create service listings
- [ ] Consumers can browse and book services
- [ ] Payments process successfully
- [ ] Email notifications send
- [ ] Application is deployed and accessible

### Technical Implementation
- [ ] Clean, typed codebase
- [ ] Secure authentication
- [ ] Proper error handling
- [ ] Responsive design
- [ ] Production-ready deployment

### Learning Outcomes
- [ ] Confidence with Next.js and React
- [ ] Understanding of full-stack development
- [ ] Knowledge of authentication patterns
- [ ] Experience with payment integration
- [ ] Production deployment experience

## What Makes This Different

### Real-World Focus
- Based on actual marketplace patterns
- Production-ready code and practices
- Industry-standard tools and services

### Guided Learning
- Step-by-step instructions with explanations
- Code examples with context
- Troubleshooting guidance
- Best practice explanations

### Modern Stack
- Latest Next.js features
- TypeScript throughout
- Modern tooling (Cursor IDE)
- Current industry patterns

## Getting Started Mindset

### Embrace Learning
- Every step builds on the previous
- Ask questions and experiment
- Use the provided resources
- Don't worry about perfection

### Focus on Understanding
- Read the explanations, don't just copy code
- Understand *why* we make certain choices
- Learn the patterns, not just the syntax
- Build confidence through practice

### Practical Application
- This is a real project you can extend
- The patterns apply to other projects
- The skills transfer to professional work
- The deployment can be showcased

## Expected Time Investment

- **Total Time**: 12-16 hours
- **Per Step**: 30-90 minutes average
- **Flexibility**: Work at your own pace
- **Support**: Detailed troubleshooting provided

## Next Steps

Ready to begin? Let's set up your development environment and get you coding!

**Prerequisites Check**: Before proceeding, ensure you've completed the [Prerequisites](../prerequisites.md) checklist.

---

**Next Step**: [Step 2: Environment Setup](./02-environment-setup.md)

**Questions?** Review the project overview or check the [Resources](../resources.md) for additional context.