# Memory: running log

Read at the start of every session (see AGENTS.md §4). Keep entries short. Newest entries go at the top of each section.

---

## Current status

_Last updated: 2026-11-18_

- **Phase:** planning done; no app code written yet.
- **Done:**
  - Next.js 16.4.6 starter app
  - Neon project with a `dev` branch, and `.env` (git-ignored)
  - docs: PRD, Architecture, Design, Tasks, Memory
  - AGENTS.md as the rulebook, with CLAUDE.md importing it
  - Skill fixes: Prisma import paths, ship checks, errors.md
- **Next task:** Phase 1, "Pin versions" (docs/Tasks.md).
- **Open items:**
  - The new-feature and add-field skills still need rewriting (Phase 1 task).
  - Nothing has been committed since the initial Create Next App commit.

---

## Key decisions

| Date | Decision | Reason |
|---|---|---|
| 2026-11-18 | Prisma pinned to exactly **7.10.1** (`prisma`, `@prisma/client`, `@prisma/adapter-neon`); every dependency pinned with no `^` | npm `latest` is now Prisma 8 (`8.0.0-rc.34`); exact pins keep installs reproducible |
| 2026-11-18 | TypeScript stays on **5.9.3** | npm `latest` is 7.x, a major jump with no benefit for this project |
| 2026-11-18 | Neon adapter (`PrismaNeon`) instead of `PrismaPg` | Fits Neon serverless on Vercel; the user's choice |
| 2026-11-18 | `DATABASE_URL` = pooled (app runtime), `DIRECT_URL` = direct (Prisma CLI via `prisma.config.ts`) | Pooling for serverless; migrations need a direct connection |
| 2026-11-18 | Two Neon branches: `dev` (local, `migrate dev`, seed) and `main` (production, `migrate deploy` only) | `migrate dev` never touches production |
| 2026-11-18 | `vercel-build` runs `prisma migrate deploy` only when `VERCEL_ENV=production`; previews only build | Previews share the `main` DB and must not migrate it |
| 2026-11-18 | Role is a Better Auth `additionalField` backed by a Prisma `enum Role`, chosen at sign-up; DB hooks validate it on create and strip it on update | The PRD says roles are permanent; Better Auth's update-user endpoint would otherwise let users change their own role |
| 2026-11-18 | Sign-up, sign-in and sign-out run as Server Actions calling `auth.api.*` with the `nextCookies()` plugin | Keeps the "Server Actions for mutations" rule; no client auth SDK needed |
| 2026-11-18 | No `proxy.ts` (formerly middleware); guards (`requireUser`, `requireRole`) in every protected page and action | Server checks are the real protection; a page guard doesn't protect its actions |
| 2026-11-18 | Ownership is enforced in the query `where` clause; IDs come from the session | A mismatch looks like "not found"; nothing comes from the client |
| 2026-11-18 | All IDs are strings: Better Auth IDs, `cuid()` for app tables | Consistent with Better Auth's `User.id` |
| 2026-11-18 | One application per job per candidate: `@@unique([jobId, candidateId])`, with a pre-check plus a `P2002` backstop | Handles the double-submit race without extra queries |
| 2026-11-18 | Jobs can't be edited or deleted, only toggled Open/Closed; relations use `onDelete: Restrict` | Keeps the MVP small; accidental deletes fail loudly |
| 2026-11-18 | Resume is a URL, not an upload; no logos | File uploads would need storage and a new dependency |
| 2026-11-18 | Search uses Prisma `contains` (insensitive) across title, description and company name; `minSalary` hides jobs without a salary | About 15 jobs, so no full-text search needed |
| 2026-11-18 | Notices and the return path go in URL params (`?notice=`, `?next=` checked by `safeNext`) | No cookies or client state needed; blocks open redirects |
| 2026-11-18 | All pages render dynamically; mutations call `revalidatePath` | Every page reads the session or `searchParams` |
| 2026-11-18 | Design: Swiss-minimal, light only, sky-700 `#0369A1` primary on slate neutrals, Inter via `next/font`, lucide icons | From the `ui-ux-pro-max` design system, trimmed: no GSAP, no funnel pattern, no new dependencies |
| 2026-11-18 | Employer dashboard uses native `<details>`; the status select submits on change | No client state; works without JS except the select |
| 2026-11-18 | AGENTS.md is the single rulebook; CLAUDE.md only contains `@AGENTS.md` | One source of truth for every agent |
| 2026-11-18 | PRD.md and Architecture.md change only with the user's approval | They're the agreed contract |

---

## Gotchas / things that broke and how we fixed them

| Date | What happened | Fix / rule |
|---|---|---|
| 2026-11-18 | `sed -i` in Git Bash on Windows converted the skill files from CRLF to LF line endings | Restored from a backup and re-ran with `sed -b -i`, or use the Edit tool. Always diff whole files after a shell edit |
| 2026-11-18 | `python` on Windows opens the Microsoft Store shortcut | Use `py -3` (Python 3.12.6 is installed) |
| 2026-11-18 | `py -3 -I` breaks the `ui-ux-pro-max` script (`No module named 'core'`) | Run it without `-I`; it's a trusted local skill |
| 2026-11-18 | Skills had wrong Prisma import paths (`../generated/client`) | Fixed to `@/generated/prisma/client` and output `../src/generated/prisma` in prisma-client-api and prisma-cli |
| 2026-11-18 | The `ship` skill skipped lint and `tsc`, and ran `migrate deploy` by hand | Added lint and `tsc` before the build; migrations now run in `vercel-build` |
| 2026-11-18 | The `new-feature` and `add-field` skills generate API routes plus client fetch pages with no auth or Zod | Don't use them until they're rewritten (Tasks Phase 1) |
| 2026-11-18 | The `prisma-cli` seed example uses `new PrismaClient()` with no adapter, which fails in Prisma 7 | The seed imports the `@/lib/prisma` singleton instead |
| 2026-11-18 | Next.js 16 renamed `middleware.ts` to `proxy.ts`; APIs differ from older versions | Read `node_modules/next/dist/docs/` before using a Next.js API |
| 2026-11-18 | `Claude.md` had the wrong case | Renamed to `CLAUDE.md` (Windows ignores case, git on other systems doesn't) |

**To verify when we get there:**
- Better Auth table fields: regenerate them with its CLI.
- Whether Prisma's `env("DIRECT_URL")` fails when the variable is missing, which would break `prisma generate` in Vercel previews.
- Whether the `nextCookies()` plugin works when the seed runs outside Next.js.
- Whether the shadcn `empty` and `spinner` components exist in CLI 4.23.0.

---

## Session log

### 2026-11-18: Planning
- Reviewed all 16 skills in `.claude/skills/` and mapped them to project stages.
- Fixed the Prisma import paths in the skills, added lint and `tsc` to `ship`, and updated `fix-error/errors.md` (Neon adapter, exact pins, migration flow).
- Grilled requirements (Q1–Q27), then wrote `docs/PRD.md`, rewritten to the requested structure (overview, goals, roles, stories, pages, scope, acceptance criteria, demo flow).
- Grilled architecture (Q1–Q19), then wrote `docs/Architecture.md`.
- Ran the `ui-ux-pro-max` design system and focused searches, then wrote `docs/Design.md` (tokens, type, spacing, components, wireframes, states).
- Wrote `docs/Tasks.md`: 47 tasks in 10 phases, each with a "done when" test.
- Wrote `AGENTS.md` as the rulebook, replaced CLAUDE.md with `@AGENTS.md` and fixed its filename case.
- Set up this file.
