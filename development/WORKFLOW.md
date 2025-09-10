# SkyMarket Development Workflow

This document outlines the development workflow, git strategy, and collaboration processes for the SkyMarket project.

## Git Workflow

### Branch Strategy

We use a modified GitFlow strategy optimized for rapid development:

```bash
main                    # Production-ready code
├── develop            # Integration branch for features
├── feature/*          # Feature development
├── hotfix/*           # Critical production fixes
└── release/*          # Release preparation
```

#### Branch Naming Convention

```bash
# Feature branches
feature/user-authentication
feature/provider-onboarding
feature/payment-integration
feature/map-integration

# Bug fixes
fix/booking-validation-error
fix/profile-update-bug
fix/stripe-webhook-handling

# Hotfixes (production)
hotfix/critical-payment-issue
hotfix/security-vulnerability

# Release branches
release/v1.0.0
release/v1.1.0
```

### Development Process

#### 1. Starting New Work

```bash
# Update develop branch
git checkout develop
git pull origin develop

# Create feature branch
git checkout -b feature/user-authentication

# Work on feature
# ... make changes ...

# Commit changes with conventional commits
git add .
git commit -m "feat: add user authentication system"
```

#### 2. Conventional Commits

We use [Conventional Commits](https://www.conventionalcommits.org/) for consistent commit messages:

```bash
# Format: type(scope): description
# Types: feat, fix, docs, style, refactor, test, chore

feat: add user authentication system
fix: resolve booking validation error  
docs: update API documentation
style: format code with prettier
refactor: improve component structure
test: add unit tests for booking flow
chore: update dependencies

# Examples with scope
feat(auth): implement social login
fix(payments): handle stripe webhook failures
docs(api): document booking endpoints
```

#### 3. Pull Request Process

```bash
# Push feature branch
git push origin feature/user-authentication

# Create Pull Request with template
# PR Title: feat: Add user authentication system
```

**PR Template:**
```markdown
## Description
Brief description of changes made

## Type of Change
- [ ] Bug fix
- [ ] New feature  
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
```

#### 4. Code Review Process

**Review Requirements:**
- All PRs require at least 1 approval
- Automated checks must pass (lint, type-check, build)
- No merge conflicts with target branch

**Review Checklist:**
- [ ] Code follows project conventions
- [ ] Logic is clear and well-commented
- [ ] Error handling is appropriate
- [ ] Security considerations addressed
- [ ] Performance implications considered
- [ ] Tests are adequate

#### 5. Merging Strategy

```bash
# Squash and merge for feature branches
# This keeps develop history clean

# Regular merge for release branches
# This preserves release history
```

## Development Standards

### Code Style

#### TypeScript/JavaScript
```typescript
// Use TypeScript strict mode
interface User {
  id: string
  email: string
  role: 'consumer' | 'provider' | 'admin'
}

// Prefer const for immutable values
const API_BASE_URL = 'https://api.skymarket.com'

// Use descriptive function names
async function createBookingWithValidation(
  bookingData: CreateBookingRequest
): Promise<Booking> {
  // Implementation
}

// Use early returns to reduce nesting
function validateBookingData(data: BookingData): ValidationResult {
  if (!data.listingId) {
    return { valid: false, error: 'Listing ID required' }
  }
  
  if (!data.scheduledAt) {
    return { valid: false, error: 'Schedule time required' }
  }
  
  return { valid: true }
}
```

#### React Components
```tsx
// Use function components with TypeScript
interface BookingCardProps {
  booking: Booking
  onCancel: (bookingId: string) => void
  className?: string
}

export function BookingCard({ booking, onCancel, className }: BookingCardProps) {
  // Use custom hooks for logic
  const { isLoading, cancel } = useBookingActions(booking.id)
  
  return (
    <Card className={className}>
      <CardHeader>
        <CardTitle>{booking.service.title}</CardTitle>
      </CardHeader>
      <CardContent>
        <Button 
          onClick={() => onCancel(booking.id)} 
          disabled={isLoading}
        >
          Cancel Booking
        </Button>
      </CardContent>
    </Card>
  )
}
```

#### CSS/Tailwind
```tsx
// Use Tailwind utilities
<div className="flex flex-col gap-4 p-6">
  <h1 className="text-2xl font-bold text-gray-900">
    Booking Details
  </h1>
  <p className="text-gray-600">
    Review your booking information below.
  </p>
</div>

// For complex styles, use CSS modules or styled-components
<div className="booking-card booking-card--active">
```

### File Organization

```
components/
├── ui/                 # shadcn/ui components (don't modify)
├── layout/             # Layout components
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── Sidebar.tsx
├── forms/              # Complex form components
│   ├── AuthForms.tsx
│   └── BookingWizard.tsx
├── providers/          # Provider-specific features
│   └── ListingCard.tsx
├── consumers/          # Consumer-specific features
│   └── SearchFilters.tsx
└── shared/             # Shared across user types
    ├── MapView.tsx
    └── RatingStars.tsx

lib/
├── supabase/          # Database utilities
│   ├── client.ts
│   ├── server.ts
│   └── queries.ts
├── stripe/            # Payment utilities
│   └── client.ts
├── utils/             # General utilities
│   ├── format.ts
│   └── validation.ts
└── constants/         # App constants
    └── config.ts

hooks/                 # Custom React hooks
├── useAuth.ts
├── useBookings.ts
└── useSupabase.ts

types/                 # TypeScript definitions
├── database.ts
├── api.ts
└── common.ts
```

### Testing Strategy

#### Unit Testing
```typescript
// utils/format.test.ts
import { formatPrice, formatDate } from './format'

describe('formatPrice', () => {
  test('formats USD currency correctly', () => {
    expect(formatPrice(1234)).toBe('$12.34')
  })
  
  test('handles zero values', () => {
    expect(formatPrice(0)).toBe('$0.00')
  })
})
```

#### Component Testing
```tsx
// components/BookingCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react'
import { BookingCard } from './BookingCard'

const mockBooking = {
  id: '123',
  service: { title: 'Food Delivery' },
  status: 'pending'
}

test('renders booking information', () => {
  render(
    <BookingCard 
      booking={mockBooking} 
      onCancel={jest.fn()} 
    />
  )
  
  expect(screen.getByText('Food Delivery')).toBeInTheDocument()
})

test('calls onCancel when cancel button is clicked', () => {
  const mockOnCancel = jest.fn()
  
  render(
    <BookingCard 
      booking={mockBooking} 
      onCancel={mockOnCancel} 
    />
  )
  
  fireEvent.click(screen.getByText('Cancel Booking'))
  expect(mockOnCancel).toHaveBeenCalledWith('123')
})
```

#### Integration Testing
```typescript
// __tests__/auth.integration.test.ts
import { createClient } from '@/lib/supabase/client'
import { signUp, signIn } from '@/lib/auth'

describe('Authentication Integration', () => {
  test('user can sign up and sign in', async () => {
    const email = 'test@example.com'
    const password = 'testpassword123'
    
    // Sign up
    const signUpResult = await signUp(email, password)
    expect(signUpResult.user).toBeTruthy()
    
    // Sign in
    const signInResult = await signIn(email, password)
    expect(signInResult.user?.email).toBe(email)
  })
})
```

## Phase Development Workflow

### Phase Planning

Each development phase follows this structure:

1. **Phase Planning Meeting**
   - Review phase requirements
   - Identify dependencies
   - Estimate effort
   - Assign tasks

2. **Phase Kickoff**
   - Create phase branch: `phase/02-authentication`
   - Set up project board
   - Define acceptance criteria

3. **Daily Development**
   - Daily standup (async)
   - Feature branch development
   - Regular integration with phase branch

4. **Phase Review**
   - Demo completed features
   - QA testing
   - Documentation updates
   - Merge to develop

### Phase Documentation

Each phase maintains documentation in `/docs/development/`:

```markdown
# Phase 2: User Authentication

## Objectives
- [ ] User registration and login
- [ ] Profile management
- [ ] Role-based access control

## Technical Tasks
- [ ] Set up Supabase Auth
- [ ] Create auth forms
- [ ] Implement middleware
- [ ] Add profile components

## Acceptance Criteria
- Users can register with email/password
- Users can log in and log out
- Users can update their profiles
- Provider role grants access to provider features

## Testing
- [ ] Unit tests for auth functions
- [ ] Integration tests for auth flow
- [ ] Manual testing on all devices
```

## Quality Assurance

### Automated Checks

```bash
# Run before committing
npm run lint          # ESLint checks
npm run type-check    # TypeScript validation
npm run test          # Unit tests
npm run build         # Production build test
```

### Pre-commit Hooks

```bash
# .husky/pre-commit
#!/bin/sh
npm run lint-staged
npm run type-check
```

### Continuous Integration

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test
      - run: npm run build
```

## Deployment Workflow

### Development Deployment
```bash
# Automatic deployment on push to develop
git push origin develop
# Deploys to: https://skymarket-dev.vercel.app
```

### Staging Deployment
```bash
# Manual deployment for testing
git checkout release/v1.0.0
git push origin release/v1.0.0
# Deploys to: https://skymarket-staging.vercel.app
```

### Production Deployment
```bash
# Production deployment from main
git checkout main
git merge release/v1.0.0
git tag v1.0.0
git push origin main --tags
# Deploys to: https://skymarket.com
```

## Issue Management

### Issue Templates

**Bug Report:**
```markdown
## Bug Description
Clear description of the bug

## Steps to Reproduce
1. Go to '...'
2. Click on '...'
3. See error

## Expected Behavior
What should happen

## Environment
- OS: [e.g. macOS]
- Browser: [e.g. Chrome]
- Version: [e.g. 1.0.0]
```

**Feature Request:**
```markdown
## Feature Description
Clear description of the feature

## Business Value
Why is this feature needed?

## Acceptance Criteria
- [ ] Criteria 1
- [ ] Criteria 2

## Technical Considerations
Any technical notes or requirements
```

### Issue Labels

```bash
# Type
bug          # Something isn't working
enhancement  # New feature request
documentation # Documentation needs

# Priority
critical     # Blocking release
high        # Important for release
medium      # Nice to have
low         # Future consideration

# Status
ready       # Ready for development
in-progress # Currently being worked on
review      # In code review
testing     # In QA testing
blocked     # Waiting on external dependency

# Component
frontend    # UI/UX related
backend     # Server/database related
auth        # Authentication related
payments    # Payment processing
maps        # Map integration
```

## Communication

### Daily Standup (Async)
**Format:** Slack/Discord updates
```markdown
**Yesterday:**
- Completed user authentication forms
- Fixed profile update bug

**Today:**
- Working on provider onboarding flow
- Review PR for booking system

**Blockers:**
- Waiting for Stripe webhook configuration
```

### Weekly Planning
**Format:** Video call or detailed async planning
- Review completed work
- Plan upcoming week
- Address blockers
- Update project timeline

### Documentation Updates

All changes must include documentation updates:

1. **Code Comments**: Complex logic must be commented
2. **API Documentation**: Update endpoint docs for API changes
3. **README Updates**: Update setup instructions if needed
4. **Architecture Documentation**: Update for significant changes

## Tools and Environment

### Required Tools
- **Node.js 18+**: Runtime environment
- **Git**: Version control
- **VSCode**: Recommended editor
- **Docker** (optional): For local database

### Recommended Extensions
- **ESLint**: Code linting
- **Prettier**: Code formatting
- **Tailwind CSS IntelliSense**: CSS class completion
- **TypeScript Importer**: Auto imports
- **GitLens**: Git visualization

### Environment Configuration
```bash
# Development
npm run dev          # Start development server
npm run db:reset     # Reset local database
npm run db:seed      # Seed test data

# Testing
npm run test         # Run unit tests
npm run test:e2e     # Run E2E tests
npm run test:watch   # Watch mode

# Production
npm run build        # Production build
npm run start        # Start production server
npm run analyze      # Bundle analysis
```

---

This workflow ensures consistent, high-quality development while maintaining rapid iteration speed for the SkyMarket platform.