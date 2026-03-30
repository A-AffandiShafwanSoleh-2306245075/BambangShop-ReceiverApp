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
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambanshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1

## 1. Mengapa menggunakan RwLock dan bukan Mutex

Pada tutorial ini digunakan `RwLock<Vec<Notification>>` untuk menyimpan data notifikasi di receiver. Pemilihan ini berkaitan dengan kondisi aplikasi yang dapat diakses secara bersamaan oleh beberapa proses, terutama saat publisher mengirim notifikasi dalam waktu yang berdekatan.

`RwLock` memberikan dua jenis akses, yaitu read dan write. Beberapa thread dapat melakukan read secara bersamaan, tetapi hanya satu thread yang dapat melakukan write dalam satu waktu. Dalam implementasi yang saya lakukan, method `write()` digunakan saat menambahkan notifikasi, sedangkan `read()` digunakan saat menampilkan daftar notifikasi.

Jika dilihat dari kebutuhan sistem, operasi membaca data kemungkinan terjadi lebih sering dibandingkan penambahan data. Dengan menggunakan `RwLock`, proses membaca dapat berjalan secara paralel tanpa harus menunggu proses lain selesai, selama tidak ada proses write yang sedang berlangsung.

Berbeda dengan `Mutex`, yang hanya mengizinkan satu thread mengakses data dalam satu waktu, baik untuk read maupun write. Hal ini dapat menyebabkan bottleneck ketika banyak request masuk, karena semua akses harus menunggu giliran.

Berdasarkan hal tersebut, penggunaan `RwLock` dirasa lebih sesuai karena tetap menjaga keamanan data sekaligus memberikan performa yang lebih baik pada kondisi concurrent.

---

## 2. Mengapa menggunakan lazy_static dan bukan static biasa seperti di Java

Dalam tutorial ini digunakan `lazy_static` untuk mendefinisikan variabel global seperti `Vec` dan `DashMap`. Hal ini berbeda dengan Java, di mana variabel static dapat diubah secara langsung melalui method static.

Rust memiliki aturan yang lebih ketat terkait penggunaan variabel global. Secara default, variabel `static` bersifat immutable dan tidak dapat diubah secara langsung. Hal ini bertujuan untuk mencegah masalah seperti data race yang bisa terjadi dalam lingkungan multi-threading.

Untuk mengatasi kebutuhan tersebut, digunakan `lazy_static` yang memungkinkan inisialisasi variabel global saat runtime. Selain itu, variabel tersebut tetap dapat diakses secara aman jika dibungkus dengan struktur seperti `RwLock` atau `Mutex`.

Dalam implementasi yang saya lakukan, `NOTIFICATIONS` dibuat menggunakan `lazy_static` dan dibungkus dengan `RwLock`. Dengan cara ini, data dapat diakses dan dimodifikasi dengan aman oleh beberapa thread.

Jika menggunakan `static` biasa, Rust tidak akan mengizinkan perubahan data secara langsung. Hal ini berbeda dengan Java, tetapi justru menjadi keunggulan Rust dalam menjaga keamanan program.

Oleh karena itu, penggunaan `lazy_static` dalam kasus ini merupakan solusi yang tepat untuk mengelola data global secara aman dalam lingkungan concurrent.
#### Reflection Subscriber-2

## 1. Eksplorasi di luar tutorial (misalnya src/lib.rs)

Selama mengerjakan tutorial ini, saya tidak hanya mengikuti langkah yang diberikan, tetapi juga sempat melihat beberapa file lain seperti `src/lib.rs`. Dari situ saya mulai paham bahwa file tersebut berfungsi sebagai pusat konfigurasi aplikasi.

Di dalamnya terdapat beberapa hal penting seperti `APP_CONFIG`, `REQWEST_CLIENT`, serta tipe `Result` dan fungsi `compose_error_response`. Dengan melihat bagian ini, saya jadi lebih mengerti bagaimana aplikasi ini bekerja secara keseluruhan, tidak hanya di bagian controller atau service saja.

Saya juga menyadari bahwa penggunaan HTTP client dibuat secara global agar bisa digunakan di banyak tempat, dan penanganan error dibuat seragam supaya lebih mudah dikelola. Dari sini, saya jadi punya gambaran yang lebih jelas tentang struktur project yang baik.

---

## 2. Kemudahan Observer Pattern dalam menambah subscriber

Setelah mencoba menjalankan beberapa instance Receiver di port yang berbeda, saya merasa bahwa penggunaan Observer pattern sangat membantu dalam menambahkan subscriber baru.

Setiap Receiver hanya perlu melakukan subscribe, tanpa perlu mengubah bagian lain dari sistem. Publisher juga tidak perlu tahu detail persis tiap subscriber, karena semuanya sudah dikelola oleh repository. Jadi ketika ada event seperti create atau delete product, semua subscriber akan otomatis menerima notifikasi.

Namun, ketika saya membayangkan jika ada lebih dari satu Main App (Publisher), situasinya jadi lebih rumit. Karena masing-masing publisher punya data sendiri, subscriber harus subscribe ke masing-masing publisher secara terpisah. Tidak ada sinkronisasi otomatis antar publisher.

Jadi menurut saya, Observer pattern ini sangat cocok untuk kasus banyak subscriber, tapi akan butuh penyesuaian tambahan kalau jumlah publisher juga bertambah.

---

## 3. Penggunaan Postman untuk testing dan dokumentasi

Dalam tutorial ini, saya menggunakan Postman untuk mencoba semua endpoint yang sudah dibuat, seperti subscribe, unsubscribe, receive, dan melihat daftar notifikasi.

Postman sangat membantu karena saya bisa langsung mengirim request dan melihat hasilnya tanpa perlu membuat frontend. Ini membuat proses testing jadi lebih cepat dan praktis.

Selain itu, fitur collection juga memudahkan saya dalam mengelompokkan endpoint, jadi tidak perlu mengetik ulang setiap request. Saya juga jadi lebih mudah memahami alur komunikasi antara publisher dan receiver.

Walaupun saya belum mencoba fitur testing otomatis di Postman, saya merasa fitur tersebut akan sangat berguna jika digunakan di project yang lebih besar, terutama untuk memastikan API tetap berjalan dengan baik setelah ada perubahan.

Secara keseluruhan, penggunaan Postman cukup membantu saya dalam memahami dan menguji sistem yang saya buat di tutorial ini.