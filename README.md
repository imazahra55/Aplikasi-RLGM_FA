# RLGM — Development Model Reflective Learning Based on Growth Mindset

Aplikasi web penelitian pengembangan (Plomp) dengan **database terpusat Google Sheets** (melalui Google Apps Script) dan **hosting GitHub Pages**, sehingga dosen, observer, dan mahasiswa dapat memakai data yang sama dari perangkat mana pun.

```
Perangkat (HP / laptop)  ──HTTPS──>  GitHub Pages (index.html)
        │
        └──── sinkronisasi per-record ────>  Google Apps Script (Code.gs)  ──>  Google Sheets + Google Drive
```

## Isi paket

| Berkas | Fungsi |
|---|---|
| `index.html` | Aplikasi lengkap (satu berkas) |
| `apps-script/Code.gs` | Backend: login, pendaftaran, hak akses, sinkronisasi, riwayat, backup |
| `.nojekyll` | Agar GitHub Pages menyajikan berkas apa adanya |

---

## Langkah 1 — Siapkan database (Google Sheets + Apps Script)

1. Buka <https://sheets.new> dan beri nama spreadsheet, misalnya **RLGM Database**.
2. Pilih menu **Ekstensi → Apps Script**.
3. Hapus isi `Code.gs`, lalu tempel seluruh isi `apps-script/Code.gs`. Simpan (ikon disket).
4. Di bilah atas, pilih fungsi **`setup`**, lalu klik **Jalankan**. Izinkan akses ke Spreadsheet dan Drive (pilih akun → *Advanced* → *Go to project* → *Allow*).
   Fungsi ini membuat sheet `records`, `history`, `audit`, `revisions`, membuat folder Drive untuk berkas besar, akun admin awal, dan pemicu backup harian.
5. Klik **Terapkan → Deployment baru**, lalu pilih ikon roda gigi dan **Aplikasi web**:
   - *Jalankan sebagai*: **Saya**
   - *Siapa yang memiliki akses*: **Siapa saja** (Anyone)
6. Klik **Terapkan** dan salin **URL aplikasi web** (berakhiran `/exec`).

> Setiap kali `Code.gs` diubah, buat versi baru lewat **Terapkan → Kelola deployment → Edit → Versi: Baru**, agar URL tetap sama.

## Langkah 2 — Tanam URL ke aplikasi

Buka `index.html`, cari bagian berikut di dekat akhir berkas, lalu tempel URL tadi:

```js
window.RLGM_CONFIG = {
  GAS_URL: 'https://script.google.com/macros/s/XXXXXXXX/exec'
};
```

Semua perangkat yang membuka aplikasi akan langsung tersambung ke database yang sama.
Cara lain tanpa mengedit berkas: di halaman login, klik **atur koneksi**, tempel URL, lalu **Simpan**. Cara ini hanya berlaku untuk perangkat itu.

## Langkah 3 — Terbitkan di GitHub Pages

1. Masuk ke <https://github.com>, klik **New repository**, beri nama misalnya `rlgm`, pilih **Public**, lalu **Create**.
2. Klik **Add file → Upload files**, unggah `index.html`, `.nojekyll`, dan folder `apps-script`. Klik **Commit changes**.
3. Buka **Settings → Pages**. Pada *Source*, pilih **Deploy from a branch**, *Branch* **main** / **(root)**, lalu **Save**.
4. Setelah 1–2 menit, aplikasi tersedia di `https://<username>.github.io/rlgm/`. Bagikan tautan ini kepada dosen, observer, dan mahasiswa.

> Repositori publik hanya berisi kode aplikasi. **Data penelitian tidak ada di GitHub**; data tersimpan di Google Sheets milik Anda dan hanya dapat diakses melalui login.

## Langkah 4 — Login pertama & data awal

1. Buka aplikasi, pilih **Admin/Peneliti**, lalu masuk dengan `admin` / `admin123`.
2. Karena database masih kosong, aplikasi menawarkan tiga pilihan:
   - **Struktur awal (tanpa data contoh)**: indikator PCK, sintaks ALACT termodifikasi, instrumen, angket, dan bab buku model. Pilih ini untuk penelitian sungguhan.
   - **Struktur + data contoh**: untuk latihan atau demo (20 mahasiswa fiktif). Akun contoh: `peneliti/admin123`, `dosen/dosen123`, `observer1/observer123`, mahasiswa `26010001/26010001`.
   - **Unggah data di perangkat ini**: bila Anda sudah mengisi data di versi lokal sebelumnya.
3. **Segera ganti password admin** (aplikasi akan meminta).
4. Buat mata kuliah dan kelas di **Identitas Perkuliahan**. **Kode kelas penelitian** (misalnya `RLGM-5A`) adalah kode yang dipakai mahasiswa saat mendaftar mandiri.
5. Tambahkan akun dosen dan observer di **Settings → Pengguna** (username + password awal).

## Akun & login

| Peran | ID login | Cara mendapatkan akun |
|---|---|---|
| Admin/Peneliti | username / email | `setup` membuat `admin`; admin menambah admin lain |
| Dosen | username / email | Dibuat admin |
| Observer | username / email | Dibuat admin (instrumen yang ditugaskan dapat dibatasi) |
| Mahasiswa | NIM / email | **Daftar mandiri** di halaman login (dengan kode kelas), impor Excel/CSV, atau ditambah dosen (password awal = NIM) |

- **Pendaftaran mandiri** mengisi NIM, nama, **universitas** (Universitas Bosowa / Universitas Negeri Malang / Universitas Negeri Makassar / Lainnya, isi sendiri), **prodi** (PGSD / Pendidikan Matematika / Lainnya, isi sendiri), semester, email, kode kelas, dan password.
- Admin dapat menutup pendaftaran atau mewajibkan verifikasi di **Settings → Perkuliahan**. Akun yang menunggu verifikasi disetujui dari **Database Mahasiswa → Setujui**.
- Lupa password: admin/dosen memakai tombol **Reset password**. Bila password admin sendiri terlupa, jalankan fungsi `resetAdmin` dari editor Apps Script (password kembali `admin123`).

## Keamanan & integritas data

- Password disimpan sebagai **hash SHA-256 + salt** dan tidak pernah dikirim ke perangkat.
- Setiap permintaan memerlukan token login yang ditandatangani server (berlaku 12 jam).
- **Hak tulis diperiksa di server.** Mahasiswa hanya dapat mengubah datanya sendiri dan tidak dapat mengubah skor. Skor soal objektif **dihitung ulang di server**. Kunci jawaban tidak dikirim ke perangkat mahasiswa. Data mahasiswa lain (NIM, email, jawaban) tidak dikirim ke mahasiswa.
- **Setiap perubahan mengarsipkan versi lama** ke sheet `history`. Bila dua perangkat mengubah data yang sama, versi yang lebih lama ditolak dan juga diarsipkan, sehingga tidak ada data yang tertimpa tanpa jejak.
- Bekerja offline: perubahan tetap tersimpan di perangkat dan dikirim otomatis saat koneksi kembali.
- **Backup**: salinan spreadsheet otomatis setiap hari (14 salinan terakhir) di folder `RLGM_Berkas_…` di Google Drive, ditambah backup JSON dari menu *Etika & Data Governance → Backup*.
- Jangan membagikan akses edit spreadsheet kepada mahasiswa. Cukup bagikan tautan aplikasi.

## Batas yang perlu diketahui

- Google Apps Script memiliki kuota harian dan batas eksekusi bersamaan (±30 permintaan sekaligus, maks. 6 menit per eksekusi). Batas ini memadai untuk beberapa kelas berukuran ±25–40 mahasiswa. Bila banyak perangkat terbuka bersamaan, sinkronisasi otomatis tiap ±30 detik akan sedikit lebih lambat, tetapi data tidak hilang.
- Berkas unggahan besar (PDF, gambar) disimpan sebagai berkas JSON di Google Drive. Disarankan maksimal ±10 MB per berkas.
- Pembatasan "dosen hanya melihat kelasnya" diterapkan di tampilan aplikasi. Pemisahan data yang ketat di server saat ini berlaku untuk peran mahasiswa.
