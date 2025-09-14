# Step 8: Styling System

> **Objective**: Build Tailwind CSS 4 + shadcn/ui system using Context7 documentation and COMPONENTS.md specifications to create a comprehensive design system with all 15 core UI components.

## Learning Outcomes

- Master shadcn/ui installation and configuration with Context7 documentation
- Implement complete component library following COMPONENTS.md specifications
- Set up design tokens and theme configuration for consistent styling
- Configure Class Variance Authority (CVA) for component variants
- Establish accessibility patterns and responsive design system

## Context Setup

### Load Project Specifications

Load these specifications into Cursor chat (`Cmd+L`):

```
@AGENTS.md
@docs/specs/components/COMPONENTS.md
```

### Enhanced Context7 Integration

Context7 will automatically access Tailwind CSS and shadcn/ui documentation when you include "use context7" in your prompts. This provides current configuration patterns and component implementations.

## Instructions

### Phase 1: Design System Foundation

#### 1. Initialize shadcn/ui

Set up the shadcn/ui system:

```
@AGENTS.md
@docs/specs/components/COMPONENTS.md

Based on our component specifications, initialize comprehensive shadcn/ui design system:

1. Initialize shadcn/ui in Next.js 15 project with current best practices
2. Configure components.json with complete TypeScript and Tailwind CSS integration
3. Set up design tokens and theme configuration per COMPONENTS.md requirements
4. Configure proper import aliases and component organization
5. Implement accessibility-first patterns and responsive breakpoints

use context7
   - @/ path aliases
   - src directory structure (false, using root structure)

3. Set up the ui directory structure:
   - components/ui/ for base components
   - Proper export patterns for component library

Follow the exact configuration patterns from Context7 shadcn/ui docs.
```

#### 2. Configure Design Tokens

Set up the theme system based on COMPONENTS.md:

```
Based on COMPONENTS.md and Context7 Tailwind documentation, configure:

1. Color system with CSS custom properties:
   - Primary colors (SkyMarket branding)
   - Secondary colors (Sky Blue theme)
   - Semantic colors (success, warning, error)
   - Neutral grays for backgrounds and text

2. Typography system:
   - Font family: Inter (as specified in COMPONENTS.md)
   - Type scale with proper line heights
   - Font weights for hierarchy

3. Spacing system:
   - 8px grid system
   - Consistent spacing tokens
   - Responsive spacing utilities

4. Component styling foundations:
   - Border radius tokens
   - Shadow system
   - Focus states and accessibility

Update both tailwind.config.ts and globals.css with these tokens.
```

### Phase 2: Core UI Components Installation

#### 3. Install Base Components

Install the 15 core components specified in COMPONENTS.md:

```
Using Context7 shadcn/ui documentation, install these exact components:

1. Button - 6 variants (default, destructive, outline, secondary, ghost, link)
2. Card - Complete card system with header, title, description, content, footer
3. Dialog - Modal system with accessibility features
4. Input - Form input with validation states
5. Label - Form labels with proper association
6. Select - Dropdown with keyboard navigation
7. Textarea - Multi-line input with auto-resize
8. Form - React Hook Form integration with validation

Use these shadcn/ui commands:
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add input
npx shadcn-ui@latest add label
npx shadcn-ui@latest add select
npx shadcn-ui@latest add textarea
npx shadcn-ui@latest add form

Ensure each component follows the patterns specified in COMPONENTS.md.
```

#### 4. Install Navigation Components

Add navigation-specific components:

```
Continue with navigation and feedback components:

9. Sonner - Toast notification system
10. Avatar - User profile images with fallbacks
11. Badge - Status indicators and labels
12. Dropdown Menu - Context menus with keyboard navigation
13. Navigation Menu - Site navigation with responsive behavior
14. Sheet - Slide-out panels and mobile drawers
15. Tabs - Tabbed content organization

Install commands:
npx shadcn-ui@latest add sonner
npx shadcn-ui@latest add avatar
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add navigation-menu
npx shadcn-ui@latest add sheet
npx shadcn-ui@latest add tabs

Verify each component matches the specifications in COMPONENTS.md.
```

### Phase 3: Component System Configuration

#### 5. Set Up Class Variance Authority

Configure variant management system:

```
Using Context7 documentation and COMPONENTS.md patterns:

1. Install and configure Class Variance Authority (CVA)
2. Set up variant patterns for component customization
3. Create TypeScript interfaces for component props
4. Implement the CVA pattern shown in COMPONENTS.md:

```tsx
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

Apply this pattern across all 15 components for consistency.
```

#### 6. Configure Component Utilities

Set up the utility system:

```
Based on COMPONENTS.md implementation patterns:

1. Set up clsx and tailwind-merge utilities in lib/utils.ts
2. Create the cn() utility function for className merging
3. Configure proper TypeScript types for component props
4. Set up proper export patterns from components/ui/

Follow the exact patterns specified in COMPONENTS.md for:
- Component structure
- Prop interfaces
- Export conventions
- TypeScript integration
```

### Phase 4: Theme Integration

#### 7. Implement Design System

Create the complete theme system:

```
Using the design specifications from COMPONENTS.md:

1. Update globals.css with:
   - CSS custom properties for all color tokens
   - Typography styles and font loading
   - Base component styles
   - Focus states and accessibility styles

2. Configure Tailwind config with:
   - Custom color palette
   - Typography scale
   - Spacing system
   - Responsive breakpoints
   - Animation utilities

3. Set up proper font loading:
   - Inter font family
   - Font weights (400, 500, 600, 700)
   - Proper font display and loading

Reference the styling patterns from Context7 Tailwind documentation.
```

#### 8. Test Component System

Verify all components work correctly:

```
Create a component showcase page to test all 15 components:

1. Create app/components-test/page.tsx
2. Import and display all 15 components with:
   - Different variants and sizes
   - Interactive states (hover, focus, disabled)
   - Accessibility features working
   - Responsive behavior

3. Test each component's:
   - TypeScript props and variants
   - Styling and theming
   - Accessibility features (keyboard navigation, ARIA labels)
   - Mobile responsiveness

This page will serve as documentation and verification of the component system.
```

## Expected Implementation

### Directory Structure
```
components/
├── ui/
│   ├── button.tsx           # 6 variants, 4 sizes
│   ├── card.tsx            # Complete card system
│   ├── dialog.tsx          # Modal with accessibility
│   ├── input.tsx           # Form input with states
│   ├── label.tsx           # Form labels
│   ├── select.tsx          # Dropdown with navigation
│   ├── textarea.tsx        # Multi-line input
│   ├── form.tsx            # React Hook Form integration
│   ├── sonner.tsx          # Toast notifications
│   ├── avatar.tsx          # Profile images
│   ├── badge.tsx           # Status indicators
│   ├── dropdown-menu.tsx   # Context menus
│   ├── navigation-menu.tsx # Site navigation
│   ├── sheet.tsx           # Slide-out panels
│   └── tabs.tsx            # Tabbed content
lib/
├── utils.ts                # cn() utility and helpers
tailwind.config.ts          # Theme configuration
app/
├── globals.css             # Design system styles
└── components-test/
    └── page.tsx            # Component showcase
```

### Tailwind Configuration
```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      fontFamily: {
        sans: ["Inter", "sans-serif"],
      },
      keyframes: {
        "accordion-down": {
          from: { height: "0" },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: "0" },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
} satisfies Config

export default config
```

### Global Styles (app/globals.css)
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 4 85% 37%; /* SkyMarket Red */
    --primary-foreground: 210 40% 98%;
    --secondary: 225 83% 46%; /* Sky Blue */
    --secondary-foreground: 222.2 84% 4.9%;
    --muted: 210 40% 96%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96%;
    --accent-foreground: 222.2 84% 4.9%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

## Verification Checklist

### shadcn/ui Installation
- [ ] shadcn/ui CLI initialized successfully
- [ ] components.json configured correctly
- [ ] All 15 components installed without errors
- [ ] TypeScript integration working

### Component Library
- [ ] All components render correctly
- [ ] Variants and sizes working (Button: 6 variants, 4 sizes)
- [ ] CVA patterns implemented consistently
- [ ] TypeScript props properly typed
- [ ] Accessibility features working (ARIA, keyboard navigation)

### Design System
- [ ] Color tokens defined in CSS custom properties
- [ ] Typography system working (Inter font family)
- [ ] Spacing system consistent (8px grid)
- [ ] Dark mode support prepared
- [ ] Focus states and accessibility styling

### Integration Testing
- [ ] Component showcase page renders all 15 components
- [ ] Interactive states working (hover, focus, disabled)
- [ ] Mobile responsiveness working
- [ ] No console errors or warnings
- [ ] Build process completes successfully

### Context7 Verification
- [ ] Used official shadcn/ui documentation
- [ ] Followed Tailwind CSS best practices
- [ ] Applied Radix UI accessibility patterns

## Troubleshooting

### Common Issues

**shadcn/ui Installation Fails**
```bash
# Ensure you're in the project root
pwd

# Update shadcn/ui CLI
npm install -g shadcn-ui@latest

# Reinitialize if needed
npx shadcn-ui@latest init
```

**Component Import Errors**
```bash
# Check tsconfig.json path mapping
cat tsconfig.json | grep -A 5 "paths"

# Verify components directory structure
ls -la components/ui/

# Check component exports
head -5 components/ui/button.tsx
```

**Styling Not Applied**
```bash
# Verify Tailwind CSS is working
npm run dev
# Check if styles appear in browser

# Check globals.css import in layout.tsx
grep "globals.css" app/layout.tsx

# Verify Tailwind config
npx tailwindcss -i app/globals.css -o test.css
```

**TypeScript Errors**
```bash
# Check TypeScript compilation
npx tsc --noEmit

# Verify component prop types
# Check that all components export proper interfaces

# Clear Next.js cache
rm -rf .next
```

**Context7 Documentation Issues**
- If shadcn/ui patterns aren't loading, verify MCP Context7 is running in Cursor
- Test with: "Install shadcn/ui button component with variants. use context7"
- Restart Cursor IDE if Context7 responses seem outdated
- Check internet connection for accessing official component documentation

## Expected Outcome

After completing this step, you should have:

1. **Complete Component Library**: All 15 shadcn/ui components installed and working
2. **Design System**: Comprehensive theme with colors, typography, and spacing
3. **CVA Integration**: Variant management system for component customization
4. **Accessibility Features**: WCAG 2.1 compliance with keyboard navigation and ARIA
5. **TypeScript Integration**: Fully typed components with proper interfaces
6. **Component Showcase**: Testing page demonstrating all components and variants
7. **Production-Ready Styling**: Optimized CSS with Tailwind CSS 4

The styling system should be comprehensive and ready for building complex UI layouts in the next step.

## Next Steps

In **Step 9: Landing Pages**, you'll:
- Create homepage and supporting pages using the component library
- Implement navigation and layout components
- Set up basic routing structure with Next.js 15 App Router
- Apply SkyMarket branding and Detroit-specific content

---

**Continue to**: [Step 9: Landing Pages](./09-landing-pages.md) →