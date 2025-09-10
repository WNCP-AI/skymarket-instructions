# SkyMarket Setup Instructions

This guide will walk you through setting up the SkyMarket development environment.

## Prerequisites

Before starting, ensure you have the following installed:

- **Node.js** (v18.17.0 or higher)
- **npm** (v9.0.0 or higher)
- **Git** (v2.30.0 or higher)
- A **Supabase** account
- A **Stripe** account (for payments)
- A **Mapbox** account (for maps)

## Environment Setup

### 1. Clone the Repository

```bash
git clone <repository-url>
cd skymarket-subpabase
```

### 2. Install Dependencies

```bash
npm install
```

This will install all required packages including:
- Next.js 14
- TypeScript
- Tailwind CSS 3
- shadcn/ui components
- Supabase client libraries
- Stripe SDK
- Mapbox GL JS

### 3. Environment Configuration

Copy the example environment file:

```bash
cp .env.example .env.local
```

## Service Configuration

### Supabase Setup

1. **Create a Supabase Project**
   - Go to [supabase.com](https://supabase.com)
   - Click "Start your project"
   - Create a new organization (if needed)
   - Create a new project
   - Choose a database password (save it securely)
   - Select the region closest to Detroit (US East)

2. **Get Supabase Credentials**
   - Go to Project Settings → API
   - Copy your project URL and anon key
   - Update `.env.local`:
   ```bash
   NEXT_PUBLIC_SUPABASE_URL=https://your-project-id.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
   SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   ```

3. **Set up Database Schema**
   - Go to the SQL Editor in your Supabase dashboard
   - Copy the contents of `supabase/schema.sql`
   - Paste and run the SQL script
   - This creates all necessary tables and policies

4. **Configure Authentication**
   - Go to Authentication → Settings
   - Enable email confirmations (optional for development)
   - Configure redirect URLs for localhost:3000

### Stripe Setup

1. **Create a Stripe Account**
   - Go to [stripe.com](https://stripe.com)
   - Create an account
   - Complete the registration process

2. **Get API Keys**
   - Go to Developers → API Keys
   - Copy your publishable key and secret key
   - Update `.env.local`:
   ```bash
   NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
   STRIPE_SECRET_KEY=sk_test_...
   ```

3. **Set up Webhooks** (for production)
   - Go to Developers → Webhooks
   - Add endpoint: `https://yourdomain.com/api/webhooks/stripe`
   - Select events: `payment_intent.succeeded`, `payment_intent.payment_failed`

### Mapbox Setup

1. **Create a Mapbox Account**
   - Go to [mapbox.com](https://mapbox.com)
   - Create an account
   - Choose the free tier

2. **Get Access Token**
   - Go to Account → Access Tokens
   - Copy your default public token
   - Update `.env.local`:
   ```bash
   NEXT_PUBLIC_MAPBOX_TOKEN=pk.eyJ1...
   ```

## Development Server

### Start the Development Server

```bash
npm run dev
```

The application will be available at:
- **Frontend**: http://localhost:3000
- **API Routes**: http://localhost:3000/api/*

### Verify Setup

1. **Homepage**: Visit http://localhost:3000
   - Should see the SkyMarket homepage
   - Navigation should work
   - Categories should display properly

2. **Database Connection**: Check browser console
   - Should not see Supabase connection errors
   - Auth state should initialize properly

3. **Styling**: Verify Tailwind CSS
   - Components should be styled correctly
   - Responsive design should work

## Development Tools

### Recommended VSCode Extensions

Install these extensions for the best development experience:

```bash
# Essential
code --install-extension bradlc.vscode-tailwindcss
code --install-extension ms-vscode.vscode-typescript-next
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode.vscode-eslint

# Helpful
code --install-extension ms-vscode.vscode-json
code --install-extension formulahendry.auto-rename-tag
code --install-extension christian-kohler.path-intellisense
```

### VSCode Settings

Create `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
    ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ],
  "typescript.preferences.importModuleSpecifier": "relative"
}
```

### Git Configuration

Set up Git hooks for code quality:

```bash
# Optional: Set up pre-commit hooks
npx husky-init && npm run prepare
npx husky add .husky/pre-commit "npm run lint"
```

## Database Management

### Supabase Local Development (Optional)

For advanced development, you can run Supabase locally:

```bash
# Install Supabase CLI
npm install -g supabase

# Initialize local Supabase
supabase init

# Start local services
supabase start

# Link to your project
supabase link --project-ref your-project-id
```

### Schema Updates

When making database changes:

1. Update `supabase/schema.sql`
2. Apply changes in Supabase dashboard
3. Or use migrations:
   ```bash
   supabase db diff --schema public > migrations/001_initial.sql
   ```

## Common Issues

### Port Already in Use

If port 3000 is occupied:

```bash
# Use a different port
npm run dev -- --port 3001
```

### Supabase Connection Issues

1. Check your environment variables
2. Verify project URL and keys
3. Check Supabase project status
4. Review RLS policies in database

### Tailwind CSS Not Loading

1. Restart the development server
2. Check `tailwind.config.ts` configuration
3. Verify `globals.css` imports

### TypeScript Errors

1. Check `tsconfig.json` settings
2. Restart TypeScript server in VSCode
3. Run type checking: `npm run type-check`

## Next Steps

Once setup is complete:

1. **Explore the Codebase**: Review the project structure
2. **Run Tests**: `npm run test` (once tests are added)
3. **Read Documentation**: Review other docs in `/docs`
4. **Start Development**: Begin with Phase 2 implementation

## Development Workflow

### Branch Strategy

```bash
# Create feature branch
git checkout -b feature/user-authentication

# Make changes and commit
git add .
git commit -m "feat: add user authentication system"

# Push and create PR
git push origin feature/user-authentication
```

### Code Quality

Before committing:

```bash
# Check linting
npm run lint

# Check types
npm run build

# Format code
npm run format
```

## Support

If you encounter issues:

1. Check this documentation
2. Review the [troubleshooting guide](./TROUBLESHOOTING.md)
3. Check existing GitHub issues
4. Contact the development team

---

**Next**: Read the [Development Workflow](./WORKFLOW.md) guide.