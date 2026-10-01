# Tugas 3 — Draft API Contract
## InfoBencana Dunia

## 1. Tujuan

Tugas ini bertujuan menerapkan konsep API contract dan resource modelling pada project InfoBencana Dunia.

Aplikasi InfoBencana Dunia dirancang untuk menampilkan informasi kejadian bencana atau fenomena alam dari berbagai wilayah di dunia dengan menggunakan data NASA EONET.

API contract digunakan sebagai acuan untuk menentukan resource, endpoint, HTTP method, parameter, struktur JSON, dan status code sebelum implementasi API dilakukan.

## 2. Perubahan

Pada Minggu 3 dilakukan beberapa perubahan pada rancangan API:

- M- Menentukan `events` sebagai resource utama.
- Membuat resource dictionary untuk event.
- Menentukan endpoint REST API.
- Menentukan filter berdasarkan kategori.
- Menentukan filter berdasarkan negara.
- Menentukan endpoint untuk melihat detail suatu event.
- Menentukan struktur response JSON.
- Menentukan status code untuk response sukses dan error.
- Membuat response examples pada Postman.
- Membuat dokumentasi API contract.
## 3. Resource

Resource utama yang digunakan adalah `events`.

Dalam aplikasi ini, event berarti satu kejadian alam atau bencana yang tercatat dari sumber NASA EONET.

Event:

- Kebakaran hutan.
- Badai.
- Aktivitas gunung berapi.
- Gempa bumi.
- Tanah longsor.

Satu event memiliki beberapa informasi seperti ID, nama kejadian, kategori, negara, status, tanggal, koordinat, dan sumber data.

### Resource Dictionary

| Field | Type | Akses | Keterangan |
|---|---|---|---|
| id | string | Read-only | ID kejadian dari sumber data |
| title | string | Read-only | Nama kejadian |
| category | string | Read-only | Kategori kejadian |
| country | string | Read-only | Negara lokasi kejadian |
| status | string | Read-only | Status kejadian |
| date | string | Read-only | Waktu kejadian dalam ISO 8601 |
| latitude | number | Read-only | Koordinat lintang |
| longitude | number | Read-only | Koordinat bujur |
| source | string | Read-only | Sumber data |

## 4. Endpoint / API Contract

| Kebutuhan | Method | Endpoint | Success | Error |
|---|---|---|---|---|
| Daftar kejadian | GET | `/api/events` | 200 | — |
| Filter kategori | GET | `/api/events?category={category}` | 200 | 422 |
| Filter negara | GET | `/api/events?country={country}` | 200 | 422 |
| Detail kejadian | GET | `/api/events/{event}` | 200 | 404 |

Semua endpoint menggunakan resource `events`.

Filter kategori dan negara menggunakan query parameter karena filter hanya menentukan data yang ditampilkan dan tidak membuat resource baru.

## 5. Bukti Response dari API Sumber

### success get event

#### GET https://eonet.gsfc.nasa.gov/api/v3/events
```json
 {
            "id": "EONET_24909",
            "title": "Tropical Storm Hanna",
            "description": null,
            "link": "https://eonet.gsfc.nasa.gov/api/v3/events/EONET_24909",
            "closed": null,
            "categories": [
                {
                    "id": "severeStorms",
                    "title": "Severe Storms"
                }
            ],
            "sources": [
                {
                    "id": "NOAA_NHC",
                    "url": "https://www.nhc.noaa.gov/archive/2026/HANNA.shtml"
                }
            ],
            "geometry": [
                {
                    "magnitudeValue": 40.00,
                    "magnitudeUnit": "kts",
                    "date": "2026-09-28T15:00:00Z",
                    "type": "Point",
                    "coordinates": [
                        -50.4,
                        36.6
                    ]
                }
            ]
        }
  ``` 

![Screenshot Success Get Event](gambar/Screenshot%202026-10-01%20101709.png)

### error get event

![Screenshot Error Get Event](gambar/Screenshot%202026-10-01%20102031.png)
