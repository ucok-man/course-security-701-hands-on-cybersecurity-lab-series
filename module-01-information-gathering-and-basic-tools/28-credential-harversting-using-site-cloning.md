# 📝 Credential Harvesting Menggunakan Site Cloning

---

## 1. Ringkasan Singkat

Video ini merupakan demonstrasi lab yang menunjukkan bagaimana serangan **credential harvesting** (pemanenan kredensial) dilakukan menggunakan teknik **site cloning** (penggandaan situs). Alat yang digunakan adalah **Social Engineer Toolkit (SET)** yang berjalan di **Kali Linux**. Target dalam lab ini adalah aplikasi web yang rentan (_broken web application_) dan Instagram sebagai contoh situs yang dikloning.

Tujuan utama video ini bersifat edukatif — yaitu memperlihatkan **bagaimana penyerang mengumpulkan kredensial** dan, yang lebih penting, **bagaimana para defender dapat mendeteksi dan memitigasi ancaman ini**.

> ⚠️ **Peringatan:** Teknik ini hanya boleh dipraktikkan pada lab/lingkungan yang Anda kendalikan sendiri. Melakukannya pada sistem orang lain tanpa izin adalah tindakan ilegal.

---

## 2. Konsep Utama

### a. Social Engineer Toolkit (SET)

#### Apa itu SET?

**Social Engineer Toolkit (SET)** adalah sebuah framework open-source berbasis Python yang dibuat khusus untuk kebutuhan _penetration testing_, terutama yang berfokus pada teknik _social engineering_. Artinya, SET tidak hanya menyerang sistem dari sisi teknis, tetapi juga memanfaatkan kelemahan manusia sebagai target.

Biasanya, tool ini sudah tersedia secara default di sistem operasi seperti Kali Linux. Namun, pada varian tertentu seperti Kali Purple, SET tidak langsung tersedia sehingga perlu diinstal secara manual.

#### Cara Instalasi

Jika SET belum terpasang di sistem, kamu bisa menginstalnya dengan perintah berikut:

```bash
sudo apt install set
# atau
sudo setoolkit
```

Penjelasan:

- `sudo apt install set` digunakan untuk mengunduh dan menginstal paket SET dari repository resmi.
- `sudo setoolkit` kadang juga bisa langsung memicu instalasi atau menjalankan tool jika sudah tersedia.

Setelah instalasi selesai, kamu bisa menjalankan SET dengan perintah:

```bash
sudo setoolkit
```

#### Modul-Modul Utama dalam SET

SET menyediakan berbagai modul yang siap digunakan untuk mensimulasikan serangan _social engineering_. Berikut beberapa modul pentingnya:

- **Spearphishing Attack Vectors**
  Digunakan untuk membuat serangan phishing yang lebih terarah (target spesifik), biasanya melalui email dengan payload berbahaya.

- **Social Engineering Attacks**
  Kumpulan teknik umum social engineering, seperti manipulasi pengguna agar memberikan informasi sensitif.

- **Credential Harvester Attacks**
  Modul ini memungkinkan kamu membuat halaman login palsu untuk mencuri kredensial (username & password).

- **Mass Mailer Attacks**
  Digunakan untuk mengirim email dalam jumlah besar (bulk email), biasanya untuk kampanye phishing.

- **Website Attack Vectors**
  Menyediakan berbagai metode serangan berbasis web, seperti cloning website untuk phishing.

- **Wireless Attacks**
  Fokus pada serangan terhadap jaringan wireless (Wi-Fi), termasuk skenario social engineering berbasis jaringan.

- **PowerShell Attack Vectors**
  Digunakan untuk menjalankan payload berbasis PowerShell, sering dipakai dalam serangan pada sistem Windows.

#### Gambaran Alur Penggunaan

Secara umum, penggunaan SET mengikuti alur seperti ini:

1. Menjalankan SET dari terminal
2. Memilih jenis serangan (misalnya phishing atau credential harvesting)
3. Mengatur target dan metode serangan
4. Menjalankan payload atau halaman palsu
5. Mengumpulkan data dari korban (jika berhasil)

Pendekatannya sangat interaktif—SET akan memandu kamu melalui menu berbasis teks, sehingga cukup mudah digunakan bahkan untuk pemula di bidang _penetration testing_.

### b. Credential Harvesting

#### Apa itu Credential Harvesting?

**Credential harvesting** adalah teknik dalam _social engineering_ di mana penyerang mencoba menipu korban agar secara sukarela memasukkan data sensitif seperti username dan password ke dalam sebuah halaman palsu.

Halaman palsu ini biasanya dibuat semirip mungkin dengan situs asli—baik dari segi tampilan, logo, hingga struktur halaman—sehingga korban sulit membedakan mana yang asli dan mana yang palsu. Ketika korban memasukkan kredensialnya, data tersebut tidak dikirim ke server resmi, melainkan ke server milik penyerang.

#### Bagaimana Cara Kerjanya?

Konsep utama dari credential harvesting sebenarnya cukup sederhana: **meniru tampilan, mengelabui pengguna, lalu menangkap data yang dimasukkan**.

Berikut alur serangannya dalam bentuk yang lebih mudah dipahami:

```
[Penyerang]                         [Korban]
    |                                   |
    |-- Kloning situs asli (misal IG) ->|
    |-- Hosting di IP lokal penyerang ->|
    |-- Bagikan link ke korban -------->|
    |<-- Korban isi username+password --|
    |   (POST request tertangkap)       |
    |-- Kredensial berhasil dicuri! ----|
```

#### Penjelasan Tiap Langkah

Mari kita breakdown alurnya supaya lebih jelas:

1. **Kloning situs asli**
   Penyerang membuat salinan dari halaman login suatu layanan (misalnya Instagram, Facebook, atau email). Biasanya ini dilakukan menggunakan tools seperti SET atau metode manual.

2. **Hosting halaman palsu**
   Halaman hasil kloning tersebut kemudian dijalankan di server milik penyerang, bisa berupa:
   - Localhost (untuk testing)
   - VPS atau hosting publik (agar bisa diakses korban)

3. **Menyebarkan link ke korban**
   Penyerang mengirimkan link halaman palsu ini ke target, biasanya melalui:
   - Email (phishing)
   - Chat (WhatsApp, Telegram, dll)
   - Media sosial

   Link ini sering dibuat menyerupai URL asli agar terlihat meyakinkan.

4. **Korban memasukkan kredensial**
   Karena tampilan halaman terlihat asli, korban tidak curiga dan memasukkan username serta password mereka.

5. **Data tertangkap melalui POST request**
   Saat korban menekan tombol login, data yang dimasukkan dikirim menggunakan metode HTTP POST—tetapi bukan ke server resmi, melainkan ke server penyerang.

6. **Kredensial berhasil dikumpulkan**
   Penyerang kini memiliki akses ke username dan password korban, yang bisa digunakan untuk berbagai tujuan lanjutan.

#### Kenapa Teknik Ini Efektif?

Credential harvesting sangat efektif karena:

- Tidak mengeksploitasi bug teknis, tetapi **kelemahan manusia**
- Tampilan halaman yang identik membuat korban percaya
- Prosesnya cepat dan bisa ditargetkan ke banyak orang sekaligus

Karena itu, teknik ini sering digunakan dalam skenario _penetration testing_ untuk menguji seberapa sadar pengguna terhadap ancaman phishing.

### c. Website Attack Vectors dalam SET

#### Jalur Menu Menuju Site Cloning

Di dalam **SET (Social Engineer Toolkit)**, semua fitur disusun dalam bentuk menu interaktif berbasis teks. Untuk mencapai fitur _site cloning_ (yang digunakan dalam credential harvesting), kamu perlu mengikuti alur navigasi berikut:

```
SET Menu
└── 1. Social Engineering Attacks
    └── 2. Website Attack Vectors
        └── 3. Credential Harvester Attack Method
            └── 2. Site Cloner
```

Setiap langkah dalam menu ini memiliki peran tertentu:

- **Social Engineering Attacks**
  Ini adalah kategori utama yang berisi berbagai teknik manipulasi pengguna.

- **Website Attack Vectors**
  Berisi metode serangan yang memanfaatkan website, termasuk phishing berbasis halaman login palsu.

- **Credential Harvester Attack Method**
  Fokus pada teknik untuk menangkap kredensial yang dimasukkan oleh korban.

- **Site Cloner**
  Fitur yang memungkinkan kamu menyalin (clone) tampilan sebuah website secara otomatis.

Dengan mengikuti jalur ini, kamu akan masuk ke mode di mana SET membantu membuat halaman phishing berbasis cloning website target.

#### Tiga Metode dalam Credential Harvester

Di dalam menu **Credential Harvester**, terdapat tiga metode utama yang bisa dipilih, masing-masing dengan pendekatan berbeda:

| No  | Metode        | Penjelasan                                                                                                                                                  |
| --- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Web Templates | Menggunakan template halaman login yang sudah disediakan oleh SET. Cocok untuk penggunaan cepat tanpa perlu konfigurasi tambahan.                           |
| 2   | Site Cloner   | Meng-clone website target secara otomatis berdasarkan URL yang diberikan. Ini adalah metode yang paling realistis karena meniru situs asli secara langsung. |
| 3   | Custom Import | Menggunakan halaman buatan sendiri (custom), biasanya untuk skenario yang lebih spesifik atau kebutuhan testing tertentu.                                   |

#### Penjelasan Lebih Dalam Tiap Metode

**Web Templates**
Metode ini paling sederhana. SET sudah menyediakan beberapa template login populer (misalnya halaman login email atau media sosial). Kamu hanya perlu memilih template, lalu SET akan langsung menjalankan server phishing.

**Site Cloner**
Ini adalah metode yang paling sering digunakan. Kamu cukup memasukkan URL target (misalnya halaman login), lalu SET akan:

- Mengambil konten HTML dari situs tersebut
- Menyesuaikan form agar data yang dikirim masuk ke server penyerang
- Menyajikan halaman yang tampak identik dengan aslinya

Hasilnya, korban akan melihat halaman yang sangat meyakinkan.

**Custom Import**
Jika kamu ingin kontrol penuh, metode ini memungkinkan kamu menggunakan file HTML sendiri. Biasanya digunakan ketika:

- Ingin membuat skenario phishing yang sangat spesifik
- Ingin menguji halaman yang tidak bisa di-clone dengan baik oleh SET

#### Gambaran Penggunaan dalam Praktik

Dalam praktik _penetration testing_, pemilihan metode tergantung pada kebutuhan:

- Jika ingin cepat → gunakan **Web Templates**
- Jika ingin realistis → gunakan **Site Cloner**
- Jika butuh fleksibilitas tinggi → gunakan **Custom Import**

Dengan memahami perbedaan ini, kamu bisa memilih pendekatan yang paling sesuai dengan skenario pengujian yang sedang dilakukan.

### d. Site Cloner — Cara Kerja Teknis

#### Apa itu Site Cloner?

**Site Cloner** adalah metode dalam SET yang digunakan untuk menyalin (clone) tampilan sebuah website target secara otomatis, lalu menjalankannya di mesin penyerang. Hasilnya adalah halaman palsu yang secara visual sangat mirip dengan situs asli.

Ketika korban membuka halaman tersebut dan memasukkan data login, informasi yang dimasukkan tidak dikirim ke server resmi, melainkan ke mesin penyerang melalui **HTTP POST request**.

#### Konsep Cara Kerja Secara Teknis

Agar lebih mudah dipahami, bayangkan alurnya seperti ini:

- SET mengambil tampilan website target
- Form login di dalamnya dimodifikasi
- Semua input pengguna diarahkan ke server penyerang

Jadi dari sisi korban:

- Halaman terlihat normal
- Tidak ada indikasi mencurigakan
- Namun di belakang layar, alur pengiriman data sudah diubah

#### Langkah-Langkah Konfigurasi

##### 1. Menemukan IP Address Mesin Penyerang

Langkah pertama adalah mengetahui alamat IP dari mesin yang digunakan untuk menjalankan SET. IP ini nantinya akan menjadi tujuan pengiriman data dari halaman palsu.

Gunakan perintah berikut:

```bash
ifconfig
# Catat IP address (misal: 192.168.1.10)
```

Penjelasan:

- `ifconfig` menampilkan konfigurasi jaringan yang aktif
- Kamu perlu mencari IP lokal, biasanya berada di interface seperti `eth0` atau `wlan0`
- Contoh hasilnya adalah `192.168.1.10`

##### 2. Memasukkan IP ke dalam SET

Setelah mendapatkan IP address, masukkan IP tersebut saat SET memintanya.

Fungsinya:

- Sebagai endpoint tempat data login akan dikirim
- SET akan “mendengarkan” data yang masuk ke IP tersebut

Dengan kata lain, semua kredensial korban akan diarahkan ke sini.

##### 3. Menentukan URL Target

Selanjutnya, kamu diminta memasukkan URL website yang ingin di-clone:

```
URL to clone: https://www.instagram.com
```

Penjelasan:

- SET akan mengakses URL tersebut
- Mengambil struktur HTML dan elemen penting lainnya
- Biasanya yang ditarget adalah halaman login

##### 4. Proses Cloning dan Menjalankan Web Server

Setelah URL dimasukkan:

- SET mulai mengunduh konten dari website target
- Form login dimodifikasi agar mengarah ke IP penyerang
- Web server lokal dijalankan secara otomatis

Pada tahap ini, halaman phishing sudah aktif.

##### 5. Mengakses Halaman Hasil Cloning

Untuk memastikan semuanya berjalan dengan baik, buka browser dan akses IP mesin penyerang:

```
http://192.168.1.10
```

Jika berhasil:

- Halaman yang tampil akan terlihat seperti website asli
- Secara visual hampir tidak ada perbedaan

##### 6. Menangkap Data Login Korban

Ketika korban:

- Mengakses halaman tersebut
- Memasukkan username dan password
- Menekan tombol login

Maka yang terjadi:

- Data dikirim menggunakan metode POST
- Tujuan pengiriman adalah mesin penyerang
- SET menangkap dan menampilkan data tersebut di terminal

Contoh output di terminal:

```bash
[*] WE GOT A HIT!
Username: example_user
Password: example_pass
```

Penjelasan:

- Pesan **WE GOT A HIT!** menandakan ada data yang berhasil ditangkap
- Username dan password ditampilkan dalam bentuk plaintext (tidak terenkripsi)

#### Inti yang Perlu Dipahami

Hal terpenting dari teknik ini adalah:

- Tidak menyerang sistem target secara langsung
- Melainkan memanipulasi interaksi pengguna

Selama korban percaya bahwa halaman tersebut asli, maka proses pengambilan kredensial bisa berjalan tanpa hambatan.

### e. Typosquatting & Domain Spoofing

#### Apa itu Typosquatting dan Domain Spoofing?

Dalam praktik serangan _phishing_ di dunia nyata, penyerang jarang hanya mengandalkan halaman palsu saja. Mereka biasanya menambahkan teknik lain agar serangan terlihat lebih meyakinkan. Dua teknik yang paling umum digunakan adalah **typosquatting** dan **domain spoofing**.

- **Typosquatting** adalah teknik membuat domain yang mirip dengan domain asli, biasanya dengan memanfaatkan kesalahan ketik pengguna.
- **Domain spoofing** adalah teknik memalsukan tampilan domain agar terlihat identik atau sangat mirip dengan domain resmi.

Tujuan utama dari kedua teknik ini sama: **membuat korban percaya bahwa mereka sedang mengakses situs yang sah**.

#### Kenapa Teknik Ini Efektif?

Sebagian besar pengguna tidak membaca URL secara detail. Mereka hanya melihat sekilas:

- Nama domain terlihat familiar
- Tampilan website terlihat benar

Padahal, perbedaan kecil pada URL bisa berarti situs tersebut palsu.

#### Teknik-Teknik yang Umum Digunakan

##### 1. Mengganti Huruf dengan Angka

Teknik ini mengganti huruf tertentu dengan angka yang bentuknya mirip.

Contoh:

```
instagram.com  →  1nstagram.com
```

Penjelasan:

- Huruf **"i"** diganti dengan angka **"1"**
- Secara sekilas hampir tidak terlihat berbeda
- Terutama jika dilihat cepat atau di layar kecil

##### 2. Homograph Attack (Karakter Mirip)

Teknik ini menggunakan karakter dari alfabet lain (misalnya Cyrillic) yang bentuknya sangat mirip dengan huruf Latin.

Contoh:

```
facebook.com  →  facebооk.com
```

Penjelasan:

- Huruf **"o"** diganti dengan karakter dari alfabet Cyrillic
- Secara visual hampir identik
- Tapi secara teknis adalah karakter yang berbeda, sehingga domainnya juga berbeda

Ini disebut juga sebagai **homograph attack**, karena memanfaatkan kemiripan bentuk karakter.

##### 3. Menggabungkan Karakter yang Terlihat Seperti Huruf Lain

Teknik ini memanfaatkan kombinasi dua huruf yang jika berdampingan terlihat seperti huruf lain.

Contoh konsep:

```
rn  → terlihat seperti m
```

Contoh penerapan:

```
citibank.com  →  (menggunakan "rn" agar terlihat seperti "m")
```

Penjelasan:

- Huruf **"r"** dan **"n"** jika berdekatan bisa terlihat seperti **"m"**
- Dalam font tertentu, perbedaannya sangat sulit dibedakan
- Ini bisa digunakan untuk membuat domain yang tampak valid

Contoh visual sederhana:

```
rn  →  m
```

Jika tidak diperhatikan dengan teliti, pengguna bisa dengan mudah tertipu.

#### Inti yang Perlu Dipahami

Teknik-teknik ini tidak bergantung pada celah teknis sistem, tetapi pada:

- Persepsi visual manusia
- Kebiasaan pengguna yang jarang memeriksa URL secara detail

Ketika digabungkan dengan teknik seperti _credential harvesting_ atau _site cloning_, typosquatting dan domain spoofing bisa membuat serangan menjadi jauh lebih meyakinkan dan sulit dideteksi.

### f. Mendeteksi Hasil Harvesting di Terminal SET

#### Apa yang Terjadi Setelah Korban Login?

Setelah korban mengakses halaman hasil cloning dan memasukkan data login, proses berikutnya terjadi secara otomatis di sisi penyerang.

Saat korban menekan tombol login:

- Browser mengirimkan data melalui **HTTP POST request**
- Karena halaman sudah dimodifikasi, request tidak menuju server asli
- Melainkan dikirim ke mesin yang menjalankan SET

Di sinilah SET mulai bekerja sebagai “penangkap” data tersebut.

#### Tampilan Hasil di Terminal

Begitu ada data yang masuk, SET akan langsung menampilkan hasilnya di terminal. Contohnya seperti ini:

```id="d4czow"
[*] WE GOT A HIT! Printing the output:
POSSIBLE USERNAME FIELD FOUND: username=xyz_test_he2get77
POSSIBLE PASSWORD FIELD FOUND: password=**********

[*] WHEN YOU'RE FINISHED, HIT CONTROL-C TO GENERATE A REPORT.
```

#### Penjelasan Output

Mari kita bahas bagian-bagian penting dari output tersebut:

- **WE GOT A HIT!**
  Ini adalah indikator bahwa ada seseorang (korban) yang berhasil mengirimkan data melalui form. Dengan kata lain, halaman phishing sudah “berhasil digunakan”.

- **POSSIBLE USERNAME FIELD FOUND**
  SET mendeteksi field yang kemungkinan berisi username. Nilai yang ditampilkan adalah data yang dikirim oleh korban.

- **POSSIBLE PASSWORD FIELD FOUND**
  Sama seperti username, ini adalah field password.
  Terkadang password ditampilkan dalam bentuk tersamarkan (`**********`), tetapi dalam beberapa konfigurasi bisa juga terlihat dalam bentuk plaintext.

- **CONTROL-C TO GENERATE A REPORT**
  Setelah selesai, kamu bisa menekan `CTRL + C` untuk menghentikan proses dan biasanya SET akan membuat laporan hasil tangkapan.

#### Kenapa Data Bisa Tertangkap?

Hal ini terjadi karena perubahan kecil namun krusial pada halaman hasil cloning:

- Form login yang asli seharusnya mengirim data ke server resmi
- Namun pada halaman palsu, action form sudah diubah
- Sehingga data dikirim ke server lokal (mesin penyerang)

Secara teknis:

- Browser tetap menjalankan POST request seperti biasa
- Tapi endpoint-nya berbeda
- SET membaca dan menampilkan isi request tersebut

#### Inti yang Perlu Dipahami

Yang penting untuk dipahami di sini adalah:

- SET tidak “membobol” server target
- Ia hanya menangkap data yang dikirim secara sukarela oleh pengguna

Karena itu, semua informasi yang muncul di terminal sebenarnya adalah hasil dari interaksi korban dengan halaman yang terlihat sah, padahal sudah dimodifikasi.

## 3. Hubungan Antar Konsep

Berikut adalah bagaimana semua konsep di atas saling terhubung dalam satu alur serangan:

```
[Social Engineering] ──► Teknik manipulasi psikologis untuk menipu korban
        │
        ▼
[SET - Social Engineer Toolkit] ──► Framework yang menyediakan tool otomatis
        │
        ▼
[Website Attack Vectors] ──► Kategori serangan yang memanfaatkan situs web palsu
        │
        ▼
[Site Cloning] ──► Membuat salinan identik dari situs asli
        │
        ▼
[Credential Harvesting] ──► Menangkap POST request berisi username & password
        │
        ▼
[Typosquatting/Domain Spoofing] ──► Membuat skenario lebih realistis & meyakinkan
```

Inti dari serangan ini adalah menggabungkan **rekayasa sosial** (membuat korban percaya) dengan **teknik teknis** (cloning & capturing) untuk mencuri informasi sensitif tanpa disadari korban.

---

## 4. Kesimpulan

Serangan **credential harvesting via site cloning** adalah salah satu bentuk serangan social engineering yang relatif mudah dilakukan namun sangat efektif. Berikut poin-poin kunci dari materi ini:

1. **SET (Social Engineer Toolkit)** menyediakan fitur lengkap untuk melakukan site cloning hanya dengan beberapa langkah konfigurasi sederhana.
2. **Site Cloner** bekerja dengan meng-clone halaman login situs target dan menangkap POST request yang dikirimkan korban.
3. Penyerang dapat membuat serangan lebih meyakinkan dengan teknik **typosquatting** — membeli atau membuat domain yang menyerupai domain asli.
4. **Defender** perlu meningkatkan kesadaran pengguna (**security awareness**) agar selalu memeriksa URL dengan teliti sebelum memasukkan kredensial.
5. Penggunaan **Multi-Factor Authentication (MFA/OTP)** menjadi lapisan perlindungan penting, karena meskipun kredensial berhasil dicuri, penyerang masih perlu melewati verifikasi tambahan.

> 🛡️ **Sisi Pertahanan:** Deteksi serangan ini dapat dilakukan melalui monitoring DNS, filtering URL mencurigakan di level gateway, dan pelatihan phishing awareness secara berkala kepada seluruh pengguna organisasi.

---

_Catatan ini dibuat berdasarkan subtitle video demonstrasi lab "Credential Harvesting Using Site Cloning" untuk tujuan pembelajaran keamanan siber._
