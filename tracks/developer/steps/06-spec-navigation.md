# Step 6: Specification Navigation

## Objective

Master the SkyMarket specifications structure and learn to efficiently navigate and combine specification documents for comprehensive feature development.

## Understanding the Specs Architecture

The SkyMarket project includes 13+ comprehensive specification domains organized for maximum development efficiency.

### Core Specifications Overview

```
docs/specs/
├── PLANNING-GUIDE.md           # Meta: How to use specs
├── api/
│   ├── API.md                  # REST endpoints & contracts
├── authentication/
│   └── AUTHENTICATION.md       # Auth flows & patterns
├── architecture/
│   ├── ARCHITECTURE.md         # System design
│   └── DATABASE.md             # Schema & relationships
├── business-logic/
│   └── BUSINESS-LOGIC.md       # Domain rules & constraints
├── client-server/
│   └── CLIENT-SERVER.md        # Component architecture
├── components/
│   └── COMPONENTS.md           # UI component library
├── deployment/
│   └── DEPLOYMENT.md           # Vercel & environment setup
├── email/
│   └── EMAIL.md                # Resend integration patterns
├── forms/
│   └── FORMS.md                # Form handling & validation
├── maps/
│   └── MAPBOX.md              # Interactive mapping
├── payment/
│   └── PAYMENT.md             # Stripe payment processing
├── webhooks/
│   └── WEBHOOKS.md             # Event handling patterns
└── security/
    └── SECURITY.md            # Auth, RLS, compliance
```

## Specification Categories

### 1. Foundation Specs (Always Reference)

**PLANNING-GUIDE.md**
- Spec-driven development methodology
- How to use specifications effectively
- Documentation patterns and best practices

**AGENTS.md** (Project Root)
- AI assistant instructions
- Project conventions and patterns
- Development workflow guidelines

### 2. Core Architecture Specs

**architecture/DATABASE.md**
- Complete PostgreSQL schema
- Table relationships and constraints
- RLS policies and security
- Migration strategies

**architecture/ARCHITECTURE.md**
- System design overview
- Component relationships
- Scalability considerations

**business-logic/BUSINESS-LOGIC.md**
- Service categories (food_delivery, courier, aerial_imaging, site_mapping)
- Booking state machines
- Pricing rules and calculations
- Detroit Metro constraints

### 3. Integration Specs

**authentication/AUTHENTICATION.md**
- Supabase Auth implementation
- Multi-role system (consumer, provider, admin)
- Session management
- Social auth patterns

**payment/PAYMENT.md**
- Stripe payment processing
- Secure transaction handling with variable pricing
- Webhook management
- Payment confirmation automation

**realtime/REALTIME.md**
- Supabase subscriptions
- Live booking updates
- Real-time messaging
- Connection management

**email/EMAIL.md**
- Resend integration
- Transactional email flows
- Template management
- Notification triggers

### 4. User Interface Specs

**components/COMPONENTS.md**
- UI component library (shadcn/ui based)
- Design system patterns
- Accessibility requirements
- Component composition

**forms/FORMS.md**
- Server Actions patterns
- Validation with Zod
- Error handling
- Form state management

**client-server/CLIENT-SERVER.md**
- Next.js 15 App Router patterns
- Server Components vs Client Components
- Data fetching strategies
- State management

### 5. External Integration Specs

**maps/MAPBOX.md**
- Interactive map implementation
- Detroit Metro geofencing
- Service area visualization
- Location validation

**api/API.md**
- REST endpoint specification
- Authentication middleware
- Error response patterns
- Rate limiting

**deployment/DEPLOYMENT.md**
- Vercel deployment configuration
- Environment variables
- CI/CD pipeline
- Performance optimization

## Navigation Strategies

### 1. Feature-First Navigation

When implementing a feature, identify all relevant specifications:

**Example: User Authentication Feature**
```
Primary Spec:
@docs/specs/authentication/AUTHENTICATION.md

Supporting Specs:
@docs/specs/architecture/DATABASE.md          # User schema
@docs/specs/email/EMAIL.md                    # Verification emails
@docs/specs/security/SECURITY.md              # Security requirements
@docs/specs/forms/FORMS.md                    # Auth forms
```

**Example: Payment Integration Feature**
```
Primary Spec:
@docs/specs/payment/PAYMENT.md

Supporting Specs:
@docs/specs/business-logic/BUSINESS-LOGIC.md  # Pricing rules
@docs/specs/api/WEBHOOKS.md                   # Stripe webhooks
@docs/specs/email/EMAIL.md                    # Payment receipts
@docs/specs/security/SECURITY.md              # Payment security
```

### 2. Layer-by-Layer Navigation

**Foundation Layer** (Always Load First):
```
@docs/specs/PLANNING-GUIDE.md
@AGENTS.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/architecture/DATABASE.md
```

**Feature Layer** (Add Based on Feature):
```
# For auth features:
@docs/specs/authentication/AUTHENTICATION.md

# For payment features:
@docs/specs/payment/PAYMENT.md
```

**Implementation Layer** (Technical Details):
```
# For API development:
@docs/specs/api/API.md

# For UI development:
@docs/specs/components/COMPONENTS.md
@docs/specs/client-server/CLIENT-SERVER.md

# For forms:
@docs/specs/forms/FORMS.md
```

### 3. Domain-Driven Navigation

**Frontend Domain**:
```
@docs/specs/components/COMPONENTS.md
@docs/specs/client-server/CLIENT-SERVER.md
@docs/specs/forms/FORMS.md
@docs/specs/maps/MAPBOX.md
```

**Backend Domain**:
```
@docs/specs/architecture/DATABASE.md
@docs/specs/api/API.md
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/security/SECURITY.md
```

**Integration Domain**:
```
@docs/specs/payment/PAYMENT.md
@docs/specs/email/EMAIL.md
@docs/specs/realtime/REALTIME.md
@docs/specs/api/WEBHOOKS.md
```

## Practical Navigation Examples

### Example 1: Implementing User Registration

**Step 1: Load Core Specifications**
```
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/architecture/DATABASE.md
@AGENTS.md
```

**Step 2: Add Supporting Specifications**
```
@docs/specs/email/EMAIL.md                    # Email verification
@docs/specs/forms/FORMS.md                    # Registration form
@docs/specs/security/SECURITY.md              # Password requirements
```

**Step 3: Implementation Prompt**
```
Based on the authentication specification and supporting specs:

1. Implement user registration with email verification
2. Create user profile on signup (per DATABASE.md schema)
3. Send verification email using patterns from EMAIL.md
4. Use form patterns from FORMS.md for the UI
5. Follow security requirements from SECURITY.md

Use Supabase's current best practices for Next.js 15, Resend for emails.

use context7
```

### Example 2: Building Booking System

**Step 1: Business Logic Foundation**
```
@docs/specs/business-logic/BUSINESS-LOGIC.md  # Service categories, state machine
@docs/specs/architecture/DATABASE.md          # Booking schema
```

**Step 2: Feature-Specific Specs**
```
@docs/specs/maps/MAPBOX.md                    # Location handling
@docs/specs/payment/PAYMENT.md               # Payment processing
@docs/specs/realtime/REALTIME.md             # Status updates
```

**Step 3: Implementation Specs**
```
@docs/specs/forms/FORMS.md                   # Booking forms
@docs/specs/components/COMPONENTS.md         # UI components
@docs/specs/api/API.md                       # Booking endpoints
```

**Step 4: Implementation Prompt**
```
Based on our booking system specification and supporting requirements:

1. Implement booking flow with real-time status updates
2. Include map integration for location selection
3. Handle Stripe payment processing
4. Use form patterns and UI components from our specs

Use current best practices for Mapbox, Supabase Realtime, and Stripe.

use context7
```

### Example 3: Payment Integration

**Step 1: Core Payment Specs**
```
@docs/specs/payment/PAYMENT.md               # Primary specification
@docs/specs/business-logic/BUSINESS-LOGIC.md # Pricing rules
```

**Step 2: Integration Requirements**
```
@docs/specs/api/WEBHOOKS.md                  # Stripe webhooks
@docs/specs/email/EMAIL.md                   # Payment receipts
@docs/specs/security/SECURITY.md             # Payment security
```

**Step 3: Implementation Details**
```
@docs/specs/architecture/DATABASE.md         # Payment schema
@docs/specs/components/COMPONENTS.md         # Payment UI
```

## Specification Relationships

### Understanding Dependencies

**Core Dependencies**:
- Most features depend on `business-logic/BUSINESS-LOGIC.md`
- UI features depend on `components/COMPONENTS.md`
- Data features depend on `architecture/DATABASE.md`

**Integration Dependencies**:
- Auth features → email, security, forms
- Payment features → webhooks, email, business-logic
- Real-time features → authentication, business-logic

**Implementation Dependencies**:
- Frontend features → client-server, components, forms
- Backend features → api, database, security
- Full-stack features → multiple domains

### Conflict Resolution

When specifications seem to conflict:

1. **Consult PLANNING-GUIDE.md** for resolution strategies
2. **Business logic takes precedence** over implementation preferences
3. **Security requirements override** convenience features
4. **Ask for clarification** in prompts when unclear

## Advanced Navigation Techniques

### 1. Selective Context Loading

Don't load entire specifications - load relevant sections:

```
# Instead of loading all of DATABASE.md:
@docs/specs/architecture/DATABASE.md

# Ask for specific sections:
From the database specification, show me only the user-related tables and the booking state machine.
```

### 2. Cross-Reference Patterns

```
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/business-logic/BUSINESS-LOGIC.md

Show how user roles from the auth spec relate to service provider requirements in the business logic spec.
```

### 3. Implementation Path Planning

```
@docs/specs/payment/PAYMENT.md
@docs/specs/api/WEBHOOKS.md
@docs/specs/email/EMAIL.md

Plan the implementation sequence for Stripe payment integration, including webhook setup and email notifications.
```

## Expected Outcome

After completing this step, you should have:

### Navigation Mastery
- [ ] Understand the 13+ specification domains
- [ ] Can identify which specs are needed for any feature
- [ ] Know how specifications relate to each other
- [ ] Can navigate efficiently without getting overwhelmed

### Strategic Loading
- [ ] Use feature-first navigation approach
- [ ] Apply layer-by-layer context building
- [ ] Combine related specifications effectively
- [ ] Optimize context for specific tasks

### Practical Application
- [ ] Plan complex features using multiple specs
- [ ] Resolve specification dependencies
- [ ] Handle apparent conflicts between specs
- [ ] Create implementation roadmaps from specifications

## Pro Tips

### Efficient Navigation
- **Start broad, get specific**: Begin with business logic, drill down to implementation
- **Follow the data**: Database schema often reveals spec relationships
- **Think user journeys**: User flows span multiple specification domains
- **Use PLANNING-GUIDE.md**: It contains navigation strategies and patterns

### Context Optimization
- **Load incrementally**: Start with core specs, add as needed
- **Reference selectively**: Pull specific sections rather than entire files
- **Cross-reference actively**: Look for relationships between specifications
- **Verify understanding**: Ask AI to explain spec relationships before coding

### Implementation Strategy
- **Map dependencies first**: Understand which specs depend on others
- **Plan integration points**: Identify where specifications intersect
- **Sequence carefully**: Some implementations must happen before others
- **Test integration**: Verify that multi-spec implementations work together

## Next Steps

Perfect! You now understand how to navigate and combine the comprehensive SkyMarket specifications. Next, we'll start building by initializing our Next.js project using specifications and Context7.

### What's Coming in Step 7
- Initialize Next.js 15 project with specifications
- Set up TypeScript configuration
- Configure project structure following specs
- Prepare for styling system implementation

---

**Next Step**: [Step 7: Project Initialization](./07-project-init.md) →

**Quick Links**:
- [Previous: Context Management](./05-context-management.md)
- [Track Overview](../README.md)
- [Project Specifications](../../../specs/)