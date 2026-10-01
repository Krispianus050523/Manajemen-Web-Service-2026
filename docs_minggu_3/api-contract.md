# API Contract — InfoBencana Dunia

## User Stories

- Sebagai pengguna, saya ingin melihat daftar kejadian bencana atau fenomena alam di seluruh dunia agar saya dapat mengetahui kejadian yang sedang berlangsung.

- Sebagai pengguna, saya ingin memfilter kejadian berdasarkan kategori dan negara agar saya dapat menemukan informasi bencana berdasarkan kebutuhan saya.

- Sebagai pengguna, saya ingin melihat detail kejadian bencana atau fenomena alam di seluruh dunia agar saya dapat mengetahui informasi lebih lanjut mengenai kejadian tersebut.
- Sebagai pengguna, saya ingin mendapatkan informasi bencana dengan bahasa Indonesia agar informasi lebih mudah dipahami.
## Resource Dictionary — Event

| Field | Type | Required saat create | Akses | Aturan | Contoh |
|---|---|---|---|---|---|
| id | string | Tidak | Read-only | ID berasal dari EONET | EONET_12345 |
| title | string | Tidak | Read-only | Nama kejadian | Kebakaran Hutan |
| category | string | Tidak | Read-only | Kategori kejadian | Wildfires |
| country | string | Tidak | Read-only | Negara hasil pemetaan lokasi | Indonesia |
| status | string | Tidak | Read-only | Status kejadian | Aktif |
| date | string | Tidak | Read-only | ISO 8601 datetime | 2026-09-25T09:30:00Z |
| latitude | number | Tidak | Read-only | Latitude hasil pemetaan geometry | -6.2000 |
| longitude | number | Tidak | Read-only | Longitude hasil pemetaan geometry | 106.8166 |
| source | string | Tidak | Read-only | Sumber informasi event | NASA EONET |

## Endpoint Matrix

## Endpoint Matrix

| Kebutuhan | Method | Endpoint | Success | Error |
|---|---|---|---|---|
| Daftar kejadian | GET | /api/events | 200 | — |
| Filter kategori | GET | /api/events?category={category} | 200 | 422 |
| Filter negara | GET | /api/events?country={country} | 200 | 422 |
| Detail kejadian | GET | /api/events/{event} | 200 | 404 |

## Response Examples

### 200 OK

```json
{
  "data": [
    {
      "id": "EONET_12345",
      "title": "Kebakaran Hutan",
      "category": "Kebakaran Hutan",
      "country": "Indonesia",
      "status": "Aktif",
      "date": "2026-09-25T09:30:00Z",
      "latitude": -6.2000,
      "longitude": 106.8166,
      "source": "NASA EONET"
    }
  ]
}
```
### 201 Created

```json
{
  "data": {
    "id": "EONET_12346",
    "title": "Kebakaran Hutan",
    "category": "Kebakaran Hutan",
    "country": "Indonesia",
    "status": "Aktif",
    "date": "2026-09-28T09:30:00Z",
    "latitude": -6.2000,
    "longitude": 106.8166,
    "source": "NASA EONET"
  }
}
```
## 404 - Not Found

```json
{
  "message": "Event tidak ditemukan",
  "errors": null
}
```

## 422 - Unprocessable Content

```json
{
  "message": "Parameter tidak valid",
  "errors": {
    "category": [
      "Kategori tidak tersedia."
    ]
  }
}
```
## Design Decisions

- Menggunakan `events` sebagai main resource karena aplikasi berfokus pada penyajian kejadian bencana atau fenomena alam dari NASA EONET.
- Menggunakan method GET karena aplikasi mengambil dan menampilkan data. Pengguna tidak membuat atau mengubah data kejadian NASA.
- Field kejadian bersifat read-only karena data berasal dari sumber eksternal dan tidak dibuat oleh pengguna aplikasi.
- Filter kategori dan negara menggunakan query parameter karena filter tidak mengubah resource, tetapi hanya menentukan data yang ingin ditampilkan.
- Aplikasi menggunakan GET sebagai metode utama karena data kejadian diperoleh dari NASA EONET dan pengguna hanya membaca serta memfilter informasi.
- Example 201 Created tetap disiapkan di Postman untuk memenuhi kebutuhan praktikum API Contract, tetapi bukan merupakan fitur utama aplikasi.