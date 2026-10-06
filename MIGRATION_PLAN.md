# Migration Plan — Orebi Shopping Modernization

This document outlines the step-by-step modernization strategy for the Orebi Shopping e-commerce repository.

---

## Stack Detection Summary

| Property | Current State | Target State |
| :--- | :--- | :--- |
| **Framework / Build Tool** | React 18 SPA (`react-scripts` / CRA 5.0.1) | React 19 + Vite 6 (`@vitejs/plugin-react`) |
| **Routing** | React Router DOM 6.6.0 | React Router DOM 7.x |
| **Package Manager** | `npm` (with `package-lock.json`) | `pnpm` 11.17.0 (with `pnpm-lock.yaml`) |
| **Node Version** | Node.js `v22.22.0` | Node.js `>=18.0.0` (pinned via `engines`) |
| **State Management** | Redux Toolkit 1.9.2 (`redux-persist` 6.0.0) | Zustand 5.x (`zustand/middleware` `persist`) |
| **Styling** | Tailwind CSS v3.2.4 (JS config) | Tailwind CSS v4.x (CSS `@theme` configuration) |
| **Type Checking** | JavaScript (`.js`, `.jsx`) | Strict TypeScript (`.ts`, `.tsx`, `tsc`) |

---

## Phase-by-Phase Execution Plan

### Phase 0: Audit & Preparation
- Detect repository stack and audit dependencies via npm registry.
- Document major package upgrades and breaking changes.
- Create git branch `chore/modernization`.

### Phase 1: Package Manager Standardisation (`pnpm`)
- Remove `package-lock.json` and `node_modules`.
- Configure `package.json` with `"packageManager": "pnpm@11.17.0"`, `"engines"`, and `"preinstall": "npx only-allow pnpm"`.
- Create `.npmrc` enforcing strict engine constraints.
- Replace `npm`/`yarn` references in scripts and documentation with `pnpm`.
- Execute `pnpm install` to generate `pnpm-lock.yaml`.

### Phase 2: Dependency Upgrades & Modernization
- **Build Tool**: Replace CRA (`react-scripts`) with Vite 6 + `@vitejs/plugin-react`.
- **Framework**: Upgrade `react` and `react-dom` to `^19.0.0`, `react-router-dom` to `^7.1.0`.
- **State Management**: Replace Redux Toolkit with Zustand (`zustand`). Refactor cart and user state into typed `useOrebiStore`.
- **TypeScript**: Add TypeScript (`typescript`, `@types/react`, `@types/react-dom`, `@types/react-slick`, `tsconfig.json`).
- **UI & Libraries**: Upgrade `framer-motion`, `react-icons`, `react-paginate`, `react-slick`, `slick-carousel`, `@testing-library/react`.

### Phase 3: Tailwind CSS v4 Migration
- Configure `@tailwindcss/vite` in Vite configuration.
- Replace legacy `@tailwind` directives in `src/index.css` with `@import "tailwindcss";`.
- Move custom breakpoints (`xs`, `sm`, `sml`, `md`, `mdl`, `lg`, `lgl`, `xl`), font families (`DM Sans`, `Poppins`), container width (`1440px`), and shadows into CSS using the `@theme` directive.
- Remove `tailwind.config.js` and `postcss.config.js`.
- Audit components for renamed utilities (e.g. `bg-black/50`, `shadow-sm` -> `shadow-xs`).

### Phase 4: Color System Centralization
- Define semantic color tokens in `@theme` inside `src/index.css`:
  - `--color-primary`: `#262626`
  - `--color-secondary`: `#767676`
  - `--color-accent`: `#1e88e5`
  - `--color-background`: `#ffffff`
  - `--color-foreground`: `#262626`
  - `--color-muted`: `#6d6d6d`
  - `--color-border`: `#f0f0f0`
  - `--color-destructive`: `#ea2b0f`
- Replace hardcoded hex literals (`#262626`, `#6D6D6D`, `#767676`) across components with semantic utility classes (`text-primary`, `bg-primary`, `text-muted`, etc.).

### Phase 5 & 6: Application Consistency & Verification Loop
- Verify strict TypeScript compilation (`pnpm typecheck`).
- Execute ESLint (`pnpm lint`).
- Run unit tests (`pnpm test`).
- Execute production build (`pnpm build`).
- Verify production preview (`pnpm preview`).

---

## Known Breaking Changes & Mitigations

1. **Create React App (`react-scripts`) deprecation**:
   - *Impact*: CRA is obsolete and incompatible with React 19 / Tailwind v4 Vite tooling.
   - *Fix*: Migrate to Vite with `vite.config.ts`, `index.html` at root, and `@vitejs/plugin-react`.
2. **React 19 Upgrade**:
   - *Impact*: Third-party packages using old peer dependencies or removed `ReactDOM.render`.
   - *Fix*: Verify all UI components use `createRoot` and update `react-icons`, `framer-motion`, `react-slick` to versions supporting React 19.
3. **Redux Toolkit to Zustand Migration**:
   - *Impact*: Components using `useSelector` and `useDispatch` from `react-redux`.
   - *Fix*: Refactor slice state into `useOrebiStore` hook with typed selectors and explicit state actions (`addToCart`, `increaseQuantity`, `decreaseQuantity`, `deleteItem`, `resetCart`).
4. **Tailwind CSS v4 CSS-First Architecture**:
   - *Impact*: `tailwind.config.js` is deprecated in v4; `@tailwind base/components/utilities` directives are replaced.
   - *Fix*: Import `tailwindcss` directly in `src/index.css` and declare tokens/theme extensions inside `@theme`.
