# Requirements - Nazwa Salwa Adellia | 2440304021

Kasus: Pengaduan Kerusakan Fasilitas Kampus | Constraint: persetujuan dua tingkat | P = 121

## Functional Requirement
### FR-01 - Pengajuan Laporan Kerusakan
- Aktor: Pelapor
- Deskripsi: Pelapor mengisi formulir (fasilitas, lokasi, deskripsi kerusakan) dan mengirimkannya. Sistem memberi nomor tiket dan status DIAJUKAN.
- Prioritas: High
- Alasan: Titik awal seluruh proses dan masalah utama di A1 (tidak ada kanal baku dan nomor tiket).

### FR-02 - Verifikasi Laporan (Persetujuan Tingkat 1)
- Aktor: Verifikator Sarpras
- Deskripsi: Verifikator memeriksa kelengkapan dan keabsahan laporan, mendeteksi duplikat, menetapkan kategori dan prioritas, lalu memilih Terverifikasi atau Tolak (wajib beralasan).
- Prioritas: High
- Alasan: Menyaring laporan sebelum sampai ke pimpinan dan merupakan tingkat pertama constraint wajib.

### FR-03 - Persetujuan Penugasan (Persetujuan Tingkat 2)
- Aktor: Kepala Bagian Sarpras
- Deskripsi: Hanya laporan TERVERIFIKASI yang muncul di daftarnya. Kepala Bagian menyetujui atau menolak (wajib beralasan). Pengguna yang sama tidak boleh memberi persetujuan di kedua tingkat untuk laporan yang sama.
- Prioritas: High
- Alasan: Tingkat kedua constraint wajib dan pengendali perbaikan berbiaya/prioritas tinggi.

### FR-04 - Penugasan Teknisi
- Aktor: Verifikator Sarpras
- Deskripsi: Untuk laporan DISETUJUI, verifikator memilih teknisi dan target penyelesaian. Status menjadi DITUGASKAN.
- Prioritas: High
- Alasan: Menghubungkan keputusan persetujuan dengan eksekusi di lapangan.

### FR-05 - Pembaruan Status Penanganan
- Aktor: Teknisi
- Deskripsi: Teknisi melihat tugasnya, mengubah status menjadi DIKERJAKAN atau SELESAI, dan mengisi catatan hasil perbaikan.
- Prioritas: High
- Alasan: Sumber data kemajuan dan hasil penanganan.

### FR-06 - Pelacakan Status dan Notifikasi
- Aktor: Pelapor
- Deskripsi: Pelapor melihat daftar dan detail laporannya beserta riwayat status. Setiap perubahan status menghasilkan notifikasi di dalam sistem.
- Prioritas: Medium
- Alasan: Menyelesaikan masalah di A1 (pelapor tidak bisa memantau status).

### FR-07 - Rekap Laporan
- Aktor: Pimpinan, Kepala Bagian Sarpras
- Deskripsi: Menampilkan rekap jumlah laporan per status, kategori, dan periode, serta rata-rata waktu penanganan.
- Prioritas: Medium
- Alasan: Memenuhi kebutuhan pimpinan akan data untuk perencanaan.

### FR-08 - Pengelolaan Akun dan Peran
- Aktor: Admin Sistem
- Deskripsi: Admin membuat dan menonaktifkan akun serta menetapkan peran (Pelapor, Verifikator, Kepala Bagian, Teknisi, Pimpinan).
- Prioritas: Medium
- Alasan: Prasyarat pembedaan hak akses pada dua tingkat persetujuan.

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

## User Story dan Acceptance Criteria
### US-01 (terkait FR-01, FR-06)
Sebagai Pelapor (mahasiswa/dosen/tendik), saya ingin mengirim laporan kerusakan fasilitas dan memantau progresnya, sehingga saya bisa tahu apakah laporan sudah ditangani tanpa harus bertanya ke petugas Sarpras.
Acceptance Criteria:
1.	Given saya sudah login, when saya mengisi data fasilitas, lokasi, deskripsi, lalu menekan tombol Kirim, then sistem menyimpan laporan, menerbitkan nomor tiket, dan memberi status DIAJUKAN.
2.	Given ada kolom wajib yang belum diisi, when saya menekan tombol Kirim, then laporan gagal dikirim dan sistem memberi tahu kolom mana yang harus dilengkapi.
3.	Given laporan saya sudah terkirim, when status laporan berubah, then sistem mengirimkan notifikasi dan mencatat riwayat perubahan statusnya.
4.	Given ada 121 pengguna aktif bersamaan, when saya mengirim laporan, then konfirmasi berhasil muncul dalam waktu maksimal 3 detik.

### US-02 (terkait FR-03, NFR-06)
Sebagai Kepala Bagian Sarpras, saya ingin menyetujui atau menolak laporan yang sudah diverifikasi, sehingga hanya perbaikan yang valid dan jelas alasannya yang akan diteruskan ke teknisi. 
Acceptance Criteria:
1.	Given saya login sebagai Kepala Bagian Sarpras, when saya membuka menu persetujuan, then sistem hanya menampilkan daftar laporan yang berstatus TERVERIFIKASI. 
2.	Given saya akan menolak suatu laporan, when saya mengosongkan kolom alasan penolakan, then sistem menolak menyimpan keputusan dan meminta saya mengisi alasannya.
3.	Given saya menyetujui laporan, then status laporan berubah menjadi DISETUJUI, data keputusan tersimpan (nama, waktu, catatan), dan notifikasi otomatis dikirim ke pelapor serta verifikator.
4.	Given akun saya adalah akun yang sama dengan verifikator laporan tersebut, when saya mencoba menyetujuinya, then sistem menolak tindakan tersebut demi mencegah rangkap jabatan.
5.	Given laporan tersebut sudah diproses oleh pengguna lain, when saya mencoba memprosesnya lagi, then sistem menampilkan pesan bentrok data dan status laporan tidak akan berubah.

## Requirements Traceability

| Requirement | Stakegolder | Use Case | Class Terkait |
|---|---|---|---|
| FR-01 Pengajuan Laporan |	Pelapor | UC-01 | Pengguna, Laporan, Fasilitas |
| FR-02 Verifikasi (Level 1) | Verifikator Sarpras | UC-03 | Laporan, Verifikasi, Pengguna |
| FR-03 Persetujuan (Level 2) | Kepala Bagian Sarpras | UC-04 | Laporan, Persetujuan, Pengguna |
| FR-04 Penugasan Teknisi |	Verifikator Sarpras, Teknisi | UC-05 | Penugasan, Teknisi, Laporan |
| FR-05 Pembaruan Status | Teknisi | UC-06 | Penugasan, Teknisi, RiwayatStatus |
| FR-06 Pelacakan dan Notifikasi | Pelapor | UC-02 | Laporan, RiwayatStatus, Notifikasi |
| FR-07 Rekap |	Pimpinan, Kepala Bagian | UC-07	| Laporan, Fasilitas, RiwayatStatus |
| FR-08 Kelola Akun dan Peran |	Admin Sistem | UC-08 | Pengguna, Peran |
| NFR-06 | Dua tingkat | Verifikator, Kepala Bagian | UC-03, UC-04 | Verifikasi, Persetujuan, RiwayatStatus | 
