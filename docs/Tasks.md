# Tasks

## Ready

- [ ] **Rewrite the `new-feature` and `add-field` skills** (`.claude/skills/new-feature/SKILL.md`, `.claude/skills/add-field/SKILL.md`)
  - **Unblocked:** docs/Architecture.md is done (folder layout §2, guards §6, action pattern §5).
  - **Why:** both skills currently generate REST route handlers (`api/<models>/route.ts`) and `"use client"` pages that fetch from them, with no session/role checks and no Zod. That contradicts CLAUDE.md (Server Components read, Server Actions mutate, server-side session + role checks).
  - **Done when** both skills follow what docs/Architecture.md defines:
    - folder structure (where pages, server actions, schemas and components live)
    - auth helpers (session + role check used in every protected page and action)
    - Zod schema + Server Action pattern for mutations, Server Components for reads
    - Prisma client imported only via the `src/lib/prisma.ts` singleton (`@/generated/prisma/client`)
    - still additive-only migrations in `add-field`, and the migrate step still needs my approval
