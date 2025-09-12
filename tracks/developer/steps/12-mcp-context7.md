# Step 12: MCP Context7 in Cursor - Enhanced Development Workflow

## Objective
Set up and use Context7 Model Context Protocol (MCP) in Cursor IDE to access real-time, authoritative documentation from libraries and frameworks. This supercharges your development workflow by providing accurate, up-to-date API references directly in your coding context.

## What You'll Learn
- How to install and configure Context7 MCP for Cursor
- How to use context-aware prompts for better code generation
- How to leverage official documentation in your development workflow
- Best practices for prompt engineering with external context

## Prerequisites
- Cursor IDE installed and configured (from Step 04)
- SkyMarket project open in Cursor
- Basic understanding of AI-assisted development
- Node.js environment available

## Understanding MCP & Context7

### What is Model Context Protocol (MCP)?
MCP is a standard that allows AI assistants to securely access external data sources and tools during conversations. Think of it as a bridge between your AI assistant and external services.

**Benefits:**
- **Real-time data:** Always up-to-date information
- **Authoritative sources:** Official documentation, not outdated examples
- **Context-aware:** Relevant information based on your current task
- **Reduced hallucination:** Facts from verified sources

### What is Context7?
Context7 is an MCP server that provides access to official documentation from popular development frameworks and libraries.

**Supported Libraries:**
- **Next.js:** App Router, API routes, middleware
- **Supabase:** Authentication, database queries, real-time
- **Stripe:** Payment processing, webhooks, subscriptions
- **Resend:** Email API and templates
- **Vercel AI SDK:** Text generation, streaming, embeddings
- **And many more...**

## Instructions

### 1. Install Context7 MCP Globally

Open your terminal and install Context7:

```bash
npx -y @upstash/context7-mcp@latest
```

**What this does:**
- Installs the Context7 MCP server
- Makes it available globally on your system
- Enables integration with Cursor IDE

### 2. Configure Cursor MCP Settings

#### Method A: Through Cursor Settings UI

1. **Open Cursor Settings:** `Cmd/Ctrl + ,`
2. **Navigate to MCP:** Look for "MCP" or "Model Context Protocol"
3. **Add New Server:** Click "Add new global MCP server"
4. **Configure Context7:**
   ```json
   {
     "name": "context7",
     "command": "npx",
     "args": ["-y", "@upstash/context7-mcp@latest"]
   }
   ```

#### Method B: Manual Configuration

Create/edit `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### 3. Verify Installation

1. **Restart Cursor** completely
2. **Check Settings:** Go to Settings → MCP
3. **Verify Context7:** Should appear in the list of MCP servers
4. **Test Connection:** The server should show as "Connected" or "Active"

### 4. Learn How to Use Context7

#### Basic Usage Pattern

Add "use context7" to your prompts when you need official documentation:

```
Create a Stripe checkout session for SkyMarket bookings (use context7)
```

#### Advanced Usage Patterns

**For Multiple APIs:**
```
Scan our code and pull the exact docs for these used APIs (use context7):
- Next.js App Router route handlers
- Supabase SSR auth for Next.js  
- Stripe Checkout sessions + webhook signature verification
- Resend Node send API
- Vercel AI SDK generateText/streamText/embed
Return links and 3-line usage reminders for each.
```

**For Specific Tasks:**
```
For the task "wire checkout then send email on success", only use:
@app/api/orders/[id]/checkout/route.ts and @app/api/webhooks/stripe/route.ts.
Pull current Stripe docs (use context7) and validate our webhook logic against them; propose deltas.
```

## Practical Examples for SkyMarket

### 1. Database Operations with Supabase

**Cursor Prompt:**
```
I need to create a booking system with real-time updates. Show me the latest patterns for:
- Supabase real-time subscriptions in Next.js
- Row Level Security policies for multi-tenant data
- TypeScript types generation from database schema
Use context7 to get the most current Supabase documentation.
```

### 2. Payment Processing with Stripe

**Cursor Prompt:**
```
Help me implement a secure payment flow for drone service bookings. I need:
- Stripe Payment Intents for holding funds in escrow
- Webhook handling for payment confirmations  
- Error handling for failed payments
Use context7 to reference the latest Stripe documentation and best practices.
```

### 3. Email Notifications with Resend

**Cursor Prompt:**
```
Create an email notification system for booking confirmations. Show me how to:
- Send transactional emails with Resend
- Use email templates for booking confirmations
- Handle email delivery failures
Use context7 to get current Resend API documentation.
```

### 4. AI Features with Vercel AI SDK

**Cursor Prompt:**
```
I want to add AI-powered service recommendations. Help me implement:
- Text generation for service descriptions
- Streaming responses for chat features
- Embedding generation for semantic search
Use context7 to reference the latest Vercel AI SDK patterns.
```

## Expected Outcome

After completing this step, you should have:

✅ **Context7 MCP Installed:**
- MCP server installed globally
- Cursor configured to use Context7
- Connection verified in Cursor settings

✅ **Enhanced Development Workflow:**
- Ability to access real-time documentation
- Context-aware AI assistance
- Reduced time searching for API references

✅ **Practical Experience:**
- Completed at least one task using "use context7"
- Received cited links and current documentation
- Improved code quality with authoritative references

✅ **Integration Ready:**
- Ready to use Context7 for Stripe implementation
- Prepared for email integration with Resend
- Set up for advanced AI features

## Best Practices

### Effective Prompting with Context7

#### ✅ Do's
```
✅ GOOD: "Create a Stripe webhook handler (use context7)"
✅ GOOD: "Show me Next.js middleware patterns for auth (use context7)"
✅ GOOD: "Validate our Supabase RLS policies against current docs (use context7)"
```

#### ❌ Don'ts
```
❌ AVOID: "How do I code?" (too vague)
❌ AVOID: "Fix my app" (no specific context)
❌ AVOID: Using context7 for general programming questions
```

### When to Use Context7

**Perfect for:**
- API implementation questions
- Framework-specific patterns
- Configuration and setup issues
- Best practices for specific libraries
- Version compatibility questions

**Not ideal for:**
- General programming concepts
- Language syntax questions
- Algorithm design
- Business logic decisions

## Troubleshooting Tips

### Common Issues and Solutions

**❌ Context7 not appearing in Cursor settings**
```bash
# Reinstall Context7 MCP
npx -y @upstash/context7-mcp@latest

# Restart Cursor completely
# Check ~/.cursor/mcp.json exists and is valid JSON
```

**❌ "MCP server failed to start"**
```bash
# Check Node.js installation
node --version

# Verify npx is working
npx --version

# Try manual installation
npm install -g @upstash/context7-mcp
```

**❌ Context7 not responding to prompts**
```bash
# Ensure you include "use context7" in your prompt
# Check MCP server status in Cursor settings
# Try restarting the MCP server from settings
```

**❌ Outdated or incorrect documentation**
```bash
# Update Context7 to latest version
npx -y @upstash/context7-mcp@latest

# Clear any cached data by restarting Cursor
```

### Testing Your Setup

**Quick Test Prompt:**
```
Show me the current Next.js App Router API route structure (use context7). Include TypeScript examples and error handling patterns.
```

**Expected Response:**
- Current Next.js documentation references
- Cited links to official docs
- Code examples from official sources
- Up-to-date patterns and best practices

## Advanced Usage Patterns

### 1. Code Validation Against Documentation

```
Review our authentication middleware at @middleware.ts against current Next.js and Supabase documentation (use context7). Identify any deprecated patterns or missing best practices.
```

### 2. Migration Guidance

```
We're upgrading from Next.js 14 to 15. Use context7 to identify breaking changes and migration steps for our current codebase structure.
```

### 3. Performance Optimization

```
Analyze our API routes for performance bottlenecks. Use context7 to reference current Vercel and Next.js performance best practices.
```

### 4. Security Review

```
Review our payment handling code against latest Stripe security guidelines (use context7). Focus on webhook signature verification and PCI compliance.
```

## Integration with SkyMarket Development

### Typical Development Workflow

1. **Identify Task:** Need to implement a feature
2. **Context Research:** Use Context7 for current documentation  
3. **Code Generation:** Generate code with authoritative references
4. **Validation:** Verify implementation against official patterns
5. **Testing:** Test with official examples and patterns

### Example: Implementing Real-time Booking Updates

```
I need to implement real-time booking status updates for SkyMarket. The flow should be:
1. Customer books a service
2. Provider accepts/declines
3. Both parties see live updates
4. Status changes trigger notifications

Use context7 to show me the current Supabase real-time subscription patterns for Next.js. Include TypeScript types and error handling.
```

## Next Steps

With Context7 configured, you're ready to:
- **Step 13:** Implement Stripe checkout with authoritative documentation
- **Step 14:** Set up Resend email integration using current API patterns
- **Step 15:** Add advanced features with confidence using official references

## References

### Official Documentation
- [Model Context Protocol](https://modelcontextprotocol.io) - MCP standard specification
- [Context7 MCP](https://github.com/upstash/context7) - Context7 implementation and usage
- [Cursor MCP Integration](https://cursor.sh/docs/mcp) - Cursor-specific MCP setup

### Supported Libraries
- **Next.js:** App Router, API routes, middleware, server components
- **Supabase:** Database, auth, real-time, storage
- **Stripe:** Payments, webhooks, subscriptions, Connect
- **Resend:** Email API, templates, domains
- **Vercel AI SDK:** Text generation, streaming, embeddings, tools

---

💡 **Pro Tip:** Always include "use context7" in prompts when working with external APIs or frameworks. This ensures you're getting the most current, official documentation rather than outdated examples or patterns.