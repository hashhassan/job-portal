<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# AGENTS.md: rulebook for AI agents

This file is the single source of truth for any agent working here (CLAUDE.md just imports it). When a skill, a doc example or your own habits disagree with this file, **this file wins**.

## 1. Project

**Mini Job Board** is a practice and demo project:
- Employers create one company and post jobs.
- Candidates browse, search and filter open jobs, apply with a cover letter and resume link, and track their status.
- Employers open or close jobs and set each applicant's status.

The pages are `/`, `/jobs/[id]`, `/jobs/new`, `/jobs/[id]/apply`, `/dashboard`, `/sign-in` and `/sign-up`.

| Doc | What it holds |
|---|---|
| [docs/PRD.md](docs/PRD.md) | Requirements, scope, acceptance criteria, demo flow |
| [docs/Architecture.md](docs/Architecture.md) | Stack + exact versions, folders, Prisma schema, routes, Server Actions, auth, env vars, deploy |
| [docs/Design.md](docs/Design.md) | Visual style, tokens, typography, shadcn components, wireframes, page states |
| [docs/Tasks.md](docs/Tasks.md) | Phased task checklist with "done when" tests |
| [docs/Memory.md](docs/Memory.md) | Running log of what was done, decided and learned |

## 2. Tech rules

**Stack:** Next.js 16 App Router + TypeScript (`src/`), Tailwind 4 + shadcn/ui, Prisma ORM 7 + Neon Postgres, Better Auth (email + password, Prisma adapter), Zod, Vercel. Exact versions are in Architecture §1. Pin every dependency exactly (`npm i -E`).

### Prisma 7 only (overrides any skill or doc example)
- Install Prisma 7 only, with an explicit version: `prisma@7.10.1`, `@prisma/client@7.10.1`, `@prisma/adapter-neon@7.10.1`. **Never install Prisma 8.** npm's `latest` tag is Prisma 8, so never run `npm i prisma` without a version or with `@latest`.
- The generator is `provider = "prisma-client"` with `output = "../src/generated/prisma"`.
- Import from `"@/generated/prisma/client"` (and its `/enums`, `/browser` and `/models` subpaths). **Never import from `"@prisma/client"`.**
- Use the Neon driver adapter (`PrismaNeon` from `@prisma/adapter-neon`) with the pooled `DATABASE_URL`.
- The DB URL lives in `prisma.config.ts` (`DIRECT_URL`, for the CLI), never in `schema.prisma`.
- There is one singleton client in `src/lib/prisma.ts`. Everything imports `prisma` from `@/lib/prisma`, including the seed and scripts. Never write `new PrismaClient()` anywhere else.

### Reads, writes, validation
- **Server Components for reads**, through `src/lib/queries/*` (`import "server-only"`).
- **Server Actions for writes**, in `src/lib/actions/*` (`"use server"`). No API routes for app data; the only route handler is Better Auth's `/api/auth/[...all]`.
- **Zod validation on every input:** form data, bound arguments and `searchParams`. Actions return an `ActionState` on failure and `redirect()` on success (Architecture §5).
- After a mutation, `revalidatePath` every affected route.

### Auth, role and ownership: always on the server
- **Every protected page** calls `requireUser()` or `requireRole(role)` from `src/lib/auth-guards.ts`.
- **Every Server Action** runs its own guard before touching the DB. A page guard doesn't protect its actions.
- **Ownership:** put the owner in the query (`where: { id, company: { ownerId: user.id } }`), and treat a mismatch as not found.
- IDs that identify the user, their company or ownership come from the session, never from form fields.
- Roles are `CANDIDATE` or `EMPLOYER`. They're set at sign-up and can never be updated (Better Auth DB hooks).
- Redirect targets from `?next=` must pass `safeNext()`.

## 3. Skill map

Skills live in `.claude/skills/`. Use the one that matches the work. If a skill conflicts with §2, §2 wins.

| Work | Skill | Notes |
|---|---|---|
| Clarifying a plan or decision | `grilling` (`/grill-me`) | Used for the PRD and Architecture |
| Any coding task | `karpathy-guidelines` | Keep it simple, make surgical changes, verify |
| Prisma queries | `prisma-client-api` | Use Prisma 7 + `@/generated/prisma/client` only |
| Prisma CLI (migrate, generate, seed) | `prisma-cli` | Ignore its reference to `prisma-orm-setup` (not installed) |
| New model end to end | `new-feature` | ⚠️ Rewrite pending (Tasks Phase 1). Don't use it until then |
| Add one field to a model | `add-field` | ⚠️ Rewrite pending (Tasks Phase 1). Additive migrations only |
| Visual and UX decisions, UI review | `ui-ux-pro-max` | Run its scripts with `py -3` on this machine. Design.md already holds the decisions |
| shadcn/Tailwind implementation | `ui-styling` | Follow Design.md tokens and its component list |
| Stress-testing a built UI | `break-ui` | JobCard, job detail, employer dashboard |
| Build, Prisma, Neon or Vercel errors | `fix-error` | Check `errors.md` first |
| Wrong behaviour or a hard bug | `diagnosing-bugs` | Build a feedback loop before guessing |
| Security check before deploy | `security-review` (built-in) | Phase 8 |
| Code review | `code-review` (built-in) | Phase 8 |
| Deploying | `/ship` | Checks, commit, push; migrations run in `vercel-build` |
| Ending a long session | `handoff` | Summary for the next agent |

Don't use: `design` (logos, banners) or `design-system` (formal tokens, slides). Both are out of scope.

## 4. Workflow

1. **At session start:** read `docs/Tasks.md` and `docs/Memory.md`, then pick the first unchecked task in phase order, unless the user names one.
2. **Work on one task at a time.** Plan first: state the steps and how you'll check them against the task's "done when". Then write the code.
3. **After each task:**
   1. Run `npm run lint` and `npx tsc --noEmit`, and fix every error. Don't hide errors with `any`, `@ts-ignore` or empty catches.
   2. Run the task's "done when" test.
   3. Tick the task in `docs/Tasks.md`.
   4. Add a short entry to `docs/Memory.md`: date, task, what changed, any decision or gotcha.
   5. Commit with a clear message such as `feat(auth): add sign-up page and action`, ending with the attribution line your harness requires.
4. **Ask first** before anything that changes Neon (`migrate dev`, `db seed`, anything against `main`), pushes, or deploys.
5. **Never change `docs/PRD.md` or `docs/Architecture.md` without asking.** If the code needs to diverge from them, stop and ask. `Design.md`, `Tasks.md` and `Memory.md` can be updated as work goes on, as long as you mention the changes.

## 5. Don'ts

- **Never commit `.env`** or any secret. Only `.env.example` with empty values goes in git.
- **Never add a dependency without asking** (including dev dependencies and shadcn components not listed in Design §5).
- **No features outside the PRD:** no extras, no "while I'm here" refactors. Out-of-scope items are listed in PRD §6.
- Never install Prisma 8, import from `@prisma/client`, or create a second PrismaClient.
- Never put app-data reads in client components, or create API routes for app data.
- Never trust the client for role, ownership or user IDs.
- Never run `prisma migrate reset`, `db push --accept-data-loss` or `migrate deploy` by hand against production.
