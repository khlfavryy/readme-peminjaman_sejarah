# Heritage Haven — Sistem Peminjaman Buku Perpustakaan Digital

Repository : https://github.com/khlfavryy/repository-peminjaman-sejarah.git
Mockup & Algoritma : https://drive.google.com/drive/folders/1XrREon6eosp1Ic_ow6Q8UPEAuQOLuXrW?usp=drive_link

Aplikasi web manajemen perpustakaan berbasis PHP native (tanpa framework) dengan tiga peran pengguna: **Owner**, **Pustakawan**, dan **Siswa**. Mendukung katalog buku publik, peminjaman & pengembalian mandiri oleh siswa, rekap kunjungan, label QR buku, hingga statistik dashboard.

## ✨ Fitur

### Publik (tanpa login)
- Landing page dengan katalog buku, pencarian, dan filter kategori
- Statistik ringkas perpustakaan (jumlah buku, anggota, dsb.)
- Halaman Profil perpustakaan & FAQ / Pusat Bantuan
- Ulasan & rating bintang buku

### Siswa
- Pinjam & kembalikan buku secara mandiri
- Cetak struk peminjaman
- Riwayat kunjungan perpustakaan
- Notifikasi buku jatuh tempo

### Pustakawan (role `petugas`)
- Kelola data buku (tambah/edit/hapus, upload sampul, atur stok)
- Kelola data anggota (siswa)
- Kelola transaksi peminjaman & pengembalian (termasuk pengembalian manual)
- Rekap kunjungan siswa
- Cetak label QR per buku (untuk katalog fisik)
- Dashboard dengan grafik peminjaman 7 hari terakhir & aktivitas terbaru

### Owner (role `admin`)
- Kelola akun pustakawan
- Melihat seluruh rekap & statistik (mode lihat-saja, tidak bisa ubah data buku/anggota)
- Dashboard dengan grafik peminjaman 6 bulan terakhir, buku terpopuler, dan kategori terbanyak
- Cetak rekap laporan

## 🛠️ Teknologi

- **Backend:** PHP native + PDO (MySQL)
- **Database:** MySQL / MariaDB
- **Frontend:** HTML, CSS custom (tema navy–emas ala perpustakaan klasik), vanilla JavaScript
- **Library eksternal:** [Chart.js](https://www.chartjs.org/) (grafik dashboard), [QR Server API](https://goqr.me/api/) (generate QR label buku)

## 📁 Struktur Folder

```
peminjaman-buku/
├── admin/              # Halaman & aksi untuk Pustakawan (dan sebagian bisa diakses Owner sebagai lihat-saja)
│   ├── dashboard.php
│   ├── kelola_buku.php
│   ├── kelola_anggota.php
│   ├── transaksi.php
│   ├── kunjungan.php
│   └── qr.php
├── owner/               # Halaman khusus Owner
│   ├── dashboard.php
│   └── kelola_pustakawan.php
├── siswa/               # Halaman khusus Siswa
│   ├── dashboard.php
│   ├── pinjam.php
│   ├── pengembalian.php
│   ├── struk.php
│   └── kunjungan.php
├── auth/                # Login & logout
│   ├── login.php
│   └── logout.php
├── config/
│   └── koneksi.php      # Konfigurasi koneksi database
├── includes/
│   └── cek_login.php    # Helper pengecekan sesi & role
├── assets/               # CSS, gambar, video, audio
├── uploads/buku/         # Sampul buku hasil upload
├── index.php             # Landing page / katalog publik
├── profil.php
└── faq.php
```

## 🚀 Instalasi (Lokal)

2. **Siapkan database**
   - Buat database MySQL bernama `peminjaman_buku`.
   - Import struktur tabel yang dibutuhkan: `users`, `anggota`, `buku`, `transaksi`, `kunjungan` (lihat bagian [Skema Database](#-skema-database-ringkas) di bawah).

3. **Atur koneksi database**
   Edit `config/koneksi.php` sesuai environment kamu:
   ```php
   $host = 'localhost';
   $dbname = 'peminjaman_buku';
   $user = 'root';
   $pass = '';
   ```

4. **Jalankan dengan server PHP lokal (XAMPP/Laragon/dsb.)**
   Letakkan folder project di `htdocs` (atau `www`), lalu akses lewat:
   ```
   http://localhost/peminjaman-buku/
   ```

5. **Pastikan folder upload bisa ditulis**
   Folder `uploads/buku/` perlu permission tulis (write) agar upload sampul buku berfungsi.

## ☁️ Deploy ke Hosting (mis. InfinityFree)

> ⚠️ **Penting:** semua tautan internal di project ini harus pakai **relative path** (`../index.php`, `index.php`, dst), **bukan absolute path** seperti `/peminjaman-buku/...`. Di banyak shared hosting, isi folder project langsung jadi root `htdocs`, sehingga path absolute yang menyertakan nama folder lokal akan menghasilkan 404.

Sebelum deploy, cek dan pastikan tidak ada lagi path absolute tersisa, termasuk di dalam `header('Location: ...')`. Diketahui `includes/cek_login.php` masih memakai path absolute yang tidak konsisten (`/perpustakaan-digital/...` dan `/peminjaman-buku/...`) — sesuaikan ini dengan struktur folder di hosting kamu sebelum go-live.

## 👤 Alur Login & Role

Ada 3 role yang tersimpan di kolom `role` tabel `users`:

| Role di database | Sebutan | Redirect setelah login |
|---|---|---|
| `user` | Siswa | `siswa/dashboard.php` |
| `petugas` | Pustakawan | `admin/dashboard.php` |
| `admin` | Owner | `owner/dashboard.php` |

Login Owner tidak memakai URL rahasia — sistem otomatis mengenali role dari akun yang login lewat form Login Pustakawan/Admin yang sama.

## 🗃️ Skema Database (ringkas)

Tabel inti yang digunakan aplikasi ini (sesuaikan tipe kolom dengan kebutuhan):

- **users** — `id_user`, `username`, `password` (hashed), `role` (`user`/`petugas`/`admin`), `status_aktif`
- **anggota** — `id_anggota`, `id_user`, `nama_lengkap`, `kelas`, `NIS`, `tgl_daftar`
- **buku** — `id_buku`, `judul`, `penulis`, `penerbit`, `tahun_terbit`, `kategori`, `lokasi_rak`, `sinopsis`, `stok`, `gambar`
- **transaksi** — `id_transaksi`, `id_buku`, `id_anggota`, `tgl_pinjam`, `tgl_wajib_kembali`, `tgl_kembali`, `status`, `denda`, `status_denda`, kolom snapshot (`judul_buku_snapshot`, `nama_anggota_snapshot`) untuk riwayat saat buku/anggota dihapus
- **kunjungan** — data kunjungan siswa ke perpustakaan

## 📄 Lisensi

Belum ditentukan — tambahkan lisensi sesuai kebutuhan (mis. MIT) sebelum publish sebagai open source.
