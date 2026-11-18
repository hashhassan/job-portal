# Architecture: Mini Job Board

Implements [PRD.md](PRD.md). Rules from CLAUDE.md apply: Server Components read, Server Actions mutate, and every protected page and action checks the session and role on the server.

## 1. Tech stack

All versions are pinned exactly (no `^`/`~`). Prisma is pinned to **7.10.1**: npm's `latest` tag is now Prisma 8, so never install it unversioned.

| Layer | Package | Version |
|---|---|---|
| Framework | next (App Router, Turbopack) | 16.4.6 |
| UI runtime | react, react-dom | 19.3.0 |
| Language | typescript | 5.9.3 (stay on 5.x) |
| Styling | tailwindcss, @tailwindcss/turbopack | 4.3.3 |
| Components | shadcn CLI (generates into `src/components/ui`) | 4.23.0 |
| ORM | prisma (dev), @prisma/client | 7.10.1 |
| DB driver | @prisma/adapter-neon | 7.10.1 |
| Database | Neon Postgres (`dev` and `main` branches) | n/a |
| Auth | better-auth (self-hosted, email + password, Prisma adapter) | 1.7.25 |
| Validation | zod | 4.6.7 |
| Dev tools | dotenv (loads `.env` in `prisma.config.ts`), tsx (runs the seed) | pinned at install |
| Hosting | Vercel (Node.js 22) | n/a |

shadcn adds its standard dependencies when initialised (class-variance-authority, clsx, tailwind-merge, lucide-react, Radix packages). Nothing else is added without asking.

> Next.js 16 note: `middleware.ts` is now `proxy.ts`. We don't use either (see §6). Read `node_modules/next/dist/docs/` before using any Next.js API (see AGENTS.md).

## 2. Folder structure

```
prisma/
  schema.prisma
  migrations/
  seed.ts
prisma.config.ts               # schema path, migrations, seed, datasource = DIRECT_URL
src/
  app/
    layout.tsx                 # <Header/> + notice banner
    page.tsx                   # /  listings + search/filter
    not-found.tsx
    sign-in/page.tsx
    sign-up/page.tsx
    jobs/
      new/page.tsx             # company form or job form
      [id]/page.tsx            # detail
      [id]/apply/page.tsx      # apply form
    dashboard/page.tsx         # candidate or employer view
    api/auth/[...all]/route.ts # Better Auth handler (toNextJsHandler)
  components/
    ui/                        # shadcn (generated, don't hand-edit)
    header.tsx
    job-card.tsx
    job-filters.tsx
    pagination.tsx
    notice.tsx
    status-select.tsx          # "use client": submits on change
    forms/                     # "use client" forms using useActionState
  lib/
    prisma.ts                  # the one PrismaClient singleton (PrismaNeon adapter)
    auth.ts                    # betterAuth() config
    auth-guards.ts             # getSession, requireUser, requireRole, safeNext
    format.ts                  # formatSalary, timeAgo
    notices.ts                 # notice code → message map
    validations/               # Zod schemas + action-state.ts
      action-state.ts
      auth.ts  company.ts  job.ts  application.ts
    queries/                   # read functions for Server Components
      jobs.ts  applications.ts  companies.ts
    actions/                   # "use server": one file per domain
      auth.ts  company.ts  job.ts  application.ts
  generated/prisma/            # prisma generate output (git-ignored)
```

**Import rules:**
- Import Prisma only through `@/lib/prisma`. Types and enums come from `@/generated/prisma/client`, never `@prisma/client`.
- `queries/` and `actions/` files start with `import "server-only"` (queries) or `"use server"` (actions).
- Client components never import `lib/prisma`, `lib/auth`, `queries/` or `actions/` except as Server Action references passed in or imported for `useActionState`.

## 3. Data model (draft `prisma/schema.prisma`)

The Better Auth models are a draft. Generate them once with `npx @better-auth/cli generate`, reconcile them with this file, then maintain them by hand.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
  // URL is NOT here: prisma.config.ts supplies DIRECT_URL to the CLI,
  // and src/lib/prisma.ts gives DATABASE_URL to the Neon adapter at runtime.
}

enum Role {
  CANDIDATE
  EMPLOYER
}

enum WorkType {
  REMOTE
  ONSITE
  HYBRID
}

enum EmploymentType {
  FULL_TIME
  PART_TIME
  CONTRACT
  INTERNSHIP
}

enum JobStatus {
  OPEN
  CLOSED
}

enum ApplicationStatus {
  SUBMITTED
  REVIEWED
  REJECTED
  ACCEPTED
}

// ---------- Better Auth ----------

model User {
  id            String    @id
  name          String
  email         String    @unique
  emailVerified Boolean   @default(false)
  image         String?
  role          Role      // Better Auth additionalField, set at sign-up, never updated
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  sessions      Session[]
  accounts      Account[]
  company       Company?
  applications  Application[]

  @@map("user")
}

model Session {
  id        String   @id
  expiresAt DateTime
  token     String   @unique
  ipAddress String?
  userAgent String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("session")
}

model Account {
  id                    String    @id
  accountId             String
  providerId            String
  userId                String
  user                  User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  accessToken           String?
  refreshToken          String?
  idToken               String?
  accessTokenExpiresAt  DateTime?
  refreshTokenExpiresAt DateTime?
  scope                 String?
  password              String?
  createdAt             DateTime  @default(now())
  updatedAt             DateTime  @updatedAt

  @@index([userId])
  @@map("account")
}

model Verification {
  id         String   @id
  identifier String
  value      String
  expiresAt  DateTime
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@index([identifier])
  @@map("verification")
}

// ---------- App ----------

model Company {
  id          String   @id @default(cuid())
  name        String   @unique @db.VarChar(80)
  website     String?
  description String?  @db.VarChar(1000)
  ownerId     String   @unique                       // one company per employer
  owner       User     @relation(fields: [ownerId], references: [id], onDelete: Restrict)
  jobs        Job[]
  createdAt   DateTime @default(now())
}

model Job {
  id             String         @id @default(cuid())
  title          String         @db.VarChar(100)
  description    String         @db.VarChar(5000)
  location       String         @db.VarChar(100)
  workType       WorkType
  employmentType EmploymentType
  salaryMin      Int?                                 // whole USD per year
  salaryMax      Int?
  status         JobStatus      @default(OPEN)
  companyId      String
  company        Company        @relation(fields: [companyId], references: [id], onDelete: Restrict)
  applications   Application[]
  createdAt      DateTime       @default(now())

  @@index([status, createdAt])
  @@index([companyId])
}

model Application {
  id          String            @id @default(cuid())
  coverLetter String            @db.VarChar(3000)
  resumeUrl   String
  status      ApplicationStatus @default(SUBMITTED)
  jobId       String
  job         Job               @relation(fields: [jobId], references: [id], onDelete: Restrict)
  candidateId String
  candidate   User              @relation(fields: [candidateId], references: [id], onDelete: Restrict)
  createdAt   DateTime          @default(now())

  @@unique([jobId, candidateId])                      // one application per job per candidate
  @@index([candidateId])
}
```

Notes:
- All IDs are strings. Better Auth generates User, Session, Account and Verification IDs; app tables use `cuid()`.
- A Zod check (`salaryMax >= salaryMin`) enforces the salary range, not the database.
- Nothing in the app deletes Company, Job or Application, so `Restrict` makes accidental deletes fail loudly.

### Prisma config and client

```ts
// prisma.config.ts
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: { path: "prisma/migrations", seed: "tsx prisma/seed.ts" },
  datasource: { url: env("DIRECT_URL") }, // CLI (migrate) uses the direct connection
});
```

```ts
// src/lib/prisma.ts
import { PrismaNeon } from "@prisma/adapter-neon";
import { PrismaClient } from "@/generated/prisma/client";

const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({ adapter: new PrismaNeon({ connectionString: process.env.DATABASE_URL! }) });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

The seed (`prisma/seed.ts`) imports `prisma` from `../src/lib/prisma`, so it never creates a client without the adapter. It creates users through `auth.api.signUpEmail` so passwords are hashed correctly.

## 4. Route map

Every page renders dynamically, with no `use cache` or ISR.

| Route | Access | Guard | Loads (via `lib/queries`) |
|---|---|---|---|
| `/` | Everyone | none | `listOpenJobs({ q, location, workType, employmentType, minSalary, page })`: open jobs + company name, 10 per page, newest first, plus total count |
| `/jobs/[id]` | Everyone | none (session read to pick the action) | `getJob(id)` with company and `_count.applications`; plus the viewer's application for this job (candidate) or ownership flag (employer). `notFound()` if missing |
| `/jobs/new` | EMPLOYER | `requireRole("EMPLOYER")` | `getCompanyByOwner(userId)` → company form if null, else job form |
| `/jobs/[id]/apply` | CANDIDATE | `requireRole("CANDIDATE")` | `getJob(id)`, `getApplication(jobId, userId)`. Show form only if job OPEN and no existing application |
| `/dashboard` | Signed in | `requireUser()` | Candidate: `listApplicationsByCandidate(userId)` with job + company. Employer: `getCompanyByOwner(userId)` + `listJobsWithApplicants(companyId)` (jobs, `_count`, applicants with user name/email) |
| `/sign-in`, `/sign-up` | Signed out | if session → `redirect("/dashboard")` | none |
| `/api/auth/[...all]` | Better Auth | Better Auth | Better Auth handler |

**Search query (`listOpenJobs`):**
- **Base filter:** `status: OPEN`.
- **Keyword `q`:** `OR` of `title`, `description` and `company.name`, each `{ contains: q, mode: "insensitive" }`.
- **`location`:** `contains`, insensitive.
- **`workType` / `employmentType`:** equals.
- **`minSalary`:** `OR [{ salaryMax: { gte: min } }, { salaryMax: null, salaryMin: { gte: min } }]`. Jobs with no salary are hidden while it's set.
- **Paging:** `orderBy: { createdAt: "desc" }`, `take: 10`, `skip: (page - 1) * 10`.
- Search params are parsed with a Zod schema, and invalid values are ignored.

## 5. Server Actions

All actions live in `src/lib/actions/*` and follow the same pattern:
1. Guard (session, role, ownership).
2. Validate with Zod.
3. Write.
4. Run `revalidatePath`.
5. `redirect()` on success, or return an `ActionState` on failure.

```ts
// src/lib/validations/action-state.ts
export type ActionState =
  | { ok: false; fieldErrors?: Record<string, string[]>; formError?: string; values?: Record<string, string> }
  | null;
```

Forms use `useActionState(action, null)` and redisplay `values` and `fieldErrors`. `fieldErrors` comes from `z.flattenError(error).fieldErrors`. IDs a form needs (such as `jobId`) are bound with `.bind(null, id)`, never trusted from hidden inputs.

| Action | File | Guard | Ownership / rule checks | Errors | Revalidate → redirect |
|---|---|---|---|---|---|
| `signUp` | actions/auth.ts | none (signed-out) | role ∈ {CANDIDATE, EMPLOYER} (Zod + DB hook) | Email taken → formError | → `safeNext(next) ?? "/"` |
| `signIn` | actions/auth.ts | none | none | Bad credentials → formError | → `safeNext(next) ?? "/"` |
| `signOut` | actions/auth.ts | none | none | none | → `/` |
| `createCompany` | actions/company.ts | `requireRole("EMPLOYER")` | Employer has no company yet | P2002 on `name` → "Company name taken"; on `ownerId` → redirect to `/jobs/new` | → `/jobs/new` |
| `createJob` | actions/job.ts | `requireRole("EMPLOYER")` | Employer owns a company (`companyId` from session, not form) | Zod field errors; `salaryMax >= salaryMin` | `/`, `/dashboard` → `/jobs/[id]` |
| `setJobStatus(jobId, status)` | actions/job.ts | `requireRole("EMPLOYER")` | `job.company.ownerId === user.id`, else 404-style error | none | `/`, `/jobs/[id]`, `/dashboard` |
| `applyToJob(jobId)` | actions/application.ts | `requireRole("CANDIDATE")` | Job exists and is OPEN; no existing application (pre-check) | P2002 on `[jobId, candidateId]` → "You've already applied" | `/jobs/[id]`, `/dashboard` → `/dashboard?notice=applied` |
| `setApplicationStatus(applicationId, status)` | actions/application.ts | `requireRole("EMPLOYER")` | `application.job.company.ownerId === user.id` | Invalid status → Zod | `/dashboard` |

Ownership is always checked by querying with the owner in the `where` clause (for example `where: { id, company: { ownerId: user.id } }`), so a mismatch looks the same as "not found".

## 6. Auth flow and roles

**Config (`src/lib/auth.ts`):**

```ts
export const auth = betterAuth({
  database: prismaAdapter(prisma, { provider: "postgresql" }),
  baseURL: process.env.BETTER_AUTH_URL,
  secret: process.env.BETTER_AUTH_SECRET,
  emailAndPassword: { enabled: true, minPasswordLength: 8 },
  user: {
    additionalFields: {
      role: { type: "string", required: true, input: true },
    },
  },
  databaseHooks: {
    user: {
      create: { before: async (user) => { /* throw if role not CANDIDATE|EMPLOYER */ return { data: user }; } },
      update: { before: async (data) => { const { role, ...rest } = data; return { data: rest }; } }, // role is immutable
    },
  },
  trustedOrigins: process.env.VERCEL_URL ? [`https://${process.env.VERCEL_URL}`] : [],
  plugins: [nextCookies()], // must be the last plugin
});
```

**Flow:**
1. The sign-up form posts to the `signUp` action, which runs Zod (name, email, password ≥ 8, role) and then `auth.api.signUpEmail({ body: { name, email, password, role } })`. The `nextCookies()` plugin sets the session cookie from inside the Server Action.
2. Sign-in works the same way with `auth.api.signInEmail`, and sign-out with `auth.api.signOut({ headers })`.
3. Session reads use `auth.api.getSession({ headers: await headers() })`, which returns `user.role`.

**Guards (`src/lib/auth-guards.ts`):**

| Helper | Behaviour |
|---|---|
| `getSession()` | Session or null; wrapped in React `cache()` so one request reads it once |
| `requireUser()` | No session → `redirect("/sign-in?next=<current path>")` |
| `requireRole(role)` | `requireUser()`, then wrong role → `redirect("/dashboard?notice=wrong-role")` |
| `safeNext(next)` | Returns `next` only if it starts with `/` and not `//`, else null |

- There is no `proxy.ts`. Guards run in every protected page and every action, because a page check alone doesn't protect its actions.
- Protected pages pass their own path to `requireUser` or `requireRole` so the `?next=` redirect returns there.

**Notices:** `?notice=applied | wrong-role` maps to fixed text in `lib/notices.ts`, shown by `<Notice>` in the layout. Unknown codes are ignored.

## 7. Environment variables

| Variable | Value | Local `.env` | Vercel |
|---|---|---|---|
| `DATABASE_URL` | Neon **pooled** URL (host contains `-pooler`), used by the app at runtime | `dev` branch | `main` branch, all environments |
| `DIRECT_URL` | Neon **direct** URL, used by the Prisma CLI (migrate, seed) | `dev` branch | `main` branch, all environments (`prisma.config.ts` fails without it, even in previews) |
| `BETTER_AUTH_SECRET` | `openssl rand -base64 32` | local value | its own value, all environments |
| `BETTER_AUTH_URL` | App origin | `http://localhost:3000` | Production URL (Production only) |

- `VERCEL_URL` and `VERCEL_ENV` are set by Vercel automatically.
- `.env` is git-ignored; commit an `.env.example` with empty values.

## 8. Development and deployment

**Neon branches:**
- **`dev`:** local work; `migrate dev` and the seed run here.
- **`main`:** production; only `migrate deploy` touches it.
- Previews use `main` but never migrate it, so don't deploy a preview whose schema is ahead of production.

**package.json scripts:**

```json
{
  "dev": "next dev",
  "build": "next build",
  "vercel-build": "if [ \"$VERCEL_ENV\" = \"production\" ]; then prisma migrate deploy; fi && next build",
  "start": "next start",
  "lint": "eslint",
  "postinstall": "prisma generate"
}
```

Vercel runs `vercel-build` instead of `build` when it exists.

**Local loop:**
1. Edit `schema.prisma`.
2. Run `npx prisma migrate dev --name <change>` against `dev`, then `npx prisma generate`.
3. Run `npx prisma db seed`.
4. Run `npm run dev`.

**Ship (`/ship` skill):**
1. Run the checks: Prisma 7.x, `postinstall`, `.env` not tracked, lint, `tsc --noEmit`, `npm run build`.
2. Commit and push to `main`.
3. Vercel runs `npm install` (and with it `prisma generate`), then `vercel-build`. Production builds apply pending migrations to Neon `main` before `next build`.
4. Check the build log shows the migrations applied, wait for Ready, then smoke-test the live URL.

Migrations must be additive (see the `add-field` skill). A migration that fails stops the build, and the previous deployment stays live.
