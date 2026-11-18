---
name: fix-error
description: Diagnose and fix build, Prisma, Neon or Vercel errors in this project. Use when a command or Vercel build fails, the live site returns errors, or a generated/prisma import can't be resolved.
---
1. Get the exact error text. If I haven't pasted it, ask for just the failing lines (for Vercel, the red lines from the build log or Runtime Logs, not the whole log).
2. Check it against [errors.md](errors.md). If it matches, apply that fix.
3. If not, find the root cause. Read only the files named in the error or stack trace. Don't hide it with `any`, `@ts-ignore` or try/catch that swallows errors.
4. Re-run the command that failed to prove the fix.
5. Reply in three lines: cause, fix, proof.
