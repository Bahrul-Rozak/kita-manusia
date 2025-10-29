# Performance Testing dengan K6: Skenario Akses Halaman Website

Halo teman-teman, hari ini kita akan membahas tentang implementasi performance testing menggunakan K6 untuk menguji performa sebuah website edukasi. Dalam sesi testing kali ini, kita akan melakukan pengujian terhadap website https://kf-lab.sahabatbelajar.com/ dengan pendekatan yang realistis dan bertahap. Seperti yang kita ketahui, performance testing merupakan bagian penting dalam siklus pengembangan software untuk memastikan aplikasi dapat menangani beban pengguna yang diharapkan tanpa mengalami penurunan kualitas layanan.

Latar belakang dari skenario testing yang akan kita jalankan hari ini adalah untuk mengukur kemampuan website dalam menangani traffic pengguna secara simultan. Kita memulai dengan konfigurasi yang moderat - hanya 20 virtual user - mengingat keterbatasan akun cloud gratis yang kita miliki. Pendekatan ini sangat masuk akal karena dalam tahap awal testing, kita tidak ingin langsung membebani sistem dengan load yang ekstrem, melainkan memahami dulu behavior sistem under test sebelum melakukan pengujian yang lebih intensif.

## File Structure dan Setup

Kita akan membuat sebuah file JavaScript dengan nama `performance-test-k6.js`. Pemilihan nama ini cukup straightforward karena langsung mencerminkan konten dari file tersebut - sebuah script performance testing menggunakan K6. Dalam pengembangan performance test script, penamaan yang jelas dan deskriptif sangat penting untuk memudahkan maintenance dan kolaborasi tim.

## Source Code Lengkap

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 10 },  // Ramp-up perlahan
    { duration: '3m', target: 20 },  // Stabil di 20 VU
    { duration: '2m', target: 0 },   // Ramp-down
  ],
  thresholds: {
    http_req_failed: ['rate<0.05'],    // Error rate < 5%
    http_req_duration: ['p(95)<5000'], // 95% requests < 5 detik
  },
};

export default function () {
  const url = 'https://kf-lab.sahabatbelajar.com/';
  
  const response = http.get(url);
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response body is not empty': (r) => r.body.length > 0,
    'content type is html': (r) => r.headers['Content-Type']?.includes('text/html'),
  });
  
  sleep(1);
}
```

## Penjelasan Detail Step by Step

Mari kita breakdown setiap bagian dari kode di atas untuk memahami alur dan logika di balik implementasi performance test ini. Dimulai dari bagian paling atas, kita mengimpor module-module yang diperlukan dari K6. Module `http` digunakan untuk melakukan HTTP requests, sementara `check` berfungsi untuk memvalidasi response dari server, dan `sleep` untuk mensimulasikan jeda antara iterasi seperti perilaku pengguna nyata yang tidak terus-menerus mengklik tanpa henti.

Bagian `export const options` merupakan konfigurasi inti dari test scenario kita. Di sini kita mendefinisikan pola load test melalui stages yang terdiri dari tiga fase berbeda. Fase pertama adalah ramp-up selama 2 menit dimana virtual user secara bertahap dinaikkan hingga mencapai 10 user. Fase ini penting untuk mensimulasikan peningkatan traffic secara natural, bukan tiba-tiba. Kemudian fase kedua adalah maintenance phase selama 3 menit dimana kita mempertahankan 20 virtual user secara konsisten untuk menguji stability sistem under constant load. Terakhir, fase ramp-down selama 2 menit dimana virtual user secara perlahan diturunkan hingga nol, memberikan sistem kesempatan untuk recovery.

Thresholds yang kita tetapkan dalam konfigurasi ini bersifat realistis dan tidak terlalu ketat di tahap awal. Kita menetapkan batas maksimal error rate sebesar 5% yang berarti hanya 5 dari 100 requests yang boleh gagal. Untuk response time, kita menetapkan bahwa 95% requests harus selesai dalam waktu kurang dari 5 detik. Nilai-nilai threshold ini memang sengaja dibuat longgar di awal karena tujuan kita adalah memahami baseline performance terlebih dahulu sebelum menetapkan standar yang lebih ketat.

Fungsi utama `export default function()` merupakan jantung dari test script kita. Di dalamnya, kita mendefinisikan URL target yaitu website kf-lab.sahabatbelajar.com yang akan kita uji. Kemudian kita melakukan HTTP GET request ke URL tersebut dan menyimpan response-nya dalam variabel. Setelah mendapatkan response, kita melakukan serangkaian checks untuk memvalidasi bahwa response yang diterima memenuhi kriteria yang diharapkan.

Terdapat tiga checks yang kita implementasikan: pertama memeriksa apakah status code response adalah 200 (OK), kedua memastikan bahwa body response tidak kosong, dan ketiga memverifikasi bahwa content type yang dikembalikan adalah HTML. Checks ini penting untuk memastikan bahwa tidak hanya request berhasil dikirim, tetapi juga konten yang dikembalikan sesuai dengan ekspektasi. Terakhir, kita menambahkan sleep selama 1 detik untuk mensimulasikan think time pengguna nyata sebelum melakukan request berikutnya.

Pendekatan testing seperti ini memungkinkan kita untuk mengumpulkan data performance yang akurat dan meaningful. Dengan 20 virtual user, kita dapat mengamati bagaimana sistem berperilaku under moderate load tanpa harus mengeluarkan biaya untuk infrastructure yang mahal. Data yang dikumpulkan dari test ini akan menjadi baseline untuk pengujian selanjutnya. Setelah kita yakin sistem dapat handle load ini dengan baik, barulah kita bisa menurunkan threshold menjadi lebih ketat atau menambah jumlah virtual user untuk stress testing.

Jadi seperti itu lah implementasi performance testing dengan K6 yang realistic dan bertahap. Kita mulai dengan load yang manageable, threshold yang reasonable, dan validasi yang comprehensive. Dengan pendekatan ini, kita bisa secara sistematis mengidentifikasi potential bottlenecks dan memastikan kualitas layanan website sebelum deploy ke production environment.
