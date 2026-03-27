# BambangShop Publisher App
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
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. Dalam buku Head First Design Pattern, Subscriber (Observer) biasanya didefinisikan sebagai interface. Namun, di kasus BambangShop saat ini, sebuah Model struct saja sudah cukup. Kenapa? Karena saat ini semua subscriber kita memiliki perilaku yang seragam, yaitu menerima notifikasi melalui sebuah URL (webhook).

Kita baru benar-benar membutuhkan trait jika nantinya ada kebutuhan untuk berbagai jenis subscriber yang perilakunya berbeda drastis. Misalnya, jika ada EmailSubscriber, SmsSubscriber, dan WebhookSubscriber yang masing-masing punya cara unik dalam memproses notifikasi. Untuk sekarang, karena hanya ada satu cara pengiriman, penggunaan struct jauh lebih simpel dan efisien di Rust.

2. Meskipun Vec (list) bisa digunakan, penggunaan DashMap jauh lebih diperlukan dalam kasus ini karena alasan berikut:

Uniqueness (Keunikan): DashMap secara otomatis menangani keunikan kunci (key). Jika kita menggunakan Vec, kita harus melakukan pengecekan manual setiap kali menambah data (iterasi seluruh list) untuk memastikan tidak ada URL yang ganda, yang mana memiliki kompleksitas waktu O(n).

Efisiensi: Mencari atau menghapus data berdasarkan URL di DashMap memiliki rata-rata kompleksitas O(1), sedangkan pada Vec adalah O(n). Untuk aplikasi toko yang mungkin punya banyak subscriber, perbedaan performa ini sangat terasa.

3. Sebenarnya, apa yang kita lakukan dengan lazy_static! dan SUBSCRIBERS sudah merupakan implementasi Singleton pattern. Kita memastikan hanya ada satu instance database "palsu" (di memori) yang bisa diakses secara global.

Namun, di Rust, Singleton saja tidak cukup jika aplikasinya multi-threaded (seperti web server Rocket). Compiler Rust sangat ketat soal keamanan memori. Kita tidak bisa mengubah (mutate) variabel statis global tanpa mekanisme sinkronisasi.

DashMap diperlukan bukan sebagai pengganti Singleton, melainkan sebagai container yang thread-safe.

Jika kita hanya menggunakan HashMap biasa dalam Singleton, program akan error saat compile atau crash karena terjadi data race ketika dua thread mencoba menambah subscriber secara bersamaan.

#### Reflection Publisher-2
1. Walaupun MVC klasik menggabungkan logika bisnis dan penyimpanan di dalam Model, dalam pengembangan aplikasi modern, kita memisahkannya untuk mengikuti Single Responsibility Principle (SRP) dan Separation of Concerns (SoC).

Model: Fokus hanya pada struktur data dan representasi entitas (misal: apa saja field yang ada di Subscriber).

Repository: Fokus pada abstraksi penyimpanan data (data access logic). Jika suatu saat kita ingin ganti dari DashMap ke database asli seperti PostgreSQL, kita hanya perlu mengubah Repository tanpa menyentuh logika bisnis.

Service: Tempat berkumpulnya "otak" aplikasi (business logic). Service yang mengatur alur data antara Repository dan Controller. Pemisahan ini membuat kode jauh lebih mudah diuji (unit testing) secara terpisah.

2. Jika kita hanya menggunakan Model tanpa Service/Repository, kita akan terjebak dalam pola "Fat Model" (Anti-pattern).
Bayangkan jika Subscriber struct juga harus menangani cara dia disimpan ke memory, cara dia memvalidasi URL-nya sendiri, hingga cara dia mengirimkan Notification.

Kompleksitas Tinggi: Setiap model akan saling ketergantungan secara langsung (tight coupling). Perubahan kecil pada cara kita menyimpan data di Subscriber bisa merusak logika pengiriman di Notification.

Sulit Maintenance: Kode akan menjadi sangat panjang dan sulit dibaca karena satu file menangani terlalu banyak hal sekaligus. Semakin banyak interaksi antar model, semakin sulit kita melacak di mana letak bug-nya.

3. Postman sangat membantu dalam menjembatani proses backend development sebelum frontend tersedia. Kita bisa mensimulasikan berbagai request (GET, POST, DELETE) dengan parameter yang berbeda-beda secara cepat.

Beberapa fitur Postman yang sangat berguna untuk proyek kelompok ke depannya:

Collections: Mengelompokkan semua endpoint API (seperti subscribe dan unsubscribe) dalam satu folder yang bisa dibagikan (export/import) ke anggota tim lain.

Environments: Kita bisa membuat variabel untuk URL (misal: {{base_url}}). Jadi, saat pindah dari localhost ke server production, kita cukup ganti satu variabel saja tanpa mengubah semua request satu per satu.

Automated Testing: Kita bisa menulis script sederhana di tab "Tests" untuk memastikan status code yang kembali selalu 200 OK atau datanya sesuai, sehingga kita tidak perlu mengecek respon secara manual setiap kali push kode baru.
#### Reflection Publisher-3
