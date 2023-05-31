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
API ini digunakan untuk mendapatkan maksimal 5 data lahan yang dimiliki oleh pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Image lahan akan diambil secara acak dari data dummy assets image default lahan.

### Headers
- Content-Type: application/json
- Token: {token}

### Logic
1. API menerima request dengan token JWT pada header `Token`.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari data user yang berhubungan dengan token tersebut di database dan mencari maksimal 5 lahan yang dimiliki oleh user tersebut.
        - Jika data lahan ditemukan, API akan mengembalikan response berisi data lahan tersebut.
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
      "alamat": "string"
    },
    ...
  ]
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Token: xyz987

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan data fetched successfully",
  "data": [
    {
      "id": "lahan1",
      "nama": "Lahan Pertama",
      "image": "https://example.com/default_lahan_image.jpg",
      "luas": 100.0,
      "alamat": "Jl. Lahan Pertama No.1"
    },
    {
      "id": "lahan2",
      "nama": "Lahan Kedua",
      "image": "https://example.com/default_lahan_image2.jpg",
      "luas": 200.0,
      "alamat": "Jl. Lahan Kedua No.2"
    },
    ...
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

#### Response (Lahan Not Found)
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
API ini digunakan untuk mendapatkan semua data lahan yang dimiliki oleh pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Image lahan akan diambil secara acak dari data dummy assets image default lahan.

### Headers
- Content-Type: application/json
- Token: {token}

### Logic
1. API menerima request dengan token JWT pada header `Token`.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari data user yang berhubungan dengan token tersebut di database dan mencari semua lahan yang dimiliki oleh user tersebut.
        - Jika data lahan ditemukan, API akan mengembalikan response berisi data lahan tersebut.
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
      "alamat": "string"
    },
    ...
  ]
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Token: xyz987

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan data fetched successfully",
  "data": [
    {
      "id": "lahan1",
      "nama": "Lahan Pertama",
      "image": "https://example.com/default_lahan_image.jpg",
      "luas": 100.0,
      "alamat": "Jl. Lahan Pertama No.1"
    },
    {
      "id": "lahan2",
      "nama": "Lahan Kedua",
      "image": "https://example.com/default_lahan_image2.jpg",
      "luas": 200.0,
      "alamat": "Jl. Lahan Kedua No.2"
    },
    ...
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

#### Response (Lahan Not Found)
```json
{
  "error": true,
  "message": "No lahan found",
  "data": null
}
```



---

## Tambah Lahan

### Endpoint
```
POST /lahan
```

### Deskripsi
API ini digunakan untuk menambah data lahan baru yang dimiliki oleh pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini. Image lahan akan diambil secara acak dari data dummy assets image default lahan.

### Headers
- Content-Type: application/json
- Token: {token}

### Request Body
```json
{
  "nama": "string",
  "luas": "double",
  "alamat": "string"
}
```

### Logic
1. API menerima request dengan token JWT pada header `Token` dan data nama, luas, dan alamat lahan baru pada body.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan menambahkan lahan baru dengan data yang diberikan dan image acak dari data dummy ke database, kemudian mengembalikan response sukses.

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
    "alamat": "string"
  }
}
```

### Contoh Request dan Response

#### Request
Headers:
- Content-Type: application/json
- Token: xyz987

Body:
```json
{
  "nama": "Lahan Baru",
  "luas": 150.0,
  "alamat": "Jl. Lahan Baru No.3"
}
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan successfully added",
  "data": {
    "id": "lahan3",
    "nama": "Lahan Baru",
    "image": "https://example.com/default_lahan_image3.jpg",
    "luas": 150.0,
    "alamat": "Jl. Lahan Baru No.3"
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



---

## Hapus Lahan

### Endpoint
```
DELETE /lahan/{id}
```

### Deskripsi
API ini digunakan untuk menghapus data lahan yang dimiliki oleh pengguna. Pengguna harus menyertakan token JWT dalam header untuk mengakses API ini.

### Headers
- Content-Type: application/json
- Token: {token}

### Parameters
- id (string): ID dari lahan yang ingin dihapus.

### Logic
1. API menerima request dengan token JWT pada header `Token` dan ID lahan pada path parameter.
2. API akan memvalidasi token tersebut.
    - Jika token tidak valid atau sudah expired, maka API akan mengembalikan response error.
    - Jika token valid, API akan mencari lahan dengan ID yang diberikan.
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
- Token: xyz987

Endpoint:
```
DELETE /lahan/lahan3
```

#### Response (Success)
```json
{
  "error": false,
  "message": "Lahan successfully deleted"
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
