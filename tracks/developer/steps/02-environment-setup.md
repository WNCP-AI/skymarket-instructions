# Step 2: Environment Setup

## Objective

Set up all necessary accounts, tools, and environment variables required for spec-driven development of the SkyMarket application.

## Prerequisites Check

Before proceeding, ensure you have:
- ✅ Computer with macOS, Windows, or Linux
- ✅ At least 8GB RAM (16GB recommended)
- ✅ 10GB free disk space
- ✅ Stable internet connection
- ✅ Basic command line familiarity

## Required Accounts

You'll need to create free accounts for the following services. We'll use these throughout the tutorial.

### 1. GitHub Account
**Purpose**: Code repository and version control

**Setup**:
1. Visit [github.com](https://github.com)
2. Click "Sign up"
3. Create account with your email
4. Verify your email address

**Verification**:
- [ ] Can log into GitHub
- [ ] Email verified
- [ ] Profile created

### 2. Supabase Account
**Purpose**: Database, authentication, and real-time features

**Setup**:
1. Visit [supabase.com](https://supabase.com)
2. Click "Start your project"
3. Sign up with GitHub (recommended) or email
4. No credit card required for free tier

**Verification**:
- [ ] Can access Supabase dashboard
- [ ] Account linked to GitHub (if applicable)

### 3. Stripe Account
**Purpose**: Payment processing and marketplace payments

**Setup**:
1. Visit [stripe.com](https://stripe.com)
2. Click "Sign up"
3. Create account with email
4. Complete basic information (test mode is fine)
5. Skip bank account setup for now (development only)

**Verification**:
- [ ] Can access Stripe dashboard
- [ ] Test mode enabled
- [ ] API keys visible in dashboard

### 4. Vercel Account
**Purpose**: Application deployment and hosting

**Setup**:
1. Visit [vercel.com](https://vercel.com)
2. Click "Sign up"
3. Sign up with GitHub (recommended)
4. Authorize Vercel to access your GitHub

**Verification**:
- [ ] Can access Vercel dashboard
- [ ] GitHub integration connected

### 5. Resend Account
**Purpose**: Transactional email service

**Setup**:
1. Visit [resend.com](https://resend.com)
2. Click "Sign up"
3. Create account with email
4. Verify your email address
5. Free tier includes 100 emails/day

**Verification**:
- [ ] Can access Resend dashboard
- [ ] Email verified
- [ ] API key available

### 6. Mapbox Account
**Purpose**: Interactive maps and location services

**Setup**:
1. Visit [mapbox.com](https://mapbox.com)
2. Click "Sign up"
3. Create account with email
4. Free tier includes 50,000 map loads/month

**Verification**:
- [ ] Can access Mapbox Studio
- [ ] Default public token visible

### 7. OpenAI Account (Optional)
**Purpose**: AI features and chatbot functionality

**Setup**:
1. Visit [platform.openai.com](https://platform.openai.com)
2. Sign up or sign in
3. Navigate to API keys section
4. Free credits available for new accounts

**Verification**:
- [ ] Can access API keys page
- [ ] Credits or billing set up

## Development Tools

### 1. Node.js and npm
**Purpose**: JavaScript runtime and package manager

**Installation**:

**macOS** (using Homebrew):
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js
brew install node
```

**Windows** (using Chocolatey):
```bash
# Install Chocolatey if not installed (run as Administrator)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install Node.js
choco install nodejs
```

**Linux** (Ubuntu/Debian):
```bash
# Using NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Verification**:
```bash
node --version  # Should show v18.0.0 or higher
npm --version   # Should show 9.0.0 or higher
```

### 2. Git
**Purpose**: Version control

**Installation**:

**macOS**:
```bash
brew install git
```

**Windows**:
```bash
choco install git
```

**Linux**:
```bash
sudo apt-get install git
```

**Configuration**:
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Verification**:
```bash
git --version  # Should show 2.0.0 or higher
```

### 3. VS Code (Backup Editor)
**Purpose**: Code editor as backup to Cursor

**Installation**:
1. Visit [code.visualstudio.com](https://code.visualstudio.com)
2. Download for your operating system
3. Install following the installer prompts

**Verification**:
- [ ] VS Code opens successfully
- [ ] Can create a new file

## Project Setup

### 1. Create Project Directory
```bash
# Create a workspace directory
mkdir -p ~/projects/skymarket
cd ~/projects/skymarket
```

### 2. Fork and Clone Repositories

Before cloning, fork both repositories into your GitHub account:

- skymarket-instructions (these docs)
- skymarket-supabase (app code)

Then clone your forks into the workspace directory:

```bash
# Clone docs (optional if you already have them open)
git clone https://github.com/<your-username>/skymarket-instructions.git

# Clone app code
git clone https://github.com/<your-username>/skymarket-supabase.git
cd skymarket-supabase
```

Full code demo:

```bash
# Create workspace
mkdir -p ~/projects/skymarket && cd ~/projects/skymarket

# Fork both repos on GitHub first, then:
git clone https://github.com/<your-username>/skymarket-instructions.git
git clone https://github.com/<your-username>/skymarket-supabase.git
cd skymarket-supabase
```

**Note**: Replace `<your-username>` with your GitHub username. If you use SSH, swap URLs accordingly (e.g., `git@github.com:<your-username>/skymarket-supabase.git`).

### 3. Environment Variables Template
Create a `.env.local` file template (we'll fill this in later):

```bash
# Create environment file
touch .env.local
```

Add this template:
```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=your_stripe_publishable_key
STRIPE_SECRET_KEY=your_stripe_secret_key
STRIPE_WEBHOOK_SECRET=your_webhook_secret

# Resend
RESEND_API_KEY=your_resend_api_key

# OpenAI (Optional)
OPENAI_API_KEY=your_openai_api_key

# Mapbox
NEXT_PUBLIC_MAPBOX_TOKEN=your_mapbox_token

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_SERVICE_AREA_CENTER_LAT=42.3314
NEXT_PUBLIC_SERVICE_AREA_CENTER_LNG=-83.0458
NEXT_PUBLIC_SERVICE_AREA_RADIUS_MILES=25
```

## Troubleshooting Tips

### Account Creation Issues

**Problem**: Email verification not received
**Solution**: Check spam folder, request resend, try different email

**Problem**: GitHub authorization fails with Vercel/Supabase
**Solution**: Check GitHub settings → Applications → Revoke and re-authorize

### Installation Issues

**Problem**: Permission denied during installation
**Solution**: Use `sudo` for Linux/macOS, run as Administrator on Windows

**Problem**: Old Node.js version installed
**Solution**: Uninstall existing version, use nvm for version management:
```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
# Install latest Node
nvm install node
nvm use node
```

### Environment Issues

**Problem**: Command not found after installation
**Solution**: Restart terminal or refresh PATH:
```bash
# macOS/Linux
source ~/.bashrc  # or ~/.zshrc

# Windows
# Restart PowerShell/Command Prompt
```

## Pro Tips

### API Key Management
- **Never commit** `.env.local` to Git
- **Keep keys secure** - treat them like passwords
- **Use test keys** during development
- **Rotate keys** if accidentally exposed

### Account Organization
- **Use consistent email** across services when possible
- **Enable 2FA** on GitHub and payment services
- **Document API keys** in a password manager
- **Set up billing alerts** even for free tiers

### Development Environment
- **Use a dedicated browser profile** for development
- **Install browser extensions**: React DevTools, Redux DevTools
- **Set up terminal aliases** for common commands
- **Keep dependencies updated** regularly

## Next Steps

Excellent! Your development environment is ready. Next, we'll install Cursor IDE and set it up for spec-driven development.

### What's Coming in Step 3
- Download and install Cursor IDE
- Configure Cursor for optimal performance
- Set up Claude AI integration
- Prepare for MCP Context7 installation

---

**Next Step**: [Step 3: Cursor IDE Installation](./03-cursor-install.md) →

**Quick Links**:
- [Previous: Introduction](./01-intro.md)
- [Track Overview](../README.md)
- [Project Specifications](../../../specs/)