# Requirements: Erkin Çelik — Modern Web Yenileme

**Defined:** 2026-04-25
**Core Value:** Mevcut WordPress sitesinin yerini alabilecek, B2B çelik konstrüksiyon müşterisine "bu firma profesyonel" hissi veren ve teklif/iletişim formuna kolay yönlendiren modern bir vitrin.

## v1 Requirements

Requirements for the initial release. Each maps to a roadmap phase. All dynamic data is mock; no real backend.

### Foundation (Next.js 16 + i18n + theming + mock layer)

- [ ] **FND-01**: `proxy.ts` (Next 16'nın `middleware.ts` halefi) ile locale negotiation yapılır; layout'lar header okumaz
- [ ] **FND-02**: `app/[lang]/(marketing) (portal) (legal)` route group yapısı kurulur (3 farklı chrome)
- [ ] **FND-03**: `next-intl@^4.4` ile TR primary + EN scaffold (URL segment tabanlı: `/tr/...` ↔ `/en/...`)
- [ ] **FND-04**: TR sözlük tam, EN sözlük `[EN-TODO]` placeholder ile parallel scaffold
- [ ] **FND-05**: `next-themes@^0.4.6` ile dark/light toggle; default dark, `prefers-color-scheme` desteği, `disableTransitionOnChange`
- [ ] **FND-06**: `<html suppressHydrationWarning>` + theme-toggle'da `mounted` guard ile FOUC + hydration mismatch yok
- [ ] **FND-07**: Tailwind v4 setup `app/globals.css` içinde — `@import "tailwindcss"`, `@custom-variant dark`, `@theme inline { ... }` token'ları
- [ ] **FND-08**: `src/mock/types.ts` — `LocalizedString`, `Project`, `BlogPost`, `Service`, `Certification`, `Profile`, `Office` tipleri + Zod şemaları
- [ ] **FND-09**: `src/mock/index.ts` (`'server-only'`) — async query fonksiyonları (`listProjects`, `getProjectBySlug`, vs.); 50+ proje, 7 profil veri seti, ~6 blog yazısı, 3 sertifika, 2 ofis
- [ ] **FND-10**: `next.config.ts` hardened — explicit image config (qualities, AVIF/WebP, remotePatterns, localPatterns), React Compiler aktif
- [ ] **FND-11**: `npx next typegen` çalıştırılır; `PageProps<'/...'>` type helpers kullanılır
- [ ] **FND-12**: `scripts/optimize-images.ts` (sharp) prebuild'de çalışır; AVIF + WebP üretir; hero ≤ 200KB, content ≤ 80KB cap; `blurDataURL`
- [ ] **FND-13**: erkincelik.com'dan proje görselleri scrape edilir, asset-register.csv ile kaynak/lisans/kullanım izni belgelenir (Phase 2 visual work başlamadan)

### Design System

- [ ] **DSN-01**: shadcn/ui (Tailwind v4 distribution) initialized — Button, Card, Input, Tabs, Dialog, Sheet, Select, Combobox, Slider, Table, NavigationMenu, Tooltip, Sonner, Form, Label, Separator, Skeleton primitives vendored
- [ ] **DSN-02**: Endüstriyel-premium palet (koyu zemin + bakır/turuncu metalik vurgu); body ≥ 4.5:1, large text ≥ 3:1, non-text UI ≥ 3:1 (WCAG AA contrast checked)
- [ ] **DSN-03**: Premium tipografi (Türkçe latin + latin-ext alt küme: ç ş ğ ı ö ü); display + body font hierarchy
- [ ] **DSN-04**: `motion@^12` ile `LazyMotion features={domAnimation} strict` provider; `<m.div>` only; `transform`+`opacity` only
- [ ] **DSN-05**: `<MotionConfig reducedMotion="user">` — `prefers-reduced-motion` respect
- [ ] **DSN-06**: Mevcut Erkin Çelik logosu kullanılır (light + dark variant)
- [ ] **DSN-07**: Lucide icon set
- [ ] **DSN-08**: Branded 404 + global `error.tsx` (Client Component, `unstable_retry` callback) + `not-found.tsx`

### Marketing — Anasayfa

- [ ] **HOME-01**: Sticky header — logo + primary nav + locale switcher + theme toggle + RFQ CTA
- [ ] **HOME-02**: Tek statik veya muted-loop hero (asla auto-rotating carousel değil)
- [ ] **HOME-03**: Scroll-into-view ile capacity number count-up (8.400 ton/yıl, 16+ yıl, ISO sayısı)
- [ ] **HOME-04**: Hizmetler özet bandı (3 kart: Projelendirme / İmalat / Montaj)
- [ ] **HOME-05**: Öne çıkan projeler şeridi (4-6 proje, "Tümünü Gör" CTA)
- [ ] **HOME-06**: Üretim süreci diyagramı (5-6 adım)
- [ ] **HOME-07**: Trust strip (sertifika rozetleri + temsili rakamlar)
- [ ] **HOME-08**: Final CTA bandı (Teklif Al + İletişim)
- [ ] **HOME-09**: Footer — 4 sütun (adresler, sertifikalar, legal, sosyal)

### Marketing — Kurumsal

- [ ] **CORP-01**: Hakkımızda sayfası (kuruluş hikayesi 2008'den beri, kapasite, alan)
- [ ] **CORP-02**: Misyon + vizyon + değerler bölümleri
- [ ] **CORP-03**: Sertifikalar bölümü (ISO 9001/14001/OHSAS 18001 önizleme + indirilebilir PDF)

### Marketing — Hizmetler

- [ ] **SVC-01**: Hizmetler hub sayfası (3 alt hizmete giriş)
- [ ] **SVC-02**: `/hizmetler/projelendirme` detay sayfası (içerik + inline RFQ CTA)
- [ ] **SVC-03**: `/hizmetler/imalat` detay sayfası (içerik + inline RFQ CTA)
- [ ] **SVC-04**: `/hizmetler/montaj` detay sayfası (içerik + inline RFQ CTA)

### Marketing — Projeler

- [ ] **PROJ-01**: `/projeler` proje grid (50+ kart: görsel, başlık, sektör, lokasyon, yıl, tonaj)
- [ ] **PROJ-02**: URL-state filter (sektör, yıl aralığı, lokasyon, tonaj aralığı) — paylaşılabilir link
- [ ] **PROJ-03**: Tabs — "Tamamlanan" / "Devam Eden"
- [ ] **PROJ-04**: `/projeler/[slug]` proje detay sayfası — hero, metadata bloğu (tonaj/lokasyon/yıl/sektör/müşteri), 3-10 fotoğraflık galeri (Embla carousel + lightbox), ilgili projeler şeridi
- [ ] **PROJ-05**: 6 sektörel landing sayfası: `/sektorler/havalimani`, `/sektorler/hangar`, `/sektorler/kopru`, `/sektorler/avm`, `/sektorler/otel`, `/sektorler/fabrika`

### Marketing — Teknik Bilgiler

- [ ] **TECH-01**: `/teknik-bilgiler` hub sayfası
- [ ] **TECH-02**: 7 interaktif ağırlık tablosu (HEB, HEA, IPE, IPN, köşebent, kutu profil, boru) — arama + filtre + sıralama
- [ ] **TECH-03**: Birim çevirici (kg ↔ lb, mm ↔ in)

### Marketing — Blog

- [ ] **BLOG-01**: `/blog` liste sayfası
- [ ] **BLOG-02**: `/blog/[slug]` yazı detay (başlık, kapak, içerik, okuma süresi)
- [ ] **BLOG-03**: Kategori sayfası `/blog/kategori/[slug]` + etiket sayfası `/blog/etiket/[slug]`

### Marketing — İletişim

- [ ] **CNT-01**: 2 ofis kartı (Tuzla merkez + Gebze fabrika) — adres, telefon, e-posta, çalışma saatleri
- [ ] **CNT-02**: Google Maps embed (consent sonrası yüklenir)
- [ ] **CNT-03**: RFQ (Teklif Al) formu — proje türü, sektör, tonaj tahmini, mesaj
- [ ] **CNT-04**: Genel iletişim formu — ad, firma, e-posta, telefon, mesaj
- [ ] **CNT-05**: KVKK onay checkbox + KVKK metni linki
- [ ] **CNT-06**: Honeypot field + 2 saniye time-trap (backend yok ama spam korumaya hazır)
- [ ] **CNT-07**: Action-language CTA'lar ("Teklif Al" / "Gönder", asla "Click here / Submit")
- [ ] **CNT-08**: Form submit mock UI (success toast + reset; gerçek email send sonra)

### Legal & Compliance

- [ ] **LGL-01**: KVKK aydınlatma metni sayfası
- [ ] **LGL-02**: Çerez politikası sayfası
- [ ] **LGL-03**: Kullanım şartları sayfası
- [ ] **LGL-04**: KVKK uyumlu cookie banner — eşit ağırlık Kabul et / Reddet / Tercihler; analytics/Maps/external script consent öncesi yüklenmez
- [ ] **LGL-05**: Persisted consent log (localStorage) + footer "Çerez Tercihleri" yeniden açıcı

### Performance, A11y, SEO

- [ ] **PERF-01**: Lighthouse mobile + desktop ≥ 95 (Performance, Accessibility, Best Practices, SEO)
- [ ] **PERF-02**: AVIF + WebP format, `next/image` `sizes` + `priority` only on hero, lazy below fold
- [ ] **PERF-03**: Font subsetting (latin + latin-ext), `font-display: swap`
- [ ] **A11Y-01**: WCAG 2.2 AA — keyboard navigation, ARIA, focus-visible rings, skip-to-content link
- [ ] **A11Y-02**: Tüm form input'larda label, hata mesajları aria-live ile bildirilir
- [ ] **A11Y-03**: `axe-playwright` ile her template 0 WCAG AA violation
- [ ] **SEO-01**: Per-page `metadata` (title + description, locale-specific)
- [ ] **SEO-02**: Schema.org JSON-LD — Organization (root), LocalBusiness × 2 (iletisim sayfasında), CreativeWork (proje detay), Service (hizmet detay), BlogPosting (blog detay), BreadcrumbList, FAQPage
- [ ] **SEO-03**: Sitemap (TR + EN, `alternates.languages` + hreflang), robots.txt (`/portal/*` disallow)
- [ ] **SEO-04**: Per-template `opengraph-image.tsx` (homepage, project detail, blog post, service detail)
- [ ] **SEO-05**: Canonical URL'ler her locale için

### Testing

- [ ] **TST-01**: Playwright E2E setup (`@playwright/test@^1.58`)
- [ ] **TST-02**: E2E — homepage smoke test
- [ ] **TST-03**: E2E — proje filtreleme + detay sayfası akışı
- [ ] **TST-04**: E2E — iletişim formu happy path + honeypot trigger
- [ ] **TST-05**: E2E — theme toggle persists across reload
- [ ] **TST-06**: E2E — locale toggle TR ↔ EN
- [ ] **TST-07**: E2E — mobile menu drawer
- [ ] **TST-08**: E2E — cookie banner Accept + Reject paths
- [ ] **TST-09**: `axe-playwright` her template assertion
- [ ] **TST-10**: Phase 4 calculator için Vitest unit test (`lib/calc/weight.ts`)

### Content & Marketing (Phase 3)

- [ ] **CNTM-01**: Referans logoları şeridi (8-12 logo, grayscale → renk hover)
- [ ] **CNTM-02**: Flagship case study (önerilen: İstanbul Cargo Energy 820t) — challenge / solution / result + metrik
- [ ] **CNTM-03**: Ekip / yönetim tanıtım sayfası
- [ ] **CNTM-04**: Kariyer sayfası (açık pozisyon + başvuru CTA)
- [ ] **CNTM-05**: Basın / haber arşivi (blog motorunu yeniden kullanır, basın kategorisi)
- [ ] **CNTM-06**: Newsletter abonelik UI (mock submit, localStorage demo)
- [ ] **CNTM-07**: Blog enhancements — sticky table-of-contents (uzun yazılarda), ilgili yazılar
- [ ] **CNTM-08**: İndirilebilir sertifika PDF'leri + kurumsal katalog PDF
- [ ] **CNTM-09**: İçerik sprint — her proje ≥ 800 kelime, her hizmet ≥ 1000 kelime + FAQ + `FAQPage` schema

### Engineering Tools (Phase 4)

- [ ] **TOOL-01**: `src/lib/calc/weight.ts` pure logic (Vitest-tested) — profile + length → kg
- [ ] **TOOL-02**: `WeightCalculator` UI — Combobox profile picker + length input + animated result + URL-shareable params
- [ ] **TOOL-03**: `ProfileTable` enhancement — TanStack Table, CSV export
- [ ] **TOOL-04**: `ProfileCompare` — HEB 200 vs IPE 240 dual-column, URL state
- [ ] **TOOL-05**: Calculator için maliyet tahmini opsiyonu (kg × ₺/kg input)

### Map & 3D (Phase 5)

- [ ] **MAP-01**: `react-map-gl/maplibre` + `maplibre-gl@^4` (no token, free MapTiler/OSM tiles)
- [ ] **MAP-02**: `next/dynamic({ ssr: false })` Client wrapper içinde; viewport-gated
- [ ] **MAP-03**: `supercluster` ile pin clustering
- [ ] **MAP-04**: Pin click → proje detay sayfasına navigate
- [ ] **MAP-05**: Theme-aware tile style (dark/light token)
- [ ] **MAP-06**: Mobile keyboard + touch zoom destekli
- [ ] **MAP-07**: Pin a11y — alt text + keyboard nav
- [ ] **MAP-08**: Anasayfa (kompakt) ve `/projeler` (tam) sayfasına mount edilir
- [ ] **3D-01**: `three` + `@react-three/fiber@^9.6` + `@react-three/drei@^10`
- [ ] **3D-02**: Client wrapper içinde `next/dynamic({ ssr: false })`; viewport-gated
- [ ] **3D-03**: 2D poster fallback + "View in 3D" CTA + WebGL detection
- [ ] **3D-04**: DRACO/meshopt sıkıştırılmış GLTF örnek model
- [ ] **3D-05**: `<Canvas dpr={[1, 2]}>`, mobil antialias off, `<Environment preset="warehouse">`
- [ ] **3D-06**: Bundle budget — non-3D sayfalar etkilenmez

### AI Assistant Mock UI (Phase 6)

- [ ] **AI-01**: `npx ai-elements@latest add conversation message prompt-input reasoning code-block`
- [ ] **AI-02**: `src/components/tools/ai-chat/` chat shell
- [ ] **AI-03**: `app/api/chat/route.ts` mock `ReadableStream` (Vercel AI SDK `streamText` shape ile uyumlu)
- [ ] **AI-04**: `AbortController` plumbing
- [ ] **AI-05**: Error-state UI
- [ ] **AI-06**: Reduced-motion-friendly typing indicator
- [ ] **AI-07**: Live-region announcements (a11y)
- [ ] **AI-08**: Keyboard send (Enter to send, Shift+Enter newline)

### Live Production & Media Mock (Phase 7)

- [ ] **MED-01**: Video gallery / fabrika tanıtımı — `<video muted loop playsInline poster controls>`, lazy below fold, individual play buttons, captions placeholder
- [ ] **MED-02**: Üretim süreci interaktif diyagramı (SVG) — hover/click step states, scroll-driven step reveal, reduced-motion respected
- [ ] **MED-03**: Mock "canlı akış" placeholder ("yakında" kartı + newsletter CTA)

### Customer Portal Mock UI (Phase 8)

- [ ] **PRT-01**: `(portal)/layout.tsx` — sidebar + topbar chrome
- [ ] **PRT-02**: `/portal/login` mock login formu (kullanıcı adı/şifre validation, fake-success → localStorage)
- [ ] **PRT-03**: `/portal/dashboard` — sipariş/proje takip kartları (mock data)
- [ ] **PRT-04**: `/portal/belgeler` — belge listesi + indirme UI
- [ ] **PRT-05**: `/portal/siparisler` — sipariş listesi + detay
- [ ] **PRT-06**: Roller (admin / müşteri) için ayrı görünüm (role switcher dev panel)
- [ ] **PRT-07**: `default.tsx` — paralel route slot için
- [ ] **PRT-08**: `src/lib/auth/` — Clerk-shape API (`useUser`, `useAuth`, `<SignedIn>`, `<SignedOut>`); mock implementation
- [ ] **PRT-09**: robots.txt `/portal/*` disallow + sitemap exclude
- [ ] **PRT-10**: Dev banner "Mock UI — gerçek auth değil"

## v2 Requirements

Deferred to next milestone(s). Backlog tracked but not in current roadmap.

### Real Backend Integration

- **BE-01**: Headless CMS entegrasyonu (Sanity/Payload) — `src/mock/` ile aynı kontrat
- **BE-02**: Gerçek email send (Resend/Postmark) için RFQ + iletişim formları
- **BE-03**: Cloudflare Turnstile veya BotID gerçek spam koruması (reCAPTCHA değil — KVKK uyumsuz)
- **BE-04**: Vercel AI SDK `ai` + AI Gateway gerçek LLM bağlantısı (Phase 6 mock kontratı korur)
- **BE-05**: Clerk/Auth0 gerçek auth (Phase 8 mock kontratı korur)
- **BE-06**: Müşteri portalı gerçek backend — sipariş DB, belge upload, role-based access
- **BE-07**: Sertifika/katalog PDF backend yönetimi
- **BE-08**: Newsletter gerçek listesi (Mailchimp/Resend audience)

### Hosting / DevOps

- **OPS-01**: Hosting kararı (önerilen Vercel — Next.js 16 + Fluid Compute uyumlu)
- **OPS-02**: CI/CD pipeline (preview deploys per PR)
- **OPS-03**: Production analytics (Vercel Analytics veya Plausible)
- **OPS-04**: Error monitoring (Sentry)
- **OPS-05**: Vercel BotID veya Cloudflare Turnstile production

### Content Expansion

- **CXP-01**: Tam EN içerik çevirisi (TR'den EN'e)
- **CXP-02**: Gerçek proje 3D modelleri (mock GLTF yerine)
- **CXP-03**: Gerçek müşteri/referans logoları (lisans onayı sonrası)
- **CXP-04**: Site-wide arama (~50 proje + ~20 blog için client-side index, sonra server-side)

### Differentiators (advanced)

- **ADV-01**: Live üretim akışı — fabrikadan gerçek video/görsel feed
- **ADV-02**: Interactive harita üzerinde proje sürecini gösteren timeline
- **ADV-03**: Saved searches / "Bana benzer projeler" öneri (auth gerektirir)
- **ADV-04**: Rich blog yazı tipi (galeri, video embed, kod blok)

## Out of Scope

Explicitly excluded. Documented to prevent scope creep and stakeholder requests.

| Feature | Reason |
|---------|--------|
| Real backend / DB / auth this milestone | Frontend-only milestone; tüm dinamik veri mock; backend sonraki milestone |
| Headless CMS bu milestone | Mock data layer kontrak görevi görür; CMS sonra |
| Hosting / deploy / CI bu milestone | Local dev odağı; karar sonra |
| Mevcut WordPress içerik migration'ı | Sadece proje görselleri scrape; metinler sıfırdan, premium copy |
| Mobile native app | Web önceliği; responsive web yeterli |
| Auto-rotating hero carousel | Anti-feature: %1 engagement, accessibility-hostile (referans siteler hepsi terk etti) |
| Autoplay video with sound | Anti-feature: kullanıcı düşmanı, mobil veri dostu değil |
| Splash / intro animation | Anti-feature: TTI'yı bozar, geri dönen ziyaretçi için friction |
| Scroll-jacking | Anti-feature: a11y + UX katili |
| Eager live-chat pop-up | Anti-feature: backend yok zaten; ileride opt-in |
| Cursor-following efektler | Anti-feature: motion sickness, mobil gereksiz |
| Skeuomorphic steel-textured UI | Anti-feature: estetik kararı premium-modern, retro değil |
| reCAPTCHA | KVKK uyumsuz (Google US data sharing); honeypot+timetrap şimdi → Turnstile sonra |
| Modal pop-ups on first visit | Anti-feature: B2B müşteri için saldırgan |
| Endless one-page scroll | Anti-feature: SEO + navigasyon katili |
| Hamburger-only desktop nav | Anti-feature: B2B için sticky full nav daha güvenilir |
| Auto-translate via browser hint | Anti-feature: kötü çeviri, kontrol yok; explicit locale switcher |
| `output: 'export'` static export | proxy + image optimization + cookies + Server Actions ile uyumsuz |
| Edge runtime | Next 16 proxy'de yasak; node-only sadelik |
| `framer-motion` paketi | Frozen; `motion@^12` kullanılır |

## Traceability

Filled by roadmapper agent on 2026-04-25. Roadmap renumbers PROJECT.md's 8-phase plan into 7 sequential phases (Phase 1+2 merged per PROJECT.md decision).

**Phase Map:**
- **Phase 1** = Çekirdek Site + Tasarım Sistemi (PROJECT.md Phase 1+2 merged)
- **Phase 2** = İçerik & Pazarlama (was Phase 3 in PROJECT.md)
- **Phase 3** = Mühendislik Araçları (was Phase 4 in PROJECT.md)
- **Phase 4** = Harita & 3D (was Phase 5 in PROJECT.md)
- **Phase 5** = AI Asistan UI mock (was Phase 6 in PROJECT.md)
- **Phase 6** = Canlı Üretim & Medya mock (was Phase 7 in PROJECT.md)
- **Phase 7** = Müşteri Portalı UI mock (was Phase 8 in PROJECT.md)

| Requirement | Phase | Status |
|-------------|-------|--------|
| FND-01 | Phase 1 | Pending |
| FND-02 | Phase 1 | Pending |
| FND-03 | Phase 1 | Pending |
| FND-04 | Phase 1 | Pending |
| FND-05 | Phase 1 | Pending |
| FND-06 | Phase 1 | Pending |
| FND-07 | Phase 1 | Pending |
| FND-08 | Phase 1 | Pending |
| FND-09 | Phase 1 | Pending |
| FND-10 | Phase 1 | Pending |
| FND-11 | Phase 1 | Pending |
| FND-12 | Phase 1 | Pending |
| FND-13 | Phase 1 | Pending |
| DSN-01 | Phase 1 | Pending |
| DSN-02 | Phase 1 | Pending |
| DSN-03 | Phase 1 | Pending |
| DSN-04 | Phase 1 | Pending |
| DSN-05 | Phase 1 | Pending |
| DSN-06 | Phase 1 | Pending |
| DSN-07 | Phase 1 | Pending |
| DSN-08 | Phase 1 | Pending |
| HOME-01 | Phase 1 | Pending |
| HOME-02 | Phase 1 | Pending |
| HOME-03 | Phase 1 | Pending |
| HOME-04 | Phase 1 | Pending |
| HOME-05 | Phase 1 | Pending |
| HOME-06 | Phase 1 | Pending |
| HOME-07 | Phase 1 | Pending |
| HOME-08 | Phase 1 | Pending |
| HOME-09 | Phase 1 | Pending |
| CORP-01 | Phase 1 | Pending |
| CORP-02 | Phase 1 | Pending |
| CORP-03 | Phase 1 | Pending |
| SVC-01 | Phase 1 | Pending |
| SVC-02 | Phase 1 | Pending |
| SVC-03 | Phase 1 | Pending |
| SVC-04 | Phase 1 | Pending |
| PROJ-01 | Phase 1 | Pending |
| PROJ-02 | Phase 1 | Pending |
| PROJ-03 | Phase 1 | Pending |
| PROJ-04 | Phase 1 | Pending |
| PROJ-05 | Phase 1 | Pending |
| TECH-01 | Phase 1 | Pending |
| TECH-02 | Phase 1 | Pending |
| TECH-03 | Phase 1 | Pending |
| BLOG-01 | Phase 1 | Pending |
| BLOG-02 | Phase 1 | Pending |
| BLOG-03 | Phase 1 | Pending |
| CNT-01 | Phase 1 | Pending |
| CNT-02 | Phase 1 | Pending |
| CNT-03 | Phase 1 | Pending |
| CNT-04 | Phase 1 | Pending |
| CNT-05 | Phase 1 | Pending |
| CNT-06 | Phase 1 | Pending |
| CNT-07 | Phase 1 | Pending |
| CNT-08 | Phase 1 | Pending |
| LGL-01 | Phase 1 | Pending |
| LGL-02 | Phase 1 | Pending |
| LGL-03 | Phase 1 | Pending |
| LGL-04 | Phase 1 | Pending |
| LGL-05 | Phase 1 | Pending |
| PERF-01 | Phase 1 | Pending |
| PERF-02 | Phase 1 | Pending |
| PERF-03 | Phase 1 | Pending |
| A11Y-01 | Phase 1 | Pending |
| A11Y-02 | Phase 1 | Pending |
| A11Y-03 | Phase 1 | Pending |
| SEO-01 | Phase 1 | Pending |
| SEO-02 | Phase 1 | Pending |
| SEO-03 | Phase 1 | Pending |
| SEO-04 | Phase 1 | Pending |
| SEO-05 | Phase 1 | Pending |
| TST-01 | Phase 1 | Pending |
| TST-02 | Phase 1 | Pending |
| TST-03 | Phase 1 | Pending |
| TST-04 | Phase 1 | Pending |
| TST-05 | Phase 1 | Pending |
| TST-06 | Phase 1 | Pending |
| TST-07 | Phase 1 | Pending |
| TST-08 | Phase 1 | Pending |
| TST-09 | Phase 1 | Pending |
| CNTM-01 | Phase 2 | Pending |
| CNTM-02 | Phase 2 | Pending |
| CNTM-03 | Phase 2 | Pending |
| CNTM-04 | Phase 2 | Pending |
| CNTM-05 | Phase 2 | Pending |
| CNTM-06 | Phase 2 | Pending |
| CNTM-07 | Phase 2 | Pending |
| CNTM-08 | Phase 2 | Pending |
| CNTM-09 | Phase 2 | Pending |
| TOOL-01 | Phase 3 | Pending |
| TOOL-02 | Phase 3 | Pending |
| TOOL-03 | Phase 3 | Pending |
| TOOL-04 | Phase 3 | Pending |
| TOOL-05 | Phase 3 | Pending |
| TST-10 | Phase 3 | Pending |
| MAP-01 | Phase 4 | Pending |
| MAP-02 | Phase 4 | Pending |
| MAP-03 | Phase 4 | Pending |
| MAP-04 | Phase 4 | Pending |
| MAP-05 | Phase 4 | Pending |
| MAP-06 | Phase 4 | Pending |
| MAP-07 | Phase 4 | Pending |
| MAP-08 | Phase 4 | Pending |
| 3D-01 | Phase 4 | Pending |
| 3D-02 | Phase 4 | Pending |
| 3D-03 | Phase 4 | Pending |
| 3D-04 | Phase 4 | Pending |
| 3D-05 | Phase 4 | Pending |
| 3D-06 | Phase 4 | Pending |
| AI-01 | Phase 5 | Pending |
| AI-02 | Phase 5 | Pending |
| AI-03 | Phase 5 | Pending |
| AI-04 | Phase 5 | Pending |
| AI-05 | Phase 5 | Pending |
| AI-06 | Phase 5 | Pending |
| AI-07 | Phase 5 | Pending |
| AI-08 | Phase 5 | Pending |
| MED-01 | Phase 6 | Pending |
| MED-02 | Phase 6 | Pending |
| MED-03 | Phase 6 | Pending |
| PRT-01 | Phase 7 | Pending |
| PRT-02 | Phase 7 | Pending |
| PRT-03 | Phase 7 | Pending |
| PRT-04 | Phase 7 | Pending |
| PRT-05 | Phase 7 | Pending |
| PRT-06 | Phase 7 | Pending |
| PRT-07 | Phase 7 | Pending |
| PRT-08 | Phase 7 | Pending |
| PRT-09 | Phase 7 | Pending |
| PRT-10 | Phase 7 | Pending |

**Coverage:**
- v1 requirements: **131 total** (verified by roadmapper — original "113" estimate was inaccurate)
- Mapped to phases: **131** (100%)
- Unmapped: **0** ✓

**Per-phase counts:**
- Phase 1 (Çekirdek + Tasarım Sistemi): **81** requirements
- Phase 2 (İçerik & Pazarlama): **9** requirements
- Phase 3 (Mühendislik Araçları): **6** requirements
- Phase 4 (Harita & 3D): **14** requirements
- Phase 5 (AI Asistan UI mock): **8** requirements
- Phase 6 (Canlı Üretim & Medya mock): **3** requirements
- Phase 7 (Müşteri Portalı UI mock): **10** requirements
- **Total: 131** ✓

---
*Requirements defined: 2026-04-25*
*Last updated: 2026-04-25 by roadmapper — traceability filled, coverage 131/131*
