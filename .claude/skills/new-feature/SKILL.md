---
name: new-feature
description: Build one full feature slice (Prisma model, migration, API route, page) in this Next.js + Prisma 7 + Neon app.
argument-hint: "<Model> <field:Type ...> [crud]"
disable-model-invocation: true
---
Build ONE feature slice for: $ARGUMENTS

Argument format: `Product name:String price:Float note:String? crud`. A `?` makes a field optional. `crud` adds edit and delete.
If field types or validation rules are unclear, ask me ONE short question, then continue.

Do the steps in order. Stop at the first failure and show only the error lines. Don't read files these steps don't need.

1. **Schema**: append the model to prisma/schema.prisma with `id Int @id @default(autoincrement())`, the fields, and `createdAt DateTime @default(now())`. Booleans get `@default(false)`. Run `npx prisma validate`.
2. **Migrate** (I approve it; it changes the shared Neon DB): `npx prisma migrate dev --name add-<models>`, then `npx prisma generate`.
3. **API**: create APP_DIR/api/<models>/route.ts by copying the structure of the newest existing route handler.
   - GET: all rows, ordered by createdAt desc.
   - POST: validate the body (required strings non-empty after trim, numbers finite); return 400 `{ error }` if invalid, otherwise 201 with the created row.
   - If `crud`: also APP_DIR/api/<models>/[id]/route.ts with GET one, PATCH (only the fields sent, same validation) and DELETE. Return 400 for a non-numeric id and 404 if not found. Use the dynamic-params pattern of the installed Next.js version.
4. **Check**: `npx tsc --noEmit`. With the dev server running, `curl -s http://localhost:3000/api/<models>` must return JSON (`[]` is fine).
5. **Page**: create APP_DIR/<models>/page.tsx as a "use client" component: list from GET, an add form that POSTs and refreshes, loading, error and empty states, Tailwind, `max-w-md mx-auto`, one file, no new libraries. If `crud`, add edit and delete buttons.
6. **Report**: files changed, plus the curl output. Nothing else.
