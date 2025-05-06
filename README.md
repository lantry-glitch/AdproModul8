# **AdproModul8**
### Nama: Allan Kwek
### NPM: 2306152134

### 1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?
#### - Unary RPC
- Alur: Satu permintaan, satu respons.
- Cocok untuk: Operasi sederhana seperti “getRecord”, autentikasi, atau pengecekan status (di mana hanya dibutuhkan satu balasan tunggal).

#### - Server Streaming RPC
- Alur: Klien mengirim satu permintaan, server mengirim banyak respons secara berurutan.
- Cocok untuk: Pengiriman daftar data atau update berkala misalnya, feed berita, list file, atau streaming log.

#### - Bi-directional Streaming RPC
- Alur: Klien dan server dapat saling kirim pesan secara independen dan bersamaan.
- Cocok untuk: Aplikasi real-time yang butuh interaktivitas penuh chat, kolaborasi dokumen, game multiplayer di mana kedua pihak saling kirim dan terima pesan tanpa perlu menunggu giliran.

### 2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?
#### 1. Otentikasi (Authentication)
- Gunakan TLS untuk enkripsi transport.
- Opsional: mTLS (mutual TLS) untuk memverifikasi sertifikat kedua pihak.
- Token-based (JWT) dengan crate jsonwebtoken untuk stateless authentication.

#### 2. Otorisasi (Authorization)

- Implementasikan RBAC (Role-Based Access Control) atau ACL (Access Control List).

- Scope permission per method gRPC, misalnya dengan interceptor yang memeriksa klaim token.

#### 3. Enkripsi Data

- Data in-transit: dijamin oleh TLS.

- Data at-rest: enkripsi database dengan crate seperti rust-crypto, dan kelola kunci via AWS KMS atau HashiCorp Vault.

### 3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?
- Manajemen konkurensi:
Rust async/await dan Tokio memudahkan, tapi mengoordinasi banyak stream (baca/tulis) dan mencegah race conditions perlu desain yang hati-hati.

- Backpressure & flow control:
Pastikan pesan tidak menumpuk; implementasikan mekanisme pause/resume agar tidak kehabisan memori atau overload network.

- Error handling:
Tangani error di tengah stream tanpa memutus total koneksi, dan kirim status (tonic::Status) yang tepat agar klien bisa recover atau retry.

- Message ordering:
Jaga urutan pesan, terutama untuk chat, sehingga experience pengguna tetap konsisten.
### 4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?
#### Kelebihan

- Integrasi mulus dengan Tokio async/await.

- Efisien memori (tidak memuat seluruh data di memori).

- Mendukung backpressure otomatis.

#### Kekurangan

- Abstraksi flow control terbatas(sulit kustom batching atau windowing).

- Error handling minimal; perlu lapisan tambahan untuk reconnect/disconnect.

- Buffering internal bisa menimbulkan delay atau konsumsi memori lebih besar pada throughput tinggi.
### 5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?
#### Pemisahan modul

- proto/ di file .proto & hasil codegen

- service/ di implementasi handler gRPC

- domain/ atau models/ di logika bisnis & tipe data

- utils/ di helper (logging, validasi, error conversion)

#### Trait & generik
- Abstraksi logika bisnis lewat trait (misal PaymentProcessor), di-impl pada handler, memudahkan testing dan swapping transport.

#### Error module terpadu
- Definisikan error custom, konversi ke tonic::Status di satu tempat untuk konsistensi.
### 6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?
- Validasi input: cek jumlah, format, data mandatory.

- Autentikasi & otorisasi: pastikan caller punya izin.

- Integrasi payment gateway: Stripe, PayPal, bank API; implementasi retry/backoff.

- Transaksi & konsistensi: gunakan database transaction (ACID) atau saga pattern.

- Logging & audit trail: simpan detail transaksi untuk debugging, refund, dan compliance.
### 7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?
- Kontrak-First & codegen
Memaksa schema .proto sehingga jadi konsisten lintas bahasa (Rust, Go, Python, Java).

- HTTP/2 & protobuf
Binary wire format + multiplexing sehingga latency rendah dan efisiensi bandwidth.

- Interoperabilitas
Perlu gateway (grpc-gateway) atau proxy untuk expose REST/JSON bagi klien yang tidak support gRPC.

- Microservices-friendly
Memudahkan versioning dan evolusi API.
### 8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?
#### Keunggulan HTTP/2

- Multiplexing: banyak stream dalam satu koneksi TCP.

- Header compression (HPACK).

- Native streaming & server push.

#### Kekurangan HTTP/2

- Debugging lebih sulit (binary framing).

- Infrastruktur lama (proxy, firewall) kadang tidak support sepenuhnya.

- Browser butuh grpc-web atau gateway untuk gRPC.
### 9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?
#### REST

- Unidirectional, sinkron, harus polling untuk update.

- Mudah debug (plain JSON).

- Overhead parsing & bandwidth lebih besar.

#### gRPC Streaming

- Dua arah, persistent connection, push update real-time tanpa polling.

- Latency rendah, efisiensi lebih tinggi.

- Kompleksitas lebih tinggi pada client/browser.
### 10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?
#### Protobuf (gRPC)

- Strict typing & codegen sehingga compile-time safety, versioning terstruktur.

- Binary serialization sehingga payload kecil & parsing cepat.

#### JSON (REST)

- Fleksibel & cepat prototyping.

- Risiko runtime error & inkonsistensi struktur data.

- Dokumentasi & validasi manual (OpenAPI/JSON Schema) diperlukan.
