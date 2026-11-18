# Tasks: Mini Job Board

Source docs: [PRD.md](PRD.md), [Architecture.md](Architecture.md), [Design.md](Design.md).

Each task fits in one session. Every task's "done when" also includes the CLAUDE.md rule: `npm run lint` and `npx tsc --noEmit` pass. Steps marked 🔒 change the Neon DB or deploy, so they need your approval when run.

---

## Phase 1: Setup

- [x] **Next.js app.** Create Next.js 16.4.6 + TypeScript + Tailwind 4 with `src/` dir.
  Done when: `npm run dev` serves the starter page. *(done)*
- [x] **Neon project + `.env`.** Neon project with a `dev` branch, and `.env` holding its pooled `DATABASE_URL` and direct `DIRECT_URL`.
  Done when: `.env` exists and is git-ignored. *(done)*
- [ ] **Pin versions.** Remove `^`/`~` from every package.json entry, using the installed versions.
  Done when: `grep -E '"[\^~]' package.json` prints nothing and `npm install` is clean.
- [ ] **`.env.example`.** Commit the four env var names (Architecture §7) with empty values.
  Done when: `git ls-files .env.example` lists it and it contains no secrets.
- [ ] **Theme + font.** Set the Design §2 tokens in `globals.css` and load Inter via `next/font` as `--font-sans`.
  Done when: the starter page renders in Inter with a `bg-primary` test button in `#0369A1`.
- [ ] **shadcn init + components.** Run `npx shadcn@4.23.0 init` (new-york, neutral base, CSS vars), then `add` the 15 components from Design §5.
  Done when: `src/components/ui/` has all 15 files (or the documented fallback for `empty`/`spinner`) and the build passes.
- [ ] **App shell.** `layout.tsx` gets a skip link, a container and a header placeholder; add `error.tsx` and `not-found.tsx` per Design §6.
  Done when: `/does-not-exist` shows the not-found page, and a thrown test error shows "Something went wrong" with Try again.
- [ ] **Rewrite the `new-feature` and `add-field` skills.** Make them match Architecture §2 (folders), §5 (Zod + Server Action + `ActionState`), §6 (guards) and the `src/lib/prisma.ts` singleton. Keep additive-only migrations and the migrate approval step.
  Done when: neither SKILL.md mentions `api/<models>/route.ts` or `"use client"` pages that fetch, and both reference `requireRole`, Zod and `revalidatePath`.

## Phase 2: Prisma + Neon

- [ ] **Install Prisma 7.** Run `npm i -E @prisma/client@7.10.1 @prisma/adapter-neon@7.10.1` and `npm i -D -E prisma@7.10.1 dotenv tsx`.
  Done when: `npm ls prisma @prisma/client @prisma/adapter-neon` shows only 7.10.1.
- [ ] **Prisma config + empty schema.** Add `prisma.config.ts` (DIRECT_URL, migrations path, seed), plus a `schema.prisma` with only the generator (`prisma-client` → `../src/generated/prisma`) and a datasource with no URL. Git-ignore `src/generated`.
  Done when: `npx prisma validate` passes and `npx prisma generate` writes `src/generated/prisma/client.ts`.
- [ ] **Prisma singleton.** Create `src/lib/prisma.ts` with `PrismaNeon` and DATABASE_URL (Architecture §3).
  Done when: a throwaway `npx tsx` script running `prisma.$queryRaw\`SELECT 1\`` through the singleton prints `[ { '?column?': 1 } ]`. Delete the script afterwards.
- [ ] **`postinstall` script.** Add `"postinstall": "prisma generate"`.
  Done when: deleting `src/generated` and running `npm install` regenerates it.

## Phase 3: Better Auth

- [ ] **Install + auth tables.** `npm i -E better-auth@1.7.25`; generate the User, Session, Account and Verification models with the Better Auth CLI, add `enum Role` and `User.role`, then 🔒 `migrate dev --name add-auth`.
  Done when: the migration applies on Neon `dev` and the four tables appear in `npx prisma studio`.
- [ ] **Auth config + route.** Create `src/lib/auth.ts` (Prisma adapter, email+password with min 8, `role` additional field, `nextCookies()` last) and `src/app/api/auth/[...all]/route.ts`.
  Done when: `curl localhost:3000/api/auth/ok` returns `{"ok":true}`.
- [ ] **Role hooks.** Add a create hook that rejects roles other than CANDIDATE/EMPLOYER, and an update hook that strips `role`.
  Done when: sign-up via curl with `role:"ADMIN"` fails; with `role:"EMPLOYER"` it succeeds; a `POST /api/auth/update-user` with `role:"CANDIDATE"` leaves the DB role as EMPLOYER.
- [ ] **Guards + notices.** Create `src/lib/auth-guards.ts` (`getSession` with `cache`, `requireUser`, `requireRole`, `safeNext`) and `src/lib/notices.ts`.
  Done when: `safeNext` returns null for `//evil.com`, `https://evil.com` and `evil`, and returns the path for `/jobs/1`. Check with a quick tsx script, then delete it.
- [ ] **Auth Zod schemas + `ActionState`.** Add `validations/action-state.ts` and `validations/auth.ts` (sign-up and sign-in).
  Done when: `tsc` passes and an invalid sign-up object returns field errors for name, email, password and role.
- [ ] **Sign-up page + action.** `/sign-up` form (role RadioGroup cards, name, email, password with show/hide) plus a `signUp` action using `auth.api.signUpEmail`.
  Done when: signing up as a candidate sets the session cookie and redirects to `?next=` (or `/`); a duplicate email shows the Email field error; values are kept on error.
- [ ] **Sign-in, sign-out.** `/sign-in` page plus `signIn` and `signOut` actions. A signed-in user who visits either auth page goes to `/dashboard`.
  Done when: a wrong password shows "Email or password is incorrect."; a correct one redirects to `next`; sign-out clears the session.
- [ ] **Header.** Role-based links (Design §7) and a sign-out form, with the active link highlighted.
  Done when: signed out, candidate and employer each see exactly their links (checked by signing in as each).

## Phase 4: Data models + seed

- [ ] **App models.** Add Company, Job and Application plus the four enums, relations, `@@unique([jobId, candidateId])` and the indexes (Architecture §3), then 🔒 `migrate dev --name add-job-board-models`.
  Done when: the migration applies, and `prisma studio` shows the three tables with the unique constraint visible in the migration SQL.
- [ ] **Domain Zod schemas.** `validations/company.ts`, `job.ts` (with `salaryMax >= salaryMin`), `application.ts`, and a search-params schema for `/`.
  Done when: a throwaway test with valid and invalid samples matches the PRD limits (100 / 5000 / 80 / 1000 / 3000 chars, http(s) URLs, 0–1,000,000 salary). Delete the test afterwards.
- [ ] **Format helpers.** `src/lib/format.ts` with `formatSalary`, `timeAgo`, `formatDate`, enum labels and `plural`.
  Done when: a tsx check prints `$80k – $100k`, `From $80k`, `Up to $100k`, an empty string, `3 days ago` and `4 applicants`.
- [ ] **Seed script.** `prisma/seed.ts`: wipe all tables (Application → Job → Company → Session/Account/Verification → User), then create 2 employers with companies and 2 candidates via `auth.api.signUpEmail`, 15 jobs (mixed types and salaries, 3 closed, at least 2 mentioning "React") and about 5 applications in all four statuses.
  Done when: 🔒 `npx prisma db seed` runs twice in a row without errors, and the counts are 4 users, 2 companies, 15 jobs and about 5 applications. If `nextCookies` breaks outside Next.js, use a seed-only auth instance.

## Phase 5: Public pages

- [ ] **Job queries.** `queries/jobs.ts` with `listOpenJobs` (search, filters, paging, total) and `getJob`.
  Done when: a tsx check against seed data returns 12 open jobs, the "react" filter returns only React jobs, `minSalary` hides jobs without a salary, and page 2 has 2 items.
- [ ] **Listing page `/`.** `JobCard`, `JobMeta` and `StatusBadge`, a results count and Prev/Next pagination, as Server Components.
  Done when: `/` shows 10 newest open jobs, `?page=2` shows the rest, and closed jobs never appear.
- [ ] **Filters.** A `JobFilters` GET form (keyword, location, work type, employment type, min salary) with Clear filters, in a sidebar at `lg` and stacked on mobile.
  Done when: filters combine, the URL updates, reloading or sharing the URL keeps the results, and bad params are ignored.
- [ ] **Listing states.** Two empty states (no jobs, no matches) and `app/loading.tsx` skeletons.
  Done when: a search for "zzzz" shows "No jobs match your search" with a working Clear filters link, and throttled loading shows skeletons with no layout shift.
- [ ] **Job detail `/jobs/[id]`.** Job and company info, Closed badge, an action card for each viewer type, `notFound()` and `loading.tsx`.
  Done when: each of the 5 viewer types in the PRD table sees the correct action (checked with seed accounts), a closed job shows no Apply, and a bad ID shows not-found.

## Phase 6: Employer flow

- [ ] **Company step.** `/jobs/new` guarded by `requireRole("EMPLOYER")`, with the company form and a `createCompany` action (`P2002` → "name taken").
  Done when: a new employer sees "Set up your company"; submitting continues to the job form; a duplicate name shows the field error; a candidate visiting is redirected to `/dashboard?notice=wrong-role`.
- [ ] **Post a job.** Job form plus a `createJob` action (companyId from the session, revalidate `/` and `/dashboard`).
  Done when: a valid post redirects to its detail page and appears first on `/`; invalid input (empty title, max < min, bad values) shows field errors and saves nothing.
- [ ] **Employer dashboard list.** `/dashboard` employer view: company name, job cards in `<details>` with status badges and applicant counts, plus the empty state.
  Done when: seeded employer A sees only their own jobs with correct counts, and a new employer with no jobs sees "You haven't posted any jobs yet".
- [ ] **Open/Close toggle.** A `setJobStatus` action with an ownership check in the `where` clause.
  Done when: closing a job removes it from `/` and shows the Closed badge on its detail page; reopening restores it; another employer's job ID is rejected.
- [ ] **Applicants + status.** Applicant rows (name, email, date, resume link, clamped cover letter) and a `StatusSelect` that submits on change to `setApplicationStatus` (ownership checked).
  Done when: changing a status persists after reload, and the candidate sees it on their dashboard; a job with no applicants shows "No applicants yet".

## Phase 7: Candidate flow

- [ ] **Apply page.** `/jobs/[id]/apply` guarded by `requireRole("CANDIDATE")`: form, already-applied state, closed state and not-found.
  Done when: an employer is redirected with the wrong-role notice; a signed-out user returns to the apply page after sign-in; closed or already-applied jobs show their Alert instead of the form.
- [ ] **`applyToJob` action.** Guard, open check, pre-check for duplicates, `P2002` backstop, revalidate, then redirect to `/dashboard?notice=applied`.
  Done when: applying shows "Application sent"; resubmitting the same form (browser back + submit) shows "You've already applied"; the DB has exactly one row.
- [ ] **Candidate dashboard.** The list of applications with status badges, and the empty state.
  Done when: a seeded candidate sees their applications with the correct statuses, and a new candidate sees "You haven't applied to any jobs yet" + Browse jobs.

## Phase 8: Polish + security review

- [ ] **Access matrix check.** Visit every route as signed out, candidate and employer.
  Done when: every result matches PRD §5 "Access rules", including the `?next=` return, and `?next=//evil.com` falls back to `/`.
- [ ] **Server Action guard audit.** Check that every exported action starts with a guard, and that ownership is enforced in the `where` clause.
  Done when: each action in `src/lib/actions/*` has `requireUser`/`requireRole` (or is an auth action) before any DB call, and the `/security-review` skill reports no high or medium findings.
- [ ] **Break the UI.** Run the `break-ui` skill on JobCard, job detail and the employer dashboard.
  Done when: every reported break is fixed or accepted, and long titles, long emails and 0 or 50 applicants don't overflow at 375 px.
- [ ] **Accessibility + responsive pass.** Check focus rings, labels, the error summary focus, badge text, and widths of 375, 768, 1024 and 1440 px.
  Done when: every item in the Design §9 checklist is ticked.
- [ ] **Code review.** Run `/code-review` on the full branch and fix the findings.
  Done when: no correctness findings remain open; lint, `tsc` and `npm run build` are clean.

## Phase 9: Deploy

- [ ] **Neon `main` branch URLs.** Get the production pooled and direct URLs from the Neon `main` branch.
  Done when: both URLs are noted somewhere safe (not in git) and differ from the `dev` ones.
- [ ] **`vercel-build` script.** Add the conditional `prisma migrate deploy && next build` script (Architecture §8).
  Done when: `npm run build` still works locally, and the script text matches the doc.
- [ ] **Vercel project + env vars.** Import the repo; set `DATABASE_URL`, `DIRECT_URL` and `BETTER_AUTH_SECRET` (all environments) and `BETTER_AUTH_URL` (Production); set Node 22.
  Done when: the Vercel env settings list all four variables with the correct scopes.
- [ ] 🔒 **First deploy via `/ship`.** Run the checks, commit and push.
  Done when: the production build log shows the migrations applied, the deployment is Ready, and the live `/` loads (empty state is fine).
- [ ] 🔒 **Seed production.** Run the seed once against Neon `main` (temporarily point DIRECT_URL/DATABASE_URL at `main` in your shell, never in `.env`).
  Done when: the live `/` shows 12 open jobs and the seeded accounts can sign in.
- [ ] **Live smoke test.** Sign up, apply, post a job and change a status on the live URL.
  Done when: all four work, and Vercel Runtime Logs show no errors.

## Phase 10: Demo prep

- [ ] **README.** Project summary, stack, local setup (env, migrate, seed, dev), seeded demo accounts and the live URL.
  Done when: a fresh clone can follow it to a running app without asking questions.
- [ ] **Acceptance criteria sweep.** Tick every checkbox in PRD §7 on the live site.
  Done when: all PRD acceptance criteria are checked, or any gap is logged here as a task.
- [ ] **Demo rehearsal.** Run the PRD §8 demo flow end to end on production, in about 5 minutes.
  Done when: all 7 steps pass without a hitch.
- [ ] 🔒 **Demo reset.** Re-run the seed on production right before the demo so the data is clean.
  Done when: the seed counts are back to baseline (15 jobs, about 5 applications) and the rehearsal accounts are gone.
