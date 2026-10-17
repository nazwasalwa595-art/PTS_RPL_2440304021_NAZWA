# Architecture - Nazwa Salwa Adellia | 2440304021

## Arsitektur: Layered
![Gambar 4. Arsitektur Layered](diagram/arsitektur-layered-Nazwa.png)

### Alasan memilih Layered Architecture
1.	Ruang lingkup sistem sudah jelas (mulai dari pelaporan hingga rekapitulasi) dengan beban P = 121 pengguna aktif. Layered Architecture sudah sangat mencukupi tanpa perlu kerumitan sistem terdistribusi.  
2.	Aturan persetujuan dua tingkat dipusatkan penuh di lapisan Business Logic. Hal ini menjamin aturan tidak dapat dimanipulasi atau dilompati, baik melalui antarmuka web maupun API luar.
3.	Masing-masing lapisan terisolasi (sesuai prinsip Dependency Inversion). Jika ada perubahan pada tampilan (UI) atau jenis basis data, logika bisnis inti tidak akan ikut rusak.   

### Fungsi tiap lapisan
| Lapisan | Fungsi | Komponen Utama |
|---|---|---|
| Client |	Menampilkan antarmuka dan mengirim permintaan |	Browser web, mobile web |
| Application |	Menerima permintaan, autentikasi, otorisasi berbasis peran, validasi input | Controller, RBAC, validator |
| Business Logic | Aturan bisnis: mesin status, dua tingkat persetujuan, pemisahan tugas, penerbitan notifikasi | Service, Observer |
| Data Access |	Akses data lewat repository, cache data rujukan dan rekap |	Repository, cache |
| Database | Penyimpanan permanen |	Basis data relasional |


### Aliran data utama
- Aliran 1: Aliran data pengajuan laporan baru diawali ketika pelapor mengisi formulir aduan pada antarmuka aplikasi. Permintaan ini diterima oleh Application Layer untuk diperiksa masa sesi login, otorisasi peran pengguna, serta kelengkapan isian formulir. Setelah terverifikasi valid, LaporanService di Business Logic Layer membuat entitas laporan baru bernomor tiket resmi dengan status awal DIAJUKAN, lalu meneruskannya ke Data Access Layer agar disimpan oleh Repository ke dalam basis data relasional. Proses ini kemudian secara otomatis memicu RiwayatObserver dan NotifikasiObserver untuk merekam jejak status serta mengirimkan konfirmasi nomor tiket kembali ke pelapor.   
- Aliran 2: Aliran data persetujuan Level 2 dimulai saat Kepala Bagian Sarpras memilih keputusan untuk menyetujui laporan pada dashboard. Application Layer terlebih dahulu memastikan bahwa pengguna terautentikasi memiliki peran KEPALA_SARPRAS. Selanjutnya, PersetujuanService memvalidasi tiga syarat utama: memastikan status laporan masih TERVERIFIKASI (lolos Level 1), memastikan pengguna saat ini bukan verifikator laporan tersebut demi mencegah rangkap jabatan, serta mengecek versi laporan guna menghindari bentrok data (optimistic locking). Jika seluruh syarat terpenuhi, status laporan diperbarui menjadi DISETUJUI, data keputusan tersimpan ke basis data, dan Observer dipicu untuk mengirimkan notifikasi perubahan status kepada pelapor dan verifikator.

### Cara menangani P = 121 pengguna aktif
Asumsi beban (konsisten dengan B2): beban normal ≈ 6 permintaan/detik, beban rancangan puncak ≈ 12 permintaan/detik.

| Aspek | Strategi | Argumentasi untuk P = 121  |
|---|---|---|
| Kapasitas aplikasi | Satu instance aplikasi dengan kumpulan thread terbatas (misalnya 8–16 worker); aplikasi dibuat stateless | Bila rata-rata pemrosesan ±200 ms, satu worker menangani ±5 permintaan/detik, sehingga 8 worker ≈ 40 permintaan/detik, jauh di atas 12. Angka ini asumsi rancangan, bukan hasil uji. |
| Koneksi basis data | Connection pool ukuran ±10–20 | Cukup untuk 121 sesi karena sebagian besar sesi menganggur di antara aksi. |
| Kueri | Indeks pada status, tanggal_lapor, id_pelapor; daftar dipaginasi (misalnya 20 baris) | Menjaga daftar laporan tetap cepat meski data bertambah. |
| Cache | Data rujukan (fasilitas, kategori) dan hasil rekap di-cache ±60 detik; dibatalkan oleh CacheRekapObserver saat status berubah | Rekap (FR-07) adalah kueri agregat paling berat, dan batas respons 5 detik di NFR-01 bisa dipenuhi. |
| Skalabilitas ke 2P (242) | Karena aplikasi stateless, tambah instance di belakang load balancer; basis data tetap satu | Memenuhi NFR-02 tanpa mengubah desain. Load balancer belum wajib pada P = 121, hanya disiapkan. |
| Konkurensi | Optimistic locking (versi di Laporan) | Dua orang memproses laporan yang sama: satu berhasil, satu mendapat pesan konflik. |
		
### Cara menangani constraint wajib (persetujuan dua tingkat)
| Kebutuhan NFR-06 | Mekanisme dalam arsitektur |
|---|---|
| Urutan wajib Level 1 → Level 2 | Mesin status di LaporanService: hanya transisi DIAJUKAN → TERVERIFIKASI → DISETUJUI yang diizinkan; transisi lain ditolak. |
| Hak akses per tingkat | Lapisan Application (RBAC) memeriksa peran sebelum permintaan menyentuh Business Logic; pemeriksaan ganda di PersetujuanService. |
| Pemisahan tugas | PersetujuanService menolak jika idPengguna sama dengan pemberi verifikasi laporan itu. |
| Jejak keputusan | Class Verifikasi, Persetujuan, dan RiwayatStatus menyimpan siapa, kapan, dan alasan. |
| Keputusan serentak | versi pada Laporan (optimistic locking). |

## Design Pattern: Observer
1.	Masalah Desain
Setiap kali status laporan berubah, banyak modul lain yang harus diproses (seperti mencatat RiwayatStatus, mengirim Notifikasi, dan memperbarui cache rekap). Jika kelas Laporan memanggil semua modul ini secara langsung, struktur kode akan menjadi sangat terikat (tight coupling) dan sulit dikembangkan saat ada penambahan fitur baru. 
2.	Pattern yang Dipilih
Observer Pattern (Pertemuan 6). Objek utama (Subject/Laporan) menyimpan daftar Observer dan otomatis memberi tahu seluruh Observer saat terjadi perubahan status tanpa perlu tahu detail cara kerja mereka (loose coupling). 
3.	Penerapan pada Sistem
Diterapkan pada eksekusi fungsi LaporanService.ubahStatus() di lapisan Business Logic. Observer yang terdaftar meliputi: RiwayatObserver, NotifikasiObserver, dan CacheRekapObserver. 
4.	Alasan Memilih Observer Dibanding Alternatif (Strategy)
Pola Strategy digunakan untuk memilih salah satu algoritma dari beberapa pilihan. Sedangkan kebutuhan sistem di sini adalah menyiarkan satu kejadian (event) perubahan status ke banyak modul penerima sekaligus secara dinamis (sesuai prinsip Open/Closed Principle). 

![Gambar 5. Class Diagram Penerapan Observer](diagram/Class-Diagram-Penerapan-Observer-Nazwa.png)

## Trade-off Keputusan Desain
Menggunakan satu aplikasi monolit berlapis dengan satu basis data, bukan memecah menjadi layanan-layanan terpisah (misalnya layanan pelaporan, persetujuan, dan notifikasi).

Keuntungan:
1.	Sederhana untuk dibangun, diuji, dan dikelola oleh tim kecil.
2.	Seluruh perubahan status, riwayat, dan persetujuan dapat berada dalam satu transaksi basis data, sehingga aturan dua tingkat dan penelusuran jejak tetap konsisten.
3.	Cukup untuk P = 121 (± 12 permintaan/detik) tanpa biaya infrastruktur tambahan.

Kelemahan:
1.	Seluruh komponen dirilis bersama; perubahan kecil pada notifikasi tetap mengharuskan rilis ulang seluruh aplikasi.
2.	Satu aplikasi dan satu basis data adalah titik kegagalan tunggal, sehingga ketersediaan (NFR-05) bergantung pada satu lokasi.
3.	Jika beban tumbuh jauh melampaui 2P, skala vertikal atau pemecahan layanan akan diperlukan dan memerlukan penataan ulang.

Alasan tetap dipilih karena kasus pengaduan fasilitas kampus berskala kecil dengan ruang lingkup yang sudah terdefinisi jelas, serta alur persetujuan dua tingkat membutuhkan konsistensi transaksi data yang paling efektif dijamin dalam satu basis data terpusat. Walaupun memiliki konsekuensi berupa potensi titik kegagalan tunggal (single point of failure) dan keharusan rilis aplikasi secara menyeluruh, kelemahan ini berhasil dimitigasi tanpa mengubah arsitektur dasar, yaitu dengan memisahkan tiap lapisan secara ketat agar siap ditransisikan ke microservices di masa depan, membuat aplikasi bersifat stateless untuk mempermudah penambahan kapasitas (scaling) hingga menangani beban puncak 2 x P (242 pengguna aktif), serta memastikan ketersediaan sistem sebesar 99\% pada jam kerja kampus yang sudah sangat mencukupi kebutuhan operasional.

## Aturan Persetujuan Dua Tingkat (NFR-06)
1. Alur Wajib Berurutan: Setiap laporan harus diperiksa dan diverifikasi dulu oleh Verifikator Sarpras (Level 1), baru kemudian diteruskan ke Kepala Bagian Sarpras (Level 2) untuk disetujui.   
2. Cegah Rangkap Jabatan (Dual Control): Pengguna yang sama dilarang keras memproses/menyetujui laporan pada kedua tingkat tersebut demi menjaga independensi keputusan.  
3. Pencatatan Rekam Jejak (Audit Log): Setiap keputusan (baik verifikasi, persetujuan, maupun penolakan) harus otomatis mencatat identitas pemroses, timestamp waktu, dan catatan alasannya.   
4. Wajib Alasan Penolakan: Jika laporan ditolak di Level 1 maupun Level 2, sistem tidak akan menyimpan keputusan sampai kolom alasan penolakan diisi.   
5. Penanganan Akses Serentak (Optimistic Locking): Jika ada dua pengguna yang memproses laporan yang sama di detik yang bersamaan, sistem memanfaatkan kolom versi untuk memastikan hanya satu perubahan yang berhasil disimpan.