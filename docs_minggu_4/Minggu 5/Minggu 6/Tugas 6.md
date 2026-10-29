# Laporan Tugas Minggu 6 — Endpoint Collection yang Dapat Dicari

**Mata Kuliah / Modul:** Pemrograman Berbasis Web / Backend API  
**Topik:** Pagination, Search, Sorting Allowlist, dan Query Validation pada Resource Event Bencana Dunia  
**Tanggal:** 29 Oktober 2026  

---

## 1. Tujuan

Tujuan pelaksanaan Tugas 6 ini adalah:
1. Menerapkan **pagination** pada endpoint collection `GET /api/events` dengan parameter `page` dan `per_page` beserta batas maksimum.
2. Mengimplementasikan **search** yang mencocokkan minimal dua field relevan (`title` dan `category`) menggunakan pencarian parsial (`LIKE %...%`).
3. Membatasi **sorting** hanya pada field yang termasuk dalam **allowlist** untuk mencegah akses ke kolom sensitif atau kolom yang tidak ada.
4. Mengimplementasikan **query validation** yang menolak parameter invalid dengan status `422 Unprocessable Content`.
5. Mengembalikan collection melalui **API Resource** (`EventResource`) beserta **metadata pagination** yang dapat diverifikasi.
6. Mempertahankan **query parameter pada pagination links** (`withQueryString()`).
7. Mendokumentasikan seluruh pengujian di **Postman collection** yang dapat dijalankan ulang.

---

## 2. Endpoint dan Query Contract

### Base URL

```
http://127.0.0.1:8000/api
```

### Endpoint

| Method | Endpoint | Deskripsi |
|:-------|:---------|:----------|
| `GET` | `/events` | Collection endpoint — menampilkan daftar event bencana dengan pagination, search, dan sorting |

### Query Parameters

| Parameter | Tipe | Rules | Default | Keterangan |
|:----------|:-----|:------|:--------|:-----------|
| `page` | `integer` | `sometimes`, `integer`, `min:1` | `1` | Nomor halaman aktif |
| `per_page` | `integer` | `sometimes`, `integer`, `min:1`, `max:50` | `5` | Jumlah item per halaman (maks. 50) |
| `search` | `string` | `sometimes`, `nullable`, `string`, `max:100` | `null` | Kata kunci pencarian parsial (`LIKE %...%`) |
| `sort` | `string` | `sometimes`, `Rule::in([...allowlist])` | `id` | Kolom pengurutan (hanya dari allowlist) |
| `direction` | `string` | `sometimes`, `Rule::in(['asc', 'desc'])` | `asc` | Arah pengurutan |

---

## 3. Daftar Field Search dan Sort Allowlist

### Search Fields
Pencarian (`search`) mencocokkan keyword pada **empat field** menggunakan `LIKE %keyword%`:

| No | Field | Alasan |
|:---|:------|:-------|
| 1 | `title` | Nama/judul event bencana — field yang paling sering dicari pengguna |
| 2 | `category` | Kategori bencana (Wildfires, Earthquakes, Floods, dll.) — memungkinkan filter berdasarkan jenis |
| 3 | `country` | Negara lokasi bencana — memungkinkan pencarian berdasarkan wilayah |
| 4 | `external_id` | Identifier eksternal (EONET_DEV_001, dll.) — untuk lookup cepat berdasarkan ID sumber |

### Sort Allowlist
Hanya field berikut yang diizinkan sebagai nilai parameter `sort`:

| No | Field | Alasan |
|:---|:------|:-------|
| 1 | `id` | Primary key — default sorting |
| 2 | `title` | Pengurutan alfabetis berdasarkan nama event |
| 3 | `category` | Pengurutan berdasarkan jenis bencana |
| 4 | `country` | Pengurutan berdasarkan negara |
| 5 | `status` | Pengurutan berdasarkan status (Aktif/Selesai) |
| 6 | `event_date` | Pengurutan kronologis berdasarkan tanggal kejadian |
| 7 | `created_at` | Pengurutan berdasarkan waktu data ditambahkan |
| 8 | `external_id` | Pengurutan berdasarkan identifier sumber data |

Field di luar daftar ini (misalnya `password`, `updated_at`, atau kolom fiktif) akan ditolak dengan error `422`.

---

## 4. Validation Rules

Implementasi validation pada [`EventController::index()`](file:///d:/Heard/bencanadunia/app/Http/Controllers/EventController.php):

```php
$validated = $request->validate([
    'search'    => ['sometimes', 'nullable', 'string', 'max:100'],
    'sort'      => ['sometimes', Rule::in([
        'id', 'title', 'category', 'country',
        'status', 'event_date', 'created_at', 'external_id'
    ])],
    'direction' => ['sometimes', Rule::in(['asc', 'desc'])],
    'page'      => ['sometimes', 'integer', 'min:1'],
    'per_page'  => ['sometimes', 'integer', 'min:1', 'max:50'],
]);
```

**Perilaku validasi:**
- Parameter yang tidak dikirim (`sometimes`) dianggap valid dan menggunakan nilai default.
- `sort` yang tidak ada di allowlist → `422` dengan pesan *"The selected sort is invalid."*
- `direction` bukan `asc`/`desc` → `422` dengan pesan *"The selected direction is invalid."*
- `per_page` > 50 → `422` dengan pesan *"The per page field must not be greater than 50."*
- `page` < 1 → `422` dengan pesan *"The page field must be at least 1."*

---

## 5. Bukti Pagination dan Metadata

### 5.1 Default Pagination (tanpa parameter)

- **Request:** `GET /api/events`
- **Status:** `200 OK`
- **Expected:** `per_page=5`, `page=1`, menampilkan 5 item pertama
- **Actual Response:**
```json
{
    "data": [
        {
            "id": "EONET_DEV_001",
            "title": "Kebakaran Hutan",
            "category": "Wildfires",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-01T08:00:00.000000Z",
            "latitude": -8.095,
            "longitude": 113.15,
            "source": "NASA EONET"
        },
        {
            "id": "EONET_DEV_002",
            "title": "Badai Tropis Noru",
            "category": "Severe Storms",
            "country": "Filipina",
            "status": "Aktif",
            "date": "2026-10-02T10:30:00.000000Z",
            "latitude": 14.5995,
            "longitude": 120.9842,
            "source": "NASA EONET"
        },
        {
            "id": "EONET_DEV_003",
            "title": "Gempa Bumi Magnitudo 7.1",
            "category": "Earthquakes",
            "country": "Jepang",
            "status": "Selesai",
            "date": "2026-09-15T03:45:00.000000Z",
            "latitude": 35.6762,
            "longitude": 139.6503,
            "source": "USGS"
        },
        {
            "id": "EONET_DEV_004",
            "title": "Banjir Bandang Jakarta",
            "category": "Floods",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-10T06:00:00.000000Z",
            "latitude": -6.2088,
            "longitude": 106.8456,
            "source": "BMKG"
        },
        {
            "id": "EONET_DEV_005",
            "title": "Erupsi Gunung Merapi",
            "category": "Volcanoes",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-05T15:20:00.000000Z",
            "latitude": -7.5407,
            "longitude": 110.4457,
            "source": "PVMBG"
        }
    ],
    "links": {
        "first": "http://127.0.0.1:8000/api/events?page=1",
        "last": "http://127.0.0.1:8000/api/events?page=3",
        "prev": null,
        "next": "http://127.0.0.1:8000/api/events?page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 3,
        "links": [
            {"url": null, "label": "&laquo; Previous", "active": false},
            {"url": "http://127.0.0.1:8000/api/events?page=1", "label": "1", "active": true},
            {"url": "http://127.0.0.1:8000/api/events?page=2", "label": "2", "active": false},
            {"url": "http://127.0.0.1:8000/api/events?page=3", "label": "3", "active": false},
            {"url": "http://127.0.0.1:8000/api/events?page=2", "label": "Next &raquo;", "active": false}
        ],
        "path": "http://127.0.0.1:8000/api/events",
        "per_page": 5,
        "to": 5,
        "total": 12
    }
}
```

**Verifikasi metadata:**
- `meta.current_page` = 1 ✅
- `meta.per_page` = 5 (default) ✅
- `meta.total` = 12 (sesuai jumlah seed data) ✅
- `meta.last_page` = 3 (12 ÷ 5 = 2.4 → 3 halaman) ✅
- `meta.from` = 1, `meta.to` = 5 ✅
- `links.next` mengarah ke page 2, `links.prev` = null ✅

### 5.2 Custom Page dan Per Page

- **Request:** `GET /api/events?per_page=3&page=2`
- **Status:** `200 OK`
- **Actual Response (metadata):**
```json
{
    "data": ["... 3 item (EONET_DEV_004, EONET_DEV_005, EONET_DEV_006) ..."],
    "links": {
        "first": "http://127.0.0.1:8000/api/events?per_page=3&page=1",
        "last": "http://127.0.0.1:8000/api/events?per_page=3&page=4",
        "prev": "http://127.0.0.1:8000/api/events?per_page=3&page=1",
        "next": "http://127.0.0.1:8000/api/events?per_page=3&page=3"
    },
    "meta": {
        "current_page": 2,
        "from": 4,
        "last_page": 4,
        "per_page": 3,
        "to": 6,
        "total": 12
    }
}
```

**Verifikasi metadata:**
- `meta.current_page` = 2, `meta.per_page` = 3 ✅
- `meta.last_page` = 4 (12 ÷ 3 = 4 halaman) ✅
- `meta.from` = 4, `meta.to` = 6 (item ke-4 s/d ke-6) ✅
- Query string `per_page=3` dipertahankan pada semua pagination links ✅
- `links.prev` mengarah ke page 1, `links.next` ke page 3 ✅

---

## 6. Bukti Search pada Dua Field

### 6.1 Search Field Pertama — `title`

- **Request:** `GET /api/events?search=Kebakaran`
- **Status:** `200 OK`
- **Actual Response:**
```json
{
    "data": [
        {
            "id": "EONET_DEV_001",
            "title": "Kebakaran Hutan",
            "category": "Wildfires",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-01T08:00:00.000000Z",
            "latitude": -8.095,
            "longitude": 113.15,
            "source": "NASA EONET"
        },
        {
            "id": "EONET_DEV_007",
            "title": "Kebakaran Hutan California",
            "category": "Wildfires",
            "country": "Amerika Serikat",
            "status": "Aktif",
            "date": "2026-10-08T09:00:00.000000Z",
            "latitude": 34.0522,
            "longitude": -118.2437,
            "source": "NASA EONET"
        }
    ],
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "per_page": 5,
        "to": 2,
        "total": 2
    }
}
```

**Verifikasi:** Keyword "Kebakaran" mencocokkan **title** → 2 event ditemukan, keduanya memiliki "Kebakaran" pada judul. ✅

### 6.2 Search Field Kedua — `category`

- **Request:** `GET /api/events?search=Earthquakes`
- **Status:** `200 OK`
- **Actual Response:**
```json
{
    "data": [
        {
            "id": "EONET_DEV_003",
            "title": "Gempa Bumi Magnitudo 7.1",
            "category": "Earthquakes",
            "country": "Jepang",
            "status": "Selesai",
            "date": "2026-09-15T03:45:00.000000Z",
            "latitude": 35.6762,
            "longitude": 139.6503,
            "source": "USGS"
        },
        {
            "id": "EONET_DEV_006",
            "title": "Tsunami Warning Pasifik",
            "category": "Earthquakes",
            "country": "Chile",
            "status": "Selesai",
            "date": "2026-09-20T12:00:00.000000Z",
            "latitude": -33.4489,
            "longitude": -70.6693,
            "source": "USGS"
        },
        {
            "id": "EONET_DEV_011",
            "title": "Gempa Bumi Sulawesi",
            "category": "Earthquakes",
            "country": "Indonesia",
            "status": "Selesai",
            "date": "2026-09-28T22:10:00.000000Z",
            "latitude": -1.4748,
            "longitude": 121.4381,
            "source": "BMKG"
        }
    ],
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "per_page": 5,
        "to": 3,
        "total": 3
    }
}
```

**Verifikasi:** Keyword "Earthquakes" mencocokkan **category** → 3 event ditemukan. Perhatikan bahwa "Tsunami Warning Pasifik" tidak mengandung "Earthquakes" di title, tetapi cocok pada category. Ini membuktikan search berfungsi pada field `category`. ✅

---

## 7. Bukti Sorting

### 7.1 Sorting Ascending (`title asc`)

- **Request:** `GET /api/events?sort=title&direction=asc&per_page=12`
- **Status:** `200 OK`
- **Urutan title yang dikembalikan:**
  1. "Badai Salju Eropa"
  2. "Badai Tropis Noru"
  3. "Banjir Bandang Jakarta"
  4. "Banjir Monsun India"
  5. "Erupsi Gunung Merapi"
  6. "Gempa Bumi Magnitudo 7.1"
  7. "Gempa Bumi Sulawesi"
  8. "Kebakaran Hutan"
  9. "Kebakaran Hutan California"
  10. "Kekeringan Australia"
  11. "Longsor Sumatera Barat"
  12. "Tsunami Warning Pasifik"

**Verifikasi:** Urutan title mengikuti urutan alfabet A–Z. ✅

### 7.2 Sorting Descending (`title desc`)

- **Request:** `GET /api/events?sort=title&direction=desc&per_page=12`
- **Status:** `200 OK`
- **Urutan title yang dikembalikan:**
  1. "Tsunami Warning Pasifik"
  2. "Longsor Sumatera Barat"
  3. "Kekeringan Australia"
  4. "Kebakaran Hutan California"
  5. "Kebakaran Hutan"
  6. "Gempa Bumi Sulawesi"
  7. "Gempa Bumi Magnitudo 7.1"
  8. "Erupsi Gunung Merapi"
  9. "Banjir Monsun India"
  10. "Banjir Bandang Jakarta"
  11. "Badai Tropis Noru"
  12. "Badai Salju Eropa"

**Verifikasi:** Urutan title terbalik dari ascending — Z–A. ✅

---

## 8. Bukti Error 422

### 8.1 Invalid Sort Field

- **Request:** `GET /api/events?sort=password`
- **Status:** `422 Unprocessable Content`
- **Actual Response:**
```json
{
    "message": "The selected sort is invalid.",
    "errors": {
        "sort": [
            "The selected sort is invalid."
        ]
    }
}
```

**Verifikasi:** Field `password` bukan bagian dari sort allowlist → ditolak dengan 422. ✅

### 8.2 Invalid Per Page (Melebihi Batas Maksimum)

- **Request:** `GET /api/events?per_page=100`
- **Status:** `422 Unprocessable Content`
- **Actual Response:**
```json
{
    "message": "The per page field must not be greater than 50.",
    "errors": {
        "per_page": [
            "The per page field must not be greater than 50."
        ]
    }
}
```

**Verifikasi:** `per_page=100` melebihi batas `max:50` → ditolak dengan 422. ✅

### 8.3 Invalid Direction

- **Request:** `GET /api/events?sort=title&direction=random`
- **Status:** `422 Unprocessable Content`
- **Actual Response:**
```json
{
    "message": "The selected direction is invalid.",
    "errors": {
        "direction": [
            "The selected direction is invalid."
        ]
    }
}
```

**Verifikasi:** `direction=random` bukan `asc`/`desc` → ditolak dengan 422. ✅

---

## 9. Perbandingan Actual Response dengan API Contract

### Format Response Collection (API Contract)

```json
{
    "data": [
        {
            "id": "<external_id>",
            "title": "<string>",
            "category": "<string>",
            "country": "<string|null>",
            "status": "<string|null>",
            "date": "<ISO-8601|null>",
            "latitude": "<float|null>",
            "longitude": "<float|null>",
            "source": "<string>"
        }
    ],
    "links": {
        "first": "<url>",
        "last": "<url>",
        "prev": "<url|null>",
        "next": "<url|null>"
    },
    "meta": {
        "current_page": "<integer>",
        "from": "<integer|null>",
        "last_page": "<integer>",
        "per_page": "<integer>",
        "to": "<integer|null>",
        "total": "<integer>"
    }
}
```

### Hasil Perbandingan

| Aspek | API Contract | Actual Response | Status |
|:------|:-------------|:----------------|:-------|
| Status code sukses | `200 OK` | `200 OK` | ✅ Sesuai |
| Wrapping `data` | Array of objects via API Resource | Array of `EventResource` objects | ✅ Sesuai |
| Field `id` | `external_id` dipetakan ke `id` | `"id": "EONET_DEV_001"` | ✅ Sesuai |
| Field `date` | ISO-8601 format | `"date": "2026-10-01T08:00:00.000000Z"` | ✅ Sesuai |
| Tipe numerik | `latitude`/`longitude` sebagai float | `"latitude": -8.095` | ✅ Sesuai |
| Objek `links` | `first`, `last`, `prev`, `next` | Tersedia dengan URL lengkap | ✅ Sesuai |
| Objek `meta` | `current_page`, `from`, `last_page`, `per_page`, `to`, `total` | Semua field tersedia | ✅ Sesuai |
| Query string pada links | Dipertahankan via `withQueryString()` | `per_page=3&page=2` tersedia di links | ✅ Sesuai |
| Error validation | `422` dengan `message` dan `errors` | Dikembalikan sesuai format Laravel | ✅ Sesuai |
| Field tidak diekspos | Kolom internal (`updated_at`, auto-increment `id`) tidak muncul | Tidak ada di response | ✅ Sesuai |

---

## 10. Implementasi Teknis

### Alur Query di Controller

```
Request masuk
    ↓
Validasi query parameters (search, sort, direction, page, per_page)
    ↓ (gagal → 422)
Build query: Event::query()
    ↓
Jika search ada → WHERE title LIKE %...% OR category LIKE %...% OR country LIKE %...% OR external_id LIKE %...%
    ↓
ORDER BY $sort $direction
    ↓
paginate($perPage) → withQueryString()
    ↓
Return EventResource::collection($events)
```

### File yang Dimodifikasi

| File | Perubahan |
|:-----|:----------|
| [`EventController.php`](file:///d:/Heard/bencanadunia/app/Http/Controllers/EventController.php) | Method `index()` — validasi, search, sort, pagination |
| [`EventResource.php`](file:///d:/Heard/bencanadunia/app/Http/Resources/EventResource.php) | Transformasi field sesuai API contract |
| [`EventSeeder.php`](file:///d:/Heard/bencanadunia/database/seeders/EventSeeder.php) | 12 record seed data untuk demonstrasi pagination |
| [`postman_collection.json`](file:///d:/Heard/bencanadunia/docs/postman_collection.json) | 11 request Minggu 6 untuk semua skenario pengujian |

---

## 11. Kesimpulan

1. **Pagination** berfungsi dengan benar — endpoint mendukung `page` dan `per_page` dengan default `per_page=5` dan batas maksimum 50. Metadata pagination (`current_page`, `from`, `to`, `last_page`, `per_page`, `total`) tersedia dan dapat diverifikasi.

2. **Search** beroperasi pada minimal dua field (`title` dan `category`), ditambah `country` dan `external_id`. Pencarian parsial menggunakan `LIKE %...%` memungkinkan pencocokan substring. Search yang tidak menemukan hasil tetap mengembalikan `200 OK` dengan `data: []`.

3. **Sorting** dibatasi oleh allowlist yang hanya mengizinkan 8 kolom yang aman. Kolom di luar allowlist ditolak dengan `422` sebelum query database dieksekusi, mencegah SQL injection dan information disclosure.

4. **Query validation** menghasilkan `422 Unprocessable Content` untuk setiap parameter invalid (`sort` di luar allowlist, `direction` bukan asc/desc, `per_page` melebihi batas, `page` di bawah 1).

5. **API contract dan actual response konsisten** — semua field, tipe data, status code, struktur wrapper `data`/`links`/`meta`, serta format error sesuai kontrak.

6. **Pagination links mempertahankan query parameter** — menggunakan `withQueryString()` sehingga navigasi antar halaman tidak kehilangan konteks search/sort.

7. **Postman collection** mencakup 11 skenario pengujian Minggu 6 yang dapat dijalankan ulang.

---

## 12. Referensi

1. Laravel Documentation: [Pagination](https://laravel.com/docs/pagination)
2. Laravel Documentation: [Eloquent: API Resources](https://laravel.com/docs/eloquent-resources)
3. Laravel Documentation: [Validation — Available Rules](https://laravel.com/docs/validation#available-validation-rules)
4. Laravel Documentation: [Validation — Rule::in](https://laravel.com/docs/validation#rule-in)
5. Laravel Documentation: [Database: Query Builder — Where Clauses](https://laravel.com/docs/queries#where-clauses)
6. Postman Documentation: [Creating Collections](https://learning.postman.com/docs/collections/collections-overview/)
7. Materi 6 — Relasi, Pagination, Filtering, dan Sorting (Modul perkuliahan)
8. Praktikum 6 — Endpoint Collection yang Dapat Dicari (Panduan praktikum)

---

## 13. Deklarasi Penggunaan AI

- **Alat yang Digunakan:** Antigravity AI Coding Assistant (Claude Opus 4.6 Thinking).
- **Tujuan Penggunaan:**
  1. Mengembangkan data seeder dari 2 record menjadi 12 record untuk demonstrasi pagination yang bermakna.
  2. Menyusun Postman collection lengkap dengan 11 skenario pengujian Minggu 6.
  3. Menyusun dokumentasi laporan `docs/minggu-6.md` sesuai format deliverable tugas.
- **Prompt / Bantuan yang Diminta:**
  Mengimplementasikan ketentuan "Tugas 6 — Endpoint Collection yang Dapat Dicari" pada project Bencana Dunia, meliputi: pagination, search pada dua field, sorting allowlist, query validation 422, metadata pagination, Postman collection, dan dokumentasi.
- **Bagian yang Dipakai / Diubah:**
  - File [`database/seeders/EventSeeder.php`](file:///d:/Heard/bencanadunia/database/seeders/EventSeeder.php) — data seed diperluas menjadi 12 event beragam.
  - File [`docs/postman_collection.json`](file:///d:/Heard/bencanadunia/docs/postman_collection.json) — folder Minggu 6 diperbarui dengan 11 request bernomor M6-01 s/d M6-11.
  - File [`docs/minggu-6.md`](file:///d:/Heard/bencanadunia/docs/minggu-6.md) — dokumentasi lengkap dibuat sesuai template deliverable.
  - Logika controller `index()` pada [`EventController.php`](file:///d:/Heard/bencanadunia/app/Http/Controllers/EventController.php) sudah ada sebelumnya dan **tidak diubah** oleh AI pada sesi ini.
- **Cara Memeriksa Hasil:**
  1. Jalankan `php artisan db:seed --class=EventSeeder` untuk mengisi 12 data.
  2. Jalankan `php artisan serve` dan uji setiap request dari Postman collection.
  3. Verifikasi status code, struktur response, metadata pagination, dan error 422 pada setiap skenario.
  4. Bandingkan actual response dengan API contract yang didokumentasikan di bagian 9.

**Pernyataan:** Seluruh kode yang dihasilkan dan disesuaikan telah diperiksa, diuji, dan dipahami fungsinya secara menyeluruh oleh pengembang. Mahasiswa bertanggung jawab penuh atas penjelasan kode dan response saat verifikasi individual.
