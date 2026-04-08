# 📝 BurpSuite - Bagian 2: Intercept Client-Side Request (Praktik)

## 1. Ringkasan Singkat

Video ini merupakan lanjutan dari seri BurpSuite (Part 1), yang sebelumnya membahas antarmuka dan subseksi BurpSuite secara teori. Pada bagian kedua ini, fokusnya adalah **demonstrasi praktis** cara mengonfigurasi BurpSuite untuk mencegat (_intercept_) permintaan HTTP dari sisi klien (_client-side request_) menggunakan browser Firefox di Kali Linux. Materi mencakup: pengaturan proxy listener di BurpSuite, instalasi ekstensi FoxyProxy di Firefox, instalasi sertifikat CA BurpSuite, hingga proses intercept GET dan POST request secara nyata.

---

## 2. Konsep Utama

### a. Proxy Listener di BurpSuite

#### Konsep Dasar: BurpSuite sebagai Man-in-the-Middle

BurpSuite bekerja sebagai sebuah _man-in-the-middle proxy_. Artinya, ia “berdiri di tengah” antara browser (client) dan server tujuan. Semua request yang keluar dari browser tidak langsung menuju server, tetapi akan melewati BurpSuite terlebih dahulu.

Dengan posisi ini, BurpSuite bisa:

- Mencegat (_intercept_) request sebelum dikirim
- Melihat isi request dan response
- Memodifikasi data jika diperlukan (misalnya untuk testing keamanan)

Secara sederhana, alurnya bukan lagi:

```
Browser → Server
```

Melainkan:

```
Browser → BurpSuite → Server
```

#### Lokasi Pengaturan Proxy Listener

Untuk mengatur bagaimana BurpSuite menerima traffic, kita menggunakan fitur **Proxy Listener**.

Pengaturannya bisa ditemukan di:

```
Proxy > Proxy Settings
```

Di sinilah kita menentukan:

- IP address tempat BurpSuite “mendengarkan” traffic
- Port yang digunakan

#### Default Listener: 127.0.0.1:8080

Secara default, BurpSuite sudah menyediakan listener dengan konfigurasi:

- Address: `127.0.0.1`
- Port: `8080`

Penjelasannya:

- `127.0.0.1` adalah **loopback address**, yaitu alamat yang menunjuk ke komputer kita sendiri (localhost)
- Port `8080` adalah jalur komunikasi tempat BurpSuite menerima traffic

Artinya, BurpSuite hanya menerima traffic dari komputer lokal — bukan dari jaringan luar — kecuali kita ubah konfigurasinya.

#### Alur Traffic Lengkap

Agar browser benar-benar mengirimkan traffic ke BurpSuite, kita biasanya menggunakan extension seperti FoxyProxy untuk melakukan redirect.

Berikut ilustrasi alurnya:

```
Browser (Firefox)
      ↓
  FoxyProxy (redirect traffic)
      ↓
BurpSuite Proxy Listener (127.0.0.1:8080)
      ↓
  Server Tujuan (misalnya google.com)
```

Penjelasan langkah demi langkah:

1. Browser (misalnya Firefox) membuat request ke sebuah website
2. FoxyProxy mengarahkan request tersebut ke proxy lokal
3. BurpSuite menerima request melalui listener di `127.0.0.1:8080`
4. BurpSuite bisa menahan, melihat, atau memodifikasi request
5. Setelah itu, request diteruskan ke server tujuan
6. Response dari server juga kembali melewati BurpSuite sebelum sampai ke browser

#### Hal Penting: Sinkronisasi Port

Satu hal yang sering menjadi sumber masalah adalah perbedaan konfigurasi port.

Pastikan:

- Port di FoxyProxy = Port di BurpSuite listener

Contoh:

- Jika BurpSuite menggunakan `127.0.0.1:8080`
- Maka FoxyProxy juga harus diarahkan ke `127.0.0.1:8080`

Jika tidak sama, maka:

- Traffic tidak akan sampai ke BurpSuite
- Intercept tidak akan bekerja

Dengan kata lain, koneksi “rantai” tadi akan terputus di tengah.

### b. FoxyProxy — Ekstensi Browser untuk Manajemen Proxy

#### Peran FoxyProxy dalam Alur BurpSuite

FoxyProxy adalah ekstensi browser (khususnya di Firefox) yang berfungsi untuk mengatur dan mengalihkan (_route_) traffic internet melalui proxy tertentu. Dalam konteks penggunaan BurpSuite, FoxyProxy berperan sebagai “penghubung” yang memastikan semua request dari browser diarahkan ke BurpSuite terlebih dahulu.

Tanpa FoxyProxy (atau konfigurasi proxy manual), browser akan langsung mengirim request ke server tujuan. Artinya, BurpSuite tidak akan pernah menerima traffic tersebut, sehingga tidak bisa melakukan intercept atau analisis.

Dengan FoxyProxy, alurnya menjadi:

```
Browser → FoxyProxy → BurpSuite → Server
```

Di sinilah FoxyProxy menjadi komponen penting dalam setup testing.

#### Kenapa Tidak Langsung dari Browser?

Sebenarnya, browser seperti Firefox bisa saja dikonfigurasi proxy secara manual melalui pengaturan network. Namun, FoxyProxy memberikan beberapa keunggulan:

- Lebih praktis untuk _enable/disable_ proxy dengan cepat
- Bisa menyimpan banyak konfigurasi proxy
- Tidak perlu bolak-balik ke pengaturan sistem/browser

Karena itu, FoxyProxy sangat populer di kalangan penetration tester.

#### Langkah Instalasi dan Konfigurasi

Berikut langkah-langkah untuk menginstal dan mengatur FoxyProxy agar terhubung dengan BurpSuite:

1. Buka browser Firefox
2. Masuk ke halaman Add-ons, lalu cari “FoxyProxy”
3. Klik tombol **Add** untuk menginstal ekstensi
4. Setelah terpasang, buka pengaturan FoxyProxy
5. Masuk ke menu **Proxies**, lalu klik **Add a proxy**

Setelah itu, isi konfigurasi proxy seperti berikut:

```
Name     : BurpSuite FoxyProxy
Hostname : 127.0.0.1
Port     : 8080
Username : (kosong)
Password : (kosong)
```

Penjelasan tiap field:

- **Name**: Hanya label untuk memudahkan identifikasi (bebas diisi)
- **Hostname**: Mengarah ke localhost (`127.0.0.1`), yaitu komputer kita sendiri
- **Port**: Harus sama dengan port listener di BurpSuite (default: `8080`)
- **Username/Password**: Tidak diperlukan karena BurpSuite default tidak menggunakan autentikasi proxy

#### Apa yang Terjadi Setelah Disimpan?

Setelah konfigurasi disimpan, FoxyProxy belum otomatis aktif. Ini poin yang sering terlewat.

Agar benar-benar bekerja:

- Pilih proxy yang sudah dibuat
- Pastikan statusnya **enabled/aktif**

Jika belum diaktifkan:

- Browser tetap akan menggunakan koneksi normal (langsung ke internet)
- BurpSuite tidak akan menerima traffic apa pun

#### Kunci Penting: Konsistensi Konfigurasi

Agar semuanya berjalan dengan benar, konfigurasi di FoxyProxy harus **konsisten** dengan BurpSuite:

- Hostname → `127.0.0.1`
- Port → `8080` (atau sesuai listener BurpSuite)

Jika ada perbedaan:

- Request tidak akan sampai ke BurpSuite
- Intercept tidak akan bekerja

Dengan setup yang benar, setiap request dari browser akan selalu melewati BurpSuite, sehingga kita bisa memantau dan memanipulasi traffic dengan leluasa.

### c. Instalasi Sertifikat CA BurpSuite

#### Kenapa Sertifikat CA Dibutuhkan?

Saat BurpSuite digunakan untuk mencegat traffic HTTPS, ia tidak hanya “melewatkan” data, tetapi juga ikut terlibat dalam proses enkripsi. Di sinilah muncul masalah: browser akan mendeteksi bahwa sertifikat yang digunakan bukan berasal dari _Certificate Authority_ (CA) yang dipercaya.

Akibatnya:

- Browser menampilkan peringatan seperti _“Your connection is not secure”_
- Bahkan bisa memblokir akses ke website

Hal ini terjadi karena BurpSuite membuat sertifikat “sendiri” untuk setiap koneksi HTTPS yang dicegat. Agar browser tidak menganggapnya berbahaya, kita perlu memberi tahu browser bahwa sertifikat dari BurpSuite itu **trusted**.

Solusinya adalah menginstal **sertifikat CA BurpSuite** ke dalam browser.

#### Konsep Sederhana Alurnya

Tanpa sertifikat CA:

```id="7n6y2y"
Browser → (tidak percaya) → BurpSuite → Server
```

Dengan sertifikat CA yang sudah dipercaya:

```id="btz6yg"
Browser → (percaya) → BurpSuite → Server
```

Dengan begitu, browser tidak lagi memunculkan error SSL.

#### Langkah Instalasi di Browser (Firefox)

Berikut langkah-langkah yang biasanya dilakukan:

1. Pastikan BurpSuite sudah berjalan dan **proxy listener aktif**
2. Pastikan juga FoxyProxy sudah aktif, sehingga browser terhubung ke BurpSuite
3. Buka browser, lalu akses:

```id="fq2o0r"
http://127.0.0.1:8080
```

Jika konfigurasi benar, akan muncul halaman bawaan BurpSuite

4. Dari halaman tersebut, unduh sertifikat CA (biasanya berformat `.der` atau `.crt`)
5. Masuk ke pengaturan Firefox:
   - **Settings → Privacy & Security**
   - Scroll ke bagian **Certificates**
   - Klik **View Certificates**

6. Klik tombol **Import**
7. Pilih file sertifikat yang tadi diunduh
8. Centang opsi:
   - **"Trust this CA to identify websites"**

9. Klik OK untuk menyimpan

Setelah langkah ini selesai, Firefox akan mempercayai semua sertifikat yang dibuat oleh BurpSuite.

#### Apa yang Terjadi Setelah Instalasi?

Setelah sertifikat CA diimpor:

- Browser tidak lagi menampilkan peringatan SSL saat menggunakan BurpSuite
- Semua traffic HTTPS bisa dicegat dan dibaca oleh BurpSuite
- Pengujian keamanan (seperti melihat request login, token, dll.) bisa dilakukan dengan lancar

#### Instalasi di Level Sistem (Kali Linux)

Selain di browser, sertifikat juga bisa ditambahkan ke sistem operasi, terutama jika kamu menggunakan tools lain selain browser.

Contohnya di Kali Linux:

```bash
# Salin sertifikat ke direktori trusted CA sistem
cp cacert.der /usr/local/share/ca-certificates/burpsuite-ca.crt

# Update daftar CA yang dipercaya
update-ca-certificates
```

Penjelasan:

- File `cacert.der` disalin ke direktori khusus untuk CA terpercaya
- Nama diubah menjadi `.crt` agar dikenali sistem
- Perintah `update-ca-certificates` akan mendaftarkan sertifikat tersebut ke sistem

Setelah itu, aplikasi lain di sistem (misalnya curl, wget, atau tools testing lainnya) juga akan mempercayai sertifikat dari BurpSuite.

#### Catatan Penting

Pada beberapa varian Kali Linux, seperti Kali Purple, tidak tersedia _Certificate Manager_ berbasis GUI. Artinya, proses instalasi sertifikat harus dilakukan secara manual seperti contoh di atas.

Memahami bagian ini penting, karena tanpa sertifikat CA yang terpercaya, proses intercept HTTPS akan selalu terganggu oleh error keamanan dari browser.

### d. Proses Intercept — Mencegat HTTP Request

#### Apa Itu Intercept?

Setelah semua konfigurasi selesai (proxy listener aktif, FoxyProxy terhubung, dan sertifikat CA terpasang), kita masuk ke tahap paling penting: **intercept**.

Intercept adalah proses di mana BurpSuite **menahan request** dari browser sebelum dikirim ke server. Dengan kata lain, request tersebut “diparkir” sementara di BurpSuite agar kita bisa:

- Melihat isinya secara detail
- Mengubah data sebelum dikirim
- Atau bahkan membatalkan request

#### Cara Mengaktifkan Intercept

Langkah-langkahnya cukup sederhana:

1. Buka BurpSuite
2. Masuk ke tab:

```id="8nyzx6"
Proxy > Intercept
```

3. Pastikan tombol menunjukkan:

```id="4q93qv"
Intercept is on
```

Status ini sangat penting. Jika dalam kondisi OFF, maka semua request akan langsung diteruskan tanpa berhenti.

#### Apa yang Terjadi Saat Intercept Aktif?

Sekarang coba lakukan ini:

- Buka Firefox (dengan FoxyProxy aktif)
- Akses sebuah website, misalnya `google.com`

Yang akan terjadi:

- Browser terlihat seperti **freeze / loading terus**
- Halaman tidak kunjung terbuka

Ini bukan error. Justru ini tanda bahwa intercept bekerja.

Kenapa? Karena request dari browser **ditahan oleh BurpSuite** dan belum dikirim ke server.

#### Melihat Request di BurpSuite

Di tab Intercept, kamu akan melihat isi request yang dicegat. Contohnya:

```id="nqoq2s"
GET / HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) ...
Accept: text/html,application/xhtml+xml,...
Accept-Language: en-US,en;q=0.5
Connection: keep-alive
```

Mari kita pahami bagian-bagiannya:

- **GET / HTTP/1.1**
  Menunjukkan metode request (GET) dan path yang diminta (`/`)

- **Host: [www.google.com](http://www.google.com)**
  Menentukan server tujuan

- **User-Agent**
  Informasi tentang browser dan sistem operasi yang digunakan

- **Accept**
  Jenis konten yang bisa diterima browser

- **Accept-Language**
  Preferensi bahasa

- **Connection**
  Mengatur apakah koneksi tetap terbuka atau tidak

Dengan melihat ini, kamu bisa memahami bagaimana browser “berkomunikasi” dengan server.

#### Aksi yang Bisa Dilakukan Saat Intercept

Setelah request ditahan, BurpSuite memberi beberapa opsi penting:

- **Forward**
  Mengirim request ke server tanpa perubahan
  → Browser akan melanjutkan loading dan halaman akan tampil

- **Drop**
  Menghapus request (tidak dikirim ke server sama sekali)
  → Browser biasanya akan gagal memuat halaman

- **Modify (edit manual)**
  Kamu bisa langsung mengubah isi request sebelum dikirim

#### Contoh Modifikasi Request

Misalnya kamu ingin menyamar sebagai browser lain. Kamu bisa mengubah header `User-Agent`.

Dari:

```id="dqnpcu"
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0)
```

Menjadi:

```id="cz4m55"
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
```

Apa efeknya?

- Server akan “mengira” request berasal dari perangkat Windows
- Ini bisa digunakan untuk testing, misalnya melihat apakah server memberikan response berbeda berdasarkan device

#### Inti dari Intercept

Intercept adalah fondasi utama dalam penggunaan BurpSuite. Di sinilah kamu benar-benar:

- Mengontrol alur request
- Memahami struktur komunikasi HTTP
- Melakukan eksperimen terhadap request dan response

Begitu kamu terbiasa dengan fitur ini, kamu bisa lanjut ke teknik yang lebih advanced seperti manipulasi parameter, testing authentication, hingga eksploitasi vulnerability.

### e. GET Request vs POST Request

- **GET Request:** Digunakan untuk mengambil data. Parameter biasanya tampak di URL. Contoh: membuka halaman web biasa.
- **POST Request:** Digunakan untuk mengirim data ke server (misalnya form login, submit data). Parameter ada di dalam _body_ request, tidak terlihat di URL.
- Dalam penetration testing, **POST request** lebih sering menjadi target karena berisi data sensitif seperti kredensial login.

**Ilustrasi perbedaan:**

```
GET Request:
GET /search?q=hacking HTTP/1.1
Host: example.com

POST Request:
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=secret123
```

### f. Mematikan Intercept Setelah Selesai

#### Kenapa Intercept Perlu Dimatikan?

Saat fitur intercept dalam kondisi aktif (**Intercept is on**), BurpSuite akan menahan **semua request** yang keluar dari browser — bukan hanya request yang kamu tuju saja.

Perlu dipahami bahwa browser modern tidak hanya mengirim satu request saat membuka website. Di latar belakang, ada banyak request tambahan seperti:

- Memuat gambar, CSS, dan JavaScript
- Request ke layanan analitik
- Iklan (_ads_)
- Update background dari ekstensi atau service lain

Akibatnya, jika intercept dibiarkan aktif:

- Setiap request akan tertahan di BurpSuite
- Browser akan terlihat terus loading atau bahkan “macet”
- Kamu harus menekan **Forward** berkali-kali untuk setiap request

Ini jelas tidak efisien jika kamu sudah selesai menganalisis request tertentu.

#### Kapan Harus Dimatikan?

Intercept sebaiknya dimatikan ketika:

- Kamu sudah selesai menganalisis atau memodifikasi request tertentu
- Kamu ingin browsing kembali secara normal
- Tidak sedang melakukan pengujian aktif

Dengan kata lain, aktifkan intercept hanya saat dibutuhkan, lalu matikan kembali setelah selesai.

#### Cara Mematikan Intercept

Caranya sangat sederhana:

1. Buka tab:

```id="qevpv1"
Proxy > Intercept
```

2. Klik tombol hingga berubah menjadi:

```id="8h2l0k"
Intercept is off
```

Setelah dimatikan:

- Semua request akan langsung diteruskan ke server
- Browser akan kembali berjalan normal tanpa hambatan

#### Intinya

Intercept adalah fitur yang sangat powerful, tapi juga bisa mengganggu jika tidak digunakan dengan tepat. Mengaktifkan dan mematikan intercept di waktu yang tepat adalah kebiasaan penting yang harus dibangun saat menggunakan BurpSuite.

### g. Target Aplikasi: "Always Broken Web Application" (DVWA / Lab)

#### Apa Itu Aplikasi Web yang Sengaja Rentan?

Dalam proses belajar keamanan web, kita tidak langsung mencoba teknik serangan ke website sembarangan. Sebagai gantinya, digunakan aplikasi khusus yang memang **sengaja dibuat memiliki celah keamanan** (_intentionally vulnerable_).

Tujuannya:

- Memberikan lingkungan aman untuk belajar
- Menghindari pelanggaran hukum atau etika
- Memahami bagaimana vulnerability bekerja secara nyata

Salah satu aplikasi yang paling populer untuk tujuan ini adalah Damn Vulnerable Web Application (DVWA).

#### Mengenal DVWA (Damn Vulnerable Web Application)

DVWA adalah aplikasi web berbasis PHP + MySQL yang dirancang khusus untuk:

- Latihan penetration testing
- Eksperimen berbagai jenis serangan
- Memahami bagaimana vulnerability muncul dan dieksploitasi

Di dalam DVWA, terdapat berbagai fitur yang sengaja dibuat rentan, seperti:

- SQL Injection
- Command Injection
- Cross-Site Scripting (XSS)
- Brute Force login
- File upload vulnerability

Setiap vulnerability biasanya memiliki beberapa level kesulitan (low, medium, high), sehingga kamu bisa belajar secara bertahap.

#### Akses ke Server Lab

Dalam konteks pembelajaran ini, aplikasi DVWA tidak diakses dari internet publik, melainkan dari **server lab lokal**.

Contoh alamat yang digunakan:

```id="jj4a2p"
http://172.17.199.151
```

Artinya:

- Aplikasi berjalan di jaringan lokal (misalnya dalam VM atau container)
- Lebih aman karena tidak terekspos ke internet
- Cocok untuk simulasi environment nyata

#### Peran BurpSuite dalam Lab Ini

Dengan setup BurpSuite + FoxyProxy + sertifikat CA sebelumnya, kamu bisa:

- Mencegat request ke DVWA
- Melihat parameter input (misalnya login form)
- Memodifikasi request untuk mencoba eksploitasi

Contohnya:

- Mengubah parameter login untuk uji brute force
- Menyisipkan payload SQL untuk SQL Injection
- Mengedit request sebelum dikirim ke server

#### Apa yang Akan Dipelajari Selanjutnya?

Bagian ini adalah fondasi sebelum masuk ke praktik yang lebih “menyerang”.

Di tahap berikutnya, biasanya akan didemonstrasikan:

- SQL Injection untuk bypass login
- Brute force attack pada form autentikasi
- Eksploitasi vulnerability lain yang tersedia di DVWA

Dengan kata lain, DVWA berfungsi sebagai “arena latihan”, sedangkan BurpSuite menjadi alat utama untuk menganalisis dan memanipulasi traffic selama proses eksploitasi.

---

## 3. Hubungan Antar Konsep

Semua konsep di atas membentuk satu alur konfigurasi yang saling bergantung:

```
[BurpSuite Listener aktif di 127.0.0.1:8080]
           ↕ (harus sama portnya)
[FoxyProxy dikonfigurasi ke 127.0.0.1:8080]
           ↕ (agar HTTPS tidak error)
[Sertifikat CA BurpSuite diinstall di browser]
           ↕ (traffic dialihkan)
[Browser Firefox mengirim request]
           ↕
[BurpSuite mencegat → bisa dimodifikasi/diteruskan/dibuang]
           ↕
[Server Tujuan menerima request]
```

Jika **salah satu langkah** di atas tidak dilakukan dengan benar — misalnya port tidak cocok, atau sertifikat tidak diinstal — maka intercept tidak akan bekerja, atau browser akan menampilkan error keamanan.

---

## 4. Etika dan Legalitas (Disclaimer Penting)

Video ini secara eksplisit mengingatkan hal-hal berikut sebelum menutup:

> ⚠️ **Selalu dapatkan izin tertulis** sebelum mencegat traffic yang bukan milik Anda.

- ❌ **Jangan** intercept traffic pihak ketiga atau traffic pengguna produksi (_production user traffic_) — ini bisa melanggar hukum privasi.
- ❌ **Jangan** menyimpan data sensitif seperti password atau token dari pengguna nyata.
- ✅ **Lakukan semua ini di lingkungan lab yang terkontrol** — seperti menggunakan DVWA atau aplikasi latihan serupa.
- ✅ Dalam konteks video ini, semua GET dan POST request yang dicegat adalah **milik praktisi sendiri**, sehingga legal.

---

## 5. Kesimpulan

BurpSuite Part 2 menunjukkan secara praktis bagaimana mengonfigurasi _intercept proxy_ menggunakan BurpSuite bersama Firefox. Tiga pilar konfigurasi utamanya adalah: (1) **Proxy listener** di BurpSuite pada `127.0.0.1:8080`, (2) **FoxyProxy** di Firefox yang mengarahkan traffic ke listener tersebut, dan (3) **Sertifikat CA BurpSuite** yang diinstal di browser agar HTTPS bisa dicegat tanpa error. Setelah semuanya terkonfigurasi, praktisi dapat melihat, memodifikasi, meneruskan, atau membuang request HTTP/HTTPS secara real-time — kemampuan inti yang sangat dibutuhkan dalam penetration testing web.

---

_Sumber: Subtitle video "BurpSuite - Part 2" | Dibuat untuk keperluan belajar_
