# 🎮 MC Modding Helper — Proje Yol Haritası

> **Proje Amacı:** Kullanıcıların bir web arayüzünden Minecraft modlarını seçerek custom modpack oluşturmasını, optimizasyon hatalarını tespit etmesini, tahmini FPS değerlerini görmesini, akıllı mod önerileri almasını ve tek tıkla CurseForge'a aktarmasını sağlayan bir platform.

---

## Faz 0 — Araştırma & Hazırlık

- [ ] **0.1 — CurseForge API Araştırması**
  - [ ] CurseForge API (Eternal API) dokümantasyonunu incele
  - [ ] API key başvurusu yap ve erişim al
  - [ ] Mod arama, mod detay çekme, modpack yükleme endpointlerini belirle
  - [ ] API rate limit ve kullanım koşullarını araştır
  - [ ] Modpack manifest formatını (manifest.json) detaylıca öğren

- [ ] **0.2 — Mod Uyumluluk & Bağımlılık Araştırması**
  - [ ] Forge / Fabric / NeoForge / Quilt loader farklarını araştır
  - [ ] Minecraft sürüm uyumluluk matrisini oluştur
  - [ ] Mod bağımlılık (dependency) zinciri mantığını incele
  - [ ] Bilinen mod çakışmalarını (incompatibility) listele ve kaynak bul
  - [ ] Performans modlarını kategorize et (Sodium, OptiFine, Lithium vb.)

- [ ] **0.3 — FPS ve Performans Tahmin Modeli Araştırması**
  - [ ] Farklı mod kombinasyonlarının performans etkilerini araştır
  - [ ] Mod türlerine göre ağırlık/maliyet puanlama sistemi tasarla
  - [ ] Donanım profilleri için referans FPS verisi topla (düşük/orta/yüksek PC)
  - [ ] Shader, texture pack ve mod sayısına göre FPS düşüş formülleri araştır

- [ ] **0.4 — Teknoloji Stack'i Kararı**
  - [ ] Frontend framework seçimi (Next.js / Vite + React)
  - [ ] Backend framework seçimi (Node.js + Express / Fastify / NestJS)
  - [ ] Veritabanı seçimi (PostgreSQL / MongoDB)
  - [ ] Cache katmanı (Redis)
  - [ ] Hosting & deployment stratejisi (Vercel, Railway, VPS vb.)

---

## Faz 1 — Backend Altyapısı

- [ ] **1.1 — Proje İskeletini Oluştur**
  - [ ] Backend projesini initialize et (package.json, tsconfig, vb.)
  - [ ] Klasör yapısını oluştur (`/src`, `/routes`, `/services`, `/models`, `/utils`)
  - [ ] Linting ve formatting ayarları (ESLint, Prettier)
  - [ ] Environment variables yapısı (.env, config yönetimi)
  - [ ] Logger kurulumu (Winston / Pino)

- [ ] **1.2 — Veritabanı Tasarımı & Modelleri**
  - [ ] Veritabanı şemasını tasarla:
    - [ ] `mods` tablosu (id, curseforge_id, name, slug, description, category, loader, mc_versions, performance_cost, download_count, thumbnail)
    - [ ] `mod_dependencies` tablosu (mod_id → depends_on_mod_id, dependency_type)
    - [ ] `mod_incompatibilities` tablosu (mod_a_id ↔ mod_b_id, reason)
    - [ ] `mod_tags` tablosu (mod_id, tag: optimization / content / visual / utility vb.)
    - [ ] `modpacks` tablosu (id, user_id, name, mc_version, loader, created_at)
    - [ ] `modpack_mods` tablosu (modpack_id, mod_id)
    - [ ] `users` tablosu (id, username, email, curseforge_token, created_at)
  - [ ] Migration dosyalarını oluştur
  - [ ] Seed data hazırla (popüler modların ilk verileri)

- [ ] **1.3 — CurseForge API Entegrasyonu (Servis Katmanı)**
  - [ ] CurseForge API client servisi yaz
  - [ ] Mod arama fonksiyonu (`searchMods`)
  - [ ] Mod detay çekme fonksiyonu (`getModDetails`)
  - [ ] Mod dosya/sürüm listeleme fonksiyonu (`getModFiles`)
  - [ ] Kategorilere göre mod listeleme
  - [ ] API yanıtlarını cache'leme (Redis ile)
  - [ ] Rate limiting ve hata yönetimi

- [ ] **1.4 — Mod Kataloğu Senkronizasyonu**
  - [ ] CurseForge'dan popüler modları periyodik çekme (cron job)
  - [ ] Mod verilerini veritabanına kaydetme/güncelleme
  - [ ] Mod thumbnail ve asset'leri kaydetme
  - [ ] Yeni sürüm kontrolü ve güncelleme mekanizması

---

## Faz 2 — Akıllı Modpack Motoru

- [ ] **2.1 — Uyumluluk Kontrol Sistemi**
  - [ ] Minecraft sürüm uyumluluk kontrolü
  - [ ] Mod loader (Forge/Fabric/NeoForge) uyumluluk kontrolü
  - [ ] Mod-mod çakışma kontrolü (incompatibility database)
  - [ ] Bağımlılık otomatik çözümleme (dependency resolution)
  - [ ] Eksik bağımlılıkları otomatik ekleme önerisi
  - [ ] Çakışan modlar için kullanıcıya uyarı sistemi

- [ ] **2.2 — FPS Tahmin & Performans Analiz Motoru**
  - [ ] Her mod için performans maliyeti (performance cost) puanlama sistemi
  - [ ] Donanım profilleri tanımlama (düşük / orta / yüksek / ultra)
  - [ ] Modpack toplam performans skoru hesaplama algoritması:
    - [ ] Base FPS değeri (vanilla, donanıma göre)
    - [ ] Her modun FPS etkisi (çarpan veya çıkarma)
    - [ ] Shader ve texture pack etkisi
    - [ ] Optimizasyon modlarının pozitif etkisi
  - [ ] Tahmini FPS gösterimi (min / ortalama / maks)
  - [ ] Performans uyarıları (kırmızı / sarı / yeşil gösterge)

- [ ] **2.3 — Optimizasyon Hata Tespit Sistemi**
  - [ ] Yaygın optimizasyon hatalarını tanımla:
    - [ ] OptiFine + Sodium birlikte kullanımı
    - [ ] Aynı işi yapan birden fazla mod (duplicate functionality)
    - [ ] Client-side / Server-side mod karışıklığı
    - [ ] Aşırı yüksek performans maliyetli mod kombinasyonları
    - [ ] Eksik performans modları (Sodium/Lithium/Starlight olmadan ağır modpack)
  - [ ] Hata tespit kuralları motoru (rule engine)
  - [ ] Kullanıcıya hata açıklaması ve çözüm önerisi sunma
  - [ ] Otomatik düzeltme önerileri (bir tıkla düzelt)

- [ ] **2.4 — Akıllı Mod Öneri Sistemi**
  - [ ] Seçilen modlara göre tamamlayıcı mod önerme:
    - [ ] Eğer büyük content mod varsa → gerekli kütüphane modları öner
    - [ ] Eğer ağır modlar varsa → optimizasyon modları öner
    - [ ] Eğer belirli bir tema varsa (technology, magic vb.) → o temanın popüler modlarını öner
  - [ ] "Bu modla sık kullanılan modlar" verisi (CurseForge verisinden)
  - [ ] Kategori bazlı öneriler
  - [ ] Popülerlik ve uyumluluk bazlı sıralama
  - [ ] Kullanıcının daha önce kullandığı modlara göre kişiselleştirme

---

## Faz 3 — CurseForge Modpack Aktarım Sistemi

- [ ] **3.1 — Modpack Manifest Oluşturucu**
  - [ ] CurseForge modpack formatını oluştur (`manifest.json`):
    - [ ] Minecraft version
    - [ ] Mod loader & version (Forge/Fabric)
    - [ ] Mod listesi (projectID, fileID)
    - [ ] Overrides klasörü (config dosyaları)
  - [ ] Modpack zip dosyası oluşturma
  - [ ] Modpack adı, açıklama ve versiyon bilgisi ekleme
  - [ ] İsteğe bağlı config dosyalarını dahil etme

- [ ] **3.2 — CurseForge Yükleme Entegrasyonu**
  - [ ] CurseForge OAuth2 kimlik doğrulama akışı
  - [ ] Kullanıcının CurseForge hesabını bağlama
  - [ ] Modpack'i CurseForge'a yükleme API çağrısı
  - [ ] Yükleme durumu takibi (progress tracking)
  - [ ] Başarılı yükleme sonrası CurseForge link paylaşımı
  - [ ] Hata durumları yönetimi ve kullanıcıya geri bildirim

- [ ] **3.3 — Alternatif Dışa Aktarım Seçenekleri**
  - [ ] Modpack'i `.zip` olarak indirme
  - [ ] MultiMC / Prism Launcher uyumlu export
  - [ ] Modrinth uyumlu export (mrpack formatı)
  - [ ] Modpack paylaşım linki oluşturma

---

## Faz 4 — Frontend & Web Arayüzü

- [ ] **4.1 — Proje Kurulumu & Tasarım Sistemi**
  - [ ] Frontend projesini initialize et (Vite + React / Next.js)
  - [ ] Tasarım sistemi oluştur (renk paleti, tipografi, spacing)
  - [ ] Minecraft temalı UI komponentleri tasarla
  - [ ] Responsive layout altyapısı
  - [ ] Dark mode / Light mode desteği

- [ ] **4.2 — Ana Sayfa & Mod Kataloğu**
  - [ ] Hero bölümü (projenin tanıtımı, CTA butonları)
  - [ ] Mod arama çubuğu (autocomplete, filtreleme)
  - [ ] Mod kartları (thumbnail, isim, açıklama, performans puanı, popülerlik)
  - [ ] Kategori filtreleri (Technology, Magic, Adventure, Optimization vb.)
  - [ ] Minecraft sürüm filtresi
  - [ ] Mod loader filtresi (Forge / Fabric / NeoForge)
  - [ ] Sıralama seçenekleri (popülerlik, isim, performans maliyeti)
  - [ ] Infinite scroll veya pagination

- [ ] **4.3 — Modpack Oluşturucu Sayfası**
  - [ ] Drag & Drop veya tıkla-ekle mod seçim arayüzü
  - [ ] Seçilen modlar listesi (sidebar veya alt panel)
  - [ ] Gerçek zamanlı uyumluluk kontrolü gösterimi:
    - [ ] ✅ Uyumlu modlar yeşil
    - [ ] ⚠️ Uyarı gerektiren modlar sarı
    - [ ] ❌ Çakışan modlar kırmızı
  - [ ] FPS tahmin göstergesi (gauge / progress bar)
  - [ ] Optimizasyon hataları paneli (uyarılar ve öneriler)
  - [ ] Mod öneri paneli ("Şunları da eklemek ister misiniz?")
  - [ ] Mod detay modal'ı (açıklama, ekran görüntüleri, bağımlılıklar)
  - [ ] Modpack adı ve açıklama girişi

- [ ] **4.4 — Modpack Özet & Aktarım Sayfası**
  - [ ] Modpack özet ekranı (tüm modlar, toplam boyut, FPS tahmini)
  - [ ] Performans raporu (detaylı analiz)
  - [ ] CurseForge'a aktar butonu (tek tık)
  - [ ] Alternatif export seçenekleri
  - [ ] Yükleme progress bar'ı
  - [ ] Başarı / Hata ekranı

- [ ] **4.5 — Kullanıcı Hesap Sistemi**
  - [ ] Kayıt / Giriş sayfaları
  - [ ] CurseForge hesap bağlama
  - [ ] Kaydedilen modpack'ler listesi
  - [ ] Modpack geçmişi
  - [ ] Profil ayarları (donanım profili seçimi)

---

## Faz 5 — Kullanıcı Deneyimi & Gelişmiş Özellikler

- [ ] **5.1 — Donanım Profil Sistemi**
  - [ ] Kullanıcının PC özelliklerini girmesi (RAM, GPU, CPU)
  - [ ] Hazır donanım profilleri (düşük / orta / yüksek / ultra)
  - [ ] Donanıma göre FPS tahmininin kişiselleştirilmesi
  - [ ] "Bu modpack senin PC'n için uygun mu?" kontrolü

- [ ] **5.2 — Modpack Şablonları**
  - [ ] Hazır modpack şablonları (Performance Pack, Kitchen Sink, RPG, Technical vb.)
  - [ ] Topluluk tarafından oluşturulmuş popüler paketler
  - [ ] Şablonu başlangıç noktası olarak seçip üzerine ekleme

- [ ] **5.3 — Sosyal Özellikler**
  - [ ] Modpack paylaşma (public link)
  - [ ] Modpack beğenme / puanlama
  - [ ] Yorum sistemi
  - [ ] "En popüler modpack'ler" listesi

- [ ] **5.4 — Bildirim & Güncelleme Sistemi**
  - [ ] Modpack'teki modlar güncellendiğinde bildirim
  - [ ] Yeni uyumlu mod çıktığında öneri bildirimi
  - [ ] E-posta ve web push notification desteği

---

## Faz 6 — Test & Kalite Güvence

- [ ] **6.1 — Backend Testleri**
  - [ ] Unit testler (servis katmanı, uyumluluk kontrolü, FPS motoru)
  - [ ] Integration testler (API endpointleri, veritabanı işlemleri)
  - [ ] CurseForge API mock testleri
  - [ ] Performans/yük testleri (k6 / Artillery)

- [ ] **6.2 — Frontend Testleri**
  - [ ] Component testleri (React Testing Library / Vitest)
  - [ ] E2E testler (Playwright / Cypress)
  - [ ] Cross-browser test (Chrome, Firefox, Edge)
  - [ ] Responsive tasarım testleri (mobil, tablet, masaüstü)

- [ ] **6.3 — Kullanıcı Testleri**
  - [ ] Beta kullanıcı grubu oluştur
  - [ ] Kullanıcı geri bildirimi toplama
  - [ ] Hata raporlama mekanizması
  - [ ] UX iyileştirmeleri

---

## Faz 7 — Deployment & Lansman

- [ ] **7.1 — CI/CD Pipeline**
  - [ ] GitHub Actions / GitLab CI kurulumu
  - [ ] Otomatik test çalıştırma
  - [ ] Otomatik deployment (staging & production)
  - [ ] Docker containerization

- [ ] **7.2 — Production Ortamı**
  - [ ] Domain ve SSL sertifikası
  - [ ] CDN kurulumu (CloudFlare)
  - [ ] Monitoring & Alerting (Sentry, Uptime Robot)
  - [ ] Veritabanı backup stratejisi
  - [ ] Log yönetimi

- [ ] **7.3 — Lansman**
  - [ ] Landing page hazırla
  - [ ] Sosyal medya duyurusu
  - [ ] Minecraft topluluk forumlarında paylaşım (Reddit, Discord)
  - [ ] CurseForge partner başvurusu
  - [ ] SEO optimizasyonu

---

## 📊 İlerleme Özeti

| Faz | Durum | İlerleme |
|-----|-------|----------|
| Faz 0 — Araştırma & Hazırlık | ⬜ Başlanmadı | 0% |
| Faz 1 — Backend Altyapısı | ⬜ Başlanmadı | 0% |
| Faz 2 — Akıllı Modpack Motoru | ⬜ Başlanmadı | 0% |
| Faz 3 — CurseForge Aktarım | ⬜ Başlanmadı | 0% |
| Faz 4 — Frontend & Web Arayüzü | ⬜ Başlanmadı | 0% |
| Faz 5 — Gelişmiş Özellikler | ⬜ Başlanmadı | 0% |
| Faz 6 — Test & Kalite Güvence | ⬜ Başlanmadı | 0% |
| Faz 7 — Deployment & Lansman | ⬜ Başlanmadı | 0% |

---

> 📝 **Not:** Bu dosya projenin tüm yaşam döngüsü boyunca güncellenecektir. Her görev tamamlandığında `[ ]` → `[x]` olarak işaretlenecektir.
