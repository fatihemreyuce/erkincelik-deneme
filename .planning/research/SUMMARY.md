# Project Research Summary — Erkin Çelik Modern Web Yenileme

**Project:** Erkin Çelik — Modern Web Yenileme (erkincelik.com rebuild)
**Domain:** Premium B2B marketing site for a steel-construction fabricator (frontend-only milestone, mock data layer)
**Researched:** 2026-04-25
**Confidence:** HIGH on framework specifics (Next 16 / React 19 / Tailwind v4 verified directly from `node_modules/next/dist/docs/`); HIGH on feature landscape (peer sites inspected + 2026 B2B literature corroborated); MEDIUM on motion / 3D / map ecosystem (web sources, not first-party docs).

This document is the single source of truth for the roadmapper. It supersedes the four research files for planning purposes; consult the source files (`STACK.md`, `FEATURES.md`, `ARCHITECTURE.md`, `PITFALLS.md`) only when a decision needs deeper substantiation.

---

## Executive Summary

Erkin Çelik's existing WordPress site is a generic blue-grey TR corporate template with no project detail pages, no filtering, no i18n, no schema, no dark mode, and a carousel hero. The new site must replace it with a **server-first, premium-industrial Next.js 16 site** that feels credible to airport / bridge / hangar buyers — the trust bar is high because B2B steel deals are won on perceived professionalism. Research strongly converges on a single recommended path: keep the installed stack (Next 16.2.4 + React 19.2.4 + Tailwind v4 + React Compiler) and layer on shadcn/ui (Tailwind v4 distribution), `next-intl` for TR/EN routing, `next-themes` for dark/light, `motion` (the renamed framer-motion) for animation, MapLibre + react-map-gl for the project map, and react-three-fiber for the single 3D viewer. All dynamic data is served by a typed mock layer (`src/mock/`) wrapped in async query functions so a CMS/BFF can later replace the implementation without touching consumers.

The architecture is **server-first with client islands**: route group `(marketing)` for public pages, `(portal)` for the Phase 8 mock, `(legal)` for KVKK/cookies. i18n lives in URL segments (`app/[lang]/...`) — locale detection happens once in `proxy.ts` (the Next 16 successor to `middleware.ts`) and never in a layout, so static prerender remains intact. Client `'use client'` boundaries are pushed to the smallest leaves (theme toggle, locale switcher, filters, calculator, R3F canvas, map, AI chat). Filter state lives in URL `searchParams`; theme lives in `next-themes` + `class="dark"` on `<html>` with the Tailwind v4 `@custom-variant dark` directive; mock data flows top-down as serializable props.

The dominant risks are not technical novelty — they are **training-data drift on Next 16** (params/cookies/headers are now async, `middleware.ts` → `proxy.ts`, no Edge in proxy, Tailwind v4 has no config file, image config defaults silently degrade hero quality), **dark-mode contrast and FOUC** (premium dark palettes routinely fail WCAG AA and flash on hydration), **mock data shape lock-in** (without an upfront `LocalizedString` + slug-based contract, a future CMS swap becomes a rewrite), and **legal/asset hygiene** (image-rights provenance for scraped photos; KVKK-grade cookie banner with equal-weight reject; honeypot+timetrap on every form even without backend). Mitigations are concrete and codified per phase below.

---

## Key Findings

### Recommended Stack

The project ships with the right framework. Research validates keeping Next 16.2.4 + React 19.2.4 + Tailwind v4 (PostCSS-only) + React Compiler stable. The augmentation set is opinionated and small: shadcn/ui (vendored primitives), `next-intl@^4.4` for i18n routing, `next-themes@^0.4.6` for theming, `motion@^12` (NOT `framer-motion`), `lucide-react` icons, `react-hook-form` + `zod@^3.25` + `@hookform/resolvers@^5` for forms, `three`+`@react-three/fiber@^9.6`+`@react-three/drei@^10` for the 3D viewer, `react-map-gl@^8` (`/maplibre` subpath) + `maplibre-gl@^4` for the map, `embla-carousel-react@^8.6` for the project gallery, `date-fns@^4`, `sonner` for toasts, `@playwright/test@^1.58` + `axe-playwright` for E2E + a11y. AI chat (Phase 6) uses Vercel `ai-elements` (shadcn registry, vendored) over a fake `ReadableStream` — the real `ai` SDK is deferred to the next milestone.

**Locked versions (peer-verified compatible):**

| Layer | Pick | Version |
|-------|------|---------|
| Framework | Next.js (App Router) | **16.2.4** (installed) |
| React | React + ReactDOM | **19.2.4** (installed) |
| Compiler | React Compiler | `babel-plugin-react-compiler@1.0.0` (stable) |
| Styling | Tailwind CSS v4 + PostCSS plugin | **`tailwindcss@^4`** + `@tailwindcss/postcss@^4` (installed; no config file) |
| Primitives | shadcn/ui (Tailwind v4 flavor) | latest CLI (vendored) |
| Theming | `next-themes` | `^0.4.6` |
| i18n | `next-intl` | `^4.4.x` |
| Animation | `motion` (renamed from framer-motion) | `^12.x` — import from `motion/react` |
| Forms | `react-hook-form` / `zod` / `@hookform/resolvers` | `^7` / `^3.25` / `^5` |
| 3D | `three` / `@react-three/fiber` / `@react-three/drei` | `^0.180` / `^9.6` / `^10` |
| Map | `react-map-gl/maplibre` + `maplibre-gl` | `^8` / `^4` |
| Carousel | `embla-carousel-react` | `^8.6` |
| Icons | `lucide-react` | `^0.470+` |
| Testing | `@playwright/test` + `axe-playwright` | `^1.58` |

**The 6 most critical Next 16 / React 19 / Tailwind v4 gotchas (all framework-specific, training-data-hostile):**

1. **`params`, `searchParams`, `cookies()`, `headers()`, `draftMode()` are Promises** — every page/layout/og/sitemap must `await` them. Use `PageProps<'/[lang]/projeler/[slug]'>` global type helpers (run `npx next typegen`).
2. **`middleware.ts` is renamed to `proxy.ts`** — runs on Node (Edge runtime is **not supported** in `proxy`). Use this for `next-intl` locale negotiation; never read headers in a layout.
3. **Tailwind v4 has no `tailwind.config.{js,ts}`** — tokens live in `app/globals.css` via `@import "tailwindcss"` + `@theme inline { … }`. `darkMode: 'class'` is replaced by `@custom-variant dark (&:is(.dark *))` in CSS. v3 muscle-memory silently breaks dark mode.
4. **`next/image` config defaults degrade** — `qualities: [75]` only (hero quality 90 is silently coerced), `minimumCacheTTL: 14400`, `imageSizes` no longer includes 16, `images.domains` deprecated (use `remotePatterns`). Set explicitly in `next.config.ts`.
5. **`next/dynamic({ ssr: false })` is a Client-Component-only API** — it errors when called from a Server Component. R3F and MapLibre wrappers must themselves be `'use client'`.
6. **Removed APIs:** `experimental.ppr` → top-level `cacheComponents: true`; `experimental.dynamicIO` → `cacheComponents`; `next/legacy/image`; `next/amp`; `next lint`; `serverRuntimeConfig` / `publicRuntimeConfig`; `revalidateTag('foo')` single-arg form; `images.domains`; synchronous params/cookies/headers. Webpack config without `--webpack` flag fails the build (Turbopack is default).

Plus: **`framer-motion` is frozen** — use `motion` (import from `motion/react`); **no Edge runtime anywhere on this project**; **all parallel-route slots need `default.tsx`**.

### Expected Features

The B2B steel-fabricator landscape (Severfield, Voortman, thyssenkrupp, Schüco, Kingspan, ArcelorMittal) is well-charted. Buyers (airport / bridge / hangar / hotel owners) self-qualify by sector, scan for proof-of-scale within 10 seconds, and abandon any site without project detail pages or filtering. The current erkincelik.com fails on every modern table-stake; the redesign's job is to pass them all and then stack 4–5 differentiators that no peer in TR has.

**Must have (table stakes — missing any = "this firm is small / behind"):**

- Sticky header with logo + primary nav + language switcher + RFQ CTA top-right
- Single static or muted-loop hero (NEVER auto-rotating carousel)
- Capacity / scale number block on homepage (8.400 ton/yıl, 16+ years, ISO certs)
- Featured projects strip + services overview + process diagram + final-CTA band + footer
- **Project detail page per project** (own URL, hero, tonnage/location/year/sector/client metadata block, 3–10-photo gallery, related projects rail) — biggest gap vs current site
- **Project filter** by sector / year / location / tonnage with URL-state for shareability
- **Tabs:** Tamamlanan / Devam Eden
- Sectoral landing pages: Havalimanı / Hangar / Köprü / AVM / Otel / Fabrika
- Services hub + 3 detail pages (Projelendirme / İmalat / Montaj) with inline RFQ CTA
- **Interactive technical tables:** HEB / HEA / IPE / IPN / köşebent / kutu profil / boru, with search / filter / sort + unit converter
- Blog list + post detail with categories / tags / reading time / related posts
- **Contact:** 2 office addresses (Tuzla + Gebze), embedded map, working hours, RFQ form + general contact form, KVKK consent, action-language CTAs ("Teklif Al"), honeypot + time-trap
- **i18n:** TR primary + EN scaffold via URL segments, localized formats, hreflang, canonical URLs
- **Dark mode + light mode** toggle (dark default — primary aesthetic), persisted, respects `prefers-color-scheme`
- **Accessibility WCAG 2.2 AA:** keyboard nav, ARIA, 4.5:1 body / 3:1 large + UI contrast, alt text, `prefers-reduced-motion`, skip link, focus rings
- **Performance:** AVIF/WebP via `next/image`, font subsetting (latin + latin-ext for ç ş ğ ı ö ü), code-split, target Lighthouse ≥ 95 mobile + desktop
- **SEO:** per-page metadata, OG, `Organization` (root), `LocalBusiness` (× 2 offices), `CreativeWork` (project), `Service`, `BlogPosting`, `BreadcrumbList`, `FAQPage`, sitemap with `alternates.languages`, robots.txt
- **KVKK:** cookie banner with equal-weight Accept / Reddet / Tercihler, KVKK aydınlatma metni, çerez politikası page, consent log, scripts gated until consent
- 404 + error boundary pages, branded
- Mock data layer with TypeScript types and async query functions
- Smoke + E2E tests (Playwright) on homepage, project filter+detail, contact form, theme toggle, locale toggle

**Should have (differentiators):**

- **Cinematic industrial-premium aesthetic** (dark default, copper / orange-on-near-black metallic accents, large typography, scroll-driven animations)
- **Steel weight calculator** (profile + length → kg) — Phase 4
- **Profile comparison tool** (HEB 200 vs IPE 240) — Phase 4
- **Interactive project map** (TR + global pins, clustered, sector overlay, click → detail) — Phase 5
- **3D model viewer** for one sample project — Phase 5
- **AI assistant UI mock** — Phase 6
- **Customer portal UI mock** — Phase 8
- **Case study deep-dive** (e.g., İstanbul Cargo Energy 820t) — Phase 3
- Animated capacity number count-up — Phase 1
- Reference logos strip — Phase 3
- Newsletter signup UI (mock) — Phase 3
- Downloadable certificate PDFs + corporate catalog PDF — Phase 3
- Sticky table-of-contents on long content — Phase 3

**Defer / out of scope (explicitly):**

Real LLM / AI Gateway, real Clerk / Auth0, real customer portal backend, headless CMS, real email send for forms, WP content migration beyond project images, hosting / CI / CD, mobile native app, real 3D project models, site search.

**Anti-features (must NOT be built):**

Auto-rotating hero carousel; autoplay video with sound; splash / intro animation; scroll-jacking; eager live-chat pop-up; "Click here / Submit" CTAs; mouse-following cursors; skeuomorphic steel-textured UI; CAPTCHA-on-everything (use honeypot + Turnstile when backend lands); modal pop-ups on first visit; endless one-page scroll; hamburger-only desktop nav; auto-translate via browser hint; building any real backend this milestone.

### Architecture Approach

The target is **server-by-default with thin client islands**. The route tree is `app/[lang]/(marketing | portal | legal)/...` — three route groups so each chrome (full marketing header / portal sidebar / minimal legal) is isolated. Mock data and dictionaries are server-only; client components receive serialized props or read URL `searchParams`. There is **no global client store** — filter state is URL search params, theme is `next-themes` + `localStorage`, locale is the URL segment, calculator / chat / portal state is local `useState`.

**Major components / responsibilities:**

1. **`proxy.ts` (project root)** — Locale negotiation only. Never read headers in a layout.
2. **`app/[lang]/layout.tsx`** — Sets `<html lang>` from awaited `params`, validates locale, mounts `Providers` client wrapper (theme, motion `LazyMotion`, tooltip).
3. **`app/[lang]/(marketing)/...`** — Public pages: `/`, `/kurumsal`, `/hizmetler/[slug]`, `/projeler` + `/projeler/[slug]`, `/sektorler/[sektor]`, `/teknik-bilgiler` + `/teknik-bilgiler/[profil]`, `/blog` + `/blog/[slug]` + `/blog/kategori/[slug]` + `/blog/etiket/[slug]`, `/iletisim`. RSC pages reading from `src/mock/`, leaf Client islands for filters / forms / theme.
4. **`app/[lang]/(portal)/portal/...`** — Phase 8 mock: `/portal/login`, `/portal/dashboard`, `/portal/belgeler`, `/portal/siparisler`. Robots-disallowed.
5. **`app/[lang]/(legal)/...`** — KVKK / çerez politikası / kullanım şartları. Pure RSC, minimal layout.
6. **`src/mock/`** — Typed mock data + async query functions: `listProjects({filters})`, `getProjectBySlug(slug, lang)`, `listProfiles()`, `listBlogPosts()`. Top-of-file `import 'server-only'`. Strings are `LocalizedString = { tr: string; en: string }`. IDs are slugs. Dates are ISO strings.
7. **`src/i18n/`** — `config.ts`, `dictionaries.ts` (`'server-only'`, `getDictionary(lang)`), `dict/{tr,en}.json`. EN starts as parallel scaffold (`[EN-TODO]` markers). Dictionary never imported into a Client Component — strings flow as props.
8. **`src/components/ui/`** — shadcn primitives, vendored.
9. **`src/components/{marketing,tools,portal,providers,seo}/`** — Domain compositions. Heavy interactive bits (`tools/project-map`, `tools/model-viewer`, `tools/ai-chat`) wrapped in a Client wrapper that does `next/dynamic({ ssr: false })`.
10. **`src/lib/{cn,format,slugify,seo,calc/weight}.ts`** — Pure utilities; `lib/calc/weight.ts` is the unit-tested core of Phase 4.

**Mock layer contract (lock these shapes in Phase 1 — non-negotiable):**

```ts
type LocalizedString = { tr: string; en: string };

type Project = {
  slug: string;
  title: LocalizedString;
  summary: LocalizedString;
  body: LocalizedString;
  sector: 'havalimani' | 'hangar' | 'kopru' | 'avm' | 'otel' | 'fabrika' | 'sanayi';
  status: 'tamamlanan' | 'devam-eden';
  year: number;
  client?: string;
  location: { city: string; district?: string; country: string; lat: number; lng: number };
  tonnage: number;
  images: Array<{ src: string; alt: LocalizedString; width: number; height: number }>;
  certifications?: string[];
  publishedAt: string;
};

export async function listProjects(filters?: ProjectFilters): Promise<Project[]>;
export async function getProjectBySlug(slug: string, lang: 'tr' | 'en'): Promise<Project | null>;
```

Same pattern for `BlogPost`, `Service`, `Certification`, `Profile` (HEB/HEA/IPE/IPN with `weightPerMeter`), `Office`. **Profile data is shared between Phase 1+2 weight tables and Phase 4 calculator — define once.**

**Server / client boundary rules:**

- Default = Server Component. `'use client'` only at the smallest leaf.
- No `'use client'` at the top of any layout or page.
- Client Components never import `src/mock/*` or `src/i18n/dictionaries`.
- `error.tsx` files must be Client Components; use the new `unstable_retry` callback.
- `next/dynamic({ ssr: false })` lives only inside a Client wrapper file.
- Filter / sort / pagination state lives in URL `searchParams`.
- Theme: `<html suppressHydrationWarning>`, `next-themes attribute="class" defaultTheme="dark" enableSystem disableTransitionOnChange`, theme-toggle button uses `mounted` guard.
- Animation: `LazyMotion features={domAnimation} strict`, `<m.div>` not `<motion.div>`, only animate `transform` + `opacity`, `<MotionConfig reducedMotion="user">`.

### Critical Pitfalls (top 10)

1. **[Every phase] Next 16 training-data drift** — sync params/cookies/headers, `middleware.ts` named exports, `experimental.ppr`, `next/legacy/image`, `images.domains`, `next lint` in build, single-arg `revalidateTag`, `next/amp`. Mitigation: AGENTS.md doc-read-first rule; run `npx next typegen`; type all routes via `PageProps<'/...'>`.
2. **[Phase 1] `searchParams`/`cookies()`/`headers()` in root layout** opts entire app into dynamic rendering. Locale detection MUST be in `proxy.ts`. Verify by checking `next build` — every marketing route should be `○` (Static).
3. **[Phase 1] Tailwind v4 dark mode silently broken** if you bring v3 muscle memory. Add `@custom-variant dark (&:where(.dark, .dark *))` in `app/globals.css` and pair with `next-themes attribute="class"`.
4. **[Phase 1] Dark-mode FOUC + hydration mismatch** without `<html suppressHydrationWarning>` and a `mounted` guard.
5. **[Phase 1] Mock data shape lock-in** — without `LocalizedString` + slug IDs + ISO dates + async signatures + Zod schemas defined upfront, future CMS swap = rewrite.
6. **[Phase 1] `next/image` config defaults silently degrade hero** — explicit `qualities: [50, 75, 85, 95]`, `formats: ['image/avif', 'image/webp']`, `remotePatterns: []`. Plus a pre-commit `sharp` script that hard-caps source weight (≤ 200KB hero, ≤ 80KB content).
7. **[Phase 1] Image rights for scraped erkincelik.com photos** — "same client, same images" assumption is legally false. Block: written confirmation per image, asset register CSV in repo. Halts Phase 2+ visual work if unresolved.
8. **[Phase 1+2] KVKK cookie banner UX violations** — equal-weight "Reddet"; analytics/Mapbox load only after consent; persisted consent log; footer "Çerez Tercihleri" re-opener. Forms ship with **honeypot + 2-second time-trap** even without backend. Replace reCAPTCHA mention with **Cloudflare Turnstile** when backend lands.
9. **[Phase 2] Premium dark theme contrast failure** — body ≥ 4.5:1, large text ≥ 3:1, non-text UI ≥ 3:1. Test every `@theme` token pair. Reserve metallic accent for icons + decoration, not body.
10. **[Phase 5] 3D viewer + Map bundle / token / a11y trio** — Three.js + drei + R3F + GLTF + DRACO ≈ 300+ KB. Mapbox tokens leak in client bundle. Both lazy via `next/dynamic({ ssr: false })` inside Client wrapper; viewport-gated; poster fallback + WebGL detection. **Recommend MapLibre** to avoid token entirely.

**Honorable mentions:** Sitemap missing `alternates.languages`; wrong Schema.org type (`Product` for projects → use `CreativeWork`/`Service`/`BlogPosting`/`Organization`+`LocalBusiness`); thin SEO content (≥ 800 words/project, ≥ 1000 words/service); animation jank from animating `top`/`left`/`height` (use `transform`+`opacity`); mock auth `localStorage` leaking into "real auth" assumptions in Phase 8 — define `AuthProvider` interface mirroring Clerk's API.

---

## Implications for Roadmap

The 8-phase structure named in PROJECT.md is supported by the research and should be the spine. Phases below (with Phase 1+2 combined per PROJECT.md → 7 roadmap entries):

### Phase 1+2: Çekirdek Site + Tasarım Sistemi (combined)

**Rationale:** Foundation + chrome + content + projects + design system. PROJECT.md merges 1 and 2; ARCHITECTURE.md's Waves 0–3 all land here. Heaviest phase; the next 6 phases each add one feature surface to a now-stable platform.

**Delivers:**
- `proxy.ts` for locale negotiation; `app/[lang]/...` route tree; `(marketing) (portal-skeleton) (legal)` route groups; root + locale layouts; `error.tsx` + `not-found.tsx`; `Providers` client wrapper.
- Stack init: deps from STACK.md; `npx shadcn@latest init`; `npx shadcn@latest add` for ~18 primitives; `npx next typegen`; `npx playwright install`.
- `next.config.ts` hardened: explicit image config, React Compiler, `eslint-plugin-react-compiler`.
- Tailwind v4 setup in `app/globals.css`: `@import "tailwindcss"`, `@custom-variant dark`, `@theme inline { … }` with palette tuned against contrast checker.
- i18n: `next-intl` + `src/i18n/{config,dictionaries}.ts`, `dict/{tr,en}.json` parallel scaffold, locale switcher.
- Mock data layer: `src/mock/types.ts` + Zod schemas + 50+ projects across all sectors with coords + 7 profile datasets (shared with Phase 4) + ~6 blog posts + 3 ISO certifications + 2 offices.
- Image pipeline: `scripts/optimize-images.ts` running `sharp` in `pnpm prebuild`; AVIF + WebP; `blurDataURL` generation.
- Marketing chrome: header, mobile drawer, footer, cookie banner (KVKK-grade), skip-to-content link.
- Marketing pages: anasayfa, kurumsal, hizmetler hub + 3 service detail, **projeler grid + URL-state filter + tabs + detail page**, 6 sektörel landings, teknik-bilgiler (7 interactive tables), blog list + detail, iletisim (2 office cards + Maps embed + RFQ form + general form, KVKK consent, honeypot), legal pages, branded 404 + error pages.
- Theming: dark default; toggle `mounted` guard; `<html suppressHydrationWarning>`; `disableTransitionOnChange`.
- Animation: `LazyMotion strict`, scroll-driven CSS first, `<MotionConfig reducedMotion="user">`, capacity-number count-up.
- SEO: per-page `metadata`, Schema.org JSON-LD, sitemap with `alternates.languages` + hreflang, robots.txt with `/portal/*` disallow, canonical URLs per locale, per-template OG.
- Tests: Playwright E2E + `axe-playwright`.

**Avoids pitfalls:** #1, #2, #3, #5, #6, #8, plus 9 honorable mentions.
**Research flag:** **Skip** — proceed to `/gsd-plan`. Open decision: exact accent palette OKLCH values (visual judgment in planning).

### Phase 3: İçerik & Pazarlama

Reference logos strip; case study deep-dive (İstanbul Cargo Energy 820t); ekip; kariyer; basın arşivi; newsletter UI (mock); blog enhancements (TOC, related posts); certificate PDFs + corporate catalog; per-template OG images; content sprint (≥ 800 words/project, ≥ 1000 words/service + FAQ + `FAQPage` schema). **Avoids:** #24, #25. **Skip research.**

### Phase 4: Mühendislik Araçları

`src/lib/calc/weight.ts` (pure, Vitest-tested); `WeightCalculator` (Combobox + length input + sharable URL); `ProfileTable` enhancement (TanStack Table, CSV export); `ProfileCompare`; unit converter (kg ↔ lb, mm ↔ in). **Skip research.**

### Phase 5: Harita & 3D

- **Project map:** `react-map-gl/maplibre` + `maplibre-gl` (no token); Client wrapper with `next/dynamic({ ssr: false })`; `supercluster`; theme-aware tile style; mobile keyboard + touch zoom; pin a11y.
- **3D viewer:** `three` + `@react-three/fiber@^9.6` + `@react-three/drei@^10`; Client wrapper; viewport-gated; 2D poster fallback + "View in 3D" CTA; WebGL detection; DRACO/meshopt compressed GLTF.

**Research flag:** ⚠️ **Recommend `/gsd-research-phase`** — confirm free MapLibre tile-style URLs for v4, DRACO decoder hosting, R3F v9 Suspense fallback under React Compiler.

### Phase 6: AI Asistan UI (mock)

`npx ai-elements@latest add conversation message prompt-input reasoning code-block`; chat shell; server stub returning mock `ReadableStream`; `AbortController`; reduced-motion-friendly typing indicator; live-region announcements. **Skip research.**

### Phase 7: Canlı Üretim & Medya (mock)

Video gallery / fabrika tour (`<video muted loop playsInline poster controls>`, lazy below fold, captions placeholder); üretim süreci interaktif diyagramı (SVG with hover/click + scroll-driven step reveal); mock "canlı akış" placeholder ("yakında" card with newsletter CTA). **Skip research.**

### Phase 8: Müşteri Portalı UI (mock)

`(portal)/layout.tsx` (sidebar + topbar); `/portal/login`, `/portal/dashboard`, `/portal/belgeler`, `/portal/siparisler`; `default.tsx` for parallel slots; `src/lib/auth/` with Clerk-shape API (`useUser`, `useAuth`, `<SignedIn>`, `<SignedOut>`); robots disallow `/portal/*`; sitemap excludes; dev banner "Mock UI — no real auth". **Optional research** to lock future-Clerk interface precisely.

### Phase Ordering Rationale

- **Phase 1+2 first** — design system + çekirdek share primitives, mock contract, routing shell.
- **Phase 3 second** — content depth requires Phase 1+2 templates.
- **Phase 4 before 5/6/7/8** — pure logic; lowest-risk domain feature; profile data shared with Phase 1+2 tables.
- **Phase 5 after 4** — first `next/dynamic({ ssr: false })` inside Client wrapper; highest bundle-risk; primitives + mock shapes must be stable.
- **Phase 6 after 5** — AI chat reuses chat-bubble + typing-indicator design language.
- **Phase 7 after 6** — lowest-risk; sequenced for AI mock first claim on floating-widget pattern.
- **Phase 8 last** — new route group + parallel routes; highest "looks-done-but-isn't" risk.

### Research Flags Summary

| Phase | Flag | Reason |
|-------|------|--------|
| Phase 1+2 | **Skip** | All decisions locked; only palette OKLCH tuning is open |
| Phase 3 | **Skip** | Standard content / OG / case-study patterns |
| Phase 4 | **Skip** | Pure logic; standard TanStack Table |
| Phase 5 | **⚠️ Recommend** | R3F v9 + drei v10 + MapLibre dark tiles under React Compiler |
| Phase 6 | **Skip** | AI Elements well-documented |
| Phase 7 | **Skip** | Standard HTML5 video + SVG |
| Phase 8 | **Optional** | Lock future-Clerk interface precisely |

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | **HIGH** | Next 16 specifics from local docs; library versions verified; peer-compat matrix corroborated |
| Features | **HIGH** | 8 reference sites inspected directly; 2026 B2B literature corroborated; PROJECT.md mapped |
| Architecture | **HIGH** | Next 16 conventions verified against local docs; route-group / boundary patterns match official guides |
| Pitfalls | **HIGH** | Next 16 / React 19 / Tailwind v4 verified; KVKK + scraping legal exposure verified; motion / 3D / map MEDIUM |

**Overall:** **HIGH** for proceeding to `/gsd-roadmap` and Phase 1+2 planning.

### Gaps to Address During Planning

- **Hosting decision** — `output: 'export'` is incompatible with project requirements. **Default to standard Node-server build.** Document in PROJECT.md. Re-validate end of Phase 7.
- **Map library final pick** — research strongly recommends MapLibre (no token). Pick MapLibre by default; revisit only if Phase 5 tile-style review finds free options inadequate.
- **3D library final scope** — recommend R3F for consistency. Confirm in Phase 5 research.
- **Project image rights** — must resolve BEFORE Phase 2 visual work begins. Owner-confirmed-in-writing per scraped image; asset register CSV in repo.
- **CAPTCHA strategy** — reCAPTCHA is KVKK-conflicting; BotID is Vercel Pro+. **Phase 1+2 ships honeypot + 2-second time-trap**; plan Cloudflare Turnstile when backend lands.
- **Future AI SDK target** — Phase 6 mock matches Vercel AI SDK's `streamText` shape; document the contract in Phase 6 planning.

---

## Sources

### Primary (HIGH confidence — `node_modules/next/dist/docs/`)
`01-app/02-guides/upgrading/version-16.md`; `01-app/01-getting-started/{02-project-structure,05-server-and-client-components,10-error-handling,11-css,12-images}.md`; `01-app/02-guides/{internationalization,lazy-loading,json-ld,static-exports,forms,production-checklist,testing/playwright}.md`; `01-app/03-api-reference/03-file-conventions/{01-metadata/sitemap,proxy,dynamic-routes}.md`; `01-app/03-api-reference/05-config/01-next-config-js/cacheComponents.md`; `01-app/03-api-reference/07-edge.md`; `node_modules/next/package.json`.

### Secondary (MEDIUM-HIGH confidence)
shadcn/ui Tailwind v4 + Next.js install + dark-mode docs; next-intl GitHub README; next-themes README; Motion upgrade guide; React 19 release notes; Tailwind v4 upgrade + dark-mode docs; r3f installation + bundle-reduction; @react-three/fiber + drei npm; react-map-gl What's New (v8 subpath); MapLibre/Mapbox/Leaflet 2026 comparison + Mapbox pricing; Vercel AI Elements; Embla; Severfield/Voortman/thyssenkrupp/Schüco/Kingspan inspected; 2026 B2B/RFQ/trust-signal literature; carousel anti-pattern evidence; KVKK 2025 amendments + draft cookie guidelines; web scraping copyright; Schema.org for contractors.

### Tertiary (LOW-MEDIUM confidence)
LogRocket animation 2026; Frontend Hero icons 2026; DEV Next.js 16 / next-intl / RHF+Zod 2026; Aurora Scharff `'use cache'` + next-intl caveats; CSS dark-mode FOUC blogs.
