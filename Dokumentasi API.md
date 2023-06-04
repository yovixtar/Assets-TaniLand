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
  "userid": "xyz987",
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
| tanggal_tanam   | Date         |                                         |
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
            - Jika data tanam ditemukan, API akan menghitung umur tanam dan menambahkannya ke response.
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
