# Step 6: GitHub Import - Opening Repository in Cursor

**Objective**: Import the SkyMarket repository from GitHub into Cursor IDE and set up the local development environment.

## Overview

Now that Cursor is configured, we need to import the SkyMarket repository. This will give us access to the existing codebase, documentation, and project structure.

## Repository Information

**Repository**: `skymarket-subpabase` (assumed to be available on GitHub)
**Tech Stack**: Next.js 15, Supabase, TypeScript, Tailwind CSS
**Status**: Foundational structure with documentation

## Import Methods

### Method 1: Clone via Cursor (Recommended)

1. **Open Command Palette** (Ctrl+Shift+P / Cmd+Shift+P)
2. **Type**: "Git: Clone"
3. **Enter Repository URL**: `https://github.com/[username]/skymarket-subpabase.git`
4. **Select Local Directory**: Choose where to save the project
5. **Open in Cursor**: Select "Open" when prompted

### Method 2: Clone via Terminal

1. **Open Terminal** in your project directory
```bash
cd ~/Development/skymarket-hackathon

# Clone the repository
git clone https://github.com/[username]/skymarket-subpabase.git

# Navigate into the project
cd skymarket-subpabase
```

2. **Open in Cursor**
```bash
# Open current directory in Cursor
cursor .

# Or drag folder to Cursor window
```

### Method 3: GitHub Desktop Integration

1. **Visit Repository on GitHub**
2. **Click "Code" → "Open with GitHub Desktop"**
3. **Choose Local Path**
4. **Right-click in GitHub Desktop → "Open in Cursor"**

## Initial Repository Exploration

### Project Structure Overview

After importing, you should see:

```
skymarket-subpabase/
├── README.md              # Project overview
├── package.json           # Node.js dependencies
├── next.config.js         # Next.js configuration
├── tailwind.config.js     # Tailwind CSS config
├── tsconfig.json          # TypeScript config
├── .env.example           # Environment variables template
├── app/                   # Next.js 15 App Router
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page
│   ├── (auth)/           # Authentication routes
│   └── dashboard/        # Dashboard routes
├── components/            # Reusable components
│   ├── ui/               # shadcn/ui components
│   └── custom/           # Custom components
├── lib/                   # Utility functions
│   ├── supabase.ts       # Supabase client
│   └── utils.ts          # Helper functions
├── types/                 # TypeScript type definitions
├── docs/                  # Project documentation
│   ├── tracks/           # Hackathon tracks
│   ├── PRD.md           # Product requirements
│   └── architecture/     # Technical docs
└── public/               # Static assets
```

### Verify Import Success

**Check for Key Files:**
- [ ] `package.json` exists
- [ ] `next.config.js` present
- [ ] `app/` directory with layout.tsx
- [ ] `docs/` directory with documentation
- [ ] `.env.example` file

**Open Important Files:**
1. `README.md` - Project overview
2. `package.json` - Dependencies and scripts
3. `docs/PRD.md` - Product requirements
4. `app/layout.tsx` - Main app layout

## Installing Dependencies

### Check Node.js Version

```bash
# Verify you're using Node.js 18+
node --version
```

### Install Project Dependencies

```bash
# Install all dependencies
npm install

# This will install:
# - Next.js 15
# - React 18
# - TypeScript
# - Tailwind CSS
# - Supabase client
# - Other project dependencies
```

### Verify Installation

```bash
# Check if installation was successful
npm list --depth=0

# You should see all main dependencies listed
```

## Environment Setup

### Create Environment File

```bash
# Copy example environment file
cp .env.example .env.local

# Open for editing
cursor .env.local
```

### Environment Variables Structure

```env
# .env.local
# Supabase (will set up in Step 10)
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url_here
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_here

# Stripe (will set up in Step 17)
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_your_publishable_key
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret

# Resend (will set up in Step 20)
RESEND_API_KEY=re_your_resend_api_key

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Note**: Don't fill in values yet - we'll configure each service in later steps.

## Testing the Development Server

### Start Development Server

```bash
# Start Next.js development server
npm run dev

# Alternative script if available
npm run start:dev
```

### Verify Server Start

1. **Check Terminal Output**:
   ```
   ✓ Ready in 2.3s
   ✓ Local:    http://localhost:3000
   ✓ Network:  http://192.168.1.100:3000
   ```

2. **Open Browser**: Navigate to `http://localhost:3000`

3. **Expected Result**: 
   - Next.js default page or custom SkyMarket landing page
   - No console errors
   - Page loads successfully

## Cursor-Specific Setup

### Pin Project Documentation

Use AI chat (Ctrl+L / Cmd+L) to pin essential docs:

```
Pin the following project documentation for context:
@file:docs/PRD.md
@file:README.md
@file:package.json
@file:docs/architecture/overview.md
```

### Configure Cursor for the Project

1. **TypeScript Support**: Verify TypeScript language service is active
2. **ESLint Integration**: Check that linting is working
3. **Tailwind IntelliSense**: Ensure class completions work
4. **Git Integration**: Verify Git panel shows repository status

### Workspace Settings

Create `.vscode/settings.json` for project-specific Cursor settings:

```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "tailwindCSS.includeLanguages": {
    "typescript": "javascript",
    "typescriptreact": "javascript"
  },
  "files.associations": {
    "*.css": "tailwindcss"
  }
}
```

## Git Configuration

### Verify Git Status

```bash
# Check git status
git status

# Check remote configuration
git remote -v

# Check current branch
git branch
```

### Set Up Git Workflow

```bash
# Create development branch (optional)
git checkout -b feature/hackathon-setup

# Or continue on main branch for hackathon
git checkout main
```

## Troubleshooting

### Clone Issues

**"Repository not found"**
- Verify repository URL is correct
- Ensure you have access to the repository
- Check GitHub authentication

**"Permission denied"**
- Set up SSH keys for GitHub
- Or use HTTPS with personal access token

### Dependency Installation Issues

**"npm install" fails**
- Clear npm cache: `npm cache clean --force`
- Delete `node_modules` and `package-lock.json`
- Run `npm install` again
- Check Node.js version compatibility

### Development Server Issues

**"Port 3000 already in use"**
```bash
# Kill process on port 3000
npx kill-port 3000

# Or start on different port
npm run dev -- --port 3001
```

**"Module not found" errors**
- Verify all dependencies installed
- Check file paths and imports
- Restart TypeScript server in Cursor

### Cursor Integration Issues

**"TypeScript not working"**
- Open Command Palette → "TypeScript: Restart TS Server"
- Verify `tsconfig.json` is correct
- Check TypeScript extension is enabled

## Expected Outcome

After completing this step, you should have:

### Repository Setup
- [ ] SkyMarket repository cloned locally
- [ ] All dependencies installed successfully
- [ ] Development server runs on localhost:3000
- [ ] No critical errors in terminal or browser

### Development Environment
- [ ] Cursor IDE recognizes the project
- [ ] TypeScript support active
- [ ] Git integration working
- [ ] Environment file created (.env.local)

### Project Understanding
- [ ] Familiar with project structure
- [ ] Key files identified and opened
- [ ] Documentation pinned in Cursor
- [ ] Ready to explore codebase

## Next Steps

Excellent! The repository is imported and the development environment is running. Next, we'll orient ourselves with the codebase structure and understand how to navigate the project effectively.

---

**Previous Step**: [Step 5: Cursor Overview](./05-cursor-overview.md) | **Next Step**: [Step 7: Repo Orientation](./07-repo-orientation.md)

**Having issues?** Check the troubleshooting section above or ensure you've completed the previous steps successfully.