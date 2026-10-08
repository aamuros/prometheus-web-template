# Template verification

Verified on 2026-10-08 with Node.js 24.19.0 and pnpm 12.10.1 on macOS arm64.

## Independent application check

Copied only the 40 source, configuration, documentation, and lockfile files into a fresh temporary directory. Neither `node_modules` nor build artifacts were copied. Installed from the lockfile there, ran all checks, and exercised development and production preview from that copy. This report was added afterward, bringing the delivered source/configuration file count to 41.

The frozen installation used pnpm's normal package store cache and preserved the lockfile byte-for-byte. There are no dependencies on the original application directory or template repository. No database, external service configuration, or application credentials were supplied. No cloud deployment was performed.

## Results

| Check                                                   | Result and evidence                                                                                                                                                                                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `pnpm install --frozen-lockfile --reporter=append-only` | Passed in the clean copy; exact direct versions and unchanged lockfile verified.                                                                                                                                                     |
| `pnpm typecheck`                                        | Passed frontend, Cloudflare Worker, and tooling/test configurations.                                                                                                                                                                 |
| `pnpm lint`                                             | Passed with zero permitted warnings; browser imports of Worker modules are prohibited.                                                                                                                                               |
| `pnpm format:check`                                     | Passed.                                                                                                                                                                                                                              |
| `pnpm test`                                             | 14 tests passed in 2 files: 8 API cases and 6 frontend cases.                                                                                                                                                                        |
| `pnpm build`                                            | Passed; separate client and Worker output produced by the official Cloudflare plugin.                                                                                                                                                |
| `pnpm check`                                            | Passed in the isolated copy, including all five checks above.                                                                                                                                                                        |
| `pnpm audit --audit-level=high`                         | Passed; reported no known vulnerabilities after the patched `sharp` override.                                                                                                                                                        |
| `pnpm dev --port 5175 --strictPort`                     | Started locally; Chromium rendered **Application ready** and **API connected**. The actual browser request to `/api/health` returned 200.                                                                                            |
| Development SPA/API routing                             | Direct `/missing` navigation rendered **Page not found**. API navigation with `Sec-Fetch-Mode: navigate` returned health JSON 200 and unknown-route JSON 404.                                                                        |
| `pnpm preview --port 4175 --strictPort`                 | Rebuilt and started the production application in local workerd. Chromium rendered the home page and received health JSON 200.                                                                                                       |
| Production SPA fallback                                 | Direct `/missing` navigation rendered the not-found view; clicking **Return home** restored the working home page without a full navigation.                                                                                         |
| Production API routing                                  | Direct health navigation returned `200 {"status":"ok"}`; unknown API navigation returned `404 {"error":"Not found"}`. `POST /api/health` returned 405 with `Allow: GET, HEAD`.                                                       |
| Production security headers                             | Static HTML included CSP, nosniff, frame denial, no-referrer, and restricted permissions. API responses included security headers and `Cache-Control: no-store`.                                                                     |
| Production browser console                              | Zero errors and zero warnings, including no CSP violations.                                                                                                                                                                          |
| Repository hygiene                                      | Environment/secret files, dependencies, and build artifacts are ignored; example files remain tracked. No absolute development paths, local package links, account identifiers, or credentials are configured in application source. |

API tests cover health payload and headers, HEAD, unsupported methods, missing routes, safe unexpected errors, deliberate HTTP error statuses, and redacted request logs. Frontend tests cover the actual health contract, not-found handling, failed requests, invalid payloads, network failures, and rendering errors. Browser verification used an external Playwright CLI; Playwright is not a project dependency and no E2E framework is installed.

## Failures found and resolved

- Initial strict typecheck rejected an explicit `undefined` router history; the router now defaults to a browser history and accepts an explicit memory history for tests.
- Initial React refresh lint failures were resolved by exporting the route components alongside the explicitly allowed `Route` export.
- The initial dependency audit found a high-severity advisory in transitive `sharp` 0.35.4. The pnpm override pins patched 0.35.5; subsequent builds, preview, and audits passed.
- The environment initially lacked the `pnpm` shim; Corepack was enabled. This is included in the setup instructions.
- A sandboxed build could not write Wrangler's user-level diagnostic log, and an isolated audit encountered sandbox DNS restrictions. Authorized runs outside that sandbox restriction passed. These were execution-environment issues, not application failures.

No tests or mandatory checks remain failing. The browser and the development/preview processes created for verification were stopped afterward.

## Installed versions

Runtime dependencies: React/React DOM 19.3.0, TanStack Router 1.170.41, Hono 4.13.13, clsx 2.1.1, and tailwind-merge 3.7.0.

Build/runtime tooling: Vite 8.3.3, React plugin 6.1.2, Cloudflare Vite plugin 1.63.0, Wrangler 4.148.0, Workers types 5.20261008.1, and Tailwind/Vite integration 4.3.3.

Verification tooling: TypeScript 6.0.3, ESLint 10.12.0, TypeScript ESLint 8.71.1, Prettier 3.9.9, Vitest 5.0.3, Testing Library, jsdom, and the pinned React ESLint plugins and type packages. All 30 direct packages and the transitive dependency graph are recorded in `package.json` and `pnpm-lock.yaml`. shadcn/ui configuration and utilities are provided without installing an unused component collection.

## Files delivered

```text
.dev.vars.example
.env.example
.github/workflows/ci.yml
.github/workflows/deploy.yml
.gitignore
.node-version
.prettierignore
.prettierrc.json
README.md
components.json
docs/architecture.md
docs/conventions.md
docs/verification.md
eslint.config.js
index.html
package.json
pnpm-lock.yaml
pnpm-workspace.yaml
public/_headers
shared/api.ts
src/app/main.tsx
src/app/router.ts
src/components/not-found.tsx
src/components/route-error.tsx
src/lib/api.ts
src/lib/utils.ts
src/routes/index.tsx
src/routes/root.tsx
src/styles/globals.css
tests/api.test.ts
tests/frontend.test.tsx
tests/setup.ts
tsconfig.app.json
tsconfig.json
tsconfig.tools.json
tsconfig.worker.json
vite.config.ts
vitest.config.ts
worker/app.ts
worker/index.ts
wrangler.jsonc
```

## Unverified and intentionally absent

- Cloudflare cloud upload, custom domains, account permissions, and deployment credentials were not tested. No production systems were accessed or modified.
- GitHub Actions workflows were authored but have not executed on GitHub. Linux CI and the manual deployment workflow still need their first remote run.
- This local Git repository has no configured GitHub remote. Publishing it and enabling GitHub's **Template repository** setting remain owner actions; the README documents them.
- Authentication, authorization, persistence, backups, external integrations, and business features are intentionally absent. Applications handling protected information must add and validate their safeguards before production.
- Production CSP must be reconsidered when adding components with inline styles or external resources. Node-only API unit tests do not replace runtime tests for future Cloudflare bindings.
- The first frozen install reused the normal pnpm package cache; a network-cold installation and a live GitHub **Use this template** operation were not separately tested.

The requested local template implementation and independent verification are complete. Remote publication, template metadata, CI execution, and cloud deployment remain unverified.
