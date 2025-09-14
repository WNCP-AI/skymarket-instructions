# Step 7: Project Initialization

> **Objective**: Initialize Next.js 15 project using Context7 documentation and project specifications to set up TypeScript, project structure, and all required dependencies.

## Learning Outcomes

- Master Context7 integration for accessing Next.js 15 official documentation
- Apply project specifications to guide framework setup decisions
- Set up TypeScript configuration aligned with project requirements
- Configure package.json with production-ready dependencies
- Establish project structure following SkyMarket specifications

## Context Setup

### Load Project Specifications

Load these specifications into Cursor chat (`Cmd+L`):

```
@AGENTS.md
@docs/specs/PLANNING-GUIDE.md
@docs/specs/components/COMPONENTS.md
```

### Access Next.js 15 Documentation via Context7

Context7 will automatically access official Next.js documentation when you use "use context7" in your implementation prompts. No commands needed - just natural language requests.

## Instructions

### Phase 1: Project Initialization

#### 1. Create Next.js 15 Project

Use this prompt with loaded specifications:

```
@AGENTS.md
@docs/specs/PLANNING-GUIDE.md

Following our tech stack specifications, create a new Next.js 15 project with current best practices:

1. TypeScript 5 configuration (strict mode) per AGENTS.md requirements
2. App Router architecture (no Pages Router) following current Next.js patterns
3. Tailwind CSS 4 integration with proper configuration
4. All required dependencies for SkyMarket marketplace development

use context7
4. ESLint configuration for code quality

Follow these requirements from AGENTS.md:
- Use absolute imports via @/* alias
- Server Components by default
- Strict TypeScript mode
- Next.js 15.5.2 with App Router

Initialize the project using the create-next-app command with appropriate flags.
```

#### 2. Configure TypeScript

After project creation, use this prompt:

```
Configure TypeScript following AGENTS.md specifications:

1. Set up strict mode with explicit types
2. Configure absolute imports with @/* path mapping
3. Add type safety rules that avoid 'any' type
4. Include React 19 and Next.js 15 type definitions

Reference the TypeScript configuration from Context7 Next.js docs.
```

#### 3. Set Up Project Structure

Using the specifications, create the directory structure:

```
Based on AGENTS.md directory structure, create the complete folder structure for SkyMarket:

app/                     # Next.js 15 App Router
├── (auth)/             # Authentication pages
├── api/                # API routes
├── booking/            # Booking flow pages
├── browse/             # Service discovery
├── dashboard/          # User dashboards
└── globals.css         # Global styles

components/             # React components
├── ui/                 # shadcn/ui base components
├── layout/             # Header, footer, navigation
├── providers/          # Provider-specific components
├── consumers/          # Consumer-specific components
└── shared/             # Shared/common components

lib/                    # Core utilities and integrations
├── supabase/           # Database clients
├── stripe/             # Payment processing
├── utils.ts            # Utility functions
├── resend.ts           # Email service

types/                  # TypeScript definitions
└── database.ts         # Supabase generated types

Create README.md files in each directory explaining their purpose.
```

### Phase 2: Dependencies Configuration

#### 4. Install Core Dependencies

Install dependencies based on AGENTS.md tech stack:

```
Based on the tech stack in AGENTS.md, install all required dependencies for SkyMarket:

Core Framework:
- Next.js 15.5.2
- React 19
- TypeScript 5

Styling & UI:
- Tailwind CSS 4
- shadcn/ui components (refer to COMPONENTS.md for the 15 core components)
- Class Variance Authority for variant management
- Lucide React for icons

Database & Auth:
- Supabase client libraries
- Supabase auth helpers for Next.js

Payment Processing:
- Stripe and Stripe Elements

Maps & Location:
- Mapbox GL JS

Email:
- Resend

AI Integration:
- OpenAI SDK (@ai-sdk/openai and ai packages)

Forms & Validation:
- React Hook Form
- Zod for schema validation

Additional Tools:
- ESLint and TypeScript
- clsx and tailwind-merge for className utility

Use npm install commands and configure package.json scripts for:
- npm run dev (Next.js development)
- npm run build (production build)
- npm run lint (ESLint)
- npm run start (production server)
```

#### 5. Configure Environment Variables

Set up environment structure:

```
Based on AGENTS.md environment setup, create:

1. .env.example with all required environment variables:
   - Supabase configuration
   - Stripe keys
   - Email service keys
   - Mapbox tokens
   - Detroit-specific app configuration

2. .env.local template with placeholder values
3. Add environment variable validation

Follow the exact environment variable names from AGENTS.md.
```

### Phase 3: Basic Configuration

#### 6. Configure Tailwind CSS

Set up styling system:

```
Using Context7 Tailwind CSS documentation and COMPONENTS.md specifications:

1. Configure Tailwind CSS 4 with Next.js 15
2. Set up CSS custom properties for theming
3. Configure responsive breakpoints
4. Add typography and color system foundation
5. Set up shadcn/ui compatibility

Reference the styling patterns from COMPONENTS.md for:
- Base component styles
- Variant management with CVA
- Design token structure
```

#### 7. Initial Layout Setup

Create basic application structure:

```
Create the basic application layout following Next.js 15 App Router patterns:

1. app/layout.tsx - Root layout with:
   - HTML structure
   - Tailwind CSS imports
   - Font configuration (Inter as specified in style guide)
   - Metadata setup

2. app/page.tsx - Homepage placeholder

3. app/globals.css - Global styles with:
   - Tailwind base styles
   - CSS custom properties
   - Basic component styles

Follow Server Component patterns from AGENTS.md unless interactivity is needed.
```

## Expected Implementation

### Project Structure
```
skymarket/
├── app/
│   ├── layout.tsx          # Root layout with fonts and metadata
│   ├── page.tsx           # Homepage
│   ├── globals.css        # Global Tailwind styles
│   ├── (auth)/           # Auth pages directory
│   ├── api/              # API routes directory
│   ├── booking/          # Booking flow directory
│   ├── browse/           # Service discovery directory
│   └── dashboard/        # User dashboards directory
├── components/
│   ├── ui/               # shadcn/ui components
│   ├── layout/           # Layout components
│   ├── providers/        # Provider components
│   ├── consumers/        # Consumer components
│   └── shared/           # Shared components
├── lib/
│   ├── supabase/        # Database clients
│   ├── stripe/          # Payment processing
│   ├── utils.ts         # Utilities
│   ├── resend.ts        # Email service
│   └── embedding.ts     # AI integration
├── types/
│   └── database.ts      # Type definitions
├── .env.example         # Environment template
├── .env.local          # Local environment
├── tsconfig.json       # TypeScript config
├── tailwind.config.ts  # Tailwind config
├── package.json        # Dependencies
└── README.md           # Project documentation
```

### TypeScript Configuration (tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### Package.json Scripts
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

## Verification Checklist

### Project Setup
- [ ] Next.js 15.5.2 project created with TypeScript
- [ ] App Router configured (no Pages Router)
- [ ] All dependencies installed without errors
- [ ] TypeScript strict mode enabled
- [ ] Absolute imports (@/*) working

### Directory Structure
- [ ] All directories from AGENTS.md created
- [ ] README.md files in major directories
- [ ] Proper separation of concerns (app/, components/, lib/)

### Configuration Files
- [ ] tsconfig.json with strict mode and path mapping
- [ ] tailwind.config.ts properly configured
- [ ] .env.example with all required variables
- [ ] ESLint configuration working

### Basic Functionality
- [ ] `npm run dev` starts development server
- [ ] `npm run build` completes without errors
- [ ] `npm run lint` passes
- [ ] Homepage loads at localhost:3000
- [ ] TypeScript compilation working

### Context7 Integration
- [ ] Successfully loaded Next.js documentation
- [ ] Applied official patterns from Context7
- [ ] Configuration follows Next.js 15 best practices

## Troubleshooting

### Common Issues

**Context7 Documentation Not Loading**
- Verify MCP Context7 is properly configured in Cursor settings
- Restart Cursor IDE completely
- Test with simple prompt: "Create a Next.js component. use context7"
- Check internet connection for Context7 documentation access

**TypeScript Configuration Errors**
```bash
# Clear Next.js cache
rm -rf .next

# Reinstall dependencies
npm install

# Check TypeScript version
npx tsc --version
```

**Dependencies Installation Issues**
```bash
# Clear npm cache
npm cache clean --force

# Use exact versions from tutorial
npm install next@15.5.2 react@19 typescript@5
```

**Tailwind CSS Not Working**
```bash
# Verify Tailwind installation
npx tailwindcss --help

# Check tailwind.config.ts paths
# Ensure globals.css is imported in layout.tsx
```

**Environment Variables**
```bash
# Copy example to local
cp .env.example .env.local

# Verify environment loading
npm run dev
# Check console for environment warnings
```

## Expected Outcome

After completing this step, you should have:

1. **Working Next.js 15 Project**: Development server running on localhost:3000
2. **TypeScript Configuration**: Strict mode with absolute imports
3. **Complete Directory Structure**: All folders from AGENTS.md specifications
4. **Dependencies Installed**: All packages for the full SkyMarket stack
5. **Environment Setup**: Variables configured with Detroit-specific defaults
6. **Build System**: All npm scripts working (dev, build, lint)
7. **Context7 Integration**: Successfully used official Next.js documentation

The project should build and run without errors, providing a solid foundation for implementing the styling system and components in the next steps.

## Next Steps

In **Step 8: Styling System**, you'll:
- Install and configure shadcn/ui component library
- Set up design tokens and theme configuration
- Create the visual foundation following COMPONENTS.md specifications
- Implement the 15 core UI components with proper TypeScript types

---

**Continue to**: [Step 8: Styling System](./08-styling-system.md) →