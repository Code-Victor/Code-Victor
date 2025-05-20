# Backend Refactor for Code Quality & Maintainability

As part of our ongoing effort to improve the overall quality of the codebase, I’d like to propose a backend refactor that aligns with modern best practices, improves type safety, and reduces overhead.

## ✅ Framework Migration: Express → Hono

I recommend migrating from our current Express-style routing to [**Hono**](https://hono.dev), a lightweight, TypeScript-first web framework. Hono offers:

* Out-of-the-box TypeScript support (no need for additional typings)
* Blazing-fast performance
* Built-in utilities: middleware, validation, cookie/session helpers, etc.
* A more ergonomic DX with better modularization

Additionally, we can easily generate **OpenAPI documentation** using [hono-openapi](https://github.com/rhinobase/hono-openapi), which would greatly help with API visibility, onboarding new contributors, and enabling tools like Swagger UI or Postman collections.

## 🔀 Cleaner Routing

Instead of manually parsing paths and methods like:

```ts
if (path === "/api/sandbox") {
  if (method === "GET") {
    // ...
  } else if (method === "DELETE") {
    // ...
  }
}
```

We could use Hono's router system, which is much cleaner and modular:

```ts
const sandbox = new Hono().basePath('/api/sandbox')

sandbox.get('/', (c) => c.text('List Sandboxes'))
sandbox.post('/', (c) => c.text('Create Sandbox'))
```

Each API route can live in its own file inside a `routes/` folder, improving organization and reducing cognitive load.

## 🧪 Schema Validation with Zod

Instead of manually validating query parameters and body data:

```ts
if (searchParams.has("id")) {
  // ...
}
```

We could use the [`zodValidator`](https://github.com/honojs/middleware/tree/main/packages/zod-validator) middleware for cleaner, type-safe validation:

```ts
sandbox.post(
  '/',
  zValidator('json', z.object({ id: z.string() })),
  (c) => {
    const { id } = c.req.valid('json')
    // ...
  }
)
```

This drastically reduces boilerplate and catches bugs at compile time.

## 📁 Suggested Folder Structure

To further improve maintainability and scalability, here's a proposed project structure under `src/`:

```bash
src/
├── routes/         # API route handlers (modularized by feature)
├── db/             # Drizzle schema & migration logic
├── middleware/     # Shared middlewares (e.g., auth, logging, zodValidator)
├── services/       # Core services (FileManager, TerminalManager, SSHSocketClient, etc.)
├── utils/          # General-purpose utility functions
```

This structure promotes a clear separation of concerns and makes onboarding new contributors much easier.

## ✨ Bonus: RPC-like Type Safety

Optionally, we could explore using [Hono’s RPC system](https://hono.dev/guides/rpc) to enable type-safe calls from the frontend to the backend, similar to **tRPC**. This would eliminate the need for writing API client wrappers manually and reduce runtime errors due to mismatched request shapes.

---

### Summary

This refactor would:

* Modernize the backend codebase
* Improve TypeScript safety and DX
* Simplify routing and validation
* Enhance modularity and scalability
* Improve onboarding and documentation

Let me know your thoughts — happy to break this down into a migration plan if we want to move forward.

Thanks!
