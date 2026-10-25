# Hands-on: Komunikasi Inter-service dengan gRPC (Polyglot)

## 1. Deskripsi Praktikum

Praktikum ini membahas implementasi komunikasi antar-layanan (*inter-service communication*) menggunakan gRPC dengan pendekatan *polyglot*, yaitu penggunaan beberapa bahasa pemrograman dalam satu sistem.

Pada proyek ini, Go digunakan sebagai server, sedangkan Python dan Node.js digunakan sebagai client. Ketiga komponen berkomunikasi menggunakan gRPC dengan definisi pesan dan layanan yang disimpan dalam file `user.proto`.

Pada tahap pengembangan, struktur pesan `UserResponse` diperluas dengan menambahkan field `phone` untuk mengirimkan nomor telepon pengguna.

## 2. Tujuan Praktikum

1. Memahami konsep komunikasi inter-service menggunakan gRPC.
2. Memahami penggunaan Protocol Buffers untuk mendefinisikan pesan dan layanan.
3. Membangun Go Server yang menyediakan layanan gRPC.
4. Menghubungkan Python Client dengan Go Server.
5. Menghubungkan Node.js Client dengan Go Server.
6. Menambahkan field `phone` pada pesan respons.
7. Menguji komunikasi antara server dan client yang menggunakan bahasa pemrograman berbeda.
8. Mengelola kode program menggunakan Git dan GitHub.

## 3. Alat dan Bahan

Perangkat lunak yang digunakan:

- Windows
- Visual Studio Code
- Go
- Python
- Node.js dan npm
- Protocol Buffers Compiler (`protoc`)
- `protoc-gen-go`
- `protoc-gen-go-grpc`
- Python packages: `grpcio` dan `grpcio-tools`
- Node.js packages: `@grpc/grpc-js` dan `@grpc/proto-loader`
- Git
- GitHub

## 4. Arsitektur Sistem

```text
              +-----------------------+
              |       Go Server       |
              |     gRPC :50051       |
              +-----------+-----------+
                          |
                 gRPC Request/Response
                    /             \
                   /               \
       +-----------+-----+   +-----+-----------+
       |  Python Client  |   |  Node.js Client |
       +----------------+   +-----------------+

              Contract: user.proto
```

Go Server menyediakan layanan `UserService` dengan metode `GetUserProfile`. Python Client dan Node.js Client mengirimkan permintaan berdasarkan ID pengguna, kemudian menerima data dari server.

## 5. Struktur Direktori

```text
grpc-polyglot-lab/
├── .gitignore
├── user.proto
├── go-server/
│   ├── go.mod
│   ├── go.sum
│   ├── main.go
│   └── pb/
│       ├── user.pb.go
│       └── user_grpc.pb.go
├── python-client/
│   ├── .venv/
│   ├── client.py
│   ├── user_pb2.py
│   └── user_pb2_grpc.py
└── node-client/
    ├── node_modules/
    ├── client.js
    ├── package.json
    └── package-lock.json
```

**Keterangan:**

- `user.proto`: definisi pesan dan layanan gRPC.
- `go-server/main.go`: implementasi server.
- `go-server/pb/`: kode Go hasil generasi Protocol Buffers.
- `python-client/client.py`: implementasi Python Client.
- `python-client/user_pb2.py`: kode pesan hasil generasi Protocol Buffers.
- `python-client/user_pb2_grpc.py`: kode layanan gRPC untuk Python.
- `node-client/client.js`: implementasi Node.js Client.
- `.gitignore`: aturan file dan direktori yang tidak perlu disimpan ke Git.

Folder `.venv` dan `node_modules` dibuat secara lokal dan tidak perlu diunggah ke repository.

## 6. Langkah-Langkah Praktikum

### 6.1 Persiapan Lingkungan

Pastikan Go, Python, Node.js, Git, dan `protoc` sudah terpasang.

Periksa instalasi menggunakan terminal PowerShell:

```powershell
go version
python --version
node --version
git --version
protoc --version
```

Pastikan plugin Protocol Buffers untuk Go juga tersedia:

```powershell
protoc-gen-go --version
protoc-gen-go-grpc --version
```

### 6.2 Membuat Struktur Proyek

Buat direktori utama proyek dengan nama `grpc-polyglot-lab`.

Di dalamnya, buat direktori:

```text
go-server/
python-client/
node-client/
```

Buat file `user.proto` pada direktori utama.

### 6.3 Membuat Definisi Layanan gRPC

Isi awal file `user.proto`:

```protobuf
syntax = "proto3";

package user;

option go_package = "go-server/pb";

message UserRequest {
    int32 user_id = 1;
}

message UserResponse {
    string name = 1;
    string email = 2;
}

service UserService {
    rpc GetUserProfile (UserRequest) returns (UserResponse);
}
```

File tersebut mendefinisikan pesan permintaan `UserRequest`, pesan respons `UserResponse`, serta layanan `UserService`.

Metode `GetUserProfile` digunakan untuk memperoleh profil pengguna berdasarkan ID.

### 6.4 Implementasi Go Server

Masuk ke direktori Go Server:

```powershell
cd C:\Users\HP\grpc-polyglot-lab\go-server
```

Inisialisasi modul Go jika belum tersedia:

```powershell
go mod init go-server
```

Jika `go.mod` sudah tersedia, lewati perintah inisialisasi.

Pasang dependensi gRPC:

```powershell
go get google.golang.org/grpc
go get google.golang.org/protobuf
```

Hasilkan kode Go dari file Protocol Buffers:

```powershell
protoc --proto_path=../ --go_out=. --go_opt=module=go-server --go-grpc_out=. --go-grpc_opt=module=go-server ../user.proto
```

Perintah tersebut menghasilkan:

- `pb/user.pb.go`
- `pb/user_grpc.pb.go`

Implementasikan server pada `main.go`. Server mendengarkan koneksi pada port `50051` dan menyediakan metode `GetUserProfile`.

Periksa kompilasi:

```powershell
go build .
```

Jalankan server:

```powershell
go run .
```

Output yang diharapkan:

```text
gRPC Server (Go) berjalan di port 50051...
```

Biarkan terminal server tetap berjalan selama pengujian client.

### 6.5 Implementasi Python Client

Buka terminal baru, kemudian masuk ke direktori Python Client:

```powershell
cd C:\Users\HP\grpc-polyglot-lab\python-client
```

Buat lingkungan virtual:

```powershell
python -m venv .venv
```

Pasang dependensi:

```powershell
.\.venv\Scripts\python.exe -m pip install grpcio grpcio-tools
```

Hasilkan kode Python dari file `.proto`:

```powershell
.\.venv\Scripts\python.exe -m grpc_tools.protoc --proto_path=../ --python_out=. --grpc_python_out=. ../user.proto
```

Buat file `client.py` untuk membuka koneksi ke `localhost:50051`, memanggil `GetUserProfile` dengan `user_id=1`, dan menampilkan respons dari server.

Jalankan client:

```powershell
.\.venv\Scripts\python.exe client.py
```

Hasil pengujian awal menunjukkan bahwa Python Client berhasil menerima nama dan email pengguna dari Go Server.

### 6.6 Implementasi Node.js Client

Buka terminal baru dan masuk ke direktori Node.js Client:

```powershell
cd C:\Users\HP\grpc-polyglot-lab\node-client
```

Inisialisasi proyek Node.js jika `package.json` belum tersedia:

```powershell
npm.cmd init -y
```

Pasang dependensi:

```powershell
npm.cmd install @grpc/grpc-js @grpc/proto-loader
```

Buat file `client.js`. Client menggunakan `@grpc/proto-loader` untuk membaca file `user.proto` dan `@grpc/grpc-js` untuk melakukan komunikasi gRPC.

Node.js Client terhubung ke `localhost:50051`, memanggil `GetUserProfile` dengan `user_id=1`, lalu menampilkan respons dari server.

Jalankan client:

```powershell
node client.js
```

Hasil pengujian awal menunjukkan bahwa Node.js Client berhasil menerima nama dan email pengguna dari Go Server.

### 6.7 Menambahkan Field Phone

Setelah komunikasi awal berhasil, file `user.proto` diperbarui untuk menambahkan field nomor telepon.

Perubahan pada `UserResponse`:

```protobuf
message UserResponse {
    string name = 1;
    string email = 2;
    string phone = 3;
}
```

Simpan perubahan file tersebut.

**Regenerasi kode Go**

Hentikan Go Server menggunakan `Ctrl + C`, lalu jalankan dari direktori `go-server`:

```powershell
protoc --proto_path=../ --go_out=. --go_opt=module=go-server --go-grpc_out=. --go-grpc_opt=module=go-server ../user.proto
```

Perbarui `main.go` agar respons pengguna dengan ID `1` menyertakan nomor telepon:

```go
return &pb.UserResponse{
    Name:  "Budi Santoso",
    Email: "budi@universitas.ac.id",
    Phone: "081234567890",
}, nil
```

Nomor tersebut merupakan data contoh untuk praktikum.

Periksa kompilasi:

```powershell
go build .
```

**Regenerasi kode Python**

Masuk ke direktori `python-client` dan jalankan:

```powershell
.\.venv\Scripts\python.exe -m grpc_tools.protoc --proto_path=../ --python_out=. --grpc_python_out=. ../user.proto
```

Perbarui `client.py` agar menampilkan `response.phone`.

**Pembaruan Node.js Client**

Node.js menggunakan `@grpc/proto-loader` untuk membaca definisi `.proto` saat program berjalan. Oleh karena itu, setelah `user.proto` diperbarui, sesuaikan `client.js` agar menampilkan `response.phone`.

### 6.8 Pengujian Setelah Penambahan Phone

Jalankan kembali Go Server dari direktori `go-server`:

```powershell
go run .
```

Biarkan server tetap berjalan.

Pada terminal Python Client:

```powershell
cd C:\Users\HP\grpc-polyglot-lab\python-client
.\.venv\Scripts\python.exe client.py
```

Pada terminal Node.js Client:

```powershell
cd C:\Users\HP\grpc-polyglot-lab\node-client
node client.js
```

Kedua client diharapkan menerima nama, email, dan nomor telepon dari Go Server.

## 7. Hasil Pengujian

### 7.1 Python Client

Output pengujian setelah penambahan field `phone`:

```text
Mengirim request ke Go Server menggunakan Python...
Respon dari Server -> Nama: Budi Santoso, Email: budi@universitas.ac.id, Telepon: 081234567890
```

### 7.2 Node.js Client

Output pengujian setelah penambahan field `phone`:

```text
Mengirim request ke Go Server menggunakan Node.js...
Respon dari Server -> Nama: Budi Santoso, Email: budi@universitas.ac.id, Telepon: 081234567890
```

### 7.3 Analisis

Hasil pengujian menunjukkan bahwa Go Server berhasil melayani permintaan dari Python Client dan Node.js Client. Kedua client menggunakan kontrak Protocol Buffers yang sama sehingga dapat membaca data respons dari server.

Penambahan field `phone` berhasil diterapkan dengan memperbarui file `.proto`, menghasilkan ulang kode Go dan Python, serta menyesuaikan implementasi server dan client.

## 8. Pengelolaan Proyek Menggunakan Git dan GitHub

Proyek dikelola menggunakan Git
