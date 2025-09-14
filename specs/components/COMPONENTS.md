# COMPONENTS.md - UI Component Library Specification

## Overview

SkyMarket utilizes a comprehensive component architecture built on shadcn/ui and Radix UI primitives, providing consistent design patterns and reusable UI elements across the application.

## Component Architecture

### Design System Foundation
- **Base**: shadcn/ui components with Tailwind CSS 4 styling
- **Primitives**: Radix UI for accessibility and behavior
- **Customization**: Class Variance Authority (CVA) for variant management
- **Consistency**: Unified design tokens and styling patterns

### Component Categories

#### Core UI Components (`/components/ui/`)
**15 foundational shadcn/ui components providing the design system foundation:**

1. **Button** (`button.tsx`) - 59 lines
   - 6 variants: default, destructive, outline, secondary, ghost, link
   - 4 sizes: default, sm, lg, icon
   - Accessibility features with focus states and ARIA support

2. **Card** (`card.tsx`) - 92 lines
   - Structured content containers
   - Components: Card, CardHeader, CardTitle, CardDescription, CardContent, CardFooter

3. **Dialog** (`dialog.tsx`) - 143 lines
   - Modal overlays and popups
   - Accessible with focus management and ESC key handling

4. **Input** (`input.tsx`)
   - Form input fields with validation states
   - Consistent styling with error/success states

5. **Label** (`label.tsx`)
   - Form labels with proper association
   - Required/optional indicators

6. **Select** (`select.tsx`) - 185 lines
   - Dropdown selection components
   - Keyboard navigation and search support

7. **Textarea** (`textarea.tsx`)
   - Multi-line text input
   - Auto-resize capabilities

8. **Form** (`form.tsx`) - 167 lines
   - React Hook Form integration
   - Validation state management
   - Field context and error handling

9. **Sonner** (`sonner.tsx`)
   - Toast notification system
   - Multiple notification types and positioning

10. **Avatar** (`avatar.tsx`) - 53 lines
    - User profile images with fallbacks
    - Placeholder initials generation

11. **Badge** (`badge.tsx`)
    - Status indicators and labels
    - Color variants for different contexts

12. **Dropdown Menu** (`dropdown-menu.tsx`) - 257 lines
    - Context menus and action lists
    - Keyboard navigation and nested menus

13. **Navigation Menu** (`navigation-menu.tsx`) - 168 lines
    - Site navigation components
    - Responsive mobile/desktop variations

14. **Sheet** (`sheet.tsx`) - 139 lines
    - Slide-out panels and sidebars
    - Mobile-friendly drawer behavior

15. **Tabs** (`tabs.tsx`) - 66 lines
    - Tabbed content organization
    - Accessible tab navigation

#### Layout Components (`/components/layout/`)

**SiteHeader** (`SiteHeader.tsx`) - 54 lines
- Main application header with authentication state
- Navigation menu with role-based visibility
- Authentication buttons and user menu

**ViewSwitcher** (`ViewSwitcher.tsx`) - 51 lines
- Role-based dashboard view switching
- Consumer/Provider perspective toggle
- Dropdown menu with current view indication

#### Feature Components

**Chat System** (`/components/chat/`)
- **ChatLauncher** (`ChatLauncher.tsx`) - 34 lines
  - Fixed-position chat button
  - Dynamic component loading with SSR disabled
  - Responsive sizing for mobile/desktop

- **ChatWindow** (`ChatWindow.tsx`) - 170 lines
  - AI-powered chat interface
  - OpenAI GPT-4o-mini integration
  - Real-time message streaming

**AI Elements** (`/components/ai-elements/`)
- **Conversation** (`conversation.tsx`) - 62 lines
- **Response** (`response.tsx`)
- **Message** (`message.tsx`)

**Payment Integration** (`/components/payments/`)
- **StripePayment** (`StripePayment.tsx`) - 73 lines
  - Stripe Elements integration
  - Payment method collection
  - Error handling and validation

**Map Integration** (`/components/map/`)
- **MapboxMap** (`MapboxMap.tsx`) - 67 lines
  - Interactive Mapbox GL JS map
  - Detroit-centered coordinates
  - Service area visualization

**Real-time Features** (`/components/realtime/`)
- **MessageSubscriber** (`MessageSubscriber.tsx`) - 31 lines
  - Supabase real-time subscriptions
  - Toast notification integration

## Implementation Patterns

### Component Structure
```tsx
// Standard component pattern
export function ComponentName({ prop1, prop2, ...props }: ComponentProps) {
  // Component logic
  return (
    <div className={cn("base-styles", className)} {...props}>
      {/* Component content */}
    </div>
  )
}
```

### Styling Patterns
```tsx
// CVA variant system
const componentVariants = cva("base-classes", {
  variants: {
    variant: {
      default: "variant-classes",
      secondary: "secondary-classes"
    },
    size: {
      sm: "small-classes",
      lg: "large-classes"
    }
  },
  defaultVariants: {
    variant: "default",
    size: "sm"
  }
})
```

### Client/Server Component Usage
```tsx
// Server Component (default)
export default async function ServerComponent() {
  const data = await fetchData()
  return <ComponentTree data={data} />
}

// Client Component
'use client'
export function ClientComponent() {
  const [state, setState] = useState()
  return <InteractiveElement />
}
```

## Integration with Business Logic

### Detroit-Specific Validation
- Address validation for Detroit Metro area
- Coordinate bounds checking (42.0-42.6 lat, -83.5--82.8 lng)
- Service area radius enforcement (25 miles)

### Role-Based Rendering
- Consumer/Provider/Admin role differentiation
- Conditional component rendering based on user permissions
- Dynamic navigation and menu items

### Payment Integration
- Stripe payment processing patterns
- Secure checkout flow integration
- Variable pricing calculation and display

## Accessibility Features

### WCAG 2.1 Compliance
- Keyboard navigation support
- Screen reader compatibility
- Focus management and indicators
- Color contrast compliance
- ARIA labels and descriptions

### Form Accessibility
- Proper label association
- Validation error announcements
- Required field indicators
- Input format guidance

## Performance Optimizations

### Code Splitting
- Dynamic imports for chat components
- Lazy loading of heavy components
- Route-based code splitting

### Bundle Optimization
- Tree-shaking for unused components
- SVG icon optimization
- CSS-in-JS optimization with Tailwind

## Dependencies

### Core Dependencies
```json
{
  "@radix-ui/react-*": "Various primitives for accessibility",
  "class-variance-authority": "^0.7.1",
  "clsx": "^2.0.0",
  "tailwind-merge": "^2.0.0",
  "lucide-react": "^0.263.1",
  "react-hook-form": "^7.45.4",
  "sonner": "^1.0.0"
}
```

### Type Safety
- Full TypeScript coverage with strict mode
- Component prop type definitions
- Variant prop type inference from CVA
- Database type integration for data components

## Component Documentation Standards

### JSDoc Comments
```tsx
/**
 * Button component with multiple variants and sizes
 * @param variant - Visual style variant
 * @param size - Component size
 * @param asChild - Render as child element
 */
```

### Prop Interface Documentation
```tsx
interface ButtonProps {
  /** Visual appearance variant */
  variant?: 'default' | 'destructive' | 'outline' | 'secondary' | 'ghost' | 'link'
  /** Button size */
  size?: 'default' | 'sm' | 'lg' | 'icon'
  /** Render as child component */
  asChild?: boolean
}
```

## Testing Patterns

### Component Testing
- Unit tests for individual components
- Integration tests for component combinations
- Visual regression testing for UI consistency
- Accessibility testing with automated tools

### Story-Driven Development
- Storybook integration for component documentation
- Interactive component playground
- Visual testing and documentation

## Future Enhancements

### Planned Components
- DataTable for listing services and bookings
- DatePicker for scheduling drone services
- FileUpload for provider documentation
- StatusIndicator for real-time tracking
- NotificationCenter for system messages

### Design System Evolution
- Dark mode theme variants
- Mobile-first responsive breakpoints
- Animation and micro-interaction patterns
- Custom icon library development

## Migration Notes

### shadcn/ui Updates
- Regular updates to latest component versions
- Breaking change migration guides
- Custom component conflict resolution

### Tailwind CSS 4 Integration
- New utility classes and features
- Performance optimizations
- Design token standardization