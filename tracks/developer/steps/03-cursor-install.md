# Step 3: Cursor IDE Installation

## Objective

Install and configure Cursor IDE as your primary development environment for AI-assisted, spec-driven development.

## What is Cursor IDE?

Cursor is a code editor built specifically for AI-assisted programming. It integrates Claude directly into your development workflow, making it perfect for spec-driven development where you load specifications as context and generate implementation code.

### Why Cursor for Spec-Driven Development?

- **Native AI Integration**: Claude is built into the editor
- **Context Loading**: Easy `@` mention system for loading files
- **Smart Autocomplete**: AI-powered code completion
- **Chat Interface**: Cmd+L/Ctrl+L for conversational coding
- **File Context**: AI understands your entire project structure

## Installation

### Download Cursor

1. **Visit the Official Site**
   - Go to [cursor.com](https://cursor.com)
   - Click "Download for [Your OS]"

2. **Choose Your Platform**
   - **macOS**: Download `.dmg` file
   - **Windows**: Download `.exe` installer
   - **Linux**: Download `.AppImage` or `.deb` package

### Installation Steps

#### macOS Installation
```bash
# If downloaded via browser:
1. Open the downloaded .dmg file
2. Drag Cursor to Applications folder
3. Launch from Applications or Spotlight

# Or install via Homebrew Cask:
brew install --cask cursor
```

#### Windows Installation
```bash
# Run the downloaded installer
1. Double-click the .exe file
2. Follow installation wizard
3. Choose installation directory
4. Launch from Start Menu or Desktop

# Or install via winget:
winget install Cursor.Cursor
```

#### Linux Installation
```bash
# For .AppImage:
chmod +x cursor-*.AppImage
./cursor-*.AppImage

# For .deb (Ubuntu/Debian):
sudo dpkg -i cursor-*.deb
sudo apt-get install -f  # Fix any dependencies

# Or use snap:
sudo snap install cursor --classic
```

## First Launch Setup

### 1. Initial Configuration

When you first open Cursor:

1. **Accept Terms**: Read and accept the terms of service
2. **Sign In**: Create account or sign in with GitHub/Google
3. **Import Settings** (optional): Import VS Code settings if available

### 2. AI Setup

Configure Claude integration:

1. **AI Model Selection**:
   - Click the AI button in the bottom right
   - Verify Claude is selected as your model
   - Check that you have access to Claude 3.5 Sonnet

2. **Subscription** (if needed):
   - Cursor Pro provides unlimited AI usage
   - Free tier has usage limits
   - Consider upgrading for this tutorial

### 3. Essential Extensions

Install these key extensions for our stack:

```
Extensions to install:
1. TypeScript and JavaScript Support (usually pre-installed)
2. Tailwind CSS IntelliSense
3. ES7+ React/Redux/React-Native snippets
4. Prettier - Code formatter
5. ESLint
6. GitLens
```

**Installation Process**:
1. Press `Cmd+Shift+X` (Mac) or `Ctrl+Shift+X` (Windows/Linux)
2. Search for each extension
3. Click "Install"

## Configuration for Spec-Driven Development

### 1. Workspace Settings

Create optimal settings for our workflow:

**File** → **Preferences** → **Settings** (or `Cmd+,`)

Add these settings:

```json
{
  "editor.fontSize": 14,
  "editor.lineHeight": 1.5,
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "files.exclude": {
    "node_modules": true,
    ".next": true,
    "out": true,
    ".env.local": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.updateImportsOnFileMove.enabled": "always"
}
```

### 2. Cursor-Specific Settings

Configure AI features:

1. **Open Settings** → Search for "cursor"
2. **AI Settings**:
   - Enable "Auto-complete suggestions"
   - Enable "Code context awareness"
   - Set context length to maximum available

3. **Chat Settings**:
   - Enable "Use project context"
   - Enable "Include file tree in context"

### 3. Keyboard Shortcuts

Memorize these essential shortcuts:

```
AI Chat: Cmd+L (Mac) / Ctrl+L (Windows/Linux)
AI Inline: Cmd+K (Mac) / Ctrl+K (Windows/Linux)
Command Palette: Cmd+Shift+P / Ctrl+Shift+P
File Explorer: Cmd+Shift+E / Ctrl+Shift+E
Search: Cmd+Shift+F / Ctrl+Shift+F
Terminal: Cmd+` / Ctrl+`
```

## Testing Your Setup

### 1. Create Test Project

Let's verify everything works:

```bash
# Create a test directory
mkdir cursor-test
cd cursor-test

# Initialize a simple project
echo "console.log('Hello from Cursor!');" > test.js

# Open in Cursor
cursor .
```

### 2. Test AI Features

#### Test 1: Basic Chat
1. Press `Cmd+L` / `Ctrl+L` to open chat
2. Type: "Explain what this JavaScript code does"
3. Drag `test.js` into the chat or use `@test.js`
4. Verify you get a response about console.log

#### Test 2: Context Loading
1. In chat, type: `@test.js`
2. Verify the file appears in context
3. Ask: "Add error handling to this code"
4. Check that AI provides relevant suggestions

#### Test 3: Code Generation
1. Create new file: `example.ts`
2. Press `Cmd+K` / `Ctrl+K` for inline AI
3. Type: "Create a TypeScript function that adds two numbers"
4. Verify AI generates appropriate TypeScript code

## Optimizing for SkyMarket Development

### 1. Project Structure Awareness

Configure Cursor to understand our project:

1. **Add to workspace settings** (`.vscode/settings.json`):
```json
{
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "typescript.suggest.autoImports": true,
  "typescript.preferences.importModuleSpecifier": "relative",
  "tailwindCSS.includeLanguages": {
    "typescript": "javascript",
    "typescriptreact": "javascript"
  },
  "tailwindCSS.experimental.classRegex": [
    "class:\\s*?[\"'`]([^\"'`]*).*?[\"'`]",
    "(?:enter|leave)(?:From|To)?:\\s*?[\"'`]([^\"'`]*).*?[\"'`]"
  ]
}
```

### 2. Snippet Configuration

Add useful snippets for our stack:

**Create** `.vscode/snippets.json`:
```json
{
  "Next.js Page Component": {
    "prefix": "npage",
    "body": [
      "export default function ${1:PageName}() {",
      "  return (",
      "    <div className=\"${2:container}\">",
      "      <h1>${3:Page Title}</h1>",
      "    </div>",
      "  )",
      "}"
    ]
  },
  "Supabase Client": {
    "prefix": "supa",
    "body": [
      "import { createClient } from '@/lib/supabase/client'",
      "",
      "const supabase = createClient()"
    ]
  }
}
```

### 3. File Associations

Set up file associations for our project:

```json
{
  "files.associations": {
    "*.css": "tailwindcss",
    ".env.local": "dotenv",
    ".env.example": "dotenv"
  }
}
```

## Troubleshooting

### Common Installation Issues

**Problem**: Cursor won't start after installation
**Solution**:
- Check system requirements (64-bit OS required)
- Try running as administrator (Windows) or with sudo (Linux)
- Verify antivirus isn't blocking the application

**Problem**: AI features not working
**Solution**:
- Check internet connection
- Sign out and sign back in
- Verify account has AI access
- Check usage limits

**Problem**: Extensions not installing
**Solution**:
- Disable firewall temporarily
- Clear extension cache: `Cmd+Shift+P` → "Reload Window"
- Try installing extensions manually from marketplace

### Performance Issues

**Problem**: Cursor running slowly
**Solution**:
- Close unnecessary files and tabs
- Disable unused extensions
- Increase memory allocation in preferences
- Close other resource-intensive applications

**Problem**: AI responses are slow
**Solution**:
- Reduce context size (don't load too many files)
- Use more specific prompts
- Check network connection
- Consider upgrading to Cursor Pro

### File Context Issues

**Problem**: `@` mentions not finding files
**Solution**:
- Ensure file exists and is saved
- Check file permissions
- Reload window: `Cmd+Shift+P` → "Reload Window"
- Verify correct project root is open

## Expected Outcome

After completing this step, you should have:

### Working Cursor Installation
- [ ] Cursor IDE installed and launches successfully
- [ ] Signed in with account
- [ ] AI features (Claude) working
- [ ] Can access chat with `Cmd+L`/`Ctrl+L`

### Proper Configuration
- [ ] Essential extensions installed
- [ ] Settings configured for our stack
- [ ] Keyboard shortcuts memorized
- [ ] File associations set up

### Verified AI Features
- [ ] Chat interface responds to questions
- [ ] `@` file mentions work correctly
- [ ] Inline AI (`Cmd+K`/`Ctrl+K`) functions
- [ ] Code completion suggestions appear

## Pro Tips

### Context Management
- **Be selective**: Don't load too many files at once
- **Use specific files**: `@filename` rather than entire folders
- **Clear context**: Start fresh chats for different topics
- **Verify loading**: Check that files appear in context sidebar

### Prompt Writing
- **Be specific**: "Create a TypeScript interface for user data" vs "make interface"
- **Include requirements**: Reference specs when available
- **Ask for explanations**: "Explain why you chose this approach"
- **Request verification**: "Check this against the requirements"

### Workflow Optimization
- **Multiple windows**: Open specs in second window
- **Split panes**: Keep chat open while coding
- **Quick access**: Pin frequently used files
- **Terminal integration**: Use built-in terminal for commands

## Next Steps

Perfect! Cursor IDE is installed and configured for spec-driven development. Next, we'll set up MCP Context7 to access official library documentation.

### What's Coming in Step 4
- Install and configure MCP Context7 server
- Connect Context7 to Cursor IDE
- Test official documentation access
- Learn Context7 command syntax

---

**Next Step**: [Step 4: MCP Context7 Setup](./04-mcp-context7.md) →

**Quick Links**:
- [Previous: Environment Setup](./02-environment-setup.md)
- [Track Overview](../README.md)
- [Project Specifications](../../../specs/)


