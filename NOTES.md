# EvFinans — Geliştirici Notları

Bu dosya, uygulamanın mimarisini, bilinen sınırlarını ve gelecekte yapılabilecek geliştirmeleri özetler.
AI asistanlarla (veya yeni bir geliştiriciyle) çalışırken bağlamı hızlıca aktarmak için kullanılır.

## Mimari

Uygulama tek bir `EvFinans.html` dosyasıdır. iOS'ta "Ana Ekrana Ekle" ile PWA olarak çalışır.

- **Veri deposu:** Tarayıcı `localStorage` (anahtar: `txs` + kategoriler/ayarlar).
- **Kimlik doğrulama:** MSAL.js (`@azure/msal-browser`) — `loginRedirect` + `acquireTokenSilent` / `acquireTokenRedirect` (popup iOS PWA'da çalışmaz).
- **Bulut senkronu:** Microsoft Graph API üzerinden OneDrive'da `EvFinans.xlsx` dosyasına yazma/okuma.
- **XLSX üretim:** SheetJS (`window.XLSX`) CDN'den yükleniyor.
- **Auto-sync:** `odInit` içinde 3 dakikada bir `odSync(true)` tick'i + `visibilitychange` tetikleyici.
- **Concurrency koruması:** Global `odSyncLock` mutex (aynı anda sadece bir sync/pull).

## Dikkat Edilecekler

- **Boş veri koruması:** `odSync` `txs.length === 0` ise buluta yazmaz (veri kaybını önler).
- **Silip yeniden kurulum:** `odInit` bağlıyken local boşsa otomatik `odPull(true)` yapar.
- **iOS Safari ITP:** Üçüncü taraf çerezler 7 günde silinebilir → silent token fail → redirect akışı devreye girer.
- **409 / 423 retry:** `createUploadSession` çakışmasında 2 saniye bekle, bir kez tekrar dene.

## Bilinen Sınırlar

- **Excel satır limiti:** 1.048.576 (pratikte asla ulaşılmaz).
- **Performans sınırı:** ~10–20 bin işlemden sonra iPhone'da sync yavaşlar; dosya 5 MB'ı geçince fark edilir.
- **localStorage kotası:** iOS Safari'de ~5–10 MB — 50 binin üstünde işlemde sorun olabilir.
- **Her sync tam dosyayı yeniden yazar** — delta sync yok.

## Planlanmış Geliştirmeler

1. **Arşivleme modu:** 2+ yıl öncesi işlemleri ayrı `EvFinans-Arsiv-YYYY.xlsx` dosyasına taşıma.
2. **Delta / range sync:** Microsoft Graph Excel API ile sadece değişen satırları güncelleme (hücre bazlı).
3. **Sync debounce:** Çok sık işlem girişlerinde auto-sync'i batch'leme (pil/veri tasarrufu).
4. **Çakışma tespiti:** Birden fazla cihazdan eş zamanlı düzenlemede son yazanın kazanması yerine merge mantığı.
5. **Yedek / versiyon:** OneDrive'da haftalık snapshot dosyaları (`EvFinans-Backup-YYYY-WW.xlsx`).

## Commit Geçmişi Önemli Noktaları

- `eebf1c0` — OneDrive session (iOS PWA) + boş veri koruması + ilk açılışta otomatik pull
- `02f58a5` — HTTP 409 conflict için sync lock mutex + 409/423 retry

## Deployment

GitHub Pages üzerinden yayında. `main` branch'e her commit otomatik deploy olur.
