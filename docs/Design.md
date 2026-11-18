# Design: Mini Job Board

Implements [PRD.md](PRD.md) on the stack in [Architecture.md](Architecture.md). Derived with the `ui-ux-pro-max` skill (design-system query "job board recruitment professional clean", variance 3, motion 2, density 6), then trimmed to the project's rules: no new dependencies, Server Components first.

## 1. Visual direction

**Clean, Swiss-style minimalism for a work tool.** Content first, high contrast, generous whitespace, a strict grid, one accent colour. It should feel like a calm, trustworthy job site, not a marketing page.

- **Do:** neutral surfaces, a single blue for actions and links, clear type hierarchy, thin borders, flat cards (border plus at most a `shadow-xs`), filters always visible.
- **Don't:** gradients, illustrations, glassmorphism, scroll animations, emoji as icons, colour-only meaning, hidden filters, more than one primary button per view.
- **Theme:** light only for the MVP. The tokens are shadcn's, so dark mode can be added later without touching components.
- **Icons:** `lucide-react` (bundled with shadcn), 16 px inline and 20 px standalone, stroke 2. Decorative icons get `aria-hidden`.
- **Motion:** only shadcn's default transitions (150–200 ms colour and opacity on hover and focus). No entrance animations. Respect `prefers-reduced-motion`.

## 2. Colour palette

Set as shadcn CSS variables in `src/app/globals.css` (`:root`). Components use tokens (`bg-primary`, `text-muted-foreground`), never raw hex.

| Token | Hex | Tailwind ref | Used for |
|---|---|---|---|
| `--background` | `#FFFFFF` | white | Page |
| `--foreground` | `#0F172A` | slate-900 | Body text, headings |
| `--card` / `--card-foreground` | `#FFFFFF` / `#0F172A` | n/a | Job cards, panels |
| `--muted` | `#F1F5F9` | slate-100 | Filter bar, table header, skeletons |
| `--muted-foreground` | `#475569` | slate-600 | Meta text (company, dates), helper text |
| `--border` / `--input` | `#E2E8F0` | slate-200 | Card and input borders, separators |
| `--primary` | `#0369A1` | sky-700 | Primary buttons, links, active nav |
| `--primary-foreground` | `#FFFFFF` | n/a | Text on primary |
| `--secondary` | `#F1F5F9` | slate-100 | Secondary buttons, neutral badges |
| `--secondary-foreground` | `#0F172A` | n/a | n/a |
| `--accent` | `#F0F9FF` | sky-50 | Hover background on ghost buttons and rows |
| `--accent-foreground` | `#0C4A6E` | sky-900 | n/a |
| `--destructive` | `#DC2626` | red-600 | Field errors, Rejected |
| `--ring` | `#0369A1` | sky-700 | Focus ring (2 px + offset) |
| `--radius` | `0.5rem` | n/a | Cards, inputs, buttons |

**Status colours** (badges always carry a text label too, so colour is never the only signal):

| Status | Badge style |
|---|---|
| Open | `outline`, green text `#15803D` (green-700) + dot |
| Closed | `secondary` (slate) |
| Submitted | `secondary` (slate) |
| Reviewed | blue: bg `#E0F2FE` / text `#075985` (sky-100/800) |
| Accepted | green: bg `#DCFCE7` / text `#166534` (green-100/800) |
| Rejected | red: bg `#FEE2E2` / text `#991B1B` (red-100/800) |

All text/background pairs above meet WCAG AA (≥ 4.5:1).

## 3. Typography

**Inter** for everything (skill pairing "Minimal Swiss"), loaded with `next/font/google` in `layout.tsx` as `--font-sans`. No extra package and no layout shift.

| Role | Tailwind | Size / line-height | Weight |
|---|---|---|---|
| Page title (h1) | `text-2xl sm:text-3xl tracking-tight` | 24–30 px / 1.2 | 600 |
| Section title (h2) | `text-xl` | 20 px / 1.3 | 600 |
| Card title (h3) | `text-base sm:text-lg` | 16–18 px / 1.4 | 600 |
| Body | `text-base` | 16 px / 1.6 | 400 |
| Meta / helper | `text-sm text-muted-foreground` | 14 px / 1.5 | 400 |
| Label | `text-sm` | 14 px | 500 |
| Badge | `text-xs` | 12 px | 500 |

Rules:
- Salaries and counts use `tabular-nums`.
- Job descriptions and cover letters render as plain text with `whitespace-pre-line`, max width `max-w-prose` (about 65 characters).
- Long words and URLs use `break-words`.

## 4. Spacing and layout

- **Scale:** Tailwind's 4 px scale; use 1, 2, 3, 4, 6, 8 and 12 (4–48 px).
- **Container:** `mx-auto max-w-5xl px-4 sm:px-6`. Forms use `max-w-xl`.
- **Vertical rhythm:**
  - Page padding: `py-8 sm:py-10`
  - Between sections: `space-y-8`
  - Inside cards: `p-4 sm:p-6`
  - Between form fields: `space-y-5`
  - Between list items: `gap-3`
- **Breakpoints:** mobile-first. `sm` (640) adds 2-column filters; `lg` (1024) puts filters in a left sidebar next to results on `/`. Check at 375, 768, 1024 and 1440 px with no horizontal scroll.
- **Targets:** buttons and inputs are at least `h-10` (40 px) with 8 px between targets. Icon-only buttons are 40×40 px with an `aria-label`.
- **Header:** sticky `h-14`, `border-b bg-background/95`. Content gets no extra offset because the header is in normal flow.

## 5. shadcn/ui components

Add them with the shadcn CLI 4.23.0:
`npx shadcn@4.23.0 add button input textarea label field select radio-group card badge alert separator skeleton spinner empty pagination`

| Component | Where |
|---|---|
| `button` | All actions. Variants: `default` (one primary per view), `outline`, `ghost` (nav), `link` |
| `input`, `textarea`, `label` | All forms |
| `field` (`Field`, `FieldLabel`, `FieldDescription`, `FieldError`, `FieldGroup`) | Form structure. `FieldError` gets `state.fieldErrors[name]`, with `aria-invalid` on the control |
| `select` | Filters (work type, employment type), job form enums, applicant status. Uses the `name` prop so it posts with the form |
| `radio-group` | Role picker on sign-up (two large option cards) |
| `card` | Job cards, form panels, dashboard job blocks |
| `badge` | Work type, employment type, job status, application status |
| `alert` | Notice banner (`?notice=`), form-level `formError`, "already applied" / "closed" messages |
| `separator` | Between detail sections and header groups |
| `skeleton` | `loading.tsx` for each route |
| `spinner` | Inside pending submit buttons |
| `empty` | Empty states (no jobs, no applications, no applicants) |
| `pagination` | Prev/Next on `/` (links, so they work without JS) |

The employer dashboard uses native `<details>`/`<summary>` to expand each job. No other components are needed, so leave out dialog, toast/sonner, dropdown-menu, sheet and table.

**Shared app components** (in `src/components/`):
- `Header`
- `Notice`
- `JobCard`
- `JobMeta` (location, work type, employment type, salary)
- `StatusBadge`
- `JobFilters`
- `SubmitButton` (uses `useFormStatus`, disabled while pending, shows a spinner)
- `EmptyState` (a wrapper around `empty`)

## 6. Global patterns

**Forms:**
- Visible label above each field; required fields marked with `*`. Help text sits below the input, and the error sits below the help text in `text-destructive text-sm`, linked with `aria-describedby`.
- Validation runs on submit through the Server Action, and the form keeps the user's input on error.
- When there are field errors, a `formError` Alert at the top summarises them and receives focus.
- The submit button is disabled and shows a spinner while pending ("Posting…", "Sending…").
- Use the right input types: `email`, `password` (with `autocomplete`), `url`, and `number` with `inputMode="numeric"`. Pasting and password managers are allowed.

**Feedback:**
- `?notice=` shows an `Alert` under the header. `applied` uses a green check icon, and `wrong-role` uses an info icon with neutral styling.
- Notices aren't auto-dismissed but include an × link that removes the parameter.

**Loading:**
- Each route has a `loading.tsx` with Skeleton blocks matching the final layout, to avoid layout shift.
- Mutations show the pending button state, with no page-level spinner.

**Errors:**
- `src/app/error.tsx` (client) shows: "Something went wrong" + short text + **Try again** (`reset()`) + **Back to jobs**.
- `not-found.tsx` shows: "Job not found" + **Browse jobs**.
- Each page also uses these two files, so there's no per-page error handling beyond them.

**Accessibility:**
- A visible focus ring on every interactive element.
- A "Skip to content" link as the first element.
- One `h1` per page, with headings in order.
- The status `select` has an `aria-label` such as "Status for Jane Doe".

## 7. Wireframes

### Header (all pages)

```
Signed out
┌──────────────────────────────────────────────────────────────┐
│ ■ JobBoard        Jobs                     [Sign in] [Sign up]│
└──────────────────────────────────────────────────────────────┘
Candidate:  ■ JobBoard   Jobs  Dashboard          Jane D. · [Sign out]
Employer:   ■ JobBoard   Jobs  Dashboard   [+ Post a job]  Sam K. · [Sign out]
```
- The active link is shown in `text-primary` with `font-medium`.
- On mobile, the links wrap onto a second row (fewer than 4 items, so there's no hamburger menu).
- "Sign up" and "Post a job" are the only filled buttons in the header.

### `/`: job listings

```
┌──────────────────────────────────────────────────────────────┐
│ Find your next role                                     (h1) │
│ 15 open jobs                                    (muted, sm)  │
├───────────────┬──────────────────────────────────────────────┤
│ FILTERS       │ ┌──────────────────────────────────────────┐ │
│ Keyword       │ │ Senior React Developer           Remote  │ │
│ [react      ] │ │ Acme Corp · Berlin                       │ │
│ Location      │ │ [Full-time] [Remote]   $80k – $100k      │ │
│ [Berlin     ] │ │                               3 days ago │ │
│ Work type     │ └──────────────────────────────────────────┘ │
│ [Any      ▾]  │ ┌──────────────────────────────────────────┐ │
│ Employment    │ │ Product Designer                          │ │
│ [Any      ▾]  │ │ Globex · Remote                           │ │
│ Min salary    │ │ [Contract] [Hybrid]    From $60k          │ │
│ [$ 50000    ] │ └──────────────────────────────────────────┘ │
│ [Search]      │   … 10 per page                              │
│ Clear filters │                                              │
│               │        [‹ Prev]  Page 1 of 2  [Next ›]       │
└───────────────┴──────────────────────────────────────────────┘
```
- Filters are a GET `<form>`, so they work without JS and the URL is the state.
- Below `lg`, the filters sit above the results in a `muted` panel (2 columns at `sm`).
- The whole card is the link, with its title as the accessible name and hover `bg-accent`.

| State | Design |
|---|---|
| Loading | Header text skeleton, filter panel skeleton, 5 card skeletons (`h-28`) |
| Empty (no jobs at all) | `Empty`: briefcase icon, "No open jobs yet", "Check back soon." |
| Empty (filters) | `Empty`: search icon, "No jobs match your search", "Try fewer filters or a different keyword." + **Clear filters** |
| Error | Global `error.tsx` |
| Bad query params | Ignored silently; the page shows the unfiltered results |

### `/jobs/[id]`: job detail

```
┌──────────────────────────────────────────────────────────────┐
│ ‹ Back to jobs                                               │
│                                                              │
│ Senior React Developer                     (h1)   [Closed]   │
│ Acme Corp · Berlin · Posted 3 days ago                       │
│ [Full-time] [Remote]  $80k – $100k / year                    │
│                                          ┌─────────────────┐ │
│ ─────────────────────────────────────    │ [  Apply now  ] │ │
│ About the role                  (h2)     │ or              │ │
│ Lorem ipsum… (max-w-prose,               │ Applied ·       │ │
│ whitespace-pre-line)                     │ [Reviewed]      │ │
│                                          │ or              │ │
│ ─────────────────────────────────────    │ [View applicants│ │
│ About Acme Corp                 (h2)     │  (4)]           │ │
│ Short description…                       └─────────────────┘ │
│ acme.com ↗                                                   │
└──────────────────────────────────────────────────────────────┘
```
- Layout: a two-column grid at `lg` with the action card sticky on the right; on mobile the action card sits under the meta row.
- Action card variants:
  - Signed out: **Sign in to apply**
  - Candidate: **Apply now**, or "You applied on 12 Nov" + status badge + link to Dashboard
  - Owner: **View applicants (n)**
  - Other employer: no card
  - Closed job: grey Alert "This job is no longer accepting applications."
- The website link opens in a new tab with `rel="noopener noreferrer"` and an external-link icon.

| State | Design |
|---|---|
| Loading | Title, meta and badge skeletons, 3 paragraph skeletons, action card skeleton |
| Not found | `not-found.tsx` |
| Closed | `Closed` badge next to the title + Alert in the action card |
| Error | Global `error.tsx` |

### `/jobs/new`: post a job (employer)

```
Step A: no company yet
┌──────────────────────────────────────┐
│ Set up your company           (h1)  │
│ You need a company before posting.   │
│ Step 1 of 2                          │
│ ┌──────────────────────────────────┐ │
│ │ Company name *                   │ │
│ │ [                              ] │ │
│ │ Website                          │ │
│ │ [https://                      ] │ │
│ │ Description       0 / 1000       │ │
│ │ [                              ] │ │
│ │                [Continue →]      │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘

Step B: post a job
┌──────────────────────────────────────┐
│ Post a job                    (h1)  │
│ Posting as Acme Corp                 │
│ ┌──────────────────────────────────┐ │
│ │ Job title *                      │ │
│ │ [                              ] │ │
│ │ Location *                       │ │
│ │ [e.g. Berlin or Remote         ] │ │
│ │ Work type *      Employment *    │ │
│ │ [Remote    ▾]    [Full-time  ▾]  │ │
│ │ Salary (USD / year, optional)    │ │
│ │ [Min      ]  –  [Max        ]    │ │
│ │ Description *     0 / 5000       │ │
│ │ [                              ] │ │
│ │ [                              ] │ │
│ │                [Post job]        │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```
- "Step 1 of 2" appears only on the company step of a first-time flow.
- Character counters appear on the long textareas (a small client island, or the max length shown as help text).

| State | Design |
|---|---|
| Loading | Card skeleton with 5 field skeletons |
| Validation error | Inline `FieldError` under each field + summary Alert; values kept |
| Name taken | `FieldError` on Company name: "That company name is taken." |
| Pending | Button disabled: "Saving…" / "Posting…" |
| Wrong role / signed out | Redirect (no UI) |

### `/jobs/[id]/apply`: apply (candidate)

```
┌──────────────────────────────────────┐
│ ‹ Back to job                        │
│ Apply for Senior React Developer (h1)│
│ Acme Corp · Berlin                   │
│ ┌──────────────────────────────────┐ │
│ │ Resume URL *                     │ │
│ │ [https://                      ] │ │
│ │ Link to a PDF, Google Drive or   │ │
│ │ LinkedIn profile.                │ │
│ │ Cover letter *    0 / 3000       │ │
│ │ [                              ] │ │
│ │ [                              ] │ │
│ │           [Send application]     │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

| State | Design |
|---|---|
| Loading | Title skeleton + form skeleton |
| Already applied | No form. Alert: "You already applied to this job on 12 Nov." + **Go to dashboard** |
| Job closed | No form. Alert: "This job is no longer accepting applications." + **Back to jobs** |
| Not found | `not-found.tsx` |
| Validation error | Inline field errors + summary Alert; values kept |
| Duplicate race (`P2002`) | Form-level Alert: "You've already applied to this job." |
| Pending | "Sending…" |
| Success | Redirect to `/dashboard?notice=applied` |

### `/dashboard`: candidate view

```
┌──────────────────────────────────────────────────────────────┐
│ ✓ Application sent.                                     [×]  │  ← notice
│ My applications                                         (h1) │
│ 3 applications                                               │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ Senior React Developer                       [Reviewed]  │ │
│ │ Acme Corp · Applied 12 Nov                               │ │
│ ├──────────────────────────────────────────────────────────┤ │
│ │ Product Designer                            [Submitted]  │ │
│ │ Globex · Applied 10 Nov                                  │ │
│ └──────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```
- The rows are a list inside one Card, separated by dividers. Each title links to the job.

| State | Design |
|---|---|
| Loading | Title skeleton + 3 row skeletons |
| Empty | `Empty`: file icon, "You haven't applied to any jobs yet" + **Browse jobs** |
| Error | Global `error.tsx` |

### `/dashboard`: employer view

```
┌──────────────────────────────────────────────────────────────┐
│ Acme Corp                                   [+ Post a job]   │
│ Your jobs · 3                                                │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ ▸ Senior React Developer   [Open]   4 applicants [Close] │ │
│ └──────────────────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ ▾ Product Designer         [Open]   2 applicants [Close] │ │
│ │ ──────────────────────────────────────────────────────── │ │
│ │ Jane Doe · jane@example.com · 12 Nov     [Reviewed  ▾]   │ │
│ │ Resume ↗                                                 │ │
│ │ "I'm excited about…" (cover letter, line-clamp-3,        │ │
│ │  full text in a nested <details> "Read more")            │ │
│ │ ──────────────────────────────────────────────────────── │ │
│ │ John Smith · john@example.com · 11 Nov   [Submitted ▾]   │ │
│ │ …                                                        │ │
│ └──────────────────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ ▸ Data Analyst            [Closed]  0 applicants  [Open] │ │
│ └──────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```
- Each job is a `Card` wrapping `<details>`.
- The `summary` holds the title (linked to the job), the status badge, the applicant count, and an `outline` toggle button ("Close job" or "Reopen job"), which is a tiny form.
- The status `Select` submits on change. The select keeps its new value and is disabled until the action finishes (no optimistic state, per Architecture §5).
- On mobile, the summary row wraps: title on the first line, then badge, count and toggle.

| State | Design |
|---|---|
| Loading | Company title skeleton + 3 job card skeletons |
| No company / no jobs | `Empty`: briefcase icon, "You haven't posted any jobs yet" + **Post a job** |
| Job with no applicants | Inside the expanded card: muted text "No applicants yet." |
| Toggle / status pending | That control is disabled with a spinner |
| Error | Global `error.tsx` |

### `/sign-in` and `/sign-up`

```
┌──────────────────────────────────────┐
│            ■ JobBoard                │
│ ┌──────────────────────────────────┐ │
│ │ Create your account        (h1)  │ │
│ │ I want to…                       │ │
│ │ ┌──────────────┐┌──────────────┐ │ │
│ │ │ ○ Find a job ││ ○ Hire       │ │ │
│ │ │  Candidate   ││  Employer    │ │ │
│ │ └──────────────┘└──────────────┘ │ │
│ │ Name *                           │ │
│ │ [                              ] │ │
│ │ Email *                          │ │
│ │ [                              ] │ │
│ │ Password *                [eye]  │ │
│ │ [                              ] │ │
│ │ At least 8 characters.           │ │
│ │ [        Create account        ] │ │
│ │ Already have an account? Sign in │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘

Sign in: same card, title "Welcome back", Email + Password,
[Sign in], "New here? Create an account".
```
- The card is centred at `max-w-sm`, on a `bg-muted` page.
- The role picker is a `RadioGroup` styled as two selectable cards with lucide `Search` and `Building2` icons. It has a visible label and is required.
- The password field has a show/hide icon button (`aria-label="Show password"`, lucide `Eye`). This is the only client island here.
- The cross-links keep `?next=`.

| State | Design |
|---|---|
| Loading | Card skeleton |
| Validation error | Inline field errors; email and name kept, password cleared |
| Bad credentials | Form-level Alert: "Email or password is incorrect." |
| Email taken | `FieldError` on Email: "An account with this email already exists." + Sign in link |
| Pending | "Creating account…" / "Signing in…" |
| Already signed in | Redirect to `/dashboard` (no UI) |

## 8. Formatting rules

| Value | Format |
|---|---|
| Salary | `$80k – $100k`, `From $80k`, `Up to $100k`, hidden when both are empty. Shown as `/ year` on the detail page only |
| Posted date | Relative: "Today", "Yesterday", "3 days ago", then "12 Nov" after 30 days |
| Applied date | "12 Nov" (or "12 Nov 2025" if not the current year) |
| Enums | `REMOTE`→Remote, `ONSITE`→On-site, `HYBRID`→Hybrid, `FULL_TIME`→Full-time, `PART_TIME`→Part-time, `CONTRACT`→Contract, `INTERNSHIP`→Internship |
| Counts | "1 applicant", "4 applicants", "15 open jobs" (pluralised) |

## 9. Pre-ship UI checklist

- [ ] Text contrast is at least 4.5:1, and the focus ring is visible on every control.
- [ ] No horizontal scroll at 375 px; the filters stack and the header wraps.
- [ ] Every input has a label; errors are linked with `aria-describedby`; the summary receives focus.
- [ ] Badges show text, never just colour.
- [ ] Each route's `loading.tsx` matches the final layout (no layout shift).
- [ ] Every empty state has a message and a next action.
- [ ] Run the `break-ui` skill on JobCard, job detail and the employer dashboard (long titles, long company names, emails without spaces, 0 or many applicants).
