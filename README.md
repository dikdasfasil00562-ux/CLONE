# Test Items Clone — versi DPLY

Aplikasi soal paralel Bahasa Inggris oleh **Suryo Agung Nugroho**. Pembaruan 28 September 2026.

## Memasang / memperbarui Netlify

Paket ini berisi kode siap-deploy, bukan situs yang sudah online. Pemrosesan memerlukan API key aktif. Jangan menaruh kunci di frontend atau repository publik.

1. Ekstrak ZIP. Isi folder `test-items-clone` menjadi akar repository: `public/`, `netlify/`, `netlify.toml`, `package.json`.
2. Import repository ke Netlify, atau perbarui repository proyek yang sudah digunakan.
3. Build command kosong, publish directory `public`, functions directory `netlify/functions`. Konfigurasi tersedia di `netlify.toml`.
4. Isi environment di pengaturan proyek, scope **Functions**, konteks **Production**. Isi konteks Deploy Preview juga jika digunakan.
5. Deploy ulang **seluruh paket**, termasuk endpoint baru `review` dan `passage`.
6. Muat ulang halaman, unggah sumber, dan lakukan telaah baru. Jangan meneruskan tab dari versi lama.

| Variabel | Isi |
| --- | --- |
| `GEMINI_API_KEY_1` | Kunci Gemini pertama |
| `GEMINI_API_KEY_2` | Kunci Gemini kedua |
| `GROQ_API_KEY` | Kunci Groq |
| `GEMINI_MODEL` | Opsional; default `gemini-3.5-flash` |
| `GROQ_MODEL` | Opsional; default `qwen/qwen3.8-27b` |

Dua kunci Gemini dari proyek yang sama berbagi batas proyek. Menambah kunci tidak menambah kuota. Pilih model yang tersedia pada akun dan mendukung vision serta keluaran JSON.

Alternatif Netlify CLI (Node.js 22+), jalankan dari folder proyek:

```sh
npx netlify-cli login
npx netlify-cli deploy --prod --dir=public --functions=netlify/functions
```

Untuk lokal gunakan `npx netlify-cli dev`. Jangan hanya menyeret `public/` ke Netlify Drop: deployment statis saja tidak memasang functions. Tidak ada bundler frontend; `public/index.html` memuat CSS/JS inline dan memuat pustaka dokumen dari CDN saat diperlukan.

## Alur penggunaan

### 1. Sumber

- Unggah Word/PDF atau tempel teks.
- Berkas diekstrak untuk pratinjau sebelum analisis; tersedia **Lihat seluruh sumber** dan **Ganti berkas**.
- PDF pindai ditampilkan sebagai gambar sumber, bukan diperlakukan seolah sudah menjadi teks.
- Pengingat **1 halaman, maksimal 10 soal** muncul sekali per sesi tab; pengingat ringkas tetap ada dan bisa dibuka ulang.
- Pratinjau Word menampilkan hasil ekstraksi teks, bukan reproduksi layout visual.

### 2. Telaah dan koreksi

- Telaah keseluruhan: jumlah soal dan bahasa.
- Tabel bacaan: Teks 1/2/…, genre, CEFR bacaan, jumlah kata, nomor soal terkait; tombol melihat bacaan sumber.
- Tabel pola per nomor: genre, jenis soal, kesulitan, Barrett, CEFR soal, indikator kisi-kisi. Header tetap terlihat saat tabel digulir; penjelasan panjang dapat dibuka.
- **Koreksi hasil telaah / kisi-kisi** memungkinkan guru mengubah indikator, jenis teks/soal, kesulitan, Barrett, CEFR, pola, dan jumlah opsi. Koreksi menjadi acuan pembuatan. Tombol simpan memperbarui tabel; tombol lanjut juga menggunakan nilai koreksi terbaru yang valid.
- Catatan kelemahan sumber ditampilkan bila ditemukan. Kisi-kisi adalah indikator hasil inferensi, bukan klaim kode CP/KD resmi. Kesulitan dan CEFR tetap perkiraan. Barrett ditandai tidak relevan bila tidak mengukur pemahaman bacaan.

### 3. Pembuatan, pemeriksaan, suntingan

- Pembuatan mengelompokkan soal menurut bacaan, bukan sekadar memotong nomor berurutan. Soal mandiri memiliki kelompok sendiri.
- Kelompok hingga enam soal dibuat bersama. Untuk kelompok lebih panjang, bacaan baru dibuat dan ditetapkan lebih dulu, kemudian soal dibuat bertahap dengan bacaan tetap.
- Jika seluruh percobaan gagal karena hasil terputus/tidak valid atau timeout, kelompok dipecah menjadi dua lalu satu soal. Pembuatan bacaan awal tetap mempertimbangkan seluruh indikator dalam kelompok.
- Setelah kelompok selesai, sistem memanggil pemeriksa melalui permintaan terpisah. **Kunci awal dan soal sumber tidak dimasukkan ke prompt pemeriksa.** Pemeriksa menjawab dari soal baru, memberikan dasar jawaban, dan menilai keterjawaban, indikator serta pengecoh.
- Server membandingkan jawaban pemeriksaan dengan kunci. Perbedaan pada PG tunggal ditandai; jawaban uraian/mapping dengan redaksi berbeda perlu perbandingan manual. Tidak ada penggantian kunci diam-diam.
- Pemeriksaan ini bukan jaminan benar dan bukan pengujian psikometrik. Guru tetap meninjau hasil. Sistem tidak mengklaim memakai penyedia/model berbeda; independensi berarti permintaan baru tanpa kunci awal.
- Jika pemeriksaan gagal, soal yang sudah selesai tetap tersedia untuk ditinjau dan diunduh. Statusnya **Belum diperiksa**.
- **Perbaiki soal ini** hanya mengganti butir yang dipilih, mengikuti indikator dan bacaan tetap, lalu memeriksanya kembali. Bila gagal, butir lama tetap ada. Jika masalah terletak pada bacaan, guru dapat menyunting bacaan bersama lalu memeriksa ulang semua soal terkait.
- **Telaah mendalam** opsional menambah pemeriksaan petunjuk jawaban, kesejajaran opsi, tuntutan bahasa dan Barrett. Pemanggilan tambahan memakai kuota layanan.

## Tampilan hasil

- Pratinjau naskah menjadi tampilan awal. Bacaan tampil sekali untuk seluruh soal terkait.
- Mode **Sunting soal**, atau tombol sunting per butir/bacaan.
- Kunci dan dasar jawaban berada dalam panel yang dapat dibuka, dengan pilihan **Tampilkan kunci**.
- Navigasi nomor di kiri pada desktop; pilihan nomor ringkas pada ponsel.
- Toolbar mode dan unduhan tetap mudah dijangkau saat menggulir.
- Progres menunjukkan jumlah butir yang benar-benar diterima, bukan persentase perkiraan.
- Saat gagal, hasil parsial masih bisa dilihat. Tombol lanjut hanya membuat soal yang belum ada.
- **Hentikan sementara** membatalkan permintaan browser dan mempertahankan hasil diterima. Permintaan yang sudah sampai ke penyedia mungkin tetap dihitung sebagai penggunaan.
- Suntingan butir membatalkan pemeriksaan butir itu. Suntingan bacaan membatalkan pemeriksaan seluruh soal terkait; bukti lama tidak lagi ditampilkan sebagai valid.
- Word/PDF mengambil suntingan terkini, mengelompokkan bacaan, dan meletakkan kunci pada halaman terpisah. Catatan pemeriksaan tidak dimasukkan ke naskah siswa. Unduhan hanya tersedia setelah jumlah soal lengkap.

## Kuota, waktu, dan pemulihan

- Setiap percobaan memakai satu slot layanan dalam satu Netlify request: Gemini 1 → Gemini 2 → Groq. Kunci tetap di server.
- Anggaran per percobaan 50 detik; batas tunggu browser 60 detik. Anggaran keluaran disesuaikan menurut operasi/jumlah soal, tidak selalu 16.384 token.
- Backend membaca `Retry-After` atau informasi retry Google, lalu mengirim durasi tunggu yang aman tanpa membocorkan isi error penyedia.
- Browser melewati slot yang masih cooldown. Jika ada waktu tunggu hingga 60 detik, tersedia hitung mundur dan maksimal satu putaran retry tambahan. Waktu tunggu panjang ditampilkan sebelum tombol lanjut aktif. Tidak ada retry tanpa batas.
- Kuota harian yang teridentifikasi tidak langsung dicoba lagi; reset sebenarnya tetap mengikuti penyedia. Bila penyedia tidak memberi waktu tunggu, digunakan jeda awal 30 detik untuk pembatasan sementara.
- Generate mengirim konteks kelompok yang relevan ke model. Sumber lengkap/gambar tidak dikirim ulang ke model jika telaah sudah menghasilkan kutipan soal per butir. Frontend tetap membawa blueprint sesi ke server karena tidak ada database.
- Kuota habis tidak bisa ditambah oleh perbaikan aplikasi. Pemeriksaan tambahan tetap memerlukan permintaan tambahan.

| Kode | Makna / tindakan |
| --- | --- |
| `RATE_LIMIT` | Batas sementara atau jenis kuota belum diketahui; ikuti waktu tunggu. |
| `DAILY_QUOTA` | Pembatasan harian teridentifikasi; periksa kapan kuota pulih. |
| `SERVICE_TIMEOUT` | Permintaan belum selesai tepat waktu; lanjutkan hasil sebelumnya. |
| `SERVICE_AUTH` | Kunci/izin ditolak; periksa konfigurasi. |
| `MODEL_UNAVAILABLE` | Model tidak tersedia pada akun/endpoint. |
| `CONFIG` / `CONFIG_SLOT` | Kunci belum diatur; isi environment dan redeploy. |
| `INVALID_REVIEW` | Pemeriksaan tidak lengkap; soal tetap tersedia. |
| `OUTPUT_INCOMPLETE` / `INVALID_ITEM` | Hasil pembuatan belum lengkap atau tidak sesuai kontrak. |
| HTTP 404 / bukan JSON | Pastikan seluruh functions ikut dideploy. |

Log function memuat urutan layanan, rentang, durasi dan kode, tanpa teks soal/API key. Pesan teknis berada di **Detail kendala**.

## Batas dan data

Rekomendasi 1 halaman/10 soal merupakan saran mutu. Batas teknis tetap: 20 MB berkas, 160.000 karakter sumber, 200 soal, PDF teks 100 halaman, PDF pindai/campuran 5 halaman. Lima halaman digabung berpasangan menjadi maksimal tiga gambar. Dokumen melebihi batas pindai tidak dipotong diam-diam.

Mammoth mengekstrak teks Word; gambar tertanam tidak dibaca. Gunakan PDF pindai bila gambar diperlukan. Deteksi gambar pada PDF bergantung hasil ekstraksi teks; periksa pratinjau. Jumlah kata dihitung dari transkripsi bacaan, termasuk judul dan angka, tidak termasuk soal/opsi. Akurasi pindai bergantung kualitas transkripsi.

Sumber, koreksi dan hasil berada dalam memori halaman. Menutup/memuat ulang menghapusnya. `sessionStorage` hanya menyimpan tanda pop-up sudah ditutup, bukan isi soal. Tidak ada database/server storage aplikasi. Konten dikirim ke penyedia untuk pemrosesan dan tunduk pada kebijakan penyedia. Pustaka impor/ekspor memerlukan jsDelivr/cdnjs.

## Isi paket dan pengujian

`public/index.html`, `netlify/functions/{analyze,generate,passage,review}.cjs`, modul bersama `lib/core.cjs`, `netlify.toml`, `.env.example`, serta pengujian backend.

Jalankan `npm test`. Pengujian memakai respons simulasi, tidak memanggil API produksi. Pengujian browser memakai pustaka Word/PDF asli untuk impor/ekspor, sementara respons pembuatan/pemeriksaan disimulasikan. Kredensial API dan akses deployment produksi belum tersedia dalam sesi pengerjaan; lakukan uji langsung setelah pemasangan.

Rujukan teknis:
- https://docs.netlify.com/build/functions/configuration/
- https://ai.google.dev/gemini-api/docs/rate-limits
- https://console.groq.com/docs/rate-limits
- https://console.groq.com/docs/vision
- https://eric.ed.gov/?id=ED064672
