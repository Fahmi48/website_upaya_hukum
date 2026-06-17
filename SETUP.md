# KasasiTrack v2 — Panduan Setup Firebase
# (Hanya menggunakan Authentication + Firestore — TANPA Storage)

## Fitur Firebase yang Digunakan
- ✅ Firebase Authentication (Email/Password + Google)
- ✅ Cloud Firestore (database)
- ❌ Firebase Storage (tidak digunakan)
- ❌ Firebase Hosting (opsional, bisa pakai hosting lain)

---

## 1. Buat Firebase Project

1. Buka https://console.firebase.google.com
2. Klik **Add project** → nama: `kasasitrack`
3. Nonaktifkan Google Analytics (opsional) → **Create project**

---

## 2. Aktifkan Authentication

1. Sidebar → **Build → Authentication → Get started**
2. Tab **Sign-in method**, aktifkan:
   - **Email/Password** → Enable → Save
   - **Google** → Enable → masukkan Project support email → Save

---

## 3. Buat Firestore Database

1. Sidebar → **Build → Firestore Database → Create database**
2. Pilih **Start in production mode** → Next
3. Region: **asia-southeast2 (Jakarta)** → Enable

---

## 4. Firestore Security Rules

Buka **Firestore → Rules**, paste berikut, lalu **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Profil user: hanya pemilik yang bisa akses
    match /users/{userId} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }

    // Perkara kasasi: hanya pemilik (field uid) yang bisa CRUD
    match /cases/{caseId} {
      // Baca & update & hapus: hanya jika uid di dokumen == uid yang login
      allow read, update, delete: if request.auth != null
                                  && resource.data.uid == request.auth.uid;
      // Buat baru: pastikan uid di data == uid yang login
      allow create: if request.auth != null
                    && request.resource.data.uid == request.auth.uid;
    }
  }
}
```

---

## 5. Dapatkan Firebase Config

1. **Project Settings** (ikon gear kiri atas) → tab **General**
2. Scroll ke **Your apps** → klik ikon `</>` (Web)
3. Beri nama app → **Register app** → copy objek `firebaseConfig`

---

## 6. Pasang Config ke index.html

Buka `index.html`, cari:

```javascript
const firebaseConfig = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

Ganti semua nilai `"YOUR_..."` dengan nilai dari Firebase Console.

> **Catatan:** `storageBucket` ada di config bawaan Firebase tapi
> aplikasi ini **tidak** menggunakannya sama sekali — tidak ada
> `import` dari `firebase/storage`.

---

## 7. Authorized Domains (untuk Google Login)

Jika hosting di domain selain localhost:
1. **Authentication → Settings → Authorized domains**
2. Klik **Add domain** → masukkan domain Anda

---

## 8. Jalankan Lokal

Tidak perlu build step — cukup buka dengan live server:

```bash
# Opsi 1: VS Code Live Server extension
# Klik kanan index.html → Open with Live Server

# Opsi 2: Python
python3 -m http.server 3000

# Opsi 3: Node.js
npx serve .
```

Buka: http://localhost:3000

---

## 9. Deploy (Opsional)

### Firebase Hosting
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
# public dir: . (titik)
# single-page app: No
# overwrite index.html: No
firebase deploy --only hosting
```

### Atau hosting lainnya
Upload kedua file (`index.html` + `style.css`) ke:
- Netlify (drag & drop folder)
- Vercel
- GitHub Pages
- Shared hosting biasa

---

## Struktur Data Firestore

```
firestore/
│
├── users/{uid}
│   ├── name           : string
│   ├── email          : string
│   └── createdAt      : timestamp
│
└── cases/{caseId}
    ├── uid            : string   ← UID pemilik (untuk security rules)
    ├── num            : string   ← Nomor perkara
    ├── type           : string   ← Pidana / Perdata / TUN / dll
    ├── parties        : string   ← Para pihak
    ├── court          : string   ← Pengadilan asal
    ├── registered     : string   ← YYYY-MM-DD
    ├── notes          : string   ← Catatan umum
    ├── createdAt      : timestamp
    ├── steps          : {
    │   ├── pernyataan         : { status, date, notes }
    │   ├── memori             : { status, date, notes }
    │   ├── pbt_memori         : { status, date, notes }
    │   ├── kontra_memori      : { status, date, notes }
    │   ├── pbt_kontra         : { status, date, notes }
    │   ├── pbt_inzage         : { status, date, notes }
    │   ├── inzage             : { status, date, notes }
    │   └── putusan            : { status, date, notes }
    │ }
    └── activity       : Array<{ text: string, time: string }>
```

**Status nilai yang valid:** `pending` | `active` | `done` | `overdue`

---

## Tips Keamanan

- Jangan commit `firebaseConfig` ke repo publik → gunakan `.env`
- Batasi API Key di **Google Cloud Console → APIs & Services → Credentials**
  - HTTP referrers: tambahkan domain Anda
- Aktifkan **Firebase App Check** untuk proteksi ekstra
- Monitor usage di **Firebase Console → Usage & Billing**
