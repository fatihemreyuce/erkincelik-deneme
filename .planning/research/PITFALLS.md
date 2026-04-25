# Pitfalls Research

**Domain:** Premium B2B marketing site (steel construction) — Next.js 16 + React 19 + Tailwind v4 + i18n + dark mode + heavy imagery + scroll animations + 3D + mock data
**Researched:** 2026-04-25
**Confidence:** HIGH (Next 16 / React 19 / Tailwind v4 specifics verified against `node_modules/next/dist/docs/`); MEDIUM on motion/3D/map ecosystem (web sources, multiple agreement)

> **Companion to AGENTS.md warning:** Every pitfall flagged with `[READ DOCS FIRST]` requires reading the relevant file under `node_modules/next/dist/docs/` before writing code, because Next.js 16 has breaking changes from any version Claude has training data on.

---

## Critical Pitfalls

### Pitfall 1: Treating Next.js 16 like Next.js 14/15 (training-data drift) `[READ DOCS FIRST]`

**What goes wrong:**
Synchronous `params.slug`, `cookies()`, `headers()`, `searchParams.foo`. Sync `params` in `opengraph-image`/`icon`/`sitemap` generators. `middleware.ts` named exports. `experimental.ppr` flag. `next/legacy/image` imports. `images.domains` config. `next lint` in build pipeline. `serverRuntimeConfig`/`publicRuntimeConfig`. `experimental.dynamicIO`. `unstable_rootParams`. `unstable_cacheLife`/`unstable_cacheTag`. Single-arg `revalidateTag('posts')`. `next/amp` imports. — All of these *worked in Next 15* and **break or are removed in Next 16**.

**Why it happens:**
Claude (and any contributor) has training-data primarily from Next 14/15. The official `AGENTS.md` directly warns: *"This is NOT the Next.js you know."* Sample/blog code online is overwhelmingly pre-16.

**How to avoid:**
1. **Before** writing any new route/layout/page/proxy/sitemap/icon/og/route-handler, run `Read` on the matching file in `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/` or `04-functions/`.
2. Always type props with `PageProps<'/route'>` / `LayoutProps<'/route'>` / `RouteContext<'/route'>` (run `npx next typegen` after creating routes).
3. `params`/`searchParams` are **always Promises** — `const { slug } = await params`.
4. Use `proxy.ts` (not `middleware.ts`) and rename function to `proxy`.
5. Use `revalidateTag('tag', 'max')` two-argument form; for read-your-writes use `updateTag` in Server Actions.
6. Replace `images.domains` with `images.remotePatterns`.

**Warning signs:**
- TypeScript error: `Property 'slug' does not exist on type 'Promise<...>'`
- Runtime: `params is a Promise. Use await params...`
- Build failure: *"webpack configuration found, build will fail"* (Next 16 default is Turbopack)
- ESLint warning: `revalidateTag` called with single arg

**Phase to address:** Phase 1 (foundation — establishes the convention) and **enforced every phase** via the AGENTS.md doc-read-first rule.

---

### Pitfall 2: `searchParams` / dynamic API in root layout opts entire app into dynamic rendering

**What goes wrong:**
Reading `cookies()`, `headers()`, or `searchParams` in `app/[lang]/layout.tsx` (or accessing them via i18n locale-detection logic) cascades dynamic rendering to every page. `next build` produces fewer/no static pages, Lighthouse drops, hosting cost rises (if eventually deployed serverless), and prefetching benefits disappear.

**Why it happens:**
Locale detection is a tempting place to read the `Accept-Language` header in the root layout. This silently pessimizes everything below it.

**How to avoid:**
- Do locale detection in `proxy.ts` (redirect-only) — *never* read headers in the layout.
- Use `[lang]` segment in URL (`/tr/...`, `/en/...`) and build static pages via `generateStaticParams()`.
- Wrap any unavoidable request-time read in a `<Suspense>` boundary so only that subtree is dynamic.
- For form-with-search-results pages, use `searchParams` at the *page* level only, not the layout.

**Warning signs:**
- `next build` output shows ƒ (Dynamic) instead of ○ (Static) for marketing routes.
- Build log: *"Route ... could not be statically generated"*.
- All routes show as dynamic when only one needed to be.

**Phase to address:** Phase 1 (i18n scaffold). Verify in Phase 2 build by checking build output table.

---

### Pitfall 3: `next/image` quality / minimumCacheTTL / imageSizes Next 16 defaults silently break existing images

**What goes wrong:**
Designer specifies `quality={80}` or `quality={90}` on hero images → coerced to 75 in Next 16 (default `qualities: [75]`) → noticeably softer hero. `minimumCacheTTL` jumped 60s → 4h, so updates to scraped images don't refresh during dev/staging. `imageSizes` no longer includes 16, breaking certain icon srcsets.

**Why it happens:**
These are silent Next 16 breaking changes — code keeps working but visuals/cache change. Documented in `docs/01-app/02-guides/upgrading/version-16.md` lines 755–815.

**How to avoid:**
Set explicitly in `next.config.ts`:
```ts
images: {
  qualities: [50, 75, 85, 95],          // re-enable premium-quality hero images
  minimumCacheTTL: 60,                  // for dev iteration; raise in prod
  imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],  // restore 16 if icons need it
  formats: ['image/avif', 'image/webp'],
  remotePatterns: [],                   // no domains; deprecated
  localPatterns: [{ pathname: '/projects/**' }],    // if using ?v= cache busts
}
```

**Warning signs:**
- "Hero looks softer than the Figma comp" — likely quality coercion.
- Hard-refresh required to see new image during dev.

**Phase to address:** Phase 1 (image config), validated in Phase 2 (design system + visual QA).

---

### Pitfall 4: Missing `default.tsx` for parallel routes → build failure in Next 16

**What goes wrong:**
Customer-portal mock or modal-as-intercepted-route uses parallel slots (`@modal`, `@dashboard`). In Next 15 these worked without `default.js`; in Next 16 **builds fail** unless every slot has an explicit `default.tsx`.

**Why it happens:**
Documented in `version-16.md` as a hard breaking change. Easy to forget when copying patterns from Next 15 examples.

**How to avoid:**
For each parallel slot folder, create `default.tsx` returning `null` or calling `notFound()`:
```tsx
// app/@modal/default.tsx
export default function Default() { return null }
```

**Warning signs:**
- Build error: *"Missing default.js file for parallel route ..."*

**Phase to address:** Phase 8 (customer portal mock — most likely consumer of parallel routes).

---

### Pitfall 5: Tailwind v4 dark mode silently broken when migrating v3 patterns

**What goes wrong:**
Designer/dev uses `darkMode: 'class'` in a `tailwind.config.js` (which doesn't exist in v4 PostCSS-only setup) → config is ignored → `dark:` variant defaults to `prefers-color-scheme: dark` → manual toggle ("force dark mode") doesn't work → users in light-mode OS can't switch site to dark even after clicking the toggle.

**Why it happens:**
Tailwind v4 has **no `tailwind.config.js`** in the new PostCSS-only setup. Configuration moves to CSS via `@theme` and `@custom-variant`. v3 muscle memory, and most online tutorials still show v3 syntax.

**How to avoid:**
In `app/globals.css`:
```css
@import 'tailwindcss';

/* For class-based dark mode (manual toggle) */
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-bg: oklch(0.12 0.02 250);
  --color-fg: oklch(0.95 0 0);
  --color-accent: oklch(0.72 0.14 50);  /* metallic copper */
  /* ... */
}
```
Then pair with `next-themes` configured `attribute="class"`.

**Warning signs:**
- Toggle button updates state but nothing visually changes.
- `dark:` utilities only apply when OS is in dark mode (system preference).
- Build/HMR shows dark variant emitting `@media (prefers-color-scheme: dark)` instead of `.dark &`.

**Phase to address:** Phase 1 (design system / globals.css setup).

---

### Pitfall 6: Dark mode FOUC + hydration mismatch with `next-themes` + RSC

**What goes wrong:**
Site renders in light mode for ~200ms → flips to dark on hydration. Console: *"Hydration failed because the initial UI does not match what was rendered on the server"*. Looks unprofessional on a "premium" B2B site — exactly the audience that'll judge polish harshly.

**Why it happens:**
Server cannot read `localStorage` or system preference, so SSR always renders default theme. `useTheme()` returns `undefined` until mounted. If you render theme-dependent UI without `mounted` guard, hydration mismatches.

**How to avoid:**
1. **Wrap only the body content** with `<ThemeProvider>`, keep `<html>` and `<body>` static.
2. Add `suppressHydrationWarning` to `<html>` element.
3. Theme toggle buttons must use a `mounted` state pattern:
   ```tsx
   const [mounted, setMounted] = useState(false)
   useEffect(() => setMounted(true), [])
   if (!mounted) return <div className="size-9" />  // skeleton same size
   ```
4. Configure provider:
   ```tsx
   <ThemeProvider attribute="class" defaultTheme="dark" enableSystem disableTransitionOnChange>
   ```
5. Verify the `next-themes` injected `<script>` runs before paint (it does by default).

**Warning signs:**
- White flash on initial paint (look for it on slow 3G in DevTools).
- React hydration warning in console.
- Theme-toggle icon swaps after a beat.

**Phase to address:** Phase 1 (root layout + providers). Add to Phase 2 visual-QA checklist.

---

### Pitfall 7: Static export incompatibility for chosen features (kills `output: 'export'` route)

**What goes wrong:**
Team picks `output: 'export'` for cheap static hosting (Cloudflare Pages, S3) → discovers later that `cookies`, `headers`, `proxy`, `revalidate`, on-demand image optimization, intercepting routes, and dynamic routes without `generateStaticParams` are unsupported. Forces a hosting pivot mid-project.

**Why it happens:**
`PROJECT.md` says hosting decision is deferred. If team optimistically codes assuming static export, then needs `proxy.ts` for i18n, `next/image` optimization for arbitrary remote images, or any cookie-based UX later, it's a rewrite.

**How to avoid:**
- **Do not commit to `output: 'export'`** until the feature surface is frozen.
- Default to **standard build** (`next start` or Node-server-compatible host like Coolify/Dokku/Vercel/Railway).
- If static export is mandatory: i18n via static segment + `generateStaticParams`, no `proxy.ts`, custom image loader (Cloudinary or similar), no cookies, no Server Actions.

**Warning signs:**
- Build error on `proxy.ts` matching when `output: 'export'`.
- "Image optimization not supported" warning.
- Dynamic route lacking `generateStaticParams` errors at build.

**Phase to address:** Phase 1 (decide & document hosting model assumptions in `STACK.md`); revisit Phase 9 (deploy).

---

### Pitfall 8: i18n hydration mismatch from server-vs-client locale drift

**What goes wrong:**
Server renders Turkish strings (from `getDictionary('tr')`), but client picks up `navigator.language` or stored cookie → renders English on hydration → flash of wrong language + hydration error. Or: dictionary loaded as JS bundle on client, doubling page weight.

**Why it happens:**
- Client component reads navigator/localStorage instead of trusting URL `[lang]` segment.
- Mixing `next-i18next` (Pages-router-era) with App Router.
- Dictionary imported into client component → bundled into JS.

**How to avoid:**
- **URL-based locale only** (`/tr/...`, `/en/...`). Toggle = `<Link>` to other locale variant, not `localStorage`.
- `getDictionary` must be `import 'server-only'` and only called in Server Components.
- For Client Components needing strings, pass them as props from a Server Component parent.
- Add `<html lang={lang}>` from layout `params.lang` (after `await`).
- For TR/EN scaffold without translation content: keep one `dictionaries/tr.json` + identical `dictionaries/en.json` placeholder. Don't fall back at runtime — keep them parallel.

**Warning signs:**
- Hydration mismatch with text content diff in console.
- Bundle analyzer shows `dictionaries/*.json` in client chunks.
- `<html lang="">` is empty or wrong on view-source.

**Phase to address:** Phase 1 (i18n scaffold).

---

### Pitfall 9: Sitemap missing `alternates.languages` → no hreflang → poor Turkish/English SEO split

**What goes wrong:**
Google indexes `/tr/projeler` and `/en/projects` as separate competing pages instead of language alternates of the same canonical entity → keyword cannibalization, lost ranking, wrong language served to wrong audience in search results.

**Why it happens:**
`app/sitemap.ts` returns plain `url` entries without `alternates.languages` map. Many examples online predate the feature.

**How to avoid:**
Use `alternates.languages` per entry (Next 16 emits proper `<xhtml:link rel="alternate" hreflang="...">`):
```ts
{
  url: 'https://erkincelik.com/tr/projeler',
  lastModified: new Date(),
  alternates: {
    languages: {
      tr: 'https://erkincelik.com/tr/projeler',
      en: 'https://erkincelik.com/en/projects',
      'x-default': 'https://erkincelik.com/tr/projeler',
    },
  },
}
```
Validate with Google Search Console + `https://technicalseo.com/tools/hreflang/`.

**Warning signs:**
- `/sitemap.xml` view-source has no `xhtml:link` elements.
- Google Search Console "International Targeting" tab shows hreflang errors.

**Phase to address:** Phase 2 (SEO setup) — but stub structure in Phase 1.

---

### Pitfall 10: Schema.org type chosen wrong → Rich Results don't appear

**What goes wrong:**
Using `Product` schema for steel-construction projects (which are *services/works*, not products in the e-commerce sense) → Google rejects rich results. Or putting `LocalBusiness` on every page instead of just contact/about → diluted signals. Missing `Organization` on root layout → no knowledge-panel data.

**Why it happens:**
`Product` is the schema most devs reach for. Steel-construction project = a *built work*, closer to `CreativeWork` / `Project` (not in core schema.org) / hybrid `Service` + `Place`. The B2B firm itself is `Organization` + `LocalBusiness` (because two physical addresses: Tuzla + Gebze).

**How to avoid:**
- **Root layout (`app/layout.tsx`):** single `Organization` JSON-LD + nested `address` array for both offices.
- **`/iletisim` (Contact page):** `LocalBusiness` for each office (two separate JSON-LDs or `branchOf` reference).
- **Project detail pages:** `CreativeWork` (or `Project` from extension) with `creator`, `location`, `dateCompleted`, `image`. *Not* `Product`.
- **Service pages:** `Service` with `provider` reference back to `Organization`.
- **Blog posts:** `BlogPosting` with `author` (Organization) + `datePublished`.
- Sanitize `<` → `<` in JSON-LD output (per Next docs `json-ld.md` line 11).
- Use `schema-dts` for TypeScript safety.
- Test every page in https://search.google.com/test/rich-results before claiming "SEO done".

**Warning signs:**
- Rich Results Test shows 0 valid items.
- Search Console flags "missing field" errors weeks after launch.

**Phase to address:** Phase 2 (SEO foundations); revisit per content-type as Phases 3+ add new templates.

---

### Pitfall 11: Scroll-driven animations cause CLS, jank on mobile, and accessibility violations

**What goes wrong:**
Hero parallax/scroll effect ships beautifully on dev laptop → on a mid-range Android in 4G, frames drop to 15fps → "premium" turns into "broken". Text reveals trigger layout shift, blowing CLS budget (Lighthouse 95+ goal misses). User with vestibular disorder gets nauseated; site has no `prefers-reduced-motion` opt-out → accessibility complaint and KVKK-adjacent-but-WCAG real exposure.

**Why it happens:**
- Animating `top`/`left`/`height` (layout-triggering properties) instead of `transform`/`opacity`.
- Loading entire Framer Motion / GSAP into the initial bundle.
- Forgetting `prefers-reduced-motion`.
- Stacking too many simultaneous scroll triggers.

**How to avoid:**
- **Animate only `transform` + `opacity`.** Set fixed sizes (or `aspect-ratio`) on slots before the animation paints.
- **Wrap `motion.*` components in `'use client'` files**, lazy-load the heavy hero with `next/dynamic({ ssr: false })`.
- **Global `<MotionConfig reducedMotion="user">`** in providers.
- **CSS `@media (prefers-reduced-motion: reduce)`** path that flattens all decorative animations to fades/instant.
- **Performance budget:** ≤ 50KB added by motion libs, ≥ 55fps on a Moto G Power class device, CLS impact ≤ 0.05.
- **Prefer CSS `@scroll-timeline` / Tailwind v4 animation utilities** for simple effects instead of JS lib.
- Test in Chrome DevTools Performance panel with 4× CPU throttle.

**Warning signs:**
- Lighthouse "Avoid large layout shifts" diagnostic flags hero.
- Long task warnings (>50ms) on scroll.
- Mobile users report "site stutters when I scroll".
- Reduced-motion user sees the same parallax.

**Phase to address:** Phase 2 (design system establishes motion primitives + `MotionConfig`); Phase 5 (3D viewer adds bigger trap).

---

### Pitfall 12: 3D viewer (R3F + Three.js) tanks bundle, breaks on low-end devices

**What goes wrong:**
Three.js (~155 KB gzipped) + drei + R3F + GLTF + DRACO = instant 300+ KB JS hit on every page if not isolated. Initial load drops below Lighthouse 95. iOS Safari simulator crashes (incomplete WebGL ES); old Android shows black canvas. Customer on a tablet at a construction site sees "loading…" forever.

**Why it happens:**
- Importing R3F at top-level of a page that should be static.
- No suspense fallback; no fallback when `WebGL` is unsupported.
- Loading huge GLTF synchronously (no DRACO/meshopt compression).
- Running on every render, not just when user scrolls into the canvas.

**How to avoid:**
- **`const Viewer = dynamic(() => import('@/components/3d/viewer'), { ssr: false, loading: () => <PosterImage /> })`** — never SSR R3F.
- **Render only when scrolled in viewport** via `react-intersection-observer` or `IntersectionObserver`.
- **Provide a 2D poster image fallback** that's always shown until user opts-in ("View in 3D" CTA).
- **Detect WebGL support**, hide button if absent.
- **Use compressed GLTF** (DRACO/meshopt) and lazy-load DRACO decoder.
- **Cap pixel ratio** `<Canvas dpr={[1, 2]}>`, disable anti-alias on mobile.
- **Test on real iOS device 16+ minimum** (PROJECT.md scope = modern browsers; document the floor).
- **Bundle budget:** entire 3D feature must lazy-load and not affect non-3D pages.

**Warning signs:**
- Bundle analyzer: three / @react-three in main chunk.
- Lighthouse "Reduce unused JavaScript" flagging > 100KB.
- Mobile: black canvas, console errors about EXC_BAD_ACCESS or WebGL context lost.

**Phase to address:** Phase 5 (3D viewer is its own phase) — define the bundle isolation contract there.

---

### Pitfall 13: Map embeds (Mapbox) leak token in client bundle and rack up bills

**What goes wrong:**
`NEXT_PUBLIC_MAPBOX_TOKEN` ships to every client. Token is reusable by anyone who view-sources. Spike in load events → free-tier (50K/month) blown → site "breaks" or charges hit owner. For a B2B site with many project pins, even legitimate traffic flips the meter quickly.

**Why it happens:**
Mapbox/Google Maps both require a key for tile rendering, and that key must be in the browser. There's no "secret" version of a tile key.

**How to avoid:**
- **Restrict the Mapbox token by URL** (Mapbox dashboard → Tokens → URL restrictions = `*.erkincelik.com`). Now stolen tokens can't be used elsewhere.
- **Prefer MapLibre GL JS + free tile provider (MapTiler free tier, OSM tiles) when no proprietary tiles are needed** — no token, no rate limit on the rendering library.
- **Cluster pins** (`supercluster`) — 50 markers ≠ 50 DOM nodes; render one cluster.
- **Lazy-load the map** (`dynamic(() => import('./map'), { ssr: false })`) — page doesn't pay map cost unless user scrolls.
- Set up a Mapbox **usage alert** at 50% of free tier.

**Warning signs:**
- View-source shows token in HTML.
- Mapbox dashboard usage spiking with no marketing campaign.
- Map pins visibly stutter when many are present.

**Phase to address:** Phase 5 (project map). Decide MapLibre-vs-Mapbox in `STACK.md`.

---

### Pitfall 14: Scraping erkincelik.com images without explicit IP clarity

**What goes wrong:**
"Same client, so it's fine" → except: the photographer who shot the projects holds copyright unless transferred; project owners (airport authority, hotel chain) may hold publishing rights to their project's images; the original web designer (Vesna Bilgi Teknolojileri) may have a license that's not transferable. Statutory damages for willful copyright infringement can reach $150K per work in some jurisdictions; in TR similar exposure under Law No. 5846. A *single* takedown letter post-launch destroys the launch.

**Why it happens:**
Defaulting to "we're rebuilding the same site for the same client = same content" without checking the chain of rights.

**How to avoid:**
- **Get written confirmation** from Erkin Çelik that they own (not just "have used") the images, OR that the photographer/owner consented to indefinite republishing on a new domain/design.
- **Maintain an asset register** (CSV: image filename → source → rights status → permission proof) for every scraped image.
- **For project images where third-party owns the building:** typically the steel-fabricator's own working photos are fine; finished-building beauty shots from owner's marketing might not be. When in doubt, don't ship.
- **Watermark/attribute** in EXIF if reusing photographer-shot images.
- **Do not scrape image *files* — re-export from owner's local archive when possible** (also lets you get higher-res versions than web-compressed copies).

**Warning signs:**
- No documented permission in repo for any scraped image.
- File metadata shows external photographer name / agency watermark.

**Phase to address:** Phase 1 (asset acquisition discipline) — *blocks* Phase 2+ visual work if not resolved.

---

### Pitfall 15: KVKK cookie banner UX violations (especially "reject all" missing or buried)

**What goes wrong:**
Banner only has "Accept" → KVKK + 2025 amendments + GDPR-aligned expectations require **freely given, specific, informed consent**, with reject as easy as accept. Türkish DPK (KVKK Kurumu) has issued draft cookie guidelines aligning with GDPR; B2B doesn't exempt you. Fine for non-compliance can reach millions TL.

**Why it happens:**
- Designer treats banner as cosmetic, not legal artifact.
- "Continue browsing = consent" pattern (banned under both KVKK and GDPR).
- Pre-ticked checkboxes for non-essential cookies.
- No way to withdraw consent later.

**How to avoid:**
- **Banner shows "Kabul Et" + "Reddet" + "Tercihler" with equal visual weight.** No "X" close that defaults to accept.
- **No non-essential scripts (Analytics, Mapbox tracking, hotjar) load until explicit consent.** Use Next.js `<Script strategy="lazyOnload">` gated on consent state.
- **Persist consent log** with timestamp + version of policy + categories (functional / analytics / marketing). For a frontend-only milestone, store in `localStorage` with future-API-shape JSON; flag in `STACK.md` that real CMP comes later.
- **Footer link to "Çerez Tercihleri"** that re-opens banner.
- **Privacy policy + cookie policy** as separate pages, linked from banner first sentence.
- Forms: KVKK aydınlatma metni (informed-consent text) above submit, with a separate non-prechecked checkbox for marketing-related consent if used.

**Warning signs:**
- "Reject" button missing or hidden.
- Network tab: GA / Mapbox / etc. firing before user clicks "Accept".
- Pre-checked consent checkboxes.

**Phase to address:** Phase 2 (cookie banner + KVKK metni are explicit Phase 1+2 requirements per PROJECT.md). Verify with a network-tab consent test.

---

### Pitfall 16: Form spam without backend → demo-stage data harvesting, post-launch flood

**What goes wrong:**
RFQ + contact form UI is built (`Phase 1+2 requirement`) without rate limiting because *"there's no backend yet"*. Attackers find form via crawler, submit thousands of times to whatever endpoint it eventually proxies to. When real backend lands later, inbox is already poisoned by bot signups.

**Why it happens:**
Mock-data milestone defers backend, so spam protection feels premature. But the *form HTML and field structure* is what spammers fingerprint — once it ships, it's discoverable.

**How to avoid:**
- **Honeypot field** (CSS-hidden, `aria-hidden="true"`, `tabindex="-1"`, name `website` or `company_url`) on every form. Reject submissions where it's filled.
- **Time-trap** (record render timestamp; reject if submission < 2s after render — humans take longer).
- **Origin-bound submission** when backend lands: same-site cookie + CSRF token.
- **Cloudflare Turnstile** (free, accessible, no Google) is a good "real" CAPTCHA when hosting decision is made. PROJECT.md mentions reCAPTCHA — flag as suboptimal: poor a11y, Google data sharing, KVKK conflict (sends data to US). **Replace with Turnstile** in Stack research.
- **Vercel BotID** is only available on Vercel Pro+; not free.
- **Plan for `proxy.ts` rate limiting** (memory-store stub now; Redis later) — interface designed in mock phase, real wiring later.

**Warning signs:**
- No honeypot input in form HTML.
- Submission accepted with empty/instant timestamp.

**Phase to address:** Phase 1+2 (form scaffold). Document the no-backend-but-anti-spam-nonetheless contract.

---

### Pitfall 17: Premium dark theme with low contrast → fails WCAG AA, looks "moody but unreadable"

**What goes wrong:**
Designer picks `#0F0F12` background + `#9A9A9A` text → contrast ratio 4.06:1 (fails AA for body 4.5:1). Metallic copper accent on dark = `#C77845` on `#0F0F12` = 5.2:1 (passes for large text only). Customer in his 50s reading an RFQ form on a Tuzla-office monitor in afternoon sun gives up. PROJECT.md: WCAG AA explicitly required.

**Why it happens:**
"Premium dark" aesthetics pull toward smoky, low-contrast palettes (Stripe, Linear, Vercel). Those companies have huge design teams calibrating millimetrically. Imitating without checking contrast ratios fails.

**How to avoid:**
- **Define palette with contrast first.** Body text against bg: ≥ 4.5:1. Large text (18.66px+ regular / 14px bold): ≥ 3:1. Non-text UI (button borders, focus rings, icons): ≥ 3:1.
- **Test every token pair** in `app/globals.css` `@theme` with a contrast checker (e.g. `npm i -D @tailwindcss/colors-contrast`, or Stark in Figma).
- **Watch for accent-on-dark gotchas:** copper/orange (the brand metallic) tends to fail on near-black; bump saturation and lightness, or reserve for icons + non-essential decoration only.
- **Avoid dark-on-dark for forms.** Inputs need a clearly-elevated surface (e.g., `bg-neutral-900` body + `bg-neutral-800` input + `border-neutral-700` border).
- **Never use pure white on near-black** for body — eye strain. Use `oklch(0.94 0 0)` family.
- **Provide system preference + manual override** so users in bright environments can flip to light.

**Warning signs:**
- Lighthouse a11y score < 95.
- Axe DevTools flags "Insufficient contrast".
- Beta users say "looks great but I can't read the small print".

**Phase to address:** Phase 2 (design system).

---

### Pitfall 18: Mock data shape diverges from likely future API → expensive refactor when CMS lands

**What goes wrong:**
`projects.ts` mock has `{ id, title, ton, location }` → CMS (Sanity/Payload, per PROJECT.md out-of-scope deferred) returns `{ _id, slug, title: { tr, en }, capacity_tons, location: { city, district, country, lat, lng } }`. Every component that consumed mock now needs rewrite. i18n strings were hard-coded in mock as just Turkish, now must be locale-mapped. A 2-week "swap mocks for real API" turns into a month.

**Why it happens:**
- Mock is written for current screen, not future shape.
- Strings hard-coded in TR rather than nested as `{ tr: '...', en: '...' }`.
- IDs as numbers instead of slugs.
- No type-only seam between data layer and UI.

**How to avoid:**
- **Define types in `types/api.ts` *first*** — model them after a likely CMS shape (slugs not numeric IDs, localized strings as `{ tr, en }` or `LocalizedString` type, dates as ISO strings, refs as `{ _ref }` or `slug`).
- **Mock implements the type, doesn't define it.** `getProjects(): Promise<Project[]>` always async, so swap is one-line.
- **All data access through a single layer** (e.g., `lib/data/projects.ts`) — components never import mock JSON directly.
- **Localize strings in the mock itself** (`title: { tr: 'X', en: 'X' }`) even if EN is just placeholder — forces correct shape.
- **Use Zod schemas** for runtime + compile-time validation. Same schema validates mock and real API later.

**Warning signs:**
- Components import from `data/projects.json` directly.
- Strings appear in component code instead of via dictionary or data.
- Mock returns sync data (real API will be async).

**Phase to address:** Phase 1 (mock data layer is in scope) — establishes contract before any consumer.

---

### Pitfall 19: Heavy imagery from scrape → oversized hero kills LCP

**What goes wrong:**
Designer drops 4MB JPEG hero (scraped from old site, full-res) → LCP > 4s on 4G → Lighthouse 50s → "premium" first impression destroyed. Even with `next/image`, the source weight matters because optimization is bound by source quality+size.

**Why it happens:**
- Scraped images are whatever the WordPress theme served, often unoptimized.
- Designer uses original asset; nobody runs `sharp` over it pre-commit.
- `priority` set on multiple above-fold images → all download eagerly, contention.

**How to avoid:**
- **Pre-process every committed image** with `sharp` to AVIF (≤ 200KB hero, ≤ 80KB content) + WebP fallback + JPEG legacy. Pipeline: `scripts/optimize-images.ts` runs in `pnpm prebuild`.
- **Cap source dimensions** at 2× the largest layout (e.g., 2560px wide for full-bleed hero).
- **`priority` on exactly one above-fold image** per page (the hero).
- **Always set `sizes` prop** matching the layout (`sizes="(max-width: 768px) 100vw, 60vw"`).
- **Use `placeholder="blur"`** for static imports; for dynamic imports use a tiny pre-computed `blurDataURL`.
- **`fetchPriority="high"`** explicitly on hero (Next.js sets when `priority`).
- **Lighthouse run on every PR** (CI gate at LCP < 2.5s, CLS < 0.05).

**Warning signs:**
- Network panel: hero image > 500KB.
- Lighthouse "Properly size images" / "Serve images in next-gen formats" flags.
- LCP > 2.5s on slow 3G simulation.

**Phase to address:** Phase 2 (visual content + image pipeline) — before Phase 3 (more content).

---

### Pitfall 20: Modal/dialog/dropdown without focus management → keyboard users trapped or lost

**What goes wrong:**
RFQ "Teklif Al" opens a dialog → focus stays on body or jumps somewhere random → keyboard user can't tab into form, or tabs out into background while dialog is open. Closes dialog → focus lost (jumps to top of page). Carousel/slider on home is `<div>` based with no arrow-key support. Disclosure (mobile menu) toggles `aria-expanded={false}` always. **Fails WCAG 2.1.1, 2.1.2, 2.4.3, 4.1.2** — required for kamu (public-sector) procurement which is a stated audience.

**Why it happens:**
Custom components built from `<div>` without using primitives that handle a11y. shadcn/ui (PROJECT.md mentions it) is good *if* used; bypassed when juniors build from scratch.

**How to avoid:**
- **shadcn/ui dialog / sheet / dropdown / popover** (built on Radix) handle focus trap, ESC close, return focus, ARIA correctly. **Use them, don't reimplement.**
- **Carousels:** use Embla Carousel (a11y-first) or shadcn's wrapper; ensure arrow-key + swipe + dots; respect reduced motion.
- **Menus:** Radix `<NavigationMenu>` or `<Menubar>` correctly handles `aria-expanded`, `aria-controls`, focus.
- **Skip-link** at top of root layout (`<a href="#main">İçeriğe atla</a>` visible on focus).
- **Focus-visible ring** styled clearly in dark theme — not removed.
- **Run axe DevTools on every page** before considering it done.
- **Test by tabbing through every page with no mouse**.

**Warning signs:**
- Tab key doesn't reach a button.
- Dialog opens; tab reaches background content.
- `:focus` outline missing or invisible on dark theme.

**Phase to address:** Phase 2 (design system primitives) + verify in every interactive-feature phase.

---

### Pitfall 21: React Compiler false negatives on memoization + manual `useMemo` conflicts

**What goes wrong:**
React Compiler is enabled (`reactCompiler: true` in next.config — already in PROJECT.md). Existing code has manual `useMemo`/`useCallback`/`React.memo`. Compiler sometimes can't analyze a function (closure over a ref, mutating an object, complex destructuring) → silently skips memoization. Or: dev sees Compiler is "doing it" → removes manual memos → in the un-analyzable case, perf regresses unnoticed.

**Why it happens:**
React Compiler 1.0 is stable in Next 16, but its heuristics are conservative. Babel-based, so it lengthens build time. It also will refuse to compile components that violate Rules of React (mutation, conditional hooks, etc.) — silently leaving them un-optimized.

**How to avoid:**
- **Run with the React Compiler ESLint plugin** (`eslint-plugin-react-compiler`) — flags un-compilable patterns at lint time.
- **Don't strip manual memos en masse.** Keep them where measured benefit exists; let Compiler handle the rest.
- **Profile critical paths** (hero scroll, project list, map) before/after enabling Compiler.
- **Watch for Rules-of-React violations** Compiler flags: in-render mutation, conditional hooks, accessing refs in render — these now matter for perf, not just correctness.
- **Build time hit:** Document expected longer builds; consider running Compiler only in production builds initially if dev iteration suffers.

**Warning signs:**
- ESLint output: "Component skipped by React Compiler".
- Performance profile shows re-renders that "should" be memoized.

**Phase to address:** Phase 1 (config check) + Phase 2 (lint setup).

---

### Pitfall 22: `useTransition` async pitfalls — stale data, post-await transition loss

**What goes wrong:**
RFQ form submit uses `startTransition(async () => { await fetch(...); setOptimistic(...) })`. User types fast, fires multiple submits (or filter changes on project list trigger fetches). Slow earlier request resolves last → UI shows stale data. Or: state updates *after* the `await` aren't in transition context anymore → blocking UI.

**Why it happens:**
React 19 allows async functions in transitions, but post-`await` updates lose transition context unless explicitly re-wrapped. Concurrency unsafe by default.

**How to avoid:**
- **Track a request ID / use `AbortController`** — discard responses from superseded requests.
- **Wrap post-`await` setState calls in `startTransition` again**:
  ```ts
  startTransition(async () => {
    const res = await fetch(...)
    startTransition(() => setData(res))  // re-enter transition
  })
  ```
- **Prefer `useActionState`** for form submissions — handles pending/error/result state correctly.
- **Prefer `useOptimistic`** for filter UIs (project list filtering, blog category filtering).
- **Keep transitions short**; don't await long operations. Show optimistic UI + revalidate after.

**Warning signs:**
- Filter UI flashes wrong results briefly.
- Pending indicator never clears.
- React warning about state update outside transition.

**Phase to address:** Phase 4 (engineering tools — calculator filter, comparison) and Phase 1+2 (form + project filter).

---

### Pitfall 23: Server-Component-only assumption breaks when adding client interactivity later

**What goes wrong:**
Page is built as RSC fetching mock data → later add a "favorite" toggle / filter / theme-aware element → must convert to client component → loses streaming, prefetching benefits. Or: developer adds `'use client'` to top-level page to avoid the friction → entire subtree client-rendered, bundle balloons, dictionary leaks to client.

**Why it happens:**
Confusion about RSC composition rules. The "fix" of adding `'use client'` at the wrong level is silent and easy.

**How to avoid:**
- **Default = Server Component.** Only add `'use client'` to the smallest leaf that needs interactivity (a button, a form input, a single carousel).
- **Server components can render Client components**; client components cannot render server components — but they can receive them as `children`. Use the `<ServerWrapper>{<ClientLeaf>{<ServerNested/>}</ClientLeaf>}</ServerWrapper>` pattern.
- **Never import dictionaries into a client component.** Pass strings as props from the server parent.
- **Audit: any page where `'use client'` is at the page root is a smell.**

**Warning signs:**
- Bundle analyzer shows page-level component in client chunks.
- Adding any feature requires a `'use client'` cascade.

**Phase to address:** Phase 1 (establish convention in design-system docs) + every phase verification.

---

### Pitfall 24: SEO content depth too thin for B2B (industrial buyers Google long-tail)

**What goes wrong:**
Each project page has 200 words. Each service page has hero + 3 bullets. Buyer searches "İstanbul havalimanı çelik konstrüksiyon imalatçısı" → competitor's 1500-word case study with tonnage, schedule, certifications outranks Erkin's 200-word page. B2B SEO loses on thin content even with perfect technical setup.

**Why it happens:**
Designer-led project optimizes for visual impact; copy is often last and treated as filler ("lorem ipsum until launch").

**How to avoid:**
- **Project detail template designed for ≥ 800 words minimum**: project narrative, scope, materials (HEB/HEA used, tonnage), challenges, timeline, certifications applied, photo captions, related-projects.
- **Service pages ≥ 1000 words**: process, capability, case studies, FAQ, downloadable spec PDF, comparable certifications.
- **Blog focused on long-tail B2B queries**: "X profili kullanılan yerler", "havalimanı hangarı çelik konstrüksiyon süreci", profile selection guides.
- **FAQ sections** with proper `FAQPage` schema.org markup.
- **Internal linking**: every project links to related services, used profiles in technical-info pages.
- **Don't ship with placeholder copy** — copy is a Phase-3 deliverable, not an afterthought.

**Warning signs:**
- Project pages < 500 words.
- No `FAQPage` schema anywhere.
- Internal linking sparse.
- Search Console shows "Discovered – not indexed" for many URLs.

**Phase to address:** Phase 3 (content & marketing) explicitly; design template in Phase 2 to *accommodate* depth.

---

### Pitfall 25: Open Graph for B2B — generic OG image every page

**What goes wrong:**
Every share on LinkedIn (B2B's primary social) shows the same generic logo OG image. RFQ links shared in email previews look like spam. Recruiters share career page → looks like every other page → no differentiation.

**Why it happens:**
B2B audiences "don't share on social, so OG doesn't matter" is a myth — LinkedIn, WhatsApp business chats, Slack, email previews all use OG. Skipped because audience research undervalues it.

**How to avoid:**
- **Per-page OG images** via `app/[lang]/projects/[slug]/opengraph-image.tsx` (Next.js `ImageResponse` API).
- **Project OG**: project hero image + tonnage + location overlay.
- **Service OG**: service icon + process diagram.
- **Static homepage OG**: hero + tagline.
- **Twitter Card** = `summary_large_image`.
- **OG image dimensions**: 1200×630 (LinkedIn, FB, Twitter).
- **Test in LinkedIn Post Inspector + opengraph.xyz before launch.**
- Note Next 16 breaking change: `opengraph-image` `params` is now `Promise<...>`; await it.

**Warning signs:**
- LinkedIn preview shows same image for all pages.
- WhatsApp link preview shows nothing or wrong title.

**Phase to address:** Phase 2 (set up OG infrastructure) + Phase 3 (per-content OG).

---

### Pitfall 26: Customer portal mock leaks "fake auth" patterns into "real auth" assumptions

**What goes wrong:**
Phase 8 mock portal stores `localStorage.setItem('user', { role: 'customer' })` to simulate login → developer wires components to read directly from localStorage → when real Clerk/Auth0 lands, every component must change. Worse: mock has client-side-only "role check" that real app inherits → privilege escalation when backend ships and the localStorage guard is the only one (security disaster).

**Why it happens:**
Mock auth feels clearly temporary, so people skip the architecture work. But components are durable.

**How to avoid:**
- **Define an `AuthProvider` interface** matching Clerk's API shape (or whatever's likely): `useUser()`, `useAuth()`, `<SignedIn>`, `<SignedOut>`, server-side `auth()`.
- **Mock implements the interface**; consumers use the abstraction.
- **Server-side check stub** (returns `null` / fake user) — so when real Clerk lands, server-side guard already exists.
- **No `localStorage.user` reads in components.** Always go through `useUser()`.
- **Document explicitly:** mock portal is UI-shape-validation only; real auth requires Phase 8+ milestone (not in this milestone).
- **Security warning banner in dev** when running with mock auth.

**Warning signs:**
- Components import from `localStorage` directly.
- Role checks in components vs. one place.
- Mock pattern persists into production builds (no env guard).

**Phase to address:** Phase 8 (portal mock).

---

### Pitfall 27: AI assistant mock simulates streaming poorly → real streaming will look worse

**What goes wrong:**
Mock UI for AI chat (Phase 6) uses `setTimeout` chunks, no `Suspense`, no abort, no error states. When real LLM lands, every UX detail (cancel mid-stream, typing indicator, tool-call rendering, error recovery) needs new design.

**Why it happens:**
Mock is faked at the most superficial layer. Designer shows the happy path only.

**How to avoid:**
- **Use `streamText` shape from Vercel AI SDK as the contract**, even when mocking. Mock = `ReadableStream` over a fake message split into chunks.
- **Implement abort, error, and partial-tool-call states** in the mock.
- **Implement reduced-motion-friendly typing indicator** (don't only rely on cursor animation).
- **Server-side stub route** (`app/api/chat/route.ts`) returns mock `ReadableStream` — same shape as real LLM call. When real backend lands, swap the body of one route.

**Warning signs:**
- Mock has only happy path.
- No `AbortController` plumbing.
- No error UI.

**Phase to address:** Phase 6 (AI assistant mock).

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Skip `generateStaticParams` on dynamic routes | Faster scaffolding | All pages dynamic, build slow, Lighthouse worse, hosting cost rises | **Never for this project** (mock data is fully known at build time) |
| Hard-code TR strings instead of dictionary | Quick first pass | Every component must change when EN content arrives | First commit only, refactor in same phase |
| Inline JSON-LD per page instead of helper | Less code | Inconsistent schema across pages, drift | **Never** — ship a `lib/seo.ts` from day one |
| Use shadcn defaults without dark-theme tuning | Looks fine in light | Premium dark theme inconsistent, contrast fails | Phase 1 only; Phase 2 must tune palette |
| Skip pre-commit image optimization | Faster commit | LCP regressions ship to prod | **Never** — wire `sharp` script in Phase 1 |
| `'use client'` at page root to silence errors | Unblocks dev | Bundle bloat, dictionary leaks, RSC benefits gone | **Never** — find the actual leaf |
| Mock data as plain JSON imports | Quick | No type safety, no async swap path | **Never** — go through `lib/data/*.ts` always |
| reCAPTCHA v3 for forms | Familiar, free-ish | KVKK risk (sends to US), poor a11y, Google data | Replace with Cloudflare Turnstile in Stack |
| `disable @next/eslint-plugin-next no-img-element` to use `<img>` | Quick | Loses optimization, layout shift, no AVIF | **Never** — use `<Image>` always |
| Skip `default.tsx` for parallel slot to "fix later" | Build passes once | Build will fail in Next 16 — must fix immediately | **Never** in Next 16 |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| `next-themes` | Render theme-dependent UI without `mounted` guard → hydration mismatch | `useEffect(() => setMounted(true), [])`; render skeleton until mounted |
| `next-themes` + Tailwind v4 | Forget `@custom-variant dark (&:where(.dark, .dark *))` in CSS | Define custom variant; pair with `attribute="class"` |
| Mapbox GL | Token in `NEXT_PUBLIC_*` without URL restriction | Restrict token to `*.erkincelik.com` in Mapbox dashboard; or use MapLibre + free tiles |
| Framer Motion / `motion` | Importing in Server Component | All motion components in `'use client'` files; lazy-load decorative ones |
| Three.js / R3F | SSR'ing the canvas | `dynamic(..., { ssr: false })`; gate render on viewport intersection |
| shadcn/ui `Dialog` | Adding `position: fixed` without overlay → focus escapes | Use shipped `<DialogOverlay>`; don't override `pointer-events` |
| Embla Carousel | No `aria-roledescription="carousel"` on root | Wrap with proper a11y roles or use shadcn carousel that handles it |
| Google Fonts via `next/font/google` | Using both Google Fonts + custom `@font-face` in CSS → double load | Pick one path; prefer `next/font/local` for `.woff2` shipped in repo |
| Next `<Script>` for analytics | `strategy="beforeInteractive"` + cookie consent gate broken | `strategy="lazyOnload"` and only render `<Script>` *after* consent state is true |
| reCAPTCHA | Adding it without consent gate | Replace with Turnstile; or gate behind consent + KVKK metni |
| Vercel BotID | Assuming free | Pro+ only; budget item or pick alternative |
| Self-host image optimization | Forgetting `sharp` install or thinking Next 16 still needs it manually | Next 16 has `sharp` baked-in; verify CPU + memory budget on host |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| 4MB hero JPEG | LCP > 4s, Lighthouse 50s | `sharp` pre-process to AVIF, source ≤ 200KB, `priority` + `sizes` | First mobile user on 4G |
| All pages dynamic via root-layout cookies/headers | `next build` shows ƒ on every route, slow build, no static benefit | Locale detection in `proxy.ts`, `[lang]` in URL only | Always — visible in build log |
| Three.js in main bundle | Initial JS > 300KB, Lighthouse < 90 | `dynamic({ ssr: false })`, viewport-gated, DRACO | Phase 5 ships |
| Embla Carousel auto-play during scroll | Jank, `prefers-reduced-motion` violation | Pause on scroll, respect reduced-motion, `IntersectionObserver` to pause when off-screen | Mid-range Android |
| Non-clustered map pins (50+) | Frame drops on pan/zoom | `supercluster` + render only visible | Scrolling Türkiye-wide map |
| `next/font/google` with many weights | TTFB regress, CLS during font swap | Limit to 2-3 weights; `display: 'swap'`; preload only display fonts | Always with bad config |
| Dictionary in client bundle | Bundle analyzer shows JSON in client chunk; bundle 2× | `import 'server-only'` in dictionary loader; pass strings as props | Adding any dictionary-using client component |
| Animations on layout-changing properties | CLS > 0.1, jank | Animate only `transform`+`opacity`; reserve space with `aspect-ratio` | Always on slow CPUs |
| Synchronous image decode for many thumbs | Long task spikes on grid pages | `loading="lazy"` (default for non-priority `<Image>`), `decoding="async"` | 50+ project grid |
| Re-rendering map provider on theme toggle | Whole map re-instantiates | Memoize map config; theme changes via CSS variables, not re-render | Light/dark toggle while map visible |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Mapbox token in client without URL restriction | Token theft, billing fraud | Restrict to domain in Mapbox dashboard; rotate on suspicion |
| JSON-LD with un-sanitized strings (`<` in user content) | XSS via structured data | `JSON.stringify(jsonLd).replace(/</g, '\\u003c')` (per Next docs) |
| Form submission without origin check (when backend lands) | CSRF | Same-origin cookie + CSRF token; Server Action with `'use server'` does this automatically |
| `dangerouslySetInnerHTML` for blog content from "future CMS" | XSS via stored content | Sanitize via `isomorphic-dompurify` or use MDX/portable-text — never raw HTML |
| `next.config` `images.remotePatterns: [{ hostname: '**' }]` | SSRF / image-proxy abuse | Whitelist exact hostnames |
| `dangerouslyAllowLocalIP: true` in image config | Internal-network image probe via image proxy | Default false; only enable in private dev |
| `images.unoptimized: true` to silence errors | Unbounded image fetch from anywhere | Don't disable; configure `remotePatterns` properly |
| Mock auth `localStorage` user-role read in production build | Privilege escalation when backend lands without proper guard | Server-side check is mandatory; mock = UI scaffold only |
| Client-side reCAPTCHA without server verification | Bot-bypass; PII leaked to Google | Move to Turnstile + server verification when backend lands |
| Loading analytics before consent | KVKK / GDPR fine | Gate `<Script strategy="lazyOnload">` on consent state |
| Email address as raw `mailto:` text | Spam crawlers harvest | Either use form-based contact or obfuscate (component renders email at runtime if consent given) |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Cookie banner blocks first paint with no escape | User leaves before reading | Banner overlays content, doesn't block; "Reject" + "Accept" + "Settings" equal weight |
| Theme toggle requires page refresh to take effect | Feels broken | `next-themes` with `disableTransitionOnChange` for instant feel; CSS variables not class-based duplications |
| Map pins all open same generic popup | Feels under-built | Each pin = real project preview with image + tonnage + link |
| Carousel that auto-plays without pause control | A11y violation, annoying | Visible pause; pause on hover/focus; respect `prefers-reduced-motion` (no autoplay) |
| Mobile menu without hamburger animation feedback | "Did I tap?" | Hamburger → X morph; haptic feel via subtle animation |
| Form with inline errors only on submit | Frustrating retry loop | Validate on blur for individual fields; full-form on submit |
| Dark theme with low-contrast disabled-button state | Looks like a bug | Disabled = clearly different (lower opacity + cursor) but still ≥ 3:1 contrast for non-text UI |
| RFQ form with too many fields up front | Drop-off | Multi-step or progressive disclosure; only ask what's essential first |
| Blog without estimated read time / categories | Bounce | Read-time + category + related posts |
| 3D viewer always-on, no opt-in | Mobile users on data plan annoyed | Poster image + "View in 3D" CTA |
| Project detail without "Get a Quote for Similar" CTA | Lost conversion | Every project page → contextual RFQ CTA pre-filled with project category |
| Loading spinner instead of skeleton | Feels slower | Skeleton matching layout; reserves space; no CLS |
| Toast errors that auto-dismiss too fast | User misses message | Errors persist until dismissed; only success/info auto-dismiss |
| Search/filter with no "no results" empathy state | Dead-end | Suggest alternatives, "View all", contact CTA |
| KVKK / privacy text in tiny grey | Looks like dark pattern | Same readable contrast as body; don't visually de-prioritize legal text |

---

## "Looks Done But Isn't" Checklist

Things that appear complete but are missing critical pieces.

- [ ] **i18n routing:** Often missing **`alternates.languages` in sitemap + `<html lang>` from params + canonical URLs per locale** — verify by inspecting `/sitemap.xml` and view-source `<html>` element on each locale.
- [ ] **Dark mode toggle:** Often missing **`mounted` guard, `suppressHydrationWarning` on `<html>`, no-FOUC script injection** — verify by hard-refresh on slow 3G; should never flash light.
- [ ] **Cookie banner:** Often missing **equal-weight Reject button, consent log, gating of analytics scripts** — verify network panel: no GA/Mapbox/etc. before clicking Accept.
- [ ] **Schema.org JSON-LD:** Often missing **on root layout (Organization), per-project (CreativeWork), per-blog (BlogPosting), test passes** — verify in Rich Results Test.
- [ ] **Forms:** Often missing **honeypot field, time-trap, KVKK aydınlatma metni above submit, accessible error states** — verify by submitting empty form + filling honeypot.
- [ ] **Project detail page:** Often missing **OG image per project, breadcrumb schema, related-projects, RFQ CTA, image alt text** — verify per template.
- [ ] **Modals/dialogs:** Often missing **focus trap, ESC close, return focus on close, ARIA labels** — verify with keyboard-only tab through.
- [ ] **Carousels:** Often missing **arrow-key navigation, swipe support, dots/numbers, pause-on-hover, reduced-motion fallback** — verify each.
- [ ] **3D viewer:** Often missing **WebGL detection fallback, loading state, mobile pixel-ratio cap, abort on unmount** — verify on Moto-G-class device.
- [ ] **Map:** Often missing **clustering, mobile touch-friendly zoom controls, keyboard accessibility, theme-aware tiles** — verify on touch + keyboard.
- [ ] **Image pipeline:** Often missing **`sizes` prop, `priority` on hero only, AVIF generation, blur placeholder** — verify Lighthouse "Properly sized images" passes.
- [ ] **Mock data layer:** Often missing **TypeScript types, async signatures, locale-aware shape, Zod validation** — verify by `tsc --noEmit` and trying to import non-async pattern.
- [ ] **Customer portal mock:** Often missing **AuthProvider abstraction, server-side auth stub, security warning in dev** — verify by checking no component imports `localStorage` directly.
- [ ] **AI chat mock:** Often missing **abort controller, error state, reduced-motion typing indicator, ReadableStream contract** — verify via browser DevTools network panel.
- [ ] **Lighthouse pass:** Often **only desktop checked; mobile fails** — verify mobile + desktop both ≥ 95 (PROJECT.md requirement).
- [ ] **Accessibility:** Often **only axe automated check; manual keyboard test skipped** — verify full-page tab through every template.
- [ ] **Build output:** Often **missing static prerender (all routes ƒ instead of ○)** — verify in `next build` output table.
- [ ] **Browser support:** Often **only Chrome tested; Safari iOS 16+ minimum unverified** — verify per modern browsers floor (PROJECT.md constraint).
- [ ] **Open Graph:** Often **same image for every page** — verify by sharing on LinkedIn for 3 different page types.
- [ ] **Project image rights:** Often **scraped without permission documentation** — verify asset register before launch.

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Sync `params`/`cookies` shipped (Pitfall 1) | LOW | Run `npx @next/codemod@canary upgrade latest` then fix lints; build will surface remaining sites |
| All-routes-dynamic from layout (Pitfall 2) | MEDIUM | Move locale detection to `proxy.ts`, refactor any shared layout to not call `headers()`/`cookies()`/`searchParams` |
| Tailwind v4 dark variant broken (Pitfall 5) | LOW | Add `@custom-variant dark (&:where(.dark, .dark *))` to globals.css; test toggle |
| FOUC on theme load (Pitfall 6) | LOW–MED | Move `<ThemeProvider>` boundary inside `<body>`; add `suppressHydrationWarning`; ensure no theme-deciding logic in layout |
| Missing `default.tsx` (Pitfall 4) | LOW | Add `default.tsx` returning `null` to each parallel slot |
| Sitemap without hreflang (Pitfall 9) | LOW | Add `alternates.languages` to each entry; redeploy; resubmit in Search Console |
| Wrong Schema.org type (Pitfall 10) | LOW–MED | Refactor `lib/seo.ts` helper; touch every page using it; re-test in Rich Results |
| Animation jank/CLS (Pitfall 11) | MEDIUM | Profile in DevTools; replace `top`/`left`/`height` animations with `transform`; add `aspect-ratio` to slots; add `MotionConfig reducedMotion="user"` |
| 3D viewer in main bundle (Pitfall 12) | MEDIUM | Replace direct import with `dynamic(..., { ssr: false })`; gate via viewport |
| Mapbox token leak (Pitfall 13) | LOW | Add URL restriction in Mapbox dashboard; rotate token; update env |
| Image rights unclear (Pitfall 14) | HIGH | Halt launch; obtain written permission for each scraped image, OR replace with new photography, OR remove image |
| Cookie banner non-compliant (Pitfall 15) | LOW–MED | Rebuild banner with equal Accept/Reject; gate scripts; document consent log shape |
| Form spammed (Pitfall 16) | LOW | Add honeypot + time-trap; if launched without, push hotfix |
| Low contrast (Pitfall 17) | MEDIUM | Re-tune `@theme` tokens with contrast checker; re-audit pages |
| Mock-shape lock-in (Pitfall 18) | HIGH (when CMS lands) | Type-first refactor: define `lib/types/api.ts`, retrofit consumers — touches most of the app |
| Heavy hero (Pitfall 19) | LOW | Run `sharp` script; replace asset; verify Lighthouse |
| A11y broken interactives (Pitfall 20) | MEDIUM | Replace custom dialogs/menus with shadcn primitives; run axe + keyboard audit |
| Compiler skip (Pitfall 21) | LOW | Install `eslint-plugin-react-compiler`; fix flagged components |
| Stale data from transitions (Pitfall 22) | LOW | Add `AbortController` or request-id check; re-wrap post-`await` set-state |
| `'use client'` cascade (Pitfall 23) | LOW–MED | Identify smallest interactive leaf; lift `'use client'` down; pass server data as props |
| Thin SEO content (Pitfall 24) | HIGH | Content sprint; expand each project + service to long-form |
| Generic OG (Pitfall 25) | LOW | Add `opengraph-image.tsx` per template using `ImageResponse` |
| Mock auth leaks (Pitfall 26) | MEDIUM | Refactor consumers behind `useUser()`; remove `localStorage` reads |

---

## Pitfall-to-Phase Mapping

How roadmap phases should address these pitfalls.

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| 1. Next 16 training-data drift | **Phase 1 (foundation)** + every phase | AGENTS.md doc-read rule enforced; no sync `params` in PR review |
| 2. Dynamic root layout | Phase 1 | `next build` shows ○ for marketing routes |
| 3. Image config defaults | Phase 1 | `next.config.ts` has explicit `qualities`, `imageSizes`, `formats`, `remotePatterns` |
| 4. Parallel-routes `default.tsx` | Phase 8 (portal mock) | Build passes |
| 5. Tailwind v4 dark variant | Phase 1 (globals.css) | Toggle visibly switches on a light-OS machine |
| 6. FOUC + hydration | Phase 1 (providers) | Slow-3G reload shows no flash; no console hydration error |
| 7. Static export feature mismatch | Phase 1 (hosting decision documented) | `STACK.md` declares hosting model; deferred to Phase 9 if not yet decided |
| 8. i18n hydration drift | Phase 1 (i18n scaffold) | URL-based locale; no client `navigator.language` reads; bundle analyzer clean |
| 9. Sitemap hreflang | Phase 2 (SEO) | `/sitemap.xml` has `xhtml:link`; passes hreflang validator |
| 10. Wrong Schema.org type | Phase 2 (SEO foundation) | Rich Results Test passes for each template |
| 11. Animation jank/CLS/a11y | Phase 2 (motion primitives) + Phase 5 (3D) | Lighthouse CLS < 0.05; reduced-motion respected; mobile profile clean |
| 12. 3D viewer bundle | Phase 5 | Bundle analyzer: three not in main chunk; dynamic-only |
| 13. Mapbox token / map perf | Phase 5 | Token has URL restriction; map clusters pins |
| 14. Image rights | Phase 1 (asset acquisition) | Asset register exists; covers every scraped image |
| 15. KVKK cookie banner | Phase 2 (cookie banner is Phase 1+2 requirement) | Network tab clean before consent; consent persists; reject equally easy |
| 16. Form spam | Phase 1+2 (form scaffold) | Honeypot + time-trap present; Turnstile contract documented for backend |
| 17. Low-contrast dark theme | Phase 2 (design system) | Contrast checker passes for every defined token pair |
| 18. Mock-shape lock-in | Phase 1 (data layer) | `lib/types/api.ts` exists; consumers go through `lib/data/*.ts`; localized strings |
| 19. Heavy imagery | Phase 2 (image pipeline) | `pnpm prebuild` runs `sharp`; LCP < 2.5s |
| 20. A11y broken interactives | Phase 2 (primitives) + every interactive feature phase | Keyboard-only walkthrough + axe pass on every page |
| 21. React Compiler skip | Phase 1 (config + lint) | `eslint-plugin-react-compiler` installed; no skip warnings on critical paths |
| 22. `useTransition` async traps | Phase 4 (filters/forms) | AbortController or request-id; `useActionState`/`useOptimistic` for forms |
| 23. `'use client'` cascade | Phase 1 (convention doc) + every phase | No `'use client'` at page root in PR review |
| 24. Thin SEO content | Phase 3 (content & marketing) | Project pages ≥ 800 words; service pages ≥ 1000 words; FAQPage schema |
| 25. Generic OG images | Phase 2 (OG infra) + Phase 3 (per-content) | `opengraph-image.tsx` per template; LinkedIn preview different per page |
| 26. Mock auth leak | Phase 8 (portal) | No `localStorage` reads in components; AuthProvider interface exists |
| 27. Mock AI streaming poor | Phase 6 (AI mock) | Mock returns `ReadableStream`; abort + error states present |

---

## Sources

### Authoritative (HIGH confidence — local Next.js 16.2.4 docs verified)

- `node_modules/next/dist/docs/01-app/02-guides/upgrading/version-16.md` — All Next 15→16 breaking changes (cookies/headers/params async, sitemap async id, opengraph-image async, middleware→proxy, `revalidateTag` 2-arg, removed AMP/`next lint`/runtime config, image config defaults)
- `node_modules/next/dist/docs/01-app/02-guides/internationalization.md` — Dictionary patterns, `import 'server-only'`, `[lang]` segment, `generateStaticParams` for locale
- `node_modules/next/dist/docs/01-app/02-guides/static-exports.md` — Unsupported features list when `output: 'export'`
- `node_modules/next/dist/docs/01-app/02-guides/json-ld.md` — XSS sanitization (`<` → `<`), `<script type="application/ld+json">` pattern
- `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/01-metadata/sitemap.md` — `alternates.languages`, async `id` from `generateSitemaps`
- `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/proxy.md` — Renamed from middleware; nodejs runtime; matcher
- `node_modules/next/dist/docs/01-app/03-api-reference/03-file-conventions/dynamic-routes.md` — Async params, `PageProps`/`LayoutProps`/`RouteContext` typegen helpers
- `node_modules/next/dist/docs/01-app/02-guides/forms.md` — Server Actions for forms (forward-compatible)
- `node_modules/next/dist/docs/01-app/02-guides/production-checklist.md` — A11y, perf, security, metadata checklists
- `node_modules/next/dist/docs/01-app/01-getting-started/11-css.md` — Tailwind v4 PostCSS-only setup
- `node_modules/next/dist/docs/01-app/01-getting-started/12-images.md` — `<Image>` API, blur placeholder, sizes

### Web (MEDIUM confidence — multiple sources agree)

- [Tailwind CSS v4 Upgrade Guide](https://tailwindcss.com/docs/upgrade-guide) — `@custom-variant`, no `tailwind.config.js`, `prefers-color-scheme` default
- [Tailwind CSS Dark Mode Docs](https://tailwindcss.com/docs/dark-mode) — `@custom-variant dark` for class-based mode
- [tailwindlabs/tailwindcss Discussion #16517 — Upgrading: Missing Defaults, Broken Dark Mode](https://github.com/tailwindlabs/tailwindcss/discussions/16517) — Real-world v3→v4 dark mode breaks
- [pacocoursey/next-themes README](https://github.com/pacocoursey/next-themes) — `attribute="class"`, `suppressHydrationWarning`, mounted guard pattern
- [Fixing Hydration Mismatch in Next.js (next-themes)](https://medium.com/@pavan1419/fixing-hydration-mismatch-in-next-js-next-themes-issue-8017c43dfef9)
- [Fixing Dark Mode Flickering (FOUC)](https://notanumber.in/blog/fixing-react-dark-mode-flickering)
- [shadcn/ui Next.js Dark Mode Docs](https://ui.shadcn.com/docs/dark-mode/next) — Recommended provider boundary
- [React 19 Release Notes](https://react.dev/blog/2024/12/05/react-19) — Actions, `useActionState`, `useOptimistic`, ref-as-prop, async transitions
- [useTransition Reference](https://react.dev/reference/react/useTransition) — Post-await state-update gotcha
- [React 19 Concurrency Deep Dive](https://dev.to/a1guy/react-19-concurrency-deep-dive-mastering-usetransition-and-starttransition-for-smoother-uis-51eo) — Stale-data race
- [Framer Motion Performance Patterns and Pitfalls](https://dev.to/whoffagents/framer-motion-animations-that-dont-kill-performance-patterns-and-pitfalls-5cki) — `MotionConfig reducedMotion`, layoutScroll
- [React Three Fiber Bundle Reduction](https://github.com/pmndrs/react-three-fiber/discussions/812) — Three.js bundle ~155KB
- [Mapbox Pricing 2026](https://www.mapbox.com/pricing) — 50K free web map loads
- [Mapbox vs MapLibre Comparison](https://www.pkgpulse.com/blog/mapbox-vs-leaflet-vs-maplibre-interactive-maps-2026)
- [MapLibre Free Alternative](https://www.geoapify.com/mapbox-gl-new-license-and-6-free-alternatives/)
- [KVKK Practical Compliance Guide (Cookie-Script)](https://cookie-script.com/guides/practical-guide-to-kvkk-compliance) — Equal-weight reject; consent log
- [Turkey KVKK Cookie Guidelines (CookieYes)](https://www.cookieyes.com/blog/turkey-data-protection-law-kvkk/) — 2025 amendments aligning with GDPR
- [Turkish DPA Draft Cookie Guidelines](https://secureprivacy.ai/blog/turk-data-protection-authority-published-draft-cookie-guidelines)
- [Vercel BotID](https://vercel.com/docs/botid) — Pro+ feature; CAPTCHA-free
- [Honeypot Spam Protection](https://www.craftengineer.com/honeypot-spam-protection-a-simpler-alternative-to-captcha/)
- [Web Scraping & Copyright Law](https://www.neudata.co/blog/web-scraping-and-copyright-law) — Republishing risk; statutory damages
- [Data Scraping IP Rights and Risks (Eversheds Sutherland)](https://www.eversheds-sutherland.com/en/global/insights/data-scraping-intellectual-property-rights-and-risks)
- [Self-host Next.js Image Optimization](https://www.flightcontrol.dev/blog/secret-knowledge-to-self-host-nextjs)
- [Scroll Animation Tools 2026](https://cssauthor.com/scroll-animation-tools/) — CLS budget; `prefers-reduced-motion` accommodation

---

*Pitfalls research for: Premium B2B marketing site (steel construction) on Next.js 16 + React 19 + Tailwind v4*
*Researched: 2026-04-25*
