# 📦 Inventory Management System (Laravel) — Pertemuan 5

Proyek ini adalah implementasi **Pertemuan 5: Laravel Fundamental** yang memindahkan dan merefaktor seluruh fitur dari **Inventory Management System (P4 PHP Native)** ke dalam framework **Laravel** berarsitektur **Model-View-Controller (MVC)**.

---

## 📑 Daftar Isi
1. [Struktur Folder Project](#-struktur-folder-project)
2. [Arsitektur MVC & Alur Request ke Response](#-arsitektur-mvc--alur-request-ke-response)
3. [Daftar Routing & HTTP Methods](#-daftar-routing--http-methods)
4. [Skema Database & Relasi Eloquent](#-skema-database--relasi-eloquent)
5. [Business Rules Stok & Keamanan Transaksi](#-business-rules-stok--keamanan-transaksi)
6. [Demonstrasi Alur Lengkap CRUD](#-demonstrasi-alur-lengkap-crud)
7. [Panduan Instalasi & Menjalankan Aplikasi](#-panduan-instalasi--menjalankan-aplikasi)
8. [Kredensial Akun Pengujian](#-kredensial-akun-pengujian)
9. [Checklist Pemenuhan Modul P5](#-checklist-pemenuhan-modul-p5)

---

## 📁 Struktur Folder Project

Sesuai dengan panduan arsitektur Laravel pada Slide 36:

```text
inventory-laravel/
├── app/
│   ├── Http/
│   │   └── Controllers/
│   │       ├── AuthController.php            # Otentikasi sesi login/logout
│   │       ├── CategoryController.php        # Resource controller master kategori
│   │       ├── DashboardController.php       # Widget agregasi statistik & dashboard
│   │       ├── ProductController.php         # Resource controller utama produk (CRUD + Filter)
│   │       ├── StockMovementController.php   # Transaksi stok aman (IN, OUT, History)
│   │       └── SupplierController.php        # Resource controller supplier
│   └── Models/
│       ├── Category.php                      # Model Kategori (One-to-Many ke Product)
│       ├── Product.php                       # Model Produk (BelongsTo Category, BelongsToMany Supplier)
│       ├── StockMovement.php                 # Model Mutasi Stok (BelongsTo Product, BelongsTo User)
│       ├── Supplier.php                      # Model Supplier (Many-to-Many ke Product)
│       └── User.php                          # Model User Pengguna (Admin & Staff)
├── database/
│   ├── factories/
│   │   ├── CategoryFactory.php               # Factory data dummy kategori
│   │   ├── ProductFactory.php                # Factory data dummy produk (Slide 29)
│   │   ├── SupplierFactory.php               # Factory data dummy supplier
│   │   └── UserFactory.php                   # Factory data user
│   ├── migrations/
│   │   ├── 0001_01_01_000000_create_users_table.php
│   │   ├── 2026_10_11_000001_create_categories_table.php
│   │   ├── 2026_10_11_000002_create_products_table.php       # Slide 26
│   │   ├── 2026_10_11_000003_create_suppliers_table.php
│   │   ├── 2026_10_11_000004_create_product_suppliers_table.php # Junction Many-to-Many
│   │   └── 2026_10_11_000005_create_stock_movements_table.php   # Audit trail mutasi stok
│   └── seeders/
│       └── DatabaseSeeder.php                # Seeder data master P4 + 50 dummy data (Slide 30)
├── resources/
│   └── views/
│       ├── auth/
│       │   └── login.blade.php               # Form autentikasi
│       ├── categories/
│       │   └── index.blade.php               # Halaman CRUD kategori
│       ├── components/
│       │   └── alert.blade.php               # Blade component alert reusable (Slide 23)
│       ├── dashboard.blade.php               # Dashboard agregasi metrik
│       ├── layouts/
│       │   ├── app.blade.php                 # Master Blade layout (Slide 22)
│       │   └── navigation.blade.php          # Navbar menu navigasi responsif
│       ├── products/
│       │   ├── create.blade.php              # Form create produk + error feedback (Slide 25)
│       │   ├── edit.blade.php                # Form edit produk
│       │   ├── index.blade.php               # Listing produk + search/filter/sort (Slide 24, 39)
│       │   └── show.blade.php                # Rincian detail produk
│       ├── stock/
│       │   ├── history.blade.php             # Tabel audit trail mutasi stok
│       │   ├── in.blade.php                  # Form transaksi stok masuk
│       │   └── out.blade.php                 # Form transaksi stok keluar (Anti minus)
│       └── suppliers/
│           └── index.blade.php               # Halaman CRUD supplier
├── routes/
│   └── web.php                               # Definisi seluruh endpoint & Resource Route
├── .env                                      # Konfigurasi environment (Slide 8)
├── run_server.bat                            # Skrip otomatis 1-klik jalan di Windows
└── README.md                                 # Dokumentasi komprehensif proyek
```

---

## 🏛️ Arsitektur MVC & Alur Request ke Response

Aplikasi menerapkan pola kerja MVC secara konsisten sesuai diagram Slide 10:

$$\text{Request} \longrightarrow \text{Route} \longrightarrow \text{Controller} \longrightarrow \text{Model} \longrightarrow \text{Database} \longrightarrow \text{Response}$$

| Layer | Peran & Tanggung Jawab | Implementasi di Proyek |
|---|---|---|
| **Request** | Masukan HTTP dari pengguna (GET/POST/PUT/DELETE) | Query string pencarian, parameter ID, payload input form |
| **Route** | Menghubungkan URL ke Controller yang tepat | [`routes/web.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/routes/web.php) (`Route::resource`, `Route::prefix`) |
| **Controller** | Menerima request, validasi input, pemrosesan transaksi, memilih view/redirect | [`ProductController.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/app/Http/Controllers/ProductController.php), [`StockMovementController.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/app/Http/Controllers/StockMovementController.php) |
| **Model** | Representasi objek data, aturan relasi antar-tabel, logika query Eloquent | [`Product.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/app/Models/Product.php), [`Category.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/app/Models/Category.php), [`Supplier.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/app/Models/Supplier.php) |
| **Database** | Tempat persistensi data relasional | MySQL (`inventory_db`) dikelola via Laravel Migrations |
| **Response** | Menyajikan hasil ke browser pengguna | View HTML hasil render Blade atau redirect dengan session flash |

---

## 🛣️ Daftar Routing & HTTP Methods

Routing dideklarasikan pada [`routes/web.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/routes/web.php) dengan menggunakan **Resource Controller** (`Route::resource`) dan **Named Routes**:

| HTTP Method | URI | Named Route | Controller & Action | Fungsi |
|---|---|---|---|---|
| `GET` | `/` | `dashboard` | `DashboardController@index` | Ringkasan metrik statistik inventaris |
| `GET` | `/products` | `products.index` | `ProductController@index` | Listing produk (Search, Filter, Sorting, Paginasi) |
| `GET` | `/products/create` | `products.create` | `ProductController@create` | Menampilkan form tambah produk baru |
| `POST` | `/products` | `products.store` | `ProductController@store` | Validasi & menyimpan produk baru ke database |
| `GET` | `/products/{product}` | `products.show` | `ProductController@show` | Menampilkan rincian detail 1 produk |
| `GET` | `/products/{product}/edit` | `products.edit` | `ProductController@edit` | Menampilkan form edit produk |
| `PUT/PATCH` | `/products/{product}` | `products.update` | `ProductController@update` | Validasi & memperbarui data produk |
| `DELETE` | `/products/{product}` | `products.destroy` | `ProductController@destroy` | Menghapus produk (Dengan proteksi relasi stok) |
| `GET` | `/stock/history` | `stock.history` | `StockMovementController@history` | Riwayat audit trail transaksi stok |
| `GET` | `/stock/in` | `stock.in` | `StockMovementController@createIn` | Form transaksi stok masuk |
| `POST` | `/stock/in` | `stock.in.store` | `StockMovementController@storeIn` | Menambah stok produk secara aman (`DB::transaction`) |
| `GET` | `/stock/out` | `stock.out` | `StockMovementController@createOut` | Form transaksi stok keluar |
| `POST` | `/stock/out` | `stock.out.store` | `StockMovementController@storeOut` | Mengurangi stok (Perlindungan anti minus) |
| `GET` | `/categories` | `categories.index` | `CategoryController@index` | Master data kategori produk |
| `GET` | `/suppliers` | `suppliers.index` | `SupplierController@index` | Master data supplier |
| `GET` / `POST` | `/login` | `login` / `login.post` | `AuthController@showLoginForm / login` | Form dan proses autentikasi |
| `POST` | `/logout` | `logout` | `AuthController@logout` | Hapus sesi login |

---

## 🗄️ Skema Database & Relasi Eloquent

### 1. Relasi Antar Model:
* **One-to-Many**: `Category` $\rightarrow$ `Product`
  * `Category::hasMany(Product::class)`
  * `Product::belongsTo(Category::class)`
* **Many-to-Many**: `Product` $\leftrightarrow$ `Supplier` via tabel pivot `product_suppliers`
  * `Product::belongsToMany(Supplier::class, 'product_suppliers')->withPivot('supplier_price')`
  * `Supplier::belongsToMany(Product::class, 'product_suppliers')->withPivot('supplier_price')`
* **Audit Trail Transaksi**: `Product` $\rightarrow$ `StockMovement`
  * `Product::hasMany(StockMovement::class)`
  * `StockMovement::belongsTo(Product::class)`
  * `StockMovement::belongsTo(User::class)`

### 2. File Migration yang Dihasilkan:
1. `0001_01_01_000000_create_users_table.php`: Menambahkan field `username` dan `role` (`admin`, `staff`).
2. `2026_10_11_000001_create_categories_table.php`: Tabel kategori dengan constraint unique nama.
3. `2026_10_11_000002_create_products_table.php`: Tabel produk dengan foreign key `category_id` berstatus `restrictOnDelete()`, SKU unik, harga, stok, dan batas minimum stok.
4. `2026_10_11_000003_create_suppliers_table.php`: Tabel supplier kontak dan alamat.
5. `2026_10_11_000004_create_product_suppliers_table.php`: Composite primary key `[product_id, supplier_id]` berstatus `cascadeOnDelete()`.
6. `2026_10_11_000005_create_stock_movements_table.php`: Audit trail pergerakan stok dengan tipe `ENUM('IN', 'OUT')`.

---

## 🛡️ Business Rules Stok & Keamanan Transaksi

1. **Perlindungan Stok Minus (Zero-Stock Minus Protection)**:
   Pada [`StockMovementController@storeOut`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/app/Http/Controllers/StockMovementController.php), transaksi stok keluar menggunakan **Database Transaction** dan **Pessimistic Locking**:
   ```php
   DB::transaction(function () use ($validated) {
       $product = Product::lockForUpdate()->findOrFail($validated['product_id']);
       if ($product->stock < $validated['quantity']) {
           throw new \Exception("Stok tidak mencukupi! Stok saat ini: {$product->stock}, permintaan: {$validated['quantity']}");
       }
       $product->decrement('stock', $validated['quantity']);
       StockMovement::create([...]);
   });
   ```
   Jika jumlah keluar melebihi stok, transaksi secara otomatis **dibatalkan (ROLLBACK)** sehingga nilai stok tidak pernah menjadi negatif.

2. **Integritas Relasional (RESTRICT on Delete)**:
   Produk yang sudah memiliki riwayat mutasi stok di tabel `stock_movements` dilarang dihapus untuk menjaga keabsahan riwayat audit gudang.

3. **Keamanan Query & Sanitasi**:
   - Pencarian kata kunci menggunakan Prepared Statement (`where('name', 'like', "%{$search}%")`).
   - Fitur sorting dilindungi menggunakan **Allowlist** (`name`, `sku`, `price`, `stock`, `created_at`) untuk mencegah SQL Injection via nama kolom.
   - Paginasi mempertahankan state filter URL dengan `withQueryString()`.

---

## 🔄 Demonstrasi Alur Lengkap CRUD

Contoh siklus hidup penambahan produk baru:
1. **Buka Form Tambah**: Pengguna mengklik tombol `+ Tambah Produk` $\rightarrow$ Route `products.create` $\rightarrow$ Controller memuat data kategori & supplier $\rightarrow$ Merender [`products/create.blade.php`](file:///C:/Users/nurul/.gemini/antigravity/scratch/inventory-laravel/resources/views/products/create.blade.php).
2. **Validasi Server-Side**: Jika field kosong atau SKU duplikat, Laravel mengembalikan redirect dengan `$errors` dan nilai input lama (`old()`) ditampilkan kembali disertai pesan merah dari directive `@error`.
3. **Simpan Data**: Input valid diterima di `ProductController@store`, dibungkus dalam `DB::transaction`, produk disimpan, supplier dipetakan, dan mutasi stok awal dicatat.
4. **Redirect**: Controller mengembalikan `redirect()->route('products.index')->with('success', 'Produk berhasil ditambahkan.')`.
5. **Tampilkan Data**: Halaman index menampilkan flash message via Blade Component `<x-alert type="success">`, dan produk baru langsung terlihat di tabel daftar produk.

---

## 🚀 Panduan Instalasi & Menjalankan Aplikasi

### Opsi A: Cara Otomatis 1-Klik (Windows)
1. Buka folder `inventory-laravel`.
2. Klik 2x file **`run_server.bat`**.
3. Browser akan otomatis terbuka ke alamat: **`http://127.0.0.1:8000`**.

### Opsi B: Cara Manual via Terminal / PowerShell
1. Buka PowerShell dan arahkan ke direktori project:
   ```powershell
   cd C:\Users\nurul\.gemini\antigravity\scratch\inventory-laravel
   ```
2. Pastikan file `.env` sudah sesuai (koneksi MySQL `inventory_db`):
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=inventory_db
   DB_USERNAME=root
   DB_PASSWORD=
   ```
3. Jalankan migrasi dan seeder data awal:
   ```powershell
   php artisan migrate:fresh --seed
   ```
4. Jalankan local development server:
   ```powershell
   php artisan serve
   ```
5. Buka peramban di: **`http://127.0.0.1:8000`**

### Menjalankan Automated Test (Pengujian Unit & Feature):
```powershell
php artisan test
```
*Hasil: `6 passed, 30 assertions` (Semua skenario CRUD, relasi Eloquent, dan transaksi stok aman 100% lulus).*

---

## 🔑 Kredensial Akun Pengujian

Database telah dilengkapi dengan akun bawaan dari Pertemuan 4:

| Role | Username | Email | Password | Hak Akses |
|---|---|---|---|---|
| **Admin** | `admin` | `admin@inventory.com` | `admin123` | Akses penuh seluruh master data, produk, dan transaksi stok |
| **Staff** | `staff` | `staff@inventory.com` | `staff123` | Akses katalog produk dan pencatatan transaksi stok masuk / keluar |

---

## ✅ Checklist Pemenuhan Modul P5

Berdasarkan slide 46 (Checklist Pertemuan 5):

- [x] **Project Laravel dapat dijalankan** (`php artisan serve` aktif tanpa kendala).
- [x] **`.env` dan koneksi database benar** (Terhubung ke MySQL `inventory_db` port 3306).
- [x] **`route:list` menampilkan route yang diharapkan** (Seluruh 33 route resource & named terdaftar).
- [x] **Migration dapat dijalankan dari database kosong** (`php artisan migrate:fresh` berhasil membuat seluruh tabel).
- [x] **Seeder menghasilkan data uji** (Menghasilkan master data P4 dan 50 produk dummy via factory).
- [x] **Model memiliki relationship yang benar** (One-to-Many `Category-Product`, Many-to-Many `Product-Supplier`, HasMany `Product-StockMovement`).
- [x] **Resource controller menangani CRUD** (`ProductController` dengan 7 aksi lengkap).
- [x] **Blade memakai layout dan escaping** (Inheritance `@extends('layouts.app')`, directive `@yield`, escaping otomatis `{{ }}`).
- [x] **Form memiliki `@csrf` dan validasi** (Proteksi CSRF aktif dan pesan error `@error` di setiap input).
- [x] **Search dan pagination mempertahankan query string** (Menggunakan `->withQueryString()`).
- [x] **Inventory Management System P4 berhasil dipindahkan** (Seluruh fitur P4 kini beroperasi di arsitektur Laravel MVC).
