# Known errors in this stack

| Symptom | Cause | Fix |
|---|---|---|
| `Could not find Prisma Schema`, or npx asks "Ok to proceed? (y)" | Command run outside the project folder | `cd` to the folder with package.json and prisma/, then re-run |
| `Cannot find module '.../generated/prisma'` (ts2307) | Prisma 7's generated folder has no index file | Import path must end in `/client`; then restart the TS server |
| `prisma` has no `generate`/`migrate dev`, ignores schema.prisma, or `npm ls prisma` shows 8.x | Prisma 8 installed through the `latest` tag | `npm install -D -E prisma@7.10.1` and `npm install -E @prisma/client@7.10.1 @prisma/adapter-neon@7.10.1` (exact pins, see docs/Architecture.md) |
| `Using engine type "client" requires either "adapter" or "accelerateUrl"` | PrismaClient created without the driver adapter | Use the shared client in src/lib/prisma.ts (`PrismaNeon` adapter from `@prisma/adapter-neon`); scripts like the seed import it too |
| Prisma refuses to run because of the Node.js version | Prisma 7 needs Node 20.19+, 22.12+ or 24+ | Install Node 22 or 24 LTS; on Vercel set Node.js Version in Project Settings |
| Route or page returns 404, or edits are ignored | Both app/ and src/app exist; Next.js uses app/ | Put files in the folder that has layout.tsx (APP_DIR) |
| `Route segment config "dynamic" is not compatible with nextConfig.cacheComponents` | `export const dynamic` used with Cache Components on | Remove the export; in a GET handler that reads the DB, call `await connection()` from "next/server" first |
| List doesn't show new rows, or GET returns old data after deploy | GET handler was prerendered or cached | Same as above: `await connection()` at the top of GET (Cache Components), or the installed version's no-cache option |
| Vercel build fails with a Prisma client error | Generated client is gitignored and not built on Vercel | Add `"postinstall": "prisma generate"` to scripts, push, redeploy |
| Live site builds, but reads or writes fail (500) | DATABASE_URL missing or wrong in Vercel | Set it to the POOLED Neon string in Vercel env vars, redeploy, check Runtime Logs |
| Neon "too many connections" from the live site | Direct (non-pooled) string used on Vercel | Use the pooled string (host contains `-pooler`) in Vercel |
| New Vercel env var has no effect | Env vars apply only to new deployments | Redeploy (push a commit or Redeploy in Vercel) |
| New field works locally, not live | Migration not applied, or deployment not Ready yet | Production builds run `prisma migrate deploy` in `vercel-build`: check the build log for applied migrations and that `VERCEL_ENV` was production. Preview builds never migrate |
| Prisma CLI can't find DIRECT_URL, or can't connect locally | .env in the wrong folder, misspelled name, bad quotes, missing `import "dotenv/config"` in prisma.config.ts, or pooled string used for DIRECT_URL | .env next to package.json with `DATABASE_URL="<pooled>"` and `DIRECT_URL="<direct>"` (Neon `dev` branch) |
| `CONFLICT (add/add): Merge conflict in README.md` on first push | Remote repo already had a README | `git checkout --ours README.md`, `git add README.md`, `git commit -m "merge old repo"`, `git push -u origin main` |
| `credential-manager-core is not a git command` | Harmless Git for Windows warning | Ignore |
