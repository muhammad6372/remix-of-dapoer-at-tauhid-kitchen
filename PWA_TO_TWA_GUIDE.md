# 🚀 PANDUAN PWA → TWA (APK Android) untuk Dapoer Attauhid

## ✅ STATUS PERSIAPAN PWA

| Item | Status | Keterangan |
|------|--------|-----------|
| `vite-plugin-pwa` | ✅ Installed | v1.2.0 |
| Manifest.json | ✅ Configured | Akan ter-generate di `/manifest.json` |
| Service Worker | ✅ Auto-Update | Via Workbox |
| Mobile Friendly | ✅ Yes | Meta viewport & theme-color sudah ada |
| Icons | ✅ Ready | 192px & 512px PNG di `/public` |
| Display Mode | ✅ Standalone | Siap jadi native app |

---

## 📋 STEP 0: PRE-CHECK SEBELUM MULAI

Sebelum install Bubblewrap, pastikan:

```bash
# 1. Cek Node.js sudah terpasang
node --version    # v18+ required

# 2. Test build PWA
npm run build

# 3. Cek manifest.json bisa diakses
# Nanti setelah deploy: https://domain-kamu.com/manifest.json
```

### ✅ Checklist Web App:

- [ ] Punya domain HTTPS (wajib untuk TWA)
- [ ] App sudah ter-host online
- [ ] Bisa akses: `https://domain-kamu.com/manifest.json`
- [ ] Service worker berfungsi (test offline)
- [ ] Responsive di mobile (test di DevTools)

---

## 🔰 STEP 1: INSTALL TOOLS

### 1A. Install Bubblewrap CLI

```bash
npm install -g @bubblewrap/cli
```

### 1B. Verifikasi instalasi

```bash
bubblewrap --version
```

**Prasyarat OS:**
- **Windows**: Butuh Java SDK + Android SDK (akan guide)
- **macOS/Linux**: Sama, butuh Java + Android SDK

---

## 🔰 STEP 2: GENERATE PROJECT TWA (INIT)

### Command:

```bash
bubblewrap init --manifest=https://domain-kamu.com/manifest.json
```

### Pertanyaan yang akan ditanya:

```
✐ App name
  → Dapoer Attauhid

✐ Package name (reverse domain format)
  → com.dapoer.attauhid

✐ Launcher icon URL
  → https://domain-kamu.com/pwa-icon-512.png

✐ Status bar color
  → #000000 (dari theme_color manifest)
```

### Output:
Folder baru akan dibuat, misal: `dapoer-attauhid-twа`

---

## 🔰 STEP 3: BUILD PERTAMA (UNTUK AMBIL SHA256)

### Masuk ke folder

```bash
cd dapoer-attauhid-twa
```

### Build APK

```bash
bubblewrap build
```

**Ini akan:**
- Generate APK + AAB
- Buat keystore (kalau belum ada)
- Test build semua

---

## 🔰 STEP 4: AMBIL SHA256 (PENTING!)

### Lokasi keystore

Biasanya di folder project: `my-release-key.keystore`

### Command untuk ambil SHA256

```bash
keytool -list -v -keystore my-release-key.keystore
```

**Nanti diminta password → tekan Enter (default: android)**

### Output yang dicari:

```
SHA1: XX:XX:XX:XX:...
SHA256: XX:XX:XX:XX:XX:XX:XX:XX:...
         ↑ INI YANG DIPAKAI
Certificate fingerprints:
         SHA1: ...
         SHA256: ...
```

**COPY SHA256 fingerprint-nya**

---

## 🔰 STEP 5: UPDATE assetlinks.json

### Edit file:

Buka file: `public/.well-known/assetlinks.json`

Sebelumnya:
```json
{
  "sha256_cert_fingerprints": [
    "GANTI_DENGAN_SHA256_DARI_KEYTOOL"
  ]
}
```

Sesudahnya (contoh):
```json
{
  "sha256_cert_fingerprints": [
    "AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78:90:AB:CD:EF:12:34:56:78"
  ]
}
```

---

## 🔰 STEP 6: UPLOAD KE WEBSITE

Letakkan file di struktur ini di server Anda:

```
public_html/
└── .well-known/
    └── assetlinks.json
```

**URL final harus:**
```
https://domain-kamu.com/.well-known/assetlinks.json
```

---

## 🔰 STEP 7: TEST FILE ASSETLINKS

### Buka di browser atau terminal:

```bash
curl https://domain-kamu.com/.well-known/assetlinks.json
```

### Harus:
- ✅ Tidak error 404
- ✅ Muncul JSON valid
- ✅ Tidak ada error CORS

### Atau buka di browser:
```
https://domain-kamu.com/.well-known/assetlinks.json
```

---

## 🔰 STEP 8: BUILD FINAL

### Kembali ke folder TWA

```bash
cd dapoer-attauhid-twa
```

### Build APK final

```bash
bubblewrap build
```

**Ini akan generate:**
- `app-release.apk` → untuk install langsung
- `app-release.aab` → untuk Play Store

---

## 🔰 STEP 9: INSTALL APK

### Opsi A: Test di Android emulator

```bash
# List devices
adb devices

# Install
adb install -r app-release.apk
```

### Opsi B: Test device real

1. Copy `app-release.apk` ke Android phone
2. Enable "Install from Unknown Sources" di settings
3. Tap file → install

### Opsi C: Google Play Internal Testing

1. Upload ke Google Play Console
2. Internal testing track
3. Invite testers dari email list

---

## 🔰 STEP 10: UPLOAD KE GOOGLE PLAY STORE

### Persyaratan:

- [ ] Sudah punya Google Play Developer Account ($25 one-time)
- [ ] APK/AAB sudah signed
- [ ] `assetlinks.json` sudah online
- [ ] Privacy Policy di app
- [ ] Screenshots & description

### Target Play Store:

**Build 8+ harus pakai AAB (.aab file)**

```bash
# Di Google Play Console, upload:
build/app-release.aab
```

---

## ⚠️ TIPS SUKSES

1. **Jangan cuma wrapper** → pastikan ada nilai (login, data, fitur)
2. **Test offline** → pastikan service worker bekerja
3. **assetlinks.json must match** → SHA256 harus exactly sama
4. **HTTPS wajib** → Play Store tidak terima HTTP
5. **Privacy policy** → wajib di Play Store
6. **Icon responsive** → test di berbagai ukuran

---

## 🐛 TROUBLESHOOTING

### "assetlinks.json not working"
```bash
# Cek CORS
curl -v https://domain-kamu.com/.well-known/assetlinks.json

# Harus ada header:
# Access-Control-Allow-Origin: *
```

### "APK failed to install"
```bash
# Cek kompatibilitas
adb install -r app-release.apk

# Solusi: rebuild dengan minSdkVersion lebih rendah di vite.config.ts
```

### "SHA256 tidak cocok"
```bash
# Verifikasi lagi
keytool -list -v -keystore my-release-key.keystore

# Pastikan copy paste benar tanpa spacing
```

---

## 📞 NEXT STEPS

1. ✅ **Sekarang**: Siapkan domain HTTPS
2. ✅ **Deploy**: Push app ke production server
3. ✅ **Init**: Jalankan `bubblewrap init`
4. ✅ **Build**: Generate APK
5. ✅ **SHA256**: Ambil fingerprint
6. ✅ **assetlinks**: Upload ke `.well-known/`
7. ✅ **Final**: Rebuild & test
8. ✅ **Play Store**: Submit ke Google Play

---

## 📝 NOTES

- Hyper-lokalisasi siap (menu dalam Bahasa Indonesia)
- Dark mode support sudah ada
- Catering system sudah functional
- Tinggal distibusikan ke Play Store

**Status: READY FOR TWA! 🚀**
