# Feature Research

**Domain:** B2B steel construction / industrial fabrication marketing website (Erkin Çelik — Tuzla/Gebze, TR)
**Researched:** 2026-04-25
**Confidence:** HIGH (reference sites directly inspected; B2B/industrial/manufacturing best-practice literature for 2026 corroborated across multiple independent sources)

---

## Reference Set

| Site | Role / Take-aways |
|------|------------------|
| severfield.com | Sector-organized project portfolio, case study + "shaping skylines" hero, sectoral nav (Stadia, Transport, Commercial, Industrial, Nuclear), investor section, careers pathway |
| voortman.net | Solution categories, customer testimonial gallery with metrics ("100% production boost"), online demo request, 7-language switcher, knowledge library, login portal |
| thyssenkrupp.com/en | Sustainability-anchored hero ("Technologies for green industrial transformation"), 11 industry sectors, investor + newsroom, story-driven content, multi-domain career portal |
| arcelormittal.com | Enterprise-scale corporate (403'd to fetch — known via prior literature: investor + sustainability + sectors + global locator) |
| severfield.eu / severfield.com | Certifications first-class (NEN EN 1090-1 EXC4, ISO 9001/3834-2/14001), capacity tonnage prominently displayed |
| kingspan.com | Country selector landing — mandatory pattern for global reach |
| schueco.com | Brand portfolio, sustainability awards, multi-region site (40+ Europe), digital project-phase tooling |
| Current erkincelik.com | WP-based static; baseline gap analysis: no project detail, no filter, no i18n, no dark, no map, no calculator, no schema, no detail case study |

---

## Feature Landscape

### Table Stakes (Users Expect These)

Missing any of these in 2026 = the B2B buyer concludes "this firm is small / behind / unprofessional" and bounces. For a steel fabricator selling to airport / bridge / hangar owners, trust is the entire purchase decision — these are non-negotiable.

#### A. Navigation & Information Architecture

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Sticky top header with logo + primary nav + language + RFQ CTA | Universal B2B convention; "Request Quote" top-right is the single highest-leverage conversion element | LOW | Severfield, Voortman, Thyssenkrupp all do this |
| Mega-menu (or grouped dropdown) for Services + Projects | Steel co's have 4–8 service sub-pages and many sectors; flat nav buries content | MEDIUM | Use shadcn `NavigationMenu`; keyboard accessible; mobile drawer alternate |
| Breadcrumbs on all interior pages | B2B visitors arrive deep from Google; need to orient | LOW | JSON-LD `BreadcrumbList` schema |
| Comprehensive footer (sitemap, addresses, certs, legal, social) | Last-line trust check; legal addresses + KVKK + sicil no expected for TR companies | LOW | 4-column with all utility links |
| 404 + error boundary pages | Branded error pages signal care; default Next.js 404 reads as "broken" | LOW | App Router `not-found.tsx` + `error.tsx` |
| Search across site (projects + blog + tech tables) | Buyers search for specific sector/profile; absence forces them to use Google with `site:` | MEDIUM | Phase 2+ — local index over mock data acceptable v1 |

#### B. Hero & Homepage

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Above-fold hero with headline + sub + 1–2 CTAs | First 5 seconds determine credibility; must communicate "what we do, who we are, why us" | LOW | Severfield: "Shaping skylines"; Thyssenkrupp: "Technologies for green industrial transformation". Use single hero image/video, NOT auto-rotating slider |
| Static hero image OR muted hero video (controls + poster) | Cinematic industrial imagery is the table stake; video must NOT autoplay with sound | LOW–MED | Hero video: `<video muted loop playsInline poster>`; honor `prefers-reduced-motion` |
| Capacity / scale number block ("8.400 ton/yıl", "16+ years", "ISO certified") | B2B buyers scan for proof-of-scale within 10s; numbers convert | LOW | 3–4 metrics, animated count-up on scroll-into-view (respect reduce-motion) |
| Featured projects strip (3–6 cards) | Severfield, Voortman both lead with featured projects; visual proof outperforms claims | LOW | Image + name + sector + tonnage; click → project detail |
| Services overview (3–4 cards: Projelendirme / İmalat / Montaj) | Forces clarity on "what they actually do"; each → service detail | LOW | Icon + 1-line + link |
| Trust strip: certification logos + 8–12 client logos | "Eight to twelve well-chosen logos make a stronger impression than a wall of 50" — Webstacks 2026 | LOW | Grayscale, hover color; alt text required |
| Process / "how we work" section | B2B differentiator vs commodity competitors; airport-grade buyers want to see methodology | MEDIUM | 4–5 step visual: Engineering → Production → QA → Logistics → Erection |
| Final-CTA band before footer | Last conversion opportunity; "Tell us about your project" form trigger | LOW | Full-bleed dark band, single CTA |

#### C. Project Portfolio

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Project listing grid with cover image + meta | Severfield, Voortman, all premium B2B steel sites do this; static-only WP page is the #1 reason erkincelik.com feels dated | LOW | Responsive grid, lazy images, AVIF/WebP |
| **Project detail page** (own URL per project) | Required for SEO + share + case-study credibility; named in PROJECT.md as gap vs current site | MEDIUM | Hero, metadata block (tonnage / location / year / sector / client), gallery, narrative, related projects |
| Filter by sector / year / location / tonnage | Buyers self-qualify by sector ("show me airports"); a 50+ project grid without filter is unusable | MEDIUM | URL-state filters (`?sector=airport&year=2023`) for shareability |
| Tabs: Tamamlanan / Devam Eden | Active site discrimination; signals "still in business + active pipeline" | LOW | Single segmented control |
| Project metadata block (tonnage, location, year, client, status) | Engineers need spec data; named in Active Requirements | LOW | Definition list with Schema.org `Project` markup |
| Image gallery / lightbox (3–10 photos per project) | Steel erection photos are the proof; keyboard + swipe navigation expected | MEDIUM | shadcn `Dialog` + Embla; preload neighbor images |
| Map pin per project (coords) | Geographic coverage = sales credibility; powers Phase 5 Türkiye/dünya map | LOW (data) | Required for Phase 5 dependency |
| "Related projects" rail at end of detail | Standard B2B discovery pattern; keeps visitor in-funnel | LOW | Same sector or same location |
| Search within projects | If 50+ projects, list-only browse is friction | LOW | Client-side filter on mock JSON |

#### D. Services Pages

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Hub page + per-service detail (Projelendirme, İmalat, Montaj) | "Çelik konstrüksiyon" alone is too generic; sub-services map to buyer intents | LOW | Hub with 3 cards → detail per service |
| Per-service: scope, process, equipment, certifications, sample projects | Each service page must answer "are you qualified to do my specific job" | MEDIUM | Reuse process diagram component; link to relevant projects |
| Inline RFQ CTA on every service page | "RFQ buttons on every product page" — sevenatoms 2026 best practice | LOW | Floating or sectional |

#### E. Technical Resources

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| HEB / HEA / IPE / IPN / köşebent / kutu profil / boru weight tables | Erkin Çelik's existing site has this; engineers reference them constantly; removing = retention loss + SEO loss | MEDIUM | shadcn `Table` with sticky header; CSV/print export |
| Table search + filter + sort | Static tables are 2010-era; interactive is 2026 standard (named in Active Reqs) | MEDIUM | TanStack Table; URL-state filters |
| Unit converter (kg ↔ lb, mm ↔ in) | Engineers working with mixed-unit clients (Gulf, EU); single line, big payoff | LOW | Pure client utility |
| Downloadable PDFs (catalog, certs, technical spec sheets) | Buyers forward PDFs to procurement; absence forces email request | LOW | Static PDF in `/public`; preview thumbnail |

#### F. Trust Signals

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Certifications page (ISO 9001 / 14001 / OHSAS 18001) with logos + PDFs | Required for tender qualification; B2B buyer's first filter | LOW | Each cert: logo, issuer, validity, downloadable PDF preview |
| Client / reference logos | "Recognizable logos build trust without overwhelming" — Webstacks 2026; 8–12 logos sweet spot | LOW | Grayscale hover-color row |
| Case study (deep-dive on 1–3 flagship projects) | "Case studies are the heavy artillery of B2B trust signals" — Trajectory 2026; named in Phase 3 reqs | MEDIUM | Long-form narrative: Challenge → Solution → Result; metrics-led |
| News / press archive | Severfield + Thyssenkrupp both do; "active company" signal | MEDIUM | Reuse blog engine with category="press" |
| Capacity / scale numbers (8.400 ton, m² fabrika, # ekipman) | Industrial buyers need to know "can you handle my volume" | LOW | Hero + dedicated "kapasite" page |

#### G. Corporate (Kurumsal)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| About / Hakkımızda (history timeline 2008→2026) | "Severfield 25 years, Voortman 80 years" — longevity is a sales asset | LOW | Vertical timeline; key milestones |
| Mission / Vision / Values | Tender boilerplate; B2B convention | LOW | Single page section |
| Certifications page (see Trust Signals) | — | — | — |
| Quality / HSE policy page | Required for some tender qualifications | LOW | Static content |

#### H. Contact

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| 2 office addresses (Tuzla merkez + Gebze fabrika) | Multi-location is a credibility marker; legally required disclosure for TR | LOW | LocalBusiness schema each |
| Embedded Google Maps for each location | Universal expectation; satellite view shows fabrika scale | LOW | Two `<iframe>` or static map images (lighter for Lighthouse) |
| Working hours, phone, email, KVKK contact | Legal + practical | LOW | Plain content |
| **RFQ form** (project type, sector, tonnage range, timeline, file upload, budget range) | Distinct from generic contact form; structured for engineering team | MEDIUM | Long form; multi-step optional; honeypot + reCAPTCHA |
| **General contact form** (name, email, phone, message) | Quick path for non-quote inquiries | LOW | Same page, separate tab |
| KVKK consent checkbox + readable policy link | TR legal requirement | LOW | Required field |
| Spam protection (honeypot + BotID/reCAPTCHA) | B2B forms are bot magnets | LOW | Honeypot first; reCAPTCHA only if needed |
| Form success/error states | Default browser alerts read as broken | LOW | Inline toast + page-level confirmation |

#### I. Blog / Insights

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Blog list + post detail | SEO + thought leadership; current site has it | LOW | MDX or markdown with frontmatter |
| Categories + tags | Filtering by topic; organic SEO benefit | LOW | URL-state |
| Reading time + publish date + author | 2026 expectation; Phase 3 req | LOW | Computed at build |
| Related posts at end | Pageview lift; standard pattern | LOW | Same category |
| RSS feed | Subscriber retention; minor SEO | LOW | App Router `route.ts` |

#### J. Internationalization (TR/EN)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Language switcher (TR ↔ EN) in header | "Worldwide" claim requires it; named in Active Reqs | MEDIUM | Persisted in URL: `/tr/...` ↔ `/en/...` |
| Localized URLs (`/tr/projeler` ↔ `/en/projects`) | Turkish-specific best practice — websoftik 2026; SEO-critical | MEDIUM | Next.js i18n routing or `[lang]` param |
| Localized number/date formats | `Intl.NumberFormat('tr-TR')` for "8.400 ton"; ISO date for EN | LOW | `Intl` API |
| `hreflang` tags | Google reads these; SEO necessity | LOW | Per-page meta |
| EN content placeholders for Phase 1 (translation later) | Accept ship-with-stub; PROJECT.md confirms | LOW | English keys exist, content TBD |

#### K. Theming & Visual

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Dark mode + light mode toggle | Modern industrial premium expectation; named in Active Reqs; dark mode IS the primary aesthetic | LOW | Next.js `next-themes` + CSS vars; dark default |
| Theme persisted across sessions | Toggling resets is a tell of low quality | LOW | localStorage |
| Honor `prefers-color-scheme` initial | Respect OS setting | LOW | `next-themes` does it |

#### L. Accessibility (WCAG 2.2 AA)

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Keyboard navigation (full site usable, focus rings) | Tender qualification + legal; named in Active Reqs | MEDIUM | Test every interactive: nav, dialogs, gallery, forms |
| ARIA labels for icon buttons, dialogs, tabs, menu | Screen reader users; airport/govt buyers care | MEDIUM | shadcn primitives are AA-ready; verify each |
| Color contrast 4.5:1 body / 3:1 large + UI | Premium dark theme often fails contrast — must audit | MEDIUM | Token-level enforced via design system |
| Alt text on all images, especially project photos | Project images are the content; alt = project name + scope | LOW | Mock data layer must include `alt` field |
| `prefers-reduced-motion` honored | Scroll-driven animation must opt out | LOW | CSS `@media` + JS check |
| Skip-to-content link | Standard AA requirement | LOW | Visually hidden until focused |
| Form errors associated with fields (`aria-describedby`) | Screen readers + inline error UX | LOW | shadcn Form does this |

#### M. Performance

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Next.js Image with AVIF/WebP, sized + lazy | Lighthouse 95+ goal in PROJECT.md; project galleries are image-heavy | LOW | `next/image` + remote pattern config |
| Font subsetting (latin + latin-ext for TR) | Turkish chars (ç ş ğ ı ö ü) need latin-ext; missing breaks layout | LOW | `next/font` with `subsets: ['latin','latin-ext']` |
| Route-level code split (default in App Router) | Large interactive pieces (calculator, map) shouldn't bloat homepage | LOW | Dynamic import for heavy widgets |
| Lazy-load below-fold sections | LCP protection | LOW | Intersection Observer or `loading="lazy"` |
| `<link rel="preconnect">` for Maps + analytics | Standard 2026 hygiene | LOW | In `<head>` |
| Compression / caching headers (deploy-side) | Out of MVP scope per PROJECT.md (no hosting decided) but plan for it | LOW | Vercel default if used |

#### N. SEO & Structured Data

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| `<title>` + meta description per page | SEO baseline | LOW | App Router `metadata` API |
| Open Graph + Twitter Card | LinkedIn/WhatsApp share previews | LOW | OG image per page or generated |
| `Organization` schema (HQ) | Knowledge panel | LOW | JSON-LD in root layout |
| `LocalBusiness` schema per office | Google Maps + local pack | LOW | One per location |
| `Project` / `CreativeWork` schema per project | Surface project data in rich results | LOW | Per project detail page |
| `BreadcrumbList` schema | Breadcrumb rich result | LOW | Per page with breadcrumbs |
| `FAQPage` schema where Q&A exists | FAQ rich result | LOW | If FAQ section added |
| `sitemap.xml` + `robots.txt` | SEO baseline; named in Active Reqs | LOW | Next.js metadata route handlers |
| `hreflang` tags for TR/EN | Multilingual SEO | LOW | Per-page meta |
| Canonical URLs | Avoid duplicate-content penalties (TR/EN especially) | LOW | App Router metadata |

#### O. Lead Generation Hygiene

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| RFQ CTA visible on every page (header + section + footer) | "Pipeline, not vanity traffic" — Thomas/sevenatoms 2026 | LOW | Component reuse |
| Concise RFQ form (only essential fields) | "Long forms kill B2B conversion" — RedMoxy 2026 | LOW | 5–7 fields max for first contact; deep qualification later |
| Action-language CTAs ("Teklif Al", "Projemizi Konuş", NOT "Submit") | Generic copy underperforms; sevenatoms 2026 | LOW | Copy pass |
| Mobile-optimized forms (large touch targets, autocomplete) | "70% of B2B research happens on mobile" — sevenatoms 2026 | LOW | shadcn Form + appropriate `inputmode` |
| Trust signals embedded in form (cert logos, "we reply within 24h") | Reduces submission friction | LOW | Inline UI |

#### P. Compliance & Legal

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Cookie consent banner (KVKK-compliant) | TR legal requirement; named in Active Reqs | MEDIUM | Categories: necessary / analytics / marketing; persist choice |
| KVKK aydınlatma metni page | Legal | LOW | Static page |
| Çerez politikası page | Legal | LOW | Static page |
| Form-submission consent checkbox | KVKK | LOW | Required, link to policy |

---

### Differentiators (Competitive Advantage)

These are how Erkin Çelik wins vs the generic "blue-grey corporate" competitor set in TR çelik konstrüksiyon. Each maps to PROJECT.md's premium-industrial positioning.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| **Steel weight calculator** (profile + length → kg, with cost option) | Engineers bookmark useful tools → recurring traffic + brand recall; SEO long-tail magnet ("ipe 200 kg") | MEDIUM | Phase 4; pure client-side; profile dropdowns + dynamic chart; sharable URL params |
| **Profile comparison tool** (compare HEB 200 vs IPE 240) | Helps buyers self-spec; positions Erkin as "advisor not vendor" | MEDIUM | Phase 4; dual-column table |
| **Interactive project map** (TR + global pins, sector filter overlay) | Visual proof of geographic + sector coverage — far stronger than a list; named Phase 5 | MEDIUM–HIGH | Mapbox GL JS or react-leaflet; cluster pins; click → project detail; mobile fallback to list |
| **Cinematic industrial-premium aesthetic** (dark default, metallic accents, large type, scroll-driven micro-interactions) | The single biggest visual differentiator vs jenerik blue-grey TR competitors; named in Active Reqs | HIGH | Design system work; framer-motion for scroll; respect `prefers-reduced-motion` |
| **3D model viewer** (sample project GLTF) | "We engineer 3D-first" signal; novelty + tech-forward perception | HIGH | Phase 5 mock; `<model-viewer>` web component or react-three-fiber; lazy-loaded; one sample suffices for v1 |
| **AI assistant UI mock** ("hangi profili?" → mock streaming answer) | Forward-looking; even as mock signals tech investment; demoable to clients | MEDIUM | Phase 6 mock; UI only, fake streaming with `setInterval`; real LLM next milestone |
| **Customer portal UI mock** (login + dashboard + document list) | Anchors enterprise/repeat-buyer story; even mocked it's a sales asset | MEDIUM | Phase 8 mock; /portal route, fake auth, mock data; signals "we serve repeat clients" |
| **Process step interactive diagram** (hover/click each step → detail) | More memorable than a static diagram; positions methodology as differentiator | MEDIUM | Phase 7; SVG + animated states |
| **Live production media gallery / fabrika tour video** | Industrial transparency = trust; mock placeholder per Phase 7 | MEDIUM | Phase 7; videos with poster + controls (NOT autoplay-with-sound) |
| **Animated hero with subtle parallax** (NOT carousel) | Cinematic feel without carousel anti-pattern; one strong scene | MEDIUM | One static or muted-loop video; respect reduce-motion |
| **Case study deep-dive** (e.g., İstanbul Cargo Energy 820t — challenge / solution / result / metrics) | "Heavy artillery of B2B trust" — Trajectory 2026; named Phase 3 | MEDIUM | Long-form template; quantified outcomes; client quote if available |
| **Capacity-visualization on homepage** (animated count-up of 8.400 ton, 16 yrs, # cert) | Numbers + animation + scroll-triggered = memorable scale signal | LOW | Phase 1 |
| **Sector-organized project landing** ("Havalimanı / Hangar / Köprü / AVM / Otel / Fabrika") | Buyers self-segment; mirrors Severfield's nav strategy; better SEO than flat grid | LOW | Phase 1 — sector pages with filtered project lists |
| **Newsletter signup UI** (Phase 3) | List-building for re-engagement; even mocked it's a sales asset | LOW | UI only Phase 3; backend later |
| **Quick steel-spec quote micro-form** ("Tonaj ne, ne kadar sürede?" → 3-field instant inquiry on every service page) | Removes RFQ-form friction for casual inquirers | LOW | Sectional component, reuses contact backend (later) |
| **Downloadable corporate catalog (one-click PDF)** | Procurement workflow — buyers attach to internal docs | LOW | Phase 3; static PDF |
| **Sticky table of contents on long pages** (case studies, services, blog) | Premium content sites do this; aids scannability | LOW | shadcn-style scroll-spy |
| **Reading-mode typography on blog** (max-width, optimized line-height) | Premium positioning extends to text content | LOW | Tailwind prose plugin |
| **Video poster + controls on hero (no autoplay sound)** | Premium production value without anti-pattern | LOW | Already in table-stakes; here as differentiator if cinematic-quality footage is sourced |

---

### Anti-Features (Commonly Requested, Often Problematic)

These are the features that **look premium but degrade the product**, or that competitor TR sites have but should NOT be ported. Documenting prevents scope-creep regressions.

| Anti-Feature | Why Requested | Why Problematic | Alternative |
|--------------|---------------|-----------------|-------------|
| **Auto-rotating hero slider / carousel** (with 5+ slides) | "We have lots to say"; common in TR corporate sites including current erkincelik.com | "Only ~1% of users click carousels" — CXL/UXMovement; "almost all testing has proven content delivered via carousels to be missed" — uxpatterns.dev; bad for accessibility (screen-reader users skip every slide); kills LCP (multiple hero images load) | Single static or muted-loop hero; or a featured-projects strip (3–6 cards, manual scroll) |
| **Autoplay video with sound** | "It's cinematic" | Universally hated; legal risk on mobile; LCP killer; accessibility violation; many browsers block it anyway | Muted loop with poster + visible controls; or click-to-play button |
| **Splash screen / intro animation** before homepage | "Premium feel" | LCP destruction; mobile users leave; SEO-hostile; users skip intros 100% of the time | Single fast-loading hero; let CSS micro-animations in-place do premium signaling |
| **Heavy 3D / WebGL on every page** | "Modern, innovative" | Lighthouse 95+ goal in PROJECT.md is incompatible; mobile thermal throttling; battery drain; not all GPUs support; accessibility nightmare | Limit 3D to ONE dedicated page (Phase 5); lazy-loaded; respect reduce-motion |
| **Mouse-following cursor effects / custom cursors** | "Awwwards-style" | Confuses users; breaks pointer affordances; hostile to a11y; no value to B2B engineer evaluating capability | Use traditional cursors + good hover states |
| **Scroll-jacking / forced-scroll experiences** (parallax full-page sections that hijack wheel) | "Cinematic storytelling" | B2B buyers scan, don't browse; scroll-jack increases bounce; accessibility-hostile (keyboard users); breaks browser back/forward | Standard scroll + scroll-driven CSS animations that respect reduce-motion |
| **Live chat widget with eager pop-up** (auto-open after 5s) | "Help convert visitors" | Annoying; slows page load (usually third-party); often unstaffed for TR B2B; mobile UX disaster | Phase later if added: opt-in only, manual open via floating button |
| **"Click here" / "Submit" / "Send" CTA copy** | Defaults from form libraries | Reduces conversion vs action-language CTAs (sevenatoms 2026); generic | "Teklif Al" / "Projemizi Konuş" / "İletişime Geç" |
| **Real-time everything** (live ticker, live chat, live presence) | "Modern, dynamic" | Cost (websockets infra), complexity, no B2B buyer-facing value; PROJECT.md is frontend-only with mock data | Static or mock-streaming where it adds storytelling (AI assistant Phase 6) |
| **Endless project grid (no pagination, no filter)** | "Show everything" | 50+ projects without filter = unusable; current erkincelik.com gap; buyers leave when they can't find their sector | Filter + pagination (or load-more) + sectoral landing pages |
| **PDF-only technical content** (weight tables as PDF only) | "We have a catalog" | Worst SEO outcome; not scannable; not searchable; not mobile-friendly | HTML interactive tables (Phase 1), with PDF download as supplement |
| **Stock photography of "businessmen shaking hands"** | "Looks corporate" | Identifies as low-budget; B2B buyers see through it; current site does this | Real fabrika + project photography exclusively |
| **Generic "Lorem ipsum"-style hero copy** ("Quality is our priority") | Default copywriting | Says nothing differentiating; current site reads like every competitor | Specific, quantified, sector-anchored copy ("8.400 ton/yıl kapasite, havalimanından köprüye") |
| **Marquee / scrolling text strips** | "Looks dynamic" | Accessibility-hostile (auto-motion); cheap-feeling; users can't read | Static logo strip with hover-color |
| **Modal pop-ups for newsletter on first visit** | "Capture leads aggressively" | Increases bounce; mobile UX hostile; brand-damaging at premium positioning | Inline newsletter section in footer or after blog post |
| **Endless one-page scroll for everything** | "Modern single-page feel" | Bad for SEO (no per-section URL); buried content; bad for mobile (huge scroll) | Multi-page architecture with deep links per section |
| **"Hamburger only" desktop nav** | "Minimalist" | Hides nav from desktop scanners; reduces engagement; B2B buyers want to see service breadth | Visible nav desktop, hamburger only mobile (<lg breakpoint) |
| **Skeuomorphic / bevel-heavy "industrial steel-textured" UI** | "Looks like our product" | Reads as 2010-era; hostile to readability and dark mode; current site's mistake | Premium typography + flat geometric layout + selective metallic accents (copper/orange on dark) |
| **CAPTCHA-on-everything** (visible reCAPTCHA on every form) | "Spam protection" | Friction kills conversion; v3 invisible is fine; honeypot covers most cases | Honeypot first; reCAPTCHA v3 invisible; v2 only as last resort |
| **Auto-translate via browser hint instead of explicit switcher** | "Easier" | Inconsistent results; bad SEO (no `hreflang`); breaks Turkish diacritics | Explicit TR/EN switcher with localized URLs |
| **Building a real backend (auth, CMS, AI gateway, email send)** | "Make it real" | **Out of scope per PROJECT.md** — milestone is frontend mock; backend = next milestone | Clearly mock all dynamic data via TypeScript layer; gate "real" features behind feature flags |

---

## Feature Dependencies

```
Design System (Phase 1+2: theme, tokens, shadcn primitives, dark/light)
    └──required-by──> ALL UI features
        └──required-by──> Hero, Project cards, Forms, Tables, etc.

Mock Data Layer (Phase 1: TS types + sample projects/blog/certs)
    └──required-by──> Project list, Project detail, Map, Calculator (cost), Blog
        └──required-by──> Filtering, Search, Sectoral landing pages

i18n Routing (Phase 1: /tr ↔ /en URL structure, dictionary)
    └──required-by──> Language switcher, hreflang, localized formats
        └──enhances──> SEO (international targeting)

Project Detail Page (Phase 1)
    └──required-by──> Project Map (Phase 5: pins click → detail)
    └──required-by──> Case Study (Phase 3: long-form is project-detail variant)
    └──required-by──> Related Projects rail
    └──enhances──> Schema.org Project markup

Project Filter (Phase 1)
    └──requires──> Mock Data Layer with sector/year/location/tonnage fields
    └──enhances──> Sectoral landing pages

Weight Tables HTML (Phase 1)
    └──required-by──> Steel Weight Calculator (Phase 4: shares profile data)
    └──required-by──> Profile Comparison Tool (Phase 4)

Steel Weight Calculator (Phase 4)
    └──requires──> Profile data (HEB/HEA/IPE/IPN/box/pipe with weight per meter)
    └──enhances──> SEO long-tail (("ipe 200 ağırlık hesapla"))

Project Map (Phase 5)
    └──requires──> Project Detail Page + project geo-coordinates in mock data
    └──requires──> Map library decision (Mapbox vs react-leaflet) — STACK.md decides

3D Model Viewer (Phase 5)
    └──requires──> Sample GLTF asset
    └──conflicts-with──> Lighthouse 95+ on the page where it loads
        └──mitigation──> Lazy-load on user gesture; dedicated subroute

AI Assistant UI Mock (Phase 6)
    └──requires──> Design system (chat bubble component)
    └──requires-NOT──> Real LLM (explicitly out of scope)

Customer Portal UI Mock (Phase 8)
    └──requires──> Mock auth state (no real Clerk)
    └──requires──> Mock orders/documents data
    └──conflicts-with──> Public site SEO if portal routes are crawlable
        └──mitigation──> robots.txt disallow /portal/*

RFQ Form (Phase 1)
    └──requires──> Spam protection (honeypot Phase 1; reCAPTCHA later)
    └──requires──> KVKK consent UI
    └──requires-NOT──> Real backend/email send (mock submission this milestone)

Cookie Consent (Phase 1)
    └──required-by──> Any analytics/tracking integration
    └──blocks──> Maps embed (until consent given) — design decision needed

Dark Mode (Phase 1)
    └──requires──> Color system with semantic tokens (not raw hex)
    └──required-by──> Premium aesthetic positioning

Scroll-driven Animations (Phase 1+)
    └──conflicts-with──> prefers-reduced-motion users
        └──mitigation──> CSS @media query gates motion

Project Map (Phase 5)
    └──conflicts-with──> Maps embed in Contact (Google Maps) for performance
        └──mitigation──> Lazy-load both; preconnect to map host
```

### Dependency Notes

- **Design System is the foundational dependency for every UI feature.** Phase 1 must ship tokens, theme switching, and core shadcn primitives before anything else gets visual polish. This is why PROJECT.md merges Phase 1+2.
- **Mock Data Layer must exist before list/detail/filter/search/map can be built.** Schema decisions made now lock in Phase 5 map data shape (must include `coords: {lat, lng}`).
- **Project Detail Page is the hub of cross-feature navigation.** Map pins, case studies, related rails, and Schema.org markup all derive from it. Build it early and richly.
- **Weight tables (HTML, interactive) and the calculator share the same profile dataset.** Define the data shape once in Phase 1 so Phase 4 calculator just consumes it.
- **3D viewer + heavy media + maps are Lighthouse-95 risks** — keep them out of the homepage critical path; lazy + dynamic-import them.
- **Cookie consent gates the Google Maps embed** if you treat Maps as marketing tracking. Decision needed: load Maps after consent (ideal for KVKK strictness) or load with placeholder before consent.
- **Customer Portal UI must be excluded from sitemap.xml + robots.txt** so the mock doesn't pollute Google's index.
- **AI assistant + Portal explicitly do NOT depend on backend** — they are UI-only mocks per PROJECT.md.
- **RFQ form has no backend this milestone** — the form should "submit" to a mock success state. Phase 1 ships this; real email send is a future milestone.

---

## MVP Definition

Ruthless cut: what must ship in Phase 1+2 (the "çekirdek site + tasarım sistemi") to validate the redesign with stakeholders. Everything else can land in later phases without compromising the launch story.

### Launch With (v1 — Phase 1+2)

Maps to PROJECT.md "Çekirdek Site" Active Requirements:

- [ ] **Design system (dark/light, tokens, shadcn, premium typography)** — without this, every other piece looks half-built
- [ ] **Homepage** (hero + capacity numbers + services overview + featured projects + process + trust strip + final CTA)
- [ ] **Kurumsal / About page** (story, mission/vision/values, certifications)
- [ ] **Services hub + 3 service detail pages** (Projelendirme / İmalat / Montaj)
- [ ] **Projects listing** (50+ mock projects, grid, tab Tamamlanan/Devam Eden)
- [ ] **Project detail page** (hero, metadata, gallery, narrative, related) — biggest gap vs current site
- [ ] **Project filter** (sector / year / location / tonnage, URL-state) — biggest UX gap vs current site
- [ ] **Sectoral landing pages** (Havalimanı / Hangar / Köprü / AVM / Otel / Fabrika)
- [ ] **Technical info: 7 weight tables** (HEB / HEA / IPE / IPN / köşebent / kutu profil / boru) — interactive (search/filter/sort)
- [ ] **Blog list + detail** (with categories/tags/reading-time/related)
- [ ] **Contact page** (2 offices, maps, hours, RFQ form + general form, KVKK, honeypot)
- [ ] **i18n routing scaffold** (TR primary, EN URL/dictionary stubs)
- [ ] **Dark/light mode toggle** (persisted, OS-respecting)
- [ ] **Accessibility AA** (keyboard, ARIA, contrast, alt, reduce-motion, skip-link)
- [ ] **Performance** (next/image AVIF/WebP, font subsetting, code-split, Lighthouse 95+)
- [ ] **SEO** (per-page metadata, OG, sitemap, robots, Schema.org Org / LocalBusiness × 2 / Project)
- [ ] **Cookie consent + KVKK page + Çerez politikası page**
- [ ] **404 + error boundary pages**
- [ ] **Mock data layer** (TS types + sample projects/blog/certs/profiles)
- [ ] **Smoke + e2e tests** (Playwright on homepage / project detail / filter / contact form / theme toggle / lang toggle)
- [ ] **Form spam protection** (honeypot)
- [ ] **Animated capacity numbers** + scroll-driven hero animation (with reduce-motion support)

### Add After Validation (v1.x — Phase 3 Content & Marketing)

Trigger: stakeholders sign off on Phase 1+2 visual + interaction quality.

- [ ] Reference logos strip (real client logos)
- [ ] Case study deep-dive (1–3 flagship projects, e.g., İstanbul Cargo Energy 820t)
- [ ] Team / leadership page
- [ ] Career page
- [ ] Press / news archive (reuses blog engine)
- [ ] Newsletter signup UI (mock submit)
- [ ] Downloadable certificate PDFs with preview thumbnail
- [ ] Downloadable corporate catalog PDF
- [ ] Blog enhancements (categories sidebar, tag pages, reading time, related posts polish)

### Phase 4 — Engineering Tools (post-content)

- [ ] **Steel weight calculator** (profile + length → kg, optional cost)
- [ ] **Profile comparison tool** (HEB 200 vs IPE 240 etc.)
- [ ] **Unit converter** (kg ↔ lb, mm ↔ in)
- [ ] Enhanced interactive tables (column-level filter, multi-sort, CSV export)

### Phase 5 — Map & 3D

- [ ] **Project map** (TR + global pins, sector overlay, click → project detail)
- [ ] **3D model viewer** (one sample GLTF, dedicated subroute)

### Phase 6 — AI Assistant UI Mock

- [ ] Chat panel UI ("hangi profil?" → mock streaming answer)
- [ ] Mock streaming animation (`setInterval` token-by-token)

### Phase 7 — Live Production & Media Mock

- [ ] Video gallery / fabrika tour (with poster, controls, no autoplay-sound)
- [ ] Interactive process diagram (SVG-based, hover/click states)
- [ ] Mock "live stream" placeholder

### Phase 8 — Customer Portal UI Mock

- [ ] Login UI (no real auth)
- [ ] Dashboard cards (sipariş/proje takip mock)
- [ ] Document list + download UI (mock files)
- [ ] Admin vs müşteri role-based view (mock state)

### Future Consideration (v2+ — Next Milestone, Not This One)

- [ ] Real LLM integration (replaces Phase 6 mock)
- [ ] Real auth (Clerk/Auth0 — replaces Phase 8 mock login)
- [ ] Real customer portal backend (orders DB, document upload)
- [ ] Headless CMS integration (Sanity / Payload — replaces mock data layer)
- [ ] Real email/RFQ form backend (Resend / Postmark / SES)
- [ ] WP content migration (decision: scrape image only per PROJECT.md)
- [ ] Hosting + CI/CD (Vercel decision pending per PROJECT.md)
- [ ] Site search (when content volume grows past local-index threshold)
- [ ] Live chat (only if proven demand from sales team)
- [ ] Real-time production data feed (replaces Phase 7 mock)
- [ ] Real 3D project models (replaces sample GLTF)
- [ ] Mobile native app (explicitly OOS per PROJECT.md)

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Project detail page | HIGH | MEDIUM | **P1** |
| Project filter (sector/year/loc/tonnage) | HIGH | MEDIUM | **P1** |
| Design system (dark/light + premium tokens) | HIGH | HIGH | **P1** |
| Hero + homepage core sections | HIGH | MEDIUM | **P1** |
| Services hub + 3 detail pages | HIGH | MEDIUM | **P1** |
| Interactive weight tables | HIGH | MEDIUM | **P1** |
| Contact page + RFQ form | HIGH | MEDIUM | **P1** |
| i18n routing scaffold | MEDIUM | MEDIUM | **P1** (constraint per PROJECT.md) |
| Dark mode toggle | HIGH | LOW | **P1** |
| Accessibility AA | HIGH | MEDIUM | **P1** (constraint) |
| Performance (Lighthouse 95+) | HIGH | MEDIUM | **P1** (constraint) |
| SEO + Schema.org | HIGH | LOW | **P1** |
| Cookie consent + KVKK | HIGH | LOW | **P1** (legal) |
| Sectoral landing pages | MEDIUM | LOW | **P1** (high SEO leverage, low cost) |
| Blog list + detail | MEDIUM | LOW | **P1** |
| Mock data layer | HIGH | MEDIUM | **P1** (foundation) |
| 404 + error pages | MEDIUM | LOW | **P1** |
| Capacity number animation | MEDIUM | LOW | **P1** (high impact / low cost) |
| Trust strip (logos + certs) | HIGH | LOW | **P1** |
| Reference logos (real clients) | HIGH | LOW | **P2** (Phase 3) |
| Case study deep-dive | HIGH | MEDIUM | **P2** (Phase 3) |
| Career page | MEDIUM | LOW | **P2** (Phase 3) |
| Team / leadership | MEDIUM | LOW | **P2** (Phase 3) |
| Newsletter UI | MEDIUM | LOW | **P2** (Phase 3) |
| Press archive | MEDIUM | LOW | **P2** (Phase 3) |
| Downloadable PDFs | MEDIUM | LOW | **P2** (Phase 3) |
| Steel weight calculator | HIGH | MEDIUM | **P2** (Phase 4) |
| Profile comparison | MEDIUM | MEDIUM | **P2** (Phase 4) |
| Project map (TR + world) | HIGH | HIGH | **P2** (Phase 5) |
| 3D model viewer | MEDIUM | HIGH | **P3** (Phase 5) |
| AI assistant UI mock | MEDIUM | MEDIUM | **P3** (Phase 6) |
| Customer portal UI mock | MEDIUM | MEDIUM | **P3** (Phase 8) |
| Live production media mock | MEDIUM | MEDIUM | **P3** (Phase 7) |
| Interactive process diagram | MEDIUM | MEDIUM | **P3** (Phase 7) |
| Site search | MEDIUM | MEDIUM | **P3** (defer) |

**Priority key:**
- **P1**: Must have for Phase 1+2 launch (the "çekirdek site")
- **P2**: Should have, scheduled in Phase 3–5 plan
- **P3**: Mock-tier or deferred — adds polish/forward-signaling but not credibility-blocking

---

## Competitor Feature Analysis

| Feature | Severfield | Voortman | Thyssenkrupp | Current erkincelik.com | **Erkin Çelik (target)** |
|---------|------------|----------|--------------|------------------------|-----------------------|
| Project detail page | Yes (deep, sector-grouped) | Yes (case-study style) | Story format | **No** | **Yes — rich w/ tonnage/loc/gallery** |
| Project filter | Sectoral nav | By solution | By industry | **No** | **Yes — sector/year/loc/tonnage** |
| Sectoral landing | Yes (Stadia/Transport/etc) | Yes | Yes (11 sectors) | No | **Yes — 6 Turkish sectors** |
| Case study (long-form) | Yes | Yes (testimonials with metrics) | Yes (stories) | No | **Yes (Phase 3)** |
| Interactive map | Limited (locations only) | Limited | Global locator | No | **Yes — full project map (Phase 5)** |
| Steel weight calculator | No | No | No | Static tables only | **Yes (Phase 4) — differentiator** |
| Interactive weight tables | No | No | No | **Static** | **Yes — search/filter/sort** |
| 3D viewer | No | No | No | No | **Yes — sample (Phase 5) — differentiator** |
| AI assistant | No | No | Featured AI in supply chain | No | **Yes — UI mock (Phase 6) — differentiator** |
| Customer portal | Yes (employee + investor) | Yes (login) | Yes (investor) | No | **Yes — UI mock (Phase 8)** |
| Multi-language | English only (UK arm) | 7 languages | DE/EN | **TR only** | **TR + EN scaffold** |
| Dark mode | No | No | No | No | **Yes — primary aesthetic — differentiator** |
| Hero strategy | Single hero + featured strip | Single hero + sector cards | Single hero + story tiles | **Carousel (anti-pattern)** | **Single static/video hero (no carousel)** |
| RFQ form | "Contact" form | "Get Quote" + "Demo Request" | Contact only | Generic contact | **Dedicated RFQ + general dual** |
| Cert visibility | Listed in About | Listed in production page | Investor section | Listed | **Hero strip + cert page + downloads** |
| Newsletter | Yes | Yes | Press subscribe | No | **Yes — UI mock (Phase 3)** |
| Career section | Rich (graduates/apprenticeships) | Standard | External portal | Minimal | **Phase 3** |
| Cookie consent | EU-compliant | EU-compliant | EU-compliant | Basic/none | **KVKK-compliant + categorized** |
| Animation style | Subtle, premium | Functional | Editorial | Carousel-heavy | **Scroll-driven cinematic, reduce-motion respected** |

**Strategic positioning vs competitors:**
- Erkin Çelik **matches** Severfield/Voortman on portfolio richness, certifications, and sectoral organization (table stakes).
- Erkin Çelik **exceeds** all reference sites on: dark-mode premium aesthetic, steel weight calculator, interactive 3D, AI assistant UI, project map, interactive technical tables, dedicated RFQ vs general contact split.
- Erkin Çelik **fixes the existing site's gaps**: project detail pages, filtering, i18n, dark mode, schema, performance, premium copy.

---

## Sources

**Reference sites (directly inspected via WebFetch):**
- [Severfield (UK) homepage and structure](https://www.severfield.com/)
- [Severfield Europe (NL/EU) — industrial steel construction](https://www.severfield.eu/en/industrial-steel-construction)
- [Voortman Steel Machinery — homepage, navigation, demo flow](https://www.voortman.net/en)
- [thyssenkrupp corporate (en) — hero, sector, story strategy](https://www.thyssenkrupp.com/en)
- [Schüco International — global brand portfolio + multi-region](https://www.schueco.com/)
- [Kingspan country selector landing](https://www.kingspan.com/) (limited content fetched, region selector confirmed)
- [Erkin Çelik — current site (gap baseline)](https://erkincelik.com/) (PROJECT.md context)

**B2B / industrial best-practice literature (HIGH confidence — multiple-source-corroborated):**
- [Top Industrial Website Design Trends & Best Practices 2026 — SRH Web Agency](https://srhwebagency.com/industrial-website-design-in-2026/)
- [Lead Generation for Steel and Metalwork Companies — Manufacturing Lead Generation](https://manufacturingleadgeneration.com/lead-generation-strategies-metal-fabricators/)
- [Lead Generation for Manufacturers — SevenAtoms](https://www.sevenatoms.com/blog/lead-generation-for-manufacturers)
- [Enhancing Your B2B RFQ Form — RedMoxy](https://redmoxy.com/enhancing-your-b2b-companys-request-for-quote-rfq-form-to-generate-more-leads/)
- [B2B Website Trust Signals — Trajectory Web Design](https://www.trajectorywebdesign.com/blog/b2b-website-trust-signals)
- [8 Trust Signals You Need on Your Website — Webstacks](https://www.webstacks.com/blog/trust-signals)
- [B2B Web Design Best Practices — Intuitia](https://www.intuitia.tech/blog/b2b-website-design)
- [B2B Website Best Practices to Drive Leads in 2026 — Paradigm Marketing](https://www.paradigmmarketinganddesign.com/b2b-website-design-best-practices-2026/)

**Hero / navigation / patterns:**
- [Website Hero Section Best Practices — Prismic](https://prismic.io/blog/website-hero-section)
- [10 best hero section examples — LogRocket](https://blog.logrocket.com/ux-design/hero-section-examples-best-practices/)
- [10 Construction Website Design Trends For 2026 — Design Hero](https://www.design-hero.com/business-tips/construction-website-design-trends/)

**Anti-pattern evidence (carousel / autoplay / motion):**
- [Don't use automatic image sliders or carousels — CXL](https://cxl.com/blog/dont-use-automatic-image-sliders-or-carousels/)
- [5 Big Usability Mistakes on Carousels — UX Movement](https://uxmovement.com/navigation/big-usability-mistakes-designers-make-on-carousels/)
- [Are Product Carousels Bad for Accessibility? — BOIA](https://www.boia.org/blog/are-product-carousels-bad-for-accessibility)
- [Carousel UX Patterns for Developers — uxpatterns.dev](https://uxpatterns.dev/patterns/content-management/carousel)

**Case study structure (project detail) :**
- [Best Practices for B2B Case Studies — Mediashower](https://www.mediashower.com/blog/best-practices-for-b2b-case-studies/)
- [Design Better B2B Case Studies — Stryve Marketing](https://www.stryvemarketing.com/blog/design-b2b-case-studies/)
- [B2B Case Study Template — Logonaut](https://www.thelogonaut.com/post/b2b-case-study-template-10-examples-2025-best-practices)
- [Writing High-Converting B2B Case Studies — Omniscient Digital](https://beomniscient.com/blog/writing-high-converting-b2b-case-studies/)

**Steel weight calculator references (Phase 4 design):**
- [Steel Beam Weight Calculator — angleroller.com](https://www.angleroller.com/profile-bending/steel-beam-weight-calculator-ipe-ipn-hea-heb-hem.html)
- [HEB/IPB Beam Calculator — Toolerz](https://www.toolerz.com/heb-beams-weight-calculator/)
- [I-Beam Weight and Cost Calculator — calculatesteel.com](https://calculatesteel.com/i-beam-weight-and-cost-calculator)
- [Iron Pipes & Beams Weight Calculator — Tubilaser](https://www.tubilaser.com/en/iron-pipes-and-beams-weight-calculator/)

**Interactive maps (Phase 5):**
- [Examples of Interactive Maps on Websites — New Media Campaigns](https://www.newmediacampaigns.com/blog/examples-of-interactive-maps-on-websites)
- [Build an Interactive Map with React Leaflet — Strapi](https://strapi.io/blog/how-to-build-an-interactive-map-with-react-leaflet-and-strapi)

**i18n / Turkish-specific:**
- [Best Practices in Website Localization (TR-specific) — Websoftik](https://websoftik.com/en/blog/what-are-the-best-practices-in-website-and-web-application-localization)
- [Website Internationalization Guide — Weglot](https://www.weglot.com/guides/website-internationalization)

**SEO / Schema.org:**
- [Schema Markup for Contractor Websites — ESEOSpace](https://eseospace.com/blog/schema-markup-for-contractor-websites/)
- [LocalBusiness Schema How-to — Schema App](https://www.schemaapp.com/schema-markup/how-to-do-schema-markup-for-local-business/)
- [Local SEO Schema Complete Guide — Search Engine Journal](https://www.searchenginejournal.com/how-to-use-schema-for-local-seo-a-complete-guide/294973/)

---

*Feature research for: B2B steel construction / industrial fabrication marketing site (Erkin Çelik)*
*Researched: 2026-04-25*
