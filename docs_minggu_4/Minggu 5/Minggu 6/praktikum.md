# Praktikum Minggu 6 — Search, Sorting, dan Pagination

## Query Contract
- **search**: `string`, maksimal 100 karakter (`sometimes|nullable|string|max:100`). Mencari secara parsial (`LIKE %...%`) pada field `title`, `category`, dan `external_id`.
- **sort**: `string` dengan batasan allowlist: `id`, `title`, `category`, `country`, `status`, `event_date`, `created_at`, `external_id`. Default: `id`.
- **direction**: `string` dengan batasan `asc` atau `desc`. Default: `asc`.
- **page**: `integer` minimal 1. Default: 1.
- **per_page**: `integer` antara 1–50. Default: 5.

---

## Verification

### 1. Pagination 200 OK
- **Request:** `GET /api/events?per_page=2&page=1`
- **Status:** `200 OK`
- **Actual Response:**
```json
{
    "data": [
        {
            "id": "EONET_DEV_001",
            "title": "Kebakaran Hutan Contoh",
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
            "title": "Badai Contoh",
            "category": "Severe Storms",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-02T10:30:00.000000Z",
            "latitude": -7.25,
            "longitude": 112.75,
            "source": "NASA EONET"
        }
    ],
    "links": {
        "first": "http://127.0.0.1:8000/api/events?per_page=2&page=1",
        "last": "http://127.0.0.1:8000/api/events?per_page=2&page=4",
        "prev": null,
        "next": "http://127.0.0.1:8000/api/events?per_page=2&page=2"
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 4,
        "per_page": 2,
        "to": 2,
        "total": 8
    }
}
```

### 2. Search Ditemukan 200 OK
- **Request:** `GET /api/events?search=Badai`
- **Status:** `200 OK`
- **Actual Response:**
```json
{
    "data": [
        {
            "id": "EONET_DEV_002",
            "title": "Badai Contoh",
            "category": "Severe Storms",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-02T10:30:00.000000Z",
            "latitude": -7.25,
            "longitude": 112.75,
            "source": "NASA EONET"
        }
    ],
    "links": {
        "first": "http://127.0.0.1:8000/api/events?search=Badai&page=1",
        "last": "http://127.0.0.1:8000/api/events?search=Badai&page=1",
        "prev": null,
        "next": null
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "per_page": 5,
        "to": 1,
        "total": 1
    }
}
```

### 3. Search Kosong 200 OK
- **Request:** `GET /api/events?search=tidak-ada-987654321`
- **Status:** `200 OK` (Bukan 404, query berhasil namun tidak menemukan data yang cocok)
- **Actual Response:**
```json
{
    "data": [],
    "links": {
        "first": "http://127.0.0.1:8000/api/events?search=tidak-ada-987654321&page=1",
        "last": "http://127.0.0.1:8000/api/events?search=tidak-ada-987654321&page=1",
        "prev": null,
        "next": null
    },
    "meta": {
        "current_page": 1,
        "from": null,
        "last_page": 1,
        "per_page": 5,
        "to": null,
        "total": 0
    }
}
```

### 4. Sorting Asc / Desc 200 OK
- **Request Desc:** `GET /api/events?sort=title&direction=desc&per_page=5`
  - Item pertama: `"title": "My Event - Updated"`
- **Request Asc:** `GET /api/events?sort=title&direction=asc&per_page=5`
  - Item pertama: `"title": "Badai Contoh"`
- **Actual Response (Asc):**
```json
{
    "data": [
        {
            "id": "EONET_DEV_002",
            "title": "Badai Contoh",
            "category": "Severe Storms",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-02T10:30:00.000000Z",
            "latitude": -7.25,
            "longitude": 112.75,
            "source": "NASA EONET"
        },
        {
            "id": "EONET_DEV_001",
            "title": "Kebakaran Hutan Contoh",
            "category": "Wildfires",
            "country": "Indonesia",
            "status": "Aktif",
            "date": "2026-10-01T08:00:00.000000Z",
            "latitude": -8.095,
            "longitude": 113.15,
            "source": "NASA EONET"
        }
    ],
    "meta": {
        "current_page": 1,
        "total": 8
    }
}
```

### 5. Invalid Sort 422 Unprocessable Content
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

### 6. Invalid Per Page 422 Unprocessable Content
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

---

## Response Structure

Format respon menggunakan standarisasi `AnonymousResourceCollection` berbasis `EventResource` dengan paginator Laravel yang mencakup 3 kunci utama:

1. **`data`**: Array berisi kumpulan item event hasil transformasi `EventResource`. Setiap item memuat representasi publik resource (`id`, `title`, `category`, `country`, `status`, `date`, `latitude`, `longitude`, `source`).
2. **`links`**: Objek navigasi hypermedia (HATEOAS) yang memuat URL untuk `first`, `last`, `prev`, dan `next`. Memudahkan klien frontend melakukan navigasi halaman dengan menyertakan query parameter yang sedang aktif (`withQueryString()`).
3. **`meta`**: Informasi metadata mengenai pagination saat ini, meliputi:
   - `current_page`: Halaman aktif yang sedang ditampilkan.
   - `from` & `to`: Nomor urut record awal dan akhir pada halaman aktif.
   - `last_page`: Total halaman yang tersedia berdasarkan ukuran `per_page`.
   - `per_page`: Jumlah item maksimum per halaman.
   - `total`: Total keseluruhan data yang cocok dengan kriteria filter pencarian.

---

## Decision

### Mengapa Sort Menggunakan Allowlist?
1. **Pencegahan SQL Injection & Information Disclosure**: Pengurutan kolom yang diteruskan langsung dari parameter pengguna ke klausa SQL `ORDER BY` rentan terhadap eksploitasi jika tidak dibatasi. Allowlist memastikan hanya nama kolom yang sah dan aman yang dapat dieksekusi oleh query builder.
2. **Perlindungan Skema Internal**: Kolom internal seperti kata sandi, token keamanan, atau metadata sensitif tidak boleh dapat diurutkan oleh pengguna eksternal untuk menebak atau mengekspos isi data.
3. **Integritas Query dan Mencegah Fatal Error**: Jika klien mengirimkan kolom yang tidak ada pada tabel database (misalnya `sort=non_existent_column`), database akan melempar fatal exception `Column not found` (500 Internal Server Error). Dengan validasi allowlist (`Rule::in(...)`), request invalid langsung ditolak secara graceful dengan status `422 Unprocessable Content`.

---

## AI Usage

- **Alat yang Digunakan:** Antigravity AI Coding Assistant (Gemini Flash).
- **Prompt:** Mengimplementasikan modul "Praktikum 6 — Endpoint Collection yang Dapat Dicari" untuk resource Event bencana dunia, mencakup pagination, search (title, category, external_id), sorting dengan allowlist, query validation, paginated response, dan update Postman collection.
- **Hasil:**
  - File [`app/Http/Controllers/EventController.php`](file:///d:/Heard/bencanadunia/app/Http/Controllers/EventController.php) diperbarui pada method `index()` dengan validasi query param, filter search, allowlist sorting, dan pagination `withQueryString()`.
  - File [`docs/postman_collection.json`](file:///d:/Heard/bencanadunia/docs/postman_collection.json) ditambahkan folder request pengujian Minggu 6.
  - File [`docs/minggu-6-praktikum.md`](file:///d:/Heard/bencanadunia/docs/minggu-6-praktikum.md) dibuat lengkap dengan actual response dari eksekusi backend.
- **Cara Memeriksa:**
  - Sintaks PHP divalidasi dengan `php -l app/Http/Controllers/EventController.php`.
  - Endpoint diuji langsung terhadap Laravel application kernel untuk setiap kasus uji (pagination, search normal, search kosong, sorting asc/desc, validasi error 422 untuk sort dan per_page).
  - Struktur respon dan status code diverifikasi sesuai kriteria acceptance.
