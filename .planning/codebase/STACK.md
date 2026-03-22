# Technology Stack

**Analysis Date:** 2026-03-22

## Languages

**Primary:**
- TypeScript 5 - Application code, API routes, server components, client components
- JavaScript - Build scripts, configuration files

**Secondary:**
- HTML/CSS - DOM structure and styling

## Runtime

**Environment:**
- Node.js (latest LTS compatible) - Server-side execution, build processes, scripts
- Browsers (modern) - Client-side React execution

**Package Manager:**
- npm - Dependency management
- Lockfile: `package-lock.json` present

## Frameworks

**Core:**
- Next.js 16.1.6 - Full-stack React framework with App Router, API routes, SSR/SSG
- React 19.2.3 - UI component library
- React DOM 19.2.3 - React DOM rendering

**Styling:**
- Tailwind CSS 3.4.4 - Utility-first CSS framework
- PostCSS 8.5.6 - CSS processing pipeline
- Autoprefixer 10.4.24 - Vendor prefix generation

**Desktop/Electron:**
- Electron 33.2.0 - Desktop application framework
- electron-builder 25.1.8 - Electron packaging and distribution

**Build & Dev:**
- TypeScript Compiler (tsc) - Type checking
- ESLint 9 - Linting and code quality
- eslint-config-next 16.1.6 - Next.js ESLint configuration

## Key Dependencies

**Critical:**
- @supabase/supabase-js 2.94.1 - PostgreSQL database client and auth management
- @supabase/ssr 0.8.0 - Supabase SSR utilities for Next.js cookie handling
- stripe 20.4.0 - Stripe payment processing SDK
- resend 4.8.0 - Email delivery service (contact form)
- @vercel/speed-insights 1.3.1 - Performance monitoring

**Data Processing:**
- xlsx 0.18.5 - Excel file parsing for data import/sync
- pdf-parse 2.4.5 - PDF text extraction from documents

**Environment:**
- dotenv 17.2.3 - Environment variable loading

**Type Definitions:**
- @types/node 20 - Node.js type definitions
- @types/react 19 - React type definitions
- @types/react-dom 19 - React DOM type definitions

## Configuration

**Environment:**
- `.env.local` - Local development environment variables (not committed)
- `.env.example` - Template for required environment variables
- Variables required: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
- Optional: `RESEND_API_KEY`, `CONTACT_EMAIL`, `STRIPE_SECRET_KEY`, `STRIPE_PRICE_ID`, `NEXT_PUBLIC_VERCEL_URL`

**Build:**
- `tsconfig.json` - TypeScript configuration (ES2017 target, strict mode, baseUrl ".", path alias "@/*")
- `next.config.ts` - Next.js configuration with Supabase URL/key env variable injection
- `eslint.config.mjs` - ESLint flat config (Next.js web vitals + TypeScript)
- `postcss.config.js` - PostCSS pipeline configuration
- `tailwind.config.js` - Tailwind CSS configuration
- `electron-builder.json` - Electron app packaging configuration

**Git Hooks:**
- `.githooks/` directory - Custom git hooks
- `scripts/install-git-hooks.mjs` - Hook installation script

## Platform Requirements

**Development:**
- Node.js 18+ (inferred from TypeScript 5 + Next.js 16 compatibility)
- npm 8+
- Windows/macOS/Linux (electron-builder supports all)

**Production:**
- Vercel (native Next.js support, referenced in env vars via `NEXT_PUBLIC_VERCEL_URL`)
- Alternative: Any Node.js-compatible server (requires `next start`)
- Desktop: Standalone Electron binary (Windows, macOS, Linux)

**Database:**
- PostgreSQL (via Supabase)

---

*Stack analysis: 2026-03-22*
