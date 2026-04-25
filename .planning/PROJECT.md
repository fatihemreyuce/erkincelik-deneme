# Erkin Çelik — Modern Web Yenileme

## What This Is

erkincelik.com'un (2008'den beri çelik konstrüksiyon imalat firması — Tuzla merkez, Gebze fabrika, ISO 9001/14001/OHSAS 18001 sertifikalı, yıllık 8.400 ton kapasite) Next.js 16 + React 19 + Tailwind v4 üzerinde **endüstriyel-premium** estetikle baştan tasarlanmış modern web sitesi. Hedef: B2B müşterilerine (havalimanı, köprü, hangar, sanayi tesisi sahipleri) güven veren, mevcut WordPress sitesinden ayrışan, performans + erişilebilirlik + ayırıcı UI özellikleriyle (çelik ağırlık hesaplayıcı, proje haritası, AI asistan, müşteri portalı) sektörün önüne geçen bir kurumsal site.

## Core Value

**Mevcut WordPress sitesinin yerini alabilecek, B2B çelik konstrüksiyon müşterisine "bu firma profesyonel" hissi veren ve teklif/iletişim formuna kolay yönlendiren modern bir vitrin.** Tüm modern özellikler (AI, portal, 3D) bu çekirdek değeri besler — onun yerine geçmez.

## Requirements

### Validated

(None yet — ship to validate)

### Active

#### Çekirdek Site (Phase 1+2)
- [ ] Anasayfa: hero, üretim süreci, kapasite, hizmetler, öne çıkan projeler, iletişim CTA
- [ ] Kurumsal sayfa: hakkımızda, misyon, vizyon, değerler, sertifikalar
- [ ] Hizmetlerimiz: çelik konstrüksiyon ana hizmeti + Projelendirme/İmalat/Montaj alt hizmetleri
- [ ] Projeler: 50+ proje grid'i + tab (Tamamlanan/Devam Eden) + **proje detay sayfası** (tonaj, lokasyon, galeri)
- [ ] Proje filtreleme/arama (kategori, yıl, lokasyon, tonaj)
- [ ] Teknik Bilgiler: HEB/HEA/IPE/IPN/köşebent/kutu profil/boru ağırlık tabloları
- [ ] Blog: liste + yazı detay + kategori/etiket
- [ ] İletişim: 2 ofis adresi, Google Maps embed, çalışma saatleri, RFQ + genel form, KVKK + reCAPTCHA
- [ ] TR/EN dil altyapısı (i18n switcher, EN içerik placeholder)
- [ ] Dark mode / light mode toggle
- [ ] Tasarım sistemi: premium tipografi, endüstriyel-premium renk paleti (koyu zemin + metalik vurgu), shadcn/ui component library
- [ ] Mevcut logoyu kullan, palet yeni
- [ ] Mikro-etkileşim + scroll-driven animasyon
- [ ] Erişilebilirlik (WCAG AA): klavye navigasyonu, ARIA, kontrast, alt text
- [ ] Performans: AVIF/WebP, lazy load, hedef Lighthouse 95+
- [ ] SEO: Schema.org (Organization, LocalBusiness, Project), Open Graph, sitemap, robots.txt
- [ ] Cookie consent + KVKK metni
- [ ] Form spam koruması (honeypot/BotID)
- [ ] 404 + error boundary sayfaları
- [ ] Mock data layer: tüm proje/blog/sertifika TypeScript ile sahte veri
- [ ] Smoke + e2e testleri (Playwright) ana akışlar için

#### İçerik & Pazarlama (Phase 3)
- [ ] Referans logoları
- [ ] Case study (öne çıkan proje detayı)
- [ ] Ekip / yönetim tanıtımı
- [ ] Kariyer sayfası
- [ ] Basın / haber arşivi
- [ ] Newsletter aboneliği UI
- [ ] Blog kategori/etiket/okuma süresi/ilgili yazılar
- [ ] İndirilebilir PDF sertifika önizleme
- [ ] Kurumsal katalog indirme

#### Mühendislik Araçları (Phase 4)
- [ ] Çelik ağırlık hesaplayıcı (profil seçimi + uzunluk → kg)
- [ ] İnteraktif teknik tablolar: arama, filtre, sıralama, birim çevirici
- [ ] Profil karşılaştırma aracı

#### Proje Haritası & 3D (Phase 5)
- [ ] Türkiye + dünya interaktif haritası, proje pin'leri
- [ ] 3D model viewer (örnek proje için, mock GLTF)

#### AI Asistan UI (Phase 6 — mock)
- [ ] Sohbet UI: "hangi profili kullanmalıyım" sorularına yanıt UI'ı
- [ ] Mock streaming yanıt simülasyonu (gerçek LLM sonra eklenecek)

#### Canlı Üretim & Medya (Phase 7 — mock)
- [ ] Video galeri / fabrika tanıtımı
- [ ] Üretim süreci interaktif şeması
- [ ] Mock "canlı akış" placeholder'ı

#### Müşteri Portalı UI (Phase 8 — mock)
- [ ] Login UI (Clerk veya benzeri için tasarım, gerçek auth sonra)
- [ ] Dashboard: sipariş/proje takip kartları
- [ ] Belge listesi + indirme UI
- [ ] Roller (admin / müşteri) için ayrı görünüm

### Out of Scope

- **Gerçek LLM/AI Gateway entegrasyonu** — Phase 6 mock UI; gerçek backend sonraki milestone
- **Gerçek auth (Clerk/Auth0) entegrasyonu** — Phase 8 mock UI; gerçek auth sonraki milestone
- **Gerçek müşteri portalı backend (sipariş DB, belge upload)** — sonraki milestone
- **Headless CMS entegrasyonu (Sanity/Payload)** — Şimdilik mock data; CMS sonraki milestone
- **Mevcut WordPress içerik migration'ı** — Sadece proje görselleri scrape; metinler sıfırdan
- **Vercel deploy/CI** — Hosting kararı henüz verilmedi; local dev odağı
- **Gerçek e-posta/iletişim formu backend'i** — Form UI yapılır, gerçek gönderim sonra
- **Mobil uygulama** — Web önceliği
- **Gerçek 3D modelleme servisi** — Sample GLTF; gerçek proje 3D'leri sonra

## Context

**Mevcut site (kaynak):** erkincelik.com — WordPress + Vesna Bilgi Teknolojileri custom theme. Eski hero slider, statik teknik tablolar, proje detay sayfası yok, filtreleme yok, dil seçeneği yok, dark mode yok, Schema.org yok, performans optimize değil.

**Sektör:** Çelik konstrüksiyon B2B. Müşteri tipi: havalimanı (İstanbul Cargo Energy 820t), AVM, otel, hangar, köprü, fabrika sahipleri. Karar süreci uzun, güvenilirlik birinci kriter. Rakip B2B çelik siteleri çoğunlukla jenerik kurumsal — premium endüstriyel estetik ayırıcı olur.

**Teknik ortam:**
- Next.js **16.2.4** — App Router, React Compiler aktif (`reactCompiler: true`)
- React **19.2.4**
- Tailwind CSS **v4** (PostCSS plugin)
- TypeScript strict mode
- ⚠️ **AGENTS.md uyarısı: "Bu senin bildiğin Next.js değil"** — Next.js 16 breaking change'leri var, kod yazmadan önce `node_modules/next/dist/docs/` okunmalı

**Tasarım yönü:** Endüstriyel-Premium — koyu zemin, metalik vurgular (turuncu/bakır accent), büyük tipografi, sinema gibi görsel anlatım. Mevcut sitenin mavi-gri jenerik kurumsal hissinden ayrışır.

**İçerik stratejisi:** Sadece **proje görselleri** mevcut siteden scrape edilecek. Tüm metin, slogan, hizmet açıklamaları **sıfırdan**, modern copywriting ile. Tüm dinamik veri **mock data layer** (TypeScript) — CMS sonra.

**Çalışma modu:** Yavaş, kalite öncelikli. Deadline yok. Her faz GSD workflow ile (discuss → plan → execute → verify, atomic commits).

## Constraints

- **Tech stack:** Next.js 16 + React 19 + Tailwind v4 — Sabit. — Çünkü zaten kurulu ve modern.
- **Next.js sürümü:** AGENTS.md uyarısı aktif — Her sayfa/route/middleware yazımında ilgili Next 16 docs (`node_modules/next/dist/docs/`) okunmalı. — Önceki sürümlerden breaking değişiklikler var.
- **Backend:** Yok (bu milestone). Tüm dinamik veri mock. — Saf frontend tasarım odağı.
- **i18n:** TR primary, EN scaffold (içerik sonra). — Global müşteri için altyapı hazır olmalı.
- **Performans hedefi:** Lighthouse 95+ (mobile + desktop). — B2B müşteri ilk izlenim için kritik.
- **Erişilebilirlik:** WCAG AA. — Kurumsal müşteri (özellikle kamu/havalimanı) için gereklilik.
- **Test:** Smoke + e2e (Playwright). Unit test minimal. — Tasarım odaklı proje, UI davranışı kritik.
- **Tarayıcı desteği:** Modern (son 2 sürüm Chrome/Edge/Safari/Firefox). IE yok.
- **KVKK uyumu:** Cookie banner + form onayı + KVKK metni zorunlu. — Türkiye yasal gereklilik.
- **Hosting:** Henüz karar verilmedi. — Vercel öneriliyor ama erken karar değil.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Mevcut Next.js 16 scaffold üzerine devam et | Zaten kurulu, modern stack | — Pending |
| Frontend-only milestone, mock data layer | Kullanıcı sadece tasarım istiyor; backend sonra | — Pending |
| Endüstriyel-Premium tasarım yönü | Sektör B2B sitelerinden ayrışma, premium algı | — Pending |
| Sadece proje görselleri mevcut siteden scrape, metinler sıfırdan | Tasarım yenileme + premium copy fırsatı | — Pending |
| Mevcut logoyu kullan, palet yenile | Marka tanınırlığı korunur, kimlik tazelenir | — Pending |
| TR primary + EN i18n scaffold (çeviri yok) | "Worldwide" iddiası için altyapı; çeviri içerik sonra | — Pending |
| Faz 1+2 birleşik (çekirdek + tasarım sistemi) | Tasarım sistemi olmadan çekirdek yapmak yarım iş | — Pending |
| AI/Portal/3D fazları UI mock olarak yapılacak | Kullanıcı backend istemiyor; tasarım dili tüm yüzeyleri kapsasın | — Pending |
| Hosting kararı ertelendi | Erken bağlanmak gerek yok, local dev önce | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-25 after initialization*
