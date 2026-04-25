# Architecture Research

**Domain:** Next.js 16 (App Router) marketing site with engineering tools, mock AI chat, mock customer portal — frontend-only milestone with mock data layer
**Researched:** 2026-04-25
**Confidence:** HIGH for Next.js 16 conventions (verified against `node_modules/next/dist/docs/`); MEDIUM for theme/animation choices (community patterns)

## Standard Architecture

### System Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                         BROWSER (Client)                          │
│                                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐    │
│  │  Marketing   │  │  Tools       │  │  Portal Mock         │    │
│  │  pages (RSC) │  │  (Client)    │  │  (Client)            │    │
│  │  + leaf      │  │  Calculator  │  │  Login form,         │    │
│  │  Client comp │  │  Filters     │  │  Dashboard cards     │    │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘    │
│         │                 │                     │                 │
└─────────┼─────────────────┼─────────────────────┼─────────────────┘
          │ RSC payload     │ next/dynamic        │ no-SSR islands
          │                 │ (3D, map)           │
┌─────────┼─────────────────┼─────────────────────┼─────────────────┐
│         ▼                 ▼                     ▼                 │
│                    NEXT.JS 16 SERVER (RSC)                        │
│                                                                    │
│  app/[lang]/(marketing) │ app/[lang]/(portal) │ app/[lang]/(legal)│
│  ───────────────────────┴─────────────────────┴───────────────── │
│                                                                    │
│  proxy.ts (locale negotiation, /tr|en redirect)                   │
│  generateStaticParams (static prerender per locale)               │
│  Metadata + Schema.org (server-rendered JSON-LD)                  │
│                                                                    │
└──────┬──────────────────────────────┬──────────────────────────────┘
       │                              │
       ▼                              ▼
┌──────────────────────┐   ┌──────────────────────────────────────┐
│  Dictionaries        │   │  Mock Data Layer (TypeScript)        │
│  src/i18n/dict/      │   │  src/mock/                           │
│  ├─ tr.json          │   │  ├─ projects.ts (50+ projects)       │
│  └─ en.json          │   │  ├─ blog.ts                          │
│                      │   │  ├─ profiles.ts (HEB/HEA/IPE…)       │
│  getDictionary()     │   │  ├─ services.ts                      │
│  hasLocale()         │   │  ├─ certifications.ts                │
│  ('server-only')     │   │  └─ index.ts (typed query funcs)     │
└──────────────────────┘   └──────────────────────────────────────┘
```

The architecture is **content-driven, server-first**. Server Components render the bulk of the UI from typed mock data; Client Components are isolated leaves (`'use client'`) for interactivity (calculator, filters, theme toggle, language switch, 3D viewer, map, AI chat, portal forms). Heavy client deps (Three.js, Leaflet) are loaded with `next/dynamic({ ssr: false })`. i18n is segment-based via `app/[lang]/...` with dictionaries loaded per request (Next.js 16 official pattern).

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| `app/[lang]/(marketing)` route group | Public pages — anasayfa, kurumsal, hizmetler, projeler, teknik-bilgiler, blog, iletisim. Shares marketing layout (header, footer, locale switcher, theme toggle). | RSC pages reading from `src/mock/`, leaf Client islands |
| `app/[lang]/(portal)` route group | Mock authenticated UI — `/portal/login`, `/portal/dashboard`. Different layout (auth chrome, sidebar). | Mostly Client Components since fully interactive forms/dashboard |
| `app/[lang]/(legal)` route group | KVKK, çerez politikası, kullanım şartları. Minimal layout (no nav). | Pure RSC; MDX-friendly |
| `proxy.ts` (root) | Locale negotiation: read `Accept-Language`, redirect `/` → `/tr` or `/en`. **Replaces middleware.ts**, see Pitfalls. | Single file at project root |
| `src/i18n/` | Locale config (`locales = ['tr', 'en']`), `getDictionary()`, `hasLocale()` type guard, dictionary JSON. | Server-only; imported by layouts/pages |
| `src/mock/` | Typed mock data (projects, blog, profiles, services, certifications) + query functions (`getProjectBySlug`, `listProjects({filters})`). | Pure TS modules; `import 'server-only'` for safety; replace with CMS later |
| `src/components/ui/` | shadcn/ui primitives (Button, Card, Input, Tabs, Dialog, Sheet, Tooltip…). Headless + Tailwind v4. | Generated via shadcn CLI; client by default |
| `src/components/marketing/` | Domain components (Hero, ProjectCard, ProjectGallery, ServiceTile, BlogCard, ContactForm, OfficeMap). | Mix of RSC + leaf Client |
| `src/components/tools/` | Engineering tools (WeightCalculator, ProfileTable, ProfileCompare, ProjectMap, ModelViewer3D). | Client Components with explicit boundaries |
| `src/components/portal/` | Mock portal UI (LoginForm, DashboardCards, DocumentList, RoleSwitch). | Client Components |
| `src/components/providers/` | ThemeProvider (`next-themes`), AnimationProvider (Motion `LazyMotion`), TooltipProvider. | All Client; mounted once in root layout |
| `src/lib/` | Pure utilities — `cn()` (clsx+tailwind-merge), `slugify`, `formatNumber(locale)`, `weightCalc(profile, length)`. | Server-safe; no React, no DOM |
| `src/styles/` | Design tokens — `tokens.css` (CSS variables for color, spacing, type), Tailwind v4 `@theme inline` config in `globals.css`. | Single source of truth for tokens |

## Recommended Project Structure

```
erkicelik-deneme/
├── proxy.ts                          # Next 16: locale negotiation (NOT middleware.ts)
├── next.config.ts                    # reactCompiler: true, images, experimental
├── postcss.config.mjs                # @tailwindcss/postcss
├── tsconfig.json                     # strict mode, paths: @/* → src/*
├── public/
│   ├── images/projects/              # Scraped from old site (only assets reused)
│   ├── images/og/                    # OpenGraph defaults
│   ├── models/                       # Sample GLTF for 3D viewer (Phase 5)
│   └── fonts/                        # Self-hosted if needed (Geist via next/font default)
└── src/
    ├── app/
    │   ├── layout.tsx                # ROOT layout: <html>, providers, fonts (no <body lang>)
    │   ├── globals.css               # @import "tailwindcss" + @theme inline + @custom-variant dark
    │   ├── not-found.tsx             # Top-level 404 (redirects to /[lang]/not-found ideally)
    │   ├── global-error.tsx          # Top-level uncaught error with html+body
    │   ├── icon.tsx                  # Generated favicon
    │   ├── apple-icon.tsx
    │   ├── opengraph-image.tsx       # Default OG image generator
    │   ├── robots.ts                 # Generated robots.txt
    │   ├── sitemap.ts                # Generated sitemap (locale-aware)
    │   └── [lang]/
    │       ├── layout.tsx            # Locale-aware layout: lang attr, dict provider
    │       ├── not-found.tsx         # Localized 404
    │       ├── error.tsx             # Locale-aware error boundary
    │       ├── (marketing)/
    │       │   ├── layout.tsx        # Marketing chrome: Header, Footer, CookieBanner
    │       │   ├── loading.tsx       # Marketing loading skeleton
    │       │   ├── page.tsx          # /[lang] — anasayfa
    │       │   ├── kurumsal/
    │       │   │   └── page.tsx
    │       │   ├── hizmetler/
    │       │   │   ├── page.tsx
    │       │   │   └── [slug]/page.tsx   # /hizmetler/projelendirme etc.
    │       │   ├── projeler/
    │       │   │   ├── page.tsx          # Grid + filter + tabs
    │       │   │   ├── loading.tsx       # Grid skeleton
    │       │   │   └── [slug]/
    │       │   │       ├── page.tsx      # Project detail (tonaj, lokasyon, galeri)
    │       │   │       ├── loading.tsx
    │       │   │       └── not-found.tsx
    │       │   ├── teknik-bilgiler/
    │       │   │   ├── page.tsx          # Profile tables + calculator entry
    │       │   │   └── [profil]/page.tsx # Per-profile detail (HEB, HEA, IPE…)
    │       │   ├── blog/
    │       │   │   ├── page.tsx          # List
    │       │   │   ├── kategori/[slug]/page.tsx
    │       │   │   ├── etiket/[slug]/page.tsx
    │       │   │   └── [slug]/
    │       │   │       ├── page.tsx      # Post detail
    │       │   │       └── not-found.tsx
    │       │   └── iletisim/
    │       │       └── page.tsx          # Form + Maps embed + offices
    │       ├── (portal)/
    │       │   ├── layout.tsx        # Portal chrome (sidebar, user menu)
    │       │   └── portal/
    │       │       ├── login/page.tsx        # Mock login UI
    │       │       └── dashboard/
    │       │           ├── page.tsx          # Cards: orders, projects, docs
    │       │           ├── belgeler/page.tsx # Document list
    │       │           └── siparisler/page.tsx
    │       └── (legal)/
    │           ├── layout.tsx        # Minimal legal chrome (just header)
    │           ├── kvkk/page.tsx
    │           ├── cerez-politikasi/page.tsx
    │           └── kullanim-sartlari/page.tsx
    ├── components/
    │   ├── ui/                       # shadcn/ui primitives — DO NOT edit by hand carelessly
    │   │   ├── button.tsx
    │   │   ├── card.tsx
    │   │   ├── input.tsx
    │   │   ├── tabs.tsx
    │   │   ├── dialog.tsx
    │   │   ├── sheet.tsx
    │   │   ├── tooltip.tsx
    │   │   ├── select.tsx
    │   │   ├── slider.tsx
    │   │   └── …
    │   ├── marketing/                # Domain UI built on top of ui/
    │   │   ├── hero/
    │   │   │   ├── hero.tsx              # RSC composer
    │   │   │   ├── hero-cta.tsx          # 'use client' (CTA hover anim)
    │   │   │   └── hero-bg.tsx           # 'use client' (parallax)
    │   │   ├── header/
    │   │   │   ├── header.tsx            # RSC; renders client islands
    │   │   │   ├── nav-mobile.tsx        # 'use client' (Sheet)
    │   │   │   ├── locale-switcher.tsx   # 'use client'
    │   │   │   └── theme-toggle.tsx      # 'use client' (next-themes)
    │   │   ├── footer/
    │   │   │   └── footer.tsx
    │   │   ├── projects/
    │   │   │   ├── project-card.tsx          # RSC
    │   │   │   ├── project-gallery.tsx       # RSC shell
    │   │   │   ├── project-filter-bar.tsx    # 'use client' (filter state)
    │   │   │   ├── project-tabs.tsx          # 'use client' (Tabs)
    │   │   │   └── project-detail-gallery.tsx # 'use client' (lightbox)
    │   │   ├── services/
    │   │   ├── blog/
    │   │   ├── contact/
    │   │   │   ├── contact-form.tsx          # 'use client' (RHF + zod + honeypot)
    │   │   │   ├── office-card.tsx           # RSC
    │   │   │   └── contact-map.tsx           # 'use client' lazy (iframe)
    │   │   ├── certifications/
    │   │   ├── cookie-banner.tsx             # 'use client' (KVKK)
    │   │   └── newsletter-form.tsx           # 'use client' (Phase 3)
    │   ├── tools/                    # Phase 4–6 (heavier client work)
    │   │   ├── weight-calculator/
    │   │   │   ├── weight-calculator.tsx     # 'use client' main shell
    │   │   │   ├── profile-picker.tsx        # 'use client'
    │   │   │   └── result-display.tsx        # 'use client'
    │   │   ├── profile-table.tsx             # 'use client' (search/sort/filter)
    │   │   ├── profile-compare.tsx           # 'use client'
    │   │   ├── project-map/                  # Phase 5 — lazy
    │   │   │   ├── project-map.tsx           # 'use client'
    │   │   │   └── map-pin.tsx
    │   │   ├── model-viewer/                 # Phase 5 — lazy, no-SSR
    │   │   │   ├── model-viewer.tsx          # 'use client'
    │   │   │   └── model-loader.tsx          # next/dynamic({ ssr: false })
    │   │   └── ai-chat/                      # Phase 6 — mock streaming
    │   │       ├── ai-chat.tsx               # 'use client'
    │   │       ├── chat-message.tsx
    │   │       ├── chat-input.tsx
    │   │       └── mock-stream.ts            # Mock SSE-ish streaming via setTimeout
    │   ├── portal/                   # Phase 8
    │   │   ├── login-form.tsx                # 'use client'
    │   │   ├── dashboard-cards.tsx           # RSC + client islands
    │   │   ├── document-list.tsx
    │   │   └── role-switch.tsx               # 'use client' (admin/musteri toggle)
    │   ├── providers/
    │   │   ├── theme-provider.tsx            # next-themes wrapper, 'use client'
    │   │   ├── motion-provider.tsx           # Motion LazyMotion, 'use client'
    │   │   └── tooltip-provider.tsx
    │   └── seo/
    │       ├── json-ld.tsx                   # RSC; server-rendered Schema.org
    │       └── breadcrumbs.tsx
    ├── i18n/
    │   ├── config.ts                 # locales, defaultLocale, routing helpers
    │   ├── dictionaries.ts           # 'server-only' getDictionary, hasLocale
    │   └── dict/
    │       ├── tr.json               # PRIMARY (full content)
    │       └── en.json               # Scaffold (TR copy or [EN-TODO] tags)
    ├── mock/
    │   ├── types.ts                  # Project, BlogPost, Service, Profile, Certification, …
    │   ├── projects.ts
    │   ├── blog.ts
    │   ├── services.ts
    │   ├── profiles.ts               # HEB/HEA/IPE/IPN/köşebent/kutu/boru rows
    │   ├── certifications.ts
    │   ├── offices.ts                # Tuzla, Gebze
    │   ├── team.ts                   # Phase 3
    │   └── index.ts                  # Re-exports + query functions (getProjectBySlug, listProjects)
    ├── lib/
    │   ├── cn.ts                     # tailwind-merge + clsx
    │   ├── format.ts                 # number/date with Intl per locale
    │   ├── slugify.ts
    │   ├── seo.ts                    # buildMetadata(locale, page) helper
    │   └── calc/
    │       └── weight.ts             # Pure: profile + length → kg (testable)
    ├── hooks/
    │   ├── use-locale.ts             # Read params or context
    │   ├── use-media-query.ts
    │   └── use-reduced-motion.ts
    ├── styles/
    │   └── tokens.css                # CSS variables (palette, type, radii)
    └── tests/
        ├── e2e/                      # Playwright specs per phase
        └── smoke/
```

### Structure Rationale

- **Route groups `(marketing) (portal) (legal)`:** Each has different chrome (header/footer for marketing, sidebar for portal, minimal for legal). Route groups give us **separate layouts without affecting URLs** — exactly what Next.js 16 docs recommend (`02-project-structure.md`). Keeping legal in its own group prevents the heavy marketing header from rendering on KVKK pages.
- **`app/[lang]/...` over domain-based:** Required by the official Next.js 16 i18n pattern (verified `02-guides/internationalization.md`). Domain-based (`erkincelik.tr` vs `erkincelik.en`) needs DNS + hosting decisions that PROJECT.md says are deferred. Sub-path routing also keeps a single deploy target.
- **`src/components/ui/` (shadcn) vs `src/components/marketing/` (domain):** Standard shadcn convention. `ui/` is generated/replaceable primitives; `marketing/`, `tools/`, `portal/` are *composed* from `ui/`. This separation lets us regenerate or theme `ui/` without touching feature code. We did **not** pick `src/design-system/` because shadcn convention is `components/ui` and downstream code completion (and the official CLI `--components-path`) defaults there.
- **`src/mock/` outside `app/`:** Mock layer is *infrastructure*, not a route. Co-locating in `app/_mock/` is allowed but mixing data with routes makes the eventual swap to a CMS or BFF (next milestone) much more disruptive. Keeping it at `src/mock/` makes the boundary explicit: pages call `import { listProjects } from '@/mock'` today, and that import target swaps to `@/lib/cms` later with **zero changes to call sites**.
- **`src/i18n/` over `app/[lang]/dictionaries.ts`:** Next.js docs show dictionaries inside `app/[lang]/`, but with multiple route groups all needing the same dict, hoisting to `src/i18n/` avoids relative-path tangles and lets `'server-only'` enforcement live in one place.
- **`proxy.ts` at project root:** Next.js 16 renamed `middleware.ts` → `proxy.ts`. Both work in 16.x but `proxy.ts` is the new convention; using it now avoids deprecation warnings later (verified `02-project-structure.md` top-level files table).
- **`src/lib/calc/`:** Pure functions are kept *out* of `components/`. The weight calculator's math is testable without React; the component is a thin shell. This pattern repeats for any feature with non-trivial logic.

## Architectural Patterns

### Pattern 1: Server-by-default, Client-at-the-leaves

**What:** Every route segment, layout, and page stays a Server Component. `'use client'` is added only to the smallest possible leaf component that needs state, effects, or browser APIs.

**When to use:** Always — this is the App Router's intended model and Next.js 16 docs explicitly recommend it (`05-server-and-client-components.md`).

**Trade-offs:**
- ✅ Smaller JS bundles, faster FCP, server-rendered SEO content
- ✅ Mock data stays on the server (no shipping `mock/projects.ts` to the browser)
- ⚠️ Requires discipline — easy to accidentally make a parent client by importing a hook
- ⚠️ Context providers must wrap `{children}` (not the whole `<html>`) to keep the tree mostly server

**Example:**
```tsx
// app/[lang]/(marketing)/projeler/page.tsx — Server Component
import { listProjects } from '@/mock'
import { ProjectFilterBar } from '@/components/marketing/projects/project-filter-bar'
import { ProjectGallery } from '@/components/marketing/projects/project-gallery'

export default async function ProjelerPage({ params }: PageProps<'/[lang]/projeler'>) {
  const { lang } = await params
  const projects = await listProjects()           // Runs on server
  return (
    <>
      <ProjectFilterBar locale={lang} />          {/* 'use client' — controls URL search params */}
      <ProjectGallery projects={projects} />      {/* RSC — server-rendered grid */}
    </>
  )
}
```

The filter bar manipulates URL search params; the gallery re-renders on the server when `searchParams` change. No client-side JS needed for the gallery itself.

### Pattern 2: URL search params as filter state (no Zustand for filters)

**What:** Project filter state (category, year, location, tonnage) lives in `searchParams`, not in a client store. The Client `ProjectFilterBar` reads/writes URL via `useRouter().replace()`; the Server `ProjectGallery` reads `searchParams` from page props and filters server-side.

**When to use:** Any list/grid that needs filtering, sorting, or pagination on a server-rendered page. Avoids hydration mismatch and keeps URLs shareable.

**Trade-offs:**
- ✅ Shareable URLs, browser back/forward works, no client store needed
- ✅ Filtering happens on the server with full mock data (later: full DB)
- ⚠️ Each filter change triggers a server round-trip (mitigated by RSC streaming + `loading.tsx`)

**Example:**
```tsx
// page.tsx (RSC)
export default async function Page({ searchParams }: PageProps<'/[lang]/projeler'>) {
  const sp = await searchParams
  const projects = await listProjects({
    category: sp.kategori,
    year: sp.yil ? Number(sp.yil) : undefined,
  })
  return <ProjectGallery projects={projects} />
}
```

### Pattern 3: Provider stack mounted once, deep in tree

**What:** Theme, motion, and tooltip providers are wrapped around `{children}` in the locale layout — not the root `<html>`. The root layout stays as much RSC as possible.

**When to use:** Any time you need `next-themes`, animation context, tooltip portals, or other client-only context.

**Trade-offs:**
- ✅ Maximizes server-rendering of static content
- ✅ Theme transitions still work because next-themes hydrates from `localStorage`
- ⚠️ Setting `<html className=...>` for theme requires `suppressHydrationWarning` on `<html>` (next-themes requirement)

**Example:**
```tsx
// app/layout.tsx (RSC — root)
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html suppressHydrationWarning className={`${geistSans.variable} ${geistMono.variable}`}>
      <body>{children}</body>
    </html>
  )
}

// app/[lang]/layout.tsx (RSC — locale)
import { Providers } from '@/components/providers'
export default async function LangLayout({ children, params }: LayoutProps<'/[lang]'>) {
  const { lang } = await params
  if (!hasLocale(lang)) notFound()
  return <Providers locale={lang}>{children}</Providers>
}

// components/providers/index.tsx ('use client')
'use client'
import { ThemeProvider } from 'next-themes'
import { LazyMotion, domAnimation } from 'motion/react'
export function Providers({ children, locale }: { children: ReactNode; locale: 'tr' | 'en' }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="dark" enableSystem>
      <LazyMotion features={domAnimation} strict>{children}</LazyMotion>
    </ThemeProvider>
  )
}
```

### Pattern 4: `next/dynamic({ ssr: false })` for browser-only widgets

**What:** Three.js/react-three-fiber, Leaflet/MapLibre, and any library touching `window` are imported via `next/dynamic` with `ssr: false`, **inside a Client Component wrapper** (Next 16 disallows `ssr: false` in Server Components — verified `02-guides/lazy-loading.md`).

**When to use:** 3D viewer (Phase 5), interactive map (Phase 5), any widget that breaks during SSR.

**Trade-offs:**
- ✅ Heavy libs stay out of the initial bundle and out of the server render path
- ⚠️ User sees a placeholder during chunk load → must design a meaningful skeleton
- ⚠️ The wrapper itself must be a Client Component

**Example:**
```tsx
// components/tools/model-viewer/model-loader.tsx
'use client'
import dynamic from 'next/dynamic'
export const ModelViewer = dynamic(
  () => import('./model-viewer').then(m => m.ModelViewer),
  { ssr: false, loading: () => <ModelSkeleton /> }
)
```

### Pattern 5: Mock data layer = "future BFF interface"

**What:** All mock data is exposed via *async* query functions that mimic the eventual BFF/CMS shape: `await getProjectBySlug(slug, lang)`, `await listProjects({filters})`. Even though they read from in-memory arrays, they're `async` so swapping to CMS/DB later requires no signature changes.

**When to use:** Always — this is the seam that defines the next milestone's backend integration.

**Trade-offs:**
- ✅ Pages don't change when CMS arrives
- ✅ `import 'server-only'` at top of `src/mock/index.ts` prevents accidental client bundling
- ⚠️ Slight overhead of async/await with no real async work (negligible)

**Example:**
```ts
// src/mock/index.ts
import 'server-only'
import { projects } from './projects'

export async function listProjects(filters?: ProjectFilters): Promise<Project[]> {
  return projects.filter(p => /* … */)
}

export async function getProjectBySlug(slug: string, lang: Locale): Promise<Project | null> {
  return projects.find(p => p.slug[lang] === slug) ?? null
}
```

### Pattern 6: Mock streaming for AI chat

**What:** The Phase 6 AI chat UI consumes a mock async iterator that emits tokens with `setTimeout`. Same component contract as a real LLM stream — only the data source differs.

**When to use:** Phase 6 only.

**Trade-offs:**
- ✅ UI is built against the final shape; backend swap is "replace the iterator"
- ⚠️ Must resist the urge to make the chat UI tightly couple with `setTimeout` semantics

**Example:**
```ts
// components/tools/ai-chat/mock-stream.ts
export async function* mockStream(prompt: string): AsyncGenerator<string> {
  const reply = mockAnswerFor(prompt)            // Pre-canned per question pattern
  for (const chunk of reply.match(/.{1,8}/g) ?? []) {
    await new Promise(r => setTimeout(r, 35))
    yield chunk
  }
}
```

### Pattern 7: Theming with Tailwind v4 + `@custom-variant dark`

**What:** Tailwind v4 dropped the `darkMode: 'class'` config. Use the new `@custom-variant dark (&:is(.dark *))` directive in `globals.css` so `next-themes` (which sets `class="dark"` on `<html>`) cascades to all `dark:` utilities.

**When to use:** Project-wide. Set up in Phase 1+2 and forget.

**Example:**
```css
/* src/app/globals.css */
@import "tailwindcss";
@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-bg: var(--bg);
  --color-fg: var(--fg);
  --color-accent: var(--accent);   /* metalik turuncu/bakır */
  --font-sans: var(--font-geist-sans);
}
:root      { --bg: #ffffff; --fg: #0a0a0a; --accent: #b85c00; }
.dark      { --bg: #0a0a0a; --fg: #ededed; --accent: #ff7a1a; }
```

## Data Flow

### Request Flow (marketing page)

```
User → /tr/projeler/cargo-energy
        ↓
proxy.ts (no-op since locale is in URL)
        ↓
app/[lang]/(marketing)/projeler/[slug]/page.tsx (RSC)
        ↓
   await params → { lang: 'tr', slug: 'cargo-energy' }
        ↓
   getDictionary(lang)           getProjectBySlug(slug, lang)
        ↓                                ↓
   src/i18n/dict/tr.json         src/mock/projects.ts
        ↓                                ↓
        └──────────┬─────────────────────┘
                   ↓
   Render server tree → RSC payload
                   ↓
   Client islands: <ProjectDetailGallery>, <ContactCTA>
                   ↓
   HTML + RSC payload → Browser
                   ↓
   React hydrates client islands
```

### Filter Flow (project list)

```
User clicks "Havalimanı" filter
        ↓
ProjectFilterBar ('use client') →
  router.replace('/tr/projeler?kategori=havalimani', { scroll: false })
        ↓
Next.js re-renders /tr/projeler on server
        ↓
page.tsx reads searchParams.kategori → listProjects({category:'havalimani'})
        ↓
ProjectGallery renders new RSC tree
        ↓
Stream new RSC payload to browser → React updates DOM
```

### Theme Flow (light/dark)

```
User clicks ThemeToggle ('use client')
        ↓
useTheme().setTheme('dark')   (next-themes)
        ↓
next-themes writes <html class="dark"> + localStorage['theme']='dark'
        ↓
Tailwind @custom-variant dark applies dark: utilities
        ↓
CSS variables (--bg, --fg, --accent) flip; transitions handled by CSS
```

### Locale Flow (TR ↔ EN)

```
User clicks LocaleSwitcher
        ↓
Component reads current pathname → /tr/projeler/cargo-energy
        ↓
Replace first segment → /en/projeler/cargo-energy
        ↓
router.push(newPath)
        ↓
Server re-renders with lang='en' → fetches en dictionary
        ↓
[EN-TODO] tags visible in scaffold; full content later
```

### State Management Map

| Concern | Where it lives | Why |
|---------|---------------|-----|
| Filter selections | URL `searchParams` | Shareable, server-readable |
| Theme | `next-themes` (localStorage + `<html class>`) | Avoids FOUC, persists |
| Locale | URL segment `[lang]` | Server-readable, SEO-friendly |
| Calculator inputs | Local `useState` in `WeightCalculator` | Ephemeral, single-component |
| AI chat history | Local `useState` in `AiChat` (no persistence) | Mock UI; persistence Phase 6+ |
| Cookie consent | `localStorage` + `useState` | Compliance; one-time |
| Portal mock auth | Local `useState` "fake-logged-in" toggle | No real auth this milestone |
| 3D model state | Local within `ModelViewer` (R3F internal) | Self-contained |

**No global client store (no Redux, no Zustand) for this milestone.** If the next milestone needs cross-page client state (e.g., real auth session, cart-like portal state), introduce Zustand at that point — not now.

## Component Boundary Patterns (per feature)

### Project Gallery + Filter

```
ProjelerPage (RSC)
├── ProjectFilterBar ('use client')      ← reads/writes searchParams
├── ProjectTabs ('use client')           ← Tamamlanan/Devam Eden tab state in URL
└── ProjectGallery (RSC)
    └── ProjectCard (RSC)
        └── (optional) HoverPreview ('use client')   ← motion-on-hover only
```

**Rule:** The grid stays RSC. Only the *controls* (filters, tabs, hover effects) are client.

### Weight Calculator

```
TeknikBilgilerPage (RSC) — renders profile tables (RSC) + WeightCalculator entry
└── WeightCalculator ('use client')
    ├── ProfilePicker (Combobox over profile JSON)
    ├── LengthInput
    └── ResultDisplay
        ↑
        └── lib/calc/weight.ts (pure, server-safe, unit-tested)
```

**Rule:** Pure math in `lib/calc/`. Component is a thin shell. Tests live next to the pure module.

### 3D Viewer (lazy)

```
ProjeDetayPage (RSC)
└── ModelViewerSection (RSC wrapper, decides whether to show)
    └── ModelLoader ('use client')        ← next/dynamic({ ssr:false })
        └── ModelViewer ('use client')    ← actual R3F canvas
```

**Rule:** Two boundaries — one to lazy-load chunks, one for R3F. Skeleton is mandatory.

### Project Map

```
AnasayfaPage / ProjelerPage (RSC)
└── ProjectMapSection (RSC)
    └── ProjectMapClient ('use client', dynamic ssr:false)
        └── Leaflet/MapLibre map + Pin components
            ↑
            └── Pins read from src/mock/projects.ts via SSR-fed prop
```

**Rule:** Map data flows top-down as a serialized prop. The client never re-fetches projects.

### AI Chat (mock)

```
AiChatSection (RSC) — could appear in /teknik-bilgiler or as floating widget
└── AiChat ('use client')
    ├── ChatMessages
    │   └── ChatMessage
    └── ChatInput
        ↓ on submit
        mockStream(prompt) — async generator → updates message list incrementally
```

**Rule:** All in client. Stream contract is the same shape a real LLM gateway will fulfil later.

### Portal (mock)

```
(portal) layout (RSC) — sidebar + topbar (mostly RSC)
└── /portal/login/page.tsx (RSC shell)
    └── LoginForm ('use client') → on submit, fakeAuth() → router.push('/portal/dashboard')
└── /portal/dashboard/page.tsx (RSC)
    ├── DashboardCards (RSC; reads mock orders)
    ├── RoleSwitch ('use client')          ← admin/musteri toggle, persisted to localStorage
    └── DocumentList (RSC) + DownloadButton ('use client')
```

**Rule:** Even the mock portal stays mostly RSC. Only forms and toggles are client. This trains the codebase for the real portal later.

## Shared Layouts, Error Boundaries, Loading

| File | Where | Purpose |
|------|-------|---------|
| `app/layout.tsx` | root | `<html>`, `<body>`, fonts, `suppressHydrationWarning`. **No locale yet.** |
| `app/[lang]/layout.tsx` | locale | Set `<html lang>` via params, mount Providers, validate locale (`hasLocale` → notFound) |
| `app/[lang]/(marketing)/layout.tsx` | marketing group | Header, Footer, CookieBanner, MainContainer |
| `app/[lang]/(portal)/layout.tsx` | portal group | Sidebar, Topbar, no public footer |
| `app/[lang]/(legal)/layout.tsx` | legal group | Minimal header, no footer nav clutter |
| `app/global-error.tsx` | root | Last-resort uncaught (must include `<html>` + `<body>`) |
| `app/not-found.tsx` | root | When locale itself is wrong (e.g., `/de/...` 404) |
| `app/[lang]/error.tsx` | locale | Locale-aware uncaught fallback (uses `unstable_retry` per Next 16 docs) |
| `app/[lang]/not-found.tsx` | locale | Localized 404 page |
| `app/[lang]/(marketing)/projeler/loading.tsx` | route | Grid skeleton while server fetches |
| `app/[lang]/(marketing)/projeler/[slug]/not-found.tsx` | route | Project-not-found state |
| `app/[lang]/(marketing)/blog/[slug]/not-found.tsx` | route | Blog-not-found state |

**Note:** `error.tsx` files **must** be Client Components (`'use client'`) — Next 16 confirms this (`10-error-handling.md`). Use the new `unstable_retry` callback (Next 16 API) instead of the old `reset` prop.

## Animation Orchestration

- **Library choice:** Motion (Framer Motion's successor) with `LazyMotion` + `domAnimation` (preferred over importing the full `motion` package — saves ~25 KB). Use `m.div` not `motion.div`.
- **Provider:** `MotionProvider` wraps everything inside the locale layout. `strict` mode catches accidental full-feature imports.
- **Scroll-driven:** Native CSS `animation-timeline: scroll()` where supported; Motion's `useScroll` as polyfill. Capability check in `hooks/use-reduced-motion.ts` to disable when user prefers reduced motion.
- **Page transitions:** Use `template.tsx` (not `layout.tsx`) for re-mounting on route change — Next 16 supports `template` per `02-project-structure.md`. Keep transitions short (≤ 250 ms) — B2B users dislike slow chrome.
- **Component-level:** Each `marketing/` component owns its enter animation via a small `m.div` wrapper. Animations are colocated, not centralized.
- **Performance discipline:** Never animate `top/left/width/height`. Only `transform` and `opacity`. Tailwind's `transform-gpu` utility on heavy elements.

## Theming (next-themes + Tailwind v4)

- `ThemeProvider` from `next-themes` mounted in `components/providers` (Client). `attribute="class"`, `defaultTheme="dark"` (industrial-premium look starts dark), `enableSystem`.
- Root `<html>` carries `suppressHydrationWarning` (next-themes requirement) — without it, hydration warnings appear because the server renders without a class but the client adds one from localStorage.
- Tokens defined as CSS variables in `globals.css` under `:root` (light) and `.dark` selectors. Tailwind v4's `@theme inline` references them so `bg-bg`, `text-fg`, `text-accent` work and respect the theme.
- `@custom-variant dark (&:is(.dark *))` is required in Tailwind v4 to make the `dark:` modifier work with class-based theming.
- `ThemeToggle` is a small Client Component using `useTheme()` — placed in the marketing header.

## Build Order (Dependencies Between Components)

```
WAVE 0 — Foundation (Phase 1+2 start)
  proxy.ts → src/i18n/{config,dictionaries} → src/i18n/dict/{tr,en}.json
  src/mock/types.ts → src/mock/{projects,blog,services,profiles,certifications,offices}
  src/lib/{cn,format,slugify,seo} → src/lib/calc/weight (Phase 4 prep)
  src/styles/tokens.css + globals.css (@theme + @custom-variant dark)
  app/layout.tsx (root) + app/[lang]/layout.tsx (locale)
  components/providers/* (theme, motion, tooltip)
  shadcn init → components/ui/{button,card,input,…} (10–12 primitives)

WAVE 1 — Marketing chrome (Phase 1+2)
  components/marketing/header (header, nav-mobile, locale-switcher, theme-toggle)
  components/marketing/footer
  app/[lang]/(marketing)/layout.tsx + loading.tsx + not-found.tsx + error.tsx
  components/marketing/cookie-banner

WAVE 2 — Marketing content pages (Phase 1+2)
  Anasayfa: hero, capacity, services preview, featured projects, contact CTA
  Kurumsal page (static + certifications)
  Hizmetler list + [slug] detail
  Teknik Bilgiler tables (RSC tables; calculator placeholder)
  Blog list + [slug] (no calculator yet)
  İletişim form + offices + maps embed
  components/seo/json-ld + sitemap.ts + robots.ts

WAVE 3 — Projects feature (Phase 1+2 finale)
  ProjectCard (RSC) → ProjectGallery (RSC) → ProjectFilterBar ('use client') → ProjectTabs
  /projeler/page.tsx wires all three
  /projeler/[slug]/page.tsx + project-detail-gallery (lightbox client)
  Test: Playwright smoke for grid filter + detail navigation

WAVE 4 — Phase 3 content & marketing extensions
  Reference logos strip, case studies (reuse project detail), team page,
  career page, press archive, newsletter form, blog category/tag/related,
  PDF preview component, catalog download

WAVE 5 — Phase 4 engineering tools
  WeightCalculator (depends on lib/calc/weight; depends on profile picker depends on src/mock/profiles)
  ProfileTable (search/sort/filter; depends on shadcn Table + Combobox + Slider)
  ProfileCompare (depends on ProfileTable selection state via URL params)

WAVE 6 — Phase 5 map + 3D
  ProjectMap: pick MapLibre or Leaflet → wrap in next/dynamic ssr:false
  ModelViewer: react-three-fiber + drei → wrap in next/dynamic ssr:false
  Add map/3D entry points on /projeler and /projeler/[slug]

WAVE 7 — Phase 6 AI chat mock
  AiChat shell + ChatInput + ChatMessage + mockStream generator
  Mount on /teknik-bilgiler (or as floating widget gated by feature flag)

WAVE 8 — Phase 7 media
  Video gallery component (HTML5 video + poster lazy load)
  Production process interactive diagram (SVG + Motion scroll-driven)
  Mock "live stream" placeholder

WAVE 9 — Phase 8 portal mock
  (portal) layout + login form + dashboard cards + document list + role switch
  Fake-auth toggle persisted to localStorage; never blocks public pages
```

**Critical dependency edges:**
- Tokens + Tailwind config **must** land before any component (Wave 0).
- `src/mock/types.ts` defines the contract for everything downstream — write types before data.
- Marketing layout (Wave 1) blocks all marketing pages (Wave 2+).
- ProjectCard is reused everywhere (anasayfa featured, /projeler grid, related projects on blog) — finalize its prop API early.
- `next-themes` must be wired before any `dark:` classes are added — otherwise the team will style only one theme.
- `lib/calc/weight.ts` (pure) gates Phase 4 calculator — write + unit-test in Wave 0 if possible.

## Phase Mapping (PROJECT.md)

| PROJECT.md Phase | Architecture Waves | Notes |
|------------------|--------------------|-------|
| **Phase 1+2** Çekirdek + Tasarım Sistemi | Waves 0, 1, 2, 3 | Foundation + chrome + content + projects. Heaviest phase by far. Design system locked here. |
| **Phase 3** İçerik & Pazarlama | Wave 4 | Pure UI additions; all data is mock. No new architectural patterns. |
| **Phase 4** Mühendislik Araçları | Wave 5 | First feature where pure logic (`lib/calc`) matters. Adds Combobox, Slider, Table to shadcn primitives if not yet. |
| **Phase 5** Harita & 3D | Wave 6 | Introduces the **`next/dynamic ssr:false` pattern** for the first time. Both libs must follow the same boundary template. |
| **Phase 6** AI Asistan UI (mock) | Wave 7 | Async generator pattern; ensures backend swap later is trivial. |
| **Phase 7** Canlı Üretim & Medya (mock) | Wave 8 | Video lazy-load and SVG scroll animations only. |
| **Phase 8** Müşteri Portalı UI (mock) | Wave 9 | Adds new route group `(portal)`. Never wire real auth. |

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 0–10k visitors/mo (initial launch) | Static prerender every route via `generateStaticParams({lang})` + ISR if needed. No backend; mock data is fine. |
| 10k–100k visitors/mo | Move mock to a real CMS (Sanity/Payload). Same `getProjectBySlug` interface — pages don't change. Cache CMS queries with `unstable_cache` or `'use cache'` (Next 16 cache components). |
| 100k+ visitors/mo | Add `'use cache'` directives to expensive RSC trees. Edge runtime for proxy.ts. CDN images via `next/image` is already in place. Consider PPR for the dashboard once portal is real. |

### Scaling Priorities (what breaks first → fix)

1. **Image weight** — biggest risk for Lighthouse. `next/image` everywhere, AVIF/WebP, explicit sizes, blurDataURL for project hero shots.
2. **Client bundle** — measure per route with `next build`. Any route > 150 KB JS gets a `next/dynamic` audit.
3. **3D viewer** — Phase 5 is the only place where bundle can blow up. Already gated by `next/dynamic ssr:false`. Consider model decimation + DRACO compression.
4. **Search/filter on profile tables** — at ~200 rows, client-side fuse.js is fine. If the table grows to thousands, move filter to server (URL params already enable that).

## Anti-Patterns

### Anti-Pattern 1: `'use client'` at the top of `app/[lang]/(marketing)/layout.tsx`

**What people do:** Add `'use client'` to the marketing layout because the header has interactivity.
**Why it's wrong:** The entire marketing tree becomes client-rendered. Bundle balloons, SEO-critical content like JSON-LD becomes harder to inject server-side.
**Do this instead:** Keep the layout as RSC. Push `'use client'` down to `<ThemeToggle />`, `<LocaleSwitcher />`, `<MobileNavSheet />` only.

### Anti-Pattern 2: Importing `src/mock/projects.ts` from a Client Component

**What people do:** Use the mock array client-side because "it's just JSON, it's small."
**Why it's wrong:** Ships the entire dataset to the browser, defeats SSR for the gallery, and breaks the future CMS swap.
**Do this instead:** Pages (RSC) read from mock and pass serialized props to client islands. Add `import 'server-only'` to `src/mock/index.ts` to enforce.

### Anti-Pattern 3: Domain-based i18n routing (`erkincelik.tr` vs `erkincelik.en`)

**What people do:** Suggest separate domains "for SEO."
**Why it's wrong:** Requires DNS + hosting decisions PROJECT.md says are deferred. Doubles deploy complexity. No measurable SEO benefit for a B2B site.
**Do this instead:** Sub-path routing (`/tr`, `/en`) with `hreflang` tags in metadata. Officially recommended by Next 16 docs.

### Anti-Pattern 4: One giant `globals.css` with all theme classes

**What people do:** Define every utility, every dark variant, all custom CSS in `globals.css`.
**Why it's wrong:** Hard to maintain, blocks tree-shaking, slow HMR.
**Do this instead:** Tokens in `globals.css`, component-specific styles via Tailwind utilities or CSS-in-JS only when truly needed (rare in this stack).

### Anti-Pattern 5: Animation provider with `motion` (full features)

**What people do:** `import { motion } from 'motion/react'` everywhere.
**Why it's wrong:** Pulls all animation features into every client bundle.
**Do this instead:** `LazyMotion strict features={domAnimation}` + `m.div` (lazy-load extras only when needed).

### Anti-Pattern 6: Mock auth that gates real routes

**What people do:** Put the fake-login behind a real check that redirects unauthed users.
**Why it's wrong:** When the milestone says "frontend only," actual route gating creates dead UX paths and confuses Phase 8 reviewers.
**Do this instead:** Show a banner "Mock UI — no real auth" and let everyone access `/portal/*`. Save real gating for the auth milestone.

### Anti-Pattern 7: Putting `error.tsx` at the root only

**What people do:** Single top-level error boundary.
**Why it's wrong:** A crash in `/projeler/[slug]` blanks the whole app shell.
**Do this instead:** `error.tsx` at the locale layer **and** at risky route segments (`projeler/[slug]`, `blog/[slug]`, `portal/dashboard`). Errors bubble up to nearest boundary per Next 16 docs.

### Anti-Pattern 8: Forgetting `suppressHydrationWarning` with `next-themes`

**What people do:** Add `next-themes` and ignore the dev warning.
**Why it's wrong:** Real hydration mismatches get drowned out by the noise; users see a flash of unthemed content.
**Do this instead:** `<html suppressHydrationWarning>` plus correct `attribute="class"` config. This is documented in next-themes README and is non-negotiable.

## Integration Points

### External Services (this milestone)

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Google Maps embed | iframe in `ContactMap` (client, lazy) | No JS API key needed for embed; cheap and accessible |
| Google Fonts (Geist) | `next/font/google` (already in scaffold) | Self-hosted; no FOUC |
| reCAPTCHA v3 | Client script in `ContactForm` | Requires public key; gate with env var |
| Mock GLTF asset | Static `public/models/*.glb` | One sample model for Phase 5 |

### External Services (deferred to next milestone — out of scope, do not architect for now)

- Real LLM gateway (AI Gateway / OpenRouter)
- Clerk / Auth0
- CMS (Sanity / Payload)
- Email backend (Resend / Postmark)

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| RSC pages ↔ `src/mock/` | Direct async function calls (server-only) | No HTTP layer; pure imports |
| RSC pages ↔ `src/i18n/` | `getDictionary()` async function | `'server-only'` enforced |
| RSC ↔ Client Components | Serializable props only | No functions, no class instances; use Date as ISO string then re-parse |
| Client `LocaleSwitcher` ↔ Router | `useRouter().push()` with rewritten path | Path manipulation lives in `src/lib/i18n-paths.ts` |
| Client `ThemeToggle` ↔ DOM | `next-themes` writes `<html class>` + localStorage | No app code touches `document` directly |
| Client `ProjectFilterBar` ↔ RSC `ProjectGallery` | URL `searchParams` | Server reads params, re-renders gallery |
| Client `WeightCalculator` ↔ `lib/calc/weight` | Direct import (pure module) | Module is server-safe but client-callable; no `server-only` here |

## Sources

- Next.js 16 — `node_modules/next/dist/docs/01-app/01-getting-started/02-project-structure.md` — top-level files (`proxy.ts`), routing files, route groups, private folders, src folder, multiple root layouts
- Next.js 16 — `node_modules/next/dist/docs/01-app/01-getting-started/05-server-and-client-components.md` — `'use client'` boundary, server-to-client prop serialization, context providers pattern, `server-only` package
- Next.js 16 — `node_modules/next/dist/docs/01-app/01-getting-started/10-error-handling.md` — `error.tsx`, `not-found.tsx`, `global-error.tsx`, `unstable_retry` (new in 16), `unstable_catchError`
- Next.js 16 — `node_modules/next/dist/docs/01-app/02-guides/internationalization.md` — `app/[lang]` segment-based routing, `getDictionary` + `hasLocale`, `proxy.ts` for locale negotiation, `generateStaticParams` for static prerender
- Next.js 16 — `node_modules/next/dist/docs/01-app/02-guides/lazy-loading.md` — `next/dynamic` with `ssr:false` only allowed inside Client Components, named-export dynamic import
- PROJECT.md — milestone scope (frontend-only, mock data), 8-phase plan, premium-industrial design direction, TR primary + EN scaffold
- Tailwind v4 release notes — `@theme inline`, `@custom-variant dark` directive (replaces `darkMode: 'class'` config)
- next-themes README — `attribute="class"`, `suppressHydrationWarning` requirement
- Motion (Framer Motion successor) — `LazyMotion` + `domAnimation` + `m.*` components for bundle reduction
- shadcn/ui conventions — `components/ui/` location, primitives composition pattern

---
*Architecture research for: Next.js 16 marketing site (erkincelik.com modern rebuild) — frontend-only milestone with mock data layer, design system, and 8 phased waves*
*Researched: 2026-04-25*
