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

