---
name: ship
description: Run the pre-deploy checks, commit and push to trigger the Vercel deploy, then walk through the production migration.
argument-hint: "[commit message]"
disable-model-invocation: true
allowed-tools: Bash(git status *) Bash(git ls-files *) Bash(npm ls *) Bash(npm run lint) Bash(npx tsc --noEmit) Bash(npm run build) Bash(git diff *)
---
## Current state
!`git status --short`
!`npm ls prisma @prisma/client --depth=0`

## Checklist (stop and tell me if a check fails)
1. `prisma` and `@prisma/client` above are both 7.x. If either is 8.x or missing, stop.
2. package.json `scripts` has `"postinstall": "prisma generate"`. If not, add it.
3. `git ls-files .env` prints nothing, and .env is not in the status above.
4. `npm run lint` passes with no errors. If it fails, show only the error lines and fix them.
5. `npx tsc --noEmit` passes. If it fails, show only the error lines and fix the cause (no `any` or `@ts-ignore`).
6. `npm run build` passes. If it fails, show only the error lines and use the fix-error skill.
7. Only if checks 1–6 all pass: commit with the message "$ARGUMENTS" (if empty, write a one-line message from the diff), then `git push` (I approve the push).
8. Tell me, in 3 lines: production builds apply migrations automatically (`vercel-build` runs `prisma migrate deploy` when `VERCEL_ENV=production`), so check the build log shows them applied; wait for the deployment to show Ready; then hard-refresh the live URL and add a record. Never run `migrate deploy` by hand against production.
