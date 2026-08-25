# Diagram Alir Data (Data Flow Diagram - DFD)
## Sistem Pengelolaan Persediaan Material dan Gudang (17 Lokasi Gudang)

**Versi:** 1.0  
**Tanggal:** 24 Agustus 2026  
**Dokumen Acuan:** `rancangan-aplikasi-persediaan-gudang.md`, `database.md`, `Detail.xlsx`  

---

## 1. Pendahuluan

Dokumen ini menyajikan **Diagram Alir Data (Data Flow Diagram - DFD / DAD)** untuk Sistem Pengelolaan Persediaan Material dan Gudang. Diagram ini menggambarkan bagaimana data mengalir dari **Entitas Luar (External Entities)**, diproses melalui **Proses Utama (Processes)**, disimpan pada **Penyimpanan Data (Data Stores)**, hingga menghasilkan **Keluaran/Informasi (Outputs & Reports)**.

---

## 2. Inventarisasi Komponen DFD

### 2.1 Entitas Luar (External Entities)

| Kode Entitas | Nama Entitas | Deskripsi / Peran Sistem |
|---|---|---|
| **E1** | **Pengguna / User** | Mengajukan transaksi permintaan pengeluaran, pemindahan, pengembalian, dan usulan penyisihan. |
| **E2** | **Staf Gudang** | Melakukan pemeriksaan fisik barang, mengisi nomor kartu, lokasi rak/bin, scan QR Code, serta hitung opname. |
| **E3** | **Kepala Gudang** | Menyetujui operasional pengeluaran/penerimaan fisik gudang dan hasil stock opname (`APPROVED`). |
| **E4** | **Fungsi Persediaan** | Memverifikasi harga, nilai finansial, klasifikasi transaksi, serta posting stok ke ledger. |
| **E5** | **Pengelola / Holder (GH OMM)** | Menerbitkan Surat Rekapitulasi Usulan Penyisihan Material sebagai dasar transaksi penyisihan. |
| **E6** | **Tim Inventarisasi** | Melaksanakan sesi inventarisasi fisik berkala/tahunan di gudang. |
| **E7** | **Accounting & Management** | Memantau dashboard agregat, nilai persediaan, serta laporan mutasi dan rekonsiliasi. |
| **E8** | **Super Admin** | Mengelola pengguna, hak akses gudang, dan bank data master. |

---

### 2.2 Penyimpanan Data (Data Stores)

| Kode Store | Nama Data Store | Tabel Database Terkait (`database.md`) |
|---|---|---|
| **D1** | **Master Users & Access Scope** | `users`, `user_warehouse_assignments`, `positions` |
| **D2** | **Master Gudang & Lokasi** | `warehouses`, `warehouse_locations` |
| **D3** | **Master Material & Katalog** | `materials`, `material_classifications`, `material_categories`, `uoms` |
| **D4** | **Saldo Stok & Lot Perolehan** | `stock_balances`, `stock_lots`, `stock_reservations` |
| **D5** | **Immutable Inventory Ledger** | `inventory_movements` (Append-Only Ledger) |
| **D6** | **Transaksi Header & Item** | `inventory_transactions`, `inventory_transaction_items` |
| **D7** | **Dokumen Lampiran Privat** | `transaction_documents` (S3/Private Storage) |
| **D8** | **Approval & Digital Watermark** | `transaction_approvals`, `transaction_histories` |
| **D9** | **Stock Opname & Adjustments** | `stock_count_sessions`, `stock_count_items`, `inventory_adjustments` |
| **D10** | **Spatie RBAC & Permissions** | `roles`, `permissions`, `model_has_roles`, `role_has_permissions` |

---

## 3. DFD Level 0 (Diagram Konteks / Context Diagram)

Diagram Konteks menggambarkan batas luar sistem (*system boundary*) dan interaksi antara entitas eksternal dengan aplikasi utama.

```mermaid
flowchart TD
    E1["E1: Pengguna / User"] -->|"Draft Transaksi, Permintaan Material, Upload Dokumen"| SYS["Sistem Persediaan & Gudang (Laravel 12 Engine)"]
    SYS -->|"PDF Form Transaksi, Status Workflow"| E1
    E2["E2: Staf Gudang"] -->|"Scan QR, Input No. Kartu & Lokasi"| SYS
    SYS -->|"Formulir & Label QR, Stempel CHECKED"| E2
    E3["E3: Kepala Gudang"] -->|"Verifikasi Fisik, Approval Gudang"| SYS
    SYS -->|"Stempel APPROVED, Task Inbox Alert"| E3
    E4["E4: Fungsi Persediaan"] -->|"Verifikasi Nilai/Harga, Posting Ledger"| SYS
    SYS -->|"Rekap Mutasi & Laporan Financial"| E4
    E5["E5: Holder Material (GH OMM)"] -->|"Surat Rekapitulasi Usulan Penyisihan"| SYS
    E6["E6: Tim Inventarisasi"] -->|"Input Hitung Fisik (Metode 1/2)"| SYS
    SYS -->|"Approval Sesi / Laporan Inventarisasi"| E6
    SYS -->|"Dashboard Analytics, Grafik Pie/Line/Bar, Laporan Rekonsiliasi"| E7["E7: Accounting & Management"]
    E8["E8: Super Admin"] -->|"Pengelolaan Bank Data, Pengguna & Scope Gudang"| SYS
    SYS -->|"Audit Log Master"| E8
```

---

## 4. DFD Level 1 (Diagram Utama / Overview Diagram)

DFD Level 1 memecah sistem menjadi **9 Proses Utama**:

```mermaid
flowchart TB
    %% Entitas
    E1["E1: User"]
    E2["E2: Staf Gudang"]
    E3["E3: Kepala Gudang"]
    E4["E4: Persediaan"]
    E5["E5: GH OMM"]
    E6["E6: Tim Inventarisasi"]
    E7["E7: Management"]
    E8["E8: Super Admin"]

    %% Process
    P1["1.0 Manajemen Pengguna & Scope Gudang"]
    P2["2.0 Pengelolaan Master Bank Data"]
    P3["3.0 Transaksi Penerimaan Material (Inbound)"]
    P4["4.0 Transaksi Pengeluaran & Retur (Outbound)"]
    P5["5.0 Pemindahan Antar Gudang (In-Transit)"]
    P6["6.0 Penyisihan Material (Quarantine/Write-Off)"]
    P7["7.0 Stock Opname & Inventarisasi"]
    P8["8.0 Movement Ledger & Balance Projection"]
    P9["9.0 Pelaporan & Dashboard Analytics"]

    %% Data Stores
    D1[("D1: Users & Access")]
    D2[("D2: Warehouses & Locations")]
    D3[("D3: Materials Catalog")]
    D4[("D4: Stock Balances & Lots")]
    D5[("D5: Inventory Movements Ledger")]
    D6[("D6: Transactions Header & Items")]
    D7[("D7: Private Documents")]
    D8[("D8: Approvals & Watermarks")]
    D9[("D9: Stock Count Sessions")]

    %% Connections
    E8 --> P1 --> D1
    E8 --> P2 --> D2 & D3
    D1 & D2 --> P1

    E1 --> P3 --> D6 & D7
    E4 --> P3
    E2 --> P3
    E3 --> P3 --> D8
    P3 --> P8

    E1 --> P4 --> D6 & D7
    E2 --> P4
    E3 --> P4 --> D8
    E4 --> P4
    P4 --> P8

    E1 --> P5 --> D6
    E2 --> P5
    E3 --> P5
    E4 --> P5 --> D8
    P5 --> P8

    E5 --> P6
    E1 --> P6 --> D6 & D7
    E2 & E3 & E4 --> P6 --> D8
    P6 --> P8

    E2 & E6 --> P7 --> D9
    E3 --> P7
    P7 --> D6

    P8 --> D5 & D4

    D4 & D5 & D6 --> P9 --> E7 & E4
```

---

## 5. DFD Level 2 (Rincian Alur Transaksi & Operasional)

### 5.1 DFD Level 2.1: Transaksi Penerimaan Material (Inbound Subtypes 1101 - 1107)

```mermaid
flowchart TD
    E1["E1: User"] -->|"1. Select Subtype (1101-1107), Items, Ref, Upload PO/DO/BAST"| P31["3.1 Registrasi Draft Penerimaan"]
    P31 -->|"Simpan Header & Items"| D6[("D6: Transactions")]
    P31 -->|"Upload File (PO/DO/BAST)"| D7[("D7: Documents")]
    
    P31 -->|"Trigger Workflow Stage 1"| P32["3.2 Verifikasi Fungsi Persediaan"]
    E4["E4: Persediaan"] -->|"Verifikasi Harga & Dokumen -> Approve"| P32
    P32 -->|"Simpan Approval Record (Persediaan)"| D8[("D8: Approvals")]

    P32 -->|"Notifikasi ke Staf Gudang"| P33["3.3 Pemeriksaan Fisik Staf Gudang"]
    E2["E2: Staf Gudang"] -->|"Input No. Kartu, Lokasi Rak/Bin -> Click CHECKED"| P33
    P33 -->|"Simpan Stempel CHECKED"| D8

    P33 -->|"Notifikasi ke Kepala Gudang"| P34["3.4 Persetujuan Kepala Gudang"]
    E3["E3: Kepala Gudang"] -->|"Approve -> Stempel APPROVED"| P34
    P34 -->|"Update Status Approved"| D6
    P34 -->|"Simpan Stempel APPROVED"| D8

    P34 -->|"Trigger Posting Service"| P8["8.0 Inventory Ledger Engine"]
    P8 -->|"Write Append-Only Movement (Inbound)"| D5[("D5: Inventory Movements")]
    P8 -->|"Update Stock Balance & Create Lot"| D4[("D4: Stock Balances & Lots")]

    P8 -->|"Generate Secure QR Token"| P35["3.5 Cetak Label QR Code"]
    E2 -->|"Scan & Cetak Label QR Material"| P35
```

---

### 5.2 DFD Level 2.2: Transaksi Pengeluaran & Pengembalian Material (Outbound Subtypes 2201 - 2206 & Retur)

```mermaid
flowchart TD
    E1["E1: User"] -->|"1. Input Request Item & Available Qty Check"| P41["4.1 Pengajuan Permintaan Pengeluaran"]
    P41 -->|"Check Available Qty"| D4[("D4: Stock Balances")]
    P41 -->|"Create Temporary Reservation"| D4
    P41 -->|"Save Draft Request & Upload Memo/SPK"| D6[("D6: Transactions")] & D7[("D7: Documents")]

    P41 -->|"Notifikasi Staf Gudang"| P42["4.2 Alokasi Lot & Scan QR Staf Gudang"]
    E2["E2: Staf Gudang"] -->|"Scan QR Label, Pilih Lot & Input No. Kartu"| P42
    P42 -->|"Update Item Lot Assignment"| D6

    P42 -->|"Notifikasi Kepala Gudang"| P43["4.3 Approval Keluar Kepala Gudang"]
    E3["E3: Kepala Gudang"] -->|"Approve Fisik Keluar -> Stempel APPROVED"| P43
    P43 -->|"Simpan Approval Record"| D8[("D8: Approvals")]

    P43 -->|"Posting Outbound Quantity"| P8["8.0 Inventory Ledger Engine"]
    P8 -->|"Write Movement Delta (-)"| D5[("D5: Inventory Movements")]
    P8 -->|"Reduce On-Hand & Close Reservation"| D4

    P8 -->|"Verifikasi Nilai Finansial"| P44["4.4 Verifikasi Nilai Persediaan"]
    E4["E4: Persediaan"] -->|"Verify Moving Average Cost & Complete"| P44
    P44 -->|"Set Status Completed"| D6
```

---

### 5.3 DFD Level 2.3: Pemindahan Material Antar Gudang (7-Step Transfer Flow)

```mermaid
flowchart TD
    E1["User Gudang Asal"] -->|"1. Submit Transfer Form (Source -> Dest WH)"| P51["5.1 Form Pemindahan Gudang Asal"]
    P51 -->|"Reservation Stock Gudang Asal"| D4[("D4: Stock Balances")]

    P51 --> P52["5.2 Scan & Check Staf Gudang Asal"]
    E2_A["Staf Gudang Asal"] -->|"Scan QR & Confirmation Qty"| P52

    P52 --> P53["5.3 Approval Kepala Gudang Asal"]
    E3_A["Kepala Gudang Asal"] -->|"Approve Keberangkatan (Status: In-Transit)"| P53
    P53 -->|"Ledger Move: On-Hand -> In-Transit"| D5[("D5: Movements")] & D4

    P53 --> P54["5.4 Konfirmasi Penerima"]
    E1_B["Penerima Material"] -->|"Konfirmasi Penerimaan Administratif"| P54

    P54 --> P55["5.5 Verifikasi Persediaan Asal"]
    E4_A["Persediaan Asal"] -->|"Verify Price & Value"| P55

    P55 --> P56["5.6 Pemeriksaan Fisik Staf Gudang Tujuan"]
    E2_B["Staf Gudang Tujuan"] -->|"Pemeriksaan Fisik & Click CHECKED"| P56

    P56 --> P57["5.7 Approval Final Kepala Gudang Tujuan"]
    E3_B["Kepala Gudang Tujuan"] -->|"Approve Arrival (Status: Completed)"| P57
    P57 -->|"Ledger Move: In-Transit -> On-Hand Tujuan"| D5 & D4
```

---

### 5.4 DFD Level 2.4: Penyisihan Material (Quarantine & Write-Off)

```mermaid
flowchart TD
    E5["E5: Holder Material (GH OMM)"] -->|"1. Terbitkan Surat Usulan Penyisihan"| E1["E1: User"]
    E1 -->|"2. Create Request & Upload Surat GH OMM"| P61["6.1 Pengajuan Penyisihan"]
    P61 -->|"Save Transaction & Document"| D6[("D6: Transactions")] & D7[("D7: Documents")]
    P61 -->|"Move Stock to Quarantine Reservation"| D4[("D4: Stock Balances")]

    P61 --> P62["6.2 Check Staf Gudang"]
    E2["E2: Staf Gudang"] -->|"Pemeriksaan Fisik & Stempel CHECKED"| P62

    P62 --> P63["6.3 Approval Kepala Gudang"]
    E3["E3: Kepala Gudang"] -->|"Approve Fisik -> Stempel APPROVED"| P63

    P63 --> P64["6.4 Verifikasi & Posting Final Persediaan"]
    E4["E4: Persediaan"] -->|"Verify Value & Final Approval"| P64
    P64 -->|"Write Ledger: Quarantine -> Write-Off"| D5[("D5: Inventory Movements")]
    P64 -->|"Update Balance Status to Written-Off"| D4
```

---

### 5.5 DFD Level 2.5: Stock Opname & Inventarisasi (Dua Metode Hitung)

```mermaid
flowchart TD
    E2["E2: Staf Gudang / E6: Tim Inventarisasi"] -->|"1. Buat Sesi Opname (Gudang, Periode, Scope)"| P71["7.1 Inisialisasi Sesi Opname"]
    P71 -->|"Freeze Stock Snapshot"| D4[("D4: Stock Balances")]
    P71 -->|"Save Session"| D9[("D9: Stock Count Sessions")]

    P71 --> P72["7.2 Pelaksanaan Perhitungan Fisik"]
    
    P72 -->|"Pilihan Metode 1"| M1["Metode 1: Manual Count Input"]
    P72 -->|"Pilihan Metode 2"| M2["Metode 2: Individual QR Scan Count"]

    M1 & M2 -->|"Hitung Variance = Physical - Snapshot"| P73["7.3 Kalkulasi Selisih & Entry Keterangan"]
    P73 -->|"Jika Variance != 0 -> Input Mandatory Explanation"| D9

    P73 --> P74["7.4 Persetujuan Hasil Opname"]
    E2 -->|"Click CHECKED"| P74
    E3["E3: Kepala Gudang"] -->|"Approve -> Stempel APPROVED"| P74

    P74 -->|"Jika Terdapat Selisih"| P75["7.5 Generate Draft Inventory Adjustment"]
    P75 -->|"Buat Transaksi Adjustment Terpisah"| D6[("D6: Transactions")]
```

---

## 6. Kamus Data (Data Dictionary)

### 6.1 Aliran Data Input Utama

1. **Data Pengajuan Transaksi (User Request Payload)**:
   - `transaction_type`: Enum (`receipt`, `issue`, `return`, `transfer`, `write_off`).
   - `transaction_subtype_id`: Foreign Key ke `transaction_subtypes` (misal: `1101`, `2202`, `3300`).
   - `source_warehouse_id` & `destination_warehouse_id`: ID Gudang.
   - `reference_text`: Nomor PO / Surat Jalan / Memo / Surat GH OMM.
   - `attached_documents`: Array File (MIME type whitelist, max 10MB, SHA256 checksum).
   - `items`: Array (`material_id`, `requested_quantity`, `uom_id`, `notes`).

2. **Data Hasil Perhitungan Opname (Stock Count Payload)**:
   - `session_id`: ID Sesi Stock Opname.
   - `count_method`: Enum (`manual`, `qr_scan`).
   - `physical_quantity`: Decimal precision UOM.
   - `variance_quantity`: Calculated (`physical_quantity - system_quantity`).
   - `explanation`: Text wajib diisi (*mandatory freetext*) jika `variance_quantity != 0`.
   - `scanned_label_tokens`: Array JSON Token QR Code (jika menggunakan `qr_scan`).

3. **Data Stempel Approval (Digital Approval Stamp Payload)**:
   - `stage_code`: Kode Tahapan Workflow (`stage_warehouse_check`, `stage_head_approve`, dll).
   - `decision`: Enum (`approved`, `rejected`, `revision_requested`).
   - `watermark_stamp`: Enum (`CHECKED`, `APPROVED`).
   - `notes`: Catatan persetujuan / alasan reject.
   - `audit_snapshot`: Snapshot (`actor_name`, `position`, `role`, `timestamp`, `ip_address`).

---

### 6.2 Aliran Data Output Utama

1. **PDF Formulir Transaksi Resmi**:
   - Header Nomor Transaksi `{SUBTYPE}-{WH}-{YYYYMM}-{SEQ}`.
   - Tabel Rincian Material, KIMAP, UOM, Kuantitas, Harga Satuan, Kuantitas Total.
   - Dynamic Stempel Digital (`CHECKED` dari Staf Gudang, `APPROVED` dari Kepala Gudang & Persediaan).
   - Embedded QR Code Token transaksi.

---

## 7. Pemetaan DFD ke Arsitektur Teknis Laravel 12

Untuk memastikan DFD ini dapat diimplementasikan secara langsung oleh developer backend/fullstack, setiap komponen DFD dipetakan secara eksplisit ke dalam **Struktur Modul & Class Laravel 12**:

### 7.1 Pemetaan Proses Utama DFD ke Class Laravel

| ID DFD | Nama Proses DFD | Controller / Livewire | Class Action / Service Domain | Eloquent Models & Traits Terlibat |
|---|---|---|---|---|
| **1.0** | Manajemen Pengguna & Scope Gudang | `UserController`, `RoleController` | `AssignUserWarehouseScopeAction`, `RoleAndPermissionSeeder` | `User` (`HasRoles`), `Role`, `Permission`, `UserWarehouseAssignment` |
| **2.0** | Pengelolaan Master Bank Data | `MaterialController`, `WarehouseController` | `ImportMaterialMasterAction` | `Material`, `MaterialClassification`, `Uom` |
| **3.0** | Transaksi Penerimaan (Inbound) | `ReceiptTransactionController` | `CreateReceiptTransactionAction`, `GenerateQrTokenAction` | `InventoryTransaction`, `TransactionDocument`, `StockLot` |
| **4.0** | Pengeluaran & Retur (Outbound) | `IssueTransactionController` | `CreateIssueTransactionAction`, `StockReservationService` | `InventoryTransaction`, `StockBalance`, `StockReservation` |
| **5.0** | Pemindahan Antar Gudang | `TransferTransactionController` | `ProcessInTransitTransferAction` | `InventoryTransaction`, `InventoryMovement` |
| **6.0** | Penyisihan Material | `WriteOffTransactionController` | `QuarantineMaterialAction`, `PostWriteOffAction` | `InventoryTransaction`, `StockBalance` |
| **7.0** | Stock Opname & Inventarisasi | `StockCountController` | `CountPhysicalStockAction` (Support Metode 1 & 2) | `StockCountSession`, `StockCountItem`, `InventoryAdjustment` |
| **8.0** | Movement Ledger Engine | Internal Event Listener | `InventoryPostingEngine` (`DB::transaction` + `lockForUpdate`) | `InventoryMovement`, `StockBalance` |
| **9.0** | Reporting & Analytics | `ReportController`, `DashboardController` | `GenerateExportReportJob` (Redis Queue), `DashboardAnalyticsQuery` | `vw_dashboard_inventory_composition`, `InventoryMovement` |

---

### 7.2 Pemetaan Transport & Layanan Infrastruktur Laravel

1. **Spatie Laravel-Permission (v7) & Security Scoping**:
   - **Spatie `HasRoles` Trait**: Model `User` menggunakan `use Spatie\Permission\Traits\HasRoles;` untuk mengelola hak akses granular (`givePermissionTo()`, `assignRole()`, `hasPermissionTo()`).
   - **Spatie RBAC Middleware**: Proteksi route pada `routes/web.php` menggunakan middleware Spatie v7 (misal: `middleware(['auth', 'permission:transactions.approve_warehouse_head'])`).
   - **Blade Authorization**: Pengkondisian tombol dan form approval menggunakan `@can('transactions.approve_warehouse_head') ... @endcan`.
   - **Warehouse Scoping (`WarehousePolicy`)**: Memastikan kueri Eloquent secara otomatis ter-scope dengan `$query->whereIn('source_warehouse_id', $userWarehouseIds)` berdasarkan penugasan user di `user_warehouse_assignments`.

2. **Database Transaction & Concurrency Control**:
   - **Pessimistic Locking**: `StockBalance::where(...)->lockForUpdate()->first()` digunakan saat posting movement atau reservasi stok untuk mencegah *double posting* dan *overselling*.
   - **Optimistic Concurrency**: Kolom `lock_version` pada `inventory_transactions` mencegah *lost update* saat dua approver membuka formulir bersamaan.

3. **Background Processing & Event Outbox**:
   - **Queue Jobs**: Ekspor laporan XLSX/PDF berukuran besar diproses secara asynchronous menggunakan `Laravel Queue Worker (Redis)`.
   - **Events & Listeners**: Event `TransactionPosted` mentrigger notifikasi email/in-app dan memperbarui proyeksi agregat saldo secara instan.

4. **Storage & Watermark Rendering**:
   - **S3 / Local Private Storage**: Seluruh lampiran file wajib (`PO`, `DO`, `SPK`, `SURAT_GH_OMM`) disimpan privat dengan akses unduhan menggunakan `Storage::disk('s3_private')->temporaryUrl()`.
   - **Dynamic Watermark PDF Generator**: Menggunakan dompdf/snappy untuk membuat PDF resmi bertanda stempel `CHECKED` atau `APPROVED` dari snapshot metadata `transaction_approvals`.

---
*Diagram Alir Data (DFD) ini disusun secara presisi dan 100% konsisten dengan `rancangan-aplikasi-persediaan-gudang.md`, `database.md`, `Spatie Laravel-Permission v7`, dan `Detail.xlsx`.*
