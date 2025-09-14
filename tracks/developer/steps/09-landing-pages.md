# Step 9: Landing Pages

> **Objective**: Create homepage and supporting pages using Context7 React patterns and component specifications to implement navigation, layout components, and establish the basic routing structure.

## Learning Outcomes

- Master Next.js 15 App Router patterns with Context7 documentation
- Implement layout components using the shadcn/ui system
- Create responsive navigation following COMPONENTS.md specifications
- Build homepage with Detroit-focused SkyMarket branding
- Set up routing structure aligned with AGENTS.md architecture

## Context Setup

### Load Project Specifications

Load these specifications into Cursor chat (`Cmd+L`):

```
@AGENTS.md
@docs/specs/components/COMPONENTS.md
@docs/PRD.md
```

### Enhanced Context7 Integration

Context7 provides access to current Next.js App Router, React Server Components, and routing documentation. Include "use context7" in implementation prompts to get official patterns for layouts, navigation, and page structure.

## Instructions

### Phase 1: Layout Foundation

#### 1. Create Root Layout

Build the main application layout using Next.js 15 patterns:

```
@AGENTS.md
@docs/specs/components/COMPONENTS.md
@docs/PRD.md

Based on our SkyMarket specifications, create comprehensive root layout for Next.js 15 App Router:

1. Update app/layout.tsx with production-ready structure:
   - Proper HTML5 semantic structure with accessibility attributes
   - Inter font loading optimization per COMPONENTS.md design system
   - Meta tags for SEO and social sharing (Detroit drone services focus)
   - Proper viewport configuration for responsive design
   - Theme provider integration for consistent styling

use context7
   - Metadata configuration for SkyMarket
   - Tailwind CSS and global styles import
   - Responsive viewport configuration

2. Configure metadata for SkyMarket:
   - Title: "SkyMarket - Drone Services Detroit"
   - Description: "Multi-modal marketplace for drone services in Detroit Metro"
   - Keywords: drone delivery, Detroit, aerial imaging, courier services
   - Open Graph tags for social sharing

Follow Server Component patterns from Context7 documentation.
```

#### 2. Build Site Header Component

Create the main navigation component:

```
Based on COMPONENTS.md SiteHeader specification (54 lines):

1. Create components/layout/SiteHeader.tsx with:
   - SkyMarket logo and branding
   - Navigation menu with role-based visibility
   - Authentication state handling (login/signup buttons)
   - Responsive mobile/desktop navigation
   - User menu for authenticated users

2. Use shadcn/ui components:
   - Navigation Menu for main nav
   - Button for auth actions
   - Avatar for user profile
   - Dropdown Menu for user menu
   - Sheet for mobile navigation drawer

3. Implement authentication state detection:
   - Show login/signup for unauthenticated users
   - Show user menu and dashboard link for authenticated users
   - Provider/consumer role differentiation

Follow the exact patterns specified in COMPONENTS.md for navigation components.
```

#### 3. Create ViewSwitcher Component

Implement role-based view switching:

```
Based on COMPONENTS.md ViewSwitcher specification (51 lines):

1. Create components/layout/ViewSwitcher.tsx with:
   - Consumer/Provider perspective toggle
   - Dropdown menu with current view indication
   - Role-based dashboard navigation
   - Responsive design for mobile/desktop

2. Features to implement:
   - Dynamic role detection
   - Smooth switching between consumer and provider views
   - Dashboard link updates based on selected role
   - State persistence across page navigation

Use shadcn/ui Select and Dropdown Menu components for the interface.
```

### Phase 2: Homepage Creation

#### 4. Design Homepage Hero Section

Create compelling hero section with Detroit focus:

```
Using PRD.md business requirements and Detroit-specific content from AGENTS.md:

1. Create app/page.tsx with hero section featuring:
   - SkyMarket value proposition
   - Detroit Metro area focus
   - Four service categories (food_delivery, courier, aerial_imaging, site_mapping)
   - Call-to-action buttons for consumers and providers

2. Hero content should include:
   - Headline: "Drone Services for Detroit Metro"
   - Subheading: Professional drone delivery and aerial services
   - Service area: 25-mile radius from downtown Detroit
   - Primary CTA: "Find Services" and "Become a Provider"

3. Use shadcn/ui components:
   - Card for service category highlights
   - Button for CTAs
   - Badge for service area indicator

Reference Detroit coordinates from AGENTS.md: lat: 42.3314, lng: -83.0458
```

#### 5. Add Service Categories Section

Showcase the four service types:

```
Based on AGENTS.md business domain and PRD.md requirements:

1. Create service categories showcase with:
   - food_delivery: Restaurant and food delivery via drone
   - courier: Package and document delivery
   - aerial_imaging: Photography, videography, inspection services
   - site_mapping: Surveying, construction site mapping, real estate

2. For each category include:
   - Service description and use cases
   - Example pricing ranges
   - Delivery timeframes
   - Detroit-specific applications

3. Implementation using:
   - Card components for each service
   - Grid layout for responsive design
   - Interactive hover states
   - Links to browse pages (to be created later)

Ensure content reflects Detroit Metro focus and real-world drone applications.
```

#### 6. Build How It Works Section

Explain the marketplace process:

```
Based on PRD.md user flows and booking system from AGENTS.md:

1. Create three-step process explanation:
   - Step 1: Browse and Select services in Detroit Metro
   - Step 2: Configure Request with location and requirements
   - Step 3: Track and Receive with real-time updates

2. Include booking state flow from AGENTS.md:
   - pending → accepted → in_progress → completed
   - Cancellation options at each stage
   - Payment escrow and release system

3. Visual elements:
   - Step numbers and arrows
   - Service category icons
   - Detroit map reference
   - Timeline visualization

Use shadcn/ui Tabs component to show different user journeys (consumer vs provider).
```

### Phase 3: Supporting Pages

#### 7. Create About Page

Build company information page:

```
Create app/about/page.tsx with:

1. SkyMarket mission and Detroit focus:
   - Company background and Detroit connection
   - Drone service marketplace vision
   - FAA Part 107 compliance and safety
   - Detroit Metro service area details

2. Team and credentials section:
   - Drone operations expertise
   - Detroit market knowledge
   - FAA compliance and safety record
   - Technology and platform capabilities

3. Use consistent layout:
   - Same header/navigation from layout
   - Professional typography and spacing
   - Card layouts for team/features
   - Call-to-action to main platform

Follow Server Component patterns for static content.
```

#### 8. Build Contact Page

Create contact and support page:

```
Create app/contact/page.tsx with:

1. Contact information:
   - Detroit-based support team
   - Business hours (EST time zone)
   - Multiple contact methods
   - Service area map reference

2. Contact form using:
   - React Hook Form integration
   - Zod validation schema
   - shadcn/ui Form components
   - Input, Textarea, Select, and Button components

3. Form fields:
   - Name and email (required)
   - Contact reason (select dropdown)
   - Message (textarea with character limit)
   - Service category interest (optional)

4. Detroit-specific content:
   - Local business support
   - Detroit Metro service coverage
   - FAA compliance questions
   - Partnership opportunities

This will require 'use client' directive due to form interactivity.
```

### Phase 4: Navigation Integration

#### 9. Connect Navigation System

Implement complete navigation flow:

```
Integrate all pages with navigation system:

1. Update SiteHeader navigation menu with:
   - Home link (logo and text)
   - Browse Services (dropdown with 4 categories)
   - How It Works (scroll anchor on homepage)
   - About page
   - Contact page
   - Dashboard (authenticated users only)

2. Add footer component:
   - Create components/layout/Footer.tsx
   - Company information and links
   - Service area information (Detroit Metro)
   - Legal links (terms, privacy - placeholders)
   - Social media placeholders

3. Mobile navigation:
   - Hamburger menu using Sheet component
   - Collapsible navigation with all main links
   - Authentication actions
   - Responsive breakpoints

Ensure all navigation follows accessibility best practices from COMPONENTS.md.
```

#### 10. Add Loading and Error Pages

Create Next.js 15 special files:

```
Using Context7 Next.js documentation:

1. Create app/loading.tsx:
   - Loading spinner for page transitions
   - SkyMarket branded loading state
   - Proper accessibility with ARIA labels

2. Create app/not-found.tsx:
   - 404 page with SkyMarket branding
   - Navigation back to homepage
   - Search suggestions or service categories

3. Create app/error.tsx:
   - Error boundary for client-side errors
   - User-friendly error messages
   - Recovery options and support contact

Use shadcn/ui components for consistent styling and responsive design.
```

## Expected Implementation

### Page Structure
```
app/
├── layout.tsx              # Root layout with Inter font and metadata
├── page.tsx               # Homepage with hero and service categories
├── about/
│   └── page.tsx          # About SkyMarket and Detroit focus
├── contact/
│   └── page.tsx          # Contact form and information
├── loading.tsx           # Loading state
├── not-found.tsx         # 404 page
└── error.tsx             # Error boundary

components/layout/
├── SiteHeader.tsx        # Main navigation (54 lines)
├── ViewSwitcher.tsx      # Role switching (51 lines)
└── Footer.tsx            # Site footer
```

### Homepage Sections
```tsx
// app/page.tsx structure
export default function HomePage() {
  return (
    <>
      <HeroSection />           // Detroit-focused value prop
      <ServiceCategoriesSection /> // 4 service types
      <HowItWorksSection />     // 3-step process
      <CtaSection />           // Provider signup CTA
    </>
  )
}
```

### SiteHeader Component
```tsx
// components/layout/SiteHeader.tsx
'use client'

import { NavigationMenu } from '@/components/ui/navigation-menu'
import { Button } from '@/components/ui/button'
import { Sheet } from '@/components/ui/sheet'
import { Avatar } from '@/components/ui/avatar'

export function SiteHeader() {
  return (
    <header className="sticky top-0 z-50 w-full border-b bg-background/95 backdrop-blur">
      <div className="container flex h-16 items-center justify-between">
        {/* Logo */}
        <div className="flex items-center space-x-2">
          <span className="text-2xl font-bold text-primary">SkyMarket</span>
        </div>

        {/* Desktop Navigation */}
        <NavigationMenu className="hidden md:flex">
          {/* Navigation items */}
        </NavigationMenu>

        {/* Auth Buttons */}
        <div className="flex items-center space-x-2">
          <Button variant="ghost">Log In</Button>
          <Button>Sign Up</Button>
        </div>

        {/* Mobile Menu */}
        <Sheet>
          {/* Mobile navigation */}
        </Sheet>
      </div>
    </header>
  )
}
```

## Verification Checklist

### Layout Components
- [ ] Root layout with proper metadata and font loading
- [ ] SiteHeader with responsive navigation and auth states
- [ ] ViewSwitcher for role-based dashboard access
- [ ] Footer with company and Detroit-specific information

### Homepage Content
- [ ] Hero section with SkyMarket value proposition
- [ ] Four service categories properly described
- [ ] How It Works section with booking flow
- [ ] Detroit Metro focus throughout content
- [ ] Responsive design on all screen sizes

### Supporting Pages
- [ ] About page with company and Detroit information
- [ ] Contact page with working form and validation
- [ ] Loading states for page transitions
- [ ] 404 and error pages with proper styling

### Navigation System
- [ ] All pages connected with proper navigation
- [ ] Mobile navigation working with Sheet component
- [ ] Accessibility features (keyboard navigation, ARIA)
- [ ] Authentication state handling (placeholders)

### Technical Implementation
- [ ] Server Components used for static content
- [ ] Client Components only where needed (forms, interactivity)
- [ ] shadcn/ui components used consistently
- [ ] TypeScript types properly defined
- [ ] Responsive design across all breakpoints

## Troubleshooting

### Common Issues

**Navigation Menu Not Responsive**
```tsx
// Check NavigationMenu viewport prop
<NavigationMenu>
  <NavigationMenuViewport />
</NavigationMenu>

// Verify mobile sheet is properly configured
<Sheet>
  <SheetTrigger asChild>
    <Button variant="ghost" size="icon" className="md:hidden">
      <Menu />
    </Button>
  </SheetTrigger>
</Sheet>
```

**Font Loading Issues**
```tsx
// Verify Inter font in layout.tsx
import { Inter } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-sans',
})

// Apply font class to body
<body className={`${inter.variable} font-sans`}>
```

**Form Validation Not Working**
```tsx
// Check React Hook Form setup
'use client'

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const formSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email'),
})
```

**Component Import Errors**
```bash
# Verify component paths
ls components/ui/
ls components/layout/

# Check tsconfig.json paths
grep -A 5 '"@/*"' tsconfig.json
```

**Context7 Integration Tips**
- For layout issues: "Fix Next.js 15 App Router layout with proper metadata and navigation structure. use context7"
- For routing problems: "Implement Next.js dynamic routing with proper loading and error boundaries. use context7"
- For component errors: "Create responsive React components using current best practices. use context7"

## Expected Outcome

After completing this step, you should have:

1. **Complete Landing Experience**: Professional homepage showcasing SkyMarket's Detroit focus
2. **Responsive Navigation**: Full navigation system with mobile and desktop support
3. **Supporting Pages**: About and Contact pages with proper forms and content
4. **Layout System**: Reusable layout components following COMPONENTS.md specifications
5. **Error Handling**: Proper loading, 404, and error pages
6. **Detroit Branding**: Consistent Detroit Metro area focus throughout content
7. **Accessibility**: WCAG-compliant navigation and interactive elements
8. **TypeScript Integration**: Fully typed components and proper Next.js patterns

The application should provide a complete user-facing experience ready for authentication and database integration in the next phase.

## Next Steps

In **Step 10: Database Schema**, you'll:
- Set up PostgreSQL database with Supabase
- Implement the complete database schema for users, services, and bookings
- Configure Row Level Security (RLS) policies
- Generate TypeScript types for database integration

---

**Continue to**: [Step 10: Database Schema](./10-database-schema.md) →