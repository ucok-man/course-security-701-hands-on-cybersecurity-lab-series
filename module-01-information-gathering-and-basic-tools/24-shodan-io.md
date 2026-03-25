# 📝 Shodan.IO — Search Engine untuk Perangkat Internet

---

## 1. Ringkasan Singkat

Video ini membahas **Shodan.IO**, sebuah alat _passive reconnaissance_ yang sangat powerful bagi para profesional keamanan siber dan _ethical hacker_. Berbeda dengan mesin pencari biasa seperti Google yang mengindeks konten web, Shodan mengindeks **perangkat yang terhubung ke internet** beserta metadata layanan yang mereka jalankan. Pengguna dapat menemukan perangkat yang terekspos, mengidentifikasi layanan yang rentan, serta menilai postur keamanan suatu target — semuanya **tanpa mengirim satu paket pun langsung ke sistem target**.

---

## 2. Konsep Utama

### a. Apa itu Shodan?

#### Gambaran Umum

Shodan adalah sebuah **mesin pencari khusus** yang dirancang untuk menemukan berbagai perangkat yang terhubung ke internet. Jika biasanya kita mengenal mesin pencari seperti Google atau Bing untuk mencari website, artikel, atau gambar, maka Shodan bekerja dengan cara yang berbeda.

Shodan sering disebut sebagai _“search engine for the Internet of Things”_, karena fokus utamanya adalah pada **perangkat**, bukan konten.

Slogan dari Shodan adalah:
_"Discover how internet intelligence can help you make better decisions."_

Artinya, Shodan membantu kita memahami kondisi perangkat di internet sehingga bisa digunakan untuk analisis, keamanan, maupun pengambilan keputusan teknis.

#### Apa yang Diindeks oleh Shodan?

Berbeda dengan mesin pencari biasa, Shodan tidak mengindeks halaman web seperti HTML atau media. Sebaliknya, Shodan mengumpulkan informasi dari **layanan yang berjalan pada perangkat yang terbuka ke internet**.

Beberapa contoh perangkat yang bisa ditemukan melalui Shodan:

- Router
- Webcam (CCTV online)
- Server (web server, database, dll.)
- Sistem industri seperti ICS (_Industrial Control Systems_)
- Perangkat IoT (_Internet of Things_)
- Dan berbagai perangkat lain yang memiliki IP publik

#### Cara Kerja Sederhana

Shodan bekerja dengan cara melakukan scanning ke berbagai alamat IP di internet, lalu:

1. Mengakses port yang terbuka (misalnya 80, 443, 22, dll.)
2. Mengambil informasi dari layanan yang berjalan
3. Menyimpan data tersebut dalam bentuk **banner**

Banner ini biasanya berisi informasi seperti:

- Versi software
- Jenis server
- Lokasi IP
- Konfigurasi tertentu

Dari sinilah Shodan bisa “menampilkan” perangkat-perangkat tersebut saat kita melakukan pencarian.

#### Perbandingan dengan Mesin Pencari Tradisional

Untuk memahami perbedaannya, perhatikan tabel berikut:

| Mesin Pencari | Fokus yang Diindeks                                       |
| ------------- | --------------------------------------------------------- |
| Google / Bing | Konten web (HTML, gambar, video, tautan)                  |
| **Shodan**    | **Perangkat & layanan internet (banner, metadata, port)** |

#### Intinya

Kalau Google membantu kita menemukan **informasi di web**, maka Shodan membantu kita menemukan **perangkat yang ada di balik internet itu sendiri**.

Ini membuat Shodan sangat powerful, terutama dalam konteks:

- Cybersecurity (mencari celah keamanan)
- Penelitian jaringan
- OSINT (Open Source Intelligence)

Namun, karena sifatnya yang sensitif, penggunaan Shodan juga harus dilakukan secara bijak dan etis.

### b. Cara Shodan Mengumpulkan Data

#### Gambaran Proses Secara Umum

Agar bisa menampilkan informasi tentang perangkat di internet, Shodan tidak sekadar “mencari” seperti mesin pencari biasa. Ia **secara aktif mengumpulkan data terlebih dahulu**, lalu menyimpannya dalam database yang bisa di-query kapan saja oleh pengguna.

Proses ini berjalan otomatis dan terus diperbarui, sehingga data yang tersedia relatif up-to-date.

Secara garis besar, ada tiga tahap utama dalam cara kerja Shodan: scanning, pengambilan banner, dan indexing.

#### 1. Scanning IP Address

Tahap pertama adalah **melakukan scanning ke seluruh alamat IP di internet**, khususnya pada ruang alamat **IPv4**.

Artinya, Shodan mencoba menghubungi banyak alamat IP publik secara rutin untuk melihat:

- Apakah IP tersebut aktif
- Port apa saja yang terbuka

Setiap IP yang aktif akan diperiksa beberapa port umum, karena port tersebut biasanya digunakan oleh layanan tertentu.

Contohnya:

- Port 80 → biasanya untuk web server (HTTP)
- Port 22 → untuk SSH
- Port 443 → untuk HTTPS

Dari sini, Shodan mulai “mengintip” layanan apa yang tersedia di sebuah perangkat.

#### 2. Service Detection & Banner Recording

Setelah menemukan IP dan port yang terbuka, langkah berikutnya adalah **mengidentifikasi layanan yang berjalan** dan merekam responsnya.

Respons ini disebut sebagai **banner**.

Banner adalah teks yang dikirim oleh server saat kita mencoba terhubung ke suatu layanan. Banner ini sering kali mengandung informasi penting seperti:

- Nama software
- Versi software
- Konfigurasi tertentu

Contoh sederhana:
Jika sebuah server menjalankan FTP di port 21, maka ketika diakses, server bisa memberikan respons seperti:

```
220 FTP Server (vsFTPd 3.0.3)
```

Dari satu baris ini saja, kita sudah tahu:

- Layanannya: FTP
- Software-nya: vsFTPd
- Versinya: 3.0.3

Beberapa contoh port yang umum direkam oleh Shodan:

- `Port 21` → FTP
- `Port 22` → SSH
- `Port 23` → Telnet
- `Port 80` → HTTP
- `Port 443` → HTTPS
- `Port 3389` → RDP (Remote Desktop)

Semua informasi ini dikumpulkan tanpa harus masuk ke sistem — cukup dari respons awal layanan.

#### 3. Indexing / Penyimpanan Metadata

Setelah data berhasil dikumpulkan, Shodan tidak langsung menampilkannya begitu saja. Data tersebut akan disimpan ke dalam **database yang terstruktur dan bisa dicari**.

Di tahap ini, informasi dari banner akan diproses menjadi metadata yang lebih lengkap.

Beberapa jenis metadata yang disimpan antara lain:

- Versi layanan (misalnya Apache 2.4.49)
- Jenis perangkat (router, kamera, server, dll.)
- Organisasi atau ISP pemilik IP
- Lokasi geografis (negara, kota)
- Informasi SSL/TLS (sertifikat HTTPS)
- Indikasi misconfiguration (kesalahan konfigurasi yang bisa membuka celah)

Dengan adanya indexing ini, pengguna bisa melakukan pencarian seperti:

- mencari server dengan versi tertentu
- mencari perangkat di negara tertentu
- mencari layanan yang terbuka di port tertentu

#### Alur Kerja Shodan Secara Keseluruhan

Jika dirangkum, alur kerja Shodan terlihat seperti ini:

```
[Scanning IPv4]
      ↓
[Deteksi layanan: IP + Port + Banner]
      ↓
[Indexing ke database Shodan]
      ↓
[Pengguna melakukan query (tanpa scanning sendiri)]
```

#### Kenapa Ini Penting?

Hal penting yang perlu dipahami adalah:
Saat kita menggunakan Shodan, kita **tidak sedang melakukan scanning langsung ke target**.

Sebaliknya, kita hanya:

- Mengakses data yang sudah dikumpulkan sebelumnya
- Melakukan query ke database Shodan

Inilah yang membuat Shodan sangat cepat dan powerful, terutama untuk:

- Reconnaissance (pengumpulan informasi awal)
- OSINT
- Analisis keamanan jaringan

Karena semua proses berat (scanning dan pengumpulan data) sudah dilakukan di belakang layar oleh Shodan.

### c. Passive vs Active Reconnaissance

#### Memahami Dua Pendekatan Utama

Dalam dunia keamanan jaringan dan OSINT, ada dua pendekatan utama saat melakukan pengumpulan informasi, yaitu **passive reconnaissance** dan **active reconnaissance**. Memahami perbedaan keduanya sangat penting, terutama ketika menggunakan tools seperti Shodan.

Kedua metode ini sebenarnya memiliki tujuan yang sama, yaitu mengumpulkan informasi tentang target. Yang membedakan adalah **cara kita mendapatkan informasi tersebut**.

#### Passive Reconnaissance (Recon Pasif)

Passive reconnaissance adalah proses mengumpulkan informasi **tanpa melakukan interaksi langsung dengan target**.

Dalam konteks Shodan, saat kita melakukan pencarian, kita hanya:

- Mengirim query ke database Shodan
- Mengambil data yang **sudah dikumpulkan sebelumnya**

Kita **tidak melakukan scanning** ke server target, tidak mengirim request langsung ke IP target, dan tidak meninggalkan jejak di sistem mereka.

Contoh sederhana:
Misalnya kita mencari:

```
apache country:"ID"
```

Artinya:

- Kita meminta Shodan menampilkan server dengan Apache di Indonesia
- Data tersebut berasal dari hasil scanning Shodan sebelumnya
- Kita tidak menghubungi server tersebut secara langsung

Hasilnya:

- Target tidak tahu bahwa kita sedang “mengintip” mereka
- Aktivitas kita bersifat **tidak terlihat (invisible)**

Inilah yang membuat passive reconnaissance relatif lebih aman dan sering digunakan pada tahap awal reconnaissance.

#### Active Reconnaissance (Recon Aktif)

Berbeda dengan passive, active reconnaissance adalah proses mengumpulkan informasi dengan **berinteraksi langsung dengan target**.

Artinya, kita sendiri yang melakukan:

- Port scanning
- Service probing
- Atau request langsung ke server target

Contoh:
Menggunakan tools seperti nmap:

```id="k3hs82"
nmap -p 80,443 192.168.1.1
```

Perintah ini akan:

- Mengirim request langsung ke IP target
- Mengecek apakah port 80 dan 443 terbuka
- Menganalisis respons dari server

Konsekuensinya:

- Target bisa mendeteksi aktivitas ini
- Bisa tercatat di log server atau firewall
- Bahkan bisa dianggap sebagai aktivitas mencurigakan

#### Perbandingan Singkat

Secara sederhana, perbedaannya bisa dipahami seperti ini:

- Passive Reconnaissance:
  - Tidak kontak langsung dengan target
  - Menggunakan data yang sudah ada
  - Tidak terdeteksi oleh target

- Active Reconnaissance:
  - Berinteraksi langsung dengan target
  - Menghasilkan data secara real-time
  - Bisa terdeteksi

#### Kesimpulan

Shodan termasuk dalam kategori **passive reconnaissance**, karena kita hanya mengakses data yang sudah dikumpulkan oleh sistem mereka.

Artinya, saat menggunakan Shodan:

- Kita tetap “di balik layar”
- Tidak mengirim traffic ke target
- Tidak mudah terdeteksi

Inilah alasan kenapa Shodan sangat powerful untuk tahap awal reconnaissance: kita bisa mendapatkan banyak informasi tanpa harus menyentuh target secara langsung.

### d. Cara Menggunakan Shodan (Antarmuka Web)

#### Akses Shodan

Shodan adalah **aplikasi berbasis web**, jadi kita tidak perlu install tools khusus seperti pada command-line. Cukup buka browser dan akses:

[Shodan](https://www.shodan.io?utm_source=chatgpt.com)

Setelah masuk, kita akan melihat tampilan utama dengan search bar. Dari sinilah semua proses pencarian dilakukan.

#### 1. Pencarian Dasar

Cara paling sederhana menggunakan Shodan adalah dengan mengetik **nama layanan atau software** di search bar.

Contoh:

```id="a1k3pz"
Apache
```

Apa yang terjadi setelah itu?

Shodan akan menampilkan:

- Daftar server Apache yang ditemukan di seluruh dunia
- Informasi IP address tiap server
- Port yang digunakan
- Lokasi geografis (negara/kota)
- Organisasi pemilik IP
- Bahkan visualisasi seperti **heatmap distribusi global**

Jadi dari satu keyword saja, kita sudah bisa mendapatkan gambaran besar tentang penyebaran suatu teknologi di internet.

#### 2. Filter Pencarian

Di sinilah kekuatan utama Shodan terasa. Kita bisa mempersempit hasil pencarian menggunakan **filter**.

Format dasarnya:

```id="m2k9sl"
<query> <filter>:<nilai>
```

Artinya:

- `query` → apa yang kita cari (misalnya Apache, nginx, SSH)
- `filter` → cara kita menyaring hasil
- `nilai` → nilai dari filter tersebut

Beberapa filter yang sering digunakan:

- `port:` → berdasarkan port
  Contoh:

  ```id="p9x1qw"
  Apache port:443
  ```

  Artinya: cari server Apache yang berjalan di port 443 (HTTPS)

- `country:` → berdasarkan negara (kode ISO)
  Contoh:

  ```id="z7lm2a"
  Apache country:IN
  ```

  Artinya: server Apache di India

- `org:` → berdasarkan organisasi
  Contoh:

  ```id="c8v3bn"
  SSH org:Microsoft
  ```

  Artinya: layanan SSH milik organisasi Microsoft

- `product:` → berdasarkan software/produk tertentu
  Contoh:

  ```id="x4rt8y"
  product:vsftpd
  ```

- `host:` → berdasarkan IP address
  Contoh:

  ```id="q1we5r"
  host:192.168.1.1
  ```

Contoh pencarian yang lebih kompleks:

```id="l0p9ok"
Apache port:443 country:IN
```

Penjelasan:

- `Apache` → layanan yang dicari
- `port:443` → hanya yang menggunakan HTTPS
- `country:IN` → hanya di India

Hasilnya: daftar server Apache HTTPS di India.

Contoh lain:

```id="t6yu8i"
nginx country:PK
```

→ Semua server Nginx di Pakistan

```id="g5h2jk"
port:3389 country:US
```

→ Semua layanan RDP di Amerika Serikat

Dengan kombinasi filter seperti ini, kita bisa melakukan pencarian yang sangat spesifik.

#### 3. Pencarian Berdasarkan IP Address

Kalau kita sudah punya target IP tertentu, kita bisa langsung melihat detailnya dengan filter `host:`.

Contoh:

```id="u7io3p"
host:8.8.8.8
```

Hasil yang ditampilkan biasanya meliputi:

- Daftar port yang terbuka
- Layanan yang berjalan
- Banner dari masing-masing layanan
- Informasi tambahan seperti SSL certificate

Dengan kata lain, kita bisa melihat “permukaan” dari sebuah server tanpa harus melakukan scanning sendiri.

Perlu diingat:
Fitur ini sangat powerful, tapi juga sensitif.

Mengakses informasi publik memang diperbolehkan, tetapi **menggunakan data tersebut untuk menyerang atau mengakses tanpa izin adalah ilegal**.

#### 4. Informasi Vulnerability (CVE)

Salah satu fitur menarik dari Shodan adalah kemampuannya menampilkan **potensi kerentanan (CVE)** berdasarkan layanan yang terdeteksi.

Misalnya:

- Jika Shodan menemukan server dengan versi software lama
- Maka Shodan bisa mengaitkannya dengan CVE yang relevan

Contoh kasus:

- Layanan RDP dengan versi lama bisa rentan terhadap exploit seperti **BlueKeep (CVE-2019-0708)**

Bagi seorang security analyst, ini sangat berguna untuk:

- Mengidentifikasi sistem yang berisiko
- Melakukan audit keamanan
- Menentukan prioritas patching

Namun perlu ditekankan:

- Informasi ini digunakan untuk **analisis dan pertahanan**, bukan eksploitasi ilegal
- Jangan mencoba login atau mengakses sistem yang ditemukan tanpa izin

#### Intinya

Dengan antarmuka web yang sederhana, Shodan memungkinkan kita:

- Melakukan pencarian global terhadap perangkat internet
- Memfilter hasil dengan sangat spesifik
- Melihat detail teknis dari layanan yang berjalan
- Mengidentifikasi potensi kerentanan

Semua itu bisa dilakukan langsung dari browser, tanpa perlu melakukan scanning sendiri.

## 3. Hubungan Antar Konsep

Semua konsep di atas membentuk alur pemahaman yang utuh sebagai berikut:

```
Shodan (mesin pencari khusus)
          |
          ▼
  Mengumpulkan data dengan cara:
  [Scanning → Deteksi Layanan → Indexing]
          |
          ▼
  Pengguna melakukan query (PASIF)
          |
  ┌───────┴──────────┐
  |                  |
Pencarian       Filter Canggih
 Dasar       (port, country, org,
              product, host)
  |                  |
  └───────┬──────────┘
          ▼
  Hasil: IP, port, banner, versi,
  organisasi, lokasi, potensi CVE
          |
          ▼
  Digunakan untuk: Passive Reconnaissance
  → Menilai attack surface target
  → Menemukan layanan yang terekspos
  → Mengidentifikasi perangkat rentan
```

- **Shodan sebagai database** memungkinkan _passive reconnaissance_ karena kita tidak perlu menyentuh target secara langsung.
- **Filter** adalah kekuatan utama Shodan — tanpa filter, hasil terlalu luas. Dengan filter, kita bisa menargetkan pencarian ke organisasi, negara, port, atau produk tertentu.
- **Informasi CVE** menghubungkan hasil pencarian Shodan dengan database kerentanan keamanan yang diketahui, menjadikan Shodan bukan sekadar alat inventarisasi, tapi juga alat **penilaian risiko keamanan**.

---

## 4. Kesimpulan

Shodan.IO adalah _search engine_ yang unik dan sangat bermanfaat bagi profesional keamanan siber karena:

1. **Mengindeks perangkat internet**, bukan konten web — membuatnya menjadi alat yang unik dibandingkan Google atau Bing.
2. **Cara kerja internal**: Shodan secara rutin melakukan scanning IPv4, merekam banner layanan, dan menyimpan data dalam database yang dapat dicari.
3. **Passive reconnaissance**: Penggunanya dapat mencari informasi tentang target tanpa mengirim satu paket pun ke target secara langsung, menjaga anonimitas.
4. **Filter yang powerful**: Kombinasi filter `port`, `country`, `org`, dan `product` memungkinkan pencarian yang sangat spesifik dan terarah.
5. **Visibility kerentanan**: Shodan dapat menampilkan informasi CVE yang terkait dengan layanan yang ditemukan, membantu identifikasi risiko keamanan.
6. **Tanggung jawab etis**: Meski powerful, Shodan harus digunakan secara bertanggung jawab dan etis. Mengakses atau mencoba login ke sistem yang ditemukan tanpa izin adalah tindakan ilegal.

---

> 💡 **Tips**: Untuk menggunakan fitur filter yang lebih lengkap di Shodan, diperlukan akun (gratis maupun berbayar). Beberapa informasi lebih detail hanya tersedia untuk pengguna premium/membership.
