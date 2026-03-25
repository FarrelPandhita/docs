# Panduan Integrasi Frontend 

**Base URL**: `http://localhost:3000/api`

Semua respons sukses dibungkus dalam `{ "data": ... }`.
Semua respons error dibungkus dalam `{ "errors": ... }`.

Semua endpoint yang memerlukan autentikasi harus menyertakan header berikut:
```
Authorization: Bearer <token>
```

Token diperoleh dari endpoint login atau OAuth Google, dan harus disimpan di sisi klien.

---

## Autentikasi

### POST /api/users — Registrasi

**Auth**: Tidak diperlukan

**Request Body**:
```json
{
  "username": "budisantoso",
  "name": "Budi Santoso",
  "email": "budi@mail.com",
  "phone_number": "081234567890",
  "password": "Pass123!"
}
```

**Response 201**:
```json
{
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "username": "budisantoso",
    "name": "Budi Santoso",
    "email": "budi@mail.com",
    "phoneNumber": "081234567890",
    "role": "citizen"
  }
}
```

---

### POST /api/users/login — Login

**Auth**: Tidak diperlukan

**Request Body**:
```json
{
  "email": "budi@mail.com",
  "password": "Pass123!"
}
```

**Response 200**:
```json
{
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

Simpan `data.token` untuk digunakan pada setiap request berikutnya yang memerlukan autentikasi.

---

### POST /api/users/oauth/google — Login via Google

**Auth**: Tidak diperlukan

**Request Body**:
```json
{
  "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6Ij..."
}
```

`idToken` adalah Google ID Token yang dikembalikan oleh SDK Google Sign-In setelah pengguna selesai proses login lewat popup Google. Backend akan memverifikasi token tersebut ke server Google.

Jika email dari akun Google tersebut belum terdaftar, akun baru akan otomatis dibuat dengan role `citizen`.

**Response 200**:
```json
{
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

### GET /api/users/current — Data User yang Sedang Login

**Auth**: Diperlukan

**Response 200**:
```json
{
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "username": "budisantoso",
    "name": "Budi Santoso",
    "email": "budi@mail.com",
    "phoneNumber": "081234567890",
    "role": "citizen"
  }
}
```

Field `role` bernilai `"citizen"` atau `"admin"`. Gunakan nilai ini untuk mengontrol elemen UI yang ditampilkan kepada pengguna.

---

## Proyek

### GET /api/projects — Daftar Semua Proyek

**Auth**: Tidak diperlukan

**Query Parameters** (semua opsional):

| Parameter | Tipe | Keterangan |
|---|---|---|
| `search` | string | Filter berdasarkan kata kunci judul |
| `tahun` | string (YYYY) | Filter berdasarkan tahun mulai |
| `page` | number | Nomor halaman (default: 1) |
| `limit` | number | Jumlah item per halaman (default: 10, maks: 100) |

**Response 200**:
```json
{
  "data": {
    "items": [
      {
        "id": "b93c2263-c28a-4ea5-97de-67a29e3344fd",
        "title": "Pembangunan Jalan Desa Sukamaju",
        "location": "Dusun Karanganyar, Desa Sukamaju",
        "totalBudget": "122222222",
        "status": "selesai",
        "progress": 100,
        "startDate": "2026-01-12T00:00:00.000Z",
        "endDate": "2026-03-30T00:00:00.000Z"
      },
      {
        "id": "bb258294-d33f-48e2-870f-d5281e5b4057",
        "title": "Renovasi Balai Desa",
        "location": "Pusat Desa Sukamaju",
        "totalBudget": "85000000",
        "status": "berjalan",
        "progress": 45,
        "startDate": "2026-02-01T00:00:00.000Z",
        "endDate": "2026-05-30T00:00:00.000Z"
      }
    ],
    "total": 5,
    "page": 1,
    "limit": 10,
    "totalPages": 1
  }
}
```

Catatan:
- `totalBudget` adalah representasi string dari angka besar (BigInt). Konversi ke number saat ingin menampilkan format Rupiah.
- Nilai `status`: `"perencanaan"`, `"berjalan"`, `"selesai"`.
- `progress` adalah integer 0–100, cocok digunakan untuk progress bar.
- Gunakan `total`, `page`, dan `totalPages` untuk kontrol pagination.

---

### GET /api/projects/:id — Detail Proyek

**Auth**: Tidak diperlukan

**Response 200**:
```json
{
  "data": {
    "id": "bb258294-d33f-48e2-870f-d5281e5b4057",
    "title": "Pembangunan Jalan Desa Sukamaju",
    "description": "Pembangunan jalan aspal di Dusun Karanganyar...",
    "location": "Dusun Karanganyar, Desa Sukamaju",
    "totalBudget": "120000000",
    "status": "berjalan",
    "progress": 65,
    "startDate": "2026-01-12T00:00:00.000Z",
    "endDate": "2026-03-30T00:00:00.000Z",
    "createdAt": "2026-01-10T08:00:00.000Z",
    "updatedAt": "2026-03-10T14:22:00.000Z",
    "timelines": [
      {
        "id": "uuid",
        "stageName": "Perencanaan",
        "stageDate": "2026-01-12T00:00:00.000Z",
        "status": "selesai",
        "createdAt": "2026-01-10T08:00:00.000Z"
      },
      {
        "id": "uuid",
        "stageName": "Konstruksi Tahap 1",
        "stageDate": "2026-02-10T00:00:00.000Z",
        "status": "diproses",
        "createdAt": "2026-01-10T08:00:00.000Z"
      },
      {
        "id": "uuid",
        "stageName": "Finishing",
        "stageDate": "2026-03-30T00:00:00.000Z",
        "status": "belum",
        "createdAt": "2026-01-10T08:00:00.000Z"
      }
    ],
    "expenses": [
      {
        "id": "uuid",
        "expenseName": "Material Aspal",
        "amount": "60000000",
        "percentage": "50.00"
      },
      {
        "id": "uuid",
        "expenseName": "Tenaga Kerja",
        "amount": "30000000",
        "percentage": "25.00"
      }
    ],
    "updates": [
      {
        "id": "uuid",
        "progress": 65,
        "description": "Pengecoran selesai 65% dari total jalan.",
        "createdAt": "2026-03-10T14:22:00.000Z"
      }
    ],
    "fundings": [
      {
        "id": "uuid",
        "amount": "120000000",
        "createdAt": "2026-01-10T08:00:00.000Z"
      }
    ],
    "comments": [
      {
        "id": "uuid-1",
        "comment": "Semoga cepat selesai!",
        "isAnonymous": false,
        "createdAt": "2026-03-11T10:00:00.000Z",
        "author": {
          "id": "uuid",
          "name": "Bayu Nugroho",
          "username": "bayu"
        }
      },
      {
        "id": "uuid-2",
        "comment": "Tolong pastikan kualitasnya baik.",
        "isAnonymous": true,
        "createdAt": "2026-03-10T09:30:00.000Z",
        "author": "S***"
      }
    ]
  }
}
```

Catatan:
- `timelines` diurutkan berdasarkan `stageDate` secara ascending. Nilai `status`: `"selesai"`, `"diproses"`, `"belum"`.
- `expenses[].amount` dan `expenses[].percentage` bertipe string. `percentage` adalah angka desimal dalam bentuk string (contoh: `"50.00"`).
- `updates` diurutkan dari yang terbaru. Setiap entri merekam log progres pada satu waktu tertentu.
- `comments[].author` bisa berupa objek user (ketika `isAnonymous: false`) atau string yang sudah disamarkan seperti `"S***"` (ketika `isAnonymous: true`). Periksa tipe data `author` sebelum merender.

---

### POST /api/projects — Buat Proyek Baru

**Auth**: Diperlukan (Admin saja)

**Request Body**:
```json
{
  "title": "Pembangunan Jalan Desa Sukamaju",
  "description": "Pembangunan jalan aspal di Dusun Karanganyar.",
  "location": "Dusun Karanganyar, Desa Sukamaju",
  "total_budget": 120000000,
  "start_date": "2026-01-12",
  "end_date": "2026-03-30",
  "status": "berjalan",
  "timeline": [
    { "stage_name": "Perencanaan", "stage_date": "2026-01-12", "status": "selesai" },
    { "stage_name": "Konstruksi", "stage_date": "2026-02-01", "status": "belum" }
  ],
  "expenses": [
    { "expense_name": "Material Aspal", "amount": 60000000, "percentage": 50 }
  ]
}
```

**Response 201**:
```json
{
  "data": {
    "id": "bb258294-d33f-48e2-870f-d5281e5b4057",
    "title": "Pembangunan Jalan Desa Sukamaju"
  }
}
```

---

### POST /api/projects/:id/updates — Tambah Update Progres

**Auth**: Diperlukan (Admin saja)

**Request Body**:
```json
{
  "progress": 75,
  "description": "Pengecoran 75% selesai, memasuki tahap finishing."
}
```

**Response 200**:
```json
{
  "data": {
    "message": "Progress updated successfully"
  }
}
```

---


## Statistik

### GET /api/statistics/dashboard — Statistik Dashboard

**Auth**: Tidak diperlukan

Digunakan untuk hero section homepage dan kartu ringkasan di dashboard admin.

**Response 200**:
```json
{
  "data": {
    "total_budget": "1314800000",
    "reports": {
      "total": 37,
      "unprocessed": 12
    },
    "projects": {
      "total": 5,
      "active": 2,
      "finished": 3
    }
  }
}
```

Catatan:
- `total_budget` adalah BigInt dalam bentuk string. Konversi ke number untuk format tampilan Rupiah.
- `reports.unprocessed` adalah laporan dengan status `"diterima"` yang belum ditinjau admin.

---

### GET /api/statistics/reports-pie — Breakdown Status Laporan

**Auth**: Tidak diperlukan

Digunakan untuk visualisasi pie chart atau donut chart di dashboard admin.

**Response 200**:
```json
{
  "data": {
    "total": 37,
    "breakdown": [
      { "status": "diterima", "count": 9, "percentage": 24 },
      { "status": "diproses", "count": 9, "percentage": 24 },
      { "status": "selesai",  "count": 19, "percentage": 51 }
    ]
  }
}
```

`percentage` adalah nilai integer (0–100) yang sudah dihitung dan siap digunakan langsung pada data chart.

---

## Penanganan Error

**Validation Error (400)**:
```json
{
  "errors": [
    { "field": "email", "message": "Invalid email" },
    { "field": "password", "message": "Must contain at least one symbol" }
  ]
}
```

**Single Error (401, 403, 404, 409, 500)**:
```json
{
  "errors": "Invalid email or password"
}
```

| HTTP Status | Keterangan |
|---|---|
| 400 | Validasi input gagal — periksa array `errors` untuk pesan per field |
| 401 | Belum autentikasi atau token kadaluarsa — minta pengguna login ulang |
| 403 | Sudah login tetapi role tidak cukup (bukan admin) |
| 404 | Data tidak ditemukan |
| 409 | Konflik data — contoh: email sudah terdaftar |
| 500 | Error internal server |

---

## Referensi Akses Berdasarkan Role

JWT token menyimpan field `role` di dalam payload. Decode token di sisi klien untuk menentukan elemen UI mana yang ditampilkan.

| Fitur | citizen | admin |
|---|---|---|
| Lihat daftar dan detail proyek | Ya | Ya |
| Lihat komentar proyek | Ya | Ya |
| Kirim komentar | Ya | Ya |
| Kirim laporan | Ya | Ya |
| Lihat laporan sendiri | Ya | Ya |
| Lihat semua laporan | Tidak | Ya |
| Buat atau update proyek | Tidak | Ya |
| Ubah status laporan | Tidak | Ya |
| Statistik dashboard lengkap | Tidak | Ya |
