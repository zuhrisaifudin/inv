# Rancangan Desain Antarmuka Lengkap (UI/UX)
## Sistem Pengelolaan Persediaan Material dan Gudang

**Versi:** 4.0 (Spesifikasi Implementasi Per Tampilan)
**Tanggal:** 27 Agustus 2026
**Dokumen acuan:** `rancangan-aplikasi-persediaan-gudang.md` dan `database.md`

Dokumen ini bukan hanya contoh visual. Setiap bagian merupakan kontrak tampilan bagi tim frontend, backend, QA, dan pemilik proses. Jika contoh pada wireframe berbeda dengan aturan tertulis di bawahnya, aturan tertulis dan dokumen acuan menjadi sumber yang lebih tinggi.

### Cara Membaca Spesifikasi

- **Tersedia di database** berarti field dapat diturunkan langsung dari tabel pada `database.md`.
- **Turunan** berarti nilai dihitung dari field yang tersedia, misalnya `available_quantity = on_hand_quantity - reserved_quantity - quarantined_quantity`.
- **Dependensi schema** berarti kebutuhan UI sudah ada pada rancangan aplikasi tetapi tabel/field pendukung belum tersedia. Tampilan boleh dirancang, namun implementasi produksi harus menunggu perubahan schema yang disebutkan.
- Seluruh contoh nama, nomor, kuantitas, dan nilai pada wireframe adalah data ilustrasi, bukan nilai default aplikasi.

---

## AUTENTIKASI DAN AKSES AKUN

### A.1 — Login

```text
+-------------------------------------------------------------+
| [Logo] INVENTORY SYSTEM                                     |
| Masuk untuk melanjutkan                                     |
| NIP / Email       [_______________________________]          |
| Password          [___________________________] [Lihat]      |
| [ ] Ingat perangkat ini                                     |
| [ MASUK ]                                                   |
| Lupa password?                                              |
+-------------------------------------------------------------+
```

| Aspek | Detail |
|---|---|
| Input | NIP/email, password, remember me. Password dapat ditampilkan sementara melalui tombol berlabel. |
| Validasi | Field wajib, format email bila memakai email, pesan kredensial generik agar tidak membocorkan keberadaan akun. |
| Status akun | Akun inactive ditolak dengan pesan aman dan arahan menghubungi admin. |
| Rate limit | Setelah batas gagal, tombol menampilkan countdown retry; jangan menghapus NIP/email. |
| Sukses | Arahkan ke intended URL bila masih berizin, selain itu dashboard role. Warehouse context pertama berasal dari assignment aktif. |
| Aksesibilitas | Fokus awal pada identitas; error summary diumumkan; Caps Lock warning; form dapat digunakan penuh dengan keyboard. |

### A.2 — Lupa dan Reset Password

Lupa password meminta email/NIP dan selalu memberi respons generik. Reset password menampilkan status token, identitas yang dimask, password baru, konfirmasi, indikator aturan password, serta state token invalid/expired/already-used. Setelah berhasil, sesi lain dapat dicabut sesuai policy dan pengguna kembali ke login.

> **Dependensi framework/infrastruktur:** state token reset hanya boleh ditampilkan bila aplikasi memakai penyimpanan token reset Laravel/Fortify yang memiliki expiry dan single-use semantics. Opsi **Cabut sesi lain** hanya boleh diaktifkan bila session driver menyimpan sesi secara server-side dan layanan dapat mencabut seluruh sesi pengguna selain sesi baru. `database.md` belum mendefinisikan token reset maupun session store; tanpa kontrak tersebut, UI hanya menyediakan tautan bantuan admin dan tidak mengklaim token atau sesi telah dicabut.

### A.3 — MFA dan Reautentikasi

Rancangan mengizinkan MFA diwajibkan bagi Super Admin/Accounting. Layar challenge bila diaktifkan berisi kode OTP/recovery code, resend/cooldown, rate limit, dan kembali ke login.

> **Dependensi schema:** `database.md` belum memiliki MFA secret, recovery code, metode faktor, verified time, atau trusted device. Layar setup/challenge tidak boleh diaktifkan sampai penyimpanan dan kebijakan MFA tersedia.

Untuk aksi sensitif—ubah password, perubahan role/permission, download dokumen sensitif—UI dapat meminta reautentikasi sebelum melanjutkan.

---

## TATA LETAK GLOBAL

```text
+---------------------------+--------------------------------------------------+
| [Logo] Inventory System   | [Gudang: MDN Medan ▼] [🔔 3] [Saifudin ▼]       |
+---------------------------+--------------------------------------------------+
| 📊 Dashboard              |                                                  |
| 📦 Master Data          ⌄ | ← CONTENT AREA (AJAX - Berganti tanpa reload)   |
|    ├ Katalog Material     |                                                  |
|    ├ Gudang & Lokasi      |                                                  |
|    ├ Subtipe Transaksi    |                                                  |
|    └ Klasifikasi & UOM    |                                                  |
| 📥 Inbound                |                                                  |
| 📤 Outbound             ⌄ |                                                  |
|    ├ Pengeluaran          |                                                  |
|    └ Retur Material       |                                                  |
| 🔄 In-Transit             |                                                  |
| 🛑 Quarantine             |                                                  |
| 📋 Stock Opname         ⌄ |                                                  |
|    ├ Stock Opname         |                                                  |
|    └ Inventarisasi        |                                                  |
| 📈 Reporting            ⌄ |                                                  |
|    ├ Laporan Mutasi       |                                                  |
|    ├ Laporan Rekonsiliasi |                                                  |
|    └ Dashboard Analytics  |                                                  |
| ⚙️  Settings            ⌄ |                                                  |
|    ├ Pengguna & Akses     |                                                  |
|    ├ Roles & Permissions  |                                                  |
|    └ Audit Log (target)   |  Hidden sampai permission + audit source tersedia|
+---------------------------+--------------------------------------------------+
```

### 0.1 Kontrak Application Shell

| Area | Perilaku wajib |
|---|---|
| Sidebar | Menu dan submenu dirender dari permission pengguna. Menu aktif mengikuti URL/deep-link, bukan hanya klik terakhir. Sidebar dapat diperkecil di desktop dan menjadi off-canvas di tablet/mobile. |
| Warehouse switcher | Menampilkan gudang aktif dari `user_warehouse_assignments` yang masih berlaku. Pengguna dengan permission lintas gudang dapat memilih semua gudang; pengguna biasa hanya melihat gudang yang ditugaskan. Perubahan gudang me-reset filter lokasi/material yang tidak lagi valid dan me-reload konten aktif. |
| Header | Menampilkan gudang aktif, notification/task bell, koneksi/server status saat request gagal, serta menu profil/logout. Baseline badge menghitung task approval; unread count hanya aktif setelah notification store tersedia. |
| Content area | Mendukung navigasi AJAX, tetapi URL, judul halaman, breadcrumb, tombol Back/Forward browser, dan refresh harus tetap benar. Deep-link yang tidak berizin menampilkan state 403, bukan halaman kosong. |
| Session | Lima menit sebelum sesi berakhir tampilkan peringatan. Jika sesi berakhir saat form terbuka, pertahankan draft lokal yang tidak sensitif dan arahkan login ulang; setelah login, data wajib dimuat ulang sebelum disimpan. |

**Aturan scope data:**

1. Permission menentukan fitur/aksi yang dapat digunakan.
2. Policy gudang menentukan record yang boleh dilihat atau diubah.
3. Scope yang sama wajib diterapkan pada DataTable, Select2, ekspor, detail URL, scan QR, dan download dokumen; menyembunyikan tombol saja tidak cukup.
4. Jika penugasan gudang berakhir saat layar masih terbuka, request berikutnya menghasilkan 403 dan UI menutup modal/form sensitif.

### 0.2 Responsive Layout

| Breakpoint | Perilaku |
|---|---|
| Desktop `≥1200px` | Sidebar tetap, filter satu baris bila cukup, DataTable lengkap, detail transaksi dapat memakai layout dua kolom. |
| Tablet `768–1199px` | Sidebar off-canvas, filter menjadi dua baris, detail menjadi tab/accordion, tombol aksi utama tetap sticky. Grid stock count dan scan QR dioptimalkan untuk sentuhan. |
| Mobile `<768px` | Tabel prioritas berubah menjadi kartu ringkas atau horizontal scroll dengan kolom identitas dan aksi tetap terlihat. Modal kompleks menjadi full-screen. Grafik tersusun satu kolom. Input angka menggunakan keypad numerik. |

Tidak ada aksi yang hanya dapat ditemukan melalui hover. Target sentuh minimum 44×44 px dan tombol destruktif harus memiliki label teks selain ikon.

### 0.3 Kontrak DataTable Server-Side

Semua daftar master, transaksi, saldo, sesi, laporan, pengguna, approval, dan audit menggunakan Yajra DataTables server-side.

- Pencarian dijalankan setelah jeda 300–500 ms dan dapat dihapus dengan satu tombol.
- Filter hanya mengirim field yang di-whitelist; filter gudang selalu dibatasi oleh policy pengguna.
- Default page size 25; pilihan 25/50/100. Default sort harus disebut pada masing-masing layar.
- URL menyimpan filter, sort, page, dan tab agar tampilan dapat dibagikan atau dipulihkan setelah kembali dari detail.
- Tombol **Reset** mengembalikan filter default role/gudang, bukan membuka seluruh data.
- Reload setelah mutasi mempertahankan page bila record masih ada.
- Header menampilkan jumlah hasil terfilter dan waktu pembaruan data.
- Bulk action hanya tersedia bila seluruh baris terpilih memenuhi permission dan status yang sama.

Kontrol **Saved View** menyimpan nama, filter, sort, page size, dan kolom terlihat per pengguna. Sampai penyimpanan preferensi tersedia, baseline hanya menyediakan URL yang dapat dibookmark/dibagikan; jangan mengklaim saved view lintas perangkat.

**State standar DataTable:**

| State | Tampilan |
|---|---|
| Initial loading | Skeleton header dan 5 baris; filter tetap terlihat tetapi action yang bergantung data dinonaktifkan. |
| Empty database | Ilustrasi ringkas, penjelasan record belum tersedia, dan CTA create bila pengguna berizin. |
| No filter result | Pesan “Tidak ada data sesuai filter” serta tombol Reset Filter. |
| Request error | Inline alert dengan correlation ID, tombol Coba Lagi, dan data lama diberi label “mungkin tidak terbaru”. |
| 403 | Panel akses ditolak; jangan mempertahankan isi data dari scope sebelumnya. |

### 0.4 Kontrak Form, Wizard, dan AJAX

- Form panjang memakai langkah **Header → Referensi & Dokumen → Item → Review → Submit**. Perpindahan langkah tidak mengirim transaksi bila validasi langkah aktif gagal.
- Tombol **Simpan Draft** dapat digunakan sebelum seluruh dokumen/item lengkap, tetapi field identitas minimum tiap layar tetap wajib.
- Autosave hanya untuk draft dan berjalan setelah perubahan berhenti; indikator menampilkan `Menyimpan…`, `Tersimpan HH:mm`, atau `Gagal—Coba lagi`.
- Error validasi JSON ditampilkan pada field dan diringkas di atas form. Fokus berpindah ke error pertama.
- Tombol mutasi terkunci selama request. Request approval/posting memakai idempotency key sehingga klik ulang tidak membuat posting ganda.
- Edit draft mengirim `lock_version`. Respons 409 menampilkan modal perbandingan “Data di server telah berubah”; pengguna harus reload sebelum mengedit lagi.
- Meninggalkan form dengan perubahan yang belum tersimpan memicu konfirmasi. Tidak ada konfirmasi jika autosave terakhir berhasil dan tidak ada perubahan baru.
- Field sistem—nomor transaksi, status, current stage, processed quantity setelah diproses, nilai turunan, QR token, dan timestamp—selalu read-only.

**Guard create dan submit transaksi:**

| Aksi | Permission baseline |
|---|---|
| Buat/edit draft sendiri | `transactions.create`, status `draft`/bagian revisi yang dibuka, ownership, dan warehouse policy. |
| Submit/submit ulang | `transactions.submit`, ownership, dokumen/item valid, dan warehouse policy. Semua tombol **Ajukan/Submit** pada wireframe tunduk pada guard ini. |
| Cancel | `transactions.cancel`, status sebelum posting, alasan wajib, dan policy. |

Seeder memberi Holder Material `transactions.create` tanpa `transactions.submit`. Baseline karena itu hanya mengizinkan Holder menyimpan draft sendiri; tombol Submit disembunyikan. Schema tidak memiliki draft owner assignment/handoff, sehingga UI tidak menawarkan **Serahkan Draft** seolah persisten. Agar Holder dapat memulai workflow bisnis, deployment harus memberikan `transactions.submit` kepada role tersebut atau terlebih dahulu menambah model penugasan/handoff yang eksplisit.

**Visibilitas harga/nilai baseline:** belum ada permission khusus nilai. Pengguna yang lolos policy record dan memiliki `transactions.view` melihat harga/nilai transaksi; pemegang `reports.view` melihat nilai laporan. Hasil scan QR umum tetap tidak mengembalikan harga sebagai prinsip minimisasi endpoint, bukan akibat permission nilai. Bila diperlukan pembatasan finansial, tambahkan permission seperti `inventory_value.view`/`reports.view_value` dan policy server sebelum UI mulai menyembunyikan kolom secara kondisional.

### 0.5 Status dan Tahap Workflow

Status utama dan tahap aktif harus tampil bersamaan. Badge status tidak boleh menggantikan informasi “menunggu siapa”.

| Status | Warna/ikon | Aksi umum yang tersedia |
|---|---|---|
| `draft` | Abu-abu / pensil | Edit, simpan, submit, atau **Batalkan** dengan `transactions.cancel`; transaksi tidak menawarkan hard delete. |
| `submitted` | Biru / kirim | Lihat; pembatalan hanya bila aturan proses mengizinkan dan wajib alasan. |
| `under_review` | Kuning / jam | Lihat timeline; actor tahap aktif dapat check/approve/reject/revision. |
| `revision_requested` | Jingga / ulang | Pembuat mengedit bagian yang dikembalikan lalu submit ulang. |
| `rejected` | Merah / silang | Read-only; tampilkan alasan dan pihak penolak. |
| `approved` | Hijau muda / centang | Read-only menunggu posting atau langkah operasional berikutnya. |
| `posted` | Hijau tua / buku besar | Kuantitas/nilai terkunci; tampilkan movement dan bukti posting. |
| `completed` | Hijau tua / selesai | Seluruh aktivitas operasional selesai. |
| `cancelled` | Abu gelap / batal | Read-only dengan alasan pembatalan. |
| `reversed` | Ungu / balik | Tautkan transaksi dan movement pembalik. |

### 0.6 Kontrak Error dan Feedback

| Kondisi | Respons UI |
|---|---|
| Stok tidak cukup | Tandai baris, tampilkan available terbaru, dan paksa refresh lot/label sebelum submit ulang. |
| Dokumen wajib kurang | Buka langkah dokumen dan tampilkan checklist tipe yang belum valid. |
| Upload gagal | Pertahankan draft dan metadata file lokal; sediakan Retry/Remove tanpa mengulang seluruh form. |
| Konflik versi | Modal 409 dengan waktu/pelaku perubahan terakhir dan CTA Muat Ulang. Jangan menimpa data server. |
| Approval bukan giliran | Tutup action panel, tampilkan 403, lalu refresh timeline dan current stage. |
| Posting ganda | Tampilkan hasil posting yang sudah ada dan arahkan ke movement; jangan menampilkan error generik. |
| QR invalid/nonaktif | Pesan aman tanpa data material, tombol Scan Lagi, dan correlation ID bila perlu. |
| Rate limited | Countdown sebelum retry untuk login, scan, atau ekspor. |
| Server/offline | Banner global; form transaksi tidak boleh submit. Draft lokal tidak boleh menyimpan dokumen atau data sensitif. |

Toast dipakai untuk hasil singkat yang tidak memerlukan keputusan. Error yang menghalangi proses wajib tampil inline atau dalam modal dan tidak boleh hilang otomatis.

### 0.7 Aksesibilitas dan Format Data

- Semua input memiliki label, deskripsi singkat bila konteks bisnis tidak jelas, serta relasi `aria-describedby` untuk error.
- Urutan tab mengikuti alur visual; modal menjebak fokus dan mengembalikannya ke pemicu saat ditutup.
- Status tidak dibedakan hanya dengan warna; selalu sertakan teks dan ikon.
- Format tanggal tampilan `DD-MM-YYYY`, waktu `HH:mm:ss`, dan timezone mengikuti `warehouses.timezone`; nilai backend tetap ISO-8601.
- Kuantitas mengikuti `uoms.decimal_precision`. Nilai Rupiah memakai pemisah ribuan dan minimum dua desimal pada detail finansial.
- KIMAP, nomor transaksi, nomor kartu, lot, dan token yang diizinkan dapat disalin melalui tombol khusus.

### 0.8 Banner Dependensi Schema

Jika layar memerlukan data yang belum dimodelkan, tampilkan catatan desain berikut pada spesifikasinya:

> **Dependensi schema:** kebutuhan ini berasal dari rancangan aplikasi, tetapi belum memiliki tabel/field pada `database.md`. Implementasi tidak boleh menyimpan data pada field serbaguna tanpa keputusan perubahan schema.

Banner ini dipakai antara lain untuk aturan dokumen wajib per subtype, penugasan Tim Inventarisasi, item adjustment, historical stock position, threshold low-stock, audit global, notification store, dan pusat ekspor.

### 0.9 Peta Navigasi dan Entry Point

| Area | Entry point utama | Deep-link terkait |
|---|---|---|
| Dashboard | Sidebar Dashboard | KPI/chart menuju daftar/laporan dengan filter diwariskan. |
| Master Data | Material; Gudang/Lokasi; Referensi; Organisasi/Posisi; Status Material | Detail material menuju saldo, lot, label, ledger, status history. |
| Inbound | Daftar Penerimaan | Task inbox menuju verifikasi/check/approval; detail menuju cetak QR. |
| Outbound | Daftar Pengeluaran; Retur | Task inbox menuju picking/check/approval/verifikasi nilai. |
| In-Transit | Daftar Transfer | Dashboard aging menuju detail; actor tujuan menuju receive/check. |
| Quarantine | Usulan; Quarantined; Written-Off | Posisi stok/material card menuju transaksi dan ledger. |
| Stock Opname | Stock Opname; Inventarisasi; Adjustment | Dashboard variance/task menuju grid/review/approval. |
| Reporting | Persediaan; Posisi Stok; Ledger Mutasi; Rekap Transaksi; Stock Count; Rekonsiliasi; Ekspor | Drill-down mempertahankan warehouse scope dan filter. |
| Settings | Pengguna/Akses; Roles/Permissions; Audit Log target (disabled baseline) | Profile/Password dan preferensi notifikasi berada di user menu. |
| Operasional QR | Quick action Scan QR atau tautan dari item/label | Hasil scan menuju kartu material, lot, label, atau proses aktif. |

Sidebar hanya memuat entry point yang sering digunakan. Detail, check, approval, print, timeline, dan hasil scan merupakan deep-link; tidak perlu menjadi menu utama terpisah.

---

## MENU 1: 📊 DASHBOARD
**Akses runtime:** `@canany(['dashboard.view_assigned_warehouse', 'dashboard.view_all_warehouses'])`; role tanpa salah satu permission tidak menerima menu/route/API Dashboard.

**Baseline seeder saat ini:** hanya Accounting dan Management memiliki `dashboard.view_all_warehouses`, sehingga hanya Dashboard 1A aktif. Dashboard operasional 1B/1C merupakan target deployment; sebelum dipakai, tambahkan `dashboard.view_assigned_warehouse` kepada role operasional yang disetujui—misalnya Pejabat User, Staf Gudang, Kepala Gudang, Persediaan, Holder Material, dan Tim Inventarisasi—beserta uji warehouse policy. UI tidak boleh menganggap “semua role” otomatis berizin.

### 1A — Dashboard Management & Accounting
```text
+═════════════════════════════════════════════════════════════════════════════+
║  DASHBOARD ANALITIK — PERTAMINA INVENTORY SYSTEM                           ║
╠═════════════════════════════════════════════════════════════════════════════╣
║ Filter: [Semua Region ▼]  [17 Gudang ▼]  [Periode: 2026 ▼]  [Terapkan]  ║
╠═══════════════╦════════════════════╦════════════════════╦═══════════════════╣
║   TOTAL NILAI  ║    NILAI MPS       ║    NILAI ABT       ║  TRANSAKSI BULAN ║
║ Rp45,5 Miliar ║ Rp30,0 Miliar      ║ Rp15,5 Miliar      ║   +127 Transaksi ║
╠═══════════════╩════════════════════╩════════════════════╩═══════════════════╣
║                                                                             ║
║  ┌──────────────────────────────┐  ┌──────────────────────────────────────┐ ║
║  │  🥧 Komposisi Nilai MPS      │  │  🥧 Komposisi Nilai ABT              │ ║
║  │                              │  │                                      │ ║
║  │     FM: 65%                  │  │     FM: 80%                          │ ║
║  │     SM: 20%                  │  │     SM: 10%                          │ ║
║  │     PDS: 10%                 │  │     PDS: 7%                          │ ║
║  │     DS:  5%                  │  │     DS:  3%                          │ ║
║  │   (per Gudang & Kategori)    │  │   (per Gudang & Kategori)            │ ║
║  └──────────────────────────────┘  └──────────────────────────────────────┘ ║
║                                                                             ║
║  ┌───────────────────────────────────────────────────────────────────────┐  ║
║  │  📈 Tren Saldo Akhir Bulanan (MPS vs ABT) — Line Chart               │  ║
║  │  Rp │                                   ╭────╮                        │  ║
║  │     │                          ╭────────╯    ╰───── MPS               │  ║
║  │     │               ╭──────────╯                                      │  ║
║  │     │──────╮────────╯                               ABT               │  ║
║  │     └──────────────────────────────────────────────────────── Bulan   │  ║
║  │      Jan   Feb   Mar   Apr   Mei   Jun   Jul   Agu                    │  ║
║  └───────────────────────────────────────────────────────────────────────┘  ║
║                                                                             ║
║  ┌───────────────────────────────────────────────────────────────────────┐  ║
║  │  📊 Perbandingan Saldo Tahunan Per Gudang — Bar Chart                 │  ║
║  │                                                                       │  ║
║  │      MDN    BGR    JKT    SBY    PKR    BPN   ...                    │  ║
║  │      ▐█▌    ▐█▌    ▐█▌    ▐█▌    ▐█▌    ▐█▌                         │  ║
║  └───────────────────────────────────────────────────────────────────────┘  ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 1B — Dashboard Kepala Gudang / Persediaan
```text
+═════════════════════════════════════════════════════════════════════════════+
║  DASHBOARD KEPALA GUDANG / FUNGSI PERSEDIAAN                                ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐  ║
║  │ 🔔 Pending       │  │ ✅ Approved Today │  │ 🔄 Draft (Belum Submit)  │  ║
║  │    Approval: 8   │  │      Trx: 15     │  │       Trx: 3             │  ║
║  └──────────────────┘  └──────────────────┘  └──────────────────────────┘  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  🔔 ANTRIAN TASK INBOX — Membutuhkan Tindakan Anda                          ║
║  [Cari no. transaksi.........] [Semua Tipe ▼] [Status Saya ▼]              ║
║                                                                             ║
║  No. Transaksi  │ Tipe          │ Gudang  │ Pemohon   │ Tahap       │ Aksi  ║
║  ───────────────┼───────────────┼─────────┼───────────┼─────────────┼────── ║
║  1101-MDN-0021  │ Penerimaan PO │ MDN     │ Budi S.   │ ⏳Ka.Gudang │[👁️]  ║
║  2202-BGR-0008  │ Pengeluaran   │ BGR     │ Susi A.   │ ⏳Persediaan│[👁️]  ║
║  3300-JKT-0003  │ Penyisihan    │ JKT     │ Anton R.  │ ⏳Ka.Gudang │[👁️]  ║
║  << Halaman 1 dari 3 >>                                                     ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 1C — Dashboard Staf Gudang (Operasional Harian)
```text
+═════════════════════════════════════════════════════════════════════════════+
║  DASHBOARD OPERASIONAL — GUDANG: MDN MEDAN                                  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  ⚡ AKSI CEPAT                                                               ║
║  [➕ Buat Penerimaan]  [📤 Buat Pengeluaran]  [🔄 Pemindahan]  [📷 SCAN QR] ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  📋 PEKERJAAN HARI INI                                                      ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ 1. DRAFT: Pengeluaran Operasional BGR-002 (3 item) → [Lanjutkan ✏️] │ ║
║  │ 2. DRAFT: Stock Opname RAK-A1 (25 Material) → [Buka Form Input 🔢]  │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  ⚠️ PERINGATAN STOK KRITIS (aktif jika threshold telah dikonfigurasi)        ║
║  Material: Valve Gate 4" SS │ Sisa: 2 EA │ Lokasi: RAK-A1  [Lihat 👁️]     ║
║  Material: Pipa Seamless 8" │ Sisa: 0 MTR│ Lokasi: ZONE-B  [Lihat 👁️]     ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 1D — Kontrak Detail Dashboard dan Notification Center

#### Dashboard Management & Accounting

**Tujuan:** memberikan gambaran kuantitas, nilai, komposisi, dan tren lintas gudang tanpa menyediakan aksi mutasi stok.

| Aspek | Spesifikasi |
|---|---|
| Akses | `dashboard.view_all_warehouses`; Management read-only, Accounting dapat drill-down ke laporan nilai. |
| Sumber | `stock_balances` untuk posisi saat ini, `inventory_movements` untuk tren historis, `materials`, klasifikasi/kategori, `material_status_assignments`, dan `warehouses`. |
| Filter global | Periode, region, gudang multi-select, klasifikasi, kategori, status material FM/SM/PDS/DS, tipe/subtype transaksi. Default: tahun berjalan dan seluruh scope pengguna. |
| KPI | Total kuantitas, total nilai, nilai MPS, nilai ABT, penerimaan periode, pengeluaran periode, nilai in-transit, nilai quarantined, approval tertunda, dan variance opname belum selesai. Setiap kartu menampilkan nilai, perubahan terhadap periode sebelumnya, waktu refresh, dan dapat membuka daftar terfilter. |
| Grafik | Pie komposisi MPS dan ABT menurut status/gudang/kategori; pie penerimaan vs pengeluaran per subklasifikasi; line saldo akhir bulanan MPS vs ABT; bar perbandingan tahunan per gudang; aging approval; usia transfer in-transit. |
| Perhitungan | Nilai posisi saat ini = `on_hand_quantity × average_unit_cost`. Saldo akhir historis harus dihitung kumulatif dari ledger, bukan sekadar menjumlahkan `amount_delta` pada bulan terpilih. |

**Interaksi dan state:**

- Klik legend menyembunyikan/menampilkan seri tanpa mengubah filter global.
- Klik segmen atau titik grafik membuka laporan sumber dengan filter yang sama.
- Filter tidak otomatis dieksekusi sampai **Terapkan**, kecuali pencarian gudang pada selector.
- Saat satu widget gagal, widget lain tetap tampil; widget gagal menampilkan Retry dan correlation ID.
- Empty state membedakan “belum ada transaksi” dari “tidak ada hasil sesuai filter”.
- Pada mobile, KPI menjadi dua kolom dan seluruh grafik menjadi satu kolom dengan tinggi minimum 280 px.

#### Dashboard Kepala Gudang / Fungsi Persediaan

**Tujuan:** pusat pekerjaan yang membutuhkan keputusan pengguna pada scope gudang/region yang ditugaskan.

- Sumber task: `transaction_approvals` berkeputusan `pending`, dicocokkan dengan `inventory_transactions.current_stage`, `assigned_user_id`/`required_role`, dan warehouse policy.
- KPI minimum baseline: pending milik saya, usia task tertua, revision returned, approved hari ini, draft milik saya, dan sesi stock count completed yang belum dapat dilanjutkan.
- Filter inbox baseline: pencarian nomor/referensi, tipe/subtype, gudang, stage, rentang usia, dan assignment (`Milik Saya`/`Role Saya`).
- Kolom: nomor, tipe/subtype, gudang asal/tujuan, pemohon, current stage, assigned role/user, submitted time, usia berjalan, status utama, dan aksi.
- Default sort baseline: submitted time terlama, lalu nomor transaksi ascending.
- Aksi **Review** membuka detail/modal approval; setelah keputusan berhasil, baris hilang dari inbox tanpa memindahkan page.
- Jika task sudah diproses orang lain, tampilkan informasi pelaku/waktu terbaru dan refresh inbox; jangan mengulang keputusan.

#### Dashboard Staf Gudang

**Tujuan:** mengarahkan pekerjaan fisik harian pada satu gudang aktif.

- Quick action hanya tampil sesuai permission: buat transaksi, pemeriksaan inbound, picking outbound, dispatch/receive transfer, scan QR, input stock count, dan cetak label.
- Daftar pekerjaan menggabungkan draft milik pengguna, transaksi yang sedang menunggu pemeriksaan fisik, label penerimaan belum dicetak, transfer belum diterima, dan sesi hitung yang ditugaskan.
- Setiap kartu pekerjaan menampilkan prioritas, nomor transaksi/sesi, jumlah item, lokasi, batas waktu, tahap, dan CTA berikutnya.
- Perubahan warehouse switcher meminta konfirmasi bila ada scan/grid yang belum tersimpan.

> **Dependensi schema:** `database.md` belum memiliki reorder point/minimum stock. Widget low-stock hanya boleh diaktifkan setelah threshold material per gudang dimodelkan; tanpa itu, tampilkan pekerjaan operasional lain dan jangan menebak batas kritis.

> **Dependensi konfigurasi SLA:** belum ada threshold per tipe/subtype/stage, kalender kerja, timezone per aturan, maupun masa berlaku konfigurasi SLA. Baseline hanya menampilkan **usia berjalan** dari timestamp transaksi/approval. Badge overdue, due-status filter, pengurutan overdue, deadline, dan alert SLA harus disembunyikan sampai tersedia konfigurasi deterministik yang sekurangnya memuat stage, durasi, kalender, timezone, dan effective period.

#### Notification Center (Desain Target)

Sebelum schema notifikasi tersedia, notification bell berfungsi sebagai **Task Inbox real-time** yang berisi pending approval dari `transaction_approvals` dan draft milik pengguna dari `inventory_transactions`; belum ada read/unread state. Setelah notification store tersedia, bell membuka drawer di desktop dan halaman penuh di mobile dengan kontrak berikut.

| Elemen | Detail |
|---|---|
| Tab | Semua, Belum Dibaca, Membutuhkan Aksi, Sistem/Ekspor. |
| Isi kartu | Ikon jenis, judul, ringkasan aman, nomor transaksi/sesi, gudang, waktu relatif + timestamp, status read/unread, dan deep-link. |
| Pemicu | Transaksi diajukan, approval ditugaskan, revision, reject, approval selesai, QR belum dicetak, reservation mendekati expired, variance opname, dan ekspor selesai. Pemicu transfer melewati SLA baru aktif setelah konfigurasi SLA tersedia. |
| Aksi | Buka, Tandai Dibaca, Tandai Semua Dibaca. Notifikasi task tidak dianggap selesai hanya karena dibaca. |
| Error | Deep-link yang sudah tidak berizin menampilkan 403; notifikasi tetap dapat ditandai dibaca tanpa membuka data. |

> **Dependensi schema:** kebutuhan notifikasi tersedia pada rancangan, tetapi tabel penyimpanan notifikasi dan preferensi email belum didefinisikan di `database.md`.

---

## MENU 2: 📦 MASTER DATA
**Akses:** `@canany(['master_data.view', 'master_data.edit'])` | Edit: `@can('master_data.edit')`

Gate read memakai union secara eksplisit karena Spatie tidak membuat `master_data.edit` mewarisi `master_data.view`. Dengan kontrak ini role Persediaan pada seeder saat ini dapat membuka data yang memang boleh diedit; actor yang hanya memiliki `master_data.view` tetap read-only.

### 2.1 — Katalog Material
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📦 KATALOG MATERIAL (BANK DATA)                                            ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [Tab: Katalog Material ✔] [Tab: Klasifikasi] [Tab: Kategori] [Tab: UOM]    ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  @can('master_data.create') [➕ Tambah Material]  @endcan                   ║
║  [📥 Import Excel]  [📤 Export Template]                                    ║
║  [Cari KIMAP atau Nama Barang.......]  [Klasifikasi: Semua ▼] [Status ▼]   ║
╠══════════╦════════════════════════╦════════════╦══════════╦═════╦══════════╣
║  KIMAP   ║  Nama Barang           ║ Klasifikasi║ Kategori ║ UOM ║  Aksi    ║
╠══════════╬════════════════════════╬════════════╬══════════╬═════╬══════════╣
║ 10001928 ║ VALVE GATE 4" SS 316   ║ 🟦 MPS     ║ Tubular  ║ EA  ║ [✏️][Nonaktifkan] ║
║ 10002133 ║ PIPA SEAMLESS 8"       ║ 🟦 MPS     ║ Tubular  ║ MTR ║ [✏️][Nonaktifkan] ║
║ 90001123 ║ LAPTOP ASUS ROG G15    ║ 🟩 ABT     ║ Elektron ║ SET ║ [✏️][Nonaktifkan] ║
╚══════════╩════════════════════════╩════════════╩══════════╩═════╩══════════╝
```
> Klik `[➕ Tambah Material]` → Modal AJAX:
```text
┌─────────────────── FORM TAMBAH MATERIAL ─────────────────┐
│ KIMAP (No. Katalog) : [___________________________]        │
│ Nama Barang         : [___________________________]        │
│ Klasifikasi         : [MPS ▼ ] (Select2 AJAX)             │
│ Kategori            : [Tubular Goods ▼] (Select2 AJAX)    │
│ UOM Dasar           : [EA ▼ ]                             │
│ Deskripsi           : [Textarea...]                        │
│ Serialized? (berSN) : [○ Ya  ● Tidak]                     │
│                                                           │
│              [Batal]   [💾 Simpan Material]               │
└───────────────────────────────────────────────────────────┘
```

### 2.2 — Gudang & Lokasi Penyimpanan
```text
+═════════════════════════════════════════════════════════════════════════════+
║  🏢 MASTER GUDANG & LOKASI PENYIMPANAN (17 Gudang)                         ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  @can('master_data.create') [➕ Tambah Gudang] @endcan                      ║
╠═════╦════════════════════════╦════════╦══════════════╦════════╦════════════╣
║ Kode║  Nama Gudang           ║ Region ║ Zona Waktu   ║ Status ║ Aksi       ║
╠═════╬════════════════════════╬════════╬══════════════╬════════╬════════════╣
║ ⊕MDN║ Gudang Utama Medan     ║ REG-1  ║ Asia/Jakarta ║ 🟢Aktif║ [✏️][⚙️Rak]║
║ ─── Lokasi Rak/Bin (expand dengan klik ⊕): ────────────────────────────── ║
║     │ [➕ Tambah Rak/Bin] [Kode: RAK-A1 | Tipe: Rack] [✏️][Nonaktifkan]  ║
║     │                    [Kode: ZONE-B  | Tipe: Zone] [✏️][Nonaktifkan]  ║
║     │                    [Kode: QRN-01  | Tipe: Quarantine] [✏️][Nonaktifkan] ║
║ ⊕BGR║ Gudang Utama Bogor     ║ REG-3  ║ Asia/Jakarta ║ 🟢Aktif║ [✏️][⚙️Rak]║
║ ⊕JKT║ Gudang Pusat Jakarta   ║ REG-2  ║ Asia/Jakarta ║ 🟢Aktif║ [✏️][⚙️Rak]║
╚═════╩════════════════════════╩════════╩══════════════╩════════╩════════════╝
```

### 2.3 — Klasifikasi, Kategori & UOM (Referensi)
```text
+═════════════════════════════════════════════════════════════════════════════+
║  [Tab: Klasifikasi] [Tab: Kategori Material] [Tab: UOM] [Tab: Subtipe Trx] ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  TAB: KLASIFIKASI MATERIAL                                                  ║
║  Kode │ Nama Klasifikasi               │ Urutan │ Status       │ Aksi     ║
║  ─────┼────────────────────────────────┼────────┼──────────────┼───────── ║
║  MPS  │ Material Persediaan            │   10   │ 🟢 Aktif     │ [✏️]     ║
║  ABT  │ Aset Belum Tercatat            │   20   │ 🟢 Aktif     │ [✏️]     ║
║  SKL  │ Material Skala                 │   30   │ ⚪ Nonaktif  │ [✏️]     ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  TAB: SUBTIPE TRANSAKSI                                                   ║
║  Kode │ Nama Subtipe             │ Main Type │ Referensi? │ Status │ Aksi  ║
║  ─────┼───────────────────────────┼───────────┼────────────┼────────┼────── ║
║  1101 │ Penerimaan Pembelian     │ receipt   │ Ya         │ Aktif  │ [✏️]  ║
║  2202 │ Pengeluaran Operasional  │ issue     │ Ya         │ Aktif  │ [✏️]  ║
║  3300 │ Penyisihan / Write-Off   │ write_off │ Ya         │ Aktif  │ [✏️]  ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 2.4 — Detail Material, Form, dan Riwayat Status per Gudang

#### Kontrak Daftar Katalog

| Aspek | Detail |
|---|---|
| Sumber | `materials` + `material_classifications` + `material_categories` + `uoms`. |
| Filter | KIMAP/nama, klasifikasi, kategori, UOM, aktif/nonaktif, serialized. |
| Kolom | KIMAP, nama, klasifikasi, kategori, UOM dasar, serialized, status aktif, updated time, dan aksi. |
| Aksi | Tambah, detail, edit, aktifkan/nonaktifkan, import, unduh template. Material yang telah direferensikan tidak dihapus fisik. |

Form Tambah/Edit mencakup seluruh field `materials`: KIMAP unik maksimum 30 karakter, nama, classification, category, base UOM, description, serialized, dan active. Selector hanya menampilkan referensi aktif untuk data baru; detail historis tetap menampilkan referensi yang kemudian nonaktif.

#### Halaman Detail Material

Header menampilkan identitas material dan tab:

1. **Ringkasan** — KIMAP, nama, klasifikasi, kategori, UOM, deskripsi, serialized, status aktif, timestamps.
2. **Status per Gudang** — status efektif FM/SM/PDS/DS, periode, alasan, dan penetap dari `material_status_assignments`.
3. **Saldo Saat Ini** — on-hand/reserved/in-transit/quarantined/available per gudang-lokasi-lot.
4. **Lot & Label** — lot/perolehan, kartu, status label, dan sisa qty.
5. **Ledger** — movement immutable untuk material tersebut.

Form penetapan status meminta gudang, status, berlaku dari/sampai, dan alasan. Server harus menolak periode tumpang tindih untuk material–gudang yang sama walaupun constraint tersebut belum tersedia pada database.

### 2.5 — Import Material

Alur import: **Unduh Template → Upload XLSX → Validasi Preview → Perbaiki/Upload Ulang → Konfirmasi Import**.

- Template berisi KIMAP, nama, kode klasifikasi, kode kategori, kode UOM, deskripsi, dan serialized.
- Preview menampilkan jumlah valid/invalid serta alasan per baris: duplikat file/database, kode referensi tidak ada/nonaktif, atau format salah.
- Import hanya aktif bila seluruh baris valid dan disimpan dalam satu database transaction.
- Konflik KIMAP yang muncul setelah preview menghasilkan 409 dan mengharuskan validasi ulang.

> **Dependensi schema:** belum ada tabel batch/import history. UI tidak menyediakan resume, replay, atau histori import persisten.

### 2.6 — Form Gudang dan Pohon Lokasi

#### Form Gudang

Field: kode unik maksimum 10 karakter, nama, region code, alamat, timezone IANA, dan active. Penonaktifan meminta konfirmasi dampak terhadap assignment dan proses aktif; data historis tidak dihapus.

#### Form Lokasi

Field: gudang read-only dari konteks, kode unik per gudang, nama, tipe (`zone`, `rack`, `bin`, `staging`, `transit`, `quarantine`), parent opsional, dan active.

- Parent harus berada di gudang sama, bukan dirinya sendiri, dan tidak membentuk siklus.
- Pohon memuat anak secara lazy; error pada satu node tidak menutup node lain.
- Lokasi yang memiliki saldo/transaksi tidak dihapus; gunakan nonaktif.
- Desktop memakai tree/table; mobile memakai daftar bertingkat dan detail lokasi terpisah.

### 2.7 — Kontrak Empat Tab Referensi

| Tab | Kolom daftar | Field form dan validasi |
|---|---|---|
| Klasifikasi | Kode, nama, sort order, active, updated time | Kode unik, nama wajib, urutan integer, active. |
| Kategori | Kode, nama, sort order, active, updated time | Kode unik, nama wajib, urutan integer, active. |
| UOM | Kode, nama, decimal precision, active, updated time | Kode unik maks. 10, nama wajib, presisi unsigned tiny integer. |
| Subtype | Kode, main type, nama, description, requires reference, active | Main type hanya `receipt`, `issue`, `return`, `transfer`, `write_off`, `adjustment`. |

Referensi nonaktif tidak muncul pada selector transaksi/master baru, tetapi tetap dirender pada record lama. Hapus fisik tidak tersedia bila sudah direferensikan.

> **Dependensi schema:** tab subtype hanya dapat menyimpan `requires_reference`; aturan jenis dokumen wajib belum tersedia dan tidak boleh disamarkan sebagai kolom yang tersimpan pada `transaction_subtypes`.

### 2.8 — Fungsi Organisasi, Posisi, dan Penugasan Jabatan

Sediakan tab **Fungsi Organisasi**, **Posisi**, dan panel **Riwayat Jabatan** pada detail pengguna.

| Tampilan | Data/aksi |
|---|---|
| Fungsi Organisasi | Kode, nama, parent, active; create/edit/nonaktif; parent tidak boleh membentuk siklus. |
| Posisi | Kode, nama, fungsi, active; fungsi wajib aktif untuk posisi baru. |
| Penugasan Jabatan | User, posisi/fungsi, valid from/until, primary; pertahankan histori dan cegah lebih dari satu primary aktif pada periode sama. |

Sumber: `organizational_functions`, `positions`, `user_position_assignments`, dan `users`. Struktur menjadi tree di desktop dan daftar bertingkat di mobile.

---

## MENU 3: 📥 INBOUND (PENERIMAAN MATERIAL)
**Akses:** `@can('transactions.view')` | Buat: `@can('transactions.create')`

### 3.1 — Daftar Transaksi Penerimaan
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📥 PENERIMAAN MATERIAL (INBOUND)                                           ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  @can('transactions.create') [➕ Buat Penerimaan Baru] @endcan               ║
║  Filter: [Tanggal: 01-08-2026 s/d 25-08-2026 📅] [Subtipe ▼] [Status ▼]   ║
╠══════════════════╦══════════╦═══════════╦══════════════╦════════════════════╣
║  No. Transaksi   ║ Subtipe  ║ Tgl Trx   ║ Pemohon      ║ Status      │ Aksi ║
╠══════════════════╬══════════╬═══════════╬══════════════╬═════════════╪══════╣
║ 1101-MDN-202608-0│ Perm.PO  ║ 25-08-2026║ Budi S.      ║ 🟢 APPROVED │[👁️] ║
║ 1103-BGR-202608-0│ Penerimaan Langsung║24-08-2026║ Susi A.  ║ 🟠 ⏳Gudang  │[👁️] ║
║ 1104-JKT-202608-0│ Penerimaan Lainnya │23-08-2026║ Anton R. ║ ⚪ DRAFT     │[✏️] ║
╚══════════════════╩══════════╩═══════════╩══════════════╩═════════════╩══════╝
```

### 3.2 — Form Buat / Edit Draft Penerimaan
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📥 FORMULIR PENERIMAAN MATERIAL                              [DRAFT]       ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  HEADER TRANSAKSI                                                           ║
║  ┌──────────────────────────────────────────────────────────────────────┐  ║
║  │ Subtipe Transaksi : [1101 - Penerimaan Pembelian Rutin ▼]           │  ║
║  │ Gudang Penerima   : [MDN - Gudang Utama Medan (readonly)]           │  ║
║  │ Tanggal Transaksi : [25 Agustus 2026 📅]                            │  ║
║  │ No. Referensi PO  : [PO-2026-12345_____________]                    │  ║
║  │ Keterangan        : [Penerimaan rutin Q3 2026__________]            │  ║
║  │ Upload Dokumen    : [📎 Upload PO (PDF, max 10MB)] ← *WAJIB*        │  ║
║  │                    [📎 Upload DO/Surat Jalan] ← *WAJIB*             │  ║
║  │                    [📎 Upload BAP] ← *WAJIB sesuai rule subtype*     │  ║
║  │                    [📎 Upload BAST] ← *WAJIB sesuai rule subtype*    │  ║
║  └──────────────────────────────────────────────────────────────────────┘  ║
║                                                                             ║
║  DETAIL ITEM MATERIAL                                                       ║
║  ┌────────────────────────────────────────────────────────────────────────┐ ║
║  │ Material (KIMAP/Nama) │ Lokasi Rak/Bin │ Qty Terima │ UOM │ Hapus     │ ║
║  │ ──────────────────────┼───────────────┼────────────┼─────┼────────── │ ║
║  │ [Select2 AJAX... ▼]   │ [RAK-A1 ▼]    │ [  50  ]   │ EA  │ [🗑️]      │ ║
║  │ [Select2 AJAX... ▼]   │ [ZONE-B ▼]    │ [  10  ]   │ MTR │ [🗑️]      │ ║
║  │                                                                        │ ║
║  │ [➕ Tambah Baris Material]                     [📷 Scan QR Material]  │ ║
║  └────────────────────────────────────────────────────────────────────────┘ ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [← Batal]                     [💾 Simpan Draft]  [🚀 Ajukan Approval →]  ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 3.3 — Detail Transaksi & Workflow Approval (View Mode)
```text
+═════════════════════════════════════════════════════════════════════════════+
║  👁️  DETAIL PENERIMAAN 1101-MDN-202608-0021                                ║
╠══════════════════════════════════════╦══════════════════════════════════════╣
║  INFO TRANSAKSI                      ║  HISTORI WORKFLOW APPROVAL           ║
║  No. Transaksi: 1101-MDN-202608-0021 ║  ─────────────────────────────────  ║
║  Subtipe: Penerimaan Pembelian       ║  ✅ Dibuat Draft — Budi S. 08:00     ║
║  Gudang : MDN - Medan                ║  ✅ Verifikasi Persediaan — Dewi     ║
║  Tgl Trx: 25 Agustus 2026           ║  ✅ Cek Fisik Gudang — Agus 09:30    ║
║  Status : 🔵 POSTED                 ║  ✅ CHECKED (Stempel) — Staf         ║
║                                      ║  ✅ Approve Ka.Gudang — Hendra 10:15 ║
║  LAMPIRAN DOKUMEN                    ║  ✅ APPROVED (Stempel) — Final       ║
║  📎 PO-2026-12345.pdf [👁️ Lihat]    ║  ✅ Posted — Sistem 10:16            ║
║  📎 DO-Surat-Jalan.pdf [👁️ Lihat]   ║                                      ║
╠══════════════════════════════════════╩══════════════════════════════════════╣
║  DETAIL ITEM YANG DITERIMA                                                  ║
║  KIMAP    │ Nama Barang          │ Qty Terima │ UOM │ Lokasi  │ Label QR   ║
║  ─────────┼──────────────────────┼────────────┼─────┼─────────┼─────────── ║
║  10001928 │ VALVE GATE 4" SS 316 │ 50         │ EA  │ RAK-A1  │ [🏷️ Print] ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  MODE READ-ONLY POSTED: [📄 Cetak Form] [🏷️ Cetak QR] [👁️ Lihat Ledger]   ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 3.4 — Spesifikasi Implementasi Daftar dan Wizard Penerimaan

**Workflow baseline:** User membuat → Fungsi Persediaan memverifikasi informasi/harga/dokumen → Staf Gudang memeriksa fisik → Kepala Gudang menyetujui → sistem posting → Staf Gudang mencetak QR.

#### Daftar Penerimaan

| Aspek | Detail |
|---|---|
| Scope | `transactions.view` dan warehouse policy pada gudang operasional/penerima. |
| Subtype | Mapping bisnis target untuk penerimaan langsung adalah 1101/1103/1104/1107. Runtime memuat row aktif `main_type='receipt'` lalu memvalidasi kode yang dikonfigurasi; 1102/1105/1106 hanya diarahkan ke Retur bila row aktifnya ber-`main_type='return'`, dan 2204 ke In-Transit bila aktif sebagai `transfer`. Kode yang hilang/salah tipe menonaktifkan CTA terkait dengan error konfigurasi; primary key tidak pernah di-hard-code. |
| Filter | Gudang, rentang tanggal, subtype, status utama, current stage, pemohon, KIMAP/nama material, nomor transaksi/referensi, dan indikator QR belum dicetak. |
| Kolom | Nomor, subtype, tanggal, gudang, pemohon, jumlah item, nilai, status, tahap/pemegang aksi, submitted/posted time, status cetak QR, dan aksi. |
| Default sort | `transaction_date` terbaru, lalu nomor transaksi. Saved view dan filter tersimpan di URL. |
| Aksi | Buat, lihat, lanjutkan draft/revisi, review sesuai role, preview/cetak formulir, dan cetak QR setelah posting. |

#### Wizard Penerimaan

| Langkah | Field dan perilaku |
|---|---|
| 1. Header | Nomor transaksi read-only setelah draft dibuat; subtype; gudang penerima sesuai scope; tanggal; `reference_text`; `explanation`. |
| 2. Dokumen | Checklist PO, DO, BAP, dan BAST sesuai rule subtype. Tampilkan tipe, nama, ukuran, MIME, checksum status, versi, pengunggah, progress, retry/remove. |
| 3. Item | Material aktif via Select2, UOM dasar read-only, qty diterima, harga satuan terlihat menurut kontrak nilai global dan hanya editable pada tahap verifikasi Persediaan, total turunan, lokasi tujuan usulan, nomor kartu/catatan bila tahapnya mengizinkan. |
| 4. Review | Ringkasan header, dokumen kurang, item invalid/duplikat, total kuantitas/nilai, dan tujuan submit berikutnya. |

Field utama bersumber dari `inventory_transactions`, `inventory_transaction_items`, dan `transaction_documents`. Pada schema saat ini, gudang operasional penerimaan tetap menggunakan field warehouse yang diwajibkan oleh `inventory_transactions`; label UI harus konsisten dengan keputusan implementasi backend tentang semantik `source_warehouse_id` untuk receipt.

> **Dependensi schema:** aturan tipe dokumen wajib per subtype belum mempunyai tabel konfigurasi. `transaction_subtypes.requires_reference` hanya menandai kebutuhan referensi, bukan daftar PO/DO/BAP/BAST. `transaction_references` juga disebut dalam rancangan aplikasi tetapi belum memiliki migration; baseline yang tersedia adalah `inventory_transactions.reference_text`.

**Validasi:**

- Material harus aktif saat dipilih; record historis tetap menampilkan material yang kemudian nonaktif.
- UOM mengikuti `materials.base_uom_id` sampai tersedia master konversi UOM.
- Kuantitas harus positif dan mengikuti `uoms.decimal_precision`.
- Kombinasi material/lokasi yang sama boleh digabung atau ditolak secara konsisten; jangan diam-diam membuat baris duplikat.
- Submit ditolak bila tidak ada item, dokumen wajib kurang, upload belum selesai, atau ada total tidak valid.
- Draft/revision mengirim `lock_version`; konflik 409 memaksa reload.

### 3.5 — Verifikasi Persediaan dan Check Fisik Gudang

#### Panel Verifikasi Fungsi Persediaan

Panel read-only menampilkan header, klasifikasi/kategori/status material per gudang, seluruh versi dokumen, harga satuan, total per item, total transaksi, serta catatan pemohon. Aksi: **Minta Revisi**, **Tolak**, atau **Loloskan ke Pemeriksaan Gudang**. Reject dan revision wajib catatan; revision juga memilih tahap tujuan.

#### Layar Check Fisik Staf Gudang

```text
+--------------------------------------------------------------------------------+
| 📷 CHECK FISIK PENERIMAAN — 1101-MDN-202608-000021                            |
| Status: UNDER REVIEW | Tahap: Pemeriksaan Staf Gudang                         |
+--------------------------------------------------------------------------------+
| KIMAP | Qty disetujui | Qty fisik | Lot/perolehan | Kartu | Lokasi | Catatan |
| ...   | 50 EA         | [50 EA]   | [2026-08]      | [...] | [A1 ▼] | [...]  |
+--------------------------------------------------------------------------------+
| [Simpan Pemeriksaan]                               [BUBUHKAN CHECKED]          |
+--------------------------------------------------------------------------------+
```

- Header, referensi, dokumen, dan harga tidak dapat diedit pada tahap ini.
- Setiap item wajib memiliki `processed_quantity`, periode/lot, `card_number`, lokasi tujuan, dan hasil pemeriksaan.
- Scan QR memvalidasi token ke server; hasil scan tidak pernah menampilkan harga.
- Aksi CHECKED menambah record approval/timeline dan watermark, bukan mengubahnya menjadi approval Kepala Gudang.
- Pada tablet, item menjadi kartu per material dengan CTA sticky dan keypad numerik.

### 3.6 — Approval Kepala Gudang, Posting, dan QR

Kepala Gudang membandingkan qty diajukan, qty hasil pemeriksaan, lokasi, kartu, catatan, dokumen, dan watermark CHECKED. Setelah APPROVED, layanan posting harus membentuk `stock_lots`, `inventory_movements`, memperbarui `stock_balances`, dan menyiapkan `stock_labels` secara atomik.

Detail setelah posting menampilkan:

- ID/nomor transaksi, status `posted`/`completed`, `posted_at`, dan pelaku posting.
- Item, lot, periode perolehan, kartu, label, lokasi, kuantitas, harga, dan nilai.
- Movement inbound yang terbentuk dan saldo hasil posting.
- Timeline dari `transaction_approvals` dan `transaction_histories`.
- Dokumen beserta version history; setelah posting hanya versi baru yang boleh ditambahkan.
- CTA preview/cetak PDF resmi dan cetak label QR.

Cetak label menampilkan card number, KIMAP/nama, lot/periode, lokasi, qty label, dan QR token yang tidak memuat harga. Baseline hanya mengizinkan cetak awal; cetak ulang/void tetap disabled sampai alasan, actor/time, dan audit event dapat disimpan. Label void/consumed legacy tidak dapat dipakai untuk transaksi baru.

---

## MENU 4: 📤 OUTBOUND (PENGELUARAN & RETUR)

### 4.1 — Pengeluaran Material (Tampilan & Form)
Form hampir identik dengan Inbound, namun dengan perbedaan:
- Menampilkan **sisa stok tersedia** secara real-time dari `stock_balances`.
- Validasi otomatis: jika Qty Diminta > Stok Tersedia, input berubah **merah** & submit di-*disable*.
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📤 FORMULIR PENGELUARAN MATERIAL                              [DRAFT]      ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Subtipe   : [2202 - Pengeluaran Operasional ▼]                             ║
║  Gudang    : [MDN - Gudang Utama Medan (readonly)]                          ║
║  Upload SPK: [📎 Upload SPK (PDF, max 10MB)] ← *WAJIB*                     ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  DETAIL ITEM MATERIAL                                                       ║
║  Material (KIMAP)   │ Stok Tersedia │ Qty Diminta │ UOM │ Hapus             ║
║  ───────────────────┼───────────────┼─────────────┼─────┼────────────────  ║
║  Valve Gate 4" SS   │  50 EA        │ [  5  ]     │ EA  │ [🗑️]             ║
║  Pipa Seamless 8"   │   2 MTR       │ [  5  ] ⚠️  │ MTR │ [🗑️]             ║
║                                                                             ║
║  ⚠️ PERINGATAN: Qty melebihi stok tersedia (2 MTR)! Submit dinonaktifkan.  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [← Batal]                     [💾 Simpan Draft]  [🚀 Ajukan Approval →]  ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 4.2 — Retur Material

Retur merupakan alur penerimaan kembali dan **bukan** salinan form pengeluaran. Pengguna memilih transaksi pengeluaran asal bila tersedia; material, KIMAP, UOM, harga, dan kuantitas maksimum diturunkan sebagai read-only dari transaksi tersebut. Mapping bisnis target retur adalah 1102/1105/1106, tetapi runtime hanya menerima row aktif dengan `main_type='return'`; bila konfigurasi kode hilang/salah tipe, CTA subtype terkait dinonaktifkan. Retur tanpa referensi lama memerlukan validasi harga oleh Fungsi Persediaan sebelum posting final.

### 4.3 — Daftar dan Wizard Permintaan Pengeluaran

**Workflow baseline:** User meminta → reservation dibuat saat submit → Staf Gudang scan/menyiapkan → Kepala Gudang approve dan posting kuantitas → Fungsi Persediaan memverifikasi harga/nilai → reservation ditutup → completed.

#### Daftar Pengeluaran

| Aspek | Detail |
|---|---|
| Subtype | Mapping bisnis target pengeluaran adalah 2201/2202/2203/2205/2206. Runtime hanya menampilkan row `transaction_subtypes.is_active=true` dengan `main_type='issue'` yang cocok konfigurasi. Kode 2204 hanya masuk menu Pemindahan bila tervalidasi sebagai `transfer`; kode hilang/salah tipe menonaktifkan CTA dan primary key tidak di-hard-code. |
| Filter | Gudang asal, periode, subtype, status, current stage, material/KIMAP, klasifikasi, kategori, status material, status reservation, dan reservation expiry. |
| Kolom | Nomor, subtype, pemohon, gudang, jumlah item, requested/processed qty, nilai, status, tahap, status/expiry reservation, dan aksi. |
| Aksi | Buat, lanjut draft/revisi, lihat, penyiapan gudang, review/approval, cancel sebelum posting, dan reversal bila berizin. |

#### Wizard Pengeluaran

| Langkah | Isi |
|---|---|
| Header | Subtype, gudang asal sesuai scope, tanggal, referensi/keperluan, penjelasan. |
| Dokumen | Checklist Nota Dinas, Memo Internal, atau SPK sesuai rule subtype; metadata/versioning mengikuti kontrak upload. |
| Material | Material aktif, lokasi, requested qty, on-hand, reserved, quarantined, available, serta lot/label sebagai preferensi opsional sebelum check fisik. |
| Review | Ringkasan qty, dokumen, reservation yang akan dibuat, expiry, serta peringatan stok yang berubah. |

Selector mengikuti rantai gudang → lokasi → material → lot → label. `available_quantity` merupakan turunan read-only dari `stock_balances`; validasi browser hanya membantu dan server wajib memeriksa ulang menggunakan row lock saat submit.

Saat submit berhasil, tampilkan ringkasan `stock_reservations.reserved_quantity`, status, dan `expires_at`. Jika satu baris gagal direservasi, jangan menyisakan reservation parsial.

> **Dependensi schema:** jenis dokumen alternatif/wajib per subtype belum dimodelkan. `stock_reservations.transaction_item_id` juga belum mempunyai foreign key sehingga detail harus menangani item yang tidak ditemukan.

### 4.4 — Penyiapan Gudang, Scan QR, dan Posting Pengeluaran

```text
+--------------------------------------------------------------------------------+
| 📷 PENYIAPAN PENGELUARAN — 2202-MDN-202608-000015                             |
| Reservation aktif sampai: 25-08-2026 16:00                                   |
+--------------------------------------------------------------------------------+
| KIMAP | Diminta | Reserved | Lot/Kartu hasil scan | Qty keluar | Sisa lot    |
| ...   | 5 EA    | 5 EA     | LOT-... / KARTU-...  | [5 EA]     | 45 EA       |
+--------------------------------------------------------------------------------+
| [SCAN QR] [Simpan Penyiapan]                              [BUBUHKAN CHECKED]  |
+--------------------------------------------------------------------------------+
```

- Scan memvalidasi label aktif, material, lot, lokasi, gudang, dan saldo label.
- Staf mengisi `processed_quantity`; nilai tidak boleh melebihi requested qty, reservation, atau saldo lot terbaru.
- Hasil scan menampilkan identitas/qty/status, tanpa harga.
- CHECKED mengunci pilihan fisik untuk review Kepala Gudang.
- Approval Kepala Gudang menulis movement outbound dan mengonsumsi reservation sesuai kuantitas fisik.
- Fungsi Persediaan kemudian melihat harga dari lot/valuation service, memverifikasi `unit_price`/`total_amount`, dan menyelesaikan transaksi.

Timeline detail: Draft → Submitted/Reserved → CHECKED Staf → APPROVED Kepala Gudang → Posted Kuantitas → Verifikasi Nilai Persediaan → Completed. Detail memperlihatkan reservation, movement, lot/label, dokumen, watermark, waktu posting, dan tautan reversal.

### 4.5 — Daftar dan Wizard Retur

| Aspek | Detail |
|---|---|
| Scope | Gudang penerima sesuai assignment pengguna; detail transaksi asal tetap melalui policy. |
| Filter daftar | Gudang, periode, subtype retur aktif yang tervalidasi (target mapping 1102/1105/1106), status, tahap, nomor retur, nomor transaksi asal, material/KIMAP. |
| Kolom | Nomor retur, transaksi asal, subtype, gudang, pemohon, jumlah item, qty dikembalikan/diterima, status, tahap, dan aksi. |

Wizard menggunakan langkah **Pilih Transaksi Asal → Header & Dokumen → Qty Retur → Review**.

1. Pencarian transaksi asal menerima nomor transaksi, referensi, atau material dan hanya menampilkan pengeluaran yang boleh diakses.
2. Setelah dipilih, klasifikasi, kategori, KIMAP, nama, UOM, harga, qty pernah keluar, qty sudah pernah diretur, dan qty maksimum retur menjadi read-only.
3. User mengisi qty dikembalikan, alasan/keterangan, lokasi penerimaan usulan, dan dokumen Nota Dinas/Memo/Berita Acara Pencabutan.
4. Retur tanpa referensi historis diberi badge **Harga menunggu verifikasi Persediaan** dan tidak dapat posting final sebelum harga disahkan.

> **Dependensi schema:** `transaction_references` belum memiliki migration. Sampai schema ditambah, hubungan retur ke transaksi asal tidak boleh hanya disimpan sebagai teks bebas tanpa keputusan teknis eksplisit.

### 4.6 — Penerimaan Fisik dan Detail Retur

Staf Gudang memindai label bila ada lalu mengisi qty diterima, kartu, lokasi, lot/periode, kondisi, dan catatan. Sistem menolak qty diterima yang melebihi qty dikembalikan atau batas dari transaksi asal.

| Kolom pemeriksaan | Perilaku |
|---|---|
| Material/KIMAP/UOM | Read-only dari referensi/draft. |
| Qty dikembalikan | Read-only. |
| Qty diterima | Input positif, tidak boleh melebihi qty retur. |
| Kartu/lot/lokasi | Wajib sebelum CHECKED. |
| Kondisi/catatan | Catatan wajib bila ada selisih atau kerusakan. |

Timeline: Draft → Submitted → CHECKED Staf Gudang → APPROVED Kepala Gudang → Verifikasi Nilai Persediaan → Posted → Completed. Detail menampilkan transaksi asal, batas qty, dokumen, hasil fisik, approval, lot/label yang terbentuk, movement inbound, dan nilai final.

---

## MENU 5: 🔄 IN-TRANSIT (PEMINDAHAN ANTAR GUDANG)
Transaksi ini melibatkan **dua gudang**: gudang pengirim dan penerima.
```text
+═════════════════════════════════════════════════════════════════════════════+
║  🔄 FORMULIR PEMINDAHAN MATERIAL (IN-TRANSIT)                [DRAFT]        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Subtipe       : [2204 - Pemindahan Antar Gudang (readonly)]                ║
║  Gudang Asal   : [MDN - Gudang Utama Medan (readonly berdasarkan scope)]   ║
║  Gudang Tujuan : [BGR - Gudang Utama Bogor ▼] ← Dropdown Gudang Aktif     ║
║  Tgl Kirim     : [25 Agustus 2026 📅]                                      ║
║  No. Referensi : [MEMO-INTERNAL-001_______________]                         ║
║  Upload Surat  : [📎 Surat Jalan / Memo Pemindahan (PDF)] ← *WAJIB*        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  DETAIL ITEM YANG DIPINDAHKAN                                               ║
║  Material        │ Stok di MDN  │ Qty Kirim   │ UOM │ Hapus                ║
║  ────────────────┼──────────────┼─────────────┼─────┼──────────────────    ║
║  Valve Gate 4"   │  50 EA       │ [  10  ]    │ EA  │ [🗑️]                 ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  INFO: Material berstatus IN-TRANSIT sampai diterima di gudang tujuan.     ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 5.1 — Daftar Transfer dan Monitor In-Transit

**Workflow baseline:** User Gudang Asal → Staf Gudang Asal → Kepala Gudang Asal → In-Transit → Penerima Material → Persediaan Asal → Staf Gudang Tujuan → Kepala Gudang Tujuan → stok tujuan.

| Aspek | Detail |
|---|---|
| Scope | Actor asal melihat/memproses berdasarkan gudang asal; actor tujuan berdasarkan gudang tujuan dan tahap aktif. Pengguna lintas gudang tetap melalui policy. |
| Filter baseline | Gudang asal, tujuan, periode, status, current stage, material, nomor, hanya in-transit, dan rentang usia perjalanan. |
| Kolom baseline | Nomor, asal, tujuan, tanggal kirim, item/qty/nilai, status, tahap, waktu berangkat, usia perjalanan, penerima awal, dan aksi. |
| Indikator baseline | Reserved, siap berangkat, in-transit, diterima awal, menunggu check tujuan, atau completed. |

Klik baris membuka detail lintas tahap; dashboard/notification deep-link menggunakan filter gudang yang sesuai actor.

Kontrol **Melewati SLA**, badge overdue, deadline, dan alert transfer tidak ditampilkan pada baseline. Kontrol tersebut mengikuti dependensi konfigurasi SLA pada Dashboard; usia perjalanan tetap dihitung deterministik dari `posted_at`/waktu keberangkatan hingga `completed_at` atau waktu sekarang.

### 5.2 — Validasi Wizard Pemindahan

- Desain bisnis menggunakan kode subtype **2204**. Runtime wajib memuat `transaction_subtypes` yang aktif dan memverifikasi `code=2204` serta `main_type='transfer'`; bila seed/configuration itu tidak tersedia atau tidak cocok, tombol buat transfer dinonaktifkan dengan error konfigurasi. Jangan hard-code primary key subtype. Gudang asal berasal dari scope aktif dan tujuan harus aktif serta berbeda.
- Dokumen Surat Jalan atau Memo Pemindahan wajib sesuai rule subtype.
- Item menampilkan lokasi asal, lot/label, on-hand, reserved, quarantined, available, dan qty kirim.
- Destination location baru dipilih saat pemeriksaan tujuan atau lebih awal jika proses bisnis sudah mengetahuinya; selector selalu dibatasi gudang tujuan.
- Submit membuat reservation asal. Material quarantined atau qty melebihi available ditolak.
- Perubahan draft mengirim `lock_version`; validasi stock dilakukan kembali di server.

### 5.3 — Dispatch Gudang Asal

Staf Gudang Asal memindai QR, memilih lot/kartu, mengonfirmasi qty fisik kirim, dan mengisi catatan. Kepala Gudang Asal melihat hasil scan, reservation, dokumen, nilai, lalu APPROVE keberangkatan.

Pada approval keberangkatan, layanan posting:

1. Mengonsumsi reservation.
2. Mencatat movement `transit_out`/movement asal sesuai desain ledger.
3. Mengurangi on-hand asal dan menambah in-transit.
4. Mengisi `posted_at`/current stage in-transit.

Stok tujuan **belum bertambah** pada tahap ini. UI menampilkan banner in-transit, timestamp keberangkatan, aging, serta pemegang tahap berikutnya.

### 5.4 — Konfirmasi Penerima dan Check Gudang Tujuan

Penerima Material terlebih dahulu membuka ringkasan read-only berisi asal/tujuan, surat jalan, item, qty terkirim, lot/kartu asal, dan waktu berangkat. Aksi **Konfirmasi Diterima Awal** tidak mengubah saldo tujuan.

Setelah Persediaan asal memverifikasi harga/nilai, Staf Gudang Tujuan membuka layar:

```text
+--------------------------------------------------------------------------------+
| 📷 PENERIMAAN TRANSFER — 2204-MDN-202608-000003                               |
| MDN → BGR | IN-TRANSIT | Tahap: Pemeriksaan Staf Gudang Tujuan               |
+--------------------------------------------------------------------------------+
| Material | Qty kirim | Qty diterima | Lot/Kartu | Lokasi tujuan | Catatan     |
| ...      | 10 EA     | [10 EA]      | [...]     | [RAK-B2 ▼]   | [...]       |
+--------------------------------------------------------------------------------+
| [SCAN QR] [Simpan]                                      [BUBUHKAN CHECKED]    |
+--------------------------------------------------------------------------------+
```

Kepala Gudang Tujuan memberi APPROVED setelah check. Sistem kemudian mengurangi in-transit, menambah on-hand tujuan, menulis movement tujuan, dan mengubah transaksi menjadi completed. Selisih qty/kerusakan harus dicatat dan tidak boleh diam-diam mengubah qty terkirim.

**Kontrak lot dan label tujuan:** setiap saldo tujuan wajib menunjuk `stock_lot_id`, sedangkan lot dimiliki satu gudang. Karena itu layanan penerimaan tidak boleh memakai ulang lot asal sebagai lot tujuan. Untuk setiap material–lot sumber yang diterima, layanan membuat lot tujuan di `destination_warehouse_id`, mempertahankan `acquisition_period` dan `unit_price`, mengisi original/remaining quantity dari qty yang benar-benar diterima, serta menautkan `receipt_item_id` secara logis ke item transfer penerimaan. Movement `transit_in` hanya menunjuk lot tujuan, sedangkan item transaksi hanya memiliki `source_lot_id`; pasangan lot asal–tujuan adalah target lineage dan belum dapat diklaim tersimpan sebagai satu relasi pada schema saat ini.

Label asal tidak dipindahkan antar gudang. Setelah posting tujuan, label asal yang terkonsumsi ditutup sesuai lifecycle; layanan membuat token/kartu label tujuan pada lokasi tujuan. Target aturan serialized adalah satu label tujuan per unit/identitas yang diterima, sedangkan material non-serialized dapat memakai label agregat dengan `label_quantity` dan `remaining_quantity` sebesar qty diterima. Karena constraint quantity-per-label serialized belum ada, layanan wajib menolak pembuatan label serialized bila `label_quantity` atau `remaining_quantity` bukan 1. QR tujuan baru dapat dicetak setelah posting sukses.

> **Dependensi schema lineage transfer:** `stock_lots` belum memiliki `source_lot_id` atau FK khusus ke item transfer dan `receipt_item_id` masih referensi logis. Sebelum lineage tersebut dipakai sebagai bukti audit lintas gudang, tambahkan relasi parent/source lot atau tabel pemetaan lot transfer; UI sementara menampilkan tautan hanya bila layanan berhasil memvalidasi kedua referensi.

### 5.5 — Detail Timeline Transfer

Detail menampilkan delapan tahap, actor/role/snapshot, waktu, notes, watermark, dokumen, reservation, lot/label asal, lokasi tujuan, movement asal/tujuan, qty in-transit, aging, dan nilai. Aksi hanya muncul pada actor current stage; posted/completed read-only kecuali print atau reversal berizin.

---

## MENU 6: 🛑 QUARANTINE (PENYISIHAN / WRITE-OFF)
Membutuhkan upload **Surat Rekapitulasi Usulan dari Pengelola Material (GH OMM)** sebagai dokumen wajib.

```text
+═════════════════════════════════════════════════════════════════════════════+
║  🛑 FORMULIR PENYISIHAN MATERIAL (QUARANTINE / WRITE-OFF)      [DRAFT]     ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Subtipe    : [3300 - Penyisihan / Write-Off (Quarantine) ▼]               ║
║  Gudang     : [MDN - Gudang Utama Medan (readonly)]                         ║
║  Tgl Usulan : [25 Agustus 2026 📅]                                         ║
║  Upload Dok : [📎 Surat Rekapitulasi GH OMM (PDF)] ← *WAJIB*              ║
║  Keterangan : [Alasan penyisihan berdasarkan hasil inventarisasi...]        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  DAFTAR MATERIAL YANG DIUSULKAN UNTUK DISISIHKAN                            ║
║  Material         │ Stok Tersedia │ Qty Sisihkan │ Alasan      │ Hapus      ║
║  ─────────────────┼───────────────┼──────────────┼─────────────┼─────────── ║
║  Pipa Corrosive   │  5 MTR        │ [  5  ]      │ [Korosif ▼] │ [🗑️]       ║
║  Gasket Expired   │  12 EA        │ [  12 ]      │ [Expired ▼] │ [🗑️]       ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [← Batal]                     [💾 Simpan Draft]  [🚀 Ajukan Approval →]  ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 6.1 — Daftar Usulan, Quarantined, dan Written-Off

**Workflow baseline:** Holder/User mengusulkan → qty direservasi/dipindah ke quarantine → Staf Gudang check kartu/fisik → Kepala Gudang approve kondisi fisik → Fungsi Persediaan approve harga/nilai → posted sebagai write-off.

Sediakan tiga tab:

| Tab | Isi |
|---|---|
| Usulan | Draft, submitted, revision, rejected, dan under-review. |
| Quarantined | Qty sudah dipisahkan dari available tetapi belum written-off final. |
| Written-Off | Posting final lengkap dengan nilai, movement, dan referensi reversal bila ada. |

Filter: gudang, periode, status/tahap, material/KIMAP, lot/kartu, status material FM/SM/PDS/DS, alasan, dan pembuat. Kolom: nomor, gudang, material, lot/kartu, qty, nilai, alasan/catatan, status material, status stok, tahap, tanggal quarantine/posted, dan aksi.

Baris balance dengan `quarantined_quantity > 0` dan lot yang seluruh sisanya quarantined diberi badge serta dikeluarkan dari picker reguler. Karena label tidak memiliki status quarantined, badge/blok label diturunkan dari lokasi bertipe quarantine dan allocation yang tervalidasi sesuai 6.3. Material yang sama tetap dapat dipilih dari lot/lokasi lain bila baris tersebut tidak quarantined dan `available > 0`; larangan tidak boleh diterapkan pada agregat material secara keseluruhan.

### 6.2 — Kontrak Wizard Penyisihan

- Header: desain memakai kode 3300; runtime wajib memvalidasi row `transaction_subtypes` aktif dengan `main_type='write_off'`. Jika tidak cocok, CTA create dinonaktifkan. Gudang mengikuti scope aktif; tanggal, nomor referensi, dan penjelasan tetap wajib sesuai aturan.
- Dokumen: Nota Dinas/Memo/Surat Rekapitulasi Usulan Penyisihan GH OMM sesuai rule subtype.
- Item: material, lokasi, lot/label/kartu, on-hand, reserved, quarantined, available, qty sisihkan, harga/nilai read-only menurut kontrak nilai global, dan catatan per item.
- Qty harus positif dan tidak melebihi available/lot. Harga berasal dari lot/valuation, bukan input bebas Holder/Staf.
- Submit dan perpindahan ke quarantine dilakukan melalui layanan server; kegagalan satu baris tidak boleh menghasilkan perubahan saldo parsial.

> **Catatan schema:** `inventory_transaction_items` belum mempunyai kolom reason terstruktur. Jika `notes` dipakai, label UI harus **Catatan Item**, sedangkan alasan header menggunakan `inventory_transactions.explanation`.

### 6.3 — Invariant Saldo, Lot, Label, dan Ledger Quarantine

Quarantine adalah **reklasifikasi stok di dalam gudang**, bukan pengeluaran persediaan. Baseline mewajibkan lokasi tujuan bertipe `quarantine` dan seluruh perubahan berikut berjalan atomik per item/lot.

| Tahap | Perubahan balance/ledger yang wajib |
|---|---|
| Reservation saat submit | Pada baris sumber: `reserved += q`; `on_hand` dan `quarantined` tidak berubah. `available` turun tepat `q`. Reservation tidak membuat `inventory_movements`. |
| Pindah ke quarantine | Konsumsi/release reservation (`reserved -= q`); balance lokasi sumber `on_hand -= q`; balance lokasi quarantine untuk lot yang sama `on_hand += q` dan `quarantined += q`. Agregat gudang tetap: on-hand tidak berubah, quarantined naik `q`, available tetap turun hanya `q`. |
| Ledger reclassification | Buat pasangan `movement_type=quarantine` dengan transaction/item sama: sumber `quantity_delta=-q`, `amount_delta=-v`; lokasi quarantine `quantity_delta=+q`, `amount_delta=+v`. Net gudang/material adalah 0; grain lokasi menunjukkan perpindahan. |
| Final write-off | Pada balance quarantine: `on_hand -= q` dan `quarantined -= q` bersamaan; `available` tetap 0. Kurangi `stock_lots.remaining_quantity` sekali; untuk whole-label set `stock_labels.remaining_quantity=0` dan status `consumed`; tulis satu movement `write_off` dengan `quantity_delta=-q`, `amount_delta=-v`. Jangan mengurangi source row atau ledger untuk kedua kalinya. |
| Release/reject sebelum write-off | Balikkan pasangan reclassification dengan movement baru yang menunjuk `reversal_of_id`; balance quarantine turun, balance lokasi asal naik, label kembali ke lokasi asal, dan available pulih `q`. Ledger lama tidak diedit. |

Contoh agregat: sebelum proses `on_hand=10, reserved=0, quarantined=0, available=10`. Setelah reservation menjadi `10,2,0,8`; setelah pemindahan quarantine menjadi agregat `10,0,2,8`; setelah write-off menjadi `8,0,0,8`. Dengan urutan ini tidak ada pengurangan available atau ledger dua kali.

**Grain lot/label:** transaction item wajib menunjuk lot sumber; sumber dan tujuan quarantine memakai lot gudang yang sama, lokasi berbeda. `stock_labels` tidak memiliki status `quarantined`, sehingga eligibility label diturunkan dari `warehouse_location_id` bertipe quarantine dan balance/allocation terkait—bukan dari status label. Picker dan scanner reguler selalu mengecualikan lokasi quarantine serta qty yang teralokasi quarantine.

- Whole-label quarantine memindahkan lokasi label ke lokasi quarantine dalam transaksi yang sama dengan balance/ledger pair.
- Partial label quarantine atau satu item yang mencakup beberapa label diblokir pada baseline. Satu label tidak dapat sekaligus mewakili bagian available dan quarantined, sementara schema belum memiliki parent/split-label lineage atau tabel allocation per label.
- Untuk mengaktifkannya, tambahkan `stock_quarantine_allocations` (transaction item, lot, optional label, source/destination location, quantity, status) dan lineage label split. Sampai itu tersedia, UI meminta whole label atau memecah item menjadi unit label utuh yang dapat dibuktikan.

### 6.4 — Check Staf Gudang dan Approval

```text
+--------------------------------------------------------------------------------+
| 🛑 CHECK FISIK PENYISIHAN — 3300-MDN-202608-000004                            |
+--------------------------------------------------------------------------------+
| Material | Lot | Kartu | Qty quarantine | Lokasi quarantine | Kondisi | Catatan|
| ...      | ... | ...   | 5 EA           | [QRN-01 ▼]        | [...]   | [...] |
+--------------------------------------------------------------------------------+
| [SCAN QR] [Simpan Pemeriksaan]                         [BUBUHKAN CHECKED]      |
+--------------------------------------------------------------------------------+
```

Staf memverifikasi label/kartu, lot, kondisi, qty, dan lokasi quarantine. Kepala Gudang lalu memeriksa hasil fisik dan memberikan APPROVED. Fungsi Persediaan melihat unit cost/nilai serta dokumen sebelum approval final.

### 6.5 — Detail Quarantine hingga Write-Off

Timeline: Draft → Submitted → Reserved/Quarantined → CHECKED Staf → APPROVED Kepala Gudang → APPROVED Persediaan → Posted Write-Off → Completed.

Detail wajib membedakan:

- qty/nilai sebelum quarantine;
- qty yang quarantined dan tidak available;
- qty/nilai yang akhirnya written-off;
- saldo/lot/label terdampak;
- movement `quarantine` dan `write_off` yang terbentuk;
- dokumen, approval, watermark, actor, dan timestamp.

Setelah posting, tidak tersedia edit/delete. Koreksi menggunakan reversal atau adjustment baru yang dapat diaudit.

---

## MENU 7: 📋 STOCK OPNAME & INVENTARISASI
**Akses halaman (union):** pengguna dapat membuka modul bila memiliki salah satu dari `stock_opname.create_session`, `stock_opname.input_count`, atau `stock_opname.approve`. Memiliki salah satu izin tidak memberikan aksi dari izin lainnya; warehouse policy tetap diterapkan.

### 7.1 — Manajemen Sesi Stock Opname
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📋 STOCK OPNAME — MANAJEMEN SESI                                           ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  @can('stock_opname.create_session') [➕ Buat Sesi Baru] @endcan            ║
║  Sesi ID        │ Gudang │ Scope Lokasi │ Tgl Mulai  │ Status    │ Aksi     ║
║  ───────────────┼────────┼─────────────┼────────────┼───────────┼────────  ║
║  SO-MDN-202608-1│ MDN    │ RAK-A1      │ 25-08-2026 │ 🟡 Aktif  │ [Input 🔢]║
║  SO-BGR-202607-2│ BGR    │ Semua       │ 15-07-2026 │ 🟢 Selesai│ [Lihat 👁️]║
╚═════════════════════════════════════════════════════════════════════════════╝
```
> Klik `[➕ Buat Sesi Baru]` → Modal AJAX memilih Gudang, Scope Lokasi/Rak, dan Tanggal Mulai.

### 7.2 — Form Input Hitung Fisik (Editable Grid)
Dirancang untuk digunakan di lapangan menggunakan **tablet/iPad**.
```text
+═════════════════════════════════════════════════════════════════════════════+
║  🔢 INPUT HITUNG FISIK — Sesi: SO-MDN-202608-01 │ Gudang: MDN │ RAK-A1    ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Metode Hitung:  [○ Metode 1: Manual Input Grid]  [● Metode 2: Scan QR]    ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  METODE 1 — MANUAL INPUT:                                                  ║
║  KIMAP    │ Nama Barang         │Qty Sistem│ QTY FISIK  │ SELISIH │ CATATAN ║
║  ─────────┼─────────────────────┼──────────┼────────────┼─────────┼──────── ║
║  10001928 │ VALVE GATE 4" SS    │  50 EA   │ [ 50 ] EA  │  0 EA   │ [     ] ║
║  10002133 │ PIPA SEAMLESS 8"    │  10 MTR  │ [  8 ] MTR │ -2 MTR⚠️│ [Rusak*]║
║  90001123 │ LAPTOP ASUS ROG     │   2 SET  │ [  2 ] SET │  0 SET  │ [     ] ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  * ⚠️ Jika ada SELISIH, kolom Catatan WAJIB DIISI sebelum bisa disimpan.   ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  METODE 2 — SCAN QR AGGREGASI:                                              ║
║  [📷 MULAI SCAN QR LABEL BARANG]                                            ║
║  Token QR yang terbaca: 12 scan │ 3 Material Unik                          ║
║  [Lihat Rekap Scan ...]         │ Serialized=1; nonserial=Σ snapshot qty   ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  @can('stock_opname.input_count')                                         ║
║  [💾 Simpan Hasil Hitung Fisik]             [✅ Selesaikan Input] @endcan  ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 7.3 — Spesifikasi Daftar dan Pembuatan Sesi

| Aspek | Detail |
|---|---|
| Sumber | `stock_count_sessions`, gudang, creator/approver, agregasi `stock_count_items`. |
| Filter | Nomor, gudang, tipe `stock_opname`/`inventarisasi`, status, periode, creator, scope. |
| Kolom | Nomor, tipe, gudang, period start/end, snapshot time, scope, status, creator, approver, progres, jumlah variance, dan aksi. |
| Status | `draft`, `in_progress`, `completed`, `approved`, `cancelled`; label “Aktif” memetakan `in_progress`, sedangkan “Selesai” harus membedakan completed dan approved. |
| Aksi | Buat, edit/cancel draft, mulai sesi/snapshot, input hitung, review variance, selesaikan input, lihat status dependensi approval, dan lihat adjustment. |

Modal Buat Sesi berisi tipe, gudang scope aktif, period start/end, scope, creator read-only, nomor server-generated, dan snapshot read-only setelah sesi dimulai. Period end tidak boleh sebelum start dan session number harus unik.

| Aksi | Permission dan syarat |
|---|---|
| Buka daftar/detail | Salah satu dari tiga permission modul dan sesi berada dalam scope gudang. |
| Buat/edit/mulai/cancel draft | `stock_opname.create_session`; hanya status yang sesuai dan actor berwenang. |
| Input/scan/simpan/selesaikan input | `stock_opname.input_count`; hanya `in_progress`. Actor penghitung tidak memperoleh hak approval. |
| Review keputusan sesi | `stock_opname.approve`; tombol keputusan tetap nonaktif sampai model approval sesi pada 7.5 tersedia. |

> **Dependensi schema:** `stock_count_sessions.scope` hanya string. Selector lokasi/rak tidak memiliki FK sehingga UI tidak boleh mengklaim scope lokasi sudah tervalidasi relasional atau mendukung multi-lokasi secara aman.

### 7.4 — Kontrak Grid Manual dan QR

Setiap baris merepresentasikan material–lot dan label opsional, bukan hanya material agregat.

| Field | Perilaku |
|---|---|
| KIMAP/nama/UOM | Read-only dari material. |
| Lot/periode/status | Read-only dari `stock_lots`. |
| Label/kartu | `stock_label_id` opsional; tampilkan status dan lokasi. |
| Count method | `manual` atau `qr_scan`. |
| System quantity | Snapshot read-only. |
| Physical quantity | Input non-negatif saat in_progress. |
| Variance | Dihitung server: physical − system; read-only. |
| Explanation | Wajib bila variance tidak nol. |
| Scanned tokens | Rekap read-only untuk mode QR. |
| Counted by/at | Diisi server saat simpan. |

Mode QR memvalidasi token ditemukan, label aktif, gudang melalui rantai label–lot, lot/material cocok, aturan scope sesi, dan token belum dipindai duplikat. Karena `stock_count_sessions.scope` hanya string, cakupan lokasi dievaluasi sebagai aturan aplikasi terhadap nilai scope yang dinormalisasi; UI tidak boleh menyebutnya validasi FK/multi-lokasi. Untuk material `is_serialized=true`, aturan aplikasi menerima label sebagai satu unit hanya bila `label_quantity=1` dan `remaining_quantity=1`; pelanggaran memblokir scan sebagai **Data label serialized tidak konsisten** karena schema belum memiliki check constraint tersebut. Untuk material non-serialized, kuantitas fisik adalah jumlah `stock_labels.remaining_quantity` yang diambil sebagai snapshot server ketika scan diterima—bukan jumlah token. Rekap menampilkan token/kartu, serialized flag, quantity snapshot, accepted/rejected state, dan alasan penolakan; token duplikat tidak ikut dijumlahkan.

Snapshot `{token, quantity_snapshot, accepted_at}` disimpan bersama bukti scan pada `scanned_label_tokens` JSON. Jika implementasi lama membatasi JSON tersebut menjadi daftar token saja, mode QR non-serialized harus dinonaktifkan sampai struktur evidence dapat menyimpan quantity snapshot; memakai nilai label terkini saat membuka ulang sesi akan mengubah hasil hitung secara diam-diam. Error scan harus spesifik dan aman. Scan offline tidak termasuk baseline; banner koneksi mencegah validasi/submit ketika server tidak tersedia.

### 7.5 — Review Selisih dan Penyelesaian Sesi

Filter review: variance positif/negatif/semua, material, lot, metode, petugas, dan penjelasan belum lengkap. Kolom menampilkan system/physical/variance, explanation, token scan, counted by/at.

- **Selesaikan Input** aktif hanya jika semua item wajib dihitung dan semua variance memiliki explanation.
- Setelah completed, grid menjadi read-only.
- Panel berikutnya menampilkan **Approval sesi belum tersedia** dan menjelaskan actor yang dibutuhkan; tidak menampilkan tombol CHECKED/APPROVE aktif pada baseline schema.
- Selisih tidak pernah langsung mengubah saldo. Baseline hanya menyediakan **Lihat Status Adjustment**; CTA membuat adjustment tetap disabled sampai dependensi 7.7 tersedia.

> **Dependensi schema kritis — approval sesi:** workflow Stock Opname membutuhkan bukti **CHECKED Staf Gudang → APPROVED Kepala Gudang**, sedangkan Inventarisasi membutuhkan **APPROVED/VERIFIED Tim Inventarisasi → APPROVED Kepala Gudang**. `stock_count_sessions` hanya mempunyai satu `approved_by` tanpa stage, decision, notes, decided_at, actor/role/position snapshot, warehouse-scope snapshot, sequence, atau watermark. Tambahkan model `stock_count_session_approvals` (atau model evidence setara) sebelum kedua rantai keputusan dan dokumen berstempel diaktifkan. Sampai itu tersedia, `approved_by` hanya boleh dilabeli **Penyetuju tunggal (data legacy)**; UI tidak boleh mengklaim siapa melakukan CHECKED, urutan approval, waktu keputusan, atau watermark resmi.

### 7.6 — Sesi Inventarisasi dan Penugasan Tim

Inventarisasi memakai form sesi, dua metode hitung, dan review variance yang sama. Panel target dapat menjelaskan kebutuhan surat tugas, anggota tim, foto/dokumen opsional, serta keputusan Tim Inventarisasi dan Kepala Gudang, tetapi seluruh assignment dan tombol keputusan tersebut ditampilkan disabled dengan label dependensi sampai model penyimpanan tersedia.

Data yang tersedia saat ini: creator, approver, petugas yang sudah menghitung, dan pengguna dengan scope gudang aktif. UI boleh menampilkan daftar pelaku aktual dari `stock_count_items.counted_by`.

> **Dependensi schema:** `stock_count_assignments` disebut pada rancangan tetapi tidak memiliki migration. Belum ada penyimpanan anggota, ketua/anggota, status penerimaan tugas, surat tugas, atau assignment per sesi. Tombol **Assign Tim** tidak boleh dianggap persisten sampai schema ditambah.

### 7.7 — Daftar dan Detail Inventory Adjustment

**Sumber:** `inventory_adjustments` + `stock_count_sessions` + users approver.

| Area | Detail |
|---|---|
| Filter | Gudang, nomor/tipe sesi, status draft/pending_approval/approved/posted/rejected, periode, approver, posted date. |
| Kolom | ID, nomor sesi, gudang, reason, status, Kepala Gudang, Persediaan, posted_at, timestamps, dan aksi. |
| Detail | Header/reason, ringkasan variance sesi, kedua approval, status, posting evidence bila tersedia. |
| Aksi baseline | Lihat saja. Create/edit/submit/approve/reject/post dinonaktifkan sampai item koreksi, linkage ledger, permission, concurrency, dan decision evidence tersedia. |

**Kontrak target setelah dependensi schema tersedia:**

Pada schema saat ini, daftar/detail adjustment bersifat **read-only** untuk seluruh status. Tabel berikut menetapkan permission dan state machine target, bukan mengaktifkan CTA sebelum migration/deployment pendukung selesai.

| Tahap | Aturan UI dan server |
|---|---|
| Lihat | Salah satu permission modul stock opname atau `transactions.verify_inventory`/`transactions.post`, selalu dalam scope gudang. |
| Buat/edit/submit draft | Memerlukan permission baru `inventory_adjustments.manage`; permission ini belum ada pada seeder. |
| Menunggu Kepala Gudang | `status=pending_approval`, kedua approver null. Aksi target memerlukan `stock_opname.approve` dan actor berperan Kepala Gudang pada scope sesi. |
| Menunggu Persediaan | `status=pending_approval`, `approved_by_head` terisi, `approved_by_inventory` null. Aksi target memerlukan `transactions.verify_inventory`. |
| Approved | Setelah kedua keputusan evidence tersimpan, layanan mengisi kedua FK approver dan mengubah status menjadi `approved`; actor pembuat tidak boleh menyetujui dan satu user tidak boleh mengisi kedua tahap. |
| Posting | Aksi target memerlukan `transactions.post`, status `approved`, item adjustment tervalidasi, linkage transaksi/movement tersedia, dan posting atomik ke ledger. Setelah sukses status `posted` dan `posted_at` terisi. |

Setelah fitur target diaktifkan, request approval/post membawa idempotency key, expected `lock_version`, expected status, dan expected approver IDs; layanan melakukan conditional update dalam transaksi database dan mengembalikan 409 bila satu nilai berubah. Schema saat ini belum memiliki `lock_version` atau idempotency record, sehingga kontrak ini merupakan blocking dependency dan bukan mekanisme baseline.

Status `rejected` memang tersedia, tetapi tabel tidak memiliki `rejected_by`, `rejected_at`, atau `rejection_note`; kolom `reason` adalah alasan koreksi dan tidak boleh dipakai ulang sebagai alasan penolakan. Karena itu tombol **Reject** dinonaktifkan. Record rejected legacy hanya menampilkan status dan `updated_at` berlabel **waktu record diperbarui**, tanpa mengarang actor, waktu keputusan, atau catatan.

> **Dependensi schema kritis:** `inventory_adjustments` tidak mempunyai item detail dan tidak berelasi ke `inventory_transactions`/`inventory_movements`. Ia juga belum memiliki creator, approval/rejection decision records dan timestamp, rejection note, `lock_version`, maupun idempotency record. UI dapat menampilkan variance dari sesi sebagai konteks, tetapi tidak boleh mengklaim baris koreksi, keputusan, atau bukti ledger memiliki integritas/audit lengkap sampai schema diperluas.

---

## TAMPILAN OPERASIONAL PERSEDIAAN & QR

Tampilan berikut dibuka dari Dashboard Staf, detail material, detail transaksi, laporan posisi stok, atau hasil scan. Ia bukan menu mutasi stok langsung dan tidak menyediakan edit saldo/ledger.

### O.1 — Posisi Stok Saat Ini

**Sumber:** `stock_balances` + gudang/lokasi + material/UOM + lot + status material efektif.

Filter: gudang, lokasi, material/KIMAP, klasifikasi, kategori, lot/periode, status lot, status material, serialized, hanya available/reserved/in-transit/quarantine.

Kolom: gudang/lokasi, KIMAP/nama/UOM, lot/periode/status, on-hand, reserved, in-transit, quarantined, available, average cost, nilai, status material, updated time, dan aksi ke kartu/lot/label/reservation/ledger.

`stock_balances` adalah proyeksi saat ini. Tidak ada inline edit. Jika pengguna meminta tanggal historis, arahkan ke ledger atau tampilkan kebutuhan snapshot/reconstruction; jangan melabeli proyeksi kini sebagai saldo tanggal lampau.

### O.2 — Kartu Material

Header menampilkan KIMAP, nama, klasifikasi, kategori, UOM, serialized, active, dan status efektif per gudang. Tab:

| Tab | Isi |
|---|---|
| Saldo Saat Ini | Agregasi gudang–lokasi–lot dari `stock_balances`. |
| Lot | Acquisition period, unit price, original/remaining qty, status. |
| Label QR | Card number, lokasi, label/remaining qty, status, print count/time. |
| Mutasi | Ledger `inventory_movements` immutable. |
| Riwayat Status | FM/SM/PDS/DS per gudang dan periode. |

Semua tab mengikuti warehouse scope. Baseline menampilkan harga kepada pemegang `transactions.view`/`reports.view` yang lolos warehouse policy; permission nilai granular masih menjadi dependensi global.

### O.3 — Daftar dan Detail Lot

Filter: gudang, material, periode akuisisi, status `active`/`exhausted`/`quarantined`/`written_off`, sisa qty, dan unit price sesuai permission.

Kolom: lot ID, gudang, material/UOM, acquisition period, receipt item reference, unit price, original/remaining qty, status, timestamps, jumlah label, dan aksi detail/label/ledger. Remaining qty, status, dan harga historis tidak diedit sebagai master.

> **Catatan schema:** `stock_lots.receipt_item_id` belum mempunyai FK. Tautan penerimaan hanya aktif bila layanan menemukan item sumber.

### O.4 — Daftar Label QR dan Kartu

Filter: gudang, lokasi, material, lot, card number, status `active`/`consumed`/`void`/`reprinted`, print date/count.

| Kolom | Perilaku |
|---|---|
| Card number | Dapat disalin; pencarian exact/partial. |
| QR token | Dimask pada list, tidak pernah diekspor mentah tanpa kebutuhan resmi. |
| Gudang/lokasi/material/lot | Read-only dari relasi. |
| Label/remaining qty | Read-only dan mengikuti posting. |
| Status/printed count/last printed | Menentukan apakah scan/print diperbolehkan. |
| Aksi baseline | Detail dan print awal sesuai status/permission. Reprint/void tampil disabled dengan penjelasan dependensi audit. |

Print menambah counter/time melalui layanan. Reprint dan void adalah kontrak target yang mewajibkan alasan, konfirmasi, actor/time, dan audit; tidak aktif pada schema sekarang.

> **Dependensi schema:** `stock_labels` tidak menyimpan printed_by, alasan reprint/void, atau void time. Informasi tersebut memerlukan audit table/event yang belum tersedia.

### O.5 — Hasil Scan QR

```text
+--------------------------------------------------------------------------------+
| 📷 HASIL SCAN QR                                                               |
| Status: VALID — Label aktif | Gudang: MDN | Lokasi: RAK-A1                    |
+--------------------------------------------------------------------------------+
| KIMAP / Material : 10001928 / Valve Gate 4" SS                               |
| Klasifikasi      : MPS       | Kategori: Tubular | UOM: EA                    |
| Lot / Perolehan  : LOT-... / 2026-08                                           |
| Nomor Kartu      : CARD-00018                                                  |
| Qty Label / Sisa : 50 EA / 45 EA                                               |
| Status Material  : FM        | Status Stok: Available                         |
+--------------------------------------------------------------------------------+
| [Lihat Kartu Material] [Lihat Ledger] [Gunakan pada Proses Aktif]             |
+--------------------------------------------------------------------------------+
```

- Lookup dimulai dari `stock_labels.qr_token`, lalu lot, lokasi, material, gudang, balance, dan movement.
- Harga tidak tampil pada scan umum.
- Invalid/nonaktif/void/consumed/out-of-scope menampilkan pesan aman dan Scan Lagi tanpa identitas material.
- CTA “Gunakan pada Proses Aktif” hanya ada bila scanner dibuka dari transaksi/sesi yang cocok dan server memvalidasi gudang, material, lot, lokasi, serta status.
- Mode offline tidak dapat memvalidasi transaksi.

### O.6 — Monitor Reservasi

Sumber `stock_reservations` + item/header transaksi + lot/material/gudang. Filter: gudang, nomor transaksi, material, lot, status active/fulfilled/released/expired, expiry, periode.

Kolom: reservation ID, nomor/line transaksi, gudang, material, lot, reserved, consumed, sisa, status, expires_at, timestamps, dan link transaksi. User tidak mengedit qty reservasi dari grid; release/fulfill/expire mengikuti state machine transaksi dan menyegarkan balance.

### O.7 — State dan Responsif Operasional

- QR scanner meminta izin kamera, menyediakan input scanner hardware/manual token sesuai policy, dan mengembalikan fokus ke Scan Lagi.
- Tablet memakai kartu besar dan sticky scan/action; mobile menampilkan satu detail per layar.
- Data harga/nilai mengikuti kontrak global: baseline terlihat bagi pemegang `transactions.view`/`reports.view` yang lolos policy record; hasil scan umum tetap tanpa harga.
- Record out-of-scope tidak boleh terlihat sesaat ketika warehouse context berubah.
- Error relasi yatim diberi label “Referensi tidak tersedia” dan tidak merusak seluruh detail.

---

## MENU 8: 📈 REPORTING & ANALYTICS
**Akses:** `@can('reports.view')` | Export: `@can('reports.export')`

### 8.1 — Laporan Mutasi Stok
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📈 LAPORAN MUTASI STOK                                                     ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Filter:                                                                    ║
║  Gudang     : [Semua ▼]         Periode: [Aug 2026 ▼]                      ║
║  Klasifikasi: [MPS ▼]           Material: [Cari KIMAP/Nama...]             ║
║  [Terapkan Filter]     @can('reports.export') [📥 Export Excel] [📄 PDF] @endcan ║
╠════════╦══════════════╦═══════╦══════════╦═══════════╦═══════════╦═════════╣
║ KIMAP  ║ Nama Barang  ║ Gudang║ Saldo    ║ Masuk     ║ Keluar    ║ Saldo   ║
║        ║              ║       ║ Awal     ║ (Inbound) ║ (Outbound)║ Akhir   ║
╠════════╬══════════════╬═══════╬══════════╬═══════════╬═══════════╬═════════╣
║1000192 ║ Valve Gate4" ║ MDN   ║  100 EA  ║  +50 EA   ║  -35 EA   ║  115 EA ║
║1000213 ║ Pipa SS 8"   ║ MDN   ║   20 MTR ║    0 MTR  ║   -5 MTR  ║   15 MTR║
╚════════╩══════════════╩═══════╩══════════╩═══════════╩═══════════╩═════════╝
```

**Kontrak implementasi ringkasan mutasi:**

| Aspek | Detail |
|---|---|
| Sumber utama | `inventory_movements` sebagai ledger append-only, join ke material dan `materials.base_uom_id`, gudang/lokasi, lot/label, user poster, dan referensi transaksi bila ditemukan. `inventory_movements` tidak memiliki `uom_id`; `stock_balances` tidak dipakai untuk menghitung saldo awal/akhir periode. |
| Grain default | Satu baris per gudang–material dengan UOM dasar material sebagai label kuantitas. Pengguna dapat memperinci ke lokasi dan lot; seluruh angka dihitung kembali pada grain terpilih. |
| Batas periode | Periode wajib. Query memakai `occurred_at >= awal hari` dan `< hari setelah tanggal akhir` dalam timezone gudang, lalu mengonversi batas ke timezone penyimpanan server. Generated-at dan timezone dicetak pada layar/export. |
| Saldo awal | `SUM(quantity_delta)` dan `SUM(amount_delta)` untuk seluruh movement sebelum batas awal pada grain yang sama. Jika data opening/migrasi ledger tidak lengkap, tampilkan banner **Saldo awal belum tervalidasi** dan blok PDF resmi. |
| Mutasi periode | Jumlah signed `quantity_delta`/`amount_delta` dalam periode untuk semua tipe. Headline Masuk/Keluar eksternal tidak memasukkan pasangan `quarantine`; keduanya ditampilkan pada kolom **Reklasifikasi Quarantine Masuk/Keluar** dan bernilai net 0 pada grain gudang–material. `write_off` menjadi pengurangan qty/nilai tepat sekali saat final. Inbound/outbound, transit-in/out, adjustment, dan reversal tetap memiliki breakdown sendiri. |
| Reversal | Movement dengan `reversal_of_id` tetap dihitung menurut delta/tanggalnya dan diberi kategori Reversal. Original tidak dihapus atau di-net-off secara tersembunyi; drill-down memperlihatkan pasangan original–reversal. |
| Saldo akhir | `saldo awal + net movement periode`, masing-masing untuk qty dan nilai. Rekonsiliasi opsional membandingkan saldo akhir periode hari ini dengan proyeksi `stock_balances`, tanpa mengganti angka ledger. |
| UOM ledger | Kontrak posting wajib menormalisasi `quantity_delta` ke `materials.base_uom_id`. Karena belum ada tabel konversi UOM, posting item dengan `inventory_transaction_items.uom_id` berbeda dari UOM dasar harus ditolak atau menunggu master konversi; laporan resmi diblokir bila ditemukan movement yang tidak dapat dibuktikan sudah ternormalisasi. |
| Nilai | Kolom nilai awal, masuk, keluar, adjustment/reversal net, dan akhir berasal dari `amount_delta`. Sesuai kontrak global, baseline menampilkan nilai kepada seluruh pemegang `reports.view`; bila kerahasiaan diperlukan, tambahkan `inventory_value.view`/`reports.view_value` dan policy server sebelum menyembunyikan kolom/export. Kuantitas mengikuti presisi UOM dasar. |
| Filter | Periode, gudang, lokasi, klasifikasi, kategori, KIMAP/nama, lot, movement type, subtype/nomor transaksi, poster, dan original/reversal. Warehouse scope diterapkan server-side. |
| Kolom minimum | Gudang, KIMAP/nama, UOM, saldo awal qty/nilai, masuk qty/nilai, keluar qty/nilai, transfer-in/out, quarantine, write-off, adjustment, reversal, net periode, saldo akhir qty/nilai, movement terakhir, dan aksi. |
| Drill-down | Klik angka/baris membuka Ledger Mutasi 8.5 dengan grain, periode, tipe, dan warehouse scope yang sama. Tautan transaksi/lot/label hanya aktif bila referensi tervalidasi. |
| Default sort | `warehouses.code ASC`, lalu `materials.kimap ASC`, lalu `stock_lots.acquisition_period ASC` bila mode lot aktif. Pagination dan export memakai urutan stabil yang sama. |

Empty state tetap menampilkan periode dan scope yang dipakai. Error hitung salah satu agregat menggagalkan seluruh ringkasan—UI tidak boleh mencampur angka lama dan baru—serta menyediakan correlation ID dan **Coba Lagi**.

### 8.2 — Laporan Saldo Stok Posisi Gudang
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📋 POSISI STOK GUDANG (SALDO SAAT INI)                                     ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Filter: [Gudang: MDN ▼] [Lokasi: RAK-A1 ▼] [Material/KIMAP........]       ║
║  [📥 Export Excel]  [📄 Export PDF]                                         ║
╠══════════╦════════════════╦═════════╦═══════════╦════════════╦═════════════╣
║  KIMAP   ║ Nama Barang    ║ Lokasi  ║ On Hand   ║ Reserved   ║ Available   ║
╠══════════╬════════════════╬═════════╬═══════════╬════════════╬═════════════╣
║ 10001928 ║ Valve Gate 4"  ║ RAK-A1  ║  115 EA   ║   10 EA    ║  105 EA    ║
║ 10002133 ║ Pipa SS 8"     ║ ZONE-B  ║   15 MTR  ║    0 MTR   ║   15 MTR   ║
╚══════════╩════════════════╩═════════╩═══════════╩════════════╩═════════════╝
```

Sumber layar ini adalah `stock_balances`, sehingga ia hanya menampilkan posisi saat ini. Tambahkan kolom lot, status material, in-transit, quarantined, average cost, nilai, dan updated time pada mode tabel lengkap.

> **Dependensi schema:** permintaan posisi **per tanggal historis** tidak dapat dipenuhi langsung dari `stock_balances`. Gunakan rekonstruksi ledger yang tervalidasi atau tabel snapshot sebelum kontrol tanggal historis diaktifkan.

### 8.3 — Laporan Rekonsiliasi Penyisihan
Menampilkan ringkasan material yang masuk kategori penyisihan (SM, PDS, DS) beserta nilai finansialnya.

Filter: gudang, lokasi, material, klasifikasi, lot, status material, status lot, periode, dan tipe movement. Kolom: gudang/lokasi, KIMAP/nama, lot, status efektif beserta alasan/periode/penetap, on-hand, quarantined, available, unit cost, nilai, transaksi/dokumen terkait, dan movement terakhir.

### 8.4 — Laporan Persediaan Material

**Sumber:** `stock_balances` saat ini + material/klasifikasi/kategori/UOM + gudang/lokasi + lot + status material efektif.

| Filter | Kolom minimum |
|---|---|
| Gudang, lokasi, klasifikasi, kategori, material, lot/periode, status material, status lot, serialized, kondisi qty | KIMAP/nama, klasifikasi/kategori, UOM, gudang/lokasi, lot/periode, on-hand, reserved, in-transit, quarantined, available, average unit cost, nilai, status material, updated_at. |

Header selalu menampilkan label **Posisi Saat Ini — bersumber dari proyeksi `stock_balances`**.

### 8.5 — Ledger Mutasi Immutable

Laporan ini merupakan detail movement, berbeda dari ringkasan masuk/keluar pada 8.1.

- Filter: occurred time, gudang, lokasi, material, lot, label/kartu, movement type (`inbound`, `outbound`, `transit_in`, `transit_out`, `quarantine`, `write_off`, `adjustment`), nomor/subtype transaksi, poster, reversal/original.
- Kolom: movement UUID, occurred_at, gudang/lokasi, nomor dan line transaksi, material/UOM, lot/label/kartu, movement type, quantity delta, amount delta, posted by, reversal_of_id, created_at.
- Aksi hanya lihat transaksi/lot/label/reversal; tidak ada edit/delete.
- Reference ID yang tidak ditemukan ditampilkan “Referensi tidak tersedia”, karena transaction/item/reversal ID pada ledger belum seluruhnya berupa FK database.

### 8.6 — Rekapitulasi Transaksi

Sumber `inventory_transactions`, subtype, gudang asal/tujuan, requester, agregasi item, dokumen, dan approval.

Filter: tipe/subtype, status, current stage, gudang asal/tujuan, requester, transaction/submitted/posted/completed date, nomor, dan reference. Kolom: nomor, tipe/subtype, asal/tujuan, tanggal, requester, reference, jumlah baris, total requested/processed, nilai, status, stage, timestamps, dan aksi detail.

### 8.7 — Laporan Stock Opname dan Inventarisasi

Mode **Ringkasan Sesi** menampilkan nomor, tipe, gudang, periode, snapshot, scope, status, creator/approver, total item, progres, dan jumlah variance. Mode **Rincian Hitung** menampilkan material, lot, label, UOM, system/physical/variance, explanation, method, scanned tokens, counted by/at.

Filter: nomor/tipe sesi, gudang, periode, status, metode, petugas, hanya variance, material, dan lot. Scope lokasi diberi label teks karena belum merupakan FK.

### 8.8 — Pusat Ekspor

Setiap laporan memiliki tombol XLSX untuk analitik dan PDF untuk dokumen resmi. Filter dan warehouse policy identik dengan DataTable sumber. Export tanpa hasil harus berhenti sebelum job dibuat.

Untuk ekspor besar, desain pusat ekspor menampilkan queued/processing/completed/failed, progress, pemilik, dibuat/selesai, file, expiry, retry, dan download audit.

> **Dependensi schema:** belum ada tabel export job/metadata file, progress, owner, checksum, TTL, atau audit download. Sampai tersedia, hanya ekspor sinkron yang dapat ditampilkan secara jujur; pusat ekspor persisten belum boleh diaktifkan.

### 8.9 — State, Kalkulasi, dan Responsif Laporan

- Default sort harus stabil dan seluruh laporan memakai server-side pagination.
- Export menggunakan query/filter yang sama; jumlah hasil dan generated-at/timezone tampil pada file.
- `available` selalu read-only: `max(0, on_hand - reserved - quarantined)`.
- Saldo akhir historis direkonstruksi dari ledger/snapshot tervalidasi, bukan dari `stock_balances` saat ini.
- Empty/no-result/loading/error mengikuti kontrak global; kegagalan ekspor tidak menghapus hasil tabel.
- Desktop menyediakan pemilih kolom; tablet memindahkan filter ke off-canvas; mobile memakai kartu ringkas dan menganjurkan ekspor untuk dataset lebar.

---

## MENU 9: ⚙️ SETTINGS
**Akses:** `@can('users.manage')` untuk tab Pengguna; `@can('roles.manage')` untuk tab Roles. Tidak ada permission Audit Log pada seeder saat ini.

### 9.1 — Manajemen Pengguna & Hak Akses
```text
+═════════════════════════════════════════════════════════════════════════════+
║  ⚙️  MANAJEMEN PENGGUNA & AKSES                                              ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [Tab: Pengguna ✔] [Tab: Roles & Permissions] [Audit Log: Target/Disabled]  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [➕ Tambah Pengguna Baru]   [Cari NIP atau Nama................]          ║
╠════════╦════════════╦════════════════╦═════════════════╦═══════════════════╣
║  NIP   ║  Nama      ║  Jabatan       ║  Role Spatie    ║  Aksi             ║
╠════════╬════════════╬════════════════╬═════════════════╬═══════════════════╣
║ 123456 ║ Saifudin   ║ Staf Logistik  ║ staf_gudang     ║ [✏️] [🔑 Akses]  ║
║ 234567 ║ Budiman    ║ Kepala Gudang  ║ kepala_gudang   ║ [✏️] [🔑 Akses]  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  ── DRAWER KANAN (Kelola Akses) ──────────────────────────────────────    ║
║  🔑 KELOLA AKSES: Saifudin (NIP 123456)                                    ║
║                                                                             ║
║  1. ROLE SPATIE (Hak Fitur): [staf_gudang ×] [Tambah Role ▼]               ║
║                                                                             ║
║  2. SCOPE GUDANG (Hak Data):                                                ║
║     MDN │ warehouse_staff │ 01-01-2026 │ tanpa akhir │ [✏️]                ║
║     JKT │ inventory_verifier│01-06-2026│ 31-12-2026  │ [✏️]                ║
║     [➕ Tambah Penugasan Gudang]                                            ║
║                                                                             ║
║  [💾 SIMPAN PERUBAHAN (AJAX)]                                               ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 9.2 — Roles & Permissions (Granular Spatie)
```text
+═════════════════════════════════════════════════════════════════════════════+
║  🔐 MANAJEMEN ROLES & PERMISSIONS (Spatie v7)                               ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Role           │ Permissions yang Dimiliki                    │ Aksi       ║
║  ───────────────┼──────────────────────────────────────────────┼─────────── ║
║  super_admin    │ Semua Permissions                            │ [✏️ Edit]  ║
║  kepala_gudang  │ transactions.view; transactions.approve_warehouse_head │ [✏️ Edit] ║
║  staf_gudang    │ transactions.view; transactions.check_warehouse        │ [✏️ Edit] ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  KLIK [✏️ Edit Role] → MODAL MATRIX PERMISSION:                            ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │ Role: kepala_gudang                                                 │   ║
║  │ ─────────────────────────────────────────────────────────────────── │   ║
║  │ Permission aktual (tanpa alias):                                  │   ║
║  │ [☑] transactions.view                                             │   ║
║  │ [☑] transactions.approve_warehouse_head                           │   ║
║  │ [☑] stock_opname.create_session                                   │   ║
║  │ [☑] stock_opname.approve                                          │   ║
║  │ [☑] reports.view                                                   │   ║
║  │                                          [💾 Simpan Permissions]   │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 9.3 — Audit Log (Target, Disabled pada Baseline)

Seeder belum memiliki permission seperti `audit.view`, dan schema belum memiliki audit-event global. Karena itu menu, route, API, dan tab Audit Log global **tidak dirender pada baseline**, termasuk bagi `users.manage`/`roles.manage`; permission administrasi pengguna tidak boleh otomatis membuka histori aktivitas. Wireframe berikut hanya target setelah permission dan sumber audit tersedia.
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📜 AUDIT LOG SISTEM                                                        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Filter: [User ▼] [Modul ▼] [Aksi ▼] [Tgl: 25-08-2026 📅]                ║
╠════════════╦══════════╦═══════════════╦══════════════╦═════════════════════╣
║ Timestamp  ║ User     ║ Aktivitas     ║ Referensi    ║ Perubahan/Keputusan ║
╠════════════╬══════════╬═══════════════╬══════════════╬═════════════════════╣
║ 10:15:32   ║ Budiman  ║ Trx.Approve   ║ 2202-BGR...  ║ pending → approved  ║
║ 09:30:11   ║ Saifudin ║ Trx.Checked   ║ 1101-MDN...  ║ CHECKED             ║
║ 08:00:05   ║ System   ║ Trx.Posted    ║ 1101-MDN...  ║ approved → posted   ║
╚════════════╩══════════╩═══════════════╩══════════════╩═════════════════════╝
```

### 9.4 — Kontrak Manajemen Pengguna

| Aspek | Detail |
|---|---|
| Akses | `users.manage`; pengguna tanpa izin tidak menerima metadata akun lain. |
| Filter | NIP/nama/email, status, role, gudang, fungsi, posisi, rentang last login. |
| Kolom | NIP, nama, email, status, jabatan utama, role, gudang aktif, last login, dan aksi. |
| Aksi | Tambah, detail/edit, aktifkan/nonaktifkan, reset password sesuai policy, kelola role, scope gudang, dan riwayat jabatan. Tidak ada hard delete. |

Form akun: employee number unik, nama, email unik, status active/inactive, serta password hanya saat create/reset dan tidak pernah ditampilkan kembali.

Drawer akses menggunakan baris assignment—bukan checkbox gudang sederhana:

| Gudang | Role scope | Role Spatie opsional | Valid from | Valid until | Status |
|---|---|---|---|---|---|

Gudang wajib aktif; tanggal akhir tidak boleh sebelum awal. Sumber: `users`, roles/pivots, `user_warehouse_assignments`, warehouses, position assignments.

> **Catatan schema:** unique key user–warehouse–role_scope membatasi penyimpanan beberapa periode historis dengan scope sama. UI harus mengikuti keputusan layanan, bukan menambahkan baris historis yang pasti gagal.

### 9.5 — Kontrak Roles & Permissions

Daftar menampilkan role name, guard, jumlah/ringkasan permission, jumlah user, updated time, dan Edit. Matrix menggunakan **nama permission aktual** seperti `transactions.view`, bukan singkatan `trx.view`.

- Permission dikelompokkan Master Data, Transactions, Stock Opname, Reports/Dashboard, dan Administration.
- Simpan menyinkronkan `role_has_permissions`; guard role/permission harus sama.
- Direct permission dari `model_has_permissions` tampil pada detail akses user sebagai override terpisah dari role.
- Tidak ada tombol hapus role karena schema belum memiliki `is_system`, soft delete, atau audit perubahan role.
- Konflik perubahan permission menghasilkan 409 dan muat ulang matrix terkini.

### 9.6 — Profil Saya, Password, dan MFA

Semua user terautentikasi dapat membuka profil sendiri: NIP read-only, nama, email, last login read-only, ubah password dengan password saat ini, password baru, dan konfirmasi. Role, scope, posisi, dan status akun read-only bagi pemilik akun.

> **Dependensi schema:** belum ada MFA secret, recovery codes, metode faktor, verified time, atau device history. UI tidak menampilkan switch/QR setup MFA seolah dapat disimpan; tampilkan informasi “MFA belum tersedia pada schema saat ini”.

### 9.7 — Preferensi Notifikasi

Pusat task real-time dapat diturunkan dari pending approval dan draft, tetapi preferensi email/in-app, read/unread, digest, quiet hours, dan delivery status memerlukan penyimpanan tersendiri.

> **Dependensi schema:** belum ada tabel notification maupun notification preferences. Halaman preferensi hanya menampilkan penjelasan kemampuan saat ini sampai schema tersedia.

### 9.8 — Batas Halaman Audit

Kontrak target memakai permission baru `audit.view`; akses lintas gudang tambahan memerlukan `audit.view_all_warehouses`, sedangkan default tetap warehouse-scoped. Sebelum kedua permission tersebut ditambahkan ke katalog/seeder dan global audit source tersedia, halaman 9.3/9.8 tetap disabled. Histori transaksi masih dapat dilihat di detail transaksi oleh pengguna dengan `transactions.view`, dan riwayat status material mengikuti gate Master Data—keduanya bukan pengganti Audit Log global.

Data yang dapat ditampilkan secara sah:

1. `transaction_histories`: activity, from/to status, metadata, actor, created time.
2. `transaction_approvals`: stage, required role, decision, notes, actor/position/role snapshot, watermark, decided time, IP/user-agent sesuai policy.
3. `material_status_assignments`: material, gudang, status, periode, reason, assigner, timestamps.

Filter: jenis aktivitas, nomor transaksi, actor, gudang, from/to status, decision, dan rentang waktu. Metadata JSON dibuka pada drawer terformat; data sensitif harus disamarkan.

> **Dependensi schema:** tidak ada audit log global before/after untuk users, roles, permissions, master, warehouse, label, lot, balance, export, atau akses dokumen. UI tidak boleh mengklaim tiga sumber di atas sebagai audit sistem lengkap.

### 9.9 — State dan Responsif Settings

- Semua list mengikuti server-side DataTable, empty/no-result/error/403/409 global.
- Inactive user tidak tersedia untuk assignment/approval baru tetapi tetap terlihat pada histori.
- Drawer akses dan permission matrix menjadi full-screen/accordion di mobile.
- Password/secret tidak pernah diprefill atau dimasukkan ke log/metadata error.
- Aksi sukses merefresh record dan menampilkan waktu update; toast bukan satu-satunya bukti perubahan.

---

## MODAL APPROVAL WATERMARK DIGITAL (Shared Component)
Modal ini dipanggil dari menu mana saja saat pejabat menekan tombol `[Review / Approve]`.

```text
+═════════════════════════════════════════════════════════════════════════════+
║  🔍 PERSETUJUAN TRANSAKSI: 2202-BGR-202608-0005              [✕ Tutup]      ║
╠════════════════════════════════╦════════════════════════════════════════════╣
║  PDF PREVIEW / FORM TRANSAKSI  ║  HISTORI & AKSI APPROVAL                  ║
║  ┌────────────────────────────┐║  Log:                                      ║
║  │ PERTAMINA — SURAT          │║  ✅ Draft — Budi S. (25-08 08:00)         ║
║  │ PENGELUARAN MATERIAL       │║  ✅ CHECKED — Agus (25-08 09:30)          ║
║  │ No: 2202-BGR-202608-0005   │║  ⏳ Menunggu Ka.Gudang (ANDA)            ║
║  │ ...                        │║                                            ║
║  │ [Stempel: CHECKED ✅]      │║  Lampiran Referensi:                       ║
║  │ [Belum ada stempel approval]│║  📎 SPK-2026-876.pdf [👁️ Buka] [📥 Unduh]║
║  │  Menunggu keputusan sah     │║                                            ║
║  └────────────────────────────┘║  Catatan Penolakan/Revisi:                 ║
║                                ║  [Textarea: Isi jika Tolak/Revisi...]     ║
╠════════════════════════════════╩════════════════════════════════════════════╣
║  POLICY: current_stage + required_role/assignee + scope + permission tahap  ║
║  [❌ TOLAK]  [🔄 KEMBALIKAN UNTUK REVISI]  [✅ CHECKED / APPROVED sesuai tahap]║
╚═════════════════════════════════════════════════════════════════════════════╝
```

---

## KONTRAK KOMPONEN BERSAMA

### S.1 Approval Modal

Modal selalu menampilkan nomor transaksi, status utama, current stage, required role, assignee, **scope transaksi saat ini**, preview PDF, dokumen, timeline, dan data keputusan sebelumnya. Scope transaksi saat ini berasal dari gudang asal/tujuan serta policy pengguna; ia bukan bukti scope yang dimiliki actor pada saat keputusan historis dibuat.

| Aksi | Aturan |
|---|---|
| CHECKED | Hanya pemeriksaan Staf Gudang/Staf Gudang Tujuan pada stage yang tepat; simpan `decision=approved` dengan `watermark_stamp=CHECKED`, karena CHECKED bukan nilai enum decision. |
| APPROVED | Hanya actor berwenang pada current stage; simpan `decision=approved` dengan `watermark_stamp=APPROVED`; pembuat tidak boleh menyetujui tahap reviewer transaksi sendiri. |
| REJECT | Notes wajib; keputusan terminal sesuai workflow. |
| REVISION | Notes wajib. Target kembali dipilih oleh transition map server, bukan input bebas. Layanan menyimpan `return_to_stage` pada `transaction_histories.metadata`, memperbarui `inventory_transactions.current_stage` secara atomik, dan hanya membuka field yang diizinkan. Bila kontrak metadata/versioned transition map belum tersedia, aksi revision dinonaktifkan. |

Action bar tidak digate oleh satu permission statis. Policy harus mencocokkan `current_stage`, pending `transaction_approvals.required_role`, optional `assigned_user_id`, warehouse scope, serta permission tahap: antara lain `transactions.check_warehouse` untuk CHECKED, `transactions.approve_warehouse_head` untuk Kepala Gudang, dan `transactions.verify_inventory` untuk verifikasi Persediaan. Request yang dipaksa di luar kombinasi tersebut tetap menghasilkan 403.

**Aturan render watermark yang tidak boleh dilonggarkan:** stempel hanya dirender bila record memiliki `decision='approved'` **dan** `decided_at IS NOT NULL`. Nilai `watermark_stamp` tidak pernah cukup sebagai syarat karena kolom tersebut memiliki default `APPROVED`, termasuk pada row pending. Untuk `pending`, `rejected`, `revision_requested`, decision kosong, atau `decided_at` kosong, renderer PDF/preview/export wajib meniadakan seluruh stempel—bukan menampilkan stempel redup/kosong. Nilai stamp yang diizinkan tetap harus cocok dengan stage (`CHECKED` untuk pemeriksaan dan `APPROVED` untuk persetujuan).

Request membawa `lock_version` dan idempotency key. Bila stage telah berpindah, modal menampilkan keputusan terbaru dan menutup action panel. Snapshot nama, posisi, role, watermark sah, decision time, IP/user-agent berasal dari record keputusan `transaction_approvals`, bukan profil terkini.

> **Dependensi schema approval scope:** `transaction_approvals` belum menyimpan warehouse-scope snapshot actor pada waktu keputusan. Timeline hanya boleh menampilkan **Scope transaksi saat ini** dan tidak boleh melabelinya “Scope saat approval”. Tambahkan snapshot gudang/role-scope/effective assignment pada approval evidence sebelum scope historis ditampilkan atau dicetak.

### S.2 Attachment Manager

- Menampilkan document type code, nama, MIME, ukuran, checksum status, versi, uploader, uploaded time, preview/download.
- Validasi extension, MIME, ukuran, checksum, dan antivirus hook berlangsung server-side.
- Upload gagal dapat di-retry tanpa menghapus draft atau file lain yang berhasil.
- Setelah posted, file lama tidak ditimpa; upload membuat versi baru.
- Preview/download menggunakan authorization dan signed URL pendek; expiry menyediakan Generate Link Baru.
- Mobile menggunakan daftar kartu dan preview full-screen.

### S.3 Timeline Transaksi

Timeline menggabungkan status history dan approval dalam urutan waktu, tetapi membedakan event workflow dari keputusan approval. Setiap item menampilkan activity/stage, from/to status, actor/snapshot, timestamp/timezone, notes, watermark, dan metadata yang diizinkan.

Filter opsional: semua, status, approval, dokumen, posting. Empty state hanya muncul pada draft baru; kegagalan memuat timeline tidak menyembunyikan header transaksi.

### S.4 Status Badge dan Action Bar

`StatusBadge` menerima status utama atau status domain yang sudah didefinisikan. Ia selalu berisi ikon + teks + tooltip makna. `WorkflowStageBadge` terpisah dan menyebut pemegang tindakan berikutnya.

Action bar desktop berada di kanan header; pada tablet/mobile menjadi sticky bottom bar dengan satu primary action dan overflow untuk aksi sekunder. Tombol tidak berizin tidak dirender; tombol yang tidak valid karena status tampil disabled hanya bila penjelasannya membantu pengguna.

### S.5 QR Scanner dan Label Print

- Scanner memeriksa permission kamera, menampilkan frame panduan, input hardware scanner, hasil, Scan Lagi, dan status koneksi.
- Token invalid/nonaktif/out-of-scope tidak membocorkan data material.
- Label preview memperlihatkan data non-sensitif; QR token tidak mengandung harga.
- Print baseline memakai modal konfirmasi dan hanya dapat membuktikan `printed_count`/`last_printed_at`; UI tidak menampilkan `printed_by` karena field tersebut tidak ada.
- Reprint/void menargetkan modal alasan wajib dan audit event actor/time, tetapi kedua CTA dinonaktifkan pada baseline karena `stock_labels` tidak memiliki reason, void actor/time, dan belum ada audit-event global. Status `reprinted` legacy tetap dapat ditampilkan read-only.
- Tidak ada validasi transaksi secara offline.

### S.6 Dialog Konfirmasi dan Feedback

| Komponen | Pemakaian |
|---|---|
| Confirm standard | Submit, complete session, atau perubahan status non-destruktif; sebut hasil dan tahap berikutnya. |
| Confirm danger | Cancel, reject, void label setelah audit lifecycle tersedia, atau nonaktifkan master/user; sebut object, dampak, dan alasan bila wajib. |
| Toast region | Hasil singkat sukses/gagal; diumumkan ke screen reader dan tidak menggantikan state halaman. |
| Inline alert | Validasi bisnis, dokumen kurang, saldo berubah, atau widget gagal. |
| Conflict modal | 409/lock_version; tampilkan pelaku/waktu bila tersedia dan CTA Muat Ulang. |
| Forbidden panel | 403; hapus data lama dari DOM dan arahkan kembali ke halaman aman. |

### S.7 Kontrak Request AJAX/API

| Operasi | Loading | Sukses | Error utama |
|---|---|---|---|
| List/Select2 | Skeleton/spinner tanpa menghapus filter | Render data ter-scope | 403, 422 filter invalid, 429, 5xx + Retry. |
| Create/Edit Draft | Disable tombol aktif, status Menyimpan | Update nomor/version/time | 422 inline, 409 conflict, upload error. |
| Submit | Lock wizard, tampilkan proses validasi | Status/stage/timeline baru | Dokumen kurang, stok berubah, 409. |
| Check/Approve/Reject/Revision | Disable seluruh decision button | Modal menjadi read-only dan timeline refresh | 403 bukan giliran, 409 sudah diproses, 422 notes wajib. |
| Posting | Full action lock + progress | Movement/saldo/bukti posting | Idempotent result, stok berubah, 5xx dengan correlation ID. |
| QR scan | Scanner busy per token | Detail aman/row terisi | Invalid, inactive, duplicate, out-of-scope, rate limit. |
| Export | Tombol queued/progress | Download atau notification | No data, 403, rate limit, job failed. |

## MATRIKS TRACEABILITY DATABASE KE TAMPILAN

| No. | Tabel | Tampilan utama | Kontrak UI |
|---:|---|---|---|
| 1 | `permissions` | Roles & Permissions | Katalog izin aktual per guard. |
| 2 | `roles` | Roles, akses pengguna | Role global; tidak ada delete tanpa proteksi schema. |
| 3 | `model_has_permissions` | Detail akses pengguna | Direct permission dipisahkan dari role. |
| 4 | `model_has_roles` | Detail akses pengguna | Penetapan satu/lebih role. |
| 5 | `role_has_permissions` | Permission matrix | Sinkronisasi hak per role. |
| 6 | `users` | Pengguna, actor transaksi/approval/count/posting | NIP/email unik, status, last login. |
| 7 | `organizational_functions` | Master organisasi | Tree fungsi dan parent. |
| 8 | `positions` | Master posisi | Posisi terkait fungsi. |
| 9 | `user_position_assignments` | Riwayat jabatan pengguna | Masa berlaku dan primary. |
| 10 | `warehouses` | Switcher, master, scope, laporan | Gudang aktif untuk data baru; timezone/region/alamat. |
| 11 | `warehouse_locations` | Pohon lokasi, selector, stock/QR | Hierarki dan tipe zone/rack/bin/staging/transit/quarantine. |
| 12 | `user_warehouse_assignments` | Drawer akses dan seluruh policy scope | Role scope dan masa berlaku. |
| 13 | `material_classifications` | Katalog/filter/dashboard | Kode/nama/sort/active. |
| 14 | `material_categories` | Katalog/filter/dashboard | Kode/nama/sort/active. |
| 15 | `uoms` | Material/item/count/report | Presisi mengatur format/validasi qty. |
| 16 | `materials` | Katalog, detail, seluruh transaksi/laporan | KIMAP unik, master identitas, serialized/active. |
| 17 | `material_status_assignments` | Detail material, dashboard, rekonsiliasi | FM/SM/PDS/DS efektif per gudang. |
| 18 | `stock_lots` | Lot, picker, QR, ledger | Perolehan/harga/original/remaining/status read-only dari proses. |
| 19 | `stock_labels` | Label, scan, print/reprint/void | Token/card/location/qty/print lifecycle. |
| 20 | `inventory_movements` | Ledger, detail posting, kartu material | Append-only; tidak ada edit/delete. |
| 21 | `stock_balances` | Dashboard/posisi/picker/report | Proyeksi kini dan available turunan. |
| 22 | `stock_reservations` | Outbound/transfer/monitor | Reserved/consumed/status/expiry. |
| 23 | `transaction_subtypes` | Master subtype dan wizard | Main type/requires reference/active. |
| 24 | `inventory_transactions` | Header/list/detail seluruh transaksi | Status/stage/number/warehouses/timestamps/lock version. |
| 25 | `inventory_transaction_items` | Grid item/check/detail | Requested/processed/price/value/locations/lot/label. |
| 26 | `transaction_documents` | Attachment Manager | Type code, metadata, checksum, version, uploader/time. |
| 27 | `transaction_approvals` | Inbox/modal/timeline/watermark | Stage, role, assignee, decision, snapshots, audit teknis. |
| 28 | `transaction_histories` | Timeline/audit transaksi | Activity/from/to/metadata/actor/time. |
| 29 | `stock_count_sessions` | Sesi opname/inventarisasi | Type/period/snapshot/scope/status/creator/approver. |
| 30 | `stock_count_items` | Grid manual/QR/review/report | System/physical/variance/explanation/tokens/counter. |
| 31 | `inventory_adjustments` | Adjustment list/detail/approval | Header/reason/status/dua approver/posted time; item belum tersedia. |

## DEPENDENSI DAN GAP SCHEMA

1. **Historical/as-of balance:** `stock_balances` hanya posisi kini; membutuhkan rekonstruksi ledger tervalidasi atau snapshot historis.
2. **Low-stock threshold:** belum ada minimum stock/reorder point per material–gudang–lokasi.
3. **Aturan dokumen per subtype:** belum ada master document types/rule; `requires_reference` tidak cukup.
4. **Referensi transaksi:** `transaction_references` disebut rancangan tetapi belum memiliki migration.
5. **Scope lokasi stock count:** masih string, bukan FK/multi-location model.
6. **Tim inventarisasi:** belum ada `stock_count_assignments`, role anggota, surat tugas, atau assignment status.
7. **Detail adjustment:** belum ada item adjustment dan link kuat adjustment–transaction–movement.
8. **Audit global:** belum ada audit event before/after untuk seluruh master, user, access, label, export, dan akses dokumen.
9. **Notifikasi/preferensi:** belum ada notification store, read state, channel preference, delivery status, atau quiet hours.
10. **Pusat ekspor:** belum ada export job, progress, owner, file metadata, TTL, dan audit download.
11. **MFA:** belum ada secret/recovery code/method/device/verified time.
12. **Relasi logis tanpa FK:** receipt item lot, reservation item, dan transaction/item/reversal ledger perlu penanganan referensi yatim.
13. **Konsistensi lot–lokasi:** database belum menjamin lokasi label berada pada gudang lot.
14. **Status material overlap:** belum ada constraint untuk mencegah dua assignment efektif bersamaan.
15. **Riwayat scope gudang:** unique user–warehouse–role_scope membatasi beberapa periode historis dengan scope sama.
16. **Saved view:** belum ada penyimpanan preferensi filter/sort/kolom per pengguna; URL state hanya menjadi fallback lokal.
17. **Reset password dan pencabutan sesi:** token reset, expiry/single-use semantics, session store, serta mekanisme revoke membutuhkan kontrak Laravel/Fortify dan infrastruktur yang belum terlihat pada `database.md`.
18. **Konfigurasi SLA:** belum ada aturan durasi per stage, kalender kerja, timezone aturan, pengecualian hari libur, dan effective period; baseline hanya menampilkan elapsed age.
19. **Approval evidence transaksi:** `watermark_stamp` default `APPROVED` berisiko false stamp sehingga render wajib digate oleh decision approved + decided_at. Warehouse-scope snapshot saat approval juga belum tersedia.
20. **Approval stock count:** satu `approved_by` tidak cukup untuk CHECKED/APPROVED multi-actor, notes, timestamp, snapshot, urutan, dan watermark Stock Opname/Inventarisasi.
21. **Audit adjustment:** belum ada item correction, creator, reject actor/note/time, timestamp tiap approval, `lock_version`, idempotency record, dan relasi adjustment–transaction–movement.
22. **Lineage lot transfer:** lot tujuan wajib dibuat di gudang tujuan, tetapi belum ada `source_lot_id`/mapping lot transfer yang menjamin lineage sumber–tujuan.
23. **Evidence scan QR count:** non-serialized scan membutuhkan quantity snapshot per token; bila JSON `scanned_label_tokens` hanya menyimpan token, struktur evidence harus diperluas sebelum fitur diaktifkan.
24. **Invariant label serialized:** belum ada check constraint yang mewajibkan `label_quantity=1` dan `remaining_quantity` hanya 0/1 untuk material serialized; layanan harus memvalidasi dan menolak data tidak konsisten.
25. **UOM ledger:** `inventory_movements` tidak menyimpan UOM dan belum ada tabel konversi. Posting harus terbukti ternormalisasi ke `materials.base_uom_id` sebelum laporan qty lintas transaksi dianggap resmi.
26. **Permission nilai:** seeder belum memiliki permission nilai terpisah; baseline `transactions.view`/`reports.view` melihat nilai sesuai scope. Tambahkan `inventory_value.view`/`reports.view_value` dan policy server bila nilai perlu dibatasi.
27. **Audit label:** belum ada printed_by, alasan/actor/waktu reprint atau void, maupun audit-event global; reprint/void diaudit tidak boleh diaktifkan sebelum evidence tersedia.
28. **Konfigurasi subtype transaksi:** kode bisnis penerimaan, retur, pengeluaran, transfer (termasuk 2204), dan write-off harus tersedia aktif dengan `main_type` yang tepat; UI menolak create terkait bila seed/configuration tidak tervalidasi dan tidak pernah mengandalkan primary key tetap.
29. **Kontrak revision:** target kembali membutuhkan transition map versioned dan metadata `return_to_stage`; aksi revision tidak aktif sampai kontrak layanan tersebut tersedia.
30. **Permission dashboard operasional:** seeder saat ini hanya memberi dashboard kepada Accounting/Management; role operasional membutuhkan `dashboard.view_assigned_warehouse` sebelum Dashboard 1B/1C diaktifkan.
31. **Submit/handoff Holder:** Holder memiliki create tanpa submit dan belum ada model handoff draft. Baseline menyembunyikan Submit/Handoff sampai permission atau schema penugasan ditambah.
32. **Audit access:** belum ada `audit.view`/`audit.view_all_warehouses` maupun global audit source; halaman Audit Log global disabled.
33. **Alokasi quarantine:** partial/multi-label quarantine membutuhkan allocation grain dan lineage split-label. Baseline hanya menerima whole-label yang dapat dipindah ke lokasi quarantine secara atomik.

## CHECKLIST KELENGKAPAN SETIAP TAMPILAN

Sebelum implementasi layar dinyatakan siap, reviewer memastikan:

- [ ] Tujuan, entry point, breadcrumb, dan return destination jelas.
- [ ] Permission, role variation, warehouse/data scope, dan 403 ditulis.
- [ ] Tabel/field sumber dan field turunan disebut.
- [ ] Filter, default, reset, sort, pagination, dan URL state dijelaskan.
- [ ] Kolom/form field, format, editable/read-only, dan kondisi tampil lengkap.
- [ ] Primary/secondary/row/bulk action beserta syarat statusnya lengkap.
- [ ] Status utama, current stage, actor berikutnya, dan terminal state terlihat.
- [ ] Validasi client/server, dokumen, qty, upload, concurrency, dan idempotency dijelaskan.
- [ ] Loading, empty, no-result, error, 403, 409, offline, dan retry tersedia.
- [ ] Desktop, tablet, mobile, keyboard, focus, label, ikon, dan kontras ditentukan.
- [ ] Dependensi schema ditandai dan tidak disamarkan sebagai data yang sudah ada.
- [ ] Deep-link, notification, report, export, preview, dan audit mengikuti policy yang sama.

---

*Dokumen ini merupakan referensi visual dan kontrak implementasi untuk antarmuka Blade, AJAX, Yajra DataTables, Select2, Bootstrap 5, Spatie Laravel-Permission v7, Laravel Policy, serta warehouse-scoped access.*
