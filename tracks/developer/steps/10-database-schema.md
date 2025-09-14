# Step 10: Database Schema Implementation

> **Phase 3: Database & Authentication - Database Foundation**
>
> Create a complete PostgreSQL database schema with all tables, relationships, constraints, and Row Level Security (RLS) policies using Supabase and specification-driven development.

## Objective

Learn to implement a production-ready database schema by combining detailed specifications with official Supabase documentation. You'll create all tables, relationships, custom types, indexes, triggers, and security policies needed for the SkyMarket marketplace.

**What you'll build:**
- Complete PostgreSQL schema with 7 tables and relationships
- Custom enums for type safety (user_role, provider_type, service_category, booking_status, payment_status)
- Row Level Security (RLS) policies for data protection
- Database triggers and functions for automation
- TypeScript type generation from database schema
- Comprehensive indexes for performance optimization

## Context Setup

Load these specifications and project documentation into Cursor:

```
@docs/specs/architecture/DATABASE.md
@AGENTS.md
```

**Key points from DATABASE.md:**
- 7 core tables with comprehensive relationships
- 5 custom PostgreSQL enums for type safety
- Advanced RLS policies for multi-role security
- Automated triggers for timestamps and calculations
- Detroit Metro-specific business logic constraints

**Key points from AGENTS.md:**
- Supabase as primary database with TypeScript integration
- Migration-first approach using `supabase/migrations/`
- Generated types in `types/database.ts`
- Geographic constraints for Detroit Metro area

## Context7 Commands

Access official Supabase documentation for database patterns:

**Enhanced Context7 Integration**: Context7 provides access to current Supabase database patterns, migration workflows, RLS policies, and TypeScript integration. Include "use context7" in all database implementation prompts for official Supabase documentation.

This gives you access to:
- Database schema design best practices
- Migration file structure and SQL patterns
- RLS policy implementation techniques
- TypeScript type generation workflows

## Implementation Prompts

Use these prompts with the loaded context and Context7 documentation:

### 1. Initial Schema Creation

```
Using the Supabase database documentation from Context7 and our DATABASE.md specification, create the initial database migration with:

1. All 5 custom PostgreSQL enums (user_role, provider_type, service_category, booking_status, payment_status)
2. The profiles table that extends auth.users with SkyMarket-specific fields
3. The automatic profile creation trigger function from the specification
4. All required indexes from the DATABASE.md specification

Follow Supabase migration best practices and include proper up/down migration patterns.
```

### 2. Core Business Tables

```
Using the DATABASE.md specification and Context7 Supabase patterns, create a migration for the core business tables:

1. providers table with all fields, constraints, and relationships
2. listings table with pricing logic and full-text search index
3. bookings table with complete status workflow and geographic constraints
4. All foreign key relationships and constraints from the specification

Include the update_updated_at_column function and apply triggers to all tables.
```

### 3. Supporting Tables and Advanced Features

```
Create the final migration for supporting tables and advanced features:

1. reviews table with rating constraints and business rules
2. messages table for in-app communication
3. service_areas table with Detroit Metro zone data
4. All composite indexes for performance optimization from DATABASE.md
5. The provider rating calculation trigger function

Include comprehensive RLS policies for all tables following the security patterns in the specification.
```

### 4. TypeScript Integration

```
Using the Supabase TypeScript documentation from Context7:

1. Generate TypeScript types from the database schema
2. Create utility functions for type-safe database queries
3. Set up the proper database client configurations for server and browser use
4. Create helper functions for common query patterns shown in DATABASE.md

Follow the patterns in AGENTS.md for client/server separation and type safety.
```

## Code Examples

After running the prompts, you should generate:

### Database Migration Structure
```
supabase/
├── migrations/
│   ├── 0001_initial_schema.sql        # Enums, profiles, triggers
│   ├── 0002_core_business_tables.sql  # providers, listings, bookings
│   ├── 0003_supporting_tables.sql     # reviews, messages, service_areas
│   └── 0004_rls_policies.sql          # All security policies
└── seed.sql                           # Detroit service areas data
```

### Type Generation
```typescript
// types/database.ts (generated)
export type Database = {
  public: {
    Tables: {
      profiles: {
        Row: {
          id: string
          email: string
          full_name: string | null
          role: "consumer" | "provider" | "admin"
          // ... other fields
        }
        // ... Insert, Update types
      }
      // ... other tables
    }
    Enums: {
      user_role: "consumer" | "provider" | "admin"
      provider_type: "courier" | "drone"
      service_category: "food_delivery" | "courier" | "aerial_imaging" | "site_mapping"
      booking_status: "pending" | "accepted" | "in_progress" | "completed" | "cancelled"
      payment_status: "pending" | "paid" | "failed" | "refunded"
    }
  }
}
```

### Client Configuration
```typescript
// lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'
import type { Database } from '@/types/database'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (cookies) => cookies.forEach(({ name, value, options }) =>
          cookieStore.set(name, value, options)
        ),
      },
    }
  )
}
```

## Verification Steps

### 1. Database Structure Validation
```bash
# Apply all migrations
npx supabase db reset

# Verify table creation
npx supabase db ls

# Check RLS policies
npx supabase db show --schema=auth,public
```

### 2. Type Generation Test
```bash
# Generate types
npx supabase gen types typescript --local > types/database.ts

# Verify types compile
npx tsc --noEmit
```

### 3. Query Testing
```typescript
// Test database queries with types
import { createClient } from '@/lib/supabase/server'

const supabase = await createClient()

// Test basic queries work with generated types
const { data: profiles } = await supabase
  .from('profiles')
  .select('*')
  .eq('role', 'provider') // TypeScript should validate this

const { data: listings } = await supabase
  .from('listings')
  .select(`
    *,
    providers (
      *,
      profiles (
        full_name,
        avatar_url
      )
    )
  `)
  .eq('active', true)
```

### 4. RLS Policy Verification
```sql
-- Test RLS policies work correctly
SET ROLE authenticated;
SET request.jwt.claims TO '{"sub": "user-uuid-here"}';

-- Should only return user's own profile
SELECT * FROM profiles WHERE id = 'user-uuid-here';

-- Should return all active listings
SELECT * FROM listings WHERE active = true;
```

## Expected Outcome

After completing this step, you'll have:

✅ **Complete Database Schema**: All 7 tables with proper relationships and constraints
✅ **Type Safety**: Generated TypeScript types for all database operations
✅ **Security Layer**: Comprehensive RLS policies protecting user data
✅ **Performance Optimization**: Strategic indexes for common query patterns
✅ **Business Logic**: Triggers and functions automating marketplace operations
✅ **Migration System**: Version-controlled schema changes for team development

**Key achievements:**
- Production-ready PostgreSQL database with 7 interconnected tables
- Multi-role security system with RLS policies
- Full TypeScript integration with generated types
- Automated business logic through database triggers
- Detroit Metro-specific geographic constraints
- Search-optimized listing discovery

## Troubleshooting

### Common Issues

**Migration Errors:**
```bash
# Reset database and reapply migrations
npx supabase db reset

# Check migration syntax
npx supabase db lint
```

**Type Generation Issues:**
```bash
# Ensure database is running
npx supabase status

# Regenerate types
npx supabase gen types typescript --local > types/database.ts
```

**RLS Policy Problems:**
- Check that policies allow the intended operations
- Verify auth.uid() returns expected values
- Test policies with different user roles

**Foreign Key Violations:**
- Ensure referenced tables exist before creating relationships
- Check that enum values match exactly
- Verify UUID generation is working correctly

### Validation Queries

```sql
-- Check all tables exist
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public';

-- Verify enums are created
SELECT typname FROM pg_type WHERE typtype = 'e';

-- Check indexes
SELECT indexname, tablename FROM pg_indexes
WHERE schemaname = 'public';

-- Verify RLS is enabled
SELECT tablename, rowsecurity FROM pg_tables
WHERE schemaname = 'public';
```

## Next Steps

In **Step 11: Authentication System**, you'll:
- Build complete authentication flows using the profiles table
- Implement role-based access control with the user roles
- Create authentication middleware for route protection
- Set up email verification and password reset flows
- Build provider onboarding with role switching

This database foundation enables all authentication and user management features in the next step.

---

**Previous**: [Step 9: Landing Pages](./09-landing-pages.md) ← | → **Next**: [Step 11: Authentication System](./11-authentication.md)