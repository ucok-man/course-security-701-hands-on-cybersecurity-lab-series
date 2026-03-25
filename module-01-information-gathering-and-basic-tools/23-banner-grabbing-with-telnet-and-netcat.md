# 📝 Banner Grabbing dengan Telnet dan Netcat

## 1. Ringkasan Singkat

Video ini membahas teknik **banner grabbing** — salah satu metode reconnaissance (pengintaian) jaringan yang paling dasar namun sangat efektif. Dengan menggunakan dua tool sederhana yaitu **Telnet** dan **Netcat**, kita dapat terhubung ke layanan jaringan dan membaca respons awal (_banner_) yang dikirimkan oleh server. Informasi ini memungkinkan kita mengidentifikasi software, versi, serta potensi kelemahan pada sistem target.

---

## 2. Konsep Utama

### a. Banner Grabbing — Definisi

#### Apa itu Banner Grabbing?

Banner grabbing adalah teknik yang digunakan untuk “mengintip” informasi awal dari sebuah layanan jaringan (_network service_). Caranya sederhana: kita mencoba terhubung ke suatu service (misalnya web server, FTP, atau SSH), lalu membaca respons pertama yang dikirim oleh server tersebut.

Respons awal ini disebut **banner**.

Banner ibarat “salam pembuka” dari server. Saat kita datang (melakukan koneksi), server sering kali langsung memberikan informasi tentang dirinya tanpa diminta lebih lanjut.

#### Informasi Apa Saja yang Ada di Banner?

Banner biasanya mengandung beberapa informasi penting, seperti:

- **Protocol greetings** → pesan awal sesuai protokol (misalnya HTTP, FTP, SMTP)
- **Nama dan versi software server** → ini sangat penting untuk analisis keamanan
- **Pesan default layanan** → bisa berupa konfigurasi bawaan atau pesan custom

Informasi-informasi ini terlihat sederhana, tapi sangat berguna dalam tahap **reconnaissance** (pengumpulan informasi awal).

#### Kenapa Banner Grabbing Penting?

Dalam konteks keamanan atau pentesting, banner grabbing membantu kita:

- Mengidentifikasi **jenis server** yang digunakan (Apache, Nginx, dll)
- Mengetahui **versi software**
- Menentukan apakah server tersebut **berpotensi memiliki vulnerability**

Dengan kata lain, teknik ini membantu kita menentukan:

> “Target mana yang layak dianalisis lebih dalam?”

#### Contoh Banner dari Server Apache

Berikut contoh banner yang biasanya didapat dari web server berbasis Apache:

```http
HTTP/1.1 200 OK
Server: Apache/2.2.14 (Ubuntu)
X-Powered-By: PHP/5.3.2
```

Mari kita breakdown satu per satu:

- `HTTP/1.1 200 OK`
  Artinya request berhasil diproses oleh server

- `Server: Apache/2.2.14 (Ubuntu)`
  Menunjukkan bahwa server menggunakan Apache versi 2.2.14 di sistem Ubuntu

- `X-Powered-By: PHP/5.3.2`
  Memberi tahu bahwa aplikasi di server berjalan menggunakan PHP versi 5.3.2

#### Insight dari Contoh Tersebut

Dari banner sederhana ini saja, kita sudah mendapatkan banyak informasi:

- Server menggunakan **Apache 2.2.14** → versi lama (berpotensi punya celah keamanan)
- Menggunakan **PHP 5.3.2** → juga sudah outdated
- Berjalan di **Ubuntu**

Seorang pentester bisa langsung berpikir:

> “Apakah ada vulnerability yang diketahui di Apache 2.2.14 atau PHP 5.3.2?”

Di sinilah banner grabbing menjadi sangat powerful—karena hanya dari satu respons awal, kita sudah punya arah untuk eksplorasi lebih lanjut.

### b. Apa Itu "Banner"?

#### Definisi Banner

Dalam konteks jaringan, **banner** adalah teks atau metadata yang dikirim oleh sebuah layanan ketika kita pertama kali berhasil terhubung ke port tertentu.

Saat koneksi terjadi (biasanya melalui TCP), server sering kali langsung memberikan respons awal tanpa harus menunggu permintaan kompleks. Respons inilah yang disebut banner.

Banner bisa dianggap sebagai “identitas awal” dari sebuah service, karena sering berisi informasi tentang siapa dia dan bagaimana cara berkomunikasi dengannya.

#### Hubungan Banner dengan Port

Setiap layanan jaringan berjalan pada port tertentu, dan masing-masing port ini biasanya memiliki jenis banner yang berbeda sesuai dengan protokolnya.

Berikut beberapa contoh port yang umum:

- **Port 80 → HTTP (web server)**
  Banner biasanya berupa HTTP header

- **Port 21 → FTP**
  Banner biasanya berisi pesan sambutan FTP server

- **Port 22 → SSH**
  Banner berisi versi SSH server

- **Port 25 / 220 → SMTP (email server)**
  Banner biasanya berupa greeting dari mail server

Artinya, jenis banner yang kita terima sangat tergantung pada service apa yang berjalan di port tersebut.

#### Bagaimana Banner Dikirim?

Secara sederhana, alurnya seperti ini:

```
[Attacker/Tester]  ──TCP Connect──►  [Server:Port 80]
                   ◄──Banner (HTTP Header)──
```

Penjelasannya:

1. Client (attacker atau tester) melakukan koneksi ke suatu port di server
2. Server menerima koneksi tersebut
3. Server langsung mengirimkan banner sebagai respons awal
4. Client membaca banner tersebut untuk mendapatkan informasi

#### Kenapa Banner Penting untuk Dipahami?

Memahami banner sangat penting karena:

- Banner adalah **sumber informasi paling cepat** tentang suatu layanan
- Tidak perlu eksploitasi kompleks untuk mendapatkannya
- Bisa digunakan untuk:
  - Identifikasi service
  - Mengetahui versi software
  - Menentukan langkah reconnaissance berikutnya

Dengan kata lain, banner adalah langkah awal yang sangat sederhana, tapi memberikan insight yang cukup dalam untuk analisis lebih lanjut.

### c. Siapa yang Menggunakan Banner Grabbing?

#### Dua Perspektif Pengguna Banner Grabbing

Banner grabbing bukan hanya digunakan oleh attacker. Dalam praktik nyata, teknik ini digunakan oleh dua pihak utama dengan tujuan yang berbeda: **defender (blue team)** dan **attacker (red team)**.

Keduanya menggunakan teknik yang sama, tetapi dengan niat dan konteks yang berbeda.

#### 1. Defender (Blue Team)

Defender adalah pihak yang bertanggung jawab menjaga keamanan sistem. Mereka menggunakan banner grabbing sebagai alat bantu untuk memahami kondisi infrastruktur mereka sendiri.

Beberapa tujuan utama defender:

- **Inventarisasi layanan**
  Mengetahui service apa saja yang berjalan di server (HTTP, SSH, FTP, dll)

- **Identifikasi versi software**
  Misalnya menemukan bahwa server masih menggunakan versi lama dari Apache atau OpenSSH

- **Mendeteksi miskonfigurasi**
  Contohnya:
  - Server membocorkan terlalu banyak informasi di banner
  - Service yang tidak seharusnya terbuka ternyata bisa diakses publik

Dengan kata lain, defender menggunakan banner grabbing untuk:

> “Melihat sistem dari sudut pandang attacker sebelum attacker melakukannya.”

#### 2. Attacker (Red Team)

Attacker atau red team menggunakan banner grabbing sebagai langkah awal dalam proses reconnaissance.

Tujuan mereka biasanya:

- **Fingerprinting target**
  Mengidentifikasi teknologi yang digunakan oleh target (OS, web server, bahasa backend)

- **Mencari potensi celah keamanan**
  Misalnya:
  - Menemukan versi software yang sudah memiliki vulnerability publik
  - Menentukan attack vector yang paling efektif

Bagi attacker, banner grabbing membantu menjawab pertanyaan:

> “Target ini pakai apa, dan bagian mana yang paling lemah?”

#### Perbedaan Utama Keduanya

Walaupun tekniknya sama, perbedaannya terletak pada niat:

- Defender → untuk **melindungi dan memperbaiki sistem**
- Attacker → untuk **mencari dan mengeksploitasi kelemahan**

Inilah kenapa dalam dunia keamanan, kita sering belajar teknik “attacker” agar bisa menjadi defender yang lebih baik.

#### Etika dalam Penggunaan Banner Grabbing

Hal yang paling penting untuk diingat:

> ⚠️ Selalu lakukan banner grabbing hanya pada sistem yang kamu miliki atau sudah memberikan izin untuk diuji.

Melakukan scanning atau probing tanpa izin bisa dianggap sebagai pelanggaran hukum di banyak negara. Dalam konteks belajar atau praktik, pastikan kamu menggunakan:

- Lab pribadi (local VM, Docker, dll)
- Platform legal seperti CTF atau bug bounty (dengan scope jelas)

Dengan menjaga etika, kamu bisa tetap belajar teknik ini tanpa risiko hukum sekaligus membangun mindset profesional di bidang keamanan.

### d. Tool 1: Telnet

#### Apa itu Telnet?

Telnet adalah tool sederhana yang berfungsi sebagai **TCP client** untuk membuka koneksi langsung ke sebuah IP address dan port tertentu.

Berbeda dengan browser atau tool khusus lainnya, Telnet tidak “pintar” dalam memahami protokol seperti HTTP, FTP, atau SMTP. Ia hanya membuka koneksi mentah (_raw TCP connection_) tanpa interpretasi tambahan.

Karena sifatnya yang sederhana ini, Telnet sangat cocok digunakan untuk:

- Testing koneksi ke port tertentu
- Melihat respons awal dari server (banner grabbing)
- Mengirim request secara manual

#### Cara Kerja Telnet dalam Banner Grabbing

Saat kita menggunakan Telnet untuk terhubung ke sebuah port:

1. Telnet membuka koneksi TCP ke server
2. Server menerima koneksi
3. Kita bisa langsung berinteraksi dengan service tersebut secara manual
4. Jika server mengirim banner, kita bisa langsung melihatnya

Namun perlu diperhatikan, koneksi ini biasanya **tidak bertahan lama** (sekitar 5–10 detik), terutama pada protokol seperti HTTP yang mengharapkan respons cepat.

#### Sintaks Dasar Telnet

Berikut format dasar penggunaan Telnet:

```bash
telnet <IP_ADDRESS> <PORT>
```

Contohnya:

- `telnet 192.168.1.1 80` → konek ke web server
- `telnet 192.168.1.1 21` → konek ke FTP server

#### Contoh Banner Grabbing HTTP dengan Telnet

Mari kita lihat contoh penggunaan Telnet untuk melakukan banner grabbing pada web server.

Langkah 1: Hubungkan ke server di port 80 (HTTP)

```bash
telnet 172.17.199.151 80
```

Jika koneksi berhasil, layar akan tampak kosong (menunggu input), karena Telnet hanya membuka koneksi tanpa mengirim request otomatis.

Langkah 2: Kirim request HTTP secara manual

```bash
GET / HTTP/1.0
Host: 172.17.199.151

```

Perhatikan:

- Baris pertama adalah request untuk mengambil halaman root (`/`)
- Baris kedua adalah header `Host`
- **Enter dua kali** di akhir menandakan request selesai

#### Hasil yang Didapat

Jika server merespons, kita akan melihat output seperti ini:

```
HTTP/1.1 200 OK
Date: Mon, 01 Jan 2024 00:00:00 GMT
Server: Apache/2.2.14 (Ubuntu)
X-Powered-By: PHP/5.3.2
...
```

Mari kita pahami:

- `HTTP/1.1 200 OK` → request berhasil
- `Server: Apache/2.2.14 (Ubuntu)` → informasi server (ini inti banner grabbing)
- `X-Powered-By: PHP/5.3.2` → teknologi backend yang digunakan

Dari sini, kita sudah mendapatkan insight penting hanya dengan satu koneksi sederhana.

#### Hal Penting yang Perlu Diperhatikan

- Telnet tidak otomatis mengirim request → kita harus mengetik manual
- Koneksi bisa cepat terputus → jadi harus **cepat mengetik request**
- Tidak ada fitur parsing → semua output tampil apa adanya

Karena itu, Telnet sering digunakan untuk:

> belajar dasar protokol jaringan sekaligus memahami bagaimana komunikasi client–server sebenarnya terjadi di level rendah.

### e. Tool 2: Netcat (nc)

#### Apa itu Netcat?

Netcat (biasanya disingkat `nc`) adalah tool jaringan yang sangat fleksibel dan powerful. Karena kemampuannya yang serbaguna, Netcat sering dijuluki sebagai **“Swiss Army Knife for TCP/UDP”**.

Jika dibandingkan dengan Telnet, Netcat punya kemampuan yang jauh lebih luas. Ia tidak hanya bisa membuka koneksi, tetapi juga bisa digunakan untuk otomasi, scripting, bahkan debugging jaringan.

#### Kenapa Netcat Lebih Unggul dari Telnet?

Beberapa kelebihan utama Netcat:

- Bisa digunakan dalam dua mode:
  - **Interaktif** → seperti Telnet (manual input)
  - **Non-interaktif** → cocok untuk scripting dan otomasi

- Mendukung **timeout** → koneksi bisa otomatis ditutup setelah waktu tertentu
- Bisa mengirim data (_payload_) langsung saat koneksi dibuka
- Bisa membaca data sampai selesai (_end of file_)
- Mendukung **UDP**, tidak hanya TCP

Karena fleksibilitas ini, Netcat sering jadi tool favorit dalam banner grabbing maupun network testing secara umum.

#### Mode Interaktif Netcat

Mode ini mirip dengan Telnet, di mana kita membuka koneksi lalu berinteraksi secara manual.

Contoh:

```bash id="b0e2pu"
nc -v 172.17.199.151 22
```

Penjelasan:

- `-v` (verbose) → menampilkan detail koneksi
- `172.17.199.151 22` → konek ke port 22 (SSH)

Jika berhasil, biasanya server SSH langsung mengirim banner seperti:

```
SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

Artinya, kita langsung mendapatkan informasi versi SSH tanpa perlu mengetik apapun.

#### Mode Non-Interaktif Netcat (Otomasi)

Ini adalah mode yang paling powerful dan sering digunakan dalam praktik nyata.

Alih-alih mengetik manual, kita bisa langsung mengirim request saat koneksi dibuka.

Contoh menggunakan `printf`:

```bash id="jfdzqf"
printf "GET / HTTP/1.0\r\nHost: 172.17.199.151\r\n\r\n" | nc 172.17.199.151 80
```

Penjelasan:

- `printf` digunakan untuk membentuk request HTTP lengkap
- `\r\n` adalah format newline sesuai standar HTTP
- Output dari `printf` dikirim ke Netcat menggunakan pipe (`|`)
- Netcat langsung mengirim request ke server

Hasilnya:

- Server akan merespons dengan HTTP response + banner (header)
- Kita tidak perlu mengetik manual seperti di Telnet

Alternatif yang lebih sederhana:

```bash id="dc7y7m"
echo "" | nc -w 3 -n 172.17.199.151 80
```

Penjelasan:

- `echo ""` → mengirim input kosong (kadang cukup untuk memicu banner)
- `-w 3` → timeout 3 detik
- `-n` → tidak melakukan DNS lookup (langsung pakai IP)

#### Penjelasan Flag Penting Netcat

Berikut beberapa flag yang sering digunakan:

- `-v` → verbose, menampilkan detail koneksi
- `-w <detik>` → menentukan batas waktu koneksi sebelum ditutup
- `-n` → menggunakan IP langsung tanpa DNS resolution
- `-z` → mode scanning (tidak mengirim atau menerima data)
- `-u` → menggunakan UDP вместо TCP

#### Kapan Menggunakan Netcat?

Netcat sangat cocok digunakan ketika:

- Ingin melakukan **banner grabbing secara cepat**
- Membutuhkan **otomasi** (misalnya dalam script)
- Ingin testing service tanpa tool berat
- Melakukan scanning sederhana atau debugging jaringan

Singkatnya:

> Jika Telnet adalah alat dasar untuk belajar, maka Netcat adalah alat profesional yang digunakan dalam praktik nyata.

### f. Penggunaan Flag `-z` untuk Port Scanning

#### Apa Fungsi Flag `-z`?

Dalam Netcat, flag `-z` digunakan untuk melakukan **port scanning sederhana** dengan cara yang sangat ringan.

Berbeda dengan penggunaan Netcat biasa yang mengirim atau menerima data, mode `-z` bekerja dengan prinsip:

> “Coba konek ke port → jika berhasil, berarti port terbuka.”

Artinya, Netcat hanya mengecek status port tanpa melakukan interaksi lebih lanjut dengan service yang berjalan di dalamnya.

#### Cara Menggunakan `-z`

Contoh penggunaan:

```bash
nc -zv 172.17.199.151 80
```

Penjelasan:

- `-z` → zero I/O (tidak kirim/terima data)
- `-v` → verbose (menampilkan hasil koneksi)
- `172.17.199.151 80` → target IP dan port

#### Hasil yang Didapat

Jika port terbuka, outputnya akan seperti ini:

```id="a2x9hf"
Connection to 172.17.199.151 80 port [tcp/http] succeeded!
```

Artinya:

- Netcat berhasil membuat koneksi ke port 80
- Port tersebut dalam keadaan **open** (aktif dan menerima koneksi)

#### Bagaimana Jika Port Tidak Terbuka?

Biasanya akan muncul pesan seperti:

- `Connection refused` → port tertutup (closed)
- Tidak ada respons → kemungkinan **filtered** (diblok oleh firewall)

Ini memberikan gambaran awal tentang kondisi jaringan tanpa perlu scanning kompleks.

#### Kapan `-z` Digunakan?

Flag ini sangat berguna untuk:

- **Verifikasi cepat port tertentu**
  Misalnya memastikan apakah web server (port 80/443) aktif

- **Scripting sederhana**
  Digabung dengan loop untuk scanning banyak port

- **Debugging jaringan**
  Mengecek apakah service sudah berjalan atau belum

#### Insight Penting

Walaupun sederhana, mode `-z` sangat efisien karena:

- Tidak membebani server (tidak ada payload dikirim)
- Cepat dijalankan
- Cocok untuk pengecekan awal sebelum masuk ke tahap lebih dalam seperti banner grabbing

Dengan kata lain:

> Gunakan `-z` untuk “cek pintu terbuka atau tidak”, sebelum mencoba “melihat isi di dalamnya”.

### g. Banner Grabbing via UDP

#### Apa Bedanya dengan TCP?

Sebelumnya kita melihat banner grabbing menggunakan TCP, di mana koneksi bersifat **connection-oriented** (ada proses connect, lalu komunikasi).

Berbeda dengan itu, UDP adalah protokol **connectionless**:

- Tidak ada proses “handshake”
- Tidak ada jaminan respons dari server
- Lebih ringan, tapi juga lebih “silent”

Akibatnya, banner grabbing pada UDP sedikit lebih tricky dibanding TCP.

#### Menggunakan Netcat untuk UDP

Netcat tetap bisa digunakan untuk mengirim request ke layanan berbasis UDP, misalnya NTP (Network Time Protocol) di port 123.

Contoh:

```bash id="8u3k2d"
printf "" | nc -u -w 3 172.17.199.151 123
```

Penjelasan:

- `printf ""` → mengirim payload kosong (atau bisa juga payload tertentu)
- `-u` → mengaktifkan mode UDP
- `-w 3` → timeout 3 detik (penting karena UDP tidak menjamin respons)
- `172.17.199.151 123` → target IP dan port UDP

#### Apa yang Terjadi Saat Dijalankan?

Berbeda dengan TCP, hasil dari UDP tidak selalu jelas.

Kemungkinan yang terjadi:

1. **Ada respons dari server**
   → berarti layanan UDP aktif dan merespons request

2. **Tidak ada output sama sekali (silent)**
   → bisa berarti:
   - Service tidak aktif
   - Service aktif tapi tidak merespons payload tersebut
   - Paket diblok oleh firewall

#### Kenapa UDP Sering Tidak Memberikan Banner?

Sebagian besar layanan UDP:

- Tidak mengirim “banner” secara otomatis
- Hanya merespons jika menerima request yang valid dan sesuai format

Contohnya:

- NTP hanya merespons request dengan format tertentu
- DNS hanya merespons query yang valid

Jadi, jika kita hanya mengirim payload kosong:

> kemungkinan besar tidak akan ada respons

#### Insight Penting

Banner grabbing via UDP membutuhkan pendekatan yang berbeda:

- Tidak cukup hanya “connect” seperti TCP
- Harus tahu **format request yang benar** untuk memancing respons
- Perlu memahami protokol target (DNS, NTP, dll)

Karena itu, UDP sering digunakan untuk:

- Advanced reconnaissance
- Service enumeration yang lebih dalam

Kesimpulannya:

> Jika TCP seperti mengetuk pintu dan langsung disapa, maka UDP lebih seperti mengirim pesan—dan belum tentu dibalas.

### h. Flag `-n` (No DNS Resolution) — Penting untuk Otomasi

#### Apa Fungsi Flag `-n`?

Flag `-n` pada Netcat digunakan untuk **menonaktifkan DNS resolution**, yaitu proses menerjemahkan nama domain (seperti `example.com`) menjadi alamat IP.

Dengan kata lain, saat menggunakan `-n`, Netcat akan:

- Langsung menggunakan IP yang diberikan
- Tidak mencoba melakukan lookup ke DNS server

#### Kenapa DNS Resolution Bisa Jadi Masalah?

Secara default, ketika kita menggunakan domain, sistem akan melakukan DNS lookup terlebih dahulu. Proses ini bisa menimbulkan beberapa masalah:

- **Lebih lambat** → karena harus menunggu respons dari DNS server
- **Bisa menyebabkan delay atau hang** → jika DNS lambat atau tidak respons
- **Menambah kompleksitas saat scanning dalam jumlah besar**

Masalah ini akan terasa terutama saat:

- Melakukan scanning ke banyak target
- Mengolah ratusan atau ribuan subdomain

#### Keuntungan Menggunakan `-n`

Dengan menambahkan flag `-n`, kita mendapatkan beberapa keuntungan:

- **Lebih cepat** → langsung ke IP tanpa proses tambahan
- **Lebih stabil** → menghindari delay dari DNS
- **Lebih cocok untuk otomasi** → terutama dalam script atau pipeline

Contoh penggunaan:

```bash id="k3v9x1"
nc -zvn 172.17.199.151 80
```

Penjelasan:

- `-z` → scanning tanpa kirim data
- `-v` → verbose output
- `-n` → skip DNS resolution

#### Kapan Sebaiknya Menggunakan `-n`?

Gunakan flag ini ketika:

- Target sudah berupa **IP address**
- Melakukan **port scanning dalam jumlah besar**
- Membuat **script otomasi** (misalnya looping banyak host)
- Ingin hasil lebih cepat dan minim gangguan

#### Insight Penting

Dalam praktik nyata, flag `-n` sering dianggap “kecil tapi penting”. Banyak pemula mengabaikannya, padahal dampaknya cukup besar dalam performa.

Kesimpulannya:

> Jika kamu tidak butuh DNS, selalu gunakan `-n` untuk membuat proses scanning lebih cepat, ringan, dan stabil.

## 3. Hubungan Antar Konsep

Alur pemahaman dalam video ini membentuk sebuah rantai logis:

```
Banner Grabbing (Teknik)
        │
        ├── Tujuan: Identifikasi software & versi dari respons awal server
        │
        ├── Tool 1: Telnet
        │       └── Sederhana, interaktif, cocok untuk eksplorasi manual
        │
        └── Tool 2: Netcat
                ├── Interaktif (seperti Telnet)
                ├── Non-interaktif / scriptable (lebih powerful)
                ├── Mendukung timeout & flag no-DNS (-w, -n)
                ├── Scan port saja (-z)
                └── Mendukung UDP (-u)
```

Telnet dan Netcat pada dasarnya melakukan hal yang sama: **membuka koneksi TCP mentah ke port tertentu**. Perbedaannya terletak pada **fleksibilitas** — Netcat jauh lebih powerful karena bisa digunakan dalam script otomasi, mendukung UDP, dan memiliki lebih banyak flag kontrol.

Informasi banner yang diperoleh dari kedua tool ini (seperti `Server: Apache/2.2.14`) kemudian dapat **diverifikasi silang** dengan tool lain seperti Nmap untuk memastikan keakuratannya.

---

## 4. Kesimpulan

Banner grabbing adalah teknik reconnaissance cepat dan berbiaya rendah yang memanfaatkan respons awal (_banner_) dari layanan jaringan untuk mengidentifikasi software, versi, dan potensi kelemahan. Dua tool utama yang digunakan adalah:

- **Telnet** — sederhana, cocok untuk pengujian manual dan interaktif.
- **Netcat** — lebih fleksibel, mendukung mode non-interaktif/scripting, timeout, flag no-DNS, port scanning ringan (`-z`), dan UDP.

Teknik ini digunakan baik oleh defender maupun attacker, sehingga **izin eksplisit** sebelum melakukan pengujian pada sistem apapun adalah hal yang wajib dan tidak bisa dikompromikan.

---

> 🔐 **Pengingat Etika:** Jalankan banner grabbing **hanya** pada sistem yang kamu miliki atau yang telah memberikan izin pengujian secara tertulis.
