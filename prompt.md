You are working inside my e-commerce application repository. Your job is to fully modernize this project and verify it is production-ready. Work **phase by phase**, in the exact order below. After every phase, run the verification loop (Phase 6) before moving on — never carry broken state forward. If anything fails, fix it before proceeding.

## Phase 0 — Audit the project first

Before changing anything:

1. Detect the stack: framework (Next.js / React / Vite), current package manager, Node version, TypeScript config, Tailwind CSS version, state management (Zustand), styling setup, and monorepo/workspace layout if any.
2. List every dependency with its current version vs. the latest version on the npm registry (`pnpm outdated` / `npm outdated`).
3. Produce a short written migration plan (as `MIGRATION_PLAN.md`) summarizing what will change in each phase, and flag any packages with known breaking changes between the installed and latest major versions.
4. Create a new git branch `chore/modernization` and commit after each completed phase with a clear conventional-commit message.

## Phase 1 — Make pnpm the default package manager

1. Remove `package-lock.json` and/or `yarn.lock`, and delete `node_modules`.
2. Add a `packageManager` field in `package.json` pinned to the latest stable pnpm (check the registry for the current version — do not guess), and enable it via Corepack.
3. Create a proper `.npmrc` for this project (e.g. `engine-strict=true`, and any hoisting settings the framework needs, such as `shamefully-hoist` only if actually required — prefer not to use it).
4. Add an `engines` field for Node and pnpm, and a `preinstall` guard (`npx only-allow pnpm`) so contributors using npm/yarn get a clear error instead of a broken install — this keeps the repo pnpm-first while staying compatible for others via Corepack.
5. If this is a monorepo, set up `pnpm-workspace.yaml` correctly.
6. Update every script, README instruction, Dockerfile, and CI workflow file that references `npm`/`yarn` commands to their pnpm equivalents.
7. Run `pnpm install` and confirm a clean `pnpm-lock.yaml` is generated and the app still starts.

## Phase 2 — Upgrade all dependencies to their latest versions

1. Upgrade **incrementally, not all at once**: framework first (Next.js and React together), then styling, then state/data libraries, then tooling (TypeScript, ESLint, Prettier, testing libs), verifying the build between each group.
2. For each package, check the registry for the actual latest stable version and read the changelog/migration guide for any major-version jump before upgrading. Use official codemods where they exist (e.g. `npx @next/codemod` for Next.js, React codemods for the React upgrade).
3. **Zustand**: upgrade to the latest version and migrate all stores to the current recommended patterns — current create/middleware API, proper TypeScript typing (`create<State>()(...)`), selector usage to avoid unnecessary re-renders, and `persist`/`devtools` middleware updated to their latest signatures. Refactor any deprecated API usage.
4. Resolve every peer-dependency warning properly — do not silence them with `--force` or blanket overrides. Only use `pnpm.overrides` as a last resort, and document why.
5. Remove unused dependencies you discover along the way (verify with `knip` or `depcheck` before removing).

## Phase 3 — Migrate Tailwind CSS to v4

1. First run the official upgrade tool: `npx @tailwindcss/upgrade`. Review every change it makes rather than blindly accepting it.
2. Complete the migration to the v4 CSS-first architecture per the latest Tailwind CSS v4 specification:
   - Replace the old `@tailwind base/components/utilities` directives with `@import "tailwindcss";`
   - Move theme configuration from `tailwind.config.js` into CSS using the `@theme` directive (keep a minimal JS config only if a plugin still requires it).
   - Switch to the new PostCSS plugin (`@tailwindcss/postcss`) or the Vite plugin (`@tailwindcss/vite`) depending on the build setup, and remove `autoprefixer`/`postcss-import` if they're now redundant.
3. Sweep the entire codebase for utilities that were renamed, removed, or whose defaults changed in v4 (e.g. `shadow-sm`→`shadow-xs` style renames, `outline-none`, ring width defaults, border default color, opacity syntax like `bg-black/50`), and fix every occurrence.
4. Check compatibility of every Tailwind-adjacent package — `tailwind-merge`, `tailwindcss-animate`, class-variance-authority, any UI kit (shadcn/ui, Headless UI, Radix-based components) — and upgrade or replace them with their v4-compatible versions.
5. Visually verify key pages (home, product listing, product detail, cart, checkout) render correctly after the migration.

## Phase 4 — Centralize the color system in global CSS

1. **No hardcoded colors anywhere in components.** Scan every file for raw hex codes, `rgb()`/`hsl()` literals, arbitrary Tailwind color values like `bg-[#ff5733]`, and direct palette classes used for brand/UI colors.
2. Define the full color system once in the global stylesheet (`globals.css` / `app.css`) as CSS custom properties inside Tailwind v4's `@theme` block, following the latest Tailwind v4 token conventions, with **semantic names**: `--color-primary`, `--color-secondary`, `--color-accent`, `--color-background`, `--color-foreground`, `--color-muted`, `--color-border`, `--color-destructive`, `--color-success`, `--color-warning`, etc.
3. Replace every hardcoded color usage with the semantic token utility (`bg-primary`, `text-foreground`, `border-border`, ...), so changing a token in global CSS updates the whole application layer.
4. Wire dark mode (if present, or add the structure for it) through the same variables so themes are switched by redefining the custom properties, not by duplicating classes.
5. At the end of this phase, output a table of every token, its value, and where it's consumed.

## Phase 5 — Application-layer consistency check

Go back and forth across the layers — Next.js/React ↔ Tailwind v4 ↔ Zustand ↔ build tooling — and confirm they all agree:

- Server/client component boundaries are still correct after the React/Next upgrade (no client-only APIs in server components, `"use client"` where needed, Zustand stores only consumed client-side or via the recommended per-request store pattern).
- No deprecated framework APIs remain (check for removed/renamed Next.js APIs, async `headers()`/`cookies()`/`params` if applicable to the upgraded version).
- TypeScript passes in strict mode with zero errors.
- ESLint runs clean with the upgraded config format.

## Phase 6 — Verification loop (run after EVERY phase, and fully at the end)

Repeat until everything passes, fixing root causes rather than suppressing errors:

1. `pnpm install` — clean, no unresolved peer warnings.
2. `pnpm typecheck` (or `tsc --noEmit`) — zero errors.
3. `pnpm lint` — zero errors.
4. `pnpm test` — all existing tests pass; if critical flows (cart, checkout, auth, product listing) have no tests, add basic ones.
5. `pnpm build` — production build succeeds with no warnings that indicate breakage.
6. `pnpm start` (production mode) — boot the app, exercise the key routes/pages, and confirm no runtime or hydration errors in the console.
7. If any step fails: diagnose, fix, and re-run the full loop from step 1.

## Final deliverable

When everything is green, give me:

1. A summary of every version change (before → after).
2. All breaking changes you handled and how.
3. The final color token table.
4. Any remaining risks or manual QA steps I should do before deploying to production.

**Rules:** Never use `--force` installs, never disable TypeScript/ESLint checks to make builds pass, never leave the repo in a broken state between commits, and always check the actual latest versions and official migration guides rather than assuming versions from memory.
