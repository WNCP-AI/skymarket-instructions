# SkyMarket Installation Guide

Step-by-step installation process for SkyMarket development environment.

## System Requirements

### Hardware Requirements
- **CPU**: 2+ cores recommended
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 2GB free space for dependencies
- **Network**: Stable internet connection

### Software Requirements
- **Operating System**: macOS, Windows, or Linux
- **Node.js**: v18.17.0 or higher
- **npm**: v9.0.0 or higher (comes with Node.js)
- **Git**: v2.30.0 or higher

## Installation Steps

### Step 1: Install Node.js

#### macOS (using Homebrew)
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js
brew install node

# Verify installation
node --version
npm --version
```

#### macOS/Linux (using Node Version Manager)
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Reload terminal or run:
source ~/.bashrc

# Install and use Node.js
nvm install 18
nvm use 18

# Verify installation
node --version
npm --version
```

#### Windows
1. Download Node.js from [nodejs.org](https://nodejs.org/)
2. Run the installer
3. Verify installation in Command Prompt:
   ```cmd
   node --version
   npm --version
   ```

### Step 2: Install Git

#### macOS
```bash
# Using Homebrew
brew install git

# Or download from git-scm.com
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

#### Windows
Download Git from [git-scm.com](https://git-scm.com/download/win)

### Step 3: Clone Repository

```bash
# Clone the repository (replace with actual URL)
git clone https://github.com/your-org/skymarket-subpabase.git

# Navigate to project directory
cd skymarket-subpabase

# Verify project structure
ls -la
```

### Step 4: Install Project Dependencies

```bash
# Install all npm dependencies
npm install

# This will install:
# - Next.js and React
# - TypeScript and type definitions
# - Tailwind CSS and PostCSS
# - Supabase client libraries
# - Stripe SDK
# - Mapbox GL JS
# - shadcn/ui components
# - Development tools (ESLint, Prettier)
```

#### Expected Installation Output
```bash
added 400+ packages in 30s

138 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

### Step 5: Environment Configuration

```bash
# Copy environment template
cp .env.example .env.local

# The file contains:
# - Supabase configuration
# - Stripe API keys
# - Mapbox access token
# - App-specific settings
```

## Service Account Setup

### Supabase Project Creation

1. **Visit Supabase**
   - Go to [supabase.com](https://supabase.com)
   - Click "Start your project"

2. **Create Organization**
   - Choose a name (e.g., "SkyMarket Dev")
   - Select free tier

3. **Create Project**
   ```
   Name: skymarket-dev
   Database Password: [Generate secure password]
   Region: US East (Ohio) us-east-1
   ```

4. **Wait for Setup**
   - Project creation takes 2-3 minutes
   - Status will change to "Active"

5. **Get Project Details**
   ```bash
   # Project Settings → API
   Project URL: https://abcdefgh.supabase.co
   anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
   ```

### Database Schema Setup

1. **Access SQL Editor**
   - Go to Supabase dashboard
   - Click "SQL Editor" in sidebar
   - Click "New query"

2. **Run Schema Script**
   ```sql
   -- Copy entire contents of supabase/schema.sql
   -- Paste into SQL editor
   -- Click "Run"
   ```

3. **Verify Tables**
   - Go to "Table Editor"
   - Should see: profiles, providers, listings, bookings, reviews, messages, service_areas

### Stripe Account Setup

1. **Create Stripe Account**
   - Visit [stripe.com](https://stripe.com)
   - Click "Start now" → "Sign up"
   - Complete business information

2. **Activate Test Mode**
   - Toggle to "Test mode" in dashboard
   - All transactions will be simulated

3. **Get API Keys**
   ```bash
   # Developers → API keys
   Publishable key: pk_test_51...
   Secret key: sk_test_51...
   ```

### Mapbox Account Setup

1. **Create Account**
   - Visit [mapbox.com](https://mapbox.com)
   - Click "Get started for free"
   - Complete registration

2. **Get Access Token**
   ```bash
   # Account → Access tokens
   Default public token: pk.eyJ1...
   ```

## Environment Variable Configuration

Edit `.env.local` with your credentials:

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Stripe (Test Mode)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_51...
STRIPE_SECRET_KEY=sk_test_51...
STRIPE_WEBHOOK_SECRET=whsec_... # Leave empty for now

# Mapbox
NEXT_PUBLIC_MAPBOX_TOKEN=pk.eyJ1...

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25
```

## Verification

### Start Development Server

```bash
npm run dev
```

Expected output:
```bash
   ▲ Next.js 14.2.0
   - Local:        http://localhost:3000
   - Environments: .env.local

 ✓ Ready in 2.3s
```

### Test Application

1. **Homepage Test**
   - Visit http://localhost:3000
   - Should see SkyMarket homepage
   - Categories should display with icons
   - Navigation links should be visible

2. **API Health Check**
   - Open browser dev tools (F12)
   - Go to Console tab
   - Should not see Supabase errors
   - Auth state should initialize

3. **Database Connection**
   - Test in browser console:
   ```javascript
   // This should not throw errors
   localStorage.getItem('supabase.auth.token')
   ```

### Common Installation Issues

#### Node Version Conflicts
```bash
# Check current version
node --version

# If wrong version, use nvm
nvm install 18
nvm use 18
```

#### npm Permission Errors (macOS/Linux)
```bash
# Fix npm permissions
sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}
```

#### Port 3000 Already in Use
```bash
# Find process using port 3000
lsof -ti:3000

# Kill the process
kill -9 <PID>

# Or use different port
npm run dev -- --port 3001
```

#### Supabase Connection Issues
1. Check project URL format
2. Verify API keys are correct
3. Ensure project is "Active" status
4. Check internet connection

## Next Steps

After successful installation:

1. **Explore Project Structure**
   ```bash
   tree -I node_modules -L 3
   ```

2. **Review Documentation**
   - Read [Project Architecture](../architecture/ARCHITECTURE.md)
   - Understand [Database Schema](../architecture/DATABASE.md)

3. **Start Development**
   - Review [Development Workflow](./WORKFLOW.md)
   - Begin with Phase 2: Authentication

4. **Optional: Development Tools**
   - Install [VSCode Extensions](./SETUP.md#recommended-vscode-extensions)
   - Set up [Git Hooks](./SETUP.md#git-configuration)

## Support

### Self-Help Resources
1. Check [Troubleshooting Guide](./TROUBLESHOOTING.md)
2. Review [FAQ](./FAQ.md)
3. Search project issues on GitHub

### Getting Help
1. Create GitHub issue with:
   - Operating system and version
   - Node.js and npm versions
   - Error messages and stack traces
   - Steps to reproduce

### Environment Information Script
```bash
# Create debug info
echo "OS: $(uname -a)"
echo "Node: $(node --version)"
echo "npm: $(npm --version)"
echo "Git: $(git --version)"
echo "Project: $(pwd)"
echo "Dependencies: $(npm list --depth=0 2>/dev/null | grep -c "├\|└")"
```

---

**Installation Complete!** 

Next: Review [Setup Guide](./SETUP.md) for development configuration.