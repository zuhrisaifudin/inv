# 📦 DIGIO Inventory

**Sistem Pengelolaan Persediaan Material & Gudang — 17 Lokasi Gudang**

<!--
  Lencana di bawah menggambarkan *target stack* berdasarkan rancangan.
  Status proyek: baseline rancangan (Fase 0).
-->
[![Status Proyek](https://img.shields.io/badge/status-Design%20%20Baseline-8E44AD?style=for-the-badge)](#)
[![Laravel](https://img.shields.io/badge/Laravel-12-FF2D20?logo=laravel&logoColor=white&style=for-the-badge)](https://laravel.com)
[![PHP](https://img.shields.io/badge/PHP-8.2%2B-777BB4?logo=php&logoColor=white&style=for-the-badge)](https://www.php.net)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white&style=for-the-badge)](https://www.postgresql.org)
[![MySQL](https://img.shields.io/badge/MySQL-8.0%2B-4479A1?logo=mysql&logoColor=white&style=for-the-badge)](https://www.mysql.com)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis&logoColor=white&style=for-the-badge)](https://redis.io)

> **DIGIO Inventory** adalah sistem web internal untuk menstandarkan seluruh siklus hidup material di **17 lokasi gudang**: penerimaan (inbound), *putaway* ke zona/rak/bin, pengeluaran (outbound) dengan picking **FIFO/FEFO**, pemindahan antar gudang, pengembalian, penyisihan, *stock opname*, identifikasi **QR/Barcode**, pelaporan, hingga dashboard manajemen real-time.
>
> Setiap perubahan kuantitas atau nilai hanya terjadi melalui **immutable inventory ledger** (*append-only*), dengan dua buku stok yang dapat diaudit — **Buku Gudang** dan **Buku Persediaan**.

> [!NOTE]
> **Status repository saat ini:** repository berisi **baseline rancangan** (Fase 0) yang menjadi sumber kebenaran tunggal bagi tim developer. Kode aplikasi akan ditambahkan pada fase implementasi sesuai struktur pada [Struktur Direktori](#4-struktur-direktori-proyek). Bagian [Cara Instalasi & Menjalankan](#5-cara-instalasi--menjalankan-proyek-getting-started) menjelaskan prasyarat dan setup lingkungan target.

## Daftar Isi

1. [Gambaran Proyek & Status](#1-gambaran-proyek--status)
2. [Fitur Utama](#2-fitur-utama)
3. [Teknologi yang Digunakan (Tech Stack)](#3-teknologi-yang-digunakan-tech-stack)
4. [Struktur Direktori Proyek](#4-struktur-direktori-proyek)
5. [Cara Instalasi & Menjalankan Proyek (Getting Started)](#5-cara-instalasi--menjalankan-proyek-getting-started)
6. [Dokumentasi Terkait](#6-dokumentasi-terkait)
7. [Kontribusi & Aturan Dokumen](#7-kontribusi--aturan-dokumen)

---

## 1. Gambaran Proyek & Status

DIGIO Inventory memisahkan dua buku stok sesuai alur resmi perusahaan:

| Buku | Nama Laporan | Diposting Saat | PIC |
|---|---|---|---|
| **Buku Gudang** | Laporan Stok Gudang | Approval Kepala Gudang | Operasional gudang |
| **Buku Persediaan** | Laporan Persediaan Material | Approval Fungsi Persediaan | Pengendalian nilai & klasifikasi |

Seluruh saldo tampilan merupakan proyeksi yang wajib dapat direkonsiliasi ke *ledger*, sehingga setiap mutasi selalu dapat ditelusuri ke nomor transaksi, item, lot, kartu stok, gudang, pengguna, dan keputusan approval.

### Status Proyek

| Fase | Cakupan | Status |
|---|---|:---:|
| **Fase 0** — Analisis & Rancangan | Business process (PDF), mapping data (Excel), rancangan aplikasi, rancangan database, DFD | ✅ Selesai — *baseline* terdokumentasi di repository ini |
| **Fase 1** — Implementasi Inti | IAM/RBAC, bank data, inbound & outbound (FIFO/FEFO), ledger & saldo, QR/Barcode | 🔜 Segera |
| **Fase 2** — Operasional Lanjutan | Transfer & in-transit, opname & inventarisasi, notifikasi real-time, laporan & ekspor, dashboard | 📅 Direncanakan |

### Prinsip Arsitektur Inti

1. **Immutable Inventory Ledger** — riwayat mutasi stok hanya bersifat *append-only*; tidak pernah di-*update* atau di-*delete*.
2. **Quick-Read Balance Projection** — tabel saldo real-time sebagai cache yang di-update dalam transaksi database dengan *row locking*.
3. **Warehouse-Scoped Access** — pengguna hanya melihat gudang yang ditugaskan (*RBAC + warehouse policy scope*).
4. **Concurrency Safety** — *optimistic locking* (`lock_version`) dan *stock reservation* untuk mencegah *overselling* / *double allocation*.

---

## 2. Fitur Utama

### 🏭 Manajemen Stok & Multi-lokasi Gudang

- Pengelolaan **17 lokasi gudang** dengan *zoning* tiga tingkat: **zona → rak → bin**.
- Kartu stok (*stock card*) per kombinasi gudang + KIMAP, berisi saldo berjalan dari *ledger*.
- Posisi stok *real-time*: *on-hand*, *reserved*, *available*, *in-transit*, *quarantined*.
- Akses data gudang otomatis ter-*scope* berdasarkan penugasan pengguna.

### 🔄 Alur Inbound & Outbound (FIFO/FEFO)

- **Inbound** — penerimaan material (kode transaksi 1101, 1103, 1104, 1107) + *putaway* ke lokasi.
- **Outbound** — pengeluaran (2201–2206) dengan alur **picking → packing → surat jalan**; picking mengikuti **FIFO** (periode perolehan tertua) atau **FEFO** (kedaluwarsa terdekat) kecuali ada override beralasan.
- **Return** — pengembalian material (1102, 1105, 1106).
- **Transfer antar gudang** — mutasi keluar 2204 / masuk 1104 dengan status **in-transit** dan monitor SLA.
- **Penyisihan** — usulan penghapusan + penyisihan (3300), produktif → non-produktif.

### 🔔 Sistem Notifikasi (Low Stock & Expired Alert)

- **Low stock** — dievaluasi job setiap 15 menit dan saat *posting* (`available ≤ min_stock`).
- **Expired / expiring** — scheduler harian 00:05 WIB + saat inbound untuk lot berkedaluwarsa (`≤ today` / `≤ today + 30 hari`).
- **Alert operasional lain** — *transfer overdue* (in-transit > SLA), selisih *opname*, QR belum tercetak, dan seluruh event workflow approval (*submitted / revision / rejected / approved*).
- Kanal: **in-app (wajib)** + email opsional, via Redis queue & broadcast (Laravel Echo, fallback polling 15 detik). Deduplikasi alert: satu alert terbuka per (tipe, gudang, material, lot).

### 📊 Laporan & Ekspor Data (Excel/PDF)

- **12 laporan standar** (RPT-01 s.d. RPT-12): Laporan Persediaan Material, Laporan Stok Gudang, Mutasi, Rekapitulasi Penerimaan/Pengeluaran/Penyisihan, Stock Card, Stock Opname, Inventarisasi, In-Transit, Non-Produktif, dan Low Stock & Expired.
- **Ekspor XLSX** untuk analitik dan **PDF** untuk formulir resmi (penerimaan, pengeluaran, surat jalan, dsb.) lengkap dengan **stempel digital** `CHECKED` / `APPROVED` / `QR Passed` yang diambil dari snapshot approval (tidak dapat dipalsukan dari klien).
- Ekspor **asinkron via queue** untuk data > 10.000 baris, dengan notifikasi saat siap dan unduhan ber-*expiry* yang diaudit.

### 🔐 Role-Based Access Control (Admin, Staff Gudang, Viewer, dst.)

Otorisasi dua tingkat: **peran & permission** (Spatie Laravel-Permission v7) membatasi aksi, dan **warehouse policy scope** membatasi data gudang.

| Kategori | Peran (kode) | Tanggung Jawab Utama |
|---|---|---|
| **Admin** | `super_admin`, `admin_user` | Bank data, konfigurasi sistem, manajemen akun & penugasan |
| **Operasional Gudang** | `staf_gudang`, `kepala_gudang` | Pemeriksaan fisik, nomor kartu, *putaway*, scan QR, picking; approval operasional & posting Buku Gudang |
| **Fungsi & Manajemen** | `persediaan`, `pejabat_user`, `holder_material`, `accounting`, `tim_inventarisasi` | Reviu nilai/klasifikasi, posting Buku Persediaan, usulan penghapusan, inventarisasi fisik |
| **Viewer** | `management` | Dashboard dan laporan agregat (**read-only**) |

Dilengkapi **MFA** wajib untuk Super Admin & Accounting, *audit trail* append-only (`audit_logs`), dan rate limiting pada login, scan, ekspor, dan approval.

### Fitur Pendukung Lain

- 🏷️ **Generator QR & Barcode** — QR ber-payload token UUID + Code-128 (nomor kartu), cetak label batch, cetak ulang ber-audit, label *void* tidak dapat dipindai.
- 📷 **Scan via web/PWA + kamera** — untuk picking dan stock opname tanpa aplikasi native.
- 📈 **Dashboard real-time** — KPI komposisi nilai, transaksi, tingkat persediaan, aging approval, dan alert, dengan filter global yang ter-*scope* gudang.
- 🧮 **Stock opname & inventarisasi** — sesi hitung fisik, selisih, penjelasan, dan adjustment ter-approval.
- 📋 **Rencana Kebutuhan Material (RKM)** — pemilih item dari posisi stok, validasi NCI, generate usulan PR/CI.

---

## 3. Teknologi yang Digunakan (Tech Stack)

Target stack berikut ditetapkan dalam [`rancangan-aplikasi-persediaan-gudang.md`](rancangan-aplikasi-persediaan-gudang.md) dan [`database.md`](database.md).

| Lapisan | Teknologi | Keterangan |
|---|---|---|
| **Backend** | **Laravel 12** (Modular Monolith), PHP 8.2+ (8.3 disarankan) | Arsitektur modular per domain: IAM, Inbound, Outbound, Count, Reporting, dsb. |
| **Frontend** | **Blade + Livewire**, DataTables server-side, web/PWA | Server-side rendering; scan QR memakai kamera browser (tanpa aplikasi native) |
| **Database** | **PostgreSQL 16** *atau* **MySQL 8.0+** | ERD & skema migration lengkap di [`database.md`](database.md); indexing untuk performa KPI |
| **Cache & Queue** | **Redis 7** | Cache katalog master & dashboard, antrean notifikasi/ekspor, broadcast event |
| **Real-time** | Laravel Echo + Redis Broadcast (fallback polling 15 detik) | Notifikasi in-app tanpa memblokir transaksi |
| **Autentikasi** | Laravel Fortify / Sanctum | Login, reset password, session regenerate, MFA |
| **Otorisasi** | Spatie Laravel-Permission v7 + Policies | Role, permission granular, dan warehouse scope per query |
| **Dokumen & Ekspor** | Ekspor XLSX (analitik) & PDF (formulir resmi) | Ekspor > 10.000 baris diproses asinkron via queue |
| **Identifikasi** | QR Code (payload token UUID) + Barcode Code-128 | Cetak label, scan picking/opname |
| **Build & Dev Tools** | Node.js (LTS), Composer 2, Vite | Build aset frontend & manajemen dependensi |
| **CI/CD** | Lint, test, dependency scan, build, deploy staging, smoke | Lini minimum sesuai kebutuhan non-fungsional |

**Kebutuhan non-fungsional kunci:** ketersediaan 99,5% pada jam operasional · P95 transaksi interaktif < 2 detik · konkurensi aman terhadap *double posting* & *overselling* · bahasa Indonesia, timezone `Asia/Jakarta`, format tanggal `DD/MM/YYYY`.

---

## 4. Struktur Direktori Proyek

### Struktur Saat Ini (Fase Rancangan)

```text
inv/
├── README.md                     # ← Dokumen ini
├── rancangan-aplikasi-persediaan-gudang.md   # 📘 Master design (v2.0) — sumber kebenaran tunggal
├── database.md                   # 🗄️  Rancangan database: ERD, migration, aturan integritas
├── dfd.md                        # 🔄   Data Flow Diagram (Level 0, 1, 2)
├── Detail.xlsx                   # 📊   Sumber data bisnis asli (tab: DETAIL, MODUL, PROSES, Sheet1)
├── Proses Aplikasi Inventory.pdf # 📄   Dokumen alur bisnis operasional asli
├── Dokumen Asli/
│   └── Kerangka Pengembangan Digio Inventory.xlsx  # Kerangka pengembangan proyek
└── .vscode/
    └── settings.json
```

### Struktur Rencana (Fase Implementasi — Laravel 12 Modular Monolith)

```text
inv/
├── app/
│   ├── Domain/               # Logika bisnis per modul (IAM, Inbound, Outbound, Count, Reporting…)
│   ├── Http/Controllers/     # Endpoint API & web
│   ├── Livewire/             # Komponen interaktif (DataTables, form bertahap, inbox approval)
│   ├── Models/               # Model Eloquent (users, warehouses, materials, stock_lots, ledger…)
│   ├── Policies/             # Otorisasi per aksi + warehouse scope
│   └── Jobs/                 # Job queue: notifikasi, ekspor, rekonsiliasi
├── bootstrap/
├── config/
├── database/
│   ├── migrations/           # Skema persis mengikuti database.md (ledger append-only, dll.)
│   └── seeders/              # Master data: 17 gudang, zona/rak/bin, klasifikasi, UOM, role
├── public/
├── resources/
│   ├── css/
│   ├── js/
│   └── views/                # Template Blade (dashboard, transaksi, scan QR, laporan)
├── routes/                   # web.php, api.php, console.php (scheduler alert)
├── storage/                  # Lampiran transaksi (bucket privat, signed URL)
├── tests/
├── composer.json
├── package.json
└── .env.example
```

---

## 5. Cara Instalasi & Menjalankan Proyek (Getting Started)

> [!IMPORTANT]
> Repository saat ini berada pada **Fase 0 (baseline rancangan)** — kode aplikasi akan tersedia pada fase implementasi. Langkah di bawah adalah **setup lingkungan target** sesuai tech stack yang ditetapkan rancangan, sehingga siap dijalankan begitu kode ditambahkan.

### 5.1 Prasyarat Sistem (Requirements)

| Komponen | Versi Minimum | Keterangan |
|---|---|---|
| PHP | 8.2+ (8.3 disarankan) | Ekstensi: `pdo_pgsql` / `pdo_mysql`, `redis`, `mbstring`, `openssl`, `zip` |
| Composer | 2.x | Manajemen dependensi PHP |
| PostgreSQL **atau** MySQL | 16 / 8.0+ | Pilih salah satu sebagai *primary engine* |
| Redis | 7+ | Cache, queue, dan broadcast |
| Node.js | LTS (20+) | Build aset frontend (Vite) |
| Git | 2.x | Version control |

Opsi praktis: **Docker + Docker Compose** untuk menyediakan PostgreSQL 16 & Redis 7 secara lokal.

### 5.2 Langkah Instalasi

```bash
# 1. Clone repository
git clone https://github.com/zuhrisaifudin/inv.git
cd inv

# 2. Pasang dependensi PHP & frontend
composer install
npm install

# 3. Siapkan konfigurasi lingkungan
cp .env.example .env
php artisan key:generate
```

### 5.3 Konfigurasi `.env`

Edit file `.env` sesuai lingkungan database Anda:

```dotenv
# ===== Aplikasi =====
APP_NAME="DIGIO Inventory"
APP_ENV=local
APP_DEBUG=true
APP_TIMEZONE=Asia/Jakarta
APP_URL=http://localhost:8000

# ===== Database (PostgreSQL 16 — contoh) =====
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=digio_inventory
DB_USERNAME=postgres
DB_PASSWORD=secret

# ===== Database (MySQL 8.0+ — alternatif) =====
# DB_CONNECTION=mysql
# DB_PORT=3306

# ===== Redis (cache, queue, broadcast) =====
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

### 5.4 Migrasi Database

```bash
# Jalankan seluruh migration (skema sesuai database.md) dan seed master data
php artisan migrate --seed
```

> Migration mencakup tabel RBAC (Spatie Permission v7), 17 gudang & lokasi, material & klasifikasi, saldo stok & lot, transaksi & item, **ledger `inventory_movements` (append-only)**, approval, opname, dan `audit_logs`.

### 5.5 Menjalankan Server Lokal

```bash
# Terminal 1 — server web
php artisan serve                 # → http://localhost:8000

# Terminal 2 — watch aset frontend (opsional, untuk development)
npm run dev

# Terminal 3 — worker queue (notifikasi, ekspor, rekonsiliasi)
php artisan queue:work

# Terminal 4 — scheduler (evalusi expired harian 00:05 WIB, dsb.)
php artisan schedule:work
```

Lihat **http://localhost:8000** di browser (Chrome/Edge dua versi terbaru).

---

## 6. Dokumentasi Terkait

Seluruh keputusan desain teregistrasi dalam dokumen berikut — **baca sebelum berkontribusi**:

| Dokumen | Isi | Status |
|---|---|:---:|
| [rancangan-aplikasi-persediaan-gudang.md](rancangan-aplikasi-persediaan-gudang.md) | **Master design v2.0**: tujuan & lingkup, workflow, struktur database, spesifikasi modul, kebutuhan non-fungsional, matriks dual posting, Definition of Done | ✅ Baseline |
| [database.md](database.md) | Arsitektur database, ERD (Mermaid), skema migration Laravel per tabel, penggunaan Spatie Permission di middleware & Blade | ✅ Baseline |
| [dfd.md](dfd.md) | Data Flow Diagram Level 0/1/2: entitas luar, data store, alur proses, dan pemetaan ke controller/service/model | ✅ v1.0 |
| [Detail.xlsx](Detail.xlsx) | Sumber data bisnis asli (tab `DETAIL`, `MODUL`, `PROSES`, `Sheet1`) — acuan mapping kolom | 📄 Sumber |
| [Proses Aplikasi Inventory.pdf](Proses%20Aplikasi%20Inventory.pdf) | Dokumen alur bisnis operasional perusahaan (sumber utama workflow) | 📄 Sumber |
| [Dokumen Asli/Kerangka Pengembangan Digio Inventory.xlsx](Dokumen%20Asli/Kerangka%20Pengembangan%20Digio%20Inventory.xlsx) | Kerangka pengembangan proyek | 📄 Sumber |

### Referensi Cepat di Master Design

| Topik | Lokasi |
|---|---|
| Alur proses bisnis (inbound, outbound FIFO/FEFO, return, transfer, opname) | §2 — [Alur Proses Bisnis (Workflow)](rancangan-aplikasi-persediaan-gudang.md#2-alur-proses-bisnis-workflow) |
| Struktur database & mapping kolom bisnis → tabel | §3 — [Struktur Database & Mapping Data](rancangan-aplikasi-persediaan-gudang.md#3-struktur-database--mapping-data-sesuai-detaillxlsx) |
| Spesifikasi modul & fitur (dashboard, QR, laporan) | §4 — [Spesifikasi Modul & Fitur Aplikasi](rancangan-aplikasi-persediaan-gudang.md#4-spesifikasi-modul--fitur-aplikasi) |
| Kebutuhan non-fungsional (keamanan, performa, audit trail) | §5 — [Kebutuhan Non-Fungsional](rancangan-aplikasi-persediaan-gudang.md#5-kebutuhan-non-fungsional) |
| Matriks akses modul per role (RBAC) | §1.6.2 — di dalam [rancangan](rancangan-aplikasi-persediaan-gudang.md) |
| Mesin posting & aturan integritas untuk developer | §3.6–3.7 & [database.md](database.md) |

---

## 7. Kontribusi & Aturan Dokumen

- **Sumber kebenaran:** `rancangan-aplikasi-persediaan-gudang.md` (v2.0). Setiap perubahan desain wajib mengubah dokumen tersebut **sebelum** diimplementasikan, sesuai prinsip *baseline-first* yang dicanangkan rancangan.
- **Keputusan desain:** perbedaan aturan antar sumber bisnis (PDF vs Excel) telah direkonsiliasi dan dicatat sebagai *keputusan desain mengikat* di §1.8 rancangan — jangan mengubahnya tanpa meninjau keputusan tersebut.
- **Konvensi penulisan dokumen:** Bahasa Indonesia, timezone `Asia/Jakarta`, format tanggal `DD/MM/YYYY`.
- **Definition of Done implementasi:** lihat Lampiran C rancangan sebelum membuka pull request fitur.

---

<p align="center">
  <sub>Built with <strong>Laravel 12</strong> · <strong>PostgreSQL 16</strong> / <strong>MySQL 8</strong> · <strong>Redis 7</strong></sub><br>
  <sub>DIGIO Inventory — Sistem Pengelolaan Persediaan Material & Gudang</sub>
</p>
