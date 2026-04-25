# Roadmap: Erkin Çelik — Modern Web Yenileme

## Overview

Mevcut WordPress sitesinin (erkincelik.com) yerini alacak, **endüstriyel-premium** estetikte, B2B çelik konstrüksiyon müşterilerine güven veren modern bir vitrin. Yolculuk: önce çekirdek site + tasarım sistemi + tüm pazarlama şablonları (Phase 1) ile WordPress'i fonksiyon olarak yenip geçiyoruz; ardından içerik derinliği (Phase 2) ile rakiplerden ayrışıyoruz; sonra her faz tek bir farklılaştırıcıyı (mühendislik araçları, harita & 3D, AI asistan, üretim/medya, müşteri portalı) stabil çekirdeğin üzerine ekliyor. Tüm fazlar frontend-only — backend, gerçek auth, gerçek LLM, CMS sonraki milestone'a bırakılmıştır. Stack sabit: Next.js 16.2.4 + React 19.2.4 + Tailwind v4 + React Compiler.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Çekirdek Site + Tasarım Sistemi** - Next 16 + i18n + theming + design system + tüm pazarlama şablonları + KVKK + SEO + Lighthouse 95+ ile WordPress yedeği hazır
- [ ] **Phase 2: İçerik & Pazarlama** - Referans logoları, case study, ekip, kariyer, basın, newsletter UI, blog enhancements, indirilebilir PDF'ler, içerik sprint (FAQ + FAQPage)
- [ ] **Phase 3: Mühendislik Araçları** - Çelik ağırlık hesaplayıcı (Vitest-tested) + interaktif profil tabloları + profil karşılaştırma + birim çevirici
- [ ] **Phase 4: Harita & 3D** - MapLibre proje haritası (clustered, theme-aware) + R3F 3D model viewer (poster fallback, viewport-gated)
- [ ] **Phase 5: AI Asistan UI (mock)** - AI Elements chat shell + mock streaming `ReadableStream` + a11y (live-region, reduced-motion, keyboard send)
- [ ] **Phase 6: Canlı Üretim & Medya (mock)** - Video galeri (`<video muted loop playsInline poster>`) + interaktif SVG üretim diyagramı + "yakında" canlı akış kartı
- [ ] **Phase 7: Müşteri Portalı UI (mock)** - `(portal)` route group + mock login/dashboard/belgeler/siparişler + Clerk-shape `useUser`/`useAuth` API + robots disallow

## Phase Details

### Phase 1: Çekirdek Site + Tasarım Sistemi
**Goal**: WordPress sitesinin tüm fonksiyonlarını karşılayan + üzerine premium-endüstriyel tasarım, dark mode, i18n, KVKK, Schema.org, Lighthouse 95+ ekleyen, üzerine 6 farklılaştırıcı faz inşa edilebilecek stabil bir çekirdek + mock data layer + design system.
**Depends on**: Nothing (first phase)
**Requirements**: FND-01, FND-02, FND-03, FND-04, FND-05, FND-06, FND-07, FND-08, FND-09, FND-10, FND-11, FND-12, FND-13, DSN-01, DSN-02, DSN-03, DSN-04, DSN-05, DSN-06, DSN-07, DSN-08, HOME-01, HOME-02, HOME-03, HOME-04, HOME-05, HOME-06, HOME-07, HOME-08, HOME-09, CORP-01, CORP-02, CORP-03, SVC-01, SVC-02, SVC-03, SVC-04, PROJ-01, PROJ-02, PROJ-03, PROJ-04, PROJ-05, TECH-01, TECH-02, TECH-03, BLOG-01, BLOG-02, BLOG-03, CNT-01, CNT-02, CNT-03, CNT-04, CNT-05, CNT-06, CNT-07, CNT-08, LGL-01, LGL-02, LGL-03, LGL-04, LGL-05, PERF-01, PERF-02, PERF-03, A11Y-01, A11Y-02, A11Y-03, SEO-01, SEO-02, SEO-03, SEO-04, SEO-05, TST-01, TST-02, TST-03, TST-04, TST-05, TST-06, TST-07, TST-08, TST-09
**Success Criteria** (what must be TRUE):
  1. Ziyaretçi `/tr` ve `/en` URL'lerinde tüm pazarlama şablonlarını gezebilir (anasayfa, kurumsal, hizmetler hub + 3 alt hizmet, projeler grid + URL-state filter + tab + detay sayfası, 6 sektörel landing, teknik bilgiler hub + 7 interaktif tablo + birim çevirici, blog liste + detay + kategori + etiket, iletişim) ve dark/light tema toggle'ı bir reload sonrası tercihini hatırlar
  2. Ziyaretçi proje grid'ini sektör + yıl aralığı + lokasyon + tonaj ile filtreleyebilir, tab'lar arası geçebilir; filtre durumu URL'de yaşar ve link paylaşılabilir; her proje detay sayfasında tonaj/lokasyon/yıl/sektör/müşteri metadata bloğu, 3-10 fotoğraflık Embla galeri ve ilgili projeler şeridi vardır
  3. Lighthouse mobile + desktop **≥ 95** (Performance, Accessibility, Best Practices, SEO) — anasayfa, projeler grid, proje detay, hizmet detay, blog detay şablonlarında ölçülür; `axe-playwright` her şablonda **0 WCAG 2.2 AA violation** raporlar
  4. KVKK uyumlu cookie banner eşit ağırlık Kabul/Reddet/Tercihler sunar; Maps embed + analytics + harici scriptler **consent öncesi yüklenmez**; `localStorage` consent log persist olur ve footer "Çerez Tercihleri" linki banner'ı yeniden açar; iletişim/RFQ formları KVKK checkbox + honeypot + 2-saniye time-trap ile spam'e hazırdır (gerçek email send yok ama UI success toast + reset çalışır)
  5. `next build` çıktısında her marketing route `○ (Static)` olarak prerender olur (locale negotiation `proxy.ts`'de, layout'larda header okunmuyor); Schema.org JSON-LD `Organization` (root) + `LocalBusiness × 2` (iletişim) + `CreativeWork` (proje detay) + `Service` (hizmet detay) + `BlogPosting` (blog detay) + `BreadcrumbList` doğrulanır; sitemap `alternates.languages` + hreflang ile TR/EN'i birbirine bağlar, robots.txt `/portal/*` disallow eder
  6. Playwright E2E suite şu akışları yeşil geçer: homepage smoke, proje filtreleme + detay, iletişim formu happy path + honeypot trigger, theme toggle reload-persistence, locale toggle TR↔EN, mobile menu drawer, cookie banner Accept + Reject yolları
**Plans**: TBD
**UI hint**: yes

### Phase 2: İçerik & Pazarlama
**Goal**: Çekirdek şablonların üzerine satış-kritik içerik derinliği eklemek — referans gücü, flagship case study, kurumsal güven (ekip/kariyer/basın), lead-gen (newsletter), blog UX iyileştirmesi, indirilebilir PDF'ler ve SEO için kelime/FAQ derinliği.
**Depends on**: Phase 1
**Requirements**: CNTM-01, CNTM-02, CNTM-03, CNTM-04, CNTM-05, CNTM-06, CNTM-07, CNTM-08, CNTM-09
**Success Criteria** (what must be TRUE):
  1. Ziyaretçi anasayfada 8-12 referans logosu şeridini (grayscale → renk hover) görür; flagship case study sayfasında (İstanbul Cargo Energy 820t) challenge / solution / result + ölçülebilir metrik bloğunu okur ve oradan ilgili projeler veya RFQ CTA'sına geçebilir
  2. Ziyaretçi `/kurumsal/ekip`, `/kurumsal/kariyer` (açık pozisyon + başvuru CTA), `/basin` (blog motoru üzerinde basın kategorisi) sayfalarına ana navigasyondan ulaşabilir
  3. Newsletter aboneliği UI mock submit ile başarı durumunu gösterir (localStorage demo, gerçek backend yok); sertifika PDF'leri (ISO 9001/14001/OHSAS 18001) ve kurumsal katalog PDF'i `Cookie banner accept gerektirmeden` indirilebilir; her PDF link'i için fallback ve dosya boyutu açıkça yazılı
  4. Blog yazı detayında uzun yazılar (>1500 kelime) için sticky table-of-contents render olur, scroll ile aktif başlık vurgulanır; her yazı altında ilgili 3 yazı görünür; reading time hesaplanmış olarak görünür
  5. Her v1 proje sayfası **≥ 800 kelime**, her hizmet sayfası **≥ 1000 kelime** + yapılandırılmış FAQ bölümü içerir; her FAQ bloğu Schema.org `FAQPage` JSON-LD ile çıktı verir ve Google Rich Results Test'i geçer
**Plans**: TBD
**UI hint**: yes

### Phase 3: Mühendislik Araçları
**Goal**: Sektörel rakiplerde olmayan mühendislik araçlarıyla siteyi ziyaretçinin "bookmark'lamak isteyeceği" bir referans haline getirmek — kayıp lead'leri RFQ'ya yönlendiren çelik ağırlık hesaplayıcı, profil tabloları ve karşılaştırma aracı.
**Depends on**: Phase 1 (mock profile dataset paylaşılır)
**Requirements**: TOOL-01, TOOL-02, TOOL-03, TOOL-04, TOOL-05, TST-10
**Success Criteria** (what must be TRUE):
  1. Mühendis ziyaretçi `/araclar/agirlik-hesaplayici` sayfasında Combobox'tan profil seçip (HEB, HEA, IPE, IPN, köşebent, kutu profil, boru) uzunluk girince anlık olarak kg sonucu görür; opsiyonel ₺/kg input ile maliyet tahmini hesaplanır; sonuç URL paramlarına yansır ve link paylaşıldığında aynı state geri yüklenir
  2. `lib/calc/weight.ts` saf fonksiyonu Vitest unit testleri (TST-10) ile 7 profilin tamamı için doğru kg/m değerleri döner; uç durumlar (sıfır uzunluk, geçersiz profil) test edilmiştir
  3. `/teknik-bilgiler/[profil]` sayfaları TanStack Table ile arama / filtre / sıralama / **CSV export** sunar; tablonun her satırından "Hesaplayıcıya gönder" CTA aracılığıyla profil pre-filled olarak hesaplayıcıya geçer
  4. Profil karşılaştırma aracı (`/araclar/karsilastir?a=heb-200&b=ipe-240`) iki profili dual-column gösterir, kritik özellikleri (kesit alanı, ağırlık/m, atalet momenti, mukavemet modülü) yan yana karşılaştırır; URL state ile paylaşılabilir; mobilde dikey stack'e dönüşür
  5. Tüm hesaplayıcı / tablo / karşılaştırma sayfaları Phase 1 standartlarını korur: Lighthouse ≥ 95, axe-playwright 0 violation, klavye navigasyonu tam, dark/light tema uyumlu
**Plans**: TBD
**UI hint**: yes

### Phase 4: Harita & 3D
**Goal**: İki "wow" görsel anı eklemek — Türkiye + global proje portföyünü interaktif clustered harita ile, ve örnek bir projeyi gerçek 3D model viewer ile göstermek; her ikisi de bundle bütçesi koruyarak ve a11y kırmadan.
**Depends on**: Phase 1 (proje koordinatları + slug routing), Phase 2 (case study UX paterni)
**Requirements**: MAP-01, MAP-02, MAP-03, MAP-04, MAP-05, MAP-06, MAP-07, MAP-08, 3D-01, 3D-02, 3D-03, 3D-04, 3D-05, 3D-06
**Success Criteria** (what must be TRUE):
  1. Ziyaretçi anasayfada (kompakt) ve `/projeler` sayfasında (tam ekran) **token gerektirmeyen MapLibre + react-map-gl** haritasını görür; pin'ler `supercluster` ile cluster'lanır; cluster'a tıklayınca zoom-in olur; tek pin'e tıklayınca proje detay sayfasına navigate eder; tile style theme'e bağlıdır (dark/light token)
  2. Harita ve 3D viewer her ikisi de Client wrapper içinde `next/dynamic({ ssr: false })` + viewport-gated lazy mount ile yüklenir; harita/3D viewer içermeyen sayfaların First Load JS bundle bütçesi (Phase 1 baseline + max +5KB) bozulmaz
  3. Mobil ziyaretçi haritada touch + pinch zoom kullanabilir, klavye kullanıcısı Tab ile pin'lere odaklanıp Enter ile detay sayfasına gidebilir; her pin meaningful `aria-label` (proje adı + sektör + lokasyon) taşır
  4. 3D viewer `react-three-fiber@^9.6` + `drei@^10` ile DRACO/meshopt sıkıştırılmış GLTF modelini render eder; WebGL desteklenmiyorsa veya kullanıcı `prefers-reduced-motion` etkinse 2D poster fallback + "View in 3D" CTA gösterilir; mobilde antialias kapalı, `dpr=[1, 2]`, `<Environment preset="warehouse">` ile aydınlatma; canvas viewport dışında unmount olur
  5. Phase 4 sayfaları için Lighthouse mobile **≥ 90** (3D viewer içeren sayfa için ≥ 85 kabul); diğer tüm sayfalar Phase 1 baseline'ını (≥ 95) korur — bundle leak yok

**Research recommended**: SUMMARY.md flags this phase for `/gsd-research-phase` deep-dive — confirm free MapLibre tile-style URLs that work under Tailwind v4 dark mode, DRACO decoder hosting strategy, R3F v9 Suspense fallback behavior under React Compiler.
**Plans**: TBD
**UI hint**: yes

### Phase 5: AI Asistan UI (mock)
**Goal**: "Hangi profili kullanmalıyım?" tarzı sorulara yanıt veren bir chat UI mock'u inşa etmek — gerçek LLM yok, ama tasarım dili + UX + a11y + streaming animasyonu sonradan gerçek `ai` SDK takıldığında eksiksiz çalışacak şekilde hazır.
**Depends on**: Phase 1 (typography + button + form primitives), Phase 3 (calculator content for prompt suggestions)
**Requirements**: AI-01, AI-02, AI-03, AI-04, AI-05, AI-06, AI-07, AI-08
**Success Criteria** (what must be TRUE):
  1. Ziyaretçi `/asistan` sayfasında veya floating widget'ta `ai-elements` chat shell'i açar; bir prompt yazıp Enter (Shift+Enter newline) ile gönderdiğinde mock `ReadableStream` üzerinden token-token streaming yanıt görür; kullanıcı `Stop` butonuyla `AbortController` aracılığıyla streaming'i kesebilir
  2. Reduced-motion-friendly typing indicator `prefers-reduced-motion` etkinse animasyonsuz statik state'e döner; her assistant cevabı kullanıldığında screen reader `aria-live` region ile duyurulur
  3. Network/parse hatası durumunda chat error-state UI net bir hata mesajı + "Tekrar dene" CTA gösterir; mock stream contract Vercel AI SDK `streamText` shape'i ile bire bir uyumludur (sonradan gerçek SDK takıldığında UI değişmez)
  4. Chat shell tam klavye-erişilebilir: Tab ile input/send/stop/messages, focus-visible ring, ESC ile widget kapanır; mobilde virtual keyboard chat'i kapatmaz; dark/light tema uyumlu
  5. AI route Phase 1 baseline'ını bozmaz — `/api/chat/route.ts` mock olarak çalışır, AI shell sadece etkileşim olduğunda lazy-load edilir, AI içermeyen sayfaların bundle'ı etkilenmez
**Plans**: TBD
**UI hint**: yes

### Phase 6: Canlı Üretim & Medya (mock)
**Goal**: Fabrika kapasitesini görsel olarak hissettirmek — video galeri (Tuzla + Gebze), üretim sürecinin interaktif SVG diyagramı (5-6 adım), ve "canlı akış" placeholder'ı — hepsi kullanıcı dostu (autoplay-with-sound yok, scroll-jacking yok).
**Depends on**: Phase 1 (motion provider + reduced-motion config), Phase 2 (newsletter CTA paterni)
**Requirements**: MED-01, MED-02, MED-03
**Success Criteria** (what must be TRUE):
  1. Ziyaretçi video galeri sayfasında (`/medya` veya kurumsal alt sayfası) `<video muted loop playsInline poster controls>` ile lazy-loaded fabrika tanıtım videolarını görür; her video bireysel play butonu ile başlatılır (asla autoplay-with-sound değil); captions placeholder track tanımlıdır; videolar fold altında ise IntersectionObserver ile lazy mount olur
  2. Üretim süreci interaktif SVG diyagramı 5-6 adımı (kesim → kaynak → montaj → kalite → boya → sevkiyat) gösterir; her adıma hover/click ile detay açıklama açılır; scroll-driven step reveal `prefers-reduced-motion` etkinse statik full-state olarak render olur (animasyon yok); tam klavye-erişilebilir (Tab + Enter)
  3. "Canlı akış" placeholder kartı net "Yakında — fabrikadan canlı yayın için kayıt ol" mesajı + Phase 2 newsletter CTA'sı ile entegre olur; sahte "live" rozet veya yanıltıcı UI yok
  4. Tüm medya sayfası Lighthouse mobile **≥ 95** korur; videolar `loading="lazy"` (poster), `<source>` srcset ile mobil için ≤ 720p, desktop için ≤ 1080p; toplam medya sayfası ilk yükleme bütçesi (poster'lar + diyagram SVG + chrome) ≤ 500KB
  5. Üretim diyagramı + video galeri tamamen Phase 1 design system token'larını kullanır (renk, tipografi, spacing); dark/light tema uyumlu
**Plans**: TBD
**UI hint**: yes

### Phase 7: Müşteri Portalı UI (mock)
**Goal**: Mevcut/potansiyel müşterilere "biz dijital olarak ileri seviyedeyiz" mesajı veren bir müşteri portalı UI'ı — login, dashboard, sipariş takip, belge yönetimi — gerçek auth/backend yok, ama Clerk-shape API arkasında mock implementation, böylece sonradan gerçek Clerk takıldığında UI değişmez.
**Depends on**: Phase 1 (route group + design system + form primitives), Phase 6 (motion + lazy-load patterns)
**Requirements**: PRT-01, PRT-02, PRT-03, PRT-04, PRT-05, PRT-06, PRT-07, PRT-08, PRT-09, PRT-10
**Success Criteria** (what must be TRUE):
  1. Ziyaretçi `/portal/login` sayfasında kullanıcı adı + şifre formu doldurur; Zod validasyon kuralları (min length, required) çalışır; submit "fake-success" durumunda localStorage'a session yazıp `/portal/dashboard`'a yönlendirir; başarısız kombinasyonda inline hata mesajı + screen-reader announcement gösterir; üst kısımda "Mock UI — gerçek auth değil" dev banner kalıcıdır
  2. `(portal)` route group ayrı chrome (sidebar + topbar) ile render olur; sidebar Dashboard / Belgeler / Siparişler / Çıkış yap link'lerini gösterir; mobilde drawer'a dönüşür; `default.tsx` paralel route slot için tanımlıdır
  3. `/portal/dashboard` mock sipariş + proje takip kartlarını (status badge + ilerleme bar + son güncelleme tarihi), `/portal/belgeler` belge listesi + indirme UI'ını (sertifika, irsaliye, fatura mock), `/portal/siparisler` sipariş listesi + detay sayfasını gösterir; tüm veri `src/mock/portal/` içinden gelir
  4. Dev panel'deki role switcher (admin / müşteri) ile UI farklılıkları doğrulanabilir: admin tüm siparişleri görür, müşteri sadece kendine ait olanları; `<SignedIn>` / `<SignedOut>` / `useUser` / `useAuth` API'leri Clerk-shape ile çalışır (mock implementation `src/lib/auth/`'de)
  5. `robots.txt` `/portal/*` disallow eder, sitemap `/portal/*` route'larını exclude eder, `noindex` meta tüm portal sayfalarında bulunur — mock portal asla Google'a indekslenmez
**Plans**: TBD
**UI hint**: yes

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Çekirdek Site + Tasarım Sistemi | 0/TBD | Not started | - |
| 2. İçerik & Pazarlama | 0/TBD | Not started | - |
| 3. Mühendislik Araçları | 0/TBD | Not started | - |
| 4. Harita & 3D | 0/TBD | Not started | - |
| 5. AI Asistan UI (mock) | 0/TBD | Not started | - |
| 6. Canlı Üretim & Medya (mock) | 0/TBD | Not started | - |
| 7. Müşteri Portalı UI (mock) | 0/TBD | Not started | - |

---

## Coverage Summary

| Phase | Requirement Count |
|-------|-------------------|
| Phase 1: Çekirdek Site + Tasarım Sistemi | 81 |
| Phase 2: İçerik & Pazarlama | 9 |
| Phase 3: Mühendislik Araçları | 6 |
| Phase 4: Harita & 3D | 14 |
| Phase 5: AI Asistan UI (mock) | 8 |
| Phase 6: Canlı Üretim & Medya (mock) | 3 |
| Phase 7: Müşteri Portalı UI (mock) | 10 |
| **Total** | **131** |

**Coverage:** 131/131 v1 requirements mapped (100%) — no orphans, no duplicates.

*Note: REQUIREMENTS.md original "113 total" count was inaccurate. Actual count verified by roadmapper agent: **131 v1 requirements**.*

---
*Roadmap created: 2026-04-25*
