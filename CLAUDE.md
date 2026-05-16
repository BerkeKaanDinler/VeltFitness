# VELT Fitness — Proje Bağlamı

Offline-first Flutter fitness takip uygulaması. İnternet bağlantısı gerektirmez.
Hedef platform: iOS (öncelikli), Android, Web.

---

## Tech Stack

- **Flutter** 3.x, Dart SDK >=3.2.0
- **State:** `ValueNotifier<T>` — Provider veya Bloc yok, kasıtlı tercih
- **Kalıcılık:** `shared_preferences` — tüm veri JSON olarak cihazda
- **Font:** Inter + Inter Tight (Google Fonts)
- **Temalar:** 9 adet runtime değiştirilebilir tema

---

## Proje Yapısı

```
lib/
├── main.dart                   # Boot: PrefsService, RoutineStore, WorkoutHistoryStore, NutritionStore init
├── app.dart                    # VeltRoot (MaterialApp + tema) + VeltApp (shell + tab router)
├── velt_tabs.dart              # Tab index sabitleri (home=0 … profile=4)
│
├── screens/
│   ├── home_screen.dart        # Dashboard: streak, heatmap, PRs, son antrenman
│   ├── train_screen.dart       # 6 hazır program + kişisel rutinler
│   ├── active_workout_screen.dart  # Canlı antrenman + WorkoutSummaryScreen + modeller
│   ├── nutrition_screen.dart   # Kalori halkası, makro barlar, yemek logu
│   ├── progress_screen.dart    # Grafikler, PR listesi, vücut ağırlığı trend
│   ├── profile_screen.dart     # Ayarlar, tema seçici, vücut ağırlığı geçmişi
│   ├── onboarding_screen.dart  # 4 sayfalık ilk kurulum akışı
│   ├── routine_editor_screen.dart  # Rutin oluştur/düzenle
│   └── workout_history_screen.dart # Tüm antrenman geçmişi
│
├── services/
│   ├── prefs_service.dart      # SharedPreferences static wrapper — tek global erişim noktası
│   ├── routine_store.dart      # ValueNotifier<List<Routine>> — CRUD + persist
│   ├── workout_history_store.dart  # ValueNotifier<List<CompletedWorkout>> — streak, PR, volume hesapları
│   └── nutrition_store.dart    # ValueNotifier (entries + targets) — günlük log
│
├── models/
│   └── routine.dart            # Routine, id/name/colorValue/exercises/lastDone
│
├── data/
│   └── program_data.dart       # 6 hazır program sabit verisi (PPL, 5x5, 5/3/1, UL, GZCLP, nSuns)
│
├── widgets/
│   ├── shared_widgets.dart     # PrimaryButton, GhostButton, SectionHeader, RoutineCard, PRCard, StatCard, EmptyState
│   ├── set_row.dart            # SetRowData modeli + SetRow widget (antrenman sırasında set girişi)
│   └── rest_timer_banner.dart  # Setler arası dinlenme sayacı
│
├── theme/
│   ├── app_colors.dart         # AppColors ThemeExtension — 9 tema renk paleti
│   ├── app_theme.dart          # AppTheme._build() — MaterialApp'e bağlanır
│   ├── app_typography.dart     # AppTypography static metotlar (displayXL → caption)
│   └── app_spacing.dart        # AppSpacing (4pt grid), AppRadius, AppTouchTarget
│
└── utils/
    └── weight_unit.dart        # kg ↔ lb dönüşüm yardımcıları
```

> **Not:** `WorkoutExercise`, `CompletedWorkout`, `SetRowData` modelleri şu an
> `active_workout_screen.dart` içinde tanımlı. İleride `models/` altına taşınacak.

---

## Önemli Mimari Kararlar

- **`ValueNotifier` tercih edildi** — Riverpod/BLoC olmadan reaktif UI, kasıtlı sadelik
- **Tüm servisler static** — `RoutineStore.routines.value` gibi doğrudan erişim
- **Tema runtime değişir** — `veltThemeKey` (app.dart) global `ValueNotifier<String>`
- **Antrenman kurtarma var** — uygulama kapanınca `PrefsService.activeWorkout`'a JSON yazar, açılışta restore eder
- **Offline-first** — hiç network çağrısı yok, tüm veri cihazda

---

## Mevcut Tasarım Skoru: 6.5/10

### Güçlü Yanlar
- 9 tema, `AppColors.lerp()` ile animasyonlu geçiş
- 4pt spacing grid, Apple HIG touch target uyumu
- Inter + Inter Tight tipografi hiyerarşisi
- Rest timer: renk geçişi + haptic pulse (en polished component)
- Workout summary: staggered fade+slide animasyonları
- Onboarding: sayfa geçişleri, progress dot animasyonu

### Eksikler (Öncelik Sırasıyla)

| # | Eksik | Etki |
|---|-------|------|
| 1 | Kart gölgesi yok — her yerde sadece border | Görsel derinlik yok |
| 2 | PR anında kutlama animasyonu yok | Duygusal bağ yok |
| 3 | Empty state'lerde illüstrasyon yok | Uygulama kişiliği eksik |
| 4 | Grafikler: grid, label, data dot yok | Basic görünüm |
| 5 | Modal geçişleri Flutter default | Polished his yok |
| 6 | Ses tasarımı yok | Tek boyutlu geri bildirim |
| 7 | Skeleton/shimmer loader yok | Anlık görünüm |

---

## Kritik Buglar

### 1. Nutrition gece yarısı sıfırlanmıyor
- **Dosya:** `lib/services/nutrition_store.dart`
- **Sorun:** `_loadToday()` sadece `init()`'te çağrılıyor. Gece 00:00'dan sonra açılınca eski yemekler görünüyor.
- **Fix:** App foreground'a gelince tarihi kontrol et, değiştiyse `_loadToday()` çağır.

### 2. PR timestamp yok
- **Dosya:** `lib/services/workout_history_store.dart` satır 82-96
- **Sorun:** `allTimePRs` map'i sadece `relativeDate` ("2 hafta önce") saklıyor. Aynı günde 2 PR ayırt edilemiyor.
- **Fix:** `completedAt` DateTime'ı tuple'a ekle.

### 3. Streak sıralama hatası
- **Dosya:** `lib/services/workout_history_store.dart` satır 39-60
- **Sorun:** Geçmişe tarih girilen antrenman streak'i kırıyor.
- **Fix:** Streak hesabından önce listeyi tarihe göre sırala.

### 4. Vücut ağırlığı egzersizleri "0 kg" gösteriyor
- **Dosya:** `lib/widgets/set_row.dart` satır 172-176
- **Fix:** `weight == 0` ise "BW" göster.

---

## Test Durumu: 2/10

```
test/widget_test.dart — 5 test, TÜMÜ onboarding
```

Test edilmemiş kritik akışlar:
- Antrenman başlatma → set tamamlama → bitirme
- PR tespit algoritması
- Streak hesabı
- Beslenme log ekleme/silme
- Rutin CRUD (ekle, güncelle, sil, persist)
- Antrenman kurtarma (resume)

---

## Faz Planı (Tasarım → 9.5/10)

### Faz 1 — Hızlı Kazanımlar (2-3 gün) → 7.5/10
- [ ] Tüm kartlara `boxShadow` ekle
- [ ] Grafiklere grid + Y ekseni etiketleri
- [ ] Modal geçişlerini `slide-up + BackdropFilter blur` yap
- [ ] İkincil içerikte font weight `w300`'e indir

### Faz 2 — Kutlama & Geri Bildirim (1 hafta) → 8.5/10
- [ ] PR anında konfeti (`confetti` paketi)
- [ ] Set tamamlama sesi (`audioplayers` paketi)
- [ ] Shimmer skeleton loader (`shimmer` paketi)
- [ ] Navigation geçişleri `PageRouteBuilder` ile özelleştir

### Faz 3 — İllüstrasyon & Grafik (2 hafta) → 9/10
- [ ] 6 ekran için boş durum illüstrasyonları (unDraw.co veya Lottie)
- [ ] Grafikleri `fl_chart` ile yeniden yaz (interaktif, cursor'lu)
- [ ] Onboarding son sayfaya kutlama animasyonu

### Faz 4 — Production Kalitesi (2-3 hafta) → 9.5/10
- [ ] Kritik bug fix'leri (nutrition rollover, PR timestamp, streak sort)
- [ ] Accessibility: semantic label'lar, contrast ratio
- [ ] Core flow test coverage
- [ ] App Store görselleri

---

## Geliştirme Notları

```bash
# Çalıştır
flutter run

# Test
flutter test

# Analiz
flutter analyze

# Paket güncelle
flutter pub get
```

Renk/tema değişikliği → `lib/theme/app_colors.dart`
Yeni ekran → `lib/screens/` + `app.dart`'ta tab'a ekle
Yeni widget → `lib/widgets/shared_widgets.dart` (tek dosyada tutuluyor)
