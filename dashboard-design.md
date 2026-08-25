# Rancangan Desain Antarmuka Lengkap (UI/UX)
## Sistem Pengelolaan Persediaan Material dan Gudang

**Versi:** 3.0 (Lengkap Per Menu)
**Tanggal:** 25 Agustus 2026

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
|    └ Audit Log            |                                                  |
+---------------------------+--------------------------------------------------+
```

---

## MENU 1: 📊 DASHBOARD
**Akses:** Semua Role | **Permission:** `dashboard.view_assigned_warehouse` / `dashboard.view_all_warehouses`

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
║  5501-JKT-0003  │ Penyisihan    │ JKT     │ Anton R.  │ ⏳Ka.Gudang │[👁️]  ║
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
║  ⚠️ PERINGATAN STOK KRITIS (Low Stock Alert)                                 ║
║  Material: Valve Gate 4" SS │ Sisa: 2 EA │ Lokasi: RAK-A1  [Lihat 👁️]     ║
║  Material: Pipa Seamless 8" │ Sisa: 0 MTR│ Lokasi: ZONE-B  [Lihat 👁️]     ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

---

## MENU 2: 📦 MASTER DATA
**Akses:** `@can('master_data.view')` | Edit: `@can('master_data.edit')`

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
║ 10001928 ║ VALVE GATE 4" SS 316   ║ 🟦 MPS     ║ Tubular  ║ EA  ║ [✏️][🗑️] ║
║ 10002133 ║ PIPA SEAMLESS 8"       ║ 🟦 MPS     ║ Tubular  ║ MTR ║ [✏️][🗑️] ║
║ 90001123 ║ LAPTOP ASUS ROG G15    ║ 🟩 ABT     ║ Elektron ║ SET ║ [✏️][🗑️] ║
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
║     │ [➕ Tambah Rak/Bin] [Kode: RAK-A1 | Tipe: Rack] [✏️][🗑️]           ║
║     │                    [Kode: ZONE-B  | Tipe: Zone] [✏️][🗑️]           ║
║     │                    [Kode: QRN-01  | Tipe: Quarantine] [✏️][🗑️]     ║
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
║  Kode │ Nama Klasifikasi               │ Keterangan                │ Aksi   ║
║  ─────┼────────────────────────────────┼───────────────────────────┼─────── ║
║  MPS  │ Material Persediaan            │ Persediaan Operasional    │ [✏️]   ║
║  ABT  │ Aset Belum Tercatat            │ Aset Tetap yg blm tercatat│ [✏️]   ║
║  SKL  │ Material Skala                 │ -                         │ [✏️]   ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  TAB: SUBTIPE TRANSAKSI (Dengan Dokumen Wajib)                              ║
║  Kode │ Nama Subtipe                   │ Grup        │ Dok. Wajib  │ Aksi   ║
║  ─────┼────────────────────────────────┼─────────────┼─────────────┼─────── ║
║  1101 │ Penerimaan Pembelian Rutin     │ Inbound     │ PO, DO      │ [✏️]   ║
║  2202 │ Pengeluaran Operasional        │ Outbound    │ SPK         │ [✏️]   ║
║  5501 │ Penyisihan / Write-Off         │ Quarantine  │ Surat GH OMM│ [✏️]   ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

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
║ 1102-BGR-202608-0│ Transfer ║ 24-08-2026║ Susi A.      ║ 🟠 ⏳Gudang  │[👁️] ║
║ 1105-JKT-202608-0│ Retur    ║ 23-08-2026║ Anton R.     ║ ⚪ DRAFT     │[✏️] ║
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
║  Gudang : MDN - Medan                ║  ✅ Cek Fisik Gudang — Agus 09:30    ║
║  Tgl Trx: 25 Agustus 2026           ║  ✅ CHECKED (Stempel) — Staf         ║
║  Status : 🟢 APPROVED               ║  ✅ Approve Ka.Gudang — Hendra 10:15 ║
║                                      ║  ✅ Verifikasi Persediaan — Dewi     ║
║  LAMPIRAN DOKUMEN                    ║  ✅ APPROVED (Stempel) — Final       ║
║  📎 PO-2026-12345.pdf [👁️ Lihat]    ║                                      ║
║  📎 DO-Surat-Jalan.pdf [👁️ Lihat]   ║                                      ║
╠══════════════════════════════════════╩══════════════════════════════════════╣
║  DETAIL ITEM YANG DITERIMA                                                  ║
║  KIMAP    │ Nama Barang          │ Qty Terima │ UOM │ Lokasi  │ Label QR   ║
║  ─────────┼──────────────────────┼────────────┼─────┼─────────┼─────────── ║
║  10001928 │ VALVE GATE 4" SS 316 │ 50         │ EA  │ RAK-A1  │ [🏷️ Print] ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  @can('transactions.check_warehouse') [✅ BUBUHKAN STEMPEL CHECKED] @endcan ║
║  @can('transactions.approve_warehouse_head') [✅ APPROVE] [❌ TOLAK] @endcan ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

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
Form yang sama dengan pengeluaran. Subtipe dipilih sebagai Retur, tambahan field **Alasan Retur** (text area wajib).

---

## MENU 5: 🔄 IN-TRANSIT (PEMINDAHAN ANTAR GUDANG)
Transaksi ini melibatkan **dua gudang**: gudang pengirim dan penerima.
```text
+═════════════════════════════════════════════════════════════════════════════+
║  🔄 FORMULIR PEMINDAHAN MATERIAL (IN-TRANSIT)                [DRAFT]        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Gudang Asal   : [MDN - Gudang Utama Medan (readonly berdasarkan scope)]   ║
║  Gudang Tujuan : [BGR - Gudang Utama Bogor ▼] ← Dropdown Gudang Aktif     ║
║  Tgl Kirim     : [25 Agustus 2026 📅]                                      ║
║  No. Referensi : [MEMO-INTERNAL-001_______________]                         ║
║  Upload Surat  : [📎 Upload Surat Pengantar (PDF)] ← *Opsional/Wajib*      ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  DETAIL ITEM YANG DIPINDAHKAN                                               ║
║  Material        │ Stok di MDN  │ Qty Kirim   │ UOM │ Hapus                ║
║  ────────────────┼──────────────┼─────────────┼─────┼──────────────────    ║
║  Valve Gate 4"   │  50 EA       │ [  10  ]    │ EA  │ [🗑️]                 ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  INFO: Material berstatus IN-TRANSIT sampai diterima di gudang tujuan.     ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

---

## MENU 6: 🛑 QUARANTINE (PENYISIHAN / WRITE-OFF)
Membutuhkan upload **Surat Rekapitulasi Usulan dari Pengelola Material (GH OMM)** sebagai dokumen wajib.

```text
+═════════════════════════════════════════════════════════════════════════════+
║  🛑 FORMULIR PENYISIHAN MATERIAL (QUARANTINE / WRITE-OFF)      [DRAFT]     ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Subtipe    : [5501 - Penyisihan / Write-Off (Quarantine) ▼]               ║
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

---

## MENU 7: 📋 STOCK OPNAME & INVENTARISASI
**Akses:** `@can('stock_opname.create_session')` & `@can('stock_opname.input_count')`

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
║  [Lihat Rekap Scan ...]         │ Qty dihitung dari jumlah scan per label  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [💾 Simpan Hasil Hitung Fisik]      [@can('stock_opname.approve')]        ║
║                                      [🚀 Ajukan Approval Sesi] @endcan     ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

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

### 8.2 — Laporan Saldo Stok Posisi Gudang
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📋 POSISI STOK GUDANG (Per Tanggal)                                        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Filter: [Gudang: MDN ▼] [Per Tgl: 25-08-2026 📅] [Lokasi: RAK-A1 ▼]     ║
║  [📥 Export Excel]  [📄 Export PDF]                                         ║
╠══════════╦════════════════╦═════════╦═══════════╦════════════╦═════════════╣
║  KIMAP   ║ Nama Barang    ║ Lokasi  ║ On Hand   ║ Reserved   ║ Available   ║
╠══════════╬════════════════╬═════════╬═══════════╬════════════╬═════════════╣
║ 10001928 ║ Valve Gate 4"  ║ RAK-A1  ║  115 EA   ║  -10 EA    ║  105 EA    ║
║ 10002133 ║ Pipa SS 8"     ║ ZONE-B  ║   15 MTR  ║    0 MTR   ║   15 MTR   ║
╚══════════╩════════════════╩═════════╩═══════════╩════════════╩═════════════╝
```

### 8.3 — Laporan Rekonsiliasi Penyisihan
Menampilkan ringkasan material yang masuk kategori penyisihan (SM, PDS, DS) beserta nilai finansialnya.

---

## MENU 9: ⚙️ SETTINGS
**Akses:** `@can('users.manage')` untuk tab Pengguna; `@can('roles.manage')` untuk tab Roles.

### 9.1 — Manajemen Pengguna & Hak Akses
```text
+═════════════════════════════════════════════════════════════════════════════+
║  ⚙️  MANAJEMEN PENGGUNA & AKSES                                              ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  [Tab: Pengguna ✔] [Tab: Roles & Permissions] [Tab: Audit Log]              ║
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
║  1. ROLE SPATIE (Hak Fitur): [staf_gudang ▼]                               ║
║                                                                             ║
║  2. SCOPE GUDANG (Hak Data):                                                ║
║     [☑] MDN - Gudang Utama Medan                                            ║
║     [☐] BGR - Gudang Utama Bogor                                            ║
║     [☑] JKT - Gudang Pusat Jakarta                                          ║
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
║  kepala_gudang  │ trx.view, trx.approve_warehouse_head, ...   │ [✏️ Edit]  ║
║  staf_gudang    │ trx.view, trx.check_warehouse, opname...    │ [✏️ Edit]  ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  KLIK [✏️ Edit Role] → MODAL MATRIX PERMISSION:                            ║
║  ┌─────────────────────────────────────────────────────────────────────┐   ║
║  │ Role: kepala_gudang                                                 │   ║
║  │ ─────────────────────────────────────────────────────────────────── │   ║
║  │ Fitur                      │ View │ Create│ Edit │Approve│ Delete   │   ║
║  │ Master Data                │ [☑]  │ [☐]   │ [☐]  │ [☐]   │ [☐]    │   ║
║  │ Transactions               │ [☑]  │ [☑]   │ [☑]  │ [☑]   │ [☐]    │   ║
║  │ Stock Opname               │ [☑]  │ [☑]   │ [☑]  │ [☑]   │ [☐]    │   ║
║  │ Reports                    │ [☑]  │ [☐]   │ [☐]  │ [☐]   │ [☐]    │   ║
║  │                                          [💾 Simpan Permissions]   │   ║
║  └─────────────────────────────────────────────────────────────────────┘   ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

### 9.3 — Audit Log
```text
+═════════════════════════════════════════════════════════════════════════════+
║  📜 AUDIT LOG SISTEM                                                        ║
╠═════════════════════════════════════════════════════════════════════════════╣
║  Filter: [User ▼] [Modul ▼] [Aksi ▼] [Tgl: 25-08-2026 📅]                ║
╠════════════╦══════════╦═══════════════╦════════════════╦════════════════════╣
║ Timestamp  ║ User     ║ Modul/Aksi    ║ Data Lama      ║ Data Baru          ║
╠════════════╬══════════╬═══════════════╬════════════════╬════════════════════╣
║ 10:15:32   ║ Budiman  ║ Trx.Approve   ║ status: draft  ║ status: approved   ║
║ 09:30:11   ║ Saifudin ║ StockOpname   ║ qty_fisik: null║ qty_fisik: 50      ║
║ 08:00:05   ║ System   ║ Trx.Posted    ║ balance: 100   ║ balance: 115       ║
╚════════════╩══════════╩═══════════════╩════════════════╩════════════════════╝
```

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
║  │ [Stempel: APPROVED ⬜]     │║  📎 SPK-2026-876.pdf [👁️ Buka] [📥 Unduh]║
║  │    (akan diisi jika approve)│║                                            ║
║  └────────────────────────────┘║  Catatan Penolakan/Revisi:                 ║
║                                ║  [Textarea: Isi jika Tolak/Revisi...]     ║
╠════════════════════════════════╩════════════════════════════════════════════╣
║  @can('transactions.approve_warehouse_head')                                ║
║  [❌ TOLAK TRANSAKSI]  [🔄 KEMBALIKAN UNTUK REVISI]  [✅ BUBUHKAN APPROVED] ║
║  @endcan                                                                    ║
╚═════════════════════════════════════════════════════════════════════════════╝
```

---
*Dokumen ini merupakan referensi visual komprehensif bagi tim Frontend Developer untuk mengimplementasikan antarmuka berbasis Blade, AJAX, DataTables Yajra, Select2, dan Bootstrap 5 yang dikontrol oleh Spatie Laravel-Permission v7.*
