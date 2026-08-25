# Rancangan Desain Antarmuka Keseluruhan (UI/UX)
## Sistem Pengelolaan Persediaan Material dan Gudang

**Versi:** 2.0 (Ekspansi Keseluruhan Aplikasi)  
**Tanggal:** 25 Agustus 2026  
**Dokumen Acuan:** `rancangan-aplikasi-persediaan-gudang.md`, `database.md`, `dfd.md`, `Detail.xlsx`

Dokumen ini memberikan gambaran visual lengkap (wireframe konseptual) dan alur interaksi (*user flow*) untuk seluruh modul antarmuka aplikasi. Seluruh desain mengadopsi prinsip **AJAX DataTables Server-Side**, **Modal Interaktif**, dan dibatasi secara ketat oleh **Spatie Laravel Permission v7**.

---

## 1. Tata Letak Global (Global Layout)

Sistem menggunakan layout standar aplikasi *enterprise* (misal: Velzon Bootstrap 5 / AdminLTE).

### 1.1 Top Navigation Bar (Header)
- **[Logo Pertamina]** & Nama Aplikasi.
- **Warehouse Context Switcher**: Dropdown (AJAX) untuk memilih/berpindah konteks gudang aktif (hanya menampilkan gudang yang diotorisasi via `user_warehouse_assignments`).
- **Notification Inbox**: Lonceng badge notifikasi tugas pending (*Pending Approvals*).
- **User Profile Dropdown**: Nama, Jabatan, Role Spatie, Profil, & Logout.

### 1.2 Sidebar Menu (Role-Based Visibility)
Menu otomatis di-hide/show menggunakan `@can()`:
- 📊 **Dashboard** (Semua Role)
- 📦 **Master Data** (Katalog Material, Lokasi Rak/Bin) -> `@can('master_data.view')`
- 📥 **Inbound (Penerimaan)** -> `@can('transactions.view')`
- 📤 **Outbound (Pengeluaran & Retur)**
- 🔄 **In-Transit (Pemindahan)**
- 🛑 **Quarantine (Penyisihan/Write-Off)**
- 📋 **Stock Opname & Inventarisasi** -> `@can('stock_opname.input_count')`
- 📈 **Reporting & Analytics** -> `@can('reports.view')`
- ⚙️ **Settings** (Pengguna & Role Spatie) -> `@can('users.manage')`

---

## 2. Dashboard Landing Pages (Berdasarkan Role Spatie)

### 2.1 Dashboard Management & Accounting (Tampilan Analitik)
Fokus: Pemantauan nilai finansial (Grafik Pie/Line/Bar dari view `vw_dashboard_inventory_composition`).

```text
+-----------------------------------------------------------------------------+
| Filter: [Semua Region ▼]  [Semua Gudang ▼]  [Tahun: 2026 ▼]  [Terapkan] |
+-----------------------------------------------------------------------------+
| [Card: Total Nilai Persediaan] | [Card: Total MPS]    | [Card: Total ABT]   |
|         Rp 45.500.000.000      |   Rp 30.000.000.000  |  Rp 15.500.000.000  |
+-----------------------------------------------------------------------------+
| +-----------------------------------+ +-----------------------------------+ |
| | 🥧 PIE CHART: Komposisi Nilai MPS | | 🥧 PIE CHART: Komposisi Nilai ABT | |
| | (Berdasarkan Kategori Material &  | | (Berdasarkan Status FM, SM, PDS,  | |
| |  Status FM, SM, PDS, DS)          | |  DS)                              | |
| +-----------------------------------+ +-----------------------------------+ |
+-----------------------------------------------------------------------------+
| 📈 LINE CHART: Tren Saldo Akhir Bulanan (MPS vs ABT - Sumbu X: Bulan)       |
+-----------------------------------------------------------------------------+
```
> *Interaksi*: Mengubah filter dropdown memicu *AJAX request* untuk re-render chart secara instan tanpa me-reload halaman.

### 2.2 Dashboard Kepala Gudang & Persediaan (Tampilan Approval)
Fokus: Eksekusi *workflow approval* & pemantauan antrean dokumen masuk.

```text
+-----------------------------------------------------------------------------+
| 🔔 TASK INBOX: PENDING APPROVALS (Membutuhkan Tindakan Anda)              |
+-----------------------------------------------------------------------------+
| [Cari Dokumen............]                              [Export Excel/PDF]  |
|                                                                             |
| No Transaksi | Tipe        | Pemohon    | Status Workflow   | Aksi          |
| -------------|-------------|------------|-------------------|---------------|
| 1101-MDN-001 | Penerimaan  | Budiman    | Menunggu Ka.Gudang| [Review 🔍]  |
| 2202-BGR-005 | Pengeluaran | Susi Susil | Verifikasi Harga  | [Review 🔍]  |
+-----------------------------------------------------------------------------+
```

### 2.3 Dashboard Staf Gudang (Tampilan Operasional)
Fokus: Entri data, Scan QR, pengisian hasil hitung fisik.

```text
+-----------------------------------------------------------------------------+
| PINTASAN CEPAT (Quick Actions)                                              |
| [➕ Penerimaan]  [📤 Pengeluaran]  [🔄 Pemindahan]  [📷 SCAN QR LABEL]     |
+-----------------------------------------------------------------------------+
| 📋 CHECKLIST PEKERJAAN HARI INI                                             |
| 1. Draft Pengeluaran BGR-002 (2 Item) -> [Lanjutkan Edit ✏️]              |
| 2. Input Hitung Fisik Stock Opname RAK-A1 -> [Buka Form Opname 🔢]          |
+-----------------------------------------------------------------------------+
```
> *Interaksi*: Klik `[SCAN QR LABEL]` memanggil library kamera HMTL5. Dekode UUID QR mengarahkan staf langsung ke halaman Transaksi/Item spesifik.

---

## 3. Desain Halaman Master Data & Pengguna (Bank Data)

Sistem memiliki antarmuka khusus untuk mengelola Bank Data. Seluruh antarmuka ini dilindungi oleh permission Spatie `@can('master_data.view')` dan menggunakan **Yajra DataTables** untuk performa tinggi.

### 3.1 Manajemen Katalog Material & Atribut
Halaman ini memuat *Nav-Tabs* (AJAX Tab) untuk memisahkan entri data Material, Klasifikasi, Kategori, dan UOM.
```text
+-----------------------------------------------------------------------------+
| 📦 MASTER DATA MATERIAL                                                     |
| [Tab: Katalog Material] [Tab: Klasifikasi] [Tab: Kategori] [Tab: UOM]       |
+-----------------------------------------------------------------------------+
| [➕ Tambah Material Baru]   [📥 Import Excel (.xlsx)]   [📤 Export Template] |
+-----------------------------------------------------------------------------+
| (DataTables AJAX)                                                           |
| KIMAP      | Nama Barang            | Klasifikasi| Kategori | UOM | Aksi    |
| -----------|------------------------|------------|----------|-----|---------|
| 10001928   | VALVE GATE 4" SS       | MPS        | Tubular  | EA  | [✏️][🗑️]|
| 90001123   | LAPTOP ASUS ROG        | ABT        | Elektron | SET | [✏️][🗑️]|
+-----------------------------------------------------------------------------+
```
> *Interaksi Modal*: Mengklik `[➕ Tambah Material]` membuka form *Offcanvas* / Modal AJAX. Input `Klasifikasi` dan `Kategori` menggunakan Dropdown *Select2* (AJAX Search).

### 3.2 Manajemen Lokasi Gudang (Warehouses & Racks/Bins)
Digunakan oleh Super Admin untuk mendaftarkan 17 Gudang dan rincian lokasi fisik (*Zone/Rack/Bin*) di dalamnya.
```text
+-----------------------------------------------------------------------------+
| 🏢 MASTER DATA GUDANG & LOKASI                                              |
| [➕ Tambah Gudang]                                                          |
+-----------------------------------------------------------------------------+
| (DataTables AJAX - Detail Row / Nested Table)                               |
| ⊕ Kode | Nama Gudang          | Region | Zona Waktu   | Status | Aksi       |
| --------------------------------------------------------------------------- |
| ⊖ MDN  | Gudang Utama Medan   | REG-1  | Asia/Jakarta | Aktif  | [✏️][⚙️]  |
|    L Lokasi Rak/Bin (Sub-Tabel) [➕ Tambah Rak]                            |
|      - RAK-A1 (Tipe: Rack) [✏️]                                             |
|      - ZONE-B (Tipe: Zone) [✏️]                                             |
| ⊕ BGR  | Gudang Utama Bogor   | REG-3  | Asia/Jakarta | Aktif  | [✏️][⚙️]  |
+-----------------------------------------------------------------------------+
```
> *Interaksi AJAX*: Mengklik tombol ekspansi `⊕` akan melakukan HTTP GET untuk mengambil daftar `warehouse_locations` (Rak/Bin) yang terkait dengan gudang tersebut dan merendernya dalam *Sub-Tabel* DataTables.

### 3.3 Manajemen Subtipe Transaksi & Workflow
Digunakan untuk mengontrol referensi transaksi dan dokumen wajib.
```text
+-----------------------------------------------------------------------------+
| 🔀 REFERENSI SUBTIPE TRANSAKSI                                              |
| (Tabel Read-Only / Admin Edit)                                              |
| Kode | Nama Subtipe                      | Grup        | Dokumen Wajib      |
| -----|-----------------------------------|-------------|--------------------|
| 1101 | Penerimaan Pembelian Rutin        | Penerimaan  | PO, DO             |
| 2202 | Pengeluaran Operasional Gudang    | Pengeluaran | SPK                |
| 5501 | Penyisihan / Write-Off (Quarantine)| Penyisihan | Surat_GH_OMM       |
+-----------------------------------------------------------------------------+
```

### 3.4 Manajemen Hak Akses, Pengguna & Scope Gudang
Menetapkan otorisasi ganda: **Spatie Roles** dan **Warehouse Scoping**.
```text
+-----------------------------------------------------------------------------+
| 👥 MANAJEMEN PENGGUNA & AKSES (USER SETTINGS)                               |
+-----------------------------------------------------------------------------+
| [Tabel Daftar Pengguna - DataTables AJAX]                                   |
| NIP     | Nama      | Jabatan        | Role Spatie       | Aksi             |
| 123456  | Saifudin  | Staf Logistik  | staf_gudang       | [Kelola Akses 🔑]|
+-----------------------------------------------------------------------------+
> KETIKA KLIK [Kelola Akses 🔑] -> MUNCUL DRAWER KANAN (Offcanvas AJAX):
+-----------------------------------------------------------------------------+
| 🔑 KELOLA AKSES USER: Saifudin (123456)                                     |
| --------------------------------------------------------------------------- |
| 1. PENETAPAN SPATIE ROLE (Hak Fitur Aplikasi):                              |
|    Role Utama: [staf_gudang ▼]                                              |
|                                                                             |
| 2. PENETAPAN SCOPE GUDANG (Hak Akses Fisik / Warehouse Scoping):            |
|    Pilih Gudang yang ditugaskan kepada user ini:                            |
|    [☑] MDN - Gudang Utama Medan                                             |
|    [☐] BGR - Gudang Utama Bogor                                             |
|    [☑] JKT - Gudang Utama Jakarta                                           |
|                                                                             |
| [SIMPAN PERUBAHAN VIA AJAX]                                                 |
+-----------------------------------------------------------------------------+
```
> *Backend Integration*: Saat tombol "Simpan" ditekan, backend mengeksekusi sinkronisasi ganda:
> 1. `$user->syncRoles(['staf_gudang'])` (Spatie)
> 2. Sinkronisasi tabel `user_warehouse_assignments` untuk Gudang MDN & JKT (Mempengaruhi `WarehousePolicy` & filtering kueri data).

---

## 4. Desain Halaman Transaksi & Formulir Input (AJAX Grid)

### 4.1 Halaman Daftar Transaksi (Transaction History)
```text
+-----------------------------------------------------------------------------+
| Riwayat Transaksi Keluar/Masuk                                              |
| Filter: [Date Range 📅] [Semua Subtipe ▼] [Gudang Tujuan ▼] [Status ▼]    |
+-----------------------------------------------------------------------------+
| No Transaksi | Tipe/Subtipe | Tgl Trx    | Status / Tahap     | Aksi        |
| -------------|--------------|------------|--------------------|-------------|
| 2201-MDN-001 | Pengeluaran  | 25-08-2026 | [🟢 APPROVED]      | [Lihat 👁️] |
| 5501-JKT-002 | Penyisihan   | 24-08-2026 | [🟠 PENDING GUDANG]| [Review 🔍] |
| 1102-BGR-009 | Penerimaan   | 23-08-2026 | [⚪ DRAFT]         | [Edit ✏️]   |
+-----------------------------------------------------------------------------+
```

### 4.2 Formulir Input Transaksi (Form Mode: Create/Edit Draft)
Form ini dirancang menghindari reload halaman. Pengecekan stok dilakukan instan.
```text
+-----------------------------------------------------------------------------+
| FORMULIR PENGELUARAN MATERIAL (DRAFT)                                       |
+-----------------------------------------------------------------------------+
| Gudang Asal  : [MDN - Medan (Readonly)]                                     |
| Subtipe Trx  : [2202 - Pengeluaran Operasional ▼]                           |
| Lampiran Dok : [Upload Surat DO/SPK (PDF) 📎]  (*Mandatory validation)      |
| Keterangan   : [Input teks bebas...]                                        |
+-----------------------------------------------------------------------------+
| DETAIL ITEM MATERIAL                                                        |
| +-------------------------------------------------------------------------+ |
| | Cari KIMAP/Nama Barang | Sisa Stok Gudang | Qty Diminta | UOM | Hapus | |
| | [Select2 AJAX...... ▼] | 15 EA (Readonly) | [  5  ]     | EA  | [🗑️] | |
| | [Select2 AJAX...... ▼] | 2 EA  (Readonly) | [  3  ] <!! | EA  | [🗑️] | |
| +-------------------------------------------------------------------------+ |
| *Peringatan: Qty Diminta melebihi sisa stok (2 EA).                         |
|                                                                             |
| [➕ Tambah Baris Material]               [📷 Scan QR Barang]                 |
+-----------------------------------------------------------------------------+
| [Batal]                                   [Simpan Draft] [Ajukan Approval 🚀]|
+-----------------------------------------------------------------------------+
```

---

## 5. Desain Halaman Approval & Watermark Digital (Modal AJAX)

Ketika pejabat (Kepala Gudang / Fungsi Persediaan) mengklik `[Review 🔍]`, sebuah **Modal Besar** akan muncul di tengah layar.
```text
+-----------------------------------------------------------------------------+
| 🔍 PERSETUJUAN TRANSAKSI: 2202-BGR-005                                [X]   |
+-----------------------------------------------------------------------------+
| KIRI: PREVIEW DOKUMEN & FORM          | KANAN: HISTORI AUDIT & AKSI         |
| +-----------------------------------+ | +---------------------------------+ |
| |  [PDF VIEWER / FORM RENDER]       | | Log Approval:                     | |
| |  Pertamina Inventory System       | | 1. ⚪ Draft (Budi - 10:00)        | |
| |  No: 2202-BGR-005                 | | 2. 🟢 Checked (Staf - 10:30)      | |
| |  ...                              | | 3. 🟡 Menunggu Kepala Gudang      | |
| |  [Stempel 'CHECKED' Muncul]       | |                                   | |
| |  [Stempel 'APPROVED' (Blank)]     | | Lampiran Dokumen Referensi:       | |
| |  ...                              | | 📎 PO-12345.pdf [Unduh/Lihat]     | |
| +-----------------------------------+ | +---------------------------------+ |
|                                       | Catatan Penolakan/Revisi:           |
|                                       | [Textarea.......................]   |
+-----------------------------------------------------------------------------+
| [Tolak Transaksi ❌]   [Kembalikan untuk Revisi 🔄]   [✅ BUBOHKAN APPROVED] |
+-----------------------------------------------------------------------------+
```
> *Interaksi AJAX*:
> Saat `[✅ BUBOHKAN APPROVED]` ditekan, request dikirim ke backend. Sistem (dompdf/snappy) akan merender ulang PDF, meletakkan gambar stempel **APPROVED** di kolom tanda tangan elektronik, lalu menyimpannya ke S3 Private Server. DataTables di-refresh instan.

---

## 6. Desain Halaman Stock Opname & Hitung Fisik (Inventarisasi)

### 6.1 Input Form Hitung Fisik (Editable Grid)
Dirancang khusus agar Staf Gudang / Tim Inventarisasi bisa menginput saldo fisik di lapangan menggunakan Tablet/iPad.

```text
+-----------------------------------------------------------------------------+
| SESI STOCK OPNAME: SO-MDN-202608-01 (Gudang Medan, RAK-A1)                  |
+-----------------------------------------------------------------------------+
| Tipe Input: [🔘 Metode 1: Manual Grid]  [🔘 Metode 2: Scan QR Agregasi]      |
+-----------------------------------------------------------------------------+
| Daftar Material (Filter: RAK-A1)                                            |
|                                                                             |
| KIMAP    | Nama Barang       | Qty Sistem | QTY FISIK | Selisih | Catatan   |
|----------|-------------------|------------|-----------|---------|-----------|
| 10001928 | Valve Gate 4"     | 50 EA      | [ 50 ] EA | 0       | [       ] |
| 10002133 | Pipa Seamless 8"  | 10 MTR     | [  8 ] MTR| -2 MTR  | [Rusak..] |
| 90001123 | Laptop Asus       | 2 SET      | [  2 ] SET| 0       | [       ] |
+-----------------------------------------------------------------------------+
| *Jika ada selisih, kolom 'Catatan' WAJIB DIISI sebelum bisa disimpan.       |
|                                                                             |
| [Simpan Hasil Hitung Fisik 💾]          [Ajukan Approval Sesi Inventarisasi] |
+-----------------------------------------------------------------------------+
```
> *Interaksi*: 
> - Angka selisih dihitung *real-time* via JavaScript saat user mengetik `QTY FISIK`. 
> - Kolom catatan akan berubah warna jadi merah (harus diisi) jika selisih != 0.

---
*Dokumen desain UI/UX lengkap ini menjadi fondasi bagi tim Frontend (menggunakan Bootstrap 5, AJAX, DataTables, Select2) untuk merakit antarmuka aplikasi secara presisi.*
