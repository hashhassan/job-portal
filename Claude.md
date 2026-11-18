PROJECT: Mini Job Board. Employers create a company and post jobs. Candidates browse, search, and apply.
PAGES: / (job listings + search/filter), /jobs/[id] (detail), /jobs/new (post a job),
/jobs/[id]/apply (apply form), /dashboard (role-based applications dashboard), plus /sign-in and /sign-up.

STACK: Next.js App Router + TypeScript (src/ dir), Tailwind + shadcn/ui, Prisma ORM v7,
Neon Postgres, Better Auth (email + password), Zod for validation, deployed on Vercel.

PRISMA 7 RULES:
- Always install prisma@7 / @prisma/client@7. Never install Prisma 8.
- Generator is "prisma-client" with output to src/generated/prisma.
- Import PrismaClient from "@/generated/prisma/client", never from "@prisma/client".
- Use a driver adapter. The DB URL lives in the Prisma config file, not in schema.prisma.
- One singleton client in src/lib/prisma.ts.

CODE RULES:
- Server Components for reading data, Server Actions for mutations.
- Every protected page and server action checks the session and role on the server.
- User roles: CANDIDATE or EMPLOYER.
- Keep it simple: no features beyond what I ask; ask before adding new dependencies.
- After each task run `npm run lint` and `npx tsc --noEmit` and fix errors.