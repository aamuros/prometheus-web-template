# Conventions

## Names and ownership

Use kebab-case file names, PascalCase React components and types, and camelCase functions and variables. Name features for the application's domain rather than generic architectural layers. Route files compose features; shared components and utilities must have demonstrated reuse. Keep related functionality together and prefer direct imports over barrel files.

`src/` owns browser behavior, `worker/` owns server behavior, and `shared/` owns small environment-independent contracts. Do not import server code into frontend modules, even to reuse configuration. The `@/` alias always refers to `src/`; shared contracts use an explicit relative import.

## TypeScript and API patterns

Keep strict checks enabled. Use `unknown` for external data and narrow or validate it; avoid unchecked `as` casts, `any`, and non-null assertions. Prefer functions and explicit types at public boundaries. Use `import type` for type-only dependencies. Library declarations use `skipLibCheck`; application code remains fully checked in separate frontend, Worker, and tooling passes.

Use resource-oriented `/api/<resource>` URLs and normal HTTP verbs. Keep handlers thin and return JSON with accurate status codes. Successful response shapes may be resource-specific; errors use `{ error: string }`. Never include secrets, raw provider errors, or stack traces in responses. Throw `HTTPException` only with an intentional status; its message is not exposed by the global handler. Add explicit method handling where a route needs a 405 response and `Allow` header.

Validate request data on the server and enforce authorization there when access control is added. Do not rely on frontend validation or shared TypeScript types for security. No global CORS middleware is needed for same-origin requests.

## Errors, UI, and logging

Use the root router boundary for unexpected route loading and rendering errors. Keep expected business errors near the interaction, with accessible messages. Use semantic HTML, visible keyboard focus, and named controls. Reuse semantic Tailwind tokens rather than hardcoding arbitrary colors. Add shadcn components individually when needed, for example `pnpm dlx shadcn@4.21.4 add button`; review the resulting source and dependencies, pin any added packages, and test the production CSP. A component that uses inline styles may need a deliberate CSP adjustment.

Request logs are intentionally minimal. Never log credentials, authorization headers, full bodies, sensitive query strings, or exception messages by default. When diagnostics are required, define what may be recorded and its retention. Avoid exception details in user-facing error views.

## Testing and verification

Run the smallest relevant check while working, then `pnpm check` before merging or deploying. Add tests for meaningful business behavior, failure paths, permissions, and regressions. Prefer accessible queries in frontend tests; API tests should assert status, payload, and relevant headers. Keep the base tests independent of databases and external accounts.

Vitest uses Node for Hono and jsdom for frontend tests. These verify handlers and React behavior, not every Cloudflare binding. After changing deployment configuration or adding runtime-specific integrations, run `pnpm preview` and check the complete flow in the local Workers runtime. Browser/E2E tooling is optional and should be added when real workflows justify it.

## Dependencies and changes

Use the pinned pnpm version and commit the lockfile. CI installs with `--frozen-lockfile`. pnpm settings live in `pnpm-workspace.yaml` even though this repository contains only one application. Exact versions, strict peer checks, and the small explicit dependency-build allowlist make installation predictable. Do not approve additional dependency build scripts without inspecting why they are required.

Add a library only for a current requirement. Read its official documentation, check Node/Worker compatibility and peer requirements, and pin the tested version. Use `pnpm audit` when adding or updating dependencies. CI fails on high or critical advisories. Currently `sharp` is overridden to 0.35.5 to fix a vulnerability in Cloudflare development tooling; remove the override when upstream resolves a patched version and verification remains clean.

When upgrading Node, TypeScript, Vite, Cloudflare tooling, or test infrastructure, run all checks and local preview from a clean install. In particular, do not upgrade TypeScript beyond the range supported by TypeScript ESLint. Update the compatibility date deliberately and verify runtime changes. Keep business features separate from foundation changes, avoid opportunistic refactoring, and document decisions only when their rationale will help future maintainers.
