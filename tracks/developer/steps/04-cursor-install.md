# Step 4: Cursor IDE Install - IDE Setup and Sign-in

**Objective**: Install Cursor IDE, sign in, and verify AI features are working correctly.

## Overview

Cursor is an AI-powered code editor built on top of VS Code. It provides intelligent code completion, explanations, and debugging assistance that will significantly accelerate your development process.

## Why Cursor IDE?

### AI-Powered Development
- **Smart completions**: Context-aware code suggestions
- **Code explanations**: Understand complex code instantly
- **Error debugging**: AI helps diagnose and fix issues
- **Refactoring assistance**: Improve code quality with AI guidance

### Built on VS Code
- **Familiar interface**: If you know VS Code, you know Cursor
- **Extension compatibility**: Works with VS Code extensions
- **Terminal integration**: Built-in terminal and Git support
- **Customizable**: Themes, shortcuts, and settings

### Hackathon Advantages
- **Faster development**: AI assistance speeds up coding
- **Learning accelerator**: Understand new concepts quickly
- **Error reduction**: Catch mistakes early
- **Code quality**: AI helps write better code

## Installation Process

### Step 1: Download Cursor

Visit [cursor.so](https://cursor.so) and download the appropriate version:

**macOS:**
- Download the `.dmg` file
- Open the downloaded file
- Drag Cursor to Applications folder
- Launch from Applications

**Windows:**
- Download the `.exe` installer
- Run the installer as administrator
- Follow installation prompts
- Launch from Start Menu or Desktop

**Linux:**
- Download the `.AppImage` or `.deb` file
- For AppImage: Make executable and run
- For DEB: `sudo dpkg -i cursor-*.deb`

### Step 2: Initial Launch and Setup

1. **First Launch**
   - Open Cursor IDE
   - You may see a welcome screen

2. **Sign In Process**
   - Click "Sign In" (usually in top-right corner)
   - Choose sign-in method:
     - GitHub (recommended)
     - Google
     - Email

3. **Account Creation/Login**
   - If new user: Complete account creation
   - If existing user: Enter credentials
   - Verify email if prompted

### Step 3: Verify Installation

Check that Cursor is working correctly:

```bash
# Open terminal in Cursor (Ctrl+` or Cmd+`)
# Verify basic functionality
echo "Cursor IDE is working!"

# Check if AI features are available
# Try pressing Ctrl+L or Cmd+L to open AI chat
```

## Understanding Cursor's AI Features

### Core AI Capabilities

**1. AI Chat (Ctrl+L / Cmd+L)**
- Ask questions about code
- Get explanations of functions
- Request code examples
- Debug problems

**2. Code Completions**
- Intelligent autocomplete
- Multi-line suggestions
- Context-aware recommendations

**3. Code Generation**
- Generate functions from comments
- Create boilerplate code
- Implement common patterns

### Free vs Paid Features

**Free Tier Includes:**
- Basic AI completions
- Limited AI chat queries
- Core editing features
- Extension support

**Paid Tier (Cursor Pro):**
- Unlimited AI chat
- Advanced completions
- Priority access
- Premium models

## Basic Configuration

### Essential Settings

1. **Open Settings** (Ctrl+, / Cmd+,)

2. **Configure AI Features**
   ```json
   {
     "cursor.ai.enableCodeCompletion": true,
     "cursor.ai.enableChat": true,
     "cursor.ai.model": "gpt-4" // or your preferred model
   }
   ```

3. **Editor Preferences**
   ```json
   {
     "editor.fontSize": 14,
     "editor.tabSize": 2,
     "editor.insertSpaces": true,
     "editor.formatOnSave": true
   }
   ```

### Recommended Extensions

Install these extensions for optimal Next.js development:

**Essential Extensions:**
1. **TypeScript Importer** - Auto-import TypeScript modules
2. **Prettier** - Code formatting
3. **ESLint** - Code linting
4. **Tailwind CSS IntelliSense** - Tailwind class completions

**Install Extensions:**
1. Open Extensions panel (Ctrl+Shift+X / Cmd+Shift+X)
2. Search for extension name
3. Click "Install"
4. Reload Cursor if prompted

## Testing AI Features

### Basic AI Chat Test

1. **Open AI Chat** (Ctrl+L / Cmd+L)

2. **Test Query**: Ask a simple question
   ```
   What is Next.js and why is it popular for React development?
   ```

3. **Verify Response**: You should get an informative answer

### Code Completion Test

1. **Create Test File**
   - Create new file: `test.js`
   - Type the following:

   ```javascript
   // Create a function that adds two numbers
   function add
   ```

2. **Wait for Suggestions**
   - Cursor should suggest completing the function
   - Accept suggestion with Tab or Enter

3. **Expected Result**:
   ```javascript
   // Create a function that adds two numbers
   function add(a, b) {
     return a + b;
   }
   ```

### Code Explanation Test

1. **Select Complex Code** (if available)
2. **Right-click** → "Explain with AI"
3. **Verify**: AI provides explanation

## Customization for Development

### Theme Setup
```json
{
  "workbench.colorTheme": "Dark+ (default dark)",
  // or choose your preferred theme
}
```

### Keyboard Shortcuts
Cursor uses VS Code shortcuts by default:

**Essential Shortcuts:**
- `Ctrl/Cmd + P`: Quick file open
- `Ctrl/Cmd + Shift + P`: Command palette
- `Ctrl/Cmd + L`: AI chat
- `Ctrl/Cmd + K`: Generate code
- `Ctrl/Cmd + /`: Toggle comment
- `Ctrl/Cmd + D`: Select word/next occurrence

### Terminal Configuration
```json
{
  "terminal.integrated.shell.osx": "/bin/zsh",
  "terminal.integrated.fontSize": 13,
  "terminal.integrated.cursorBlinking": true
}
```

## Troubleshooting

### Installation Issues

**"Cursor won't launch"**
- Try running as administrator (Windows)
- Check system requirements
- Reinstall if necessary

**"Can't sign in"**
- Check internet connection
- Try different sign-in method
- Clear browser cache if using web sign-in

### AI Features Not Working

**"AI chat not responding"**
- Verify you're signed in
- Check if you've exceeded free tier limits
- Try refreshing/restarting Cursor

**"No code completions"**
- Check AI settings are enabled
- Verify file type is supported
- Ensure stable internet connection

### Performance Issues

**"Cursor is slow"**
- Disable unused extensions
- Close unnecessary files/tabs
- Increase system memory if possible
- Check for system resource conflicts

## Expected Outcome

After completing this step, you should have:

### Successful Installation
- [ ] Cursor IDE installed and launches correctly
- [ ] Account created and signed in
- [ ] AI features accessible

### Verified Functionality
- [ ] AI chat responds to queries
- [ ] Code completions work
- [ ] Extensions can be installed
- [ ] Terminal is accessible

### Basic Configuration
- [ ] Preferred theme selected
- [ ] Essential extensions installed
- [ ] Keyboard shortcuts familiar
- [ ] Settings configured

## Pro Tips for Hackathon Success

### Efficient AI Usage
1. **Ask Specific Questions**: "How do I create a Supabase client in Next.js?" vs "How do I use Supabase?"
2. **Provide Context**: Include relevant code when asking questions
3. **Iterate on Responses**: Ask follow-up questions to refine answers
4. **Use for Learning**: Ask "why" questions to understand patterns

### Development Workflow
1. **Use AI for Boilerplate**: Generate common patterns quickly
2. **Debug with AI**: Paste error messages and ask for solutions
3. **Code Review**: Ask AI to review your code for improvements
4. **Documentation**: Generate comments and documentation with AI

### Time-Saving Shortcuts
1. **Quick File Navigation**: Ctrl/Cmd + P for instant file access
2. **Multi-cursor Editing**: Ctrl/Cmd + D to select multiple instances
3. **Integrated Terminal**: Stay in one window for everything
4. **Command Palette**: Ctrl/Cmd + Shift + P for any action

## Next Steps

Excellent! Cursor IDE is installed and configured. Next, we'll dive deeper into understanding Cursor's capabilities and how to use them effectively for our project.

---

**Previous Step**: [Step 3: Definition Documents](./03-definition-documents.md) | **Next Step**: [Step 5: Cursor Overview & Basics](./05-cursor-overview.md)

**Having trouble?** Check the troubleshooting section above or visit [cursor.so/docs](https://cursor.so/docs) for official documentation.