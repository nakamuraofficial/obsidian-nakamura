## Rekap Struktur Koding — Djampi Jawi Laravel

### Tech Stack
- **Backend:** PHP 8.3, Laravel 12
- **Database:** MySQL
- **Frontend:** Blade Templates, Vite, jQuery DataTables, SweetAlert2
- **Paket tambahan:** Laravel Excel (Maatwebsite), Laravel Sanctum, Filament v3, Livewire v3, DomPDF, Blade Icons

---

### Arsitektur Aplikasi

Project ini menjalankan **dua aplikasi web dalam satu codebase**:

| Aplikasi | Prefix Route | Pengguna | Guard Auth |
|---|---|---|---|
| `web-admin` | `/` | Administrator | `auth:web-app` |
| `web-app` | `/web-app` | Staff operasional | `auth:web-app` |

---

### Struktur Direktori Utama

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Admin/              ← Controller web-admin
│   │   ├── WebApp/             ← Controller web-app
│   │   ├── API/                ← API publik
│   │   ├── APICustomer/        ← API untuk customer
│   │   ├── APIInternal/        ← API internal (bridge CodeIgniter)
│   │   ├── APINarasi/          ← API untuk fitur narasi AI
│   │   └── APIPOS/             ← API Point of Sale
│   ├── Middleware/             ← 5 middleware kustom
│   └── Requests/Admin/         ← Form Request validasi
├── Models/                     ← Eloquent models
├── Exports/Admin/              ← Export Excel per modul
├── Imports/Admin/              ← Import Excel (Stock, Master Data)
├── Jobs/Narasi/                ← Queue jobs (AI narasi)
├── Providers/                  ← AppServiceProvider, BladeServiceProvider
└── Utilities/
    ├── helpers.php             ← Helper global (format angka, terbilang, dll)
    └── Traits/                 ← FonnteChat, JsonResponse
```

---

### Modul Admin (Admin)

| Folder | Controller | Fungsi |
|---|---|---|
| `Branch/` | BranchAdminController, DivisionAdminController, LantaiAdminController | Manajemen cabang & divisi |
| `Finance/` | KasAdminController, InvoiceAdminController, PendapatanAdminController, SetoranAdminController, AdminEDCAdminController | Keuangan & kas |
| `Karyawan/` | PartTimeAdminController, UserShiftAdminController | Karyawan & shift |
| `ManageProduksi/` | ProduksiAdminController, DistribusiAdminController | Produksi jamu & distribusi |
| `ManageStock/` | OrderAdminController, PembelianStockAdminController, OlahanStockAdminController, AssetStockAdminController, PenyusutanAdminController | Manajemen stok |
| `Marketing/` | CustomerAdminController, ReservasiAdminController, TopSellerAdminController | Marketing & reservasi |
| `MasterData/` | Ingredient, OlahanMaster, ProductCategory, PaymentMethod, Printer, Unit, Variation, dll | Data master |
| `Product/` | ProductAdminController, PricingProductAdminController | Produk & harga |
| `Promo/` | — | Promo |
| `Transaction/` | TransactionAdminController, TransactionRekapAdminController | Transaksi & rekap |
| `User/` | UserAdminController, GroupAdminController, MenuAdminController | Pengguna & akses |

---

### Modul Web App (WebApp)

| Controller | Fungsi |
|---|---|
| `AuthWebAppController` | Login/logout staff |
| `AdminWebAppController` | Halaman utama staff |
| `OrderWebAppController` | Input order |
| `DropOrderWebAppController` | Pembatalan order |
| `ProduksiJamuWebAppController` | Proses produksi jamu |
| `StockWebAppController` | Pengecekan stok |
| `PembelianWebAppController` | Pencatatan pembelian |
| `AccountWebAppController` | Profil akun |

---

### API Endpoints (apis)

| File Route | Fungsi |
|---|---|
| `api-internal.php` | Bridge ke sistem CodeIgniter lama (absensi, stok, transaksi, dll.) |
| `api-pos.php` | Autentikasi Point of Sale |
| `api-narasi.php` | Generate narasi/deskripsi dengan AI |
| `api-customer.php` | API untuk aplikasi customer |
| `api-github.php` | Integrasi GitHub |

---

### Models — Pengelompokan Domain

| Namespace/Folder  | Model                                                                                              |
| ----------------- | -------------------------------------------------------------------------------------------------- |
| Root              | `User`, `Product`, `Order`, `Transaction`, `Ingredient`, `Olahan`, `Reservasi`, `Pembayaran`, dll. |
| `Branch/`         | `Branch`, `Division`, `Lantai`                                                                     |
| `Finance/`        | `KasAkun`, `KasSaldo`, `Setoran`, `Pendapatan`, `Pengeluaran`, `AdminEDC`, dll.                    |
| `Karyawan/`       | `PartTime`, `UserShift`                                                                            |
| `ManageProduksi/` | `Produksi`, `ProduksiOlahan`, `Distribusi`, `DistribusiDetail`                                     |
| `ManageStock/`    | `Asset`, `Pembelian`, `DropOrder`, `Loss`                                                          |
| `Marketing/`      | `TopSeller`                                                                                        |
| `MasterData/`     | `PaymentMethod`, `Bank`                                                                            |
| `Navigation/`     | `Menu`                                                                                             |

---

### Middleware

| File | Fungsi |
|---|---|
| `AuthorizeWebApp.php` | Guard utama + bridge sesi CodeIgniter |
| `AuthorizeJWTToken.php` | Validasi JWT untuk API |
| `ValidateAPIRequest.php` | Validasi request API internal |
| `ValidateAPINarasiRequest.php` | Validasi request narasi AI |
| `AppendJsonResult.php` | Mengubah response menjadi JSON |

---

### View & Frontend (views)

```
views/
├── web-admin/
│   ├── layouts/        ← Layout utama admin
│   ├── partials/       ← Komponen partial (sidebar, header, dll.)
│   └── pages/
│       ├── branches/, dashboard, finance/, karyawan/
│       ├── manage-produksi/, manage-stock/, marketing/
│       ├── master-data/, products/, promo/
│       ├── transactions/, users/
└── web-app/
    ├── layouts/
    └── partials/
```

---

### View Composition (BladeServiceProvider)

Semua view otomatis mendapat variabel berikut tanpa perlu dioper manual dari controller:

- `$currentUser` — User aktif beserta cabang & grup
- `$currentMenu` — Menu aktif (web-admin)
- `$currentBreadcrumb` — Breadcrumb navigasi
- `$_menus` — Daftar menu berdasarkan grup user
- `$_branch_id` — Filter cabang dari session

---

### Queue & Integrasi Eksternal

| Komponen | Keterangan |
|---|---|
| `Jobs/Narasi/ProcessNarasi` | Job async generate narasi AI |
| **Fonnte** | Notifikasi WhatsApp |
| **Narasi AI** | Generate deskripsi produk |
| **CodeIgniter** | Sistem lama yang diintegrasikan via API internal & bridge sesi |

---

### Pola Pengembangan Umum

1. **CRUD baru** → Controller + Route group explicit + View Blade + method `source()` untuk DataTables
2. **Validasi** → Form Request di Admin
3. **Export** → Class di Admin menggunakan Laravel Excel
4. **Primary key** menggunakan pola `{table}_id` (bukan `id`)
5. **Soft delete** menggunakan flag `{table}_is_active` (bukan `deleted_at`)
6. **Timestamp** menggunakan nama kustom `{table}_create_at` / `{table}_update_at`