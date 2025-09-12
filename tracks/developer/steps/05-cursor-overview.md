# Step 5: Cursor Overview & Basics - Understanding Models, Settings, and Context Usage

**Objective**: Master Cursor's AI features including models, settings, context management (@file, URLs, images), and documentation pinning.

## Overview

Now that Cursor is installed, let's explore its powerful AI features. Understanding these capabilities will make you significantly more productive during the hackathon.

## Understanding AI Models

### Available Models in Cursor

**GPT-4 (Recommended for Complex Tasks)**
- Best reasoning and code generation
- Higher quality responses
- More expensive (limited in free tier)
- Use for: Architecture decisions, complex debugging, learning

**GPT-3.5 Turbo (Fast and Efficient)**
- Quick responses
- Good for routine tasks
- More generous usage limits
- Use for: Simple completions, boilerplate code

**Claude (Alternative Model)**
- Different reasoning style
- Good for code analysis
- Available in some regions
- Use for: Code review, explanations

### Switching Between Models

1. **In AI Chat (Ctrl+L / Cmd+L)**
   - Look for model selector dropdown
   - Choose appropriate model for your task

2. **For Code Completions**
   - Set in Settings → AI → Default Model
   - Or use quick command: Ctrl+Shift+P → "Change AI Model"

## AI Chat Interface Deep Dive

### Opening AI Chat
- **Keyboard Shortcut**: Ctrl+L (Windows/Linux) or Cmd+L (Mac)
- **Command Palette**: Ctrl+Shift+P → "Open AI Chat"
- **UI Button**: Click chat icon if available

### Chat Interface Components

```
┌─────────────────────────────────────┐
│ Model Selector    [GPT-4 ▼]   [⚙️] │
├─────────────────────────────────────┤
│                                     │
│ Chat Messages Area                  │
│                                     │
├─────────────────────────────────────┤
│ Type your message...        [Send]  │
│ @file | @url | @image              │
└─────────────────────────────────────┘
```

### Effective Prompting Strategies

**Good Prompts:**
```
❌ "How do I use Supabase?"
✅ "How do I set up Supabase authentication in Next.js 15 with App Router?"

❌ "Fix this error"
✅ "I'm getting 'Module not found' error for @/lib/supabase. Here's my file structure: [context]. How do I fix the import path?"
```

**Prompt Templates for Common Tasks:**

**Learning New Concepts:**
```
Explain [concept] in the context of [framework/technology].
Give me a practical example for [specific use case].
```

**Code Generation:**
```
Create a [component/function] that [specific requirement].
Use [specific technologies/patterns].
Include TypeScript types and error handling.
```

**Debugging:**
```
I'm getting this error: [error message]
Here's the relevant code: [code snippet]
What's causing this and how do I fix it?
```

## Context Management with @-Commands

### @file - Reference Specific Files

**Usage:**
```
@file:components/Header.tsx How do I add a user menu to this header component?
```

**Benefits:**
- AI sees the entire file content
- More accurate suggestions
- Context-aware responses

**Best Practices:**
- Reference files relevant to your question
- Include multiple related files if needed
- Use for code modifications and explanations

### @url - Include Web Content

**Usage:**
```
@url:https://nextjs.org/docs/app/building-your-application/routing
How do I implement dynamic routes based on this documentation?
```

**Common Use Cases:**
- Reference official documentation
- Include Stack Overflow solutions
- Share GitHub repositories or examples

**Supported URLs:**
- Documentation sites
- GitHub repositories
- Stack Overflow questions
- Blog posts and tutorials

### @image - Analyze Images

**Usage:**
```
@image:design-mockup.png
Create a React component that matches this design
```

**Use Cases:**
- Convert design mockups to code
- Debug UI layout issues
- Analyze error screenshots
- Understand system diagrams

### Combining Context Types

```
@file:components/LoginForm.tsx @url:https://supabase.com/docs/guides/auth
How do I integrate Supabase auth with this existing form component?
```

## Settings Configuration

### Accessing Settings
1. **Menu**: Cursor → Preferences → Settings
2. **Keyboard**: Ctrl+, (Windows/Linux) or Cmd+, (Mac)
3. **Command Palette**: Ctrl+Shift+P → "Preferences: Open Settings"

### Essential AI Settings

**Core AI Configuration:**
```json
{
  "cursor.ai.enableCodeCompletion": true,
  "cursor.ai.enableChat": true,
  "cursor.ai.defaultModel": "gpt-4",
  "cursor.ai.completionSettings": {
    "maxTokens": 1000,
    "temperature": 0.1
  }
}
```

**Code Completion Settings:**
```json
{
  "cursor.ai.autoTrigger": true,
  "cursor.ai.inlineCompletions": true,
  "cursor.ai.multilineCompletions": true
}
```

### YOLO Mode vs Blocklist

**YOLO Mode (Accept All)**
- Pros: Maximum AI assistance, fastest development
- Cons: May accept incorrect suggestions
- Best for: Rapid prototyping, learning new patterns

**Blocklist Mode (Selective)**
- Pros: More control, avoid bad suggestions
- Cons: Slower, requires more manual review
- Best for: Production code, critical functionality

**Configuring Safety:**
```json
{
  "cursor.ai.safetyMode": "blocklist", // or "yolo"
  "cursor.ai.reviewSuggestions": true,
  "cursor.ai.confirmBeforeExecution": true
}
```

## Pinning Documentation

### Why Pin Documentation?

**Persistent Context:**
- Documentation stays available across chat sessions
- No need to re-reference common docs
- Faster responses for framework-specific questions

**Knowledge Base:**
- Build project-specific knowledge
- Share context with team members
- Maintain consistency across development

### How to Pin Documents

**Method 1: Through Chat**
```
@url:https://nextjs.org/docs/app
Pin this Next.js App Router documentation for our project
```

**Method 2: Settings**
1. Open Settings → AI → Pinned Documents
2. Add URLs or file paths
3. Set access permissions

### Recommended Pins for SkyMarket

**Core Documentation:**
```
https://nextjs.org/docs/app
https://supabase.com/docs/guides/getting-started
https://docs.stripe.com/payments/accept-a-payment
https://resend.com/docs/introduction
https://ui.shadcn.com/docs
```

**Project-Specific:**
```
@file:docs/PRD.md
@file:docs/architecture/overview.md
@file:types/global.d.ts
```

## Advanced Features

### Code Generation Shortcuts

**Ctrl+K / Cmd+K**: Generate code inline
```javascript
// Type comment then press Ctrl+K
// Create a function that validates email format
```

**Tab Completion**: Accept AI suggestions
- Tab: Accept full suggestion
- Ctrl+→: Accept word by word
- Esc: Reject suggestion

### Multi-file Context

**Select Multiple Files:**
```
@file:components/auth/LoginForm.tsx @file:lib/supabase.ts @file:types/auth.ts
How do these authentication files work together?
```

**Project-wide Questions:**
```
@file:package.json @file:next.config.js @file:tailwind.config.js
Review my project configuration for Next.js 15 compatibility
```

## Practical Exercises

### Exercise 1: Context Usage

1. **Create test question with different context types:**
   ```
   Without context: "How do I create a button component?"
   
   With URL context: "@url:https://ui.shadcn.com/docs/components/button How do I create a button component using shadcn/ui?"
   
   With file context: "@file:components/ui/button.tsx How do I modify this button to support different variants?"
   ```

2. **Compare the responses** - notice how context improves accuracy

### Exercise 2: Model Comparison

1. **Ask the same question to different models**
2. **Compare response quality and speed**
3. **Choose optimal model for different task types**

### Exercise 3: Documentation Pinning

1. **Pin Next.js App Router docs**
2. **Ask framework-specific questions**
3. **Notice improved response relevance**

## Troubleshooting

### AI Not Responding
- Check internet connection
- Verify you're signed in
- Try different model
- Check usage limits

### Poor Suggestion Quality
- Provide more context with @-commands
- Use more specific prompts
- Try different model
- Include relevant files/docs

### Performance Issues
- Reduce number of pinned documents
- Close unnecessary files
- Check system resources
- Restart Cursor if needed

## Expected Outcome

After completing this step, you should understand:

### AI Models
- [ ] Differences between GPT-4, GPT-3.5, and Claude
- [ ] When to use each model
- [ ] How to switch between models

### Context Management
- [ ] How to use @file, @url, and @image
- [ ] Best practices for providing context
- [ ] Combining multiple context types

### Settings Configuration
- [ ] Essential AI settings
- [ ] YOLO vs Blocklist modes
- [ ] Code completion preferences

### Documentation Pinning
- [ ] Why and how to pin docs
- [ ] Recommended documentation for the project
- [ ] Managing pinned resources

## Pro Tips for Maximum Productivity

### Efficient Workflows
1. **Start with documentation pinning** for immediate context
2. **Use @file references** when modifying existing code  
3. **Combine multiple context types** for complex questions
4. **Switch models based on task complexity**

### Quality Assurance
1. **Review AI suggestions** before accepting
2. **Test generated code** immediately
3. **Ask for explanations** of complex suggestions
4. **Iterate and refine** prompts for better results

### Learning Acceleration
1. **Ask "why" questions** to understand patterns
2. **Request alternative approaches** for comparison
3. **Use AI to explain** unfamiliar code patterns
4. **Generate examples** for new concepts

## Next Steps

Perfect! You now understand Cursor's AI capabilities. Next, we'll import the SkyMarket repository and start exploring the codebase.

---

**Previous Step**: [Step 4: Cursor Install](./04-cursor-install.md) | **Next Step**: [Step 6: GitHub Import](./06-github-import.md)

**Questions about AI features?** Experiment with the exercises above or check the [Cursor documentation](https://cursor.so/docs).