# Step 4: MCP Context7 Setup

## Objective

Install and configure MCP Context7 server to provide Cursor IDE with access to official library documentation, enabling current best practices and version-aware code generation.

## What is MCP Context7?

MCP Context7 is a Model Context Protocol server that provides real-time access to official library documentation. Instead of relying on AI training data that might be outdated, Context7 fetches current documentation directly from library maintainers.

**Key Insight**: Context7 works through **natural language only**. You simply add "use context7" to any coding request - no commands, no special syntax.

### Why Context7 for Spec-Driven Development?

- **Current Documentation**: Always up-to-date with latest versions from official sources
- **Official Patterns**: Directly from library maintainers, not AI training data
- **Version Aware**: Specific to your project's exact dependencies
- **Best Practices**: Recommended approaches from official documentation
- **Spec Integration**: Combines perfectly with project specifications for optimal development

## Understanding MCP Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Cursor IDE    │◄──►│  MCP Context7   │◄──►│ Library Docs    │
│   (Client)      │    │   (Server)      │    │ (Remote APIs)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

- **Cursor IDE**: Your development environment (MCP Client)
- **MCP Context7**: Documentation server (MCP Server)
- **Library Docs**: Official documentation sources

## Installation

### 1. Get Context7 API Key (Optional)

For higher rate limits, create a free account:
1. Visit [context7.com/dashboard](https://context7.com/dashboard)
2. Sign up and get your API key
3. Keep this key for configuration below

**Note**: Context7 works without an API key, but with lower rate limits.

### 2. Verify Node.js Compatibility

Context7 requires Node.js 18+:
```bash
node --version  # Should show v18.0.0 or higher
```

If you need to upgrade Node.js:
```bash
# Using nvm
nvm install 18
nvm use 18
nvm alias default 18
```

## Cursor IDE Configuration

### 1. Install Context7 MCP Server

**Recommended Method: One-Click Installation**

[![Install MCP Server](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=context7&config=eyJjb21tYW5kIjoibnB4IC15IEB1cHN0YXNoL2NvbnRleHQ3LW1jcCJ9)

Click the button above for instant installation in Cursor.

**Manual Method: Configuration File**

Create or update `~/.cursor/mcp.json` file:

**With API Key (Recommended)**:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"]
    }
  }
}
```

**Without API Key**:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

**Alternative: Remote Server Connection**
```json
{
  "mcpServers": {
    "context7": {
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

### 2. Restart Cursor IDE

After configuration:
1. **Completely quit Cursor** (not just close windows)
2. **Restart Cursor IDE**
3. **Wait for MCP initialization** (may take 30-60 seconds first time)

### 3. Verify MCP Connection

Check that Context7 is connected:
1. Open Cursor chat (`Cmd+L`/`Ctrl+L`)
2. Look for MCP status indicator (usually in chat interface)
3. You should see `context7` listed as available

## Testing Context7 Integration

### 1. Basic Connection Test

In Cursor chat (`Cmd+L`/`Ctrl+L`), test Context7 with natural language prompts:

**Test 1: React Hooks**
```
Create a React component that uses useState and useEffect hooks with proper TypeScript types. use context7
```

**Expected**: AI will generate current React code with up-to-date hook patterns from official React documentation.

**Test 2: Next.js App Router**
```
Show me how to create a dynamic route in Next.js 15 with proper loading and error handling using the App Router. use context7
```

**Expected**: AI will provide current Next.js 15 App Router patterns from official Next.js docs.

### 2. SkyMarket Stack Test with Specifications

Test Context7 with SkyMarket-specific implementations using project specifications:

**Supabase Authentication for Marketplace:**
```
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@AGENTS.md

Create complete Supabase authentication system for SkyMarket drone marketplace:
- Multi-role signup (consumers, providers, admins) with role-specific onboarding
- Email verification with custom templates for marketplace context
- Session management with Next.js 15 App Router middleware
- Row Level Security policies for marketplace data isolation
- Provider verification status tracking integration

use context7
```

**Stripe Payment Infrastructure:**
```
@docs/specs/payment/PAYMENT.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@AGENTS.md

Implement comprehensive Stripe payment infrastructure for SkyMarket:
- Secure payment processing with multiple payment methods
- Payment intent creation with dynamic pricing
- Webhook endpoints for payment_intent.succeeded and status updates
- Secure transaction handling until booking completion
- Provider dashboard integration with earnings tracking

use context7
```

**shadcn/ui Components for Marketplace:**
```
@docs/specs/components/COMPONENTS.md
@AGENTS.md

Create advanced booking management components for SkyMarket using shadcn/ui:
- Multi-step booking wizard with progress indicators
- Provider service cards with availability and pricing
- Real-time booking status updates with optimistic UI
- Responsive design optimized for mobile and desktop
- Accessibility compliance following WCAG guidelines

use context7
```

**Complex Form Handling for Bookings:**
```
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/api/API.md
@AGENTS.md

Create sophisticated booking form for SkyMarket using React Hook Form and Zod:
- Multi-step wizard with conditional validation
- Geographic validation for Detroit Metro bounds (42.0-42.6 lat, -83.5--82.8 lng)
- Date/time selection with provider availability checking
- Service category selection (food_delivery, courier, aerial_imaging, site_mapping)
- Real-time price calculation with distance-based factors

use context7
```

Each prompt should return current, official documentation-based code with proper TypeScript integration.

## Using Context7 Effectively

### Simple "use context7" Syntax

Context7 works by adding `use context7` to any natural language prompt. No special commands needed!

### Effective Prompt Patterns

**Basic Pattern:**
```
[What you want to build] + use context7
```

**Library-Specific Pattern:**
```
[Specific task with library name] + use context7
```

**Complex Integration Pattern:**
```
[Multi-library task] + use context7
```

### Practical Examples

**Authentication Development:**
```
Build a complete Supabase authentication system with signup, login, and email verification for Next.js 15. use context7
```

**Payment Integration:**
```
Create a Stripe payment processing flow with secure transactions and webhook handling. use context7
```

**UI Development:**
```
Build a responsive dashboard layout with Tailwind CSS and shadcn/ui components. use context7
```

## Integration with Spec-Driven Development

### The Power Combination

Context7 + Specifications + AI = Optimal Development

**Enhanced Workflow**:
1. **Load Project Specs**: `@docs/specs/authentication/AUTHENTICATION.md`
2. **Write Natural Prompt**: Include "use context7" for official documentation
3. **Generate Code**: AI combines specifications with current official docs
4. **Verify Implementation**: Check against both specs and latest patterns

### Example: Complete SkyMarket Payment Infrastructure

**Production-Ready Spec-Driven Prompt**:
```
@docs/specs/payment/PAYMENT.md
@docs/specs/business-logic/BUSINESS-LOGIC.md
@docs/specs/authentication/AUTHENTICATION.md
@docs/specs/architecture/DATABASE.md
@AGENTS.md

Implement complete Stripe payment infrastructure for SkyMarket drone service platform:

**Core Payment System:**
- Secure payment processing with comprehensive verification
- Payment intent creation with dynamic pricing calculation
- Webhook endpoints processing payment_intent.succeeded and status events
- Consumer checkout flow with secure authentication and multiple payment methods

**Provider Integration:**
- Provider earnings dashboard with transaction history tracking
- Payment analytics with detailed reporting features
- Transaction status monitoring and management
- Integration with existing Supabase providers table and authentication system

**Business Logic Integration:**
- Automatic payment confirmation triggered by booking completion (status: completed)
- Geographic validation for Detroit Metro service area compliance
- Service category-specific pricing handling for 4 drone service types
- Integration with booking state machine and real-time status updates

**Technical Requirements:**
- Next.js 15 App Router with TypeScript for all payment endpoints
- Comprehensive error handling for all Stripe API operations
- Database transaction handling for payment and booking status synchronization
- Real-time UI updates using Supabase subscriptions

Use current Stripe payment processing patterns, Next.js 15 best practices, and Supabase integration guidelines.

use context7
```

This powerful combination of:
1. **Project Specifications** (business requirements)
2. **Context7 Documentation** (current technical patterns)
3. **AI Implementation** (code generation)

Produces production-ready marketplace payment systems that match both business needs and current best practices.

## Troubleshooting

### Common Installation Issues

**Problem**: Context7 not responding to "use context7" prompts
**Solution**:
1. Verify MCP server is running in Cursor settings
2. Check Node.js version is 18+ with `node --version`
3. Restart Cursor completely (quit and reopen)
4. Test with simple prompt: "Create a React component. use context7"

**Problem**: MCP server connection fails
**Solution**:
```bash
# Test server manually
npx -y @upstash/context7-mcp

# Check Node.js version
node --version  # Must be 18+

# Clear npm cache if needed
npm cache clean --force
```

### Connection Issues

**Problem**: Context7 not appearing in Cursor
**Solution**:
1. Verify settings.json configuration
2. Completely restart Cursor (quit and reopen)
3. Check Cursor logs for MCP errors
4. Try manual server test

**Problem**: Commands timeout or fail
**Solution**:
1. Check internet connection
2. Verify firewall isn't blocking npx
3. Try different library names
4. Wait for server initialization (can take 60s first time)

### Documentation Access Issues

**Problem**: Context7 doesn't recognize library in prompt
**Solution**:
- Be specific about library names in prompts: "Create a Stripe payment processing setup. use context7"
- Include version numbers when relevant: "Implement Next.js 15 App Router patterns. use context7"
- Try alternative library names: "Supabase" vs "supabase-js"
- Not all libraries have Context7 documentation coverage

**Problem**: Generic or outdated responses despite Context7
**Solution**:
- Context7 serves current official documentation when available
- Be more specific in prompts about what you want to implement
- Some libraries may have limited Context7 coverage - fallback to official docs

## Expected Outcome

After completing this step, you should have:

### Working MCP Context7
- [ ] Context7 server installed and running
- [ ] Cursor IDE configured to use Context7
- [ ] MCP connection verified in chat interface
- [ ] Can resolve library IDs successfully

### Documentation Access
- [ ] "use context7" works in natural prompts
- [ ] Can generate current React patterns
- [ ] Can access Next.js 15 App Router documentation
- [ ] Can get current Supabase integration patterns
- [ ] Can access Stripe payment processing examples

### Spec-Driven Integration
- [ ] Understand how to combine specs + Context7 + AI
- [ ] Can write effective prompts using both sources
- [ ] Ready to build features with current best practices

## Pro Tips

### Effective Context7 Usage

**Natural Language Integration**:
- Simply add "use context7" to any development prompt - that's it!
- No commands, no special syntax - just natural conversation + "use context7"
- Be specific about the library, version, and implementation details

**SkyMarket-Specific Patterns**:
- **Stripe Payments**: "Implement Stripe payment processing with secure transactions and variable pricing. use context7"
- **Supabase + Next.js**: "Create Supabase authentication middleware for Next.js 15 App Router with TypeScript types. use context7"
- **React Hook Form**: "Build a drone service booking form with React Hook Form, Zod validation, and date/time selection. use context7"

**Spec-Driven Integration Strategy**:
- Always combine project specs with Context7: `@docs/specs/payment/PAYMENT.md` + prompt + "use context7"
- Verify generated code matches both specifications and current patterns
- When Context7 conflicts with specs, explicitly prioritize specs in prompts
- Use Context7 for implementation patterns, specs for business requirements

### Performance Optimization

**Context Management**:
- Context7 responses can be comprehensive - manage context carefully
- Use specific, focused prompts to get targeted information
- Clear chat context periodically to maintain performance

**Selective Usage**:
- Only use Context7 for features you're actively implementing
- Be specific about what you need rather than asking broad questions
- Prioritize official Context7 patterns over outdated AI knowledge

## Next Steps

Excellent! MCP Context7 is configured and ready to provide official documentation. Next, we'll master context management to effectively combine specifications with official docs.

### What's Coming in Step 5
- Master Cursor's @ mention system
- Learn to combine specs with Context7 docs
- Understand context window management
- Practice effective prompt engineering

---

**Next Step**: [Step 5: Context Management](./05-context-management.md) →

**Quick Links**:
- [Previous: Cursor Installation](./03-cursor-install.md)
- [Track Overview](../README.md)
- [Project Specifications](../../../specs/)