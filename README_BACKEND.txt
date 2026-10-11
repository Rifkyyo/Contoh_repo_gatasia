GATASIA — backend awal + sinkronisasi data
==========================================

Yang ditambahkan
- backend/server.js: API HTTP Node.js tanpa dependensi eksternal.
- backend/data/gatasia-db.json: database JSON dibuat otomatis saat server berjalan.
- api-client.js: jembatan sinkronisasi profil lokal ke backend dan papan peringkat server.
- Endpoint API:
  GET  /api/health
  GET  /api/provinces
  GET  /api/profile/:playerId
  PUT  /api/profile/:playerId
  GET  /api/leaderboard?period=week|all
  POST /api/quiz/submit

CARA MENJALANKAN
1. Instal Node.js versi 18 atau lebih baru.
2. Ekstrak ZIP.
3. Buka terminal di folder gatasia_merge_build.
4. Jalankan: node backend/server.js
5. Buka http://localhost:3000 (jangan buka index.html dengan klik dua kali).
6. Periksa API melalui http://localhost:3000/api/health

PENTING
- Situs tetap dapat dibuka sebagai demo statis, tetapi sinkronisasi backend hanya aktif saat dilayani melalui server ini.
- api-client.js disisipkan ke halaman yang memakai profil/papan peringkat. Data lokal yang sudah ada akan dikirim ke server ketika halaman dibuka atau profil disimpan.
- Ini adalah fondasi backend prototipe, bukan layanan produksi multi-pengguna. playerId di browser bukan autentikasi dan dapat dipalsukan; jangan gunakan untuk data siswa sungguhan tanpa sistem login.
- Endpoint PUT /api/profile menerima ringkasan profil dari browser. Untuk produksi, XP, lencana, dan statistik harus dihitung/diotorisasi server, bukan dipercaya dari klien.
- Endpoint POST /api/quiz/submit memvalidasi jawaban berdasarkan province_data.js dan mencegah klaim kuis provinsi yang sama lebih dari sekali per hari per playerId. Saat ini aturan ini merupakan contoh API; halaman kuis lama belum dialihkan sepenuhnya ke endpoint submit tersebut.
- Database JSON cocok untuk demo lokal atau prototipe satu server. Untuk pemakaian sekolah/produksi, pindahkan ke PostgreSQL/Supabase, tambahkan autentikasi, otorisasi, rate limiting, validasi skema, backup, dan HTTPS.
- Papan peringkat mingguan memakai event XP yang tercatat di backend. Sinkronisasi profil lama mencatat kenaikan XP sebagai event; karena profil demo mungkin berisi XP awal, data demo sebaiknya dibersihkan sebelum uji coba nyata.


PERBAIKAN UI (build ini):
- Level dan XP navbar membaca profil lokal yang sama di seluruh halaman.
- Perhitungan level konsisten: Level = floor(XP / 500) + 1.
- Progress XP menunjukkan progres menuju level berikutnya (rentang 500 XP).
- Popup XP pada Misi Harian dipaksa tetap kompak/horizontal agar tidak menutupi halaman.
Catatan: halaman masih menggunakan penyimpanan lokal untuk pengalaman pengguna; sinkronisasi server prototipe tidak menyediakan autentikasi akun sungguhan.


PERBAIKAN SINKRONISASI PROFIL (build terbaru)
- Saat halaman dimuat, api-client.js lebih dulu meminta profil ke server.
- Jika profil sudah ada, data server dipakai untuk menyelaraskan XP, nama, avatar, statistik, lencana, dan riwayat ke localStorage.
- Jika profil belum ada, data lokal yang tersedia dipakai sebagai data awal lalu dikirim ke server.
- Perubahan localStorage pada gatasiaProfile dikirim ke server dengan debounce singkat.
- Navbar dirender ulang dari satu profil yang sama dan papan peringkat membaca data server.
- Pastikan website dibuka melalui http://localhost:3000, bukan file://.
- Untuk menguji sinkronisasi dari server yang benar-benar bersih, hentikan server, cadangkan lalu hapus backend/data/gatasia-db.json (ini akan menghapus profil demo tersimpan), kemudian jalankan ulang server.
- Profil dikenali dengan gatasiaPlayerId yang tersimpan di browser. Menghapus data situs atau memakai browser/perangkat lain membuat ID berbeda. Login multi-perangkat membutuhkan autentikasi akun yang belum termasuk dalam prototipe ini.
- Sinkronisasi profil PUT masih cocok untuk demo saja: klien masih dapat mengirim nilai XP. Untuk produksi, hadiah XP perlu dicatat dan divalidasi oleh server.
