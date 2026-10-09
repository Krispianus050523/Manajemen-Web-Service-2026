# Tugas Minggu 4 — Laravel API Foundations

## 1. Tujuan

Menerapkan dasar Laravel API pada project semester InfoBencana Dunia dengan menerjemahkan API contract Minggu 3 menjadi endpoint read-only yang dapat diuji menggunakan Postman.

## 2. Perubahan

Perubahan yang dilakukan pada project meliputi:

- Membuat model `Event` untuk merepresentasikan data kejadian bencana.
- Membuat migration untuk tabel `events`.
- Membuat `EventSeeder` untuk mengisi data contoh pengembangan.
- Membuat `EventController` dengan method `index()` dan `show()`.
- Menambahkan route API untuk daftar kejadian dan detail kejadian.
- Menggunakan SQLite sebagai database pengembangan.
- Menyiapkan request Postman untuk memeriksa respons API.

Data seeder merupakan data contoh untuk pengujian, bukan klaim bahwa kejadian tersebut benar-benar terjadi atau telah diverifikasi oleh NASA EONET.

## 3. Endpoint atau Contract

| Method | Endpoint | Fungsi | Status yang diharapkan |
|---|---|---|---|
| GET | `/api/events` | Menampilkan daftar kejadian | 200 OK |
| GET | `/api/events/{event}` | Menampilkan detail kejadian berdasarkan ID | 200 OK atau 404 Not Found |

Field respons mengikuti API contract Minggu 3:

- `id`
- `title`
- `category`
- `country`
- `status`
- `date`
- `latitude`
- `longitude`
- `source`

Method `GET` digunakan karena kedua endpoint hanya membaca data. Data sumber tidak diubah melalui endpoint tersebut.

ID pada respons API menggunakan `external_id`, sedangkan parameter detail `{event}` menggunakan route model binding Laravel berdasarkan ID internal model.

## 4. Bukti Pengujian

### Pengujian A — Daftar kejadian

Request:

`GET http://127.0.0.1:8000/api/events`

![Screenshot](gambar/Screenshot%202026-10-09%20170826.png)

Hasil yang diharapkan:

- Status `200 OK`.
- Body berupa JSON dengan properti `data`.
- Daftar kejadian ditampilkan dari database SQLite.

**Hasil aktual:** 

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
    ]
}

### Pengujian B — Detail kejadian

Request:

`GET http://127.0.0.1:8000/api/events/1`


![Screenshot](gambar/Screenshot%202026-10-09%20230417.png)

Hasil yang diharapkan:

- Status `200 OK` jika event dengan ID internal `1` tersedia.
- Body JSON berisi detail event.


**Hasil aktual:** 

```json
{
    "data": {
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
}

### 5. Error Case

Request untuk ID yang tidak tersedia:

`GET http://127.0.0.1:8000/api/events/999`

![Screenshot Not Found](gambar/Screenshot%202026-10-08%20202219.png)

Hasil yang diharapkan:

- Status `404 Not Found`.
- Event tidak ditemukan.

**Hasil aktual:**

{
    "message": "No query results for model [App\\Models\\Event] 999",
    "exception": "Symfony\\Component\\HttpKernel\\Exception\\NotFoundHttpException",
    "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Exceptions\\Handler.php",
    "line": 773,
    "trace": [
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Exceptions\\Handler.php",
            "line": 721,
            "function": "prepareException",
            "class": "Illuminate\\Foundation\\Exceptions\\Handler",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Routing\\Pipeline.php",
            "line": 51,
            "function": "render",
            "class": "Illuminate\\Foundation\\Exceptions\\Handler",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 224,
            "function": "handleException",
            "class": "Illuminate\\Routing\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 137,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Routing\\Router.php",
            "line": 821,
            "function": "then",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Routing\\Router.php",
            "line": 800,
            "function": "runRouteWithinStack",
            "class": "Illuminate\\Routing\\Router",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Routing\\Router.php",
            "line": 764,
            "function": "runRoute",
            "class": "Illuminate\\Routing\\Router",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Routing\\Router.php",
            "line": 753,
            "function": "dispatchToRoute",
            "class": "Illuminate\\Routing\\Router",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Kernel.php",
            "line": 200,
            "function": "dispatch",
            "class": "Illuminate\\Routing\\Router",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 180,
            "function": "{closure:Illuminate\\Foundation\\Http\\Kernel::dispatchToRouter():197}",
            "class": "Illuminate\\Foundation\\Http\\Kernel",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Middleware\\TransformsRequest.php",
            "line": 21,
            "function": "{closure:Illuminate\\Pipeline\\Pipeline::prepareDestination():178}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Middleware\\ConvertEmptyStringsToNull.php",
            "line": 31,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Middleware\\TransformsRequest",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Middleware\\ConvertEmptyStringsToNull",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Middleware\\TransformsRequest.php",
            "line": 21,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Middleware\\TrimStrings.php",
            "line": 51,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Middleware\\TransformsRequest",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Middleware\\TrimStrings",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Http\\Middleware\\ValidatePostSize.php",
            "line": 27,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Http\\Middleware\\ValidatePostSize",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Middleware\\PreventRequestsDuringMaintenance.php",
            "line": 110,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Middleware\\PreventRequestsDuringMaintenance",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Http\\Middleware\\HandleCors.php",
            "line": 74,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Http\\Middleware\\HandleCors",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Http\\Middleware\\TrustProxies.php",
            "line": 58,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Http\\Middleware\\TrustProxies",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Middleware\\InvokeDeferredCallbacks.php",
            "line": 22,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Middleware\\InvokeDeferredCallbacks",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Http\\Middleware\\ValidatePathEncoding.php",
            "line": 28,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 219,
            "function": "handle",
            "class": "Illuminate\\Http\\Middleware\\ValidatePathEncoding",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Pipeline\\Pipeline.php",
            "line": 137,
            "function": "{closure:{closure:Illuminate\\Pipeline\\Pipeline::carry():194}:195}",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Kernel.php",
            "line": 175,
            "function": "then",
            "class": "Illuminate\\Pipeline\\Pipeline",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Http\\Kernel.php",
            "line": 144,
            "function": "sendRequestThroughRouter",
            "class": "Illuminate\\Foundation\\Http\\Kernel",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\Application.php",
            "line": 1228,
            "function": "handle",
            "class": "Illuminate\\Foundation\\Http\\Kernel",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\public\\index.php",
            "line": 20,
            "function": "handleRequest",
            "class": "Illuminate\\Foundation\\Application",
            "type": "->"
        },
        {
            "file": "D:\\Heard\\bencanadunia\\vendor\\laravel\\framework\\src\\Illuminate\\Foundation\\resources\\server.php",
            "line": 23,
            "function": "require_once"
        }
    ]
}

Laravel route model binding digunakan untuk mencari event berdasarkan parameter rute. Apabila model tidak ditemukan, Laravel menghasilkan respons `404 Not Found`.

## 6. Kesimpulan

Praktikum Minggu 4 menerapkan API contract Minggu 3 ke dalam Laravel API dengan dua endpoint read-only. Implementasi menggunakan model, migration, seeder, controller, route, dan database SQLite.

Kesesuaian implementasi diverifikasi dengan membandingkan status HTTP dan struktur JSON dari hasil pengujian Postman dengan contract yang telah dirancang.

## 7. Referensi

1. Laravel Documentation — Routing: https://laravel.com/docs/13.x/routing
2. Laravel Documentation — Eloquent ORM: https://laravel.com/docs/13.x/eloquent
3. Laravel Documentation — Database Seeding: https://laravel.com/docs/13.x/seeding
4. NASA Earth Observatory Natural Event Tracker (EONET): https://eonet.gsfc.nasa.gov/docs/v3

## 8. Deklarasi Penggunaan AI

AI digunakan sebagai alat bantu untuk memahami konsep Laravel API, menyusun implementasi awal, memperbaiki kesalahan, dan menyusun dokumentasi praktikum. Implementasi dan hasil pengujian perlu diperiksa dan diverifikasi secara mandiri oleh mahasiswa.

Tidak ada API key, token, password, cookie, atau secret yang dicantumkan dalam dokumentasi ini.
