# Rancangan Desain Antarmuka Dashboard (UI/UX)
## Sistem Pengelolaan Persediaan Material dan Gudang

**Versi:** 1.0  
**Tanggal:** 25 Agustus 2026  
**Dokumen Acuan:** `rancangan-aplikasi-persediaan-gudang.md`, `database.md`, `dfd.md`, `Detail.xlsx`

Dokumen ini memberikan gambaran visual (wireframe konseptual) dan alur interaksi (*user flow*) dari antarmuka Dashboard. Dashboard dirancang secara dinamis menggunakan **Spatie Laravel Permission v7** (Role-Based Access Control) dan **AJAX DataTables**, sehingga tampilan akan menyesuaikan secara otomatis berdasarkan Peran (Role) dan Hak Akses Gudang (Warehouse Scope) dari pengguna yang login.

---

## 1. Tata Letak Global (Global Layout)

Sistem menggunakan layout standar aplikasi *enterprise* (seperti AdminLTE atau Velzon Bootstrap 5).

### 1.1 Top Navigation Bar (Header)
- **[Logo Pertamina]** & Nama Aplikasi (Inventory System).
- **Warehouse Context Switcher**: Dropdown (AJAX) untuk memilih gudang yang sedang aktif (hanya menampilkan gudang sesuai `user_warehouse_assignments`).
- **Task Inbox / Notification Bell**: Menampilkan badge merah (jumlah tugas pending). Diklik akan memunculkan daftar *Pending Approval* (Kepala Gudang/Persediaan).
- **User Profile**: Nama (`name`), Jabatan (`position`), Role Aktif (`role_scope`), dan tombol Logout.

### 1.2 Sidebar Menu (Role-Based Visibility)
Menu disembunyikan/ditampilkan menggunakan `@can` directive dari Spatie Permission.
- 📊 **Dashboard** (Semua Role)
- 📦 **Master Data** (Katalog Material, Bank Data Gudang) -> `@can('master_data.view')`
- 📥 **Inbound (Penerimaan)** -> `@can('transactions.view')`
- 📤 **Outbound (Pengeluaran & Retur)**
- 🔄 **In-Transit (Pemindahan)**
- 🛑 **Quarantine (Penyisihan)**
- 📋 **Stock Opname & Inventarisasi** -> `@can('stock_opname.input_count')`
- 📈 **Reporting & Analytics** -> `@can('reports.view')`
- ⚙️ **Settings** (Manajemen Pengguna & Role Spatie) -> `@can('users.manage')`

---

## 2. Tampilan Dashboard Berdasarkan Peran (Role-Based Views)

### 2.1 Dashboard Management & Accounting (Eksekutif)
**Target Pengguna**: `management`, `accounting`, `super_admin`  
**Fokus**: Pemantauan nilai finansial agregat, tren, dan komposisi saldo material persediaan (MPS) & aset belum tercatat (ABT).

**[Wireframe Visualisasi]**
```text
+-----------------------------------------------------------------------------+
| Filter: [Semua Region ▼]  [Semua Gudang ▼]  [Tahun: 2026 ▼]  [Terapkan] |
+-----------------------------------------------------------------------------+
| [Card: Total Nilai Persediaan] | [Card: Total MPS]    | [Card: Total ABT]   |
|         Rp 45.500.000.000      |   Rp 30.000.000.000  |  Rp 15.500.000.000  |
+-----------------------------------------------------------------------------+
| +-----------------------------------+ +-----------------------------------+ |
| | 🥧 PIE CHART: Komposisi Nilai MPS | | 🥧 PIE CHART: Komposisi Nilai ABT | |
| | (Berdasarkan Kategori & Status    | | (Berdasarkan Kategori & Status    | |
| |  FM, SM, PDS, DS)                 | |  FM, SM, PDS, DS)                 | |
| +-----------------------------------+ +-----------------------------------+ |
+-----------------------------------------------------------------------------+
| +-------------------------------------------------------------------------+ |
| | 📈 LINE CHART: Tren Saldo Akhir Bulanan (MPS vs ABT)                    | |
| | (Sumbu X: Bulan, Sumbu Y: Nilai Rp)                                     | |
| +-------------------------------------------------------------------------+ |
+-----------------------------------------------------------------------------+
| +-------------------------------------------------------------------------+ |
| | 📊 BAR CHART: Perbandingan Saldo Akhir Tahunan (Year-on-Year)           | |
| +-------------------------------------------------------------------------+ |
+-----------------------------------------------------------------------------+
```
> *Interaksi AJAX*: Mengubah dropdown Filter (Gudang/Region) akan mengirimkan AJAX request ke endpoint `/api/dashboard/charts` yang merender ulang chart Chart.js/ApexCharts secara instan berdasarkan SQL View `vw_dashboard_inventory_composition`.

---

### 2.2 Dashboard Kepala Gudang & Fungsi Persediaan (Approval Level)
**Target Pengguna**: `kepala_gudang`, `persediaan`  
**Fokus**: Eksekusi *workflow approval*, verifikasi dokumen, dan pemantauan pergerakan gudang hari ini.

**[Wireframe Visualisasi]**
```text
+-----------------------------------------------------------------------------+
| 🔔 TASK INBOX: PENDING APPROVALS (Membutuhkan Tindakan Anda)              |
+-----------------------------------------------------------------------------+
| (DataTables AJAX - Yajra Server Side)                                       |
| [Search...................]                             [Export Excel/PDF]  |
|                                                                             |
| No Transaksi | Tipe        | Gudang Asal | Status Workflow   | Aksi         |
| -------------|-------------|-------------|-------------------|--------------|
| 1101-MDN-001 | Penerimaan  | Gudang MDN  | Menunggu Ka.Gudang| [Review 🔍] |
| 2202-BGR-005 | Pengeluaran | Gudang BGR  | Verifikasi Harga  | [Review 🔍] |
| 7701-JKT-002 | Opname Fisik| Gudang JKT  | Menunggu Approval | [Review 🔍] |
|                                                                             |
| << Page 1 of 5 >>                                                           |
+-----------------------------------------------------------------------------+
```
> *Interaksi AJAX*:
> 1. Klik tombol **[Review 🔍]** membuka modal AJAX yang berisi detail form, tabel item material, dan link download Dokumen Lampiran (PO/DO/SPK) dari Private S3.
> 2. Modal menampilkan 3 tombol: `[Approve (Stempel Digital)]`, `[Reject]`, `[Minta Revisi]`.
> 3. Klik `Approve` memicu AJAX POST. Jika sukses, notifikasi sukses muncul, dan DataTables di-refresh `table.ajax.reload(null, false)`. Stempel `APPROVED` otomatis digenerate ke PDF.

---

### 2.3 Dashboard Staf Gudang & Tim Inventarisasi (Operator Fisik)
**Target Pengguna**: `staf_gudang`, `tim_inventarisasi`, `holder_material`  
**Fokus**: Entri data cepat, Scan QR, pengisian hasil hitung fisik (Stock Opname).

**[Wireframe Visualisasi]**
```text
+-----------------------------------------------------------------------------+
| QUICK ACTIONS (Pintasan Cepat)                                              |
| [➕ Buat Draft Penerimaan]  [📤 Buat Draft Pengeluaran]  [📷 SCAN QR CODE] |
+-----------------------------------------------------------------------------+
| 📋 CHECKLIST PEKERJAAN HARI INI                                             |
+-----------------------------------------------------------------------------+
| (DataTables AJAX)                                                           |
| Daftar Transaksi Draft yang belum di-submit:                                |
| 1. Draft Pengeluaran Operasional (2 Item) -> [Lanjutkan Edit ✏️]         |
| 2. Draft Sesi Stock Opname (Gudang Blok A) -> [Input Hitung Fisik 🔢]    |
+-----------------------------------------------------------------------------+
| 📦 STATUS STOK KRITIS (Low Stock Alert)                                     |
+-----------------------------------------------------------------------------+
| - Material: Valve Gate 4" (Sisa: 2 EA, Lokasi: RAK-A1)                      |
| - Material: Pipa Seamless 8" (Sisa: 0 MTR, Lokasi: ZONE-B)                  |
+-----------------------------------------------------------------------------+
```
> *Interaksi AJAX*:
> 1. Klik tombol **[📷 SCAN QR CODE]** akan membuka modal kamera via HTML5/JS. Setelah QR di-scan, JSON Token didekode, dan diarahkan langsung ke halaman Rincian Kartu Material atau form penerimaan menggunakan AJAX GET.
> 2. Klik **[Input Hitung Fisik 🔢]** membuka form inline DataTable (Editable DataTable) di mana staf dapat mengetik angka fisik, dan kolom selisih (*variance*) otomatis terhitung secara realtime via JavaScript. Jika ada selisih, field teks `Catatan/Alasan` wajib diisi.

---

## 3. Alur Interaksi Formulir Transaksi (AJAX Form Submit)

Untuk setiap form pembuatan transaksi (Misal: Form Pengeluaran):

1. **Header Form**: Pilih Tipe (Subtipe 2201, 2202, dll), Tujuan, Keterangan, dan Upload Dokumen Pendukung (SPK/Surat OMM).
2. **Item Details (AJAX Grid)**:
   - Tambah Baris (Add Row).
   - Dropdown Material (Select2 AJAX) yang mengambil data dari katalog `materials` dengan pencarian otomatis.
   - Saat material dipilih, AJAX memanggil endpoint untuk menampilkan sisa stok yang *tersedia* (`available_quantity`).
3. **Validasi Frontend**: Jika jumlah request > stok tersedia, input ditandai merah (*disable submit button*).
4. **Submit Form**: Request dikirim via AJAX POST.
   - *Backend Validation* (FormRequest Laravel) memvalidasi ulang (menggunakan *Pessimistic Lock*).
   - Jika berhasil: `SweetAlert` menampilkan "Draft Berhasil Dibuat", halaman di-redirect ke Daftar Transaksi (DataTables).

---
*Dokumen ini merupakan panduan bagi Frontend Developer (Blade, Alpine/jQuery, Bootstrap/Tailwind) untuk mengimplementasikan antarmuka sesuai dengan aturan Spatie Permissions dan arsitektur database Laravel 12.*
