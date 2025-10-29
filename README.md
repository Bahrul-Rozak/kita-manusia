# Performance Testing dengan K6: Skenario Akses Halaman Web

Halo teman-teman, pada kesempatan hari ini kita akan membahas tentang implementasi performance testing menggunakan K6 untuk menguji performa dari sebuah halaman web. Latar belakang dari skenario testing yang akan kita jalankan adalah untuk memvalidasi kemampuan server dalam menangani traffic yang realistis pada tahap awal pengembangan. Kita akan melakukan testing terhadap halaman https://kf-lab.sahabatbelajar.com/ dengan konfigurasi yang wajar mengingat kita menggunakan akun cloud K6 gratis, sehingga kita batasi virtual users sebanyak 20 saja. Pendekatan ini sangat masuk akal karena dalam performance testing, kita tidak perlu langsung menargetkan beban yang tinggi, melainkan mulai dari beban yang realistis dan kemudian secara bertahap meningkatkan kompleksitas testing setelah sistem menunjukkan kemampuan yang memadai.

## File: `kf-lab-performance-test.js`

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  scenarios: {
    kf_lab_load_test: {
      executor: 'constant-vus',
      vus: 20,
      duration: '5m',
      gracefulStop: '0s',
    },
  },
  thresholds: {
    http_req_failed: ['rate<0.05'],
    http_req_duration: ['p(95)<2000'],
    http_reqs: ['count>1000'],
  },
  cloud: {
    distribution: {
      'amazon:us:ashburn': { loadZone: 'amazon:us:ashburn', percent: 100 },
    },
  },
};

export default function () {
  const response = http.get('https://kf-lab.sahabatbelajar.com/');
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response body is not empty': (r) => r.body.length > 0,
    'response time is acceptable': (r) => r.timings.duration < 3000,
  });
  
  sleep(1);
}
```

## Penjelasan Detail Step by Step

Mari kita bahas secara mendetail setiap bagian dari kode performance testing yang telah kita buat. Penjelasan akan dimulai dari struktur file dan kemudian berlanjut ke setiap komponen penting dalam skrip testing kita.

Pertama-tama, kita perlu memahami bahwa file yang kita buat bernama `kf-lab-performance-test.js`. Pemilihan nama ini cukup straightforward karena mencerminkan tujuan dari skrip tersebut, yaitu melakukan performance test terhadap website KF Lab. Dalam dunia testing, penamaan yang jelas dan deskriptif sangat penting untuk memudahkan maintenance dan kolaborasi antar tim developer.

Pada bagian paling atas kode, kita mengimpor modul-modul yang diperlukan dari K6. Modul `http` digunakan untuk melakukan HTTP requests, sementara `check` dan `sleep` adalah fungsi utilitas yang membantu dalam validasi response dan mengatur jeda antara eksekusi. Import statement ini merupakan fondasi dasar dari setiap skrip K6 karena tanpa modul-modul ini, kita tidak dapat melakukan testing yang efektif.

Bagian `options` adalah jantung dari konfigurasi test kita. Di dalamnya, kita mendefinisikan skenario testing yang bernama `kf_lab_load_test`. Skenario ini menggunakan executor `constant-vus` yang berarti kita akan mempertahankan jumlah virtual users yang konstan selama test berjalan. Kita menetapkan 20 virtual users dengan durasi test selama 5 menit. Pemilihan 20 virtual users ini didasarkan pada pertimbangan praktis mengingat kita menggunakan akun cloud K6 gratis, sehingga kita perlu membatasi resource yang digunakan. Durasi 5 menit dipilih karena memberikan waktu yang cukup untuk mengamati perilaku sistem under test tanpa menghabiskan resource yang berlebihan.

Thresholds atau batasan yang kita tetapkan dalam test ini dirancang untuk tidak terlalu ketat di tahap awal. Kita menetapkan tiga threshold utama: pertama, tingkat kegagalan HTTP request harus kurang dari 5%, yang berarti hanya 5 dari 100 requests yang diperbolehkan gagal. Kedua, 95 persentil dari durasi request harus di bawah 2000 milidetik atau 2 detik. Ketiga, total requests yang berhasil harus lebih dari 1000 selama test berjalan. Threshold ini realistis untuk tahap awal testing dan dapat disesuaikan kemudian berdasarkan hasil yang diperoleh.

Konfigurasi cloud distribution menunjukkan bahwa kita akan menjalankan test dari zona Amazon US Ashburn dengan persentase 100%. Ini berarti semua virtual users akan dijalankan dari data center tersebut, yang memberikan konsistensi dalam pengukuran latency dan performance.

Fungsi utama `default function` adalah tempat dimana aksi testing sebenarnya terjadi. Setiap virtual user akan menjalankan fungsi ini berulang-ulang selama test berlangsung. Di dalam fungsi ini, kita melakukan HTTP GET request ke URL target https://kf-lab.sahabatbelajar.com/. Request ini mensimulasikan pengguna yang mengakses halaman web tersebut.

Setelah menerima response dari server, kita melakukan serangkaian checks untuk memvalidasi bahwa response tersebut memenuhi kriteria yang diharapkan. Check pertama memverifikasi bahwa status code response adalah 200, yang menandakan request berhasil. Check kedua memastikan bahwa body response tidak kosong, mengindikasikan bahwa konten yang diharapkan benar-benar terkirim. Check ketiga memvalidasi bahwa waktu response masih dalam batas acceptable, yaitu di bawah 3000 milidetik.

Setelah melakukan request dan validasi, kita menambahkan sleep selama 1 detik. Ini mensimulasikan perilaku pengguna nyata yang tidak terus-menerus mengirim request, melainkan ada jeda antara interaksi. Think time seperti ini penting untuk menciptakan skenario testing yang realistis dan tidak membebani server dengan request yang beruntun tanpa jeda.

Pendekatan testing seperti yang kita terapkan hari ini sangat sesuai untuk tahap awal pengembangan sistem. Dengan memulai dari beban yang ringan dan threshold yang realistis, kita dapat mengidentifikasi masalah-masalah dasar tanpa harus menghadapi kompleksitas testing beban tinggi. Setelah sistem lolos test dengan konfigurasi ini, kita dapat secara bertahap meningkatkan intensitas test dengan menambah jumlah virtual users, memperketat threshold, atau menambah durasi test.

Jadi seperti itu lah penjelasan detail mengenai implementasi performance testing dengan K6 yang kita buat hari ini. Dari pembahasan di atas, dapat kita pahami bahwa performance testing adalah proses iteratif yang dimulai dari skenario sederhana kemudian berkembang menjadi lebih kompleks seiring dengan peningkatan kapabilitas sistem yang diuji.
