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
- Authorization: {token}

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
    "lasted_login": "datetime"
  }
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: xyz987

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
    "lasted_login": "2023-05-23T10:00:00Z"
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

## Get Home Lahan

### Endpoint
```
GET /home/lahan
```

### Deskripsi
API ini digunakan untuk mendapatkan data lahan yang dimiliki pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. API ini akan mengembalikan maksimum 5 data lahan.

### Headers
- Content-Type: application/json
- Authorization: {token}

### Logic
1. API menerima request dengan token JWT pada header `Authorization`.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari data lahan yang berhubungan dengan token tersebut di database.
        - Jika data lahan ditemukan, API akan mengembalikan response berisi data lahan (maksimum 5 data).
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
    // ... maximum 5 lahan
  ]
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Authorization: xyz987

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan data fetched successfully",
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
    // ... up to 5 lahan
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

## Get All User's Lahan

### Endpoint
```
GET /lahan
```

### Deskripsi
API ini digunakan untuk mendapatkan semua data lahan yang dimiliki oleh pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Authorization: {token}

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
- Authorization: xyz987

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
- Authorization: {token}

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
    - Jika token valid, API akan menambahkan data lahan baru ke database dengan data dari request body dan gambar acak dari data dummy.
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
- Authorization: xyz987

Body:
```json
{
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

## Delete Lahan (Field)

### Endpoint
```
DELETE /lahan/{id}
```

### Deskripsi
API ini digunakan untuk menghapus lahan pengguna. Pengguna harus menyertakan token JWT dalam header dan ID lahan dalam URL untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Authorization: {token}

### Logic
1. API menerima request dengan token JWT pada header `Authorization` dan ID lahan pada URL.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari lahan dengan ID tersebut di database.
        - Jika lahan ditemukan, API akan menghapus lahan tersebut dan mengembalikan response sukses.
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

URL: https://base_url.com/api/v1/lahan/abcd1234

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
