# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
## 1. Alasan Menggunakan `RwLock<>` daripada `Mutex<>` untuk `Vec<Notification>`

Dalam tutorial ini, kita menggunakan `RwLock<Vec<Notification>>` sebagai mekanisme sinkronisasi ketika menyimpan notifikasi-notifikasi yang telah diproses. Pemilihan `RwLock<>` dilakukan secara sadar untuk mengoptimalkan performa dan keselamatan data.

### Mengapa Menggunakan `RwLock<>`

- **Multiple Readers, Single Writer**  
  `RwLock` memungkinkan **banyak thread membaca data secara bersamaan**, selama tidak ada thread yang sedang menulis. Ini sangat cocok ketika:
  - Frekuensi *read* jauh lebih tinggi daripada *write*.
  - Operasi pembacaan tidak seharusnya menghalangi thread lain membaca data yang sama.

- **Efisiensi Tinggi dalam Operasi Baca**  
  Dibanding `Mutex`, `RwLock` memberikan **kinerja lebih baik** untuk data yang sering dibaca. Ini sangat penting dalam sistem seperti notifikasi yang lebih sering ditampilkan atau dilihat dibandingkan ditulis ulang.

- **Pengendalian Akses Lebih Detail**  
  Dengan memisahkan akses baca dan tulis (`read()` dan `write()`), kita memiliki **kontrol eksplisit** terhadap kapan data dapat dimodifikasi.

### Mengapa Tidak Menggunakan `Mutex<>`

- **Mutex Mengunci Penuh**  
  `Mutex` hanya mengizinkan satu thread untuk mengakses data, baik untuk membaca maupun menulis. Ini menyebabkan:
  - **Blocking tidak perlu** pada operasi baca sederhana.
  - Kinerja menurun jika banyak thread ingin membaca.

- **Kurang Efisien untuk Skema Read-Heavy**  
  Pada sistem yang dominan operasi baca, penggunaan `Mutex` akan menghambat skalabilitas karena seluruh akses diblokir meskipun hanya membaca.

### Kesimpulan

`RwLock<>` dipilih karena:
- Lebih efisien untuk workload yang dominan baca.
- Memberikan kontrol akses yang sesuai dengan kebutuhan sinkronisasi notifikasi.
- Menjaga performa dan tetap aman untuk penggunaan multi-threaded.

---

## 2. Mengapa Rust Tidak Mengizinkan Mutasi Langsung pada Variabel `static`

Rust membatasi mutasi langsung terhadap variabel `static` untuk menjamin **keamanan memori dan thread safety**, yang merupakan prinsip utama dari bahasa ini.

### Perbedaan Pendekatan Rust dan Java

| Aspek                  | Rust                                   | Java                                 |
|------------------------|----------------------------------------|--------------------------------------|
| Mutasi Variabel Static | Harus dibungkus dengan `Mutex`/`RwLock` | Bebas dilakukan dalam metode statis  |
| Thread Safety          | Dijamin secara eksplisit                | Harus diatur manual oleh developer   |
| Konkurensi             | Dicegah jika tidak aman                 | Rentan race condition tanpa kontrol  |

### Alasan Rust Membatasi Mutasi Langsung

1. **Keamanan Thread**
   - Variabel `static` yang dapat dimutasi secara bebas akan membuka potensi *data race*.
   - Rust menghindari kondisi ini dengan **memaksa penggunaan alat sinkronisasi eksplisit**, seperti `Mutex` atau `RwLock`.

2. **Desain Bahasa yang Aman**
   - Rust menekankan **safety-first programming**.
   - Setiap modifikasi state global harus dipikirkan secara eksplisit dan aman.

3. **Solusi: lazy_static + Synchronization**
   - Rust menyediakan makro `lazy_static!` atau crate `once_cell` untuk menginisialisasi variabel statis secara aman.
   - Untuk mutasi, nilai harus dibungkus dalam tipe sinkronisasi seperti `Mutex`, `RwLock`, atau `Atomic`.

#### Reflection Subscriber-2
## 1. Eksplorasi Kode Tambahan: `src/lib.rs`

Saya melakukan eksplorasi terhadap file `src/lib.rs` dan menemukan beberapa poin penting yang sangat membantu dalam memahami struktur dan mekanisme konfigurasi aplikasi ini.

### Hal-Hal yang Dipelajari

- **Inisialisasi Global yang Aman dan Efisien**
  - File ini menggunakan `lazy_static!` untuk mendefinisikan objek global seperti `REQWEST_CLIENT`, sehingga bisa digunakan lintas modul tanpa harus membuat ulang.
  
- **Konfigurasi Aplikasi Terstruktur**
  - Terdapat struct `AppConfig` yang memuat informasi penting seperti URL publisher dan receiver.
  - Konfigurasi dapat diatur melalui `.env` maupun langsung dari `Rocket.toml` menggunakan `figment`, yang membuat aplikasi mudah diatur ulang tanpa mengubah kode.

- **Manajemen Error yang Konsisten**
  - Disediakan alias untuk `Result<T>` dan `Error` yang digunakan secara menyeluruh agar standar penanganan kesalahan konsisten di seluruh aplikasi.
  - Fungsi `compose_error_response()` membantu membuat respons kesalahan API yang terstruktur dan mudah dipahami oleh klien.

### Kesimpulan
Eksplorasi ini memperluas pemahaman saya terhadap bagaimana Rust menangani dependency injection, inisialisasi global, dan konfigurasi runtime yang fleksibel.

---

## 2. Kemudahan Menambahkan Subscriber dan Multi-Instance melalui Observer Pattern

Setelah menyelesaikan tutorial dan mencoba menjalankan beberapa instance dari Receiver secara paralel, saya menyadari betapa fleksibelnya pola Observer dalam mendukung ekspansi sistem.

### Kemudahan Plug-and-Play untuk Subscriber

- **Tanpa Modifikasi Kode Publisher**
  - Publisher tidak perlu tahu siapa saja subscriber-nya. Mereka hanya mengirim notifikasi ke daftar URL yang ada.
  
- **Penambahan Subscriber Dinamis**
  - Subscriber cukup mendaftarkan URL-nya dan siap menerima notifikasi.
  - Tidak perlu restart atau recompile aplikasi publisher.

- **Skalabilitas**
  - Sangat mudah untuk menambahkan lebih banyak Receiver baik secara horizontal (banyak instance) maupun vertikal (subscriber dengan logika berbeda).

### Tentang Menjalankan Lebih dari Satu Main App

- **Teknisnya Bisa Dilakukan**, namun dengan beberapa catatan:
  - Perlu memastikan bahwa masing-masing instance tidak mengganggu konfigurasi global (misalnya port yang sama).
  - Sistem perlu disesuaikan agar setiap instance dapat menangani subset subscriber-nya sendiri atau sinkronisasi antar instance dilakukan dengan baik.

### Kesimpulan
Pola Observer membuat sistem menjadi fleksibel, loosely coupled, dan scalable. Meskipun penambahan banyak instance `Main app` mungkin memerlukan pengelolaan tambahan, dari sisi arsitektur hal ini tetap sangat mungkin dilakukan.

---

## 3. Pengujian dan Dokumentasi dengan Postman

Saya juga melakukan eksplorasi dan penyesuaian terhadap koleksi Postman untuk meningkatkan kualitas pengujian dan dokumentasi API.

### Pengalaman yang Didapat

- **Penambahan Test Case**
  - Saya menambahkan beberapa test untuk berbagai jenis status notifikasi seperti `CREATED`, `UPDATED`, dan `DELETED`.
  - Setiap test memverifikasi respons HTTP, isi payload, dan efek samping terhadap state aplikasi (misalnya penambahan notifikasi baru).

- **Peningkatan Dokumentasi**
  - Saya mendokumentasikan struktur JSON notifikasi, penjelasan untuk setiap status, dan contoh request lengkap.
  - Hal ini sangat membantu ketika harus menjelaskan kepada anggota tim atau melakukan debugging.

- **Manfaat Fitur Environment**
  - Saya memanfaatkan fitur Environment untuk menyimpan `baseUrl` dan token jika diperlukan, sehingga tidak perlu mengubah request satu per satu.

### Nilai Praktisnya

- **Meningkatkan Kepercayaan terhadap Aplikasi**
  - Setiap perubahan bisa diuji dengan cepat dan otomatis.
  
- **Mempermudah Kolaborasi**
  - Koleksi yang terdokumentasi dan bisa dibagikan mempercepat pemahaman tim terhadap API.

- **Dasar untuk Continuous Integration**
  - Tes ini bisa digunakan sebagai dasar untuk pipeline CI/CD di masa depan.

---

## Kesimpulan Umum

Dengan eksplorasi tambahan terhadap kode dasar, eksperimen terhadap multi-subscriber dan multi-instance, serta penguatan dokumentasi Postman, saya merasa sistem ini telah menjadi fondasi kuat untuk membangun aplikasi skala lebih besar. Pola Observer terbukti memberikan fleksibilitas yang tinggi, dan praktik pengujian melalui Postman membantu menjaga kualitas serta stabilitas sistem selama dikembangkan lebih lanjut.
