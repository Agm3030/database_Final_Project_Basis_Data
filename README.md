# database_Final_Project_Basis_Data
Database db_hiling_semata merupakan sistem basis data relasional berbasis MariaDB/MySQL yang dikembangkan khusus untuk mendukung portal pariwisata terintegrasi dan layanan pemesanan open trip. Sistem ini dirancang untuk menyimpan berbagai data penting, mulai dari informasi destinasi wisata, agenda acara, artikel, warisan budaya lokal, rekam jejak interaksi pengguna (seperti ulasan dan daftar favorit), hingga pencatatan transaksi perjalanan.

Daftar Isi :
1.Gambaran Umum
2.Arsitektur Tabel Utama
3.Sistem Otomatisasi (Procedures, Functions, Triggers)
4.Alur Transaksi Pemesanan

Arsitektur Tabel Utama
Struktur data dalam sistem ini dibagi ke dalam beberapa modul fungsional:

1.Manajemen Transaksi & Pemesanan
> open_trip: Tabel ini berfungsi sebagai katalog paket perjalanan yang ditawarkan. Data yang disimpan mencakup jadwal keberangkatan, tarif, batas maksimal
> peserta, sisa kuota, serta status ketersediaan trip (seperti tersedia, penuh, atau telah selesai).
> booking: Berfungsi untuk merekam setiap transaksi pendaftaran open trip oleh pengguna. Informasi yang dicatat meliputi jumlah peserta, total biaya, dan status     pembayaran (misalnya pending, dibayar, atau dibatalkan).
> payment_log: Tabel ini bertugas menyimpan rekam jejak pembayaran, termasuk nominal transaksi dan metode yang digunakan (seperti transfer bank, dompet digital,     atau tunai).

2.Pariwisata & Warisan Budaya
> budaya: Merupakan ensiklopedia lokal yang menyimpan informasi terkait seni dan budaya daerah (contoh: Tari Serimpi, Gamelan Jawa, Gudeg). Data yang dicatat         meliputi asal daerah, deskripsi lengkap, nilai historis, dan dokumentasi visual.
> galeri: Berperan sebagai repositori media terpusat untuk menyimpan gambar-gambar yang berkaitan dengan entitas wisata, budaya, maupun event. Tabel ini juga         mendukung pengaturan urutan tampilan gambar.
> kategori: Berfungsi sebagai tabel referensi utama untuk mengklasifikasikan berbagai entitas ke dalam kelompok yang spesifik, seperti kategori Wisata, Budaya,       atau Artikel.

3.Publikasi & Informasi
> artikel: Tabel ini mengelola berbagai konten editorial seperti berita, blog, atau panduan wisata (contoh: "5 Tips Wisata Hemat"). Informasi yang disimpan           mencakup data penulis, slug URL, dan status publikasi (Publish atau Draft).
> event: Digunakan untuk mencatat jadwal dan detail acara yang diselenggarakan di sekitar kawasan wisata, seperti Prambanan Jazz Festival.
> kontak: Menyimpan informasi profil platform, termasuk alamat fisik, nomor kontak, tautan media sosial, dan jam operasional layanan pelanggan.

4.Interaksi & Aktivitas Pengguna
> review: Tabel ini menampung ulasan dan penilaian dari pengunjung yang telah mendatangi suatu destinasi. Data yang disimpan berupa rating dengan skala 1-5           beserta komentar tertulis.
> favorite: Menyediakan fitur wishlist yang memungkinkan pengguna untuk menyimpan dan menandai destinasi wisata yang ingin mereka kunjungi di masa mendatang.

5.Keamanan & Log Audit
> log_aktivitas: Berfungsi sebagai sistem pemantauan otomatis yang mencatat setiap peristiwa penting dalam sistem, seperti waktu pembuatan pesanan booking baru.

Sistem Otomatisasi (Logic Database)
Selain menyimpan data, database ini juga mengimplementasikan aturan bisnis secara otomatis melalui fitur bawaan MySQL/MariaDB:

Stored Procedures
1.sp_booking_baru(p_id_user, p_id_trip)
Prosedur ini dirancang untuk memproses transaksi booking secara aman. Sistem akan memverifikasi ketersediaan kuota pada tabel open_trip terlebih dahulu. Jika kuota mencukupi, prosedur akan membuat catatan booking baru sekaligus mengurangi sisa kapasitas. Jika kuota tidak tersedia, transaksi akan otomatis ditolak.

2.sp_klasifikasi_rating_wisata()
Merupakan fungsi yang berjalan secara berkala menggunakan cursor loop untuk menghitung rata-rata rating setiap destinasi wisata. Hasil perhitungan kemudian diklasifikasikan ke dalam kategori seperti "Sangat Baik" (>= 4.5), "Baik", "Cukup", atau "Kurang", dan disimpan dalam tabel audit wisata_status_log.

Functions
1.fn_hitung_harga_diskon(p_harga, p_persen_diskon)
Fungsi utilitas yang digunakan untuk mengkalkulasi harga akhir suatu paket perjalanan setelah dikurangi persentase diskon tertentu.
2.fn_jumlah_wisata_aktif()
Fungsi ini bertugas menghitung total destinasi wisata yang berstatus 'aktif' secara real-time, yang nantinya akan ditampilkan pada halaman dashboard.

Triggers
1.trg_after_insert_booking
Trigger ini akan dieksekusi secara otomatis setelah ada data baru yang ditambahkan ke dalam tabel booking. Trigger ini menjalankan dua tugas penting:
-Merekam detail aktivitas ke dalam tabel log_aktivitas untuk keperluan audit.
-Memperbarui nilai pada kolom kapasitas_tersisa di tabel open_trip secara real-time guna mencegah terjadinya overbooking.

Alur Transaksi Pemesanan
1.Administrator sistem memasukkan jadwal keberangkatan baru ke dalam tabel open_trip (Contoh: "Trip Sunrise Candi Prambanan").
2.Ketika pengguna melakukan pemesanan, sistem akan memanggil prosedur sp_booking_baru dengan mengirimkan data jumlah peserta.
3.Prosedur akan memvalidasi ketersediaan kuota. Jika validasi berhasil, data transaksi akan disimpan ke tabel booking. Selanjutnya, Trigger otomatis akan bekerja untuk mengurangi sisa kuota dan mencatat aktivitas tersebut ke dalam log.
4.Setelah pengguna menyelesaikan pembayaran, sistem akan mencatat detail transaksi pada tabel payment_log dan memperbarui status booking menjadi dibayar.

