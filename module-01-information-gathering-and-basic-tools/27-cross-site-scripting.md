# 📝 Cross-Site Scripting (XSS) — Panduan & Catatan Studi

## 1. Ringkasan Singkat

Video ini merupakan sesi lab praktis yang mendemonstrasikan cara menguji kerentanan **Cross-Site Scripting (XSS)** secara aman menggunakan lingkungan yang terkontrol. Materi mencakup tiga jenis XSS utama — **Reflected**, **Stored**, dan **DOM-based** — dengan menggunakan tools seperti **Burp Suite**, **FoxyProxy**, dan aplikasi web yang sengaja dibuat rentan (**DVWA** / Damn Vulnerable Web Application). Seluruh pengujian dilakukan di mesin virtual Kali Linux milik sendiri, bukan pada target nyata yang tidak diizinkan.

---

## 2. Konsep Utama

### a. Apa Itu Cross-Site Scripting (XSS)?

Cross-Site Scripting atau **XSS** adalah salah satu jenis kerentanan keamanan pada aplikasi web yang memungkinkan penyerang menyisipkan kode berbahaya—biasanya berupa **JavaScript**—ke dalam halaman web yang kemudian dijalankan di browser pengguna lain.

#### Bagaimana XSS Bisa Terjadi?

Masalah utama dari XSS biasanya berasal dari **input pengguna yang tidak divalidasi atau tidak disaring (sanitize)** dengan benar. Ketika aplikasi menerima data dari user (misalnya dari form, query parameter, atau komentar), lalu langsung menampilkannya kembali ke halaman tanpa pengamanan, maka di situlah celah muncul.

Artinya, aplikasi “percaya begitu saja” terhadap input yang diberikan user, padahal input tersebut bisa saja berisi kode berbahaya.

#### Alur Terjadinya XSS

Supaya lebih jelas, kita lihat alur sederhananya:

```
[Penyerang] --> Memasukkan <script>alert('XSS')</script> ke form
[Server]    --> Menyimpan/memproses input tanpa sanitasi
[Korban]    --> Membuka halaman → JavaScript penyerang dieksekusi di browser korban
```

Penjelasan tiap langkah:

- **Penyerang** memasukkan script berbahaya ke dalam input (misalnya form komentar).
- **Server** menerima input tersebut, tetapi tidak melakukan filtering atau sanitasi.
- Data tersebut kemudian disimpan atau langsung ditampilkan kembali.
- **Korban** yang membuka halaman tersebut akan menjalankan script itu tanpa sadar, karena browser menganggapnya sebagai bagian dari halaman yang valid.

#### Contoh Sederhana dalam Kode

Misalnya ada aplikasi yang menampilkan komentar user seperti ini:

```html
<p>Komentar: {{ user_input }}</p>
```

Jika `user_input` berisi:

```html
<script>
  alert("XSS");
</script>
```

Maka hasil akhirnya di browser akan menjadi:

```html
<p>
  Komentar:
  <script>
    alert("XSS");
  </script>
</p>
```

Akibatnya, browser akan langsung menjalankan `alert('XSS')` tersebut.

#### Dampak Serangan XSS

XSS bukan sekadar menampilkan alert seperti contoh di atas. Dalam kasus nyata, dampaknya bisa jauh lebih serius, seperti:

- **Pencurian cookie** → digunakan untuk mengambil alih sesi login (session hijacking)
- **Pencurian token autentikasi**
- **Redirect ke situs berbahaya**
- **Manipulasi tampilan halaman** (deface ringan sampai berat)
- **Menjalankan aksi atas nama korban** tanpa sepengetahuan mereka

Karena script dijalankan di browser korban, maka penyerang bisa bertindak seolah-olah dia adalah pengguna tersebut.

#### Intinya

XSS terjadi karena aplikasi:

- Menerima input dari user
- Tidak memvalidasi atau menyaring input tersebut
- Menampilkan kembali input itu ke browser

Selama tiga kondisi ini terpenuhi, maka aplikasi berpotensi rentan terhadap XSS.

### b. Prasyarat Lab (Setup Lingkungan)

Sebelum mulai melakukan pengujian XSS, kita perlu menyiapkan lingkungan yang tepat agar proses testing bisa dilakukan dengan aman, terkontrol, dan mudah dianalisis. Bagian ini penting karena tools yang digunakan akan saling terhubung dalam satu alur kerja.

#### Komponen yang Perlu Disiapkan

Berikut adalah beberapa tools utama yang akan digunakan:

1. **Kali Linux Virtual Machine**
   Digunakan sebagai lingkungan utama untuk melakukan pengujian.
   Kenapa pakai Kali Linux? Karena sistem operasi ini sudah dilengkapi dengan berbagai tools keamanan, termasuk Burp Suite, sehingga kita tidak perlu install dari nol.

2. **Burp Suite**
   Ini adalah tool inti dalam proses testing. Fungsinya sebagai **proxy** yang bisa:
   - Mengintersep (menangkap) request dari browser
   - Melihat isi request dan response HTTP/HTTPS
   - Memodifikasi request sebelum dikirim ke server

   Dengan kata lain, Burp Suite memungkinkan kita “mengintip” dan “mengubah” komunikasi antara browser dan server.

3. **FoxyProxy (Ekstensi Browser)**
   Digunakan untuk mengarahkan traffic browser ke Burp Suite.
   Tanpa FoxyProxy, kita harus setting proxy secara manual di browser, yang biasanya kurang fleksibel.

   Dengan FoxyProxy:
   - Kita bisa dengan mudah ON/OFF proxy
   - Bisa switch konfigurasi hanya dengan satu klik

4. **Burp CA Certificate**
   Saat menggunakan HTTPS, komunikasi antara browser dan server terenkripsi.
   Agar Burp Suite tetap bisa membaca isi traffic tersebut, kita perlu menginstal **sertifikat CA dari Burp** ke browser.

   Tanpa ini:
   - Browser akan menampilkan error SSL
   - Traffic HTTPS tidak bisa diintersep dengan benar

   Dengan sertifikat ini:
   - Burp bisa melakukan dekripsi dan enkripsi ulang traffic secara transparan

5. **DVWA (Damn Vulnerable Web Application)**
   Ini adalah aplikasi web yang memang sengaja dibuat **rentan terhadap berbagai serangan**, termasuk XSS.

   DVWA sangat cocok untuk latihan karena:
   - Aman digunakan (tidak melanggar hukum)
   - Memiliki berbagai level kesulitan (low, medium, high)
   - Menyediakan banyak jenis vulnerability untuk dipelajari

#### Alur Traffic yang Dikonfigurasi

Setelah semua komponen siap, kita akan menghubungkannya dalam satu alur komunikasi seperti berikut:

```
Browser → FoxyProxy → Burp Suite → Server Target
```

Penjelasannya:

- **Browser**: tempat kita mengakses aplikasi web (DVWA)
- **FoxyProxy**: mengarahkan semua request dari browser ke Burp Suite
- **Burp Suite**: bertindak sebagai “perantara” yang bisa menangkap dan memodifikasi request
- **Server Target**: aplikasi web yang kita uji (misalnya DVWA)

Dengan setup ini, setiap request dari browser tidak langsung ke server, tetapi “berhenti” dulu di Burp Suite. Di sinilah kita bisa:

- Melihat parameter input
- Menyisipkan payload XSS
- Mengamati bagaimana server merespons

#### Catatan Penting (Etika Pengujian)

Pengujian keamanan seperti ini harus dilakukan secara bertanggung jawab.

⚠️ **Peringatan Etika:**

- Hanya lakukan pengujian pada sistem milik sendiri, seperti DVWA
- Atau pada sistem yang memang memberikan izin eksplisit untuk diuji

Melakukan testing tanpa izin pada sistem orang lain termasuk tindakan ilegal dan bisa berakibat serius secara hukum.

Dengan setup yang benar dan etika yang dijaga, proses belajar XSS akan jauh lebih aman dan efektif.

### c. Langkah Awal: Reconnaissance (Recon)

Sebelum melakukan eksploitasi seperti XSS, langkah pertama yang wajib dilakukan adalah **reconnaissance (recon)**, yaitu proses pengumpulan informasi terhadap aplikasi target. Tujuan utamanya adalah menemukan **titik-titik input yang bisa dikontrol oleh pengguna** (_user-controllable input points_), karena di situlah biasanya celah keamanan muncul.

#### Apa Itu User-Controllable Input?

User-controllable input adalah semua bagian dari aplikasi yang menerima data dari pengguna. Jika input ini tidak divalidasi atau disaring dengan benar, maka sangat berpotensi menjadi pintu masuk serangan seperti XSS.

Sebagai pentester, kita perlu berpikir:
_"Di bagian mana saja user bisa memasukkan data?"_

#### Contoh Titik Input yang Umum

Berikut beberapa titik yang paling sering menjadi target:

- **Form field**
  Misalnya:
  - Form login
  - Form registrasi
  - Form kontak

  Semua field seperti username, password, email, atau pesan adalah input yang bisa dimanipulasi.

- **Query parameter di URL**
  Contoh:

  ```bash
  https://example.com/search?search=kata
  ```

  Parameter `search` di atas bisa diubah-ubah oleh user, sehingga sangat menarik untuk diuji.

- **Comment box (kolom komentar)**
  Biasanya ditemukan di blog, forum, atau aplikasi sosial. Input dari komentar sering ditampilkan kembali ke user lain—ini sangat rawan XSS jika tidak difilter.

- **Search input (kolom pencarian)**
  Input pencarian sering ditampilkan kembali di halaman hasil (misalnya: “Hasil pencarian untuk: kata”), sehingga bisa menjadi titik refleksi XSS.

#### Cara Berpikir Saat Recon

Saat melakukan recon, jangan hanya sekadar mencoba-coba input. Coba pahami alurnya:

1. Di mana user bisa memasukkan data?
2. Apakah data tersebut ditampilkan kembali?
3. Apakah ada tanda-tanda input tersebut diproses tanpa sanitasi?

Semakin banyak titik input yang ditemukan, semakin besar peluang menemukan celah.

#### Tips Tambahan: Periksa Source Code Halaman

Salah satu teknik sederhana tapi sering overlooked adalah melihat **source code halaman** melalui fitur _View Page Source_ di browser.

Kadang, developer meninggalkan komentar di dalam HTML yang tidak terlihat oleh pengguna biasa, tetapi tetap bisa dibaca dari source code.

Contohnya:

```html
<!-- TODO: Hapus kredensial ini setelah deployment: admin/admin123 -->
```

Komentar seperti ini sangat berbahaya karena:

- Mengandung **informasi sensitif** (misalnya kredensial)
- Bisa memberikan petunjuk tentang struktur aplikasi
- Menunjukkan adanya konfigurasi yang belum dibersihkan

Bagi penyerang, informasi kecil seperti ini bisa menjadi titik awal untuk eksploitasi yang lebih besar.

#### Intinya

Recon adalah tahap eksplorasi untuk:

- Mengidentifikasi semua input yang bisa dikontrol user
- Memahami bagaimana aplikasi memproses input tersebut
- Mengumpulkan informasi tambahan dari source code

Semakin teliti proses recon, semakin besar kemungkinan menemukan vulnerability seperti XSS di tahap berikutnya.

### d. Tipe 1 — Reflected XSS (XSS yang Direfleksikan)

Reflected XSS adalah salah satu jenis XSS yang paling umum dan biasanya paling mudah ditemukan saat pengujian awal.

#### Definisi

Pada Reflected XSS, payload yang dikirim oleh penyerang **tidak disimpan di server**. Sebaliknya, payload tersebut:

- Dikirim melalui request (misalnya lewat URL atau form)
- Lalu langsung **dipantulkan kembali (reflected)** oleh server ke dalam response HTML
- Tanpa melalui proses sanitasi yang aman

Karena tidak disimpan, vulnerability ini bersifat **sementara (non-persistent)** — hanya aktif saat request tertentu dijalankan.

#### Karakteristik Utama

Beberapa ciri khas Reflected XSS:

- Payload hanya aktif saat request dikirim
- Tidak tersimpan di database
- Biasanya memanfaatkan:
  - Query parameter (`?q=...`)
  - Form input

- Sering digunakan dalam serangan berbasis **social engineering**, seperti:
  - Link phishing
  - Email berisi URL berbahaya

#### Alur Terjadinya Reflected XSS

Berikut gambaran alurnya:

```id="fyzsjm"
Penyerang membuat URL → Korban klik URL → Request dikirim ke server
→ Server memantulkan payload → Payload dieksekusi di browser korban
```

Penjelasan:

1. **Penyerang** membuat URL yang sudah disisipi payload XSS
2. **Korban** mengklik URL tersebut (biasanya tanpa sadar)
3. Request dikirim ke server
4. Server mengembalikan response yang berisi payload tersebut
5. Browser korban menjalankan script yang ada di dalam response

#### Contoh Sederhana

Misalnya ada URL seperti ini:

```bash
https://example.com/search?q=<script>alert("XSS")</script>
```

Jika aplikasi tidak melakukan sanitasi, maka nilai `q` akan langsung ditampilkan di halaman, misalnya:

```html
<p>
  Hasil pencarian untuk:
  <script>
    alert("XSS");
  </script>
</p>
```

Akibatnya, browser akan mengeksekusi `alert("XSS")`.

#### Langkah Pengujian di Lab (DVWA + Burp Suite)

Untuk memahami secara praktis, berikut alur pengujian di lab:

1. Masuk ke halaman **Reflected XSS** di DVWA
   Biasanya tersedia sebagai modul khusus untuk latihan.

2. Aktifkan **Interceptor** di Burp Suite
   Ini akan menghentikan request sebelum dikirim ke server.

3. Masukkan payload sederhana di form input:

   ```html
   <script>
     alert("XSS");
   </script>
   ```

4. Klik **Submit**
   Request akan tertahan di Burp Suite.

5. Kirim ke Repeater
   Klik kanan request → pilih **Send to Repeater**
   Tujuannya agar kita bisa mengirim ulang request dengan mudah tanpa harus ulang dari browser.

6. Analisis di tab Repeater
   Klik **Send**, lalu perhatikan response dari server.

7. Cari payload di response HTML
   Jika payload muncul seperti ini:

   ```html
   <script>
     alert("XSS");
   </script>
   ```

   tanpa diubah menjadi bentuk aman (misalnya `&lt;script&gt;`), maka aplikasi kemungkinan rentan.

8. Verifikasi di browser
   Matikan interceptor, kirim ulang request dari browser.
   Jika muncul pop-up alert, berarti payload benar-benar dieksekusi → XSS berhasil.

#### Contoh Payload Sederhana (Proof of Concept)

Payload paling dasar untuk menguji XSS adalah:

```html
<script>
  alert("XSS");
</script>
```

Kenapa pakai ini?

- Mudah dikenali
- Tidak merusak sistem
- Cukup untuk membuktikan bahwa JavaScript bisa dieksekusi

#### Intinya

Reflected XSS terjadi ketika:

- Input dari user dikirim ke server
- Server langsung mengembalikan input tersebut ke halaman
- Tanpa proses sanitasi

Karena sifatnya tidak persisten, serangan ini sangat bergantung pada **korban mengklik link berbahaya**. Oleh karena itu, Reflected XSS sering dikombinasikan dengan teknik phishing agar lebih efektif.

### e. Tipe 2 — Stored XSS (XSS yang Tersimpan)

Setelah memahami Reflected XSS, sekarang kita masuk ke jenis yang lebih serius, yaitu **Stored XSS**. Ini adalah tipe XSS yang dampaknya jauh lebih luas dan berbahaya.

#### Definisi

Stored XSS terjadi ketika payload berbahaya:

- Dikirim oleh penyerang melalui input (misalnya form)
- **Disimpan secara permanen di server** (biasanya di database)
- Kemudian ditampilkan kembali ke pengguna lain tanpa sanitasi

Artinya, berbeda dengan Reflected XSS, payload di sini tidak hilang setelah request selesai. Ia tetap ada dan akan terus dijalankan setiap kali halaman tersebut diakses.

#### Kenapa Stored XSS Lebih Berbahaya?

Ada beberapa alasan kenapa Stored XSS dianggap paling berbahaya:

- **Tidak butuh interaksi khusus dari korban**
  Korban tidak perlu klik link tertentu. Cukup membuka halaman yang terinfeksi, payload langsung berjalan.

- **Menyerang banyak pengguna sekaligus**
  Semua user yang mengakses halaman tersebut akan terkena dampaknya.

- **Persisten (berkelanjutan)**
  Selama payload masih tersimpan di database, serangan akan terus aktif.

Karena sifatnya ini, Stored XSS sering digunakan untuk serangan skala besar, seperti mencuri session banyak user sekaligus.

#### Alur Terjadinya Stored XSS

Berikut alur sederhananya:

```id="bjh6zq"
Penyerang memasukkan payload → Payload tersimpan di database server
→ Setiap pengguna yang membuka halaman → Payload dieksekusi otomatis
```

Penjelasan:

1. Penyerang memasukkan script berbahaya melalui input (misalnya kolom komentar)
2. Server menyimpan input tersebut ke database tanpa sanitasi
3. Saat halaman ditampilkan ke user lain, data dari database ikut ditampilkan
4. Browser user mengeksekusi script tersebut tanpa sadar

#### Contoh Kasus Sederhana

Misalnya ada fitur komentar:

```html
<p><strong>Nama:</strong> Alvin</p>
<p>
  <strong>Komentar:</strong>
  <script>
    alert("Stored XSS");
  </script>
</p>
```

Setiap kali halaman ini dibuka:

- Script `<script>alert("Stored XSS")</script>` akan selalu dijalankan
- Tidak peduli siapa yang membuka halaman tersebut

#### Langkah Pengujian di Lab (DVWA)

Berikut langkah praktis untuk menguji Stored XSS:

1. Masuk ke halaman **Stored XSS** di DVWA
   Biasanya berupa form komentar dengan field:
   - `name`
   - `message`

2. Uji dengan input normal terlebih dahulu
   Tujuannya memastikan form bekerja dengan benar (baseline).

3. Masukkan payload di field **message**:

   ```html
   <script>
     alert("Stored XSS");
   </script>
   ```

4. Klik **Submit**
   Payload akan dikirim ke server dan (jika rentan) disimpan di database.

5. Refresh halaman
   Jika setelah refresh muncul pop-up alert lagi, berarti payload sudah tersimpan.

#### Indikator Kunci Stored XSS

Ciri paling jelas dari Stored XSS adalah:

- Payload tetap aktif **setelah halaman di-refresh**
- Bahkan tetap berjalan saat dibuka oleh user lain

Ini menandakan bahwa script tersebut benar-benar tersimpan di server, bukan hanya sementara.

#### Perbedaan dengan Reflected XSS

Agar lebih jelas, berikut perbandingannya:

- **Penyimpanan payload**
  - Reflected XSS: tidak disimpan
  - Stored XSS: disimpan di database

- **Durasi serangan**
  - Reflected XSS: hanya saat request tertentu
  - Stored XSS: terus aktif sampai dihapus

- **Target korban**
  - Reflected XSS: hanya user yang mengklik link
  - Stored XSS: semua user yang membuka halaman

- **Tingkat bahaya**
  - Reflected XSS: sedang
  - Stored XSS: tinggi

#### Intinya

Stored XSS terjadi ketika:

- Input berbahaya disimpan di server
- Ditampilkan kembali tanpa sanitasi
- Dieksekusi otomatis oleh browser setiap user

Karena sifatnya yang persisten dan massal, Stored XSS adalah salah satu vulnerability yang harus diprioritaskan untuk diperbaiki dalam aplikasi web.

### f. Tipe 3 — DOM-Based XSS

Selain Reflected dan Stored XSS, ada satu jenis lagi yang cukup unik, yaitu **DOM-Based XSS**. Tipe ini sering membingungkan di awal karena tidak melibatkan server secara langsung.

#### Definisi

DOM-Based XSS adalah serangan yang terjadi sepenuhnya di sisi **client (browser)**, dengan memanfaatkan manipulasi **DOM (Document Object Model)**.

Artinya:

- Payload **tidak pernah dikirim ke server**
- Tidak ada response dari server yang mengandung payload
- Semua eksekusi terjadi karena **JavaScript di browser memproses data secara tidak aman**

#### Apa Itu DOM?

DOM adalah representasi struktur halaman HTML dalam bentuk objek yang bisa dimanipulasi menggunakan JavaScript.

Contohnya:

- Mengubah isi elemen (`innerHTML`)
- Mengambil nilai dari URL (`location`)
- Menampilkan data ke halaman

Masalah muncul ketika JavaScript di halaman:

- Mengambil data dari sumber yang bisa dikontrol user (misalnya URL)
- Lalu langsung memasukkannya ke DOM tanpa sanitasi

#### Sumber Input pada DOM XSS

Beberapa sumber data yang sering digunakan dalam DOM-Based XSS:

- `location.href`
- `location.search`
- `location.hash`
- `document.referrer`

Semua ini bisa dimodifikasi oleh user, sehingga berpotensi membawa payload berbahaya.

#### Contoh Sederhana

Perhatikan contoh berikut:

```javascript
// URL: http://example.com/page#<img src=x onerror=alert(1)>

// Script di halaman:
document.getElementById("output").innerHTML = location.hash.slice(1);
```

Mari kita breakdown alurnya:

1. User membuka URL:

   ```bash
   http://example.com/page#<img src=x onerror=alert(1)>
   ```

2. Bagian setelah `#` disebut **hash**, dan bisa diakses dengan:

   ```javascript
   location.hash;
   ```

3. Script mengambil isi hash:

   ```javascript
   location.hash.slice(1);
   ```

   (menghapus karakter `#` di depan)

4. Nilai tersebut langsung dimasukkan ke dalam:

   ```javascript
   innerHTML;
   ```

5. Karena menggunakan `innerHTML`, browser akan:
   - Menganggapnya sebagai HTML
   - Mengeksekusi atribut seperti `onerror`

Hasilnya:

- `<img src=x onerror=alert(1)>` akan dijalankan
- Alert muncul di browser

#### Kenapa Ini Berbahaya?

Walaupun tidak melibatkan server, dampaknya tetap serius:

- Bisa mencuri data dari halaman
- Bisa memodifikasi tampilan
- Bisa menjalankan aksi atas nama user

Yang membuatnya tricky:

- Tidak terlihat di server logs
- Tidak terdeteksi oleh filter backend
- Sepenuhnya bergantung pada JavaScript di frontend

#### Perbedaan dengan XSS Lain

Perbedaan utama dibanding jenis lain:

- **Reflected XSS**: payload dipantulkan oleh server
- **Stored XSS**: payload disimpan di server
- **DOM-Based XSS**: payload diproses langsung oleh browser

Jadi, sumber masalahnya bukan di server, melainkan di **cara JavaScript menangani data di sisi client**.

#### Intinya

DOM-Based XSS terjadi ketika:

- JavaScript mengambil data dari sumber yang bisa dikontrol user (misalnya URL)
- Data tersebut dimasukkan ke DOM tanpa sanitasi
- Browser mengeksekusi data tersebut sebagai kode

Karena sepenuhnya terjadi di sisi client, pengujian DOM XSS biasanya memerlukan analisis langsung terhadap JavaScript yang digunakan di halaman.

### g. Penggunaan Burp Suite dalam Pengujian XSS

Dalam pengujian XSS, **Burp Suite** berperan sebagai alat utama untuk mengamati dan memanipulasi komunikasi antara browser dan server. Secara konsep, Burp bekerja sebagai **proxy man-in-the-middle**, artinya semua request dari browser akan “melewati” Burp terlebih dahulu sebelum sampai ke server.

Dengan posisi ini, kita mendapatkan kontrol penuh untuk:

- Melihat isi request dan response
- Mengubah data sebelum dikirim
- Menguji berbagai payload tanpa harus bolak-balik dari browser

#### Cara Kerja Singkat

Alurnya seperti ini:

1. Browser mengirim request
2. Request ditangkap oleh Burp Suite
3. Kita bisa:
   - Menahan request
   - Mengedit isi request
   - Mengirim ulang request

4. Baru setelah itu request diteruskan ke server

Pendekatan ini sangat powerful untuk menemukan celah seperti XSS.

#### Fitur Utama yang Digunakan

Berikut fitur-fitur penting di Burp Suite yang sering dipakai saat testing XSS:

##### Proxy → Intercept

Fitur ini digunakan untuk **menahan request sebelum dikirim ke server**.

Ketika Intercept aktif:

- Setiap request dari browser akan “berhenti” di Burp
- Kita bisa melihat detail seperti:
  - Parameter input
  - Header
  - Body request

Contoh sederhana request yang diintersep:

```http
POST /comment HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

message=hello
```

Di sini, kita bisa langsung mengganti nilai `message` menjadi payload XSS, misalnya:

```http
message=<script>alert('XSS')</script>
```

Lalu klik **Forward** untuk mengirim request ke server.

##### Repeater

Fitur ini digunakan untuk **mengirim ulang request yang sama berkali-kali dengan mudah**.

Kenapa ini penting?

Saat testing XSS, kita biasanya ingin:

- Mencoba berbagai payload
- Mengubah sedikit demi sedikit input
- Melihat perbedaan response dari server

Dengan Repeater:

- Kita tidak perlu kembali ke browser untuk submit form
- Cukup edit request di Burp, lalu klik **Send**

Ini sangat menghemat waktu dan membuat proses eksplorasi lebih efisien.

##### Request Modification

Burp memungkinkan kita untuk **mengubah hampir semua bagian dari request secara manual**, seperti:

- Parameter (GET/POST)
- Header (User-Agent, Cookie, dll.)
- Body request

Contoh modifikasi:

```http
GET /search?q=test HTTP/1.1
```

Bisa diubah menjadi:

```http
GET /search?q=<script>alert(1)</script> HTTP/1.1
```

Dengan cara ini, kita bisa langsung menguji apakah parameter `q` rentan terhadap XSS.

#### Tips Penting: Gunakan “Send to Repeater”

Setelah request berhasil diintersep di tab Proxy, sangat disarankan untuk:

- Klik kanan pada request
- Pilih **Send to Repeater**

Keuntungannya:

- Request tersimpan di tab Repeater
- Bisa dimodifikasi dan dikirim ulang kapan saja
- Tidak perlu mengulang input dari browser

Ini sangat membantu terutama saat:

- Menguji banyak payload
- Membandingkan response
- Menganalisis bagaimana aplikasi memproses input

#### Intinya

Burp Suite membantu kita untuk:

- Mengontrol alur request dan response
- Menyisipkan payload XSS secara fleksibel
- Menguji berbagai skenario tanpa batasan dari UI aplikasi

Dengan memanfaatkan fitur seperti **Intercept** dan **Repeater**, proses pengujian XSS menjadi jauh lebih sistematis, cepat, dan efektif.

### h. Sanitasi Input dan Mitigasi XSS

Setelah memahami bagaimana XSS bekerja, langkah berikutnya yang tidak kalah penting adalah **bagaimana mencegahnya**. Salah satu teknik utama yang digunakan adalah **sanitasi input**.

#### Apa Itu Sanitasi Input?

Sanitasi input adalah proses:

- Menyaring
- Membersihkan
- Atau mengubah input dari user

Tujuannya agar input tersebut **aman saat ditampilkan kembali di halaman web**, tanpa berisiko dieksekusi sebagai kode.

Dalam konteks XSS, fokus utamanya adalah memastikan bahwa **input user tidak diperlakukan sebagai HTML atau JavaScript oleh browser**.

#### Bagaimana Sanitasi Mencegah XSS?

Mari kita lihat contoh payload berbahaya:

```html
<script>
  alert("XSS");
</script>
```

Jika aplikasi **tidak melakukan sanitasi**, maka:

- Browser akan membaca `<script>` sebagai tag HTML
- JavaScript di dalamnya akan dijalankan

Namun, jika aplikasi melakukan sanitasi dengan benar, karakter berbahaya akan diubah menjadi **HTML entities**.

#### Konsep Escaping (Encoding)

Karakter-karakter khusus seperti berikut:

- `<`
- `>`
- `"`
- `'`

Harus di-_escape_ menjadi:

- `<` → `&lt;`
- `>` → `&gt;`
- `"` → `&quot;`
- `'` → `&#x27;`

Dengan cara ini, browser tidak lagi menganggapnya sebagai tag HTML, melainkan sebagai teks biasa.

#### Contoh Output yang Aman

Berikut ilustrasi yang lebih jelas:

```html id="z17zuh"
<!-- Input pengguna: <script>alert('XSS')</script> -->
<!-- Output di HTML (setelah di-escape): -->
&lt;script&gt;alert('XSS')&lt;/script&gt;
<!-- Browser hanya menampilkan teks, tidak mengeksekusi -->
```

Penjelasan:

- Input asli berisi tag `<script>`
- Setelah disanitasi, tag tersebut berubah menjadi teks biasa
- Browser tidak mengeksekusi apa pun, hanya menampilkan string

Hasil di browser:

```
<script>alert('XSS')</script>
```

(sebagai teks, bukan sebagai kode yang dijalankan)

#### Kenapa Ini Efektif?

Karena browser hanya akan mengeksekusi sesuatu jika:

- Dikenali sebagai tag HTML atau JavaScript

Dengan mengubah karakter khusus menjadi HTML entities:

- Struktur HTML “rusak” (dalam arti aman)
- Payload tidak bisa dieksekusi

#### Catatan Penting

Sanitasi harus dilakukan:

- **Setiap kali menampilkan input user ke halaman**
- Terutama jika menggunakan:
  - `innerHTML`
  - Template rendering tanpa escaping otomatis

Selain itu, pendekatan terbaik biasanya adalah:

- Gunakan **auto-escaping** dari framework (seperti templating engine modern)
- Hindari memasukkan input langsung ke dalam HTML tanpa filter

#### Intinya

XSS bisa dicegah jika:

- Input user tidak langsung dipercaya
- Semua karakter berbahaya di-_escape_
- Browser hanya menerima data sebagai teks, bukan sebagai kode

Sanitasi input adalah garis pertahanan pertama yang sangat penting dalam menjaga aplikasi web tetap aman dari serangan XSS.

## 3. Hubungan Antar Konsep

Berikut adalah alur pemahaman yang utuh dari materi ini:

```
[Setup Lab]
     ↓
Kali Linux + Burp Suite + FoxyProxy + DVWA
     ↓
[Recon: Temukan Input Points]
     ↓
Cari form, query param, comment box, dll.
     ↓
[Uji Jenis XSS]
     ├──→ Reflected XSS → Payload direfleksikan langsung dari response
     ├──→ Stored XSS    → Payload disimpan di server, muncul di semua user
     └──→ DOM XSS       → Payload dieksekusi di browser via manipulasi DOM
     ↓
[Verifikasi Keberhasilan]
     ↓
Pop-up alert muncul = Kerentanan terkonfirmasi
     ↓
[Mitigasi]
     ↓
Sanitasi & validasi input di sisi server + encoding output
```

Ketiga tipe XSS semuanya berakar dari **kegagalan sanitasi input pengguna**. Perbedaannya terletak pada _di mana dan kapan_ payload dieksekusi. Burp Suite menjadi jembatan teknis yang memungkinkan pentester untuk mengamati, memodifikasi, dan mengirim ulang request secara tepat sasaran.

---

## 4. Kesimpulan

- **XSS (Cross-Site Scripting)** adalah kerentanan web di mana penyerang dapat menyuntikkan JavaScript ke halaman yang dilihat pengguna lain, akibat kurangnya sanitasi input.
- Terdapat tiga jenis XSS: **Reflected** (sementara, dari URL), **Stored** (permanen, tersimpan di server), dan **DOM-based** (di sisi browser).
- **Stored XSS** adalah yang paling berbahaya karena memengaruhi semua pengunjung halaman secara otomatis dan terus-menerus.
- **Burp Suite** adalah tool esensial untuk mengintersep, menganalisis, dan memodifikasi HTTP request dalam pengujian XSS.
- **Mitigasi utama:** Selalu lakukan **sanitasi dan validasi input** di sisi server serta **encoding output** agar karakter berbahaya tidak dieksekusi browser.
- Semua pengujian **harus dilakukan hanya pada environment yang Anda kendalikan sendiri** (lab/CTF/bug bounty dengan izin). Pengujian tanpa izin adalah tindakan ilegal.

---

> 📌 **Referensi Tools:**
>
> - **Burp Suite Community Edition** — [https://portswigger.net/burp](https://portswigger.net/burp)
> - **DVWA (Damn Vulnerable Web Application)** — [https://dvwa.co.uk](https://dvwa.co.uk)
> - **OWASP XSS Prevention Cheat Sheet** — [https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
