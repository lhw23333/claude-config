---
name: contract
description: Expand spec Zod schemas into full contracts (tRPC procedures, OpenAPI schemas, or GraphQL SDL) and generate TypeScript types. Use after /spec, before TDD.
---

# /contract — Schema → Contract Expansion

Take the Zod schemas produced by `/spec` and expand them into the transport layer's contract format (tRPC / OpenAPI / GraphQL), then derive runtime validators and TypeScript types.

## When to use

- After `/spec` has produced `src/schemas/*.schema.ts` skeletons
- When adding new API endpoints
- When cross-service contracts need to be synchronized

## Detect transport layer

Check project structure in order:

1. **tRPC** — `@trpc/server` in package.json → generate tRPC routers
2. **OpenAPI** — `openapi.json|yaml` exists or `@hono/zod-openapi` / `zod-to-openapi` installed → generate OpenAPI spec
3. **GraphQL** — `graphql` + `type-graphql` or `@graphql-tools` → generate SDL
4. **REST (plain)** — nothing specific → output typed Hono/Express handlers with Zod validation middleware

## Steps

### Step 1: Identify schemas needing expansion

```bash
grep -l "@spec" src/schemas/
```

For each schema file, list which `z.object` definitions are candidates for contracts (typically request/response pairs).

### Step 2: Generate contract artifacts

**tRPC example:**
```typescript
import { z } from "zod";
import { router, publicProcedure } from "@/trpc";
import { FeatureInputSchema, FeatureOutputSchema } from "@/schemas/feature.schema";

/** @spec NNN */
export const featureRouter = router({
  create: publicProcedure
    .input(FeatureInputSchema)
    .output(FeatureOutputSchema)
    .mutation(async ({ input }) => {
      throw new Error("not implemented");
    }),
});
```

**OpenAPI example (zod-to-openapi):**
```typescript
import { OpenAPIRegistry } from "@asteasolutions/zod-to-openapi";
import { FeatureInputSchema, FeatureOutputSchema } from "@/schemas/feature.schema";

const registry = new OpenAPIRegistry();

/** @spec NNN */
registry.registerPath({
  method: "post",
  path: "/feature",
  request: { body: { content: { "application/json": { schema: FeatureInputSchema } } } },
  responses: { 200: { description: "OK", content: { "application/json": { schema: FeatureOutputSchema } } } },
});
```

### Step 3: Generate client types (if applicable)

- **tRPC**: types are inferred automatically, no codegen needed
- **OpenAPI**: run `npx openapi-typescript openapi.json -o src/generated/api.ts`
- **GraphQL**: run `npx graphql-codegen`

### Step 4: Verify

```bash
npx tsc --noEmit
```

### Step 5: Output

```
Contract generated:
  🔌 Transport: tRPC | OpenAPI | GraphQL | REST
  📁 Files:
     - src/server/routers/<feature>.ts
     - (generated) src/generated/api.ts

Next: proceed to TDD — tests from /spec are still todo, implement them RED-GREEN-REFACTOR.
```

## Rules

- Every contract artifact MUST import from `src/schemas/*.schema.ts`, never redefine Zod types.
- Every exported route MUST have `/** @spec NNN */` tag.
- Runtime validation is non-negotiable: Zod must parse both input and output, even in trusted paths.
- Do NOT implement business logic — only the contract shape.
