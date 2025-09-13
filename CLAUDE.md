# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **skymarket-instructions**, a documentation repository for SkyMarket - a multi-modal drone service marketplace serving the Detroit metropolitan area. The repository contains comprehensive documentation, planning guides, and implementation tracks for the SkyMarket platform.

**Note**: This is a documentation-only repository. The actual application code is located in a separate `skymarket-supabase` repository.

## Repository Structure

```
skymarket-instructions/
├── README.md                    # Main project overview
├── PRD.md                      # Product Requirements Document
├── instructions/               # Build and planning guides
│   ├── BUILD-GUIDE.md
│   ├── PLANNING-GUIDE.md
│   └── CURSOR-PREREQUISITES.md
├── development/                # Development documentation
│   ├── SETUP.md               # Complete setup instructions
│   ├── WORKFLOW.md            # Git workflow and development processes
│   ├── DB_RESET.md
│   └── INSTALLATION.md
├── tracks/developer/           # Step-by-step implementation tracks
│   ├── README.md              # Track overview
│   ├── prerequisites.md       # Prerequisites for development
│   ├── resources.md           # Resource links
│   └── steps/                 # Individual implementation steps
└── architecture/               # System architecture documentation
```

## Tech Stack (for actual application)

- **Frontend**: Next.js 14, TypeScript, Tailwind CSS 3, shadcn/ui
- **Backend**: Supabase (PostgreSQL, Auth, Realtime, Storage)
- **Payments**: Stripe
- **Maps**: Mapbox
- **Deployment**: Vercel

## Key Documentation Files

### Essential Reading
- **PRD.md**: Complete product requirements and technical specifications
- **development/SETUP.md**: Complete development environment setup
- **development/WORKFLOW.md**: Git workflow, branch strategy, and development processes
- **tracks/developer/README.md**: Implementation track overview

### Setup & Development
- **development/INSTALLATION.md**: Step-by-step installation process
- **instructions/BUILD-GUIDE.md**: Build instructions and deployment
- **instructions/CURSOR-PREREQUISITES.md**: Cursor IDE prerequisites

## Development Workflow (from WORKFLOW.md)

### Commands for actual application
```bash
# Development
npm run dev              # Start development server
npm run build           # Production build
npm run lint            # ESLint checks
npm run type-check      # TypeScript validation
npm run test            # Run tests

# Git workflow
git checkout develop    # Switch to develop branch
git checkout -b feature/feature-name  # Create feature branch
git commit -m "feat: description"     # Conventional commits
```

### Branch Strategy
- `main`: Production-ready code
- `develop`: Integration branch for features
- `feature/*`: Feature development branches
- `fix/*`: Bug fix branches
- `hotfix/*`: Critical production fixes

## Service Categories

SkyMarket serves four main categories:
1. **Food Delivery**: Restaurant meals, groceries ($2.99-$7.99 delivery fee)
2. **Courier Services**: Documents, packages ($4.99-$12.99 base)
3. **Aerial Imaging**: Real estate, events ($149-$499/hour)
4. **Site Mapping**: Construction, surveying ($299-$799/project)

## Key Implementation Phases

1. **Phase 1**: Foundation (Next.js + Supabase setup)
2. **Phase 2**: User Management (auth, profiles, roles)
3. **Phase 3**: Provider System (onboarding, verification)
4. **Phase 4**: Marketplace Core (discovery, search)
5. **Phase 5**: Booking System (scheduling, confirmations)
6. **Phase 6**: Payment Processing (Stripe integration)
7. **Phase 7**: Real-time Features (tracking, messaging)
8. **Phase 8**: Reviews & Trust (ratings, verification)

## Compliance & Regulations

- **FAA Part 107**: Certificate verification required for drone operators
- **Detroit Metro Focus**: Local regulations and service areas
- **Insurance Requirements**: $1M liability minimum
- **Food Safety**: Temperature control, hygiene protocols

## Important Notes

- This repository contains **documentation only**
- Actual application development happens in `skymarket-supabase` repository
- Follow conventional commits: `type(scope): description`
- All features require comprehensive testing and documentation
- Security-first approach with Supabase RLS policies
- Mobile-first responsive design principles