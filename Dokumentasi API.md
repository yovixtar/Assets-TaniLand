### Tabel Pengguna (Users)

| Kolom       | Tipe Data  | Keterangan                         |
|-------------|------------|------------------------------------|
| id          | String     | Primary Key, ID dari pengguna      |
| username    | String     | Nama pengguna                      |
| email       | String     | Email pengguna                     |
| photo       | String     | URL foto profil pengguna           |
| premium     | Boolean    | Status premium pengguna            |
| token       | String     | Token autentikasi pengguna         |
| lasted_login| DateTime   | Waktu login terakhir pengguna      |
| created_at  | DateTime   | Waktu pembuatan akun pengguna      |
| updated_at  | DateTime   | Waktu update terakhir data pengguna|

### Tabel Lahan (Lahan)

| Kolom       | Tipe Data  | Keterangan                          |
|-------------|------------|-------------------------------------|
| id          | String     | Primary Key, ID dari lahan          |
| user_id     | String     | Foreign Key, ID pengguna pemilik lahan |
| nama        | String     | Nama lahan                          |
| image       | String     | URL gambar lahan                    |
| luas        | Double     | Luas lahan (dalam meter persegi)    |
| alamat      | String     | Alamat lahan                        |
| lat         | Double     | Latitude lahan (optional)           |
| lon         | Double     | Longitude lahan (optional)          |
| created_at  | DateTime   | Waktu pembuatan data lahan          |
| updated_at  | DateTime   | Waktu update terakhir data lahan    |
| deleted_at  | DateTime   | Waktu penghapusan data lahan (optional)|



# Dokumentasi API

```txt
Catatan : untuk semua Select / Get pastikan memberikan kondisi Where deleted_at = null
```


## Base URL
```
https://base_url.com/api/v1/
```

---

## API Authentication

### Endpoint
```
POST /auth
```

### Deskripsi
API ini digunakan untuk proses autentikasi pengguna. Saat aplikasi berhasil melakukan autentikasi melalui Google Email, aplikasi akan mengirimkan request ke API ini dengan data username dan email.

### Logic
1. API menerima request berupa data username dan email.
2. API akan mengecek apakah data user tersebut sudah ada di database atau belum.
    - Jika user belum terdaftar (data user tidak ditemukan di database), maka API akan:
        - Mendaftarkan user baru dengan data yang telah diberikan, termasuk menambahkan `created_at` dan `lasted_login`.
        - Membuat token autentikasi untuk user baru tersebut.
        - Mengembalikan respon berisi data user yang baru terdaftar, termasuk token yang telah dibuat.
    - Jika user sudah terdaftar (data user ditemukan di database), maka API akan:
        - Memperbarui `lasted_login` dan token autentikasi user.
        - Mengembalikan respon berisi data user, termasuk token yang baru.

### Headers
- Content-Type: application/json

### Request Body
```json
{
  "username": "string",
  "email": "string"
}
```

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string",
  "data": {
    "id": "string",
    "username": "string",
    "email": "string",
    "photo": "string",
    "premium": "boolean",
    "token": "string",
    "lasted_login": "datetime"
  }
}
```

### Contoh Request dan Response

#### Request
```json
{
  "username": "user1",
  "email": "user1@example.com"
}
```

#### Response (New User)
```json
{
  "error": false,
  "message": "User successfully registered",
  "data": {
    "id": "abcd1234",
    "username": "user1",
    "email": "user1@example.com",
    "photo": "https://example.com/photo.jpg",
    "premium": false,
    "token": "xyz987",
    "lasted_login": "2023-05-23T10:00:00Z"
  }
}
```

#### Response (Existing User)
```json
{
  "error": false,
  "message": "User successfully logged in",
  "data": {
    "id": "abcd1234",
    "username": "user1",
    "email": "user1@example.com",
    "photo": "https://example.com/photo.jpg",
    "premium": false,
    "token": "newtoken456",
    "lasted_login": "2023-05-23T12:00:00Z"
  }
}
```

#### Response (Error)
```json
{
  "error": true,
  "message": "Authentication failed",
  "data": null
}
```



---

## Logout User

### Endpoint
```
POST /logout
```

### Deskripsi
API ini digunakan untuk proses logout pengguna. Pengguna harus menyertakan emailnya dalam request body. Proses logout akan menghapus token autentikasi pengguna.

### Headers
- Content-Type: application/json

### Request Body
```json
{
  "email": "string"
}
```

### Logic
1. API menerima request berisi email pengguna.
2. API akan mencari data user yang berhubungan dengan email tersebut di database.
    - Jika data user ditemukan, API akan menghapus token autentikasi pengguna tersebut dan mengembalikan response sukses.
    - Jika data user tidak ditemukan, API akan mengembalikan response error.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
```json
{
  "email": "user1@example.com"
}
```

#### Response (Success)
```json
{
  "error": false,
  "message": "User successfully logged out"
}
```

#### Response (User Not Found)
```json
{
  "error": true,
  "message": "User not found"
}
```



---

## Get User

### Endpoint
```
GET /user
```

### Deskripsi
API ini digunakan untuk mendapatkan data pengguna yang saat ini autentikasi. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Logic
1. API menerima request dengan token JWT pada header `Authorization`.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari data user yang berhubungan dengan token tersebut di database.
        - Jika data user ditemukan, API akan mengembalikan response berisi data user.
        - Jika data user tidak ditemukan, API akan mengembalikan response error.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string",
  "data": {
    "id": "string",
    "username": "string",
    "email": "string",
    "photo": "string",
    "premium": "boolean",
    "lasted_login": "datetime",
    "lahan": [
      {
      "id": "string",
      "nama": "string",
      "image": "string",
      "luas": "double",
      "alamat": "string",
      "lat": "double",
      "lon": "double"
      }
      // ... Max 5 Lahan
    ]
  }
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

#### Response (Success)
```json
{
  "error": false,
  "message": "User data fetched successfully",
  "data": {
    "id": "abcd1234",
    "username": "user1",
    "email": "user1@example.com",
    "photo": "https://example.com/photo.jpg",
    "premium": false,
    "lasted_login": "2023-05-23T10:00:00Z",
    "lahan": [
      {
      "id": "ABC",
      "nama": "Lahan 1",
      "image": "url_image.png",
      "luas": 8,
      "alamat": "Alamat Lengkap",
      "lat": null,
      "lon": null
      }
      // ... Max 5 Lahan
    ]
  }
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token",
  "data": null
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired",
  "data": null
}
```

#### Response (User Not Found)
```json
{
  "error": true,
  "message": "User not found",
  "data": null
}
```



---

## Get All User's Lahan

### Endpoint
```
GET /lahan
```

### Deskripsi
API ini digunakan untuk mendapatkan semua data lahan yang dimiliki oleh pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Logic
1. API menerima request dengan token JWT pada header `Authorization`.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari semua data lahan yang berhubungan dengan token tersebut di database.
        - Jika data lahan ditemukan, API akan mengembalikan response berisi semua data lahan.
        - Jika data lahan tidak ditemukan, API akan mengembalikan response error.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string",
  "data": [
    {
      "id": "string",
      "nama": "string",
      "image": "string",
      "luas": "double",
      "alamat": "string",
      "lat": "double",
      "lon": "double"
    },
    // ... all lahan
  ]
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

#### Response (Success)
```json
{
  "error": false,
  "message": "All lahan data fetched successfully",
  "data": [
    {
      "id": "lahan1",
      "nama": "Lahan 1",
      "image": "https://example.com/lahan1.jpg",
      "luas": 10.5,
      "alamat": "Jl. Lahan 1, Bandung",
      "lat": -6.917464,
      "lon": 107.619123
    },
    {
      "id": "lahan2",
      "nama": "Lahan 2",
      "image": "https://example.com/lahan2.jpg",
      "luas": 20.0,
      "alamat": "Jl. Lahan 2, Jakarta",
      "lat": -6.200000,
      "lon": 106.816666
    },
    // ... all lahan
  ]
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token",
  "data": null
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired",
  "data": null
}
```

#### Response (No Lahan Found)
```json
{
  "error": true,
  "message": "No lahan found",
  "data": null
}
```



---

## Add Lahan

### Endpoint
```
POST /lahan
```

### Deskripsi
API ini digunakan untuk menambahkan lahan baru milik pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Request Body
```json
{
  "nama": "string",
  "luas": "double",
  "alamat": "string",
  "lat": "double",
  "lon": "double"
}
```

### Logic
1. API menerima request dengan token JWT pada header `Authorization` dan data lahan pada body request.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan menambahkan data lahan baru ke database dengan data dari request body dan gambar acak dari data dummy, jika data lat & lon null masih bisa di input.
        - Jika proses berhasil, API akan mengembalikan response sukses.
        - Jika proses gagal, API akan mengembalikan response error.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Body:
```json
{
  "user_id": "xyz987",
  "nama": "Lahan 3",
  "luas": 15.0,
  "alamat": "Jl. Lahan 3, Surabaya",
  "lat": -7.250445,
  "lon": 112.768845
}
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan added successfully"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```

#### Response (Failure in Adding Lahan)
```json
{
  "error": true,
  "message": "Failed to add lahan"
}
```



---

## Delete Lahan

### Endpoint
```
DELETE /lahan/{id}
```

### Deskripsi
API ini digunakan untuk menghapus lahan milik pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Menghapus lahan hanya akan mengubah nilai `deleted_at` di database.

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Parameters
- id: String (ID dari lahan yang akan dihapus)

### Logic
1. API menerima request dengan token JWT pada header `Authorization` dan ID lahan pada path.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari lahan dengan ID yang sesuai di database.
        - Jika lahan ditemukan, API akan mengubah nilai `deleted_at` lahan tersebut dan mengembalikan response sukses.
        - Jika lahan tidak ditemukan, API akan mengembalikan response error.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: xyz987

Endpoint:
```
DELETE /lahan/lahan1
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan deleted successfully"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```

#### Response (Lahan Not Found)
```json
{
  "error": true,
  "message": "Lahan not found"
}
```



TANAM:
| Field           | Type Data    | Keterangan                              |
|-----------------|--------------|-----------------------------------------|
| id              | String       | Primary Key                             |
| bibit_id        | String       | Foreign Key dari tabel Bibit             |
| lahan_id        | String       | Foreign Key dari tabel Lahan             |
| jarak           | Integer      | Default 30                              |
| status          | Enum         | Nilai yang dapat dipilih: plan, exec, close |
| tanggal_tanam   | Date         | Null                                    |
| tanggal_panen   | Date         | Null                                    |
| jumlah_panen    | Integer      | Null                                    |
| harga_panen     | Integer      | Null                                    |
| created_at      | Timestamp    |                                         |
| updated_at      | Timestamp    |                                         |
| deleted_at      | Timestamp    |                                         |

BIBIT:
| Field           | Type Data    | Keterangan                              |
|-----------------|--------------|-----------------------------------------|
| id              | String       | Primary Key                             |
| nama            | String       |                                         |
| photo           | String       |                                         |
| deskripsi       | Text         |                                         |
| harga_beli      | Integer      |                                         |
| jenis           | Enum         | Nilai yang dapat dipilih: Sayuran / Buah |
| link_market     | String       | Default tani.iyabos.com/marketplace      |
| created_at      | Timestamp    |                                         |
| updated_at      | Timestamp    |                                         |
| deleted_at      | Timestamp    |                                         |

AKTIVITAS:
| Field           | Type Data    | Keterangan                              |
|-----------------|--------------|-----------------------------------------|
| id              | String       | Primary Key                             |
| tanam_id        | String       | Foreign Key dari tabel Tanam             |
| nama            | String       |                                         |
| keterangan      | Text         |                                         |
| pupuk           | Integer      | null                                    |
| tanggal_aktifitas| Date         |                                         |
| created_at      | Timestamp    |                                         |
| updated_at      | Timestamp    |                                         |
| deleted_at      | Timestamp    |                                         |



---

## Get Detail Lahan

### Endpoint
```
GET /lahan/{id}
```

### Deskripsi
API ini digunakan untuk mendapatkan detail lahan pengguna termasuk data tanam, bibit, dan aktivitas. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Authorization: {token}

### Parameters
- id: String (ID dari lahan)

### Logic
1. API menerima request dengan token JWT pada header `Authorization` dan ID lahan pada path.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari lahan dengan ID yang sesuai di database.
        - Jika lahan ditemukan, API akan mencari data tanam, bibit, dan aktivitas yang berkaitan dengan lahan tersebut.
            - Jika data tanam ditemukan, API akan menghitung umur tanam (jika tanggal_panen null maka hasil perhitungan selisih hari ini dengan tanggal_tanam, jika tanggal_panen tidak null maka hasil dari selisih tanggal_tanam dan tanggal_panen (bukan negatif)) dan menambahkannya ke response.
            - Jika data bibit ditemukan, API akan menambahkannya ke response.
            - Jika data aktivitas ditemukan, API akan menambahkannya ke response.
            - Jika data tanam, bibit, atau aktivitas tidak ditemukan, API akan mengembalikan response dengan data kosong pada bagian tersebut.
        - Jika lahan tidak ditemukan, API akan mengembalikan response error.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string",
  "data": {
    "id": "string",
    "nama": "string",
    "image": "string",
    "luas": "double",
    "alamat": "string",
    "lat": "double",
    "lon": "double",
    "tanam": {
      "id": "string",
      "jarak": "integer",
      "status": "enum",
      "tanggal_tanam": "date",
      "tanggal_panen": "date",
      "jumlah_panen": "integer",
      "harga_panen": "integer",
      "umur": "integer",
      "bibit": {
        "id": "string",
        "nama": "string",
        "photo": "string",
        "deskripsi": "text",
        "harga_beli": "integer",
        "jenis": "enum",
        "link_market": "string"
      },
      "aktivitas": [
        {
          "id": "string",
          "nama": "string",
          "keterangan": "text",
          "pupuk": "integer",
          "tanggal_aktifitas": "date"
        },
        // ... max 5 aktivitas
      ]
    }
  }
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: xyz987

Endpoint:
```
GET /lahan/lahan1
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Detail lahan fetched successfully",
  "data": {
    "id": "lahan1",
    "nama": "Lahan 1",
    "image": "https://example.com/lahan1.jpg",
    "luas": 10.5,
    "alamat": "Jl. Lahan 1, Bandung",
    "lat": -6.917464

,
    "lon": 107.619123,
    "tanam": {
      "id": "tanam1",
      "jarak": 30,
      "status": "plan",
      "tanggal_tanam": "2023-06-04",
      "tanggal_panen": null,
      "jumlah_panen": null,
      "harga_panen": null,
      "umur": 0,
      "bibit": {
        "id": "bibit1",
        "nama": "Bibit Tomat",
        "photo": "https://example.com/bibit1.jpg",
        "deskripsi": "Bibit tomat unggul",
        "harga_beli": 10000,
        "jenis": "Sayuran",
        "link_market": "https://tani.iyabos.com/marketplace"
      },
      "aktivitas": [
        {
          "id": "aktivitas1",
          "nama": "Pemupukan",
          "keterangan": "Pemupukan tahap 1",
          "pupuk": 1,
          "tanggal_aktifitas": "2023-06-04"
        }
      ]
    }
  }
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```

#### Response (Lahan Not Found)
```json
{
  "error": true,
  "message": "Lahan not found"
}
```



---

## Get Semua Bibit

### Endpoint
```
GET /bibit
```

### Deskripsi
API ini digunakan untuk mendapatkan semua data bibit yang tersedia. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Token di header harus dimulai dengan kata "Bearer".

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Logic
1. API menerima request dengan token JWT pada header `Authorization`. Token harus diawali dengan kata "Bearer".
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mengambil semua data bibit dari database dan mengembalikan response sukses berupa data bibit.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string",
  "data": [
    {
      "id": "string",
      "nama": "string",
      "photo": "string",
      "deskripsi": "text",
      "harga_beli": "integer",
      "jenis": "enum",
      "link_market": "string"
    },
    // ... semua data bibit
  ]
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Endpoint:
```
GET /bibit
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Data bibit fetched successfully",
  "data": [
    {
      "id": "bibit1",
      "nama": "Bibit Tomat",
      "photo": "https://example.com/bibit1.jpg",
      "deskripsi": "Bibit tomat unggul",
      "harga_beli": 10000,
      "jenis": "Sayuran",
      "link_market": "https://tani.iyabos.com/marketplace"
    },
    // ... semua data bibit
  ]
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```



### Tabel iot

| Field     | Type   | Keterangan              |
|-----------|--------|-------------------------|
| id        | String | Primary Key             |
| user_id   | String | Foreign Key, Nullable   |
| lahan_id  | String | Foreign Key, Nullable   |
| created_at| timestamp |                       |
| updated_at| timestamp |                       |
| deleted_at| timestamp |                       |

### Tabel hasil_iot

| Field            | Type       | Keterangan                 |
|------------------|------------|----------------------------|
| id               | String     | Primary Key                |
| iot_id          | String     | Foreign Key                |
| kelembaban_udara | Double     |                            |
| suhu             | Double     |                            |
| created_at       | timestamp  |                            |
| updated_at       | timestamp  |                            |
| deleted_at       | timestamp  |                            |

### Base Data iot

| id                | user_id  | created_at          | updated_at          | deleted_at          |
|-------------------|----------|---------------------|---------------------|---------------------|
| prototype-taniland|          | Waktu Sekarang      |                     |                     |



---

## Tambah Data Hasil IoT

### Endpoint
```
GET /hasil-iot
```

### Deskripsi
API ini digunakan oleh perangkat IoT untuk mengirim data pengukuran suhu dan kelembaban udara. Data tersebut akan ditambahkan ke database.

### Parameters
- suhu: Double (Data suhu dari perangkat IoT)
- kelembaban_udara: Double (Data kelembaban udara dari perangkat IoT)
- iot_id: String (ID dari perangkat IoT)

### Logic
1. API menerima request dengan parameter `suhu`, `kelembaban_udara`, dan `iot_id`.
2. API akan memvalidasi data tersebut.
    - Jika data tidak valid (contoh: `suhu` atau `kelembaban_udara` berada di luar rentang yang diterima, atau `iot_id` tidak ditemukan dalam database), API akan mengembalikan response error.
    - Jika data valid, API akan menambahkan data tersebut ke database dan mengembalikan response sukses.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Endpoint:
```
GET /hasil-iot?suhu=25.5&kelembaban_udara=60.0&iot_id=iot1
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Data IoT berhasil ditambahkan"
}
```

#### Response (Data Tidak Valid)
```json
{
  "error": true,
  "message": "Data tidak valid"
}
```

#### Response (IoT ID Tidak Ditemukan)
```json
{
  "error": true,
  "message": "IoT ID tidak ditemukan"
}
```



---

## Tambah Data Tanam (Plan)

### Endpoint
```
POST /tanam/plan
```

### Deskripsi
API ini digunakan untuk menambah data rencana penanaman (plan) ke dalam database. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Token di header harus dimulai dengan kata "Bearer".

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Body
- bibit_id: String (ID dari bibit yang akan ditanam)
- lahan_id: String (ID dari lahan yang akan digunakan)

### Logic
1. API menerima request dengan body berisi `bibit_id` dan `lahan_id`, dan token JWT pada header `Authorization`. Token harus diawali dengan kata "Bearer".
2. API akan memvalidasi data tersebut.
    - Jika `bibit_id` atau `lahan_id` tidak ditemukan dalam database, atau jika `lahan_id` sudah memiliki data tanam dengan status "plan" atau "exec", API akan mengembalikan response error.
    - Jika data valid, API akan menambahkan data tersebut ke database dengan status "plan", `jarak` default menjadi 30, dan semua field tanggal kecuali `created_at` menjadi null. Kemudian API mengembalikan response sukses.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Body:
```json
{
  "bibit_id": "bibit1",
  "lahan_id": "lahan1"
}
```

Endpoint:
```
POST /tanam/plan
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Data plan tanam berhasil ditambahkan"
}
```

#### Response (Bibit atau Lahan Tidak Ditemukan)
```json
{
  "error": true,
  "message": "Bibit atau lahan tidak ditemukan"
}
```

#### Response (Lahan Sudah Mempunyai Rencana atau Proses Tanam)
```json
{
  "error": true,
  "message": "Lahan sudah mempunyai rencana atau proses tanam"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```



---

## Update Status Tanam (Exec)

### Endpoint
```
POST /tanam/exec
```

### Deskripsi
API ini digunakan untuk mengubah status penanaman dari "plan" menjadi "exec". Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Token di header harus dimulai dengan kata "Bearer".

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Body
- id: String (ID dari tanam yang akan diubah statusnya)
- jarak: Integer (jarak penanaman)
- tanggal_tanam: Date (tanggal penanaman)

### Logic
1. API menerima request dengan body berisi `id`, `jarak`, dan `tanggal_tanam`, dan token JWT pada header `Authorization`. Token harus diawali dengan kata "Bearer".
2. API akan memvalidasi data tersebut.
    - Jika `id` tidak ditemukan dalam database, atau jika status tanam bukan "plan", API akan mengembalikan response error.
    - Jika data valid, API akan mengubah status tanam menjadi "exec", mengupdate `jarak` dan `tanggal_tanam` sesuai data yang diterima, dan mengembalikan response sukses.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Body:
```json
{
  "id": "tanam1",
  "jarak": 30,
  "tanggal_tanam": "2023-06-12"
}
```

Endpoint:
```
POST /tanam/exec
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Status tanam berhasil diubah menjadi 'exec'"
}
```

#### Response (Tanam Tidak Ditemukan atau Status bukan Plan)
```json
{
  "error": true,
  "message": "Data tanam tidak ditemukan atau status bukan 'plan'"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```




---

## Update Status Tanam (Close)

### Endpoint
```
POST /tanam/close
```

### Deskripsi
API ini digunakan untuk mengubah status penanaman dari "exec" menjadi "close". Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Token di header harus dimulai dengan kata "Bearer".

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Body
- id: String (ID dari tanam yang akan diubah statusnya)
- tanggal_panen: Date (tanggal panen)
- jumlah_panen: Integer (jumlah panen)
- harga_panen: Integer (harga panen)

### Logic
1. API menerima request dengan body berisi `id`, `tanggal_panen`, `jumlah_panen`, dan `harga_panen`, serta token JWT pada header `Authorization`. Token harus diawali dengan kata "Bearer".
2. API akan memvalidasi data tersebut.
    - Jika `id` tidak ditemukan dalam database, atau jika status tanam bukan "exec", API akan mengembalikan response error.
    - Jika data valid, API akan mengubah status tanam menjadi "close", mengupdate `tanggal_panen`, `jumlah_panen`, dan `harga_panen` sesuai data yang diterima, dan mengembalikan response sukses.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Body:
```json
{
  "id": "tanam1",
  "tanggal_panen": "2023-07-15",
  "jumlah_panen": 500,
  "harga_panen": 10000
}
```

Endpoint:
```
POST /tanam/close
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Status tanam berhasil diubah menjadi 'close'"
}
```

#### Response (Tanam Tidak Ditemukan atau Status bukan Exec)
```json
{
  "error": true,
  "message": "Data tanam tidak ditemukan atau status bukan 'exec'"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```



---

## Ambil Data Tanam yang Sudah Dipanen (Close)

### Endpoint
```
GET /tanam/close?lahan_id={lahan_id}
```

### Deskripsi
API ini digunakan untuk mengambil data tanam yang sudah dipanen berdasarkan ID lahan. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Token di header harus dimulai dengan kata "Bearer".

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### Parameters
- lahan_id: String (ID dari lahan yang ingin dilihat data tanamnya)

### Logic
1. API menerima request dengan parameter `lahan_id` dan token JWT pada header `Authorization`. Token harus diawali dengan kata "Bearer".
2. API akan memvalidasi data tersebut.
    - Jika `lahan_id` tidak ditemukan dalam database, API akan mengembalikan response error.
    - Jika data valid, API akan mencari semua data tanam yang memiliki status "close" dan berada pada lahan dengan ID yang dimaksud. Untuk setiap data tanam, API juga akan mengambil nama bibit yang digunakan dari tabel Bibit.
    - API kemudian mengembalikan response berisi data tanam dan nama bibit yang dimaksud.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string",
  "data": [
    {
      "id": "string",
      "bibit_nama": "string",
      "jarak": "integer",
      "status": "string",
      "tanggal_tanam": "date",
      "tanggal_panen": "date",
      "jumlah_panen": "integer",
      "harga_panen": "integer"
    }
    // ... data tanam lainnya
  ]
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Endpoint:
```
GET /tanam/close?lahan_id=lahan1
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Data tanam berhasil diambil",
  "data": [
    {
      "id": "tanam1",
      "bibit_nama": "bibit1",
      "jarak": 30,
      "status": "close",
      "tanggal_tanam": "2023-06-12",
      "tanggal_panen": "2023-07-15",
      "jumlah_panen": 500,
      "harga_panen": 10000
    }
    // ... data tanam lainnya
  ]
}
```

#### Response (Lahan Tidak Ditemukan)
```json
{
  "error": true,
  "message": "Lahan tidak ditemukan"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```



---

## Delete Data Tanam

### Endpoint
```
DELETE /tanam/{id}
```

### Deskripsi
API ini digunakan untuk menghapus data tanam berdasarkan ID tanam. Sebenarnya, ini tidak menghapus data secara fisik, tetapi mengisi nilai `deleted_at` dengan timestamp saat ini. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Token di header harus dimulai dengan kata "Bearer".

### Headers
- Content-Type: application/json
- Authorization: Bearer {token}

### URL Parameters
- id: String (ID dari tanam yang ingin dihapus)

### Logic
1. API menerima request dengan `id` tanam pada URL dan token JWT pada header `Authorization`. Token harus diawali dengan kata "Bearer".
2. API akan memvalidasi data tersebut.
    - Jika `id` tidak ditemukan dalam database, atau jika `deleted_at` dari tanam sudah terisi, API akan mengembalikan response error.
    - Jika data valid, API akan mengisi `deleted_at` dengan timestamp saat ini, dan mengembalikan response sukses.

### Response
API akan mengembalikan respon dengan format berikut:
```json
{
  "error": "boolean",
  "message": "string"
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: Bearer xyz987

Endpoint:
```
DELETE /tanam/tanam1
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Data tanam berhasil dihapus"
}
```

#### Response (Tanam Tidak Ditemukan atau Sudah Dihapus)
```json
{
  "error": true,
  "message": "Data tanam tidak ditemukan atau sudah dihapus"
}
```

#### Response (Token Invalid)
```json
{
  "error": true,
  "message": "Invalid token"
}
```

#### Response (Token Expired)
```json
{
  "error": true,
  "message": "Token expired"
}
```
