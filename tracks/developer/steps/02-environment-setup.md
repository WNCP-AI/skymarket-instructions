# Step 2: Environment Setup - Development Environment Preparation

**Objective**: Set up your complete development environment with all necessary tools and verify everything is working correctly.

## Overview

Before we start coding, we need to ensure your development environment is properly configured. This includes Node.js, Git, and verifying all account access.

## Node.js Setup

### Verify Node.js Installation

```bash
# Check Node.js version (should be 18+)
node --version

# Check npm version
npm --version
```

### Install or Update Node.js (if needed)

**Option 1: Using Node Version Manager (Recommended)**
```bash
# Install nvm (if not already installed)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Restart terminal or run:
source ~/.bashrc

# Install and use Node.js 18
nvm install 18
nvm use 18
nvm alias default 18
```

**Option 2: Direct Download**
- Visit [nodejs.org](https://nodejs.org) and download LTS version
- Install and verify with the commands above

## Git Configuration

### Verify Git Installation
```bash
git --version
```

### Configure Git (if not already done)
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Verify configuration
git config --global --list
```

## Development Directory Setup

### Create Project Directory
```bash
# Navigate to your preferred development location
cd ~/Development  # or wherever you keep projects

# Create a directory for hackathon projects
mkdir skymarket-hackathon
cd skymarket-hackathon
```

## Account Verification

### GitHub Account
1. Visit [github.com](https://github.com) and ensure you're logged in
2. Verify you can create repositories
3. Set up SSH keys (optional but recommended):

```bash
# Generate SSH key (if you don't have one)
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key to add to GitHub
cat ~/.ssh/id_ed25519.pub
# Add this to GitHub Settings > SSH Keys
```

### Cursor IDE Setup
1. Download and install from [cursor.so](https://cursor.so)
2. Sign in with your account
3. Verify AI features are working:
   - Open Cursor
   - Press `Cmd+L` (Mac) or `Ctrl+L` (Windows/Linux)
   - Type a simple question and verify you get a response

### Supabase Account
1. Visit [supabase.com](https://supabase.com) and sign in
2. Verify you can access the dashboard
3. **Don't create a project yet** - we'll do this in Step 10

### Stripe Account
1. Visit [stripe.com](https://stripe.com) and sign in
2. Ensure you're in **Test Mode** (toggle in the left sidebar)
3. Navigate to Developers > API Keys
4. Verify you can see test keys (we'll use these later)

### Vercel Account
1. Visit [vercel.com](https://vercel.com) and sign in
2. Connect your GitHub account if not already connected
3. Verify you can see the dashboard

### Resend Account
1. Visit [resend.com](https://resend.com) and sign in
2. Navigate to API Keys
3. Verify you can access the dashboard

## Terminal/Command Line Setup

### Verify Terminal Access
**On macOS:**
- Open Terminal app or iTerm2
- Verify you can run basic commands

**On Windows:**
- Use PowerShell or Windows Terminal
- Consider installing Windows Subsystem for Linux (WSL2) for better compatibility

**On Linux:**
- Use your preferred terminal emulator

### Install Useful Command Line Tools

**jq (for JSON processing):**
```bash
# macOS with Homebrew
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# Windows with Chocolatey
choco install jq

# Verify installation
echo '{"test": "value"}' | jq .
```

**curl (usually pre-installed):**
```bash
curl --version
```

## Environment Verification Script

Create and run this verification script to ensure everything is set up correctly:

```bash
# Create verification script
cat > verify-setup.sh << 'EOF'
#!/bin/bash

echo "🔍 Verifying Development Environment Setup"
echo "=========================================="

# Node.js check
echo -n "Node.js: "
if command -v node >/dev/null 2>&1; then
    node_version=$(node --version)
    if [[ "$node_version" =~ v1[89]|v[2-9][0-9] ]]; then
        echo "✅ $node_version (good)"
    else
        echo "⚠️  $node_version (should be v18+)"
    fi
else
    echo "❌ Not installed"
fi

# npm check
echo -n "npm: "
if command -v npm >/dev/null 2>&1; then
    echo "✅ $(npm --version)"
else
    echo "❌ Not installed"
fi

# Git check
echo -n "Git: "
if command -v git >/dev/null 2>&1; then
    echo "✅ $(git --version)"
else
    echo "❌ Not installed"
fi

# curl check
echo -n "curl: "
if command -v curl >/dev/null 2>&1; then
    echo "✅ Available"
else
    echo "❌ Not installed"
fi

# jq check (optional)
echo -n "jq: "
if command -v jq >/dev/null 2>&1; then
    echo "✅ Available"
else
    echo "⚠️  Not installed (optional)"
fi

echo ""
echo "🎯 Summary:"
echo "If you see all ✅ checks (jq is optional), you're ready to proceed!"
echo "If you see ❌ or ⚠️  for required tools, please install them first."
EOF

# Make executable and run
chmod +x verify-setup.sh
./verify-setup.sh
```

## Development Tools Setup

### Package Manager Configuration

**Set npm registry (optional optimization):**
```bash
# Use faster npm registry (optional)
npm config set registry https://registry.npmjs.org/

# Verify configuration
npm config get registry
```

### Global Package Installation

Install useful global packages:
```bash
# TypeScript compiler (optional, projects usually include this)
npm install -g typescript

# Vercel CLI (useful for deployment later)
npm install -g vercel

# Verify installations
tsc --version
vercel --version
```

## Troubleshooting

### Node.js Issues

**"node: command not found"**
- Reinstall Node.js from [nodejs.org](https://nodejs.org)
- Restart your terminal
- Check your PATH variable

**Old Node.js version**
- Use nvm to install a newer version
- Or download latest LTS from nodejs.org

### Git Issues

**"git: command not found"**
- Install Git from [git-scm.com](https://git-scm.com)
- On macOS, install Xcode command line tools: `xcode-select --install`

### Permission Issues

**npm permission errors**
```bash
# Fix npm permissions (macOS/Linux)
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.profile
source ~/.profile
```

### Account Access Issues

**Can't access accounts**
- Verify internet connection
- Try incognito/private browsing mode
- Clear browser cache and cookies
- Check if accounts require email verification

## Expected Outcome

After completing this step, you should have:

### Working Environment
- [ ] Node.js 18+ installed and working
- [ ] npm package manager available
- [ ] Git installed and configured
- [ ] Terminal/command line access

### Account Access
- [ ] GitHub account accessible
- [ ] Cursor IDE installed and signed in
- [ ] Supabase account accessible
- [ ] Stripe account in test mode
- [ ] Vercel account connected to GitHub
- [ ] Resend account accessible

### Verification
- [ ] All environment checks pass ✅
- [ ] Can run npm commands
- [ ] Can run git commands
- [ ] All accounts accessible

## Next Steps

Great! Your development environment is ready. Next, we'll explore the project specifications and understand what we're building.

---

**Previous Step**: [Step 1: Intro](./01-intro.md) | **Next Step**: [Step 3: Definition Documents](./03-definition-documents.md)

**Having issues?** Check the troubleshooting section above or review the [Prerequisites](../prerequisites.md) page.