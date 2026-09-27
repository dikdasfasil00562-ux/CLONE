# Test Items Clone

Aplikasi penyusun soal paralel Bahasa Inggris oleh **Suryo Agung Nugroho**.

## Jalankan di Netlify

**Paket ini berisi kode siap-deploy, bukan situs yang sudah online.** Pemrosesan memerlukan kunci API aktif milik pengelola. Jangan menaruh kunci di `index.html`, GitHub, atau percakapan.

### Cara A — GitHub ke Netlify

1. Ekstrak ZIP ini. Unggah isi folder `test-items-clone` ke repository GitHub: `netlify.toml`, `package.json`, `public/`, dan `netlify/` harus berada di akar repository.
2. Di Netlify, pilih tambah/import project dari repository GitHub tersebut.
3. Biarkan build command kosong. Publish directory: `public`. Functions directory: `netlify/functions` (sudah diatur pada `netlify.toml`).
4. Di pengaturan proyek Netlify, buka **Environment variables**. Tambahkan tiga kunci berikut dengan scope **Functions** dan konteks **Production**. Jika memakai Deploy Preview, tambahkan konteks tersebut juga.
5. Deploy atau redeploy setelah mengisi variabel.
6. Buka alamat `https://nama-situs.netlify.app`. Uji pertama dengan 2–4 soal melalui tab Tempel teks, kemudian berkas Word, PDF teks, dan PDF pindai.

| Environment variable | Nilai |
| --- | --- |
| `GEMINI_API_KEY_1` | Kunci Gemini pertama |
| `GEMINI_API_KEY_2` | Kunci Gemini kedua |
| `GROQ_API_KEY` | Kunci Groq |
| `GEMINI_MODEL` | Opsional; default `gemini-3.5-flash` |
| `GROQ_MODEL` | Opsional; default `qwen/qwen3.8-27b` |

Isi ketiga kunci agar semua tahap fallback tersedia. Jika hanya satu kunci terisi, aplikasi memakai layanan yang tersedia. Dua kunci Gemini pada proyek dengan kuota yang sama dapat tetap berbagi batas kuota.

### Cara B — Netlify CLI

Dengan Node.js 22+ terpasang, buka terminal di folder hasil ekstrak:

```sh
npx netlify-cli login
npx netlify-cli deploy --prod --dir=public --functions=netlify/functions
```

Ikuti pilihan membuat atau menghubungkan site, lalu isi variabel di dashboard Netlify dan lakukan deploy ulang. Pengembangan lokal dapat memakai `npx netlify-cli dev` setelah variabel lokal diatur. `.env.example` hanya berisi nama variabel; jangan unggah `.env` berisi kunci.

**Jangan hanya menyeret folder `public` ke Netlify Drop.** Situs statis saja tidak menyertakan backend pemrosesan. Membuka `index.html` langsung juga tidak dapat menjalankan endpoint server.

## Isi paket

- `public/index.html`: satu frontend dengan CSS dan JavaScript inline; komponen dokumen dimuat dari CDN saat diperlukan.
- `netlify/functions/analyze.cjs`: endpoint telaah.
- `netlify/functions/generate.cjs`: endpoint pembuatan.
- `netlify/functions/lib/core.cjs`: validasi, prompt, pemanggilan layanan, fallback, dan penanganan galat.
- `netlify.toml`: direktori publish/functions dan header dasar.
- `tests/core.test.cjs`: pengujian validasi dan fallback dengan respons layanan simulasi.

Frontend diletakkan dalam `public/` agar kode backend dan dokumen konfigurasi tidak ikut dipublikasikan sebagai aset statis. Tidak ada bundler untuk frontend dan tidak perlu build aplikasi. Netlify tetap memaketkan functions pada saat deployment.

## Alur dan batas teknis

1. Unggah `.docx`/`.pdf` atau tempel teks. Berkas maksimal 20 MB; teks maksimal 160.000 karakter dan 200 soal per dokumen.
2. Word diekstrak sebagai teks menggunakan Mammoth. Gambar tertanam Word tidak dibaca; simpan dokumen bergambar sebagai PDF pindai jika gambar diperlukan untuk menjawab soal.
3. PDF diekstrak per halaman. Jika rata-rata teks rendah atau ada halaman minim teks, dokumen diproses sebagai gambar. PDF pindai/campuran maksimal **5 halaman**. Dokumen yang lebih panjang ditolak dengan petunjuk membagi berkas, tidak dipotong diam-diam. PDF teks maksimal 100 halaman.
4. Lima halaman gambar digabung berpasangan menjadi maksimal **3 gambar** agar cocok dengan batas vision Groq yang diperiksa pada 27 September 2026. Semua halaman tetap dikirim dalam urutan asli. Payload permintaan dibatasi di bawah batas fungsi Netlify.
5. Hasil telaah menunggu tombol **Lanjut ke Pembuatan Soal**; tidak otomatis membuat soal.
6. Pembuatan dilakukan per **4 butir**. Tiap permintaan mencoba Gemini 1 → Gemini 2 → Groq. Anggaran total layanan sekitar 53 detik per permintaan, dibagi di antara kunci yang tersedia; browser berhenti menunggu setelah 60 detik.
7. Jika satu batch gagal, hasil batch sebelumnya tetap di memori browser. **Coba kembali** melanjutkan batch yang gagal, tidak menggandakan soal. Unduhan baru tersedia setelah semua butir lengkap.
8. Bacaan bersama menggunakan identitas grup internal. Suntingan pada satu bacaan disinkronkan ke soal lain yang memakai bacaan itu. Pada ekspor, bacaan bersama hanya ditampilkan sekali, dengan nomor soal terkait.
9. Semua jenis soal mengikuti sumber. Isian/uraian tidak dipaksa menjadi pilihan ganda. Soal menjodohkan, kategori, dan pilihan kompleks ditulis melalui petunjuk, daftar pernyataan/pilihan, serta kunci lengkap.
10. Word dan PDF memakai A4. Kunci berada pada halaman baru, tidak tercampur dalam naskah soal. Pengunduhan memakai suntingan terbaru.

Data hanya berada dalam memori halaman selama sesi. Muat ulang/menutup tab akan menghapus sumber dan hasil. Tidak ada database atau penyimpanan soal di server aplikasi. Konten tetap dikirim ke penyedia layanan untuk diproses dan mengikuti kebijakan data penyedia tersebut. Log aplikasi hanya memuat jenis kegagalan, urutan layanan, dan kode status; tidak mencatat soal atau kunci API.

## Kontrak endpoint

Endpoint utama tetap `/.netlify/functions/analyze` dan `/.netlify/functions/generate` dengan input/hasil sesuai master prompt. Ekstensi internal untuk konsistensi dan batching:

- `analisis.polaButir`: satu blueprint per soal (`nomor`, `jenisSoal`, `jumlahPilihan`, `grupStimulus`, `pola`).
- Generate menerima `start`, `end`, dan `generatedStimuli` (peta ID grup ke bacaan baru) secara opsional. Frontend selalu menggunakan batching. Tanpa rentang, endpoint mencoba menghasilkan seluruh soal sekaligus, yang dapat melebihi batas waktu untuk dokumen panjang.

Tidak ada penilaian rubrik tambahan. Validasi memeriksa struktur, jumlah, nomor soal, jumlah pilihan, format kunci PG tunggal, dan konsistensi bacaan; kesetaraan pedagogis/CEFR serta kebenaran isi tetap perlu ditinjau guru.

## Jika terjadi masalah

| Pesan/kode | Tindakan |
| --- | --- |
| `CONFIG` | Isi kunci API pada scope Functions lalu redeploy. |
| `PROCESS_FAILED` | Periksa kuota, akses model, dan log function; coba ulang atau bagi dokumen. |
| HTTP 404 / respons bukan JSON | Pastikan functions ikut dideploy; jangan jalankan hanya sebagai situs statis. |
| HTTP 504 / waktu habis | Coba lagi; hasil batch sebelumnya tetap ada. Kurangi ukuran sumber bila berulang. |
| Komponen dokumen gagal dimuat | Periksa akses ke jsDelivr dan cdnjs; dibutuhkan untuk impor/ekspor. |
| PDF pindai lebih dari 5 halaman | Bagi sumber dengan mempertahankan bacaan beserta soal terkait dalam berkas yang sama. |

Log dapat ditemukan pada bagian **Functions** proyek Netlify; pilih `analyze` atau `generate`. Nama menu dashboard dapat berubah. Model dapat diperbarui lewat environment tanpa mengedit kode, asalkan mendukung gambar, JSON, dan endpoint yang dipakai.

## Verifikasi

Jalankan `npm test` untuk pemeriksaan backend dengan mock provider. Pengujian ini tidak menghubungi atau menagih layanan. Pengujian langsung dengan API produksi tetap perlu dilakukan setelah kunci dipasang. Hasil yang muncul hanya setelah respons layanan sukses; tidak ada hasil soal contoh yang disamarkan sebagai hasil nyata.

## Rujukan teknis

Diperiksa 27 September 2026:
- https://ai.google.dev/gemini-api/docs/models/gemini-3.5-flash
- https://ai.google.dev/gemini-api/docs/structured-output
- https://console.groq.com/docs/vision
- https://docs.netlify.com/build/functions/configuration/

Default Groq memakai Qwen 3.8 27B sebagai model setara pengganti contoh Qwen 3.6 pada spesifikasi, sesuai dokumentasi vision saat pengerjaan. Ketersediaan aktual bergantung akun dan perubahan penyedia.

### Hasil pemeriksaan paket

- Lima pengujian backend lulus: validasi input, validasi hasil, fallback tiga layanan, konfigurasi kosong, dan origin.
- Uji browser desktop/ponsel lulus dengan respons endpoint simulasi: konfirmasi manual, enam butir campuran, melanjutkan batch gagal tanpa duplikasi, sinkronisasi suntingan bacaan, dan tidak ada galat JavaScript/overflow horizontal pada lebar 390 px.
- Ekspor Word dan PDF asli berhasil; kedua hasil dapat diimpor kembali. Pemeriksaan isi memastikan suntingan masuk dalam unduhan dan kunci dimulai pada halaman terpisah.
- PDF pindai lima halaman menghasilkan tiga gambar gabungan; enam halaman ditolak dengan pesan pembagian berkas.
- Tampilan desktop, ponsel, dan halaman PDF diperiksa secara visual.
- Layanan Gemini/Groq produksi dan deployment Netlify belum diuji karena tidak tersedia kredensial layanan/hosting dalam sesi pengerjaan.

## Pembaruan telaah per nomor

Telaah keseluruhan hanya menampilkan jumlah soal dan bahasa terdeteksi. Telaah pola berupa tabel satu baris per nomor: jenis teks, jenis soal, tingkat kesulitan, taksonomi Barrett, level CEFR, dan perkiraan kisi-kisi soal (indikator). Seluruh kolom dianalisis per butir dan turut menjadi acuan pembuatan soal baru. Kisi-kisi bukan kutipan dokumen kurikulum resmi; kesulitan dan CEFR berupa perkiraan. Barrett diterapkan pada pemahaman bacaan dan ditandai tidak relevan untuk butir di luar cakupannya.

Kontrak terbaru: `analisis` berisi `jumlahSoal`, `bahasa`, dan `polaButir`. Setiap pola butir menyertakan `jenisTeks`, `tingkatKesulitan`, `taksonomiBarrett`, `levelCEFR`, dan `kisiKisi`, selain field internal sebelumnya. Ganti frontend dan functions sekaligus saat redeploy.

Rujukan kategori Barrett: https://eric.ed.gov/?id=ED064672

## Pembaruan bacaan/teks terdeteksi

Ditambahkan tabel di antara telaah keseluruhan dan telaah per butir, dengan kolom jenis teks, level CEFR bacaan, jumlah kata, dan nomor soal terkait. Setiap bacaan unik tampil sekali walaupun digunakan beberapa soal. Bacaan berbeda dengan genre yang sama tetap memiliki baris terpisah.

`analisis.bacaanTerdeteksi` berisi `id`, `jenisTeks`, `levelCEFR`, dan `teks` (transkripsi sumber). Server menghitung `jumlahKata` dari transkripsi dan menurunkan `untukSoal` dari grup stimulus setiap butir; angka tidak hanya dipercayakan pada perkiraan model. Judul dan isi bacaan dihitung, pertanyaan/opsi/petunjuk ujian dikecualikan. Kata berapostrof atau bertanda hubung dihitung satu kata; angka dihitung sebagai token kata. Akurasi pada pindai bergantung keterbacaan transkripsi. Soal tanpa bacaan menampilkan pesan tidak ditemukan stimulus.

Deploy ulang frontend dan backend bersama untuk menggunakan kontrak baru ini.

## Pengingat unggah

Pop-up tampil saat halaman unggah pertama kali dibuka dan saat pengguna kembali ke tab Unggah berkas. Pesan menegaskan rekomendasi **1 halaman dengan tidak lebih dari 10 soal** untuk hasil terbaik. Tombol Saya mengerti atau Escape menutup pop-up; pengingat ringkas tetap terlihat dan dapat dibuka ulang. Ini rekomendasi kualitas, bukan batas pemrosesan baru.
