# Requirements - Nazwa Salwa Adellia | 2440304021

Kasus: Pengaduan Kerusakan Fasilitas Kampus | Constraint: persetujuan dua tingkat | P = 121

## Functional Requirement
### FR-01 - Pengajuan Laporan Kerusakan
- Aktor: Pelapor
Deskripsi: Pelapor mengisi formulir (fasilitas, lokasi, deskripsi kerusakan) dan mengirimkannya. Sistem memberi nomor tiket dan status DIAJUKAN.
Prioritas: High
Alasan: Titik awal seluruh proses dan masalah utama di A1 (tidak ada kanal baku dan nomor tiket).

### FR-02 - Verifikasi Laporan (Persetujuan Tingkat 1)
Aktor: Verifikator Sarpras
Deskripsi: Verifikator memeriksa kelengkapan dan keabsahan laporan, mendeteksi duplikat, menetapkan kategori dan prioritas, lalu memilih Terverifikasi atau Tolak (wajib beralasan).
Prioritas: High
Alasan: Menyaring laporan sebelum sampai ke pimpinan dan merupakan tingkat pertama constraint wajib.

### FR-03 - Persetujuan Penugasan (Persetujuan Tingkat 2)
Aktor: Kepala Bagian Sarpras
Deskripsi: Hanya laporan TERVERIFIKASI yang muncul di daftarnya. Kepala Bagian menyetujui atau menolak (wajib beralasan). Pengguna yang sama tidak boleh memberi persetujuan di kedua tingkat untuk laporan yang sama.
Prioritas: High
Alasan: Tingkat kedua constraint wajib dan pengendali perbaikan berbiaya/prioritas tinggi.

### FR-04 - Penugasan Teknisi
Aktor: Verifikator Sarpras
Deskripsi: Untuk laporan DISETUJUI, verifikator memilih teknisi dan target penyelesaian. Status menjadi DITUGASKAN.
Prioritas: High
Alasan: Menghubungkan keputusan persetujuan dengan eksekusi di lapangan.

### FR-05 - Pembaruan Status Penanganan
Aktor: Teknisi
Deskripsi: Teknisi melihat tugasnya, mengubah status menjadi DIKERJAKAN atau SELESAI, dan mengisi catatan hasil perbaikan.
Prioritas: High
Alasan: Sumber data kemajuan dan hasil penanganan.

### FR-06 - Pelacakan Status dan Notifikasi
Aktor: Pelapor
Deskripsi: Pelapor melihat daftar dan detail laporannya beserta riwayat status. Setiap perubahan status menghasilkan notifikasi di dalam sistem.
Prioritas: Medium
Alasan: Menyelesaikan masalah di A1 (pelapor tidak bisa memantau status).

### FR-07 - Rekap Laporan
Aktor: Pimpinan, Kepala Bagian Sarpras
Deskripsi: Menampilkan rekap jumlah laporan per status, kategori, dan periode, serta rata-rata waktu penanganan.
Prioritas: Medium
Alasan: Memenuhi kebutuhan pimpinan akan data untuk perencanaan.

### FR-08 - Pengelolaan Akun dan Peran
Aktor: Admin Sistem
Deskripsi: Admin membuat dan menonaktifkan akun serta menetapkan peran (Pelapor, Verifikator, Kepala Bagian, Teknisi, Pimpinan).
Prioritas: Medium
Alasan: Prasyarat pembedaan hak akses pada dua tingkat persetujuan.

## Nonfunctional Requirement

Asumsi beban: P = 121 pengguna aktif bersamaan (concurrent active users). Dengan rata-rata jeda antaraksi sekitar 20 detik, beban normal sekitar 6 permintaan/detik, dan dengan faktor puncak 2x beban rancangan sekitar 12 permintaan/detik.

| ID | Kategori | Pernyataan Terukur | Alasan |
|---|---|---|---|
| NFR-01 | Performance | Pada 121 pengguna aktif bersamaan (±12 permintaan/detik), fungsi utama (FR-01 s.d. FR-06) merespons ≤ 3 detik untuk ≥ 95% permintaan, dan rekap (FR-07) ≤ 5 detik. | XX = 3 detik: batas wajar agar pelapor di perangkat bergerak tidak mengira sistem macet. Rekap lebih berat sehingga diberi 5 detik. |
| NFR-02 | Capacity dan Scalability | Sistem melayani minimal 121 sesi aktif bersamaan tanpa penurunan layanan, dan arsitektur dapat ditingkatkan ke 2× P (242) hanya dengan menambah sumber daya, tanpa mengubah desain. | Beban puncak (awal semester, ujian) dapat melebihi rata-rata. |
| NFR-03 | Security | (a) Akses berbasis peran: 100% endpoint FR-02 s.d. FR-08 menolak peran yang tidak berhak (HTTP 403). (b) Kata sandi disimpan ter-hash, komunikasi lewat HTTPS. (c) Sesi kedaluwarsa setelah 30 menit tidak aktif. | Verifikator dan Kepala Bagian memegang kewenangan keputusan sehingga akses harus dipisah.|
| NFR-04 | Usability | Pelapor baru dapat mengirim laporan ≤ 3 menit dengan ≤ 5 isian wajib tanpa pelatihan, diuji pada ≥ 5 responden dengan tingkat keberhasilan ≥ 80%. | Pelapor sangat beragam dan tidak terlatih. |
| NFR-05 | Reliability dan Availability | Ketersediaan ≥ 99% pada jam layanan (07.00–17.00 WITA, Senin–Jumat), yaitu toleransi gangguan ≤ ±2 jam per bulan; laporan yang sudah dikirim tidak boleh hilang (0 kehilangan data pada uji simpan). | Laporan bisa menyangkut keselamatan, tetapi sistem kampus tidak memerlukan 24/7. |
| NFR-06 | Constraint NIM: persetujuan dua tingkat | (a) 100% laporan wajib melalui Level 1 lalu Level 2 secara berurutan; uji: 0 transisi ke DISETUJUI tanpa catatan kedua tingkat. (b) Pengguna yang sama tidak boleh menyetujui kedua tingkat pada satu laporan. (c) Setiap keputusan mencatat siapa, kapan, dan alasan. (d) Dua pengguna yang memproses laporan yang sama bersamaan: hanya satu yang berhasil (optimistic locking). | Mencegah persetujuan dilompati atau dimanipulasi; poin (d) penting karena ada 121 pengguna bersamaan. |
