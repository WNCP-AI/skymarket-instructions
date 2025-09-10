# SkyMarket Documentation

Welcome to the SkyMarket drone service marketplace documentation. This documentation provides comprehensive information about the project, from setup to deployment.

## 📋 Table of Contents

### Getting Started
- [Setup Instructions](./development/SETUP.md) - How to set up the development environment
- [Installation Guide](./development/INSTALLATION.md) - Step-by-step installation process
- [Environment Configuration](./development/ENVIRONMENT.md) - Environment variables and configuration

### Project Overview
- [Product Requirements Document](./PRD.md) - Complete product requirements and specifications
- [Technical Architecture](./architecture/ARCHITECTURE.md) - System architecture and design decisions
- [Database Schema](./architecture/DATABASE.md) - Database design and relationships
- [API Documentation](./api/API.md) - API endpoints and usage

### Development
- [Development Workflow](./development/WORKFLOW.md) - Git workflow and development processes
- [Component Guide](./development/COMPONENTS.md) - UI component documentation
- [Testing Strategy](./development/TESTING.md) - Testing approach and guidelines
- [Deployment Guide](./development/DEPLOYMENT.md) - Production deployment instructions

### Phase Implementation
- [Phase 1: Foundation](./development/PHASE_1.md) - Project setup and core infrastructure
- [Phase 2: Authentication](./development/PHASE_2.md) - User authentication and profiles
- [Phase 3: UI Components](./development/PHASE_3.md) - Component library and design system
- [Phase 4: Provider Management](./development/PHASE_4.md) - Provider onboarding and listings

## 🚀 Quick Start

1. **Clone and Setup**
   ```bash
   git clone <repository-url>
   cd skymarket-subpabase
   npm install
   ```

2. **Configure Environment**
   ```bash
   cp .env.example .env.local
   # Edit .env.local with your API keys
   ```

3. **Set up Supabase**
   - Create a new Supabase project
   - Run the SQL schema from `supabase/schema.sql`
   - Update environment variables

4. **Start Development**
   ```bash
   npm run dev
   ```

## 🏗️ Project Structure

```
skymarket-subpabase/
├── app/                    # Next.js App Router pages
├── components/            # React components
│   ├── ui/               # shadcn/ui components
│   ├── layout/           # Layout components
│   ├── providers/        # Provider-specific components
│   └── consumers/        # Consumer-specific components
├── lib/                   # Utility libraries
│   ├── supabase/         # Supabase client and helpers
│   ├── stripe/           # Stripe integration
│   └── mapbox/           # Mapbox utilities
├── types/                 # TypeScript type definitions
├── hooks/                 # Custom React hooks
├── utils/                 # Utility functions
├── docs/                  # Project documentation
└── supabase/             # Database schema and migrations
```

## 🛠️ Tech Stack

- **Frontend**: Next.js 14, TypeScript, Tailwind CSS 3, shadcn/ui
- **Backend**: Supabase (PostgreSQL, Auth, Realtime, Storage)
- **Payments**: Stripe
- **Maps**: Mapbox
- **Deployment**: Vercel

## 📱 Target Market

SkyMarket serves the **Detroit Metropolitan Area** with drone services including:
- 🍔 Food Delivery
- 📦 Courier Services  
- 📸 Aerial Imaging
- 🗺️ Site Mapping

## 🤝 Contributing

Please read our [Development Workflow](./development/WORKFLOW.md) for information on how to contribute to this project.

## 📄 License

This project is proprietary software developed for ID Ventures / WNCP AI.

---

For specific setup instructions, see [SETUP.md](./development/SETUP.md)