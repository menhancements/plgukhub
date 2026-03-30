# CLAUDE.md — PLG UK Hub

## Project Overview

PLG UK Hub is a **Next.js digital forms platform** for clinical practices and service businesses. It provides secure, GDPR-compliant form management with e-signatures, audit logging, multi-brand support, and role-based access control.

**Tech stack:** Next.js 16 (App Router) · React 19 · TypeScript · Prisma 7 (PostgreSQL) · Tailwind CSS 4 · NextAuth.js

## Quick Reference

```bash
# Development
npm run dev              # Start dev server (localhost:3000)
npm run build            # Production build
npm run lint             # Run ESLint

# Database
npm run db:generate      # Generate Prisma client
npm run db:push          # Push schema to database
npm run db:migrate       # Create migration
npm run db:seed          # Seed demo data
npm run db:studio        # Open Prisma Studio

# Infrastructure
npm run docker:up        # Start PostgreSQL, Redis, MinIO
npm run docker:down      # Stop Docker services
npm run setup            # Full setup (docker + db push + seed)
```

## Project Structure

```
src/
├── app/                        # Next.js App Router pages & API routes
│   ├── layout.tsx              # Root layout (Geist fonts)
│   ├── page.tsx                # Landing page
│   ├── login/page.tsx          # Login (client component)
│   ├── admin/page.tsx          # Admin dashboard (server component)
│   └── api/auth/[...nextauth]/ # NextAuth API route
├── components/
│   ├── signature/SignaturePad.tsx  # E-signature component
│   └── ui/                     # Radix UI-based components (button, input, label)
└── lib/
    ├── auth/index.ts           # NextAuth config + RBAC helpers
    ├── db/index.ts             # Prisma client singleton
    ├── forms/
    │   ├── types.ts            # Form type definitions & enums
    │   ├── registry.ts         # Form registry singleton
    │   ├── validation.ts       # Conditional logic & stop conditions
    │   └── definitions/        # Brand-specific form definitions (~8K lines)
    │       ├── menhancements.ts
    │       ├── plg-uk.ts
    │       ├── wax-for-men.ts
    │       └── wax-for-women.ts
    ├── actions/submissions.ts  # Server actions for form submission
    ├── audit/index.ts          # Immutable audit logging
    ├── exports/csv.ts          # CSV export with filtering
    ├── pdf/index.ts            # PDF generation (PDFKit)
    ├── search/index.ts         # Search & dashboard stats
    └── utils.ts                # Formatting & utility functions

prisma/
├── schema.prisma               # Database schema (11 tables, 5 enums)
└── seed.ts                     # Demo data seeder
docs/                           # Detailed guides (setup, forms, roles, retention)
docker-compose.yml              # Local dev services (Postgres, Redis, MinIO)
```

## Architecture & Key Patterns

### Form System Pipeline

Form definitions flow through: **Definition** (`types.ts`) → **Registry** (`registry.ts`) → **Validation** (`validation.ts`) → **Submission** (`submissions.ts`) → **Audit** (`audit/index.ts`)

- Forms are declarative JSON-like structures defined in `lib/forms/definitions/`
- The registry is a singleton managing the full form library
- Validation evaluates conditional visibility, required fields, and stop/escalation logic
- 20+ field types: text, select, signature, medicationList, allergyList, rating, etc.

### Authentication & RBAC

- NextAuth.js with Credentials provider, JWT sessions (24h), bcryptjs passwords
- Three roles: `ADMIN`, `PRACTITIONER`, `RECEPTION`
- Four brands: `MENHANCEMENTS`, `WAX_FOR_MEN`, `WAX_FOR_WOMEN`, `PLG_UK`
- Permission helpers in `lib/auth/index.ts`: `canAccessBrand()`, `canManageUsers()`, `canViewAllSubmissions()`, `canExportData()`, `canAmendSubmission()`

### Database (Prisma)

- 11 tables: Site, User, Session, Client, FormSubmission, Amendment, AuditLog, FileUpload, DataRetentionPolicy, SubjectAccessRequest
- Key enums: Brand, Role, SubmissionStatus (DRAFT/SUBMITTED/SIGNED/LOCKED/AMENDED), RiskLevel
- Singleton Prisma client in `lib/db/index.ts`

### Server Components vs Client Components

- Pages under `app/` are server components by default (e.g., `admin/page.tsx`)
- Client components use the `'use client'` directive (e.g., `login/page.tsx`, `SignaturePad.tsx`)
- Server actions use `'use server'` directive for form submission logic

## Naming Conventions

| Entity | Convention | Example |
|--------|-----------|---------|
| Form IDs | `brand_form_name` | `menh_patient_registration` |
| Field IDs | `snake_case` | `date_of_birth` |
| Section IDs | `descriptive_name` | `personal_details` |
| Component files | `PascalCase.tsx` | `SignaturePad.tsx` |
| Lib/utility files | `camelCase.ts` | `registry.ts` |
| Database enums | `UPPER_SNAKE_CASE` | `WAX_FOR_MEN` |

## Environment Variables

Required in `.env` (gitignored):

```
DATABASE_URL="postgresql://..."
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="<openssl rand -base64 32>"
STORAGE_TYPE="local"          # or "s3"
STORAGE_LOCAL_PATH="./uploads"
```

## Important Context for AI Assistants

- **No test framework** is configured — there are no test files, Jest, or Vitest
- **No CI/CD pipeline** exists — no GitHub Actions or equivalent
- **No custom git hooks** — only sample hooks in `.git/hooks/`
- **Path alias:** `@/*` maps to `src/*` (configured in `tsconfig.json`)
- **TypeScript strict mode** is enabled
- Form definitions are large files (1K-3K+ lines each) — they are declarative data, not logic
- The `docs/` directory contains detailed guides for adding forms, roles/permissions, data retention, and setup
- Audit logging is immutable and tracks all data operations for GDPR compliance
- The PDF module generates signed documents with submission metadata
- Docker Compose provides local Postgres (port 5432), Redis (6379), and MinIO (9000/9001)
- Demo accounts after seeding: admin@plgukhub.com / practitioner@plgukhub.com / reception@plgukhub.com (password: `password123`)
