# PRD: Mini Job Board

## 1. Overview

A small job board built as a practice and demo project. Employers create a company and post jobs. Candidates browse, search and apply. Employers review applicants and set a status, which candidates can track.

Stack: Next.js App Router, Prisma 7 + Neon, Better Auth, Tailwind + shadcn/ui, deployed on Vercel.

## 2. Goals

- Ship a polished, working MVP that can be demoed end to end in about 5 minutes.
- Practise the core patterns: Server Components for reads, Server Actions + Zod for writes, and role checks on the server.
- Keep the scope small: nothing beyond this document.

## 3. User roles

| Role | How | Can |
|---|---|---|
| Visitor | Not signed in | Browse, search and view open jobs |
| Candidate | Picked at sign-up, permanent | Apply to jobs, track application status |
| Employer | Picked at sign-up, permanent | Create one company, post jobs, open/close jobs, review applicants, set status |

No admin, no role switching, one company per employer.

## 4. User stories

**Visitor**
- As a visitor, I can browse and search open jobs without an account.
- As a visitor, I can sign up as a candidate or an employer.

**Candidate**
- As a candidate, I can filter jobs by keyword, location, work type, employment type and minimum salary.
- As a candidate, I can apply to an open job with a cover letter and a resume link.
- As a candidate, I can see all my applications and their current status.

**Employer**
- As an employer, I can create my company and then post jobs.
- As an employer, I can close a job so it stops showing and stops taking applications.
- As an employer, I can see the applicants for each of my jobs and set their status.

## 5. Features per page

**Header (all pages):** links change by role.
- Signed out: Jobs, Sign in, Sign up
- Candidate: Jobs, Dashboard, Sign out
- Employer: Jobs, Post a job, Dashboard, Sign out

**`/sign-up`, `/sign-in`**
- Sign-up: name, email, password (8+ chars), role. Sign-in: email, password.
- After either, return the user to the page they came from (default `/`).

**`/`: job listings**
- Open jobs only, newest first, 10 per page with Prev/Next.
- Keyword search (title, description, company name; case-insensitive) plus filters: location, work type, employment type, minimum salary. All kept in the URL.
- Card: title, company, location, work type, employment type, salary, "3 days ago".

**`/jobs/[id]`: job detail**
- Job fields plus company name, website and description.
- The action depends on the viewer:
  - Signed out: "Sign in to apply"
  - Candidate: "Apply", or "Applied · status" if already applied
  - Owning employer: "View applicants (n)"
  - Other employers: nothing
- A closed job shows a "Closed" badge and no Apply button.

**`/jobs/new`: post a job (employer only)**
- If the employer has no company yet, the company form comes first: name, website, description.
- Job form: title, description, location, work type (Remote/On-site/Hybrid), employment type (Full-time/Part-time/Contract/Internship), optional salary min/max (USD per year).
- On success, go to the new job's detail page.

**`/jobs/[id]/apply`: apply (candidate only)**
- Cover letter and resume URL. Blocked if the job is closed or the candidate already applied.
- On success, go to `/dashboard` with an "Application sent" message.

**`/dashboard`**
- Candidate: their applications (job, company, date, status).
- Employer: their jobs (Open/Closed toggle, applicant count). Each job expands to show its applicants (name, email, cover letter, resume link, date) with a status dropdown: Submitted / Reviewed / Rejected / Accepted.

**Access rules**
- Signed out on a protected page: go to `/sign-in`, then come back.
- Wrong role: go to `/dashboard` with a message.
- Every page and server action checks this on the server.

**Validation limits**

| Field | Limit |
|---|---|
| Title, location | 100 chars |
| Job description | 5,000 chars |
| Company name | 80 chars, unique |
| Company description | 1,000 chars |
| Cover letter | 3,000 chars |
| URLs | Valid `http(s)` |
| Salary | 0–1,000,000, max ≥ min |

**Salary display:** "$80k – $100k", "From $80k", "Up to $100k", or hidden if both are empty.

## 6. Scope

**In scope:** everything in section 5, plus empty states, a not-found page for unknown jobs, and a seed script (2 employers with companies, 15 jobs with some closed, 2 candidates with a few applications).

**Out of scope:** email notifications, password reset, email verification, profile editing, editing or deleting jobs and companies, file uploads (resumes, logos), saved jobs, messaging, admin moderation, multiple currencies, sort options, withdrawing applications.

## 7. Acceptance criteria

**Auth and access**
- [ ] Users can sign up as a candidate or an employer; the role can't be changed later.
- [ ] Signed-out users opening a protected page are sent to sign-in and come back afterwards.
- [ ] Wrong-role users are redirected to `/dashboard` with a message.
- [ ] Server actions reject callers with no session or the wrong role, even when the UI is bypassed.

**Listings**
- [ ] Only open jobs appear, newest first, 10 per page.
- [ ] Keyword and filters combine correctly and survive a page reload or a shared URL.
- [ ] No results shows "No jobs match" with a working "Clear filters" link.

**Job detail**
- [ ] Each viewer type sees the correct action.
- [ ] A closed job shows a "Closed" badge and can't be applied to.
- [ ] An unknown ID shows the not-found page.

**Post a job**
- [ ] An employer with no company must create one first.
- [ ] Invalid input shows field errors and nothing is saved.
- [ ] A new job opens on its detail page and appears on `/`.

**Apply**
- [ ] A candidate can apply once per open job; a second attempt (including a resubmitted form) is rejected.
- [ ] After applying, the candidate lands on `/dashboard` with a confirmation.

**Dashboard**
- [ ] A candidate sees all their applications with the current status.
- [ ] An employer sees only their own jobs and applicants.
- [ ] Closing a job removes it from `/`; changing an applicant's status is visible to that candidate.
- [ ] Each role has an empty state.

## 8. Demo flow

1. Run the seed. Open `/` signed out, then search "react" and filter by Remote.
2. Open a job and click "Sign in to apply". Sign up as a new candidate, and you return to the job.
3. Apply with a cover letter and resume URL. Land on the dashboard and see the application as "Submitted".
4. Sign out. Sign up as a new employer and go to Post a job. Create the company, post a job, and land on its detail page.
5. Sign in as a seeded employer. On the dashboard, expand a job and set an applicant to "Reviewed".
6. Close that job and check it's gone from `/`.
7. Sign in as that applicant. The dashboard shows "Reviewed".
