# Stack Research — Erkin Çelik B2B Marketing Site

**Domain:** Premium B2B corporate marketing website (steel construction, frontend-only milestone, mock data)
**Researched:** 2026-04-25
**Confidence:** HIGH (Next.js 16 docs read directly from `node_modules/next/dist/docs/`; library versions verified via npm registry & official docs)

---

## TL;DR — The Prescriptive Stack

| Layer | Pick | Version |
|-------|------|---------|
| Framework | Next.js (App Router) | 16.2.4 (already installed) |
| React | React + ReactDOM | 19.2.4 (already installed) |
| Compiler | React Compiler (stable in 16) | `babel-plugin-react-compiler` 1.0.0 |
| Styling | Tailwind CSS v4 (PostCSS plugin) | `tailwindcss@^4` + `@tailwindcss/postcss@^4` (already installed) |
| Component primitives | shadcn/ui (Tailwind v4 distribution) | latest CLI (`npx shadcn@latest`) — vendored, no version pinning needed |
| Theming (dark/light) | `next-themes` | ^0.4.6 |
| i18n (TR/EN) | `next-intl` | ^4.4.x |
| Animation | `motion` (the renamed framer-motion) | ^12.x |
| Icons | `lucide-react` | ^0.x latest (shadcn default) |
| Forms | `react-hook-form` + `zod` + `@hookform/resolvers` | ^7.x / ^3.25.x / ^5.x |
| 3D viewer | `three` + `@react-three/fiber` + `@react-three/drei` | ^0.180+ / ^9.6.x / ^10.x |
| Maps | `react-map-gl` (MapLibre subpath) + `maplibre-gl` | ^8.x / ^4.x |
| Carousel | `embla-carousel-react` (via shadcn Carousel component) | ^8.6.x |
| Date utility | `date-fns` | ^4.x |
| Class merger | `clsx` + `tailwind-merge` (via shadcn `cn()` helper) | shadcn default |
| AI chat UI mock | Vercel `ai-elements` (shadcn registry) — UI only, mocked stream | latest CLI |
| E2E testing | `@playwright/test` | ^1.58.x |
| Component testing | `@testing-library/react` + Vitest (optional) | ^16.x / ^3.x |
| Linter | ESLint flat config (Next 16 default) | `eslint-config-next@16.2.4` (already installed) |

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| **Next.js (App Router)** | 16.2.4 | Framework, routing, RSC, image opt | Already installed. Next 16 ships Turbopack stable for dev+build, React Compiler stable, async `params`/`searchParams`/`cookies`/`headers` (breaking from 15), `cacheComponents` (PPR successor), `proxy` replacing `middleware` (Node-only — Edge unsupported), AMP removed, `next lint` removed. Use App Router exclusively (no `pages/`). |
| **React + ReactDOM** | 19.2.4 | UI runtime | Already installed. App Router uses React 19.2 features incl. ViewTransition (`<ViewTransition>`), `useEffectEvent`, `<Activity>` (auto-used by Next when `cacheComponents` is on for state preservation across nav). Server Components are the default. |
| **React Compiler** | `babel-plugin-react-compiler@1.0.0` (stable) | Auto-memoization | Already installed. Stable in Next 16. Enable via `reactCompiler: true` in `next.config.ts` (already on). Removes need for most `useMemo`/`useCallback`/`React.memo`. Note: builds slower because Babel runs alongside SWC/Turbopack. Acceptable trade for this project (build speed not a hot constraint). |
| **TypeScript** | ^5.9 (project) / minimum 5.1 (Next 16) | Type safety | Strict mode per project requirements. Use the `PageProps<'/[lang]/projects/[slug]'>` and `LayoutProps<'/[lang]'>` global type helpers introduced by `next typegen` (Next 15.5+) — required because `params`/`searchParams` are Promises in Next 16. |
| **Tailwind CSS v4** | ^4 (PostCSS-only plugin) | Utility-first styling | Already installed. v4 is **PostCSS-only**: no `tailwind.config.ts` file — configuration lives inside `app/globals.css` via `@theme` and `@import "tailwindcss"`. Native CSS variables, ~10× faster build, OKLCH color support. Pairs perfectly with shadcn/ui's `data-slot` + CSS variable theming model. |

### shadcn/ui Setup (Tailwind v4 flavor)

Use shadcn/ui's **Tailwind v4 distribution** (current default of `npx shadcn@latest init`). Key implications:

- Components are **vendored into `components/ui/*`** — you own them, no npm version to pin.
- Tokens live in `app/globals.css` with `@theme inline { --color-primary: var(--primary); ... }`.
- Each primitive has `data-slot` attributes for granular styling overrides.
- Radix UI primitives (`@radix-ui/react-*`) are pulled in transitively per component — use Radix latest (works with React 19).
- **Use `@base-ui-components/react` only if you need primitives shadcn doesn't ship** — Next.js team uses it internally (visible in Next's own devDependencies), but shadcn covers ~95% of B2B UI needs already.

```bash
npx shadcn@latest init      # creates components.json, components/ui, lib/utils.ts (cn helper)
npx shadcn@latest add button card sheet dialog dropdown-menu form input label select tabs accordion badge separator carousel sonner navigation-menu
```

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| **next-intl** | ^4.4.x | TR/EN i18n with App Router | Default choice in 2026 for Next.js i18n. ~2KB runtime, native Server Component support, middleware/proxy-based locale routing, full TS type-safety on translation keys, ahead-of-time message compilation, SWC plugin. Officially listed in Next 16 docs as a recommended option. **Caveat:** as of v4.4 it's compatible with Next 16, but `'use cache'` integration has rough edges — for a marketing site without `cacheComponents`, this is a non-issue. |
| **next-themes** | ^0.4.6 | Dark/light/system mode toggle | De-facto standard, used by shadcn/ui's official dark-mode docs. Adds `class="dark"` on `<html>` (or `data-theme`), respects `prefers-color-scheme`, no FOUC. Required: `<html suppressHydrationWarning>` on root layout. |
| **motion** (formerly framer-motion) | ^12.x — import from `motion/react` | Animations, scroll-driven, page transitions, micro-interactions | Renamed mid-2025 to "motion"; old `framer-motion` package is frozen. v12 supports React 19 and works with React Compiler auto-memoization. Use `motion/react` for `<motion.div>`-style declarative animation; use `motion` (vanilla) for one-off DOM animations if needed. **For Next.js App Router**: components using `motion/react` need `"use client"`. Pair with React 19's native `<ViewTransition>` for route-level transitions. |
| **lucide-react** | latest (e.g. ^0.470+) | Icon set | shadcn/ui's default icon library. ~1500 consistent stroke icons, fully tree-shakeable, single style — matches an industrial-premium aesthetic. Avoid `react-icons` (mixes styles, larger bundle). |
| **react-hook-form** | ^7.x | Form state | Lightweight, performant, uncontrolled-by-default → fewer re-renders. Built into shadcn `<Form>` component. Works with Server Actions in Next 16. |
| **zod** | ^3.25.x (or v4 if upgrade-tested) | Schema validation | TypeScript-first schema validation. Note: Next.js's own internal `next.js` repo currently uses `zod@3.25.76` (visible in `node_modules/next/package.json` devDependencies). Zod 4 is out but ecosystem (RHF resolver) is stable on 3.25+; pin to ^3.25 for safety unless you're ready to retest. |
| **@hookform/resolvers** | ^5.x | Glue between RHF and Zod | Use `zodResolver` from `@hookform/resolvers/zod`. |
| **three** | ^0.180+ | WebGL engine for 3D viewer | Required peer dep of R3F. |
| **@react-three/fiber** | ^9.6.x | React renderer for three.js | v9 is the React 19 line. Wrap any R3F scene in `next/dynamic({ ssr: false })` because WebGL needs `window`. |
| **@react-three/drei** | ^10.x | R3F helpers (OrbitControls, GLTF loader, Environment, etc.) | v10 is the React 19 / R3F v9 compatible line. Use `<useGLTF>` for the sample steel-structure model, `<OrbitControls>`, `<Environment preset="warehouse">` for industrial lighting. |
| **react-map-gl** | ^8.x (subpath: `react-map-gl/maplibre`) | React wrapper around MapLibre/Mapbox | v8 fully separates Mapbox / MapLibre / Mapbox-legacy via subpath imports. Pick `react-map-gl/maplibre` — no Mapbox token, no licensing cost, free tile providers. Better Next.js fit (no auth provisioning in mock milestone). |
| **maplibre-gl** | ^4.x | Map renderer | Open-source Mapbox GL fork; pair with free tile providers (MapTiler free tier, OSM, Stadia Maps). Wrap map components in `next/dynamic({ ssr: false })` — WebGL requires browser. |
| **embla-carousel-react** | ^8.6.x | Carousel for hero / project gallery | Used internally by shadcn `Carousel` component. Lightweight, dependency-free, great touch precision. Plugins: `embla-carousel-autoplay`, `embla-carousel-fade`. |
| **date-fns** | ^4.x | Date formatting (blog post dates, "started in 2008") | Tree-shakeable, locale-aware (`tr`, `en-US`), works with `next-intl`'s `format.dateTime`. Avoid `moment.js` (deprecated, ~70KB). |
| **clsx** + **tailwind-merge** | shadcn defaults | The `cn()` utility | Comes from `npx shadcn init`. Don't replace. |
| **sonner** | latest (shadcn) | Toast notifications | shadcn's recommended toast library since the original `<Toast>` was deprecated. |
| **@vercel/og** (built into `next/og`) | ships with Next 16 | OG image generation | Use `app/opengraph-image.tsx` route handler for dynamic OG images per project / per blog post. **Note:** Next 16 made `params`/`id` async in OG image functions — see `node_modules/next/dist/docs/.../version-16.md`. |

### AI Chat UI (Phase 6 — mock only)

| Library | Version | Purpose | Why |
|---------|---------|---------|-----|
| **`ai-elements`** (Vercel, shadcn registry) | latest CLI | Pre-built chat UI components (`<Conversation>`, `<Message>`, `<MessageContent>`, `<PromptInput>`, `<Reasoning>`, `<CodeBlock>`) | Built on top of shadcn/ui — same design language, same Tailwind tokens. Components are vendored into your repo, no SDK version lock. Ideal for **mocking a streaming chat** before backend exists: render `parts` arrays manually with a setTimeout-driven generator. |
| **`ai`** (Vercel AI SDK Core) | optional in mock phase | `useChat` hook | Skip until real LLM phase. For mock, write a small `useMockChat()` hook that yields tokens via `setInterval` — keeps the UI contract identical to future `useChat` swap. |

```bash
npx ai-elements@latest add conversation message prompt-input reasoning
```

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| **Playwright** (`@playwright/test`) | E2E + smoke tests | Next 16 has first-class Playwright integration via `next/experimental/testmode/playwright`. Use it for proxy-aware testing (form submissions hitting mock backends). Configure projects for `chromium`, `firefox`, `webkit`. |
| **axe-playwright** | A11y assertions in E2E | Wire into Playwright tests so every page asserts WCAG AA violations = 0 (project requires WCAG AA). |
| **Vitest** + `@testing-library/react` | Optional unit tests | For the steel-weight calculator math, profile lookups. Skip for visual components — rely on Playwright. |
| **MSW** (`msw`) | HTTP mocking | Useful only when stubbing fetch calls in unit tests. Mock data layer is TypeScript files, so MSW is optional. |
| **ESLint flat config** | Linting | Next 16 ships flat config by default (`eslint.config.mjs`). `next lint` was removed — call ESLint CLI directly: `eslint .`. The project's `package.json` already does this. |
| **Prettier** + `prettier-plugin-tailwindcss` | Code formatting | Auto-sorts Tailwind classes per official ordering. Install dev-only. |
| **TypeScript** `^5.9` | Static typing | Run `npx next typegen` once after scaffolding to generate `PageProps`/`LayoutProps` global helpers (required for async params in Next 16). |
| **Sharp** | Image optimization | Already an optional dep of Next 16; ensure it's installed for AVIF/WebP at build time (project requires AVIF/WebP). |

---

## Installation

```bash
# Already installed (do not re-add):
#   next@16.2.4 react@19.2.4 react-dom@19.2.4
#   tailwindcss@^4 @tailwindcss/postcss@^4
#   eslint@^9 eslint-config-next@16.2.4
#   babel-plugin-react-compiler@1.0.0

# === New runtime deps ===
npm install \
  next-intl \
  next-themes \
  motion \
  lucide-react \
  react-hook-form zod @hookform/resolvers \
  three @react-three/fiber @react-three/drei \
  react-map-gl maplibre-gl \
  embla-carousel-react embla-carousel-autoplay \
  date-fns \
  sonner \
  clsx tailwind-merge \
  class-variance-authority

# === Dev deps ===
npm install -D \
  @playwright/test \
  axe-playwright \
  prettier prettier-plugin-tailwindcss \
  @types/three

# === Scaffolders (run once, vendors files) ===
npx shadcn@latest init
npx shadcn@latest add button card sheet dialog dropdown-menu form input label \
  select textarea tabs accordion badge separator carousel sonner navigation-menu \
  table tooltip skeleton scroll-area popover

npx ai-elements@latest add conversation message prompt-input reasoning code-block

# === Type generation (run once after scaffolding routes) ===
npx next typegen

# === Playwright browsers ===
npx playwright install
```

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| **next-intl** | `next-i18next` | Pages Router only — irrelevant here. App Router users should not pick it. |
| **next-intl** | `paraglide-next` | If you want **build-time tree-shaking of unused translation strings** and your translation set grows past ~10MB. For TR/EN with placeholder EN content, overkill. |
| **next-intl** | Vercel's plain `getDictionary` pattern (from official Next docs) | If you want zero deps and only TR/EN with no plurals/dates/numbers. But you'll re-implement `next-intl`'s `format.dateTime`, `format.number`, plural rules — not worth it for a B2B site that needs locale-aware tonnage formatting. |
| **motion** | `react-spring` | If you need physics-based spring chains across 100+ elements (heavy data viz). Your micro-interactions are simpler. |
| **motion** | `gsap` (with `@gsap/react`) | If you need timeline-based, frame-accurate cinematic sequences (e.g., the hero "production process" animation might justify this). **Recommendation: still use `motion` first**; if a single hero sequence demands it, add `gsap` locally for that one component. Don't replace `motion` globally. |
| **react-map-gl + maplibre-gl** | Mapbox GL via `react-map-gl/mapbox` | If a designer demands Mapbox Studio's premium styles. Cost: paid token, monthly map-load fees. **Skip for mock milestone.** |
| **react-map-gl + maplibre-gl** | `react-leaflet` + Leaflet | If you don't need vector tiles and want raster-only OSM. Lighter (~40KB vs MapLibre's ~200KB), no WebGL — runs on more devices. **But:** raster looks dated for a "premium" site, no smooth pan/zoom. Reject for this aesthetic. |
| **react-three-fiber** | Plain `three.js` in a `useEffect` | For a single static GLTF viewer with no React state. R3F overhead is small; stick with R3F for consistency with the rest of the React tree. |
| **shadcn/ui** | Mantine, Chakra UI, MUI | If you want batteries-included, less-customizable. shadcn wins because (a) you own the code, (b) Tailwind v4 native, (c) AI Elements / shadcn-chatbot-kit reuse same primitives. |
| **AI Elements** | Custom-built chat from shadcn primitives | If chat layout is heavily branded and AI Elements' assumptions don't fit. AI Elements is shadcn-on-top so you can fork freely — usually no reason. |
| **Playwright** | Cypress | Only if your team has deep Cypress experience. Playwright is faster, has better Next 16 integration (`next/experimental/testmode/playwright`), supports all 3 engines, parallelizable. |
| **Vitest** | Jest | Vitest is faster, ESM-native, better Vite/Turbopack alignment. Pick Vitest if you write any unit tests. |
| **next-themes** | Custom React Context + `localStorage` + `<html className>` | DIY costs ~50 lines, must handle SSR, FOUC, prefers-color-scheme. `next-themes` is 1KB and battle-tested. Don't reinvent. |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| **`framer-motion`** (the old npm name) | Package is frozen — no longer actively developed; renamed to `motion` in 2025. | `motion` (import from `motion/react`) |
| **`tailwind.config.ts` / `tailwind.config.js`** | Tailwind v4 uses **PostCSS-only configuration**. Tokens live in `app/globals.css` under `@theme`. A config file is silently ignored or breaks. | `app/globals.css` with `@import "tailwindcss"; @theme { ... }` |
| **`middleware.ts` (file convention)** | Renamed to **`proxy.ts`** in Next 16 (per `version-16.md` upgrade guide). Old name still works in 16.2 but is deprecated. | Create `proxy.ts` at project root with named export `proxy(request)` |
| **`edge` runtime in `proxy.ts`** | Next 16 explicitly: *"The `edge` runtime is NOT supported in `proxy`. The `proxy` runtime is `nodejs`, and it cannot be configured."* | Use Node runtime (default). For `next-intl` middleware/proxy, this is fine — it doesn't require Edge. |
| **`export const runtime = 'edge'`** in route handlers / pages | Edge runtime is increasingly limited (no ISR, no full Node APIs, package incompatibilities). For a frontend-only mock B2B marketing site, there's zero benefit. | Default Node runtime. Use static rendering + ISR via `cacheComponents` later if needed. |
| **`next/legacy/image`** | Deprecated in Next 16. | `next/image` (default export from `next/image`) |
| **`images.domains: [...]`** in next.config | Deprecated in Next 16. | `images.remotePatterns: [{ protocol: 'https', hostname: '...' }]` |
| **`next lint` script** | **Removed** in Next 16. `next build` no longer runs linting. | Call ESLint CLI directly: `"lint": "eslint ."` (already correct in `package.json`). |
| **`experimental.ppr: true`** | Removed in Next 16; route segment `experimental_ppr` also gone. | `cacheComponents: true` (top-level config). For this milestone, **leave it off** — you don't need PPR for a marketing site with mock data. |
| **`experimental.dynamicIO`** | Removed in Next 16, renamed to `cacheComponents`. | If/when you need it later: `cacheComponents: true`. |
| **Synchronous `cookies()`, `headers()`, `params`, `searchParams`** | Fully **removed** in Next 16 (was deprecated in 15). All are now Promises. | `const { lang } = await params`. Run `npx @next/codemod@canary upgrade latest` if you have legacy code. |
| **`revalidateTag('foo')`** (single-arg form) | Deprecated in Next 16 — TypeScript will error. | `revalidateTag('foo', 'max')`. Or `updateTag('foo')` in Server Actions for read-your-writes semantics. |
| **`next/amp`, `useAmp`, `amp: true`** | Removed in Next 16. | Don't add. Modern web vitals beat AMP without the constraint. |
| **`serverRuntimeConfig`, `publicRuntimeConfig`** | Removed in Next 16. | Plain `process.env.NEXT_PUBLIC_*` for client; `process.env.*` in Server Components for server. Use `connection()` if you need runtime (not build-time) reads. |
| **CSS-in-JS in Server Components** (`styled-components`, `emotion` non-supporting versions, plain CSS-in-JS) | Server Components don't run JS at runtime — runtime CSS-in-JS doesn't work without a registry shim and adds bundle weight. | Tailwind v4 utility classes + CSS Modules for isolated component CSS. Keep all CSS-in-JS out of this project. |
| **`react-icons`** | Bundles 50K+ icons across 20 different design styles — encourages mixing stroke widths, breaks the industrial-premium consistency. | `lucide-react` only. If you need a specific icon Lucide lacks, copy that single SVG into `components/icons/`. |
| **`moment`** | ~70KB, mutable, deprecated by maintainers. | `date-fns` (tree-shakeable, immutable, locale-aware). Or native `Intl.DateTimeFormat` via `next-intl`'s `format.dateTime`. |
| **Plain `<a>` tags for internal nav** | Loses prefetching, scroll restoration, View Transitions integration. | `next/link` (`<Link>`). Next 16's enhanced routing dedupes shared layouts on prefetch — a free perf win you only get via `<Link>`. |
| **`next/dynamic` with `{ ssr: false }` in a Server Component** | `ssr: false` is a Client Component option — it errors in Server Components. | Move the `dynamic()` call into a Client Component (`"use client"`), then import that wrapper from your Server Component. Required pattern for R3F and MapLibre. |
| **Custom Webpack config** | Next 16 build defaults to **Turbopack**. If `next.config.js` has a `webpack()` block, the build **fails** unless you pass `--webpack`. | Migrate to `turbopack: { ... }` top-level config (no longer `experimental.turbopack`). For this project, you almost certainly don't need any custom bundler config. |
| **Tilde imports in Sass** (`@import '~bootstrap/...'`) | Turbopack does not support `~` prefix. | Plain bare imports: `@import 'bootstrap/...'`. (Project doesn't use Sass anyway — Tailwind v4 covers all styling.) |
| **`scroll-behavior: smooth` (alone) on `<html>` for SPA scroll-to-top** | Next 16 no longer overrides this during navigation, so route changes will *also* smooth-scroll → feels laggy on every click. | Add `data-scroll-behavior="smooth"` attribute on `<html>` to opt back into the old behavior, OR omit `scroll-behavior` entirely and use `<Link scroll>` defaults. |

---

## Stack Patterns by Variant

**If you decide to enable `cacheComponents` later (next milestone):**
- Top-level `cacheComponents: true` in `next.config.ts`
- Mark cacheable Server Components with the `'use cache'` directive
- Use `cacheLife('hours')` / `cacheLife('max')` to scope freshness
- Be aware: navigation will use React `<Activity>` to keep state across routes — design forms accordingly (see `node_modules/next/dist/docs/01-app/02-guides/preserving-ui-state.md`)
- next-intl's `'use cache'` integration has caveats — verify before turning on

**If the 3D viewer becomes a hero feature:**
- Add `@react-three/postprocessing` (^3.x) for bloom on metallic accents
- Add `@react-three/rapier` (^2.x) only if you need physics — for a static steel-structure GLTF viewer, **skip**
- Use `useGLTF.preload('/models/cargo.glb')` in a parent component to start loading early

**If the project gallery exceeds ~200 items:**
- Add `@tanstack/react-table` (^8.x) for headless filtering + sorting + virtualization
- Pair with `@tanstack/react-virtual` for windowing
- For the current ~50 projects, native `<Tabs>` + `Array.filter()` is enough

**If the steel-weight calculator gains a state-machine vibe:**
- Add `zustand` (^5.x) for lightweight global state
- Avoid Redux/RTK — over-engineered for this domain

**If you later add a real backend / CMS:**
- TanStack Query (`@tanstack/react-query` ^5.x) for client data fetching, but prefer Server Components + `fetch()` first
- For form submission, use **Server Actions** (Next 16 default), not API routes

---

## Version Compatibility (verified)

| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| `next@16.2.4` | `react@19.2.4`, `react-dom@19.2.4` | Required peer per `node_modules/next/package.json` (`react: ^18.2.0 \|\| ^19.0.0`). React 19 is the recommended pairing for App Router on 16. |
| `next@16.2.4` | `tailwindcss@^4` + `@tailwindcss/postcss@^4` | Documented installation path in `node_modules/next/dist/docs/01-app/01-getting-started/11-css.md`. |
| `next@16.2.4` | `babel-plugin-react-compiler@1.0.0` | Set `reactCompiler: true` in `next.config.ts` (already done). Stable in Next 16. |
| `@react-three/fiber@9.6.x` | `react@19`, `three@^0.180+` | v9 is the React 19 line per `pmndrs/react-three-fiber` releases. v8 is React 18-only. |
| `@react-three/drei@10.x` | `@react-three/fiber@9.x`, `react@19` | v10 is the React 19 / R3F v9 line. |
| `react-map-gl@8.x` | `maplibre-gl@^4` (via `react-map-gl/maplibre` subpath) | v8 separates Mapbox / MapLibre / legacy via subpaths — pick one explicitly. |
| `next-intl@4.4+` | `next@16.x` (App Router) | Officially listed in Next 16 i18n docs. |
| `next-themes@0.4.6` | `next@16.x`, `react@19` | Stable, framework-agnostic. Requires `<html suppressHydrationWarning>` on root. |
| `motion@12.x` (was `framer-motion`) | `react@19`, `next@16` | Import from `motion/react`. The old `framer-motion@12` package is also fine but frozen — choose `motion`. |
| `@playwright/test@1.58+` | `next@16.x` | Listed as optional peer in Next 16 (`peerDependencies` in `node_modules/next/package.json`). Next 16 ships `next/experimental/testmode/playwright` for proxy-aware testing. |
| `react-hook-form@7.x` | `react@19` | Fully compatible. No React 19 issues. |
| `zod@3.25.x` | `@hookform/resolvers@5.x` | Pinned conservatively; Next.js's own internal codebase uses `zod@3.25.76`. |
| `embla-carousel-react@8.6.x` | `react@19`, shadcn `Carousel` component | shadcn `Carousel` component pulls v8 — keep aligned. |

---

## Next.js 16 Specifics That Affect This Project

These are not optional curiosities — the AGENTS.md warning ("This is NOT the Next.js you know") refers to these. Roadmap phases must respect them:

1. **`params` and `searchParams` are Promises.** Every page/layout must `await params`. Run `npx next typegen` to get `PageProps<'/[lang]/projects/[slug]'>` global helpers. Without these, TypeScript will not narrow your route param types.

2. **`cookies()`, `headers()`, `draftMode()` are async.** Synchronous access removed entirely. (Mock milestone barely uses these — only relevant if you add a cookie-banner state read in a Server Component.)

3. **`middleware.ts` → `proxy.ts`.** For `next-intl` locale routing, create `proxy.ts` at the project root, not `middleware.ts`. Use `next-intl`'s `createMiddleware` adapter (the function name still maps).

4. **No Edge runtime in `proxy`.** Whatever you put in `proxy.ts` runs on Node. This is fine for `next-intl`.

5. **`next.config.ts` lives at the project root** with a `turbopack: { ... }` block (no longer `experimental.turbopack`).

6. **Image config breaking changes:**
   - `images.qualities` defaults to `[75]` only — if your hero images need quality 90 for the metallic detail, set `images: { qualities: [60, 75, 90] }`.
   - `images.minimumCacheTTL` is now 4 hours (up from 60s).
   - `images.imageSizes` no longer includes `16`.
   - Local images with query strings (`/hero.jpg?v=2`) require `images.localPatterns` allowlist.
   - `images.domains` is deprecated — use `images.remotePatterns`.

7. **Parallel routes require `default.js`.** If you build a modal-via-parallel-route pattern (`@modal/...`), every slot needs a `default.tsx` that returns `null` or calls `notFound()`.

8. **`opengraph-image.tsx` and `sitemap.tsx`** receive `params`/`id` as Promises — `await` them.

9. **OG metadata + `next/og`** is the standard for dynamic project-specific Open Graph cards (project requires Schema.org + OG).

10. **Concurrent dev/build:** `next dev` writes to `.next/dev` (not `.next/`), so tracing commands and any custom scripts that read `.next/` need updating.

---

## Sources

### Primary (HIGH confidence — read directly from `node_modules/next/dist/docs/`)

- `node_modules/next/dist/docs/01-app/02-guides/upgrading/version-16.md` — full v16 breaking changes (Turbopack default, async params, proxy rename, image config, AMP removal, `next lint` removal, runtime config removal, etc.)
- `node_modules/next/dist/docs/01-app/01-getting-started/11-css.md` — official Tailwind v4 setup pattern (PostCSS-only, no config file)
- `node_modules/next/dist/docs/01-app/02-guides/internationalization.md` — official i18n routing patterns; lists `next-intl` as a recommended option
- `node_modules/next/dist/docs/01-app/02-guides/testing/playwright.md` — official Playwright setup
- `node_modules/next/dist/docs/01-app/02-guides/css-in-js.md` — supported CSS-in-JS libraries (mostly informational; we're not using any)
- `node_modules/next/dist/docs/01-app/03-api-reference/05-config/01-next-config-js/cacheComponents.md` — `cacheComponents` flag reference
- `node_modules/next/dist/docs/01-app/03-api-reference/07-edge.md` — Edge runtime caveats
- `node_modules/next/package.json` — exact peer deps (React `^19.0.0`, Playwright `^1.51.1`, optional Sharp), confirmed Next 16.2.4 + Babel React Compiler 0.0.0-experimental-1371fcb-20260227 in dev deps (note the dated experimental snapshot — stable plugin is `1.0.0` per the docs)

### Secondary (MEDIUM confidence — verified via official docs / npm registry)

- [shadcn/ui — Tailwind v4 docs](https://ui.shadcn.com/docs/tailwind-v4) — current default install path is Tailwind v4 + React 19
- [shadcn/ui — Next.js install](https://ui.shadcn.com/docs/installation/next) — App Router + components.json
- [next-intl GitHub](https://github.com/amannn/next-intl) — v4.4+ Next 16 compat, ~2KB runtime
- [Aurora Scharff — Implementing Next.js 16 'use cache' with next-intl](https://aurorascharff.no/posts/implementing-nextjs-16-use-cache-with-next-intl-internationalization/) — caveats around `cacheComponents`
- [Motion — Upgrade guide](https://motion.dev/docs/react-upgrade-guide) — `framer-motion` → `motion/react` rename, v12 React 19 support
- [react-three-fiber installation](https://r3f.docs.pmnd.rs/getting-started/installation) — v9 = React 19 line
- [@react-three/fiber npm](https://www.npmjs.com/package/@react-three/fiber) — current 9.6.0
- [react-map-gl What's New](https://visgl.github.io/react-map-gl/docs/whats-new) — v8 subpath split (Mapbox / MapLibre / legacy)
- [next-themes GitHub](https://github.com/pacocoursey/next-themes) — v0.4.6 latest
- [Vercel AI Elements](https://elements.ai-sdk.dev/) — shadcn-based AI chat primitives
- [Vercel AI Elements GitHub](https://github.com/vercel/ai-elements) — installable via shadcn registry CLI
- [Embla Carousel](https://www.npmjs.com/package/embla-carousel-react) — v8.6.x current

### Tertiary (LOW confidence — community / blog posts, used only to corroborate primary sources)

- [LogRocket — Best React animation libraries 2026](https://blog.logrocket.com/best-react-animation-libraries/) — corroborates motion's dominance
- [Frontend Hero — Best Icon Libraries 2026](https://frontend-hero.com/best-icon-libraries-react) — corroborates lucide-react as the React standard
- [DEV — Next.js 16 App Router Complete Guide 2026](https://dev.to/getcraftly/nextjs-16-app-router-the-complete-guide-for-2026-2hi3) — corroborates Next 16 stack patterns
- [intlpull — next-intl tutorial 2026](https://intlpull.com/blog/next-intl-complete-guide-2026) — community confirmation of next-intl as 2026 default
- [DEV — RHF + Zod 2026 guide](https://dev.to/marufrahmanlive/react-hook-form-with-zod-complete-guide-for-2026-1em1) — pattern reference

---

*Stack research for: B2B steel construction marketing site (frontend-only, mock data milestone)*
*Researched: 2026-04-25*
