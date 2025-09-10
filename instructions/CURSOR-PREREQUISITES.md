# Cursor AI Editor: Prerequisites & Essential Knowledge

## What is Cursor?

Cursor is an AI-powered code editor built on VSCode that integrates AI assistance directly into your development workflow. It allows you to code with AI through chat, inline edits, and codebase-aware suggestions.

## Getting Started with Cursor

### 1. Download & Installation

**Download Cursor:**
- Go to [cursor.com](https://cursor.com)
- Download for your OS (Mac, Windows, or Linux)
- Install like any other application
- On first launch, it will import your VSCode settings if you have them

### 2. Account Setup

**Free vs Pro:**
- **Free Tier**: 
  - 50 requests/month with GPT-4
  - 200 requests with GPT-3.5
  - Basic features
- **Pro Tier ($20/month)**:
  - 500 fast requests with GPT-4
  - Unlimited slow requests
  - Claude-3.5-Sonnet access
  - Codebase indexing

**Initial Setup:**
1. Sign in with GitHub or email
2. Choose your plan
3. Select your AI model preference (GPT-4 or Claude)

### 3. Key Features You Need to Know

#### **Chat (Cmd+L / Ctrl+L)**
- Opens AI chat panel
- Can reference files with @filename
- Can reference documentation with @docs
- Can reference entire codebase with @codebase

**Example:**
```
@codebase how does the authentication work in this app?
```

#### **Inline Edit (Cmd+K / Ctrl+K)**
- Edit code directly in the editor
- Select code and press Cmd+K
- Type your instruction
- AI will modify the selected code

**Example:**
```
"add error handling to this function"
```

#### **Composer (Cmd+I / Ctrl+I)**
- Multi-file editing mode
- Can create or edit multiple files at once
- Best for large features or refactoring

**Example:**
```
"create a new dashboard page with navigation and user stats"
```

#### **Terminal (Right-click in terminal)**
- AI can write terminal commands
- Helps with git, npm, and deployment commands

### 4. Essential Cursor Settings

**Open Settings:** Cmd+, (Mac) or Ctrl+, (Windows/Linux)

**Recommended Settings:**
```json
{
  "cursor.aiModel": "claude-3.5-sonnet",  // or "gpt-4"
  "cursor.indexing": true,                 // Enable codebase indexing
  "cursor.copilot++": true,               // Better autocomplete
  "cursor.chat.showSuggestions": true,    // Show chat suggestions
  "cursor.format.enable": true            // Auto-format AI responses
}
```

### 5. Cursor-Specific Files

#### **.cursorrules**
Create a `.cursorrules` file in your project root to define AI behavior:

```markdown
# Project Rules for Cursor AI

## Tech Stack
- Next.js 15 with App Router
- TypeScript
- Supabase
- Tailwind CSS
- shadcn/ui

## Coding Standards
- Use functional components with TypeScript
- Prefer server components unless client interactivity needed
- Always handle errors with try-catch
- Use Zod for validation

## File Naming
- Components: PascalCase.tsx
- Utilities: camelCase.ts
- API routes: route.ts

## Do NOT
- Create test files unless asked
- Add comments unless asked
- Change existing working code without permission
```

#### **.cursorignore**
Similar to .gitignore, tells Cursor what to ignore:

```
node_modules/
.next/
dist/
*.log
.env.local
```

### 6. Best Practices for Prompting in Cursor

#### **Be Specific with Context**
❌ Bad: "Fix this"
✅ Good: "Fix the TypeScript error in the useAuth hook return type"

#### **Reference Files**
Use @ to reference specific context:
- `@file.tsx` - Reference specific file
- `@folder` - Reference entire folder
- `@codebase` - Search entire project

#### **Multi-Step Instructions**
Break complex tasks into steps:
```
1. Create a new component called UserProfile
2. Add props for user data
3. Style it with Tailwind
4. Export from components/index.ts
```

#### **Use Code Blocks for Examples**
When showing desired output:
```typescript
// I want a function like this:
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0)
}
```

### 7. Keyboard Shortcuts

**Essential Shortcuts:**

| Action | Mac | Windows/Linux |
|--------|-----|---------------|
| Open Chat | Cmd+L | Ctrl+L |
| Inline Edit | Cmd+K | Ctrl+K |
| Composer | Cmd+I | Ctrl+I |
| Accept Suggestion | Tab | Tab |
| Reject Suggestion | Esc | Esc |
| New Chat | Cmd+Shift+L | Ctrl+Shift+L |
| Toggle Sidebar | Cmd+B | Ctrl+B |
| Command Palette | Cmd+Shift+P | Ctrl+Shift+P |

### 8. Working with the SkyMarket Project

#### **Initial Setup Prompts for Cursor:**

1. **First prompt after opening project:**
```
@.cursorrules analyze this project structure and understand it's a Next.js marketplace app with Supabase
```

2. **Before making changes:**
```
@codebase what is the current authentication flow?
```

3. **When adding features:**
```
@schema.sql @types/database.ts create a new API endpoint for fetching user orders
```

### 9. Common Cursor Workflows

#### **Workflow 1: Adding a New Feature**
1. Open Composer (Cmd+I)
2. Reference related files with @
3. Describe the feature
4. Let Cursor create/modify multiple files
5. Review changes before accepting

#### **Workflow 2: Debugging**
1. Select error code
2. Press Cmd+K
3. Type "fix this error"
4. Or use chat: "@file explain this error and how to fix it"

#### **Workflow 3: Refactoring**
1. Select code block
2. Cmd+K
3. "refactor this to use modern React patterns"

#### **Workflow 4: Learning the Codebase**
1. Cmd+L for chat
2. "@codebase how does [feature] work?"
3. "@codebase where is [functionality] implemented?"

### 10. Cursor vs GitHub Copilot vs ChatGPT

**Cursor Advantages:**
- Integrated in editor
- Understands entire codebase
- Can edit multiple files
- Shows changes inline
- No copy-paste needed

**When to Use Cursor:**
- Writing new features
- Refactoring existing code
- Debugging
- Understanding codebase

**When to Use ChatGPT/Claude:**
- Planning architecture
- Learning concepts
- Writing documentation
- Complex explanations

### 11. Troubleshooting Common Issues

**Issue: Cursor not understanding context**
- Solution: Index your codebase (Settings → Features → Indexing)

**Issue: Running out of requests**
- Solution: Use GPT-3.5 for simple tasks, save GPT-4/Claude for complex ones

**Issue: Incorrect code generation**
- Solution: Be more specific, provide examples, reference relevant files

**Issue: Cursor is slow**
- Solution: Disable unused extensions, clear cache, restart

### 12. Pro Tips

1. **Use .cursorrules**: Define project standards once
2. **Reference types**: Always @ reference your TypeScript types
3. **Incremental changes**: Make small changes and test
4. **Review everything**: Never blindly accept AI suggestions
5. **Learn shortcuts**: Speed up workflow significantly
6. **Context is key**: More context = better suggestions
7. **Use the terminal**: Let Cursor write complex commands
8. **Batch operations**: Use Composer for multi-file changes

### 13. Setting Up for SkyMarket Development

**After installing Cursor:**

1. Open the skymarket folder
2. Create `.cursorrules` with project standards
3. Let Cursor index the codebase (happens automatically)
4. Test with: "explain the app structure"
5. Start with BUILD-GUIDE.md steps

### 14. Cost Optimization

**Save AI requests by:**
- Using autocomplete for simple code
- Batching related changes together
- Using GPT-3.5 for basic tasks
- Being specific to avoid regeneration
- Using @file instead of @codebase when possible

---

## Ready to Start?

You now have everything needed to use Cursor effectively with the SkyMarket project. Open Cursor, load the project, and start with the BUILD-GUIDE.md to create your marketplace app!