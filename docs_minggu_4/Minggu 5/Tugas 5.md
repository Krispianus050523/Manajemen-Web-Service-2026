# Laporan Praktikum Minggu 5: Implementasi REST API CRUD & Contract-First Testing

**Mata Kuliah / Modul:** Pemrograman Berbasis Web / Backend API  
**Topik:** REST API CRUD Event Bencana Dunia, Route Model Binding, & Integrasi Pengujian Postman  
**Tanggal:** 15 Oktober 2026  

---

## 1. Tujuan

Tujuan pelaksanaan praktikum Minggu 5 ini adalah:
1. Mengimplementasikan operasi lengkap **CRUD (Create, Read, Update, Delete)** untuk resource Event bencana dunia menggunakan framework Laravel 12.
2. Membangun dan menyelaraskan respon API dengan **API Contract** yang telah disepakati melalui implementasi `Eloquent API Resource` (`EventResource`).
3. Memahami dan mengimplementasikan **Route Model Binding** pada parameter URL kustom (`external_id` dan `id` primary key).
4. Menyelesaikan kendala teknis pada HTTP method `PUT` di lingkungan PHP/Laravel, khususnya penanganan request payload (`multipart/form-data`, raw JSON, dan `x-www-form-urlencoded`) saat melakukan update data via Postman.
5. Mendokumentasikan dan memvalidasi seluruh endpoint menggunakan **Postman Collection** untuk kasus sukses (*success cases*) dan kasus gagal (*error cases*).

---

## 2. Perubahan

Berikut rincian perubahan yang telah dilakukan pada kode sumber aplikasi:

### A. Perbaikan Sintaks Dasar Controller
- **File:** [`app/Http/Controllers/EventController.php`](file:///d:/Heard/bencanadunia/app/Http/Controllers/EventController.php)
- **Perubahan:** Menghapus baris kosong/karakter sebelum tag pembuka `<?php` yang sebelumnya memicu fatal error:
  ```text
  PHP Fatal error: Namespace declaration statement has to be the very first statement or after any declare call in the script
  ```
- **Hasil:** Controller dapat dikompilasi dan dijalankan secara normal oleh web server.

### B. Optimalisasi Resolusi Route Model Binding
- **File:** [`app/Models/Event.php`](file:///d:/Heard/bencanadunia/app/Models/Event.php)
- **Perubahan:** Menambahkan fungsi override `resolveRouteBinding($value, $field = null)`.
- **Alasan:** Sebelumnya routing hanya mengikat `external_id`. Jika pengguna atau klien Postman mengakses endpoint dengan ID angka integer primary key (misal `/api/events/3`), route akan menghasilkan error `404 Not Found`.
- **Implementasi:**
  ```php
  public function resolveRouteBinding($value, $field = null)
  {
      return $this->where('external_id', $value)
          ->when(is_numeric($value), function ($query) use ($value) {
              $query->orWhere('id', $value);
          })
          ->firstOrFail();
  }
  ```

### C. Penanganan Request Body Otomatis pada Method PUT / PATCH
- **File:** [`app/Http/Controllers/EventController.php`](file:///d:/Heard/bencanadunia/app/Http/Controllers/EventController.php)
- **Perubahan:** Menambahkan fungsi privat `parseRequestBody(Request $request)` dan memanggilnya sebelum validasi pada method `update()`.
- **Alasan:** PHP native tidak mem-parsing `multipart/form-data` pada method selain `POST`. Jika pengguna menggunakan Postman dengan mode `form-data` atau mengirim raw payload tanpa header `application/json`, `$request->all()` terbaca kosong. Rule `'sometimes'` pada validasi menganggap tidak ada input sehingga data lama dikembalikan tanpa perubahan.
- **Implementasi:**
  ```php
  private function parseRequestBody(Request $request): void
  {
      if (!empty($request->all())) {
          return;
      }

      $content = $request->getContent();
      if (empty($content)) {
          return;
      }

      // 1. Coba decode JSON
      $json = json_decode($content, true);
      if (json_last_error() === JSON_ERROR_NONE && is_array($json)) {
          $request->merge($json);
          return;
      }

      // 2. Coba parse multipart/form-data pada PUT/PATCH
      if (preg_match_all('/name="([^"]+)"\r?\n\r?\n(.*?)(?=\r?\n--)/s', $content, $matches, PREG_SET_ORDER)) {
          $parsed = [];
          foreach ($matches as $match) {
              $parsed[$match[1]] = trim($match[2]);
          }
          if (!empty($parsed)) {
              $request->merge($parsed);
              return;
          }
      }

      // 3. Coba parse urlencoded
      parse_str($content, $parsed);
      if (!empty($parsed) && is_array($parsed)) {
          $request->merge($parsed);
      }
  }
  ```

---

## 3. Keputusan Utama dan Hubungannya dengan API Contract

### Keputusan 1: Pemisahan Skema Database Internal dan Skema Publik (API Contract) Menggunakan JsonResource
- **Keputusan:** Menggunakan `EventResource` untuk memformat seluruh respon endpoint.
- **Hubungan dengan API Contract:**
  1. Kontrak API menetapkan field unik sebagai `id`. Pada internal database, terdapat `id` (bigint auto-increment) dan `external_id` (string identifier dari penyedia data seperti NASA EONET/BMKG). Pada kontrak API, `external_id` dipetakan ke field `id` di respon JSON.
  2. Format tanggal `event_date` distandarkan ke format ISO-8601 (`toISOString()`) melalui key `date`.
  3. Memastikan integritas data tipe numerik (`latitude` dan `longitude` dikonversi eksplisit ke float).

### Keputusan 2: Dual Identifier Route Model Binding
- **Keputusan:** Mendukung binding parameter `{event}` baik melalui `external_id` (contoh: `event-123`, `EONET_DEV_001`) maupun `id` numerik (contoh: `1`, `3`).
- **Hubungan dengan API Contract:**
  Klien API pihak ketiga dapat memanggil endpoint detail, update, dan delete menggunakan identifier eksternal yang mereka miliki tanpa perlu mengetahui auto-increment internal database. Namun, untuk integrasi internal / dashboard yang menggunakan id numerik, endpoint tetap fleksibel dan kompatibel ke belakang (*backward-compatible*).

### Keputusan 3: Penggunaan Validasi `'sometimes'` pada Endpoint Pembaruan Data
- **Keputusan:** Method `update()` mendukung pembaruan penuh (PUT) dan pembaruan parsial (PATCH) dengan validasi kombinasi `'sometimes'` dan `'required'`.
- **Hubungan dengan API Contract:**
  Klien dapat mengirimkan hanya field yang ingin diubah (misalnya hanya `{"title": "My Event Update"}`) tanpa dipaksa menyertakan seluruh field data lainnya. Hal ini mencegah error 422 saat klien melakukan partial update.

---

### Spesifikasi Endpoint & API Contract

Base URL: `http://127.0.0.1:8000/api`

| No | Method | Endpoint | Deskripsi | Status Code |
|:---|:---|:---|:---|:---|
| 1 | `GET` | `/events` | Menampilkan seluruh event bencana | `200 OK` |
| 2 | `GET` | `/events/{event}` | Menampilkan detail 1 event (berdasarkan id/external_id) | `200 OK` |
| 3 | `POST` | `/events` | Menambahkan event bencana baru | `201 Created` |
| 4 | `PUT` | `/events/{event}` | Memperbarui event bencana secara penuh / parsial | `200 OK` |
| 5 | `PATCH` | `/events/{event}` | Memperbarui field tertentu dari event | `200 OK` |
| 6 | `DELETE`| `/events/{event}` | Menghapus event bencana dari database | `204 No Content` |

#### Format Kontrak Respon Data (EventResource)
```json
{
  "data": {
    "id": "event-123",
    "title": "My Event Update",
    "category": "workshop",
    "country": null,
    "status": null,
    "date": null,
    "latitude": null,
    "longitude": null,
    "source": "NASA EONET"
  }
}
```

---

## 4. Bukti Pengujian

Pengujian dilakukan menggunakan Postman dan cURL terhadap server Laravel lokal (`http://127.0.0.1:8000`).

### A. Pengujian GET Index (`/api/events`)
- **Method:** `GET`
- **URL:** `http://127.0.0.1:8000/api/events`
- **Status Code:** `200 OK`
- **Respon Cuplikan:**
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
      "id": "event-123",
      "title": "My Event Update",
      "category": "workshop",
      "country": null,
      "status": null,
      "date": null,
      "latitude": null,
      "longitude": null,
      "source": "NASA EONET"
    }
  ]
}
```

### B. Pengujian POST Store (`/api/events`)
- **Method:** `POST`
- **URL:** `http://127.0.0.1:8000/api/events`
- **Request Body:**
```json
{
  "external_id": "EVT-TEST-004",
  "title": "Banjir Bandang",
  "category": "Floods",
  "country": "Indonesia",
  "status": "Aktif",
  "event_date": "2026-10-15 10:00:00",
  "latitude": -6.2088,
  "longitude": 106.8456,
  "source": "BMKG"
}
```
- **Status Code:** `201 Created`
- **Respon:**
```json
{
  "data": {
    "id": "EVT-TEST-004",
    "title": "Banjir Bandang",
    "category": "Floods",
    "country": "Indonesia",
    "status": "Aktif",
    "date": "2026-10-15T10:00:00.000000Z",
    "latitude": -6.2088,
    "longitude": 106.8456,
    "source": "BMKG"
  }
}
```

### C. Pengujian PUT Update (`/api/events/{event}`)
- **Method:** `PUT`
- **URL:** `http://127.0.0.1:8000/api/events/event-123`
- **Request Body:**
```json
{
  "title": "My Event Update"
}
```
- **Status Code:** `200 OK`
- **Respon:**
```json
{
  "data": {
    "id": "event-123",
    "title": "My Event Update",
    "category": "workshop",
    "country": null,
    "status": null,
    "date": null,
    "latitude": null,
    "longitude": null,
    "source": "NASA EONET"
  }
}
```
*Catatan: Nilai `title` berhasil terupdate menjadi `"My Event Update"` baik dikirim via `raw JSON` maupun `form-data`.*

### D. Pengujian DELETE Destroy (`/api/events/{event}`)
- **Method:** `DELETE`
- **URL:** `http://127.0.0.1:8000/api/events/EVT-TEST-004`
- **Status Code:** `204 No Content`
- **Respon Body:** *(Kosong sesuai standar RESTful)*

---

## 5. Error Case

Berikut adalah pengujian penanganan kasus error (*negative test cases*):

### Kasus 1: Error Validasi Input (Status `422 Unprocessable Content`)
- **Trigger:** Mengirim request `POST /api/events` tanpa field wajib (`external_id`, `title`, atau `category`), atau mengirim `external_id` yang sudah terdaftar (*duplicate unique key*).
- **Request Body:**
  ```json
  {
    "title": ""
  }
  ```
- **Respon Status:** `422 Unprocessable Content`
- **Respon JSON:**
  ```json
  {
    "message": "The external id field is required. (and 2 more errors)",
    "errors": {
      "external_id": [
        "The external id field is required."
      ],
      "title": [
        "The title field is required."
      ],
      "category": [
        "The category field is required."
      ]
    }
  }
  ```

### Kasus 2: Resource Tidak Ditemukan (Status `404 Not Found`)
- **Trigger:** Memanggil endpoint `GET`, `PUT`, atau `DELETE` dengan ID yang tidak terdapat pada database.
- **URL:** `http://127.0.0.1:8000/api/events/non-existent-id-999`
- **Header:** `Accept: application/json`
- **Respon Status:** `404 Not Found`
- **Respon JSON:**
  ```json
  {
    "message": "No query results for model [App\\Models\\Event]."
  }
  ```

### Kasus 3: Keterbatasan Parser Native PHP pada PUT Form-Data
- **Trigger:** Mengirim payload via Postman dengan method `PUT` dan mode `form-data`.
- **Dampak Sebelumnya:** PHP tidak mem-parsing body sehingga nilai `$request->all()` kosong dan data tidak terupdate.
- **Solusi yang Diterapkan:** Ditangani secara otomatis melalui `parseRequestBody()` di controller sehingga data tetap berhasil di-parse dan tersimpan tanpa error.

---

## 6. Kesimpulan

1. Operasi CRUD pada resource `Event` telah berfungsi secara penuh dan memenuhi standar arsitektur REST API.
2. Penggunaan `EventResource` mempermudah penyelarasan struktur data keluaran dengan *API Contract*, sehingga perubahan struktur tabel pada basis data tidak merusak format konsumsi data pada sisi klien.
3. Penambahan parser mandiri pada controller dan penyesuaian route model binding berhasil mengatasi kendala tidak munculnya update data pada pengujian menggunakan method `PUT` di Postman.
4. Seluruh skenario pengujian sukses dan penanganan error telah terdokumentasi dan siap dieksekusi melalui berkas Postman Collection yang disediakan.

---

## 7. Referensi

1. Dokumentasi Resmi Laravel 12: [Routing & Route Model Binding](https://laravel.com/docs/routing#route-model-binding)
2. Dokumentasi Resmi Laravel 12: [Eloquent: API Resources](https://laravel.com/docs/eloquent-resources)
3. Dokumentasi Resmi Laravel 12: [Validation](https://laravel.com/docs/validation)
4. Postman Documentation: [Creating and Exporting Collections](https://learning.postman.com/docs/collections/collections-overview/)
5. PHP RFC / Manual: [Handling PUT and Multipart Data in PHP](https://www.php.net/manual/en/features.file-upload.put-method.php)

---

## 8. Deklarasi Penggunaan AI

Dalam penyelesaian tugas dan perbaikan kode pada Minggu 5 ini:
- **Alat Bantu AI:** Google Antigravity Coding Assistant (Gemini 3.8 Flash).
- **Ruang Lingkup Penggunaan AI:**
  1. Menganalisis penyebab kegagalan method `PUT` saat memperbarui event di Postman (keterbatasan native parser `form-data` pada HTTP `PUT` di PHP dan fatal error tag `<?php`).
  2. Merancang implementasi helper `parseRequestBody()` dan override `resolveRouteBinding()` pada model `Event`.
  3. Membantu penyusunan berkas koleksi Postman `docs/postman_collection.json`.
  4. Membantu menyusun dokumentasi laporan praktikum `docs/minggu-5.md` sesuai dengan format yang telah ditentukan.
- **Pernyataan:** Seluruh kode yang dihasilkan dan disesuaikan telah diperiksa, diuji, dan dipahami fungsinya secara menyeluruh oleh pengembang.
### Dokumentasi
![Screenshot Success Get Event](Gambar/Tugas%205.1.png)
![Screenshot Success Create Event](Gambar/tugas%205.2.png)
![Screenshot Success Update Event](Gambar/Tugas%205.3.png)