# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Critical Context

**This is a documentation-only repository for SkyMarket** - a drone service marketplace. The actual application code lives in `skymarket-supabase`.

## Essential Documentation

- **PRD.md**: Complete product requirements and technical specifications
- **development/SETUP.md**: Complete development environment setup
- **development/WORKFLOW.md**: Git workflow and development processes

## Guiding Principles

### Development Standards
- Follow conventional commits: `type(scope): description`
- Use feature branches: `git checkout -b feature/feature-name`
- Security-first approach with Supabase RLS policies
- Mobile-first responsive design
- Comprehensive testing and documentation required

### Architecture Philosophy
- **Stack**: Next.js 14 + TypeScript + Supabase + Stripe
- **Phase-driven development**: 8 phases from Foundation to Reviews & Trust
- **Compliance-first**: FAA Part 107, insurance requirements, food safety protocols
- **Multi-modal services**: Food delivery, courier, aerial imaging, site mapping

### Key Rules
1. **Documentation drives development** - always read PRD.md first
2. **Phase-based implementation** - follow the 8-phase roadmap
3. **Regulatory compliance** - drone operations require FAA Part 107 certification
4. **Security by design** - implement Supabase RLS from day one
5. **Detroit Metro focus** - local regulations and service areas matter