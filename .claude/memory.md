# VELT Fitness — Claude Oturum Hafızası

Bu dosya Claude'un proje bağlamını hızlı kurması için yazılmıştır.
Yeni session başında: "`.claude/memory.md` dosyasını oku" de.

---

## Bu Konuşmada Ne Konuştuk (16 Mayıs 2026)

### 1. Uygulama Analizi İstendi
Kullanıcı "şuanki fitness uygulamasının gidişatı nasıl, test analiz et" dedi.
Tüm Dart dosyaları okundu, kapsamlı analiz yapıldı.

**Analiz Sonucu:**
- Genel skor: **6/10**
- Test skoru: **2/10** — sadece 5 onboarding testi var, core akışlar test edilmemiş
- Mimari skor: **7/10** — ValueNotifier tabanlı, offline-first, solid

---

### 2. Tasarım Analizi İstendi
Kullanıcı "sence tasarım olarak zayıf ve basic mi kalıyor" dedi.

**Tasarım Skoru: 6.5/10**

**Güçlü:**
- 9 tema sistemi, lerp() animasyonlu geçiş
- Rest timer component (en polished — renk geçişi + haptic pulse)
- Workout summary staggered animasyonlar
- Inter + Inter Tight tipografi sistemi
- 4pt grid, Apple HIG touch target uyumu
- Onboarding akışı (8.5/10)

**Zayıf:**
- Kart gölgesi YOK — sadece border, düz görünüm
- Empty state'lerde illüstrasyon YOK
- PR anında kutlama animasyonu YOK (konfeti vs.)
- Grafikler basic — grid yok, label yok, data dot yok
- Modal geçişleri Flutter default
- Ses tasarımı YOK

**Rakip Karşılaştırma (Strong, Hevy, Nike Training):**
- Kart gölgeleri: onlarda var, VELT'te yok
- Empty state illüstrasyon: onlarda var, VELT'te yok
- PR konfeti: onlarda var, VELT'te yok
- Grafik polish: onlarda var, VELT'te yok

---

### 3. "Ne Kadar Sürede 10/10 Yaparız" Sorusu

Gerçekçi hedef **9.5/10**, toplam ~6 hafta:

| Faz | Süre | Hedef |
|-----|------|-------|
| Faz 1: Shadows + Grafik polish | 2-3 gün | 7.5/10 |
| Faz 2: Konfeti + Ses + Shimmer | 1 hafta | 8.5/10 |
| Faz 3: İllüstrasyon + fl_chart | 2 hafta | 9.0/10 |
| Faz 4: Bug fix + Test + A11y | 2-3 hafta | 9.5/10 |

**En hızlı etki:** Kart gölgeleri (2-3 saat iş, görsel etki büyük)
**En duygusal etki:** PR kutlaması (konfeti paketi)
**En kişilik katan:** Empty state illüstrasyonlar (unDraw.co veya Lottie)

---

### 4. CLAUDE.md ve Hafıza Konuşması

Kullanıcı PC'de VS Code ile çalışıyor, bu session cloud'da çalışıyor.
Session kapanırsa hafıza gider — bu yüzden CLAUDE.md ve bu dosya oluşturuldu.

**Önemli:** Kullanıcı şu an repodaki hali **en güncel değil** diyor.
Lokal PC'de daha fazla değişiklik yapılmış, Claude'a farklı tasarımlar yaptırılmış.
Akşam push edecek, sonra bu dosyalar güncellenecek.

---

## Kritik Buglar (Kod Düzeyinde)

### Bug 1 — Nutrition gece yarısı sıfırlanmıyor
- **Dosya:** `lib/services/nutrition_store.dart`
- **Problem:** `_loadToday()` sadece `init()`'te çağrılıyor
- **Senaryo:** Gece 23:55 yemek logla → 00:05 uygulamayı aç → eski yemekler görünüyor
- **Fix:** App foreground event'inde tarihi kontrol et, değiştiyse reload

### Bug 2 — PR timestamp yok
- **Dosya:** `lib/services/workout_history_store.dart` satır 82-96
- **Problem:** `allTimePRs` sadece `relativeDate` ("2 hafta önce") saklıyor
- **Fix:** `completedAt` DateTime'ı tuple'a ekle

### Bug 3 — Streak sıralama hatası
- **Dosya:** `lib/services/workout_history_store.dart` satır 39-60
- **Problem:** Geçmişe tarih girince streak kırılıyor
- **Fix:** Streak hesabından önce listeyi tarihe göre sırala

### Bug 4 — Vücut ağırlığı egzersizleri "0 kg" gösteriyor
- **Dosya:** `lib/widgets/set_row.dart` satır 172-176
- **Fix:** `weight == 0` ise "BW" yaz

---

## Test Durumu

```
test/widget_test.dart — 5 test
✅ Onboarding smoke
✅ Adım geçişi
✅ Kg seçimi
✅ Tam onboarding akışı
✅ Ana ekran görünürlüğü

❌ Antrenman başlatma/bitirme
❌ Set tamamlama + rest timer
❌ PR tespit algoritması
❌ Beslenme log
❌ Rutin CRUD
❌ Workout history persist
❌ Antrenman kurtarma (resume)
```

---

## Kullanıcı Hakkında Notlar

- PC'de **VS Code** ile geliştiriyor
- Bu konuşmalar **web/mobil Claude** üzerinden yapılıyor
- Repoya push ettiğinde buradan analiz devam edecek
- Akşam güncel kodu push edecek → o zaman bu dosyalar güncellenecek
- Tasarım konusunda aktif çalışıyor, Claude'a tasarım yaptırıyor

---

## Sonraki Adımlar (Kullanıcının Planı)

1. Akşam lokal değişiklikleri push et
2. Bu `.claude/memory.md` ve `CLAUDE.md` güncellensin
3. Tasarım Faz 1'e başla (kart gölgeleri öncelikli)
4. Kritik bug fix'ler (nutrition rollover en kritik)
5. Test coverage artır

---

## Claude'a Komutlar

Yeni session'da şunu söyle:
> "`.claude/memory.md` oku, kaldığımız yerden devam et"

Repodaki güncel kodu analiz ettirmek için:
> "Repodaki son hali analiz et, memory.md'yi güncelle"
