# Faz 2 — Akıllı Modpack Motoru Araştırması

> Bu döküman, Faz 2'deki dört ana motorun (Uyumluluk, FPS Tahmin, Hata Tespit, Mod Öneri) teknik tasarımı için web araştırması sonuçlarını derler.

---

## 1. Uyumluluk Kontrol Sistemi

### 1.1 Kontrol Katmanları (Öncelik Sırasıyla)

| # | Katman | Açıklama | Kaynak |
|---|--------|----------|--------|
| 1 | **MC Sürüm** | Mod'un `gameVersions` alanı kullanıcının seçtiği MC sürümünü içermeli | CurseForge API |
| 2 | **Loader** | Mod'un `modLoaders` alanı (Forge/Fabric/NeoForge) modpack'in loader'ı ile eşleşmeli | CurseForge API |
| 3 | **Bağımlılıklar** | CurseForge `relationType` ile zorunlu/opsiyonel bağımlılıklar çözümlenmeli | CurseForge API |
| 4 | **Çakışmalar** | CurseForge `relationType: 5 (Incompatible)` + kendi kural veritabanımız | Hibrit |
| 5 | **Duplicate Fonksiyon** | Aynı işi yapan modlar (rendering, lighting, vb.) tespit edilmeli | Kendi kurallarımız |

### 1.2 CurseForge API Dependency Yapısı

Her mod dosyasının `dependencies` dizisi şu `relationType` değerlerini içerir:

```
1 = EmbeddedLibrary    → Modun içine gömülü, ayrıca indirmeye gerek yok
2 = OptionalDependency → İsteğe bağlı, olmasa da çalışır
3 = RequiredDependency → ZORUNLU, olmadan çalışmaz
4 = Tool               → Geliştirme aracı, son kullanıcıyı ilgilendirmez
5 = Incompatible       → Bu modla ÇAKIŞIR, birlikte kullanılamaz
6 = Include            → Modpack'e dahil edilmiş
```

### 1.3 Dependency Resolution Algoritması

```
kullanıcı_mod_ekle(mod):
  1. MC sürüm kontrolü → uyumsuzsa ENGELLE
  2. Loader kontrolü → uyumsuzsa ENGELLE  
  3. dependencies[] çek (CurseForge API)
  4. for dep in dependencies:
       if dep.relationType == 3 (Required):
         → dep modpack'te var mı?
           → Yoksa: "X modu gerekli, eklensin mi?" önerisi
           → Recursive: dep'in kendi bağımlılıklarını da çöz
       if dep.relationType == 5 (Incompatible):
         → Modpack'te çakışan mod var mı?
           → Varsa: 🔴 UYARI "X ile Y çakışır"
       if dep.relationType == 2 (Optional):
         → "X modu opsiyonel, eklemek ister misin?" önerisi
  5. Kendi kural DB'mizi kontrol et (duplicate, bilinen çakışmalar)
```

### 1.4 Çapraz-Loader Uyumluluk Notları

- **Sinytra Connector:** Bazı Fabric modlarını Forge üzerinde çalıştırır (deneysel)
- **Forgified Fabric API:** Fabric API'yi Forge'a taşır
- **Projemiz için karar:** Bu araçları şimdilik DESTEKLEMİYORUZ. Bir modpack = bir loader kuralı basit ve güvenli.

---

## 2. FPS Tahmin & Performans Analiz Motoru

### 2.1 Neden Kesin Formül Yok?

Web araştırması sonucu: **Toplulukta standart bir FPS formülü veya puanlama sistemi bulunmuyor.** Bunun sebepleri:
- Performans etkisi donanıma bağımlı (GPU, CPU, RAM)
- Mod kombinasyonları sinerjik/antagonistik etkileşimler yaratır
- Aynı mod farklı shader/texture pack ile farklı performans gösterir

**Bu nedenle kendi puanlama sistemimizi oluşturmamız gerekiyor.**

### 2.2 Önerilen Performans Puanlama Sistemi

#### Mod Kategorileri ve Performans Etki Ağırlıkları

| Kategori | Açıklama | Tipik FPS Etkisi | Puan Aralığı |
|----------|----------|------------------|-------------|
| **rendering** | GPU yoğun, görsellik modları | Yüksek negatif | 5-10 |
| **worldgen** | Dünya oluşturma değiştiren | Orta negatif (yüklenme) | 3-7 |
| **content_large** | Büyük içerik modları (Create, Mekanism) | Orta negatif | 4-8 |
| **content_small** | Küçük içerik modları | Düşük negatif | 1-3 |
| **entity_heavy** | Çok sayıda mob/entity ekleyen | Orta-yüksek negatif | 4-8 |
| **library** | Kütüphane/API modları | Minimal etki | 0-1 |
| **ui_only** | Sadece UI değiştiren (JEI/REI, minimap) | Düşük negatif | 1-2 |
| **optimization** | Performans optimizasyon modları | **POZİTİF** | -3 ile -8 |
| **shader** | Shader modları (Iris, Oculus) | Yüksek negatif (shader'a bağlı) | 8-15 |

> Negatif puanlar = FPS artışı (optimizasyon modları)

#### Donanım Profilleri (Referans Sistemler)

Kullanıcıya FPS tahmini gösterirken **iki gerçek fiyat-performans sistemi** üzerinden karşılaştırma yapıyoruz:

| Profil | GPU | CPU (Tipik) | RAM | Vanilla FPS (1080p, 16 chunk) |
|--------|-----|-------------|-----|-------------------------------|
| **Sistem A** | RTX 4060 (8GB) | Ryzen 5 5600 / i5-12400 | 16 GB | ~400-700 FPS |
| **Sistem B** | RTX 5060 (8GB) | Ryzen 7 7800X3D / i7-14700 | 16 GB | ~600-900 FPS |

> Kaynak: YouTube benchmark videoları (2024-2025). MC Java Edition, 1080p, Fancy ayarlar, 16 chunk render distance.

#### FPS Tahmin Formülü

```
// Donanım bazlı vanilla FPS değerleri (MC Java, 1080p, 16 chunk)
BASE_FPS = {
  rtx4060: 500,   // RTX 4060 + Ryzen 5 5600 / i5-12400, 16GB RAM
  rtx5060: 700    // RTX 5060 + Ryzen 7 7800X3D / i7-14700, 16GB RAM
}

// Hesaplama
total_cost = SUM(her_mod.performance_cost)
optimization_bonus = SUM(optimizasyon_modları.negative_cost)
net_cost = total_cost + optimization_bonus  // optimization_bonus negatif

// FPS etki çarpanı (her puan ~%3-5 FPS düşüşü)
impact_factor = max(0.1, 1 - (net_cost * 0.035))

estimated_fps = BASE_FPS[donanım_profili] * impact_factor

// Sonuç aralığı
min_fps = estimated_fps * 0.6   // Yoğun alanlarda (mob farm, büyük yapılar)
avg_fps = estimated_fps          // Normal oynanışta ortalama
max_fps = estimated_fps * 1.3   // Boş alanlarda (düz arazi)
```

#### Benchmark Referans Verileri (Gerçek Veriye Dayalı)

| Senaryo | RTX 4060 | RTX 5060 |
|---------|----------|----------|
| **Vanilla (mod yok)** | ~500 FPS | ~700 FPS |
| **+ Sodium** | ~800 FPS | ~1100 FPS |
| **+ Sodium + Lithium + Starlight** | ~900 FPS | ~1200 FPS |
| **+ 30 mod (optimizasyon ile)** | ~200-350 FPS | ~300-450 FPS |
| **+ 30 mod (optimizasyon yok)** | ~80-150 FPS | ~120-200 FPS |
| **+ 100 mod (optimizasyon ile)** | ~80-150 FPS | ~120-200 FPS |
| **+ 100 mod (optimizasyon yok)** | ~25-60 FPS | ~40-80 FPS |
| **+ Shader (BSL, hafif)** | ~150-200 FPS | ~200-300 FPS |
| **+ Shader (SEUS PTGI, ağır)** | ~30-50 FPS | ~45-70 FPS |

> ⚠️ Bu değerler YouTube benchmark'larından derlenmiş **yaklaşık** değerlerdir. Gerçek FPS; MC sürümüne, render distance'a, dünya karmaşıklığına ve mod kombinasyonuna göre değişebilir.

#### Örnek: Kullanıcıya Nasıl Gösterilir?

```
┌─────────────────────────────────────────────┐
│  📊 Tahmini FPS (RTX 4060 bazlı)            │
│                                             │
│  Modpack: 25 mod | Shader: BSL              │
│                                             │
│  ██████████████░░░░░░  ~140 FPS             │
│  Min: ~85  |  Ort: ~140  |  Maks: ~180      │
│                                             │
│  Durum: 🟢 İyi performans                    │
│                                             │
│  💡 FerriteCore eklerseniz: ~160 FPS (+14%)  │
└─────────────────────────────────────────────┘
```

### 2.3 Performans Seviye Göstergeleri

| Seviye | FPS Aralığı | Renk | Emoji |
|--------|------------|------|-------|
| Mükemmel | 120+ | 🟢 Yeşil | ✨ |
| İyi | 60-120 | 🟢 Yeşil | ✅ |
| Kabul Edilebilir | 30-60 | 🟡 Sarı | ⚠️ |
| Düşük | 15-30 | 🟠 Turuncu | ⚠️ |
| Oynanamaz | <15 | 🔴 Kırmızı | ❌ |

---

## 3. Optimizasyon Hata Tespit Sistemi

### 3.1 Kural Motoru (Rule Engine) Tasarımı

Her kural bir JSON/obje olarak tanımlanır:

```json
{
  "id": "OPT_001",
  "severity": "error",
  "name": "OptiFine + Sodium Çakışması",
  "condition": {
    "type": "both_present",
    "mods": ["optifine", "sodium"]
  },
  "message": "OptiFine ve Sodium birlikte kullanılamaz. Her ikisi de rendering motorunu değiştirir.",
  "fix": {
    "action": "remove_one",
    "recommendation": "Sodium'u tercih edin (daha iyi performans ve uyumluluk)"
  }
}
```

### 3.2 Tespit Edilecek Hata/Uyarı Kuralları

#### 🔴 Hatalar (Error — kesinlikle düzeltilmeli)

| ID | Kural | Koşul |
|----|-------|-------|
| ERR_001 | OptiFine + Sodium | İkisi birlikte varsa |
| ERR_002 | OptiFine + Iris | İkisi birlikte varsa |
| ERR_003 | Starlight + Phosphor | İkisi birlikte varsa |
| ERR_004 | Farklı loader modları | Forge modu Fabric modpack'te veya tersi |
| ERR_005 | MC sürüm uyumsuzluğu | Mod desteklenen listede değilse |

#### 🟡 Uyarılar (Warning — düzeltilmesi önerilir)

| ID | Kural | Koşul |
|----|-------|-------|
| WRN_001 | Duplicate rendering | Sodium + Embeddium (aynı iş, farklı loader) |
| WRN_002 | Duplicate shader | Iris + Oculus (aynı iş, farklı loader) |
| WRN_003 | Optimizasyon eksikliği | 10+ mod var ama hiç rendering optimizasyon modu yok |
| WRN_004 | Memory optimizasyonu eksik | 20+ mod var ama FerriteCore/ModernFix yok |
| WRN_005 | Lighting optimizasyonu eksik | 15+ mod var ama Starlight yok |
| WRN_006 | Entity optimizasyonu eksik | Entity-heavy mod var ama Entity Culling yok |
| WRN_007 | Aşırı RAM kullanımı | Tahmini RAM > kullanıcı RAM'inin %80'i |
| WRN_008 | Nvidium + Shader çakışması | Nvidium varken shader modu aktif |

#### 💡 İpuçları (Info — opsiyonel öneri)

| ID | Kural | Koşul |
|----|-------|-------|
| TIP_001 | Performans paketi öner | 5+ content mod var, 0 optimization mod var |
| TIP_002 | C2ME öner | Worldgen mod var ama C2ME yok |
| TIP_003 | Clumps öner | Mob-heavy modpack ama Clumps yok |
| TIP_004 | Fabric API eksik | Fabric modları var ama Fabric API yok |

### 3.3 Hata Tespiti Akışı

```
her_mod_eklendiğinde():
  1. Tüm ERROR kurallarını kontrol et → varsa kırmızı uyarı
  2. Tüm WARNING kurallarını kontrol et → varsa sarı uyarı  
  3. Her 5 mod eklendikten sonra TIP kurallarını kontrol et → varsa mavi öneri
  4. Toplam performans skorunu güncelle
  5. FPS tahminini güncelle
```

---

## 4. Akıllı Mod Öneri Sistemi

### 4.1 Öneri Stratejileri

Web araştırmasında CurseForge'un halka açık bir "benzer mod" algoritması bulunamadı. Kendi sistemimizi kurmamız gerekiyor.

#### Strateji 1: Bağımlılık Bazlı Öneriler
```
Kullanıcı bir mod eklediğinde:
  → RequiredDependency → Otomatik ekle/öner
  → OptionalDependency → "Şunu da eklemek ister misin?" 
```
**Kaynak:** CurseForge API `dependencies` alanı

#### Strateji 2: Kategori Bazlı Öneriler
```
Kullanıcı technology modu eklediğinde:
  → Aynı kategorideki popüler modları öner
  → Örn: Create eklendi → Applied Energistics 2, Mekanism öner
```
**Kaynak:** CurseForge API kategori filtreleme + popülerlik sıralaması

#### Strateji 3: Optimizasyon Tamamlayıcı Öneriler
```
Eğer Sodium varsa → Iris, Sodium Extra, Reese's Sodium Options öner
Eğer ağır modpack'se → Lithium, FerriteCore, ModernFix öner
Eğer worldgen modları varsa → C2ME, Noisium öner
Eğer mob modları varsa → Entity Culling, Clumps öner
```
**Kaynak:** Kendi kural veritabanımız (`mod_recommendations` tablosu)

#### Strateji 4: "Birlikte Sık Kullanılan" Öneriler (Gelecek)
```
CurseForge'daki popüler modpack'leri analiz ederek:
  → X modunu içeren modpack'lerin %70'i Y modunu da içeriyor
  → O zaman X seçildiğinde Y'yi öner
```
**Kaynak:** Kaggle CurseForge dataset veya manuel modpack analizi  
**Not:** Bu CurseForge cache yasağı kapsamında DEĞİL çünkü kendi analiz verimiz.

### 4.2 Öneri Veritabanı Yapısı

```sql
-- Kendi kurallarımız (CurseForge verisinden bağımsız)
CREATE TABLE mod_recommendations (
  id SERIAL PRIMARY KEY,
  trigger_mod_slug VARCHAR(255),     -- Tetikleyen mod
  recommended_mod_slug VARCHAR(255), -- Önerilen mod
  reason VARCHAR(50),                -- 'dependency', 'category', 'optimization', 'popular_combo'
  priority INT,                      -- 1-10 (10 = en güçlü öneri)
  message TEXT                       -- Kullanıcıya gösterilecek mesaj
);
```

### 4.3 Öneri Gösterim Mantığı

```
Öncelik sırası:
  1. Eksik zorunlu bağımlılıklar (MUTLAKA göster)
  2. Optimizasyon önerileri (performans skoru düşükse)
  3. Opsiyonel bağımlılıklar
  4. Kategori/tema bazlı öneriler
  5. Popüler kombinasyonlar

Maksimum 5 öneri göster (kullanıcıyı bunaltmamak için)
```

---

## 5. Teknik Kararlar & Özet

| Karar | Seçim | Gerekçe |
|-------|-------|---------|
| Uyumluluk kaynağı | CurseForge API + kendi DB | API'den dependency/incompatible, DB'den bilinen çakışmalar |
| FPS tahmin yöntemi | Kendi puanlama sistemi | Toplulukta standart yok, kendi formülümüzü kullanıyoruz |
| Performans puanları | Admin tarafından manuel atama | Her moda 0-10 performans maliyeti, zamanla geliştirilebilir |
| Hata tespit yöntemi | Kural motoru (rule engine) | JSON formatında kurallar, kolay genişletilebilir |
| Öneri kaynağı | Hibrit (API + kendi kurallar) | Bağımlılıklar API'den, öneriler kendi DB'mizden |
| Cache yasağı etkisi | Tüm mod verileri anlık API'den | Sadece kendi kurallarımız ve öneri verileri DB'de |

---

## 6. İleri Araştırma Gereksinimleri

- [ ] Spark profiler verileriyle gerçek FPS benchmark testleri yapmak (performans puanlarını kalibre etmek için)
- [ ] Kaggle CurseForge dataset'ini analiz ederek "sık birlikte kullanılan" mod çiftlerini çıkarmak
- [ ] CurseForge API'nin `getModFiles` endpointindeki dependency response formatını gerçek veriyle doğrulamak (API key geldikten sonra)
- [ ] Farklı donanım profillerinde vanilla FPS referans değerlerini doğrulamak
