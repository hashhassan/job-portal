---
name: add-field
description: Add one field to an existing Prisma model and wire it through its API route and page, as an additive schema change only.
argument-hint: "<Model> <field> <Type> <default value | optional> [validation rule]"
disable-model-invocation: true
---
Add a field: $ARGUMENTS

Additive only: the field must have `@default(...)` or be optional (`?`). If I asked to rename or remove a column, stop and tell me the safe path (add the new column, copy the data, switch the code, remove the old column after the deploy).

1. Edit only that model block in prisma/schema.prisma. Run `npx prisma validate`.
2. (I approve this step) `npx prisma migrate dev --name add-<field>`, then `npx prisma generate`. Remind me to restart `npm run dev`.
3. In APP_DIR/api/<models>/route.ts (and [id]/route.ts if it exists): POST and PATCH accept an optional `<field>`, apply the validation rule (400 `{ error }` on failure), and use the default when it is missing. GET stays the same.
4. In the page for <models>: add an input to the form, send it with the POST, and show it in the list. Keep the existing loading, error and empty states.
5. Run `npx tsc --noEmit`. Report the changed files only.
