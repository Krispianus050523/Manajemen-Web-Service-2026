# Praktikum Minggu 4 — Laravel API

## Resource

Event

## Endpoint

- GET /api/events
- GET /api/events/{event}

## Verification

### 200 List — Berhasil

Request Postman:

`GET http://127.0.0.1:8000/api/events`

Hasil:

- Status: `200 OK`
- Response berhasil menampilkan daftar event.
- Data berasal dari database Laravel yang telah diisi melalui seeder.

### 200 Detail — Berhasil

![Screenshot Success Get Event](gambar/Screenshot%202026-10-08%20202320.png)

Request Postman:

`GET http://127.0.0.1:8000/api/events/1`

Hasil:

- Status: `200 OK`
- Response berhasil menampilkan detail event berdasarkan ID.
- Data yang ditampilkan sesuai dengan data event pada database.

### 404 Not Found


![Screenshot Not Found](gambar/Screenshot%202026-10-08%20202219.png)

Request Postman:

`GET http://127.0.0.1:8000/api/events/999`

Hasil:

- Status: `404 Not Found`
- Response menunjukkan bahwa event dengan ID tersebut tidak ditemukan.

## Contract Comparison

Actual response dibandingkan dengan API Contract Minggu 3.

Endpoint `GET /api/events` telah sesuai dengan contract karena menggunakan method `GET`, mengembalikan response dalam format JSON, dan menggunakan status `200 OK` ketika data berhasil ditemukan.

Endpoint `GET /api/events/{event}` juga sesuai dengan contract karena mengembalikan detail event dengan status `200 OK` ketika data ditemukan dan `404 Not Found` ketika event tidak ditemukan.

Struktur data response juga mengikuti field yang telah ditentukan pada contract, yaitu:

- `id`
- `title`
- `category`
- `country`
- `status`
- `date`
- `latitude`
- `longitude`
- `source`

Dengan demikian, implementasi API pada Minggu 4 telah menerjemahkan contract Minggu 3 ke dalam endpoint Laravel yang dapat dijalankan dan diuji menggunakan Postman.

## Evidence

### Postman Request 1 — Get Events

Nama request:

`GET Events`

URL:

`GET http://127.0.0.1:8000/api/events`

Ringkasan hasil:

Response berhasil dengan status `200 OK` dan menampilkan dua data event dari database.

### Postman Request 2 — Get Event Detail

Nama request:

`GET Event Detail`

URL:

`GET http://127.0.0.1:8000/api/events/1`

Ringkasan hasil:

Response berhasil dengan status `200 OK` dan menampilkan detail event yang ditemukan.

### Postman Request 3 — Event Not Found

Nama request:

`GET Event Not Found`

URL:

`GET http://127.0.0.1:8000/api/events/999`

Ringkasan hasil:

Response menghasilkan status `404 Not Found` karena event dengan ID tersebut tidak tersedia.

> Catatan: Tidak ada token atau secret yang dicantumkan dalam dokumentasi.