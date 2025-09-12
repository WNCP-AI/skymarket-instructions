# Step 10: Supabase Overview - Understanding Supabase Services

**Objective**: Understand what Supabase provides, create a project, and obtain the necessary URL and API keys.

## What is Supabase?

Supabase is an open-source alternative to Firebase that provides a complete backend-as-a-service (BaaS) solution built on PostgreSQL.

### Core Services Supabase Provides

**1. Database (PostgreSQL)**
- Full-featured relational database
- Real-time subscriptions
- Row Level Security (RLS)
- Advanced SQL capabilities

**2. Authentication**
- Email/password authentication
- Social login (Google, GitHub, etc.)
- Magic links and OTP
- JWT token management

**3. Real-time**
- WebSocket connections
- Live data synchronization
- Presence tracking
- Broadcasting

**4. Storage**
- File upload and management
- Image transformations
- CDN delivery
- Access control

**5. Edge Functions**
- Server-side functions
- Custom API endpoints
- Webhook handlers
- Background jobs

## Why Supabase for SkyMarket?

### Perfect for Marketplace Applications

**Real-time Features**
- Live booking status updates
- Instant messaging between users
- Real-time location tracking
- Live notifications

**Scalable Database**
- PostgreSQL handles complex relationships
- ACID transactions for payments
- Advanced querying capabilities
- Horizontal scaling options

**Built-in Security**
- Row Level Security for data isolation
- JWT authentication
- API key management
- Audit logging

**Developer Experience**
- Auto-generated APIs
- TypeScript types
- Dashboard for data management
- Migration system

## Creating Your Supabase Project

### Step 1: Access Supabase Dashboard

1. **Visit**: [supabase.com](https://supabase.com)
2. **Sign In**: Use your existing account
3. **Navigate to**: Dashboard

### Step 2: Create New Project

1. **Click**: "New project"
2. **Select Organization**: Choose your personal org or create team
3. **Project Configuration**:
   - **Name**: `skymarket-hackathon`
   - **Database Password**: Create strong password (save it!)
   - **Region**: Choose closest to Detroit (US East usually optimal)
   - **Pricing Plan**: Free tier is sufficient for hackathon

4. **Click**: "Create new project"

### Step 3: Wait for Project Setup

**Setup Time**: 1-3 minutes
**What's Happening**:
- PostgreSQL database provisioning
- API services initialization
- Authentication setup
- Dashboard configuration

## Understanding Your Project Dashboard

### Navigation Overview

```
Dashboard Sections:
├── Home              # Project overview and quick stats
├── Table Editor      # Database management interface
├── SQL Editor        # Run custom SQL queries
├── Authentication    # User management and auth settings
├── Storage           # File upload and management
├── Edge Functions    # Server-side function deployment
├── Database          # Schema and migration management
├── API Docs          # Auto-generated API documentation
└── Settings          # Project configuration and keys
```

### Key Dashboard Areas

**Home Tab**
- Project health metrics
- Recent activity
- Quick access to common tasks

**Settings Tab** (Most Important)
- API keys and URLs
- Database connection strings
- Project configuration
- Billing and usage

## Getting Your Project Keys

### Step 1: Navigate to Settings

1. **Click**: Settings (in left sidebar)
2. **Select**: API tab

### Step 2: Copy Essential Keys

**You'll need these values:**

```env
# Project URL
NEXT_PUBLIC_SUPABASE_URL=https://[project-id].supabase.co

# Anonymous (Public) Key
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Service Role (Private) Key
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Key Types Explained**:

**Anon Key (Public)**
- Safe to use in client-side code
- Respects Row Level Security policies
- Used for user-facing operations

**Service Role Key (Private)**
- Full database access
- Bypasses Row Level Security
- Server-side operations only
- Never expose in client code

### Step 3: Update Environment Variables

```bash
# Open your .env.local file
cursor .env.local
```

```env
# Replace with your actual values
NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_here
```

## Understanding Supabase Concepts

### Database Concepts

**Tables**
- Store your application data
- Equivalent to database tables
- Support relationships and constraints

**Row Level Security (RLS)**
- Control who can access which rows
- Policy-based access control
- User-specific data isolation

**Real-time Subscriptions**
- Listen to database changes
- Automatic UI updates
- WebSocket-based communication

### Authentication Flow

**User Registration**
```
User submits email/password
    ↓
Supabase creates user record
    ↓
Email verification sent (optional)
    ↓
JWT token issued
    ↓
User authenticated in app
```

**Session Management**
- JWT tokens for authentication
- Automatic token refresh
- Secure session storage
- Multi-device support

## SkyMarket Data Architecture

### Core Tables We'll Create

**Users Table**
```sql
users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE,
  role TEXT CHECK (role IN ('consumer', 'provider')),
  profile JSONB,
  created_at TIMESTAMP DEFAULT NOW()
)
```

**Listings Table**
```sql
listings (
  id UUID PRIMARY KEY,
  provider_id UUID REFERENCES users(id),
  title TEXT NOT NULL,
  description TEXT,
  price DECIMAL,
  category TEXT,
  location JSONB,
  created_at TIMESTAMP DEFAULT NOW()
)
```

**Bookings Table**
```sql
bookings (
  id UUID PRIMARY KEY,
  consumer_id UUID REFERENCES users(id),
  listing_id UUID REFERENCES listings(id),
  status TEXT DEFAULT 'pending',
  scheduled_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
)
```

### Relationships Overview

```
Users (1) ──→ (Many) Listings
Users (1) ──→ (Many) Bookings
Listings (1) ──→ (Many) Bookings
```

## API Patterns

### Auto-Generated REST API

Supabase automatically creates REST endpoints:

```javascript
// Get all listings
GET /rest/v1/listings

// Create new booking
POST /rest/v1/bookings
{
  "consumer_id": "uuid",
  "listing_id": "uuid",
  "scheduled_at": "2024-01-15T10:00:00Z"
}

// Update booking status
PATCH /rest/v1/bookings?id=eq.uuid
{
  "status": "confirmed"
}
```

### JavaScript Client Usage

```typescript
import { createClient } from '@supabase/supabase-js'

// Initialize client
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)

// Query data
const { data: listings, error } = await supabase
  .from('listings')
  .select('*')
  .eq('category', 'drone-services')

// Real-time subscription
supabase
  .channel('bookings')
  .on('postgres_changes', 
    { event: '*', schema: 'public', table: 'bookings' },
    (payload) => console.log('Booking updated:', payload)
  )
  .subscribe()
```

## Testing Your Setup

### Verify Connection

Create a test file to verify your Supabase connection:

```typescript
// test-supabase.ts
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)

async function testConnection() {
  try {
    const { data, error } = await supabase.from('_test').select('*').limit(1)
    
    if (error) {
      console.log('Supabase connected successfully!')
      // Error is expected since _test table doesn't exist
    } else {
      console.log('Connection working:', data)
    }
  } catch (err) {
    console.error('Connection failed:', err)
  }
}

testConnection()
```

## Expected Outcome

After completing this step, you should have:

### Supabase Project Setup
- [ ] New Supabase project created
- [ ] Project dashboard accessible
- [ ] Strong database password saved securely

### API Keys Configuration
- [ ] Project URL copied
- [ ] Anonymous key obtained
- [ ] Service role key secured
- [ ] Environment variables updated

### Understanding Gained
- [ ] What Supabase provides
- [ ] Key concepts (RLS, real-time, auth)
- [ ] How it fits into SkyMarket architecture
- [ ] Auto-generated API patterns

### Ready for Next Steps
- [ ] Environment variables configured
- [ ] Dashboard navigation familiar
- [ ] Ready to create database schema
- [ ] Prepared for authentication setup

## Troubleshooting

### Project Creation Issues

**"Organization not found"**
- Verify you're signed into correct account
- Create personal organization if needed

**"Database setup failed"**
- Try different region
- Wait and retry
- Check Supabase status page

### Connection Issues

**"Invalid API key"**
- Double-check keys copied correctly
- Ensure no extra spaces or characters
- Regenerate keys if necessary

**"CORS errors"**
- Verify NEXT_PUBLIC_SUPABASE_URL is correct
- Check project is active (not paused)

## Security Best Practices

### API Key Management
- **Never commit** service role keys to git
- **Use environment variables** for all keys
- **Regenerate keys** if accidentally exposed
- **Use anon key** for client-side operations only

### Access Control Planning
- Plan Row Level Security policies
- Use least-privilege access patterns
- Implement proper user roles
- Audit database access regularly

## Next Steps

Perfect! Your Supabase project is ready. Next, we'll connect your Next.js application to Supabase and implement authentication with role-based access for consumers and providers.

---

**Previous Step**: [Step 9: Landing Pages](./09-landing-pages.md) | **Next Step**: [Step 11: Connecting Supabase](./11-connecting-supabase.md)

**Need help with Supabase?** Check the [official documentation](https://supabase.com/docs) or explore your project dashboard to get familiar with the interface.