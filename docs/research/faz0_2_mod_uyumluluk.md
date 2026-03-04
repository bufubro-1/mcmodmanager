# Faz 0.2 — Mod Uyumluluk & Bağımlılık Araştırması

## 1. Mod Loader Karşılaştırması

| Loader | Durum (2025) | Güçlü Yanı | Zayıf Yanı | Uyumluluk |
|--------|-------------|-------------|-------------|-----------|
| **Forge** | Eski versiyon desteği (≤1.20.1) | Devasa mod kütüphanesi, büyük modpack'ler | Yavaş güncelleme, ağır | NeoForge ile 1.20.1'e kadar uyumlu, sonra ayrılıyor |
| **Fabric** | Aktif, popüler | Hafif, hızlı güncelleme, performans modları | Büyük content mod sayısı daha az | Quilt ile çoğu mod uyumlu |
| **NeoForge** | Forge'un halefi (≥1.20.2) | Modern Forge, aktif geliştirme | Henüz olgunlaşıyor | 1.20.2+ için Forge ile uyumsuz |
| **Quilt** | Ölmek üzere | Fabric uyumluluğu | Geliştirici ilgisi düşük | Fabric modları çalışır |

### Projemiz İçin Karar
- **Desteklenmeli:** Forge, Fabric, NeoForge
- **Düşük öncelik:** Quilt (Fabric uyumluluğu yeterli)
- **Kural:** Bir modpack'te sadece TEK bir loader olabilir. Farklı loader modları **asla** karıştırılamaz.

---

## 2. Minecraft Sürüm Uyumluluk Matrisi

| MC Sürüm | Forge | Fabric | NeoForge | Notlar |
|----------|-------|--------|----------|--------|
| 1.7.10 | ✅ | ❌ | ❌ | Eski klasik modlar |
| 1.12.2 | ✅ | ❌ | ❌ | En popüler eski sürüm |
| 1.16.5 | ✅ | ✅ | ❌ | İlk büyük Fabric dalgası |
| 1.18.2 | ✅ | ✅ | ❌ | Popüler, geniş mod desteği |
| 1.19.2 | ✅ | ✅ | ❌ | Geniş mod desteği |
| 1.20.1 | ✅ | ✅ | ✅* | *NeoForge = Forge (son ortak sürüm) |
| 1.20.2+ | ✅* | ✅ | ✅ | *Forge desteği azalıyor |
| 1.21.x | ❌* | ✅ | ✅ | *Forge çoğu modda desteği bıraktı |

### Projemiz İçin Karar
- Kullanıcı önce **MC sürümünü** seçer → sonra uygun **loader seçenekleri** gösterilir
- CurseForge API'den her modun `gameVersions` ve `modLoaders` bilgisi çekilir

---

## 3. Bilinen Mod Çakışmaları (Incompatibility Database)

### 🔴 Kritik Çakışmalar (Crash / Bozulma)

| Mod A | Mod B | Sebep | Çözüm |
|-------|-------|-------|-------|
| OptiFine | Sodium | Her ikisi de rendering engine'i değiştirir | Birini seç: Sodium (Fabric) veya OptiFine (Forge) |
| OptiFine | Iris Shaders | Shader sistemi çakışması | Iris kullan (Sodium ile uyumlu) |
| Starlight | Phosphor | Her ikisi de lighting engine'i değiştirir | Birini seç |
| Sodium | Nvidium + Shaders | Nvidium shader desteklemiyor | Shader istiyorsan Nvidium kaldır |
| OptiFabric + OptiFine | Çoğu Fabric mod | OptiFabric kararsız | Sodium + Iris kullan |

### 🟡 İşlevsel Çakışmalar (Aynı İşi Yapan Modlar)

| Kategori | Modlar | Önerilen |
|----------|--------|----------|
| Rendering | OptiFine vs Sodium vs Embeddium | Fabric: Sodium, Forge: Embeddium |
| Lighting | Starlight vs Phosphor | Starlight (daha performanslı) |
| Shader | OptiFine Shaders vs Iris vs Oculus | Fabric: Iris, Forge: Oculus |
| Connected Textures | OptiFine CT vs Continuity | Fabric: Continuity |
| Chunk Loading | C2ME vs Chunk Pregenerator | Farklı amaçlar, birlikte olabilir |

### 🟢 Zorunlu Bağımlılıklar (Sık Unutulanlar)

| Mod | Gerekli Bağımlılık | Notlar |
|-----|-------------------|--------|
| Sodium | — | Bağımsız çalışır |
| Iris Shaders | Sodium | Sodium olmadan çalışmaz |
| Sodium Extra | Sodium | Ek ayarlar |
| Continuity | Fabric API | Connected textures |
| Çoğu Fabric modu | Fabric API | Temel kütüphane |
| Çoğu Forge modu | Forge loader | Otomatik dahil |
| Create | Flywheel | Forge versiyonunda zorunlu |
| JEI (Forge) / REI (Fabric) | — | Bağımsız ama hemen her modpack'te var |

---

## 4. Performans Modları Kategorileri

### Kategori 1: Rendering (GPU Optimizasyonu)
| Mod | Loader | Etki | FPS Artışı |
|-----|--------|------|-----------|
| **Sodium** | Fabric | Rendering engine yeniden yazımı | ~2-3x |
| **Embeddium** | Forge/NeoForge | Sodium'un Forge portu | ~2-3x |
| **Entity Culling** | Fabric/Forge | Görünmeyen entity'leri render etmez | %10-20 |
| **Enhanced Block Entities** | Fabric | Chest, furnace vb. render optimizasyonu | %5-15 |
| **ImmediatelyFast** | Fabric/Forge | Immediate mode rendering hızlandırma | %5-10 |
| **Nvidium** | Fabric | Nvidia mesh shader kullanımı | %20-50 (Nvidia only) |
| **Dynamic FPS** | Fabric | Pencere odakta değilken FPS düşürme | Dolaylı |

### Kategori 2: Game Logic (CPU Optimizasyonu)
| Mod | Loader | Etki | Performans Artışı |
|-----|--------|------|-------------------|
| **Lithium** | Fabric | AI, pathfinding, ticking, physics | TPS iyileşme |
| **C2ME** | Fabric | Chunk generation multithreading | Chunk yükleme 2-5x |
| **Starlight** | Fabric/Forge | Lighting engine yeniden yazımı | Lighting 10-50x |

### Kategori 3: Memory (RAM Optimizasyonu)
| Mod | Loader | Etki | RAM Tasarrufu |
|-----|--------|------|--------------|
| **FerriteCore** | Fabric/Forge | RAM kullanımını düşürür | %40'a kadar |
| **ModernFix** | Fabric/Forge | Bug fix + memory optimizasyonu | Değişken |
| **LazyDFU** | Fabric | Başlangıç süresini kısaltır | — |

### Kategori 4: Shader Desteği
| Mod | Loader | Notlar |
|-----|--------|--------|
| **Iris Shaders** | Fabric | Sodium ile uyumlu, OptiFine shader'ları çalıştırır |
| **Oculus** | Forge | Iris'in Forge portu, Embeddium ile uyumlu |

### Önerilen Kombinasyonlar
- **Fabric Performans Paketi:** Sodium + Lithium + Starlight + Iris + FerriteCore + Entity Culling + C2ME + ModernFix
- **Forge Performans Paketi:** Embeddium + Oculus + Starlight + FerriteCore + Entity Culling + ModernFix
- **Hazır Modpack:** Fabulously Optimized (Fabric, tüm optimizasyonları içerir)

---

## 5. Bağımlılık Zinciri Mantığı

### CurseForge API'deki Bağımlılık Türleri
```
relationType:
  1 = EmbeddedLibrary  (modun içine gömülü, ayrıca indirmeye gerek yok)
  2 = OptionalDependency (opsiyonel, olmasa da çalışır)
  3 = RequiredDependency (zorunlu, olmadan çalışmaz)
  4 = Tool (geliştirme aracı, son kullanıcıyı ilgilendirmez)
  5 = Incompatible (çakışan mod!)
  6 = Include (modpack'te dahil edilmiş)
```

### Projemiz İçin Dependency Resolution Algoritması
1. Kullanıcı mod eklediğinde → CurseForge API'den bağımlılıkları çek
2. `RequiredDependency` (3) → Otomatik ekle veya "şu mod da gerekli" uyarısı göster
3. `OptionalDependency` (2) → "Şunu da eklemek ister misin?" önerisi
4. `Incompatible` (5) → 🔴 Kırmızı uyarı, eklemeyi engelle
5. Zincirleme bağımlılıklar → Recursive olarak tüm zinciri çöz

---

## 6. Projemize Etkiler & Kararlar

| Karar | Detay |
|-------|-------|
| Loader kısıtlaması | Modpack tek loader olmalı, karıştırma engellenecek |
| Sürüm filtresi | MC sürümüne göre loader seçenekleri filtrelenecek |
| Çakışma DB'si | Kendi veritabanımızda `mod_compatibility_rules` tablosunda tutulacak |
| Bağımlılık çözümü | CurseForge API `relationType` kullanılacak |
| Performans puanlama | Her mod kategorisine göre performance_cost atanacak (Faz 0.3'te detaylandırılacak) |
| Duplicate tespiti | Aynı işi yapan modlar (Sodium vs OptiFine) için kural motoru |
