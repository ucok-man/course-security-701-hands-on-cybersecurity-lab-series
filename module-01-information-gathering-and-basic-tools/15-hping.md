# 📝 Catatan Materi: HPING3 — Packet Crafting & Network Analysis Tool

---

## 1. Ringkasan Singkat

Video ini memperkenalkan **HPING3**, sebuah tool command-line yang digunakan untuk membuat (_craft_) dan menganalisis paket jaringan secara manual. Berbeda dari perintah `ping` biasa, HPING3 memberikan kontrol penuh atas berbagai protokol jaringan seperti ICMP, TCP, UDP, dan raw IP. Materi mencakup instalasi, sintaks dasar, serta beberapa skenario pengujian praktis seperti ICMP ping, TCP SYN scan, ACK probe untuk menguji firewall, dan UDP probe. Video juga menyertakan penjelasan konsep dasar **port** dan **port scanning**.

> ⚠️ **Peringatan Etis:** HPING3 adalah tool yang sangat powerful dan dapat mengganggu jaringan jika disalahgunakan. Gunakan **hanya** di lab terisolasi atau dengan izin tertulis dari pemilik sistem.

---

## 2. Konsep Utama

### a. Apa Itu HPING3?

#### Pengertian Dasar HPING3

**HPING3** adalah sebuah tool jaringan berbasis **command-line** yang digunakan untuk membuat, mengirim, dan menganalisis paket jaringan secara manual.

Tool ini sering dipakai oleh **network engineer**, **system administrator**, maupun **security researcher** untuk melakukan berbagai aktivitas seperti:

- Network testing
- Firewall testing
- Network troubleshooting
- Security auditing
- Packet crafting

Secara sederhana, HPING3 memungkinkan kita **mengontrol paket jaringan secara langsung**, bahkan sampai ke detail kecil seperti header paket.

HPING3 merupakan **versi lanjutan dari tool lama bernama HPING**. Versi ketiga ini menambahkan banyak fitur baru dan fleksibilitas yang lebih tinggi dibandingkan versi sebelumnya.

Karena fleksibilitasnya, HPING3 sering dianggap sebagai **“ping yang jauh lebih powerful.”**

#### Perbedaan HPING3 dengan Ping Biasa

Command `ping` yang biasa kita gunakan sebenarnya sangat terbatas.

Sebagai contoh:

```bash
ping google.com
```

Perintah tersebut hanya:

- Mengirim **ICMP Echo Request**
- Menunggu **ICMP Echo Reply**
- Mengukur **latency**

Artinya kita **tidak bisa mengubah jenis paket**, **tidak bisa mengatur flags**, dan **tidak bisa memodifikasi header jaringan**.

Sebaliknya, **HPING3 memberikan kontrol penuh terhadap paket yang dikirim**.

Dengan HPING3 kita bisa:

- Membuat paket **ICMP**
- Membuat paket **TCP**
- Membuat paket **UDP**
- Membuat **raw IP packet**

Tidak hanya itu, kita juga bisa memodifikasi berbagai bagian dari paket tersebut.

#### Kemampuan Utama HPING3

Berikut beberapa kemampuan penting yang dimiliki HPING3.

##### 1. Membuat Berbagai Jenis Paket Jaringan

HPING3 memungkinkan kita membuat berbagai jenis paket seperti:

- **ICMP**
- **TCP**
- **UDP**
- **Raw IP**

Hal ini sangat berguna ketika ingin menguji bagaimana suatu server atau firewall merespons berbagai jenis traffic.

Contohnya:

- Menguji apakah port tertentu terbuka
- Menguji firewall rules
- Menguji filtering jaringan

##### 2. Mengatur TCP Flags

Dalam protokol TCP terdapat beberapa **flag penting**, seperti:

- **SYN**
- **ACK**
- **RST**
- **FIN**
- **PSH**
- **URG**

Dengan HPING3 kita bisa mengatur flag tersebut secara manual.

Contoh penggunaan:

- **SYN scan**
- **ACK scan**
- **Firewall testing**
- **Connection behavior testing**

Misalnya kita hanya ingin mengirim paket **SYN** saja untuk melihat apakah suatu port terbuka.

##### 3. IP Spoofing

HPING3 juga mendukung **IP spoofing**, yaitu kemampuan untuk **memalsukan alamat IP sumber**.

Artinya paket bisa dikirim seolah-olah berasal dari alamat IP lain.

Contohnya:

```
Source IP palsu → Server Target
```

Fitur ini sering digunakan untuk:

- Testing keamanan jaringan
- Simulasi traffic
- Analisis firewall

Namun perlu diingat bahwa penggunaan IP spoofing harus dilakukan **secara legal dan etis**, misalnya di lab testing atau environment yang diizinkan.

##### 4. Mengontrol Timing Paket

HPING3 memungkinkan kita mengatur **interval pengiriman paket**.

Artinya kita bisa mengontrol:

- Seberapa cepat paket dikirim
- Berapa banyak paket yang dikirim
- Jeda antar paket

Fitur ini sering digunakan untuk:

- **Load testing**
- **Network stress testing**
- **Packet timing analysis**

Sebagai contoh, kita bisa mengirim paket setiap **10 milidetik**, atau bahkan **ribuan paket per detik** jika diperlukan.

#### Sintaks Dasar HPING3

Untuk menggunakan HPING3, sintaks dasar yang digunakan adalah sebagai berikut:

```bash
sudo hping3 [options] <target>
```

Penjelasan komponennya:

- **sudo**
  Digunakan karena HPING3 bekerja dengan **raw packet**. Sistem operasi biasanya hanya mengizinkan pengguna dengan hak **root/administrator** untuk membuat dan mengirim raw packet.

- **hping3**
  Program utama yang digunakan untuk membuat dan mengirim paket.

- **[options]**
  Berisi berbagai parameter untuk mengatur perilaku paket, seperti:
  - jenis protokol
  - port tujuan
  - TCP flags
  - interval paket
  - spoofing IP

- **<target>**
  Alamat tujuan paket yang dikirim, bisa berupa:
  - IP address
  - hostname
  - domain

Contoh sederhana:

```bash
sudo hping3 google.com
```

Perintah ini akan mulai mengirim paket ke **google.com** menggunakan konfigurasi default HPING3.

#### Mengapa HPING3 Membutuhkan Hak Akses Root

HPING3 bekerja dengan **raw sockets**.

Raw socket memungkinkan program untuk:

- Membuat **packet header sendiri**
- Mengontrol **layer jaringan secara langsung**
- Mengirim paket tanpa melalui mekanisme standar sistem operasi

Karena kemampuan ini sangat sensitif dan berpotensi disalahgunakan, sistem operasi Linux membatasi aksesnya hanya untuk **root atau administrator**.

Itulah alasan mengapa hampir semua penggunaan HPING3 selalu diawali dengan:

```bash
sudo
```

Tanpa hak akses tersebut, sistem biasanya akan menolak pembuatan raw packet dan HPING3 tidak dapat berjalan dengan benar.

### b. Use Cases (Kasus Penggunaan) HPING3

#### Gambaran Umum Penggunaan HPING3

HPING3 merupakan tool yang sangat fleksibel karena memungkinkan kita **membuat dan mengirim paket jaringan secara manual**. Kemampuan ini membuat HPING3 sering digunakan dalam berbagai skenario yang berkaitan dengan **pengujian jaringan, troubleshooting, dan keamanan sistem**.

Berbeda dengan tool sederhana seperti `ping`, HPING3 memberikan kontrol penuh terhadap **jenis paket, flags, timing, serta struktur header paket**. Karena itu, banyak administrator jaringan dan security engineer memanfaatkannya untuk melakukan berbagai eksperimen dan pengujian terhadap infrastruktur jaringan.

Berikut beberapa kasus penggunaan HPING3 yang paling umum.

#### Firewall dan IDS Testing

Salah satu penggunaan HPING3 yang paling sering adalah untuk **menguji firewall dan sistem deteksi intrusi (IDS – Intrusion Detection System)**.

Firewall biasanya memiliki aturan tertentu untuk memfilter traffic jaringan, misalnya:

- Mengizinkan HTTP (port 80)
- Mengizinkan HTTPS (port 443)
- Memblokir port lain
- Memblokir paket dengan karakteristik tertentu

Dengan HPING3, kita bisa membuat paket yang **menyerupai berbagai jenis traffic** untuk melihat bagaimana firewall meresponsnya.

Contoh sederhana:

```bash
sudo hping3 -S -p 80 target.com
```

Penjelasan:

- `-S` → Mengirim paket TCP dengan **flag SYN**
- `-p 80` → Target port **80**
- `target.com` → Server tujuan

Paket SYN biasanya digunakan pada **awal koneksi TCP**. Dengan mengirim paket seperti ini, kita bisa melihat apakah firewall:

- Mengizinkan koneksi
- Menolak koneksi
- Mengabaikan paket

Hasil respons dari server akan membantu administrator memahami **bagaimana aturan firewall bekerja dalam praktiknya**.

Selain itu, HPING3 juga sering digunakan untuk melihat apakah **IDS mampu mendeteksi pola traffic tertentu**.

#### TTL dan Routing Experiments

HPING3 juga sering digunakan untuk melakukan eksperimen terkait **routing jaringan** dan **TTL (Time-To-Live)**.

TTL adalah nilai pada header paket IP yang menentukan **berapa banyak hop (lompatan router)** yang boleh dilewati oleh sebuah paket sebelum paket tersebut dibuang.

Setiap kali paket melewati router, nilai TTL akan berkurang **satu angka**. Jika TTL mencapai **0**, router akan membuang paket tersebut.

Dengan HPING3, kita bisa mengatur TTL secara manual untuk melihat **bagaimana paket melewati jaringan**.

Contoh:

```bash
sudo hping3 -t 5 target.com
```

Penjelasan:

- `-t 5` → Mengatur nilai **TTL = 5**

Artinya paket hanya boleh melewati **maksimal 5 router** sebelum dibuang.

Teknik ini sering digunakan oleh administrator jaringan untuk:

- Memahami jalur routing
- Menganalisis topologi jaringan
- Menguji bagaimana router menangani paket dengan TTL tertentu

Konsep ini mirip dengan cara kerja tool **`traceroute`**, tetapi HPING3 memberikan fleksibilitas yang lebih tinggi dalam mengatur jenis paket.

#### TCP Handshake Simulation

Dalam protokol TCP, sebelum dua perangkat bisa bertukar data, mereka harus melalui proses yang disebut **TCP Three-Way Handshake**.

Proses ini terdiri dari tiga langkah:

1. Client mengirim **SYN**
2. Server membalas **SYN-ACK**
3. Client mengirim **ACK**

Dengan HPING3, kita bisa mensimulasikan sebagian atau seluruh proses ini secara manual.

Contoh mengirim paket SYN:

```bash
sudo hping3 -S -p 443 target.com
```

Penjelasan:

- `-S` → Mengaktifkan **SYN flag**
- `-p 443` → Mengirim ke **port HTTPS**

Jika server merespons dengan **SYN-ACK**, biasanya itu berarti:

- Port terbuka
- Server siap menerima koneksi

Jika server merespons dengan **RST**, kemungkinan besar port tersebut **tertutup**.

Teknik ini sangat berguna untuk:

- Memahami perilaku koneksi TCP
- Menganalisis server response
- Menguji konfigurasi jaringan

#### SYN Scan (Port Scanning)

HPING3 juga bisa digunakan untuk melakukan **port scanning**, khususnya dengan teknik **SYN scan**.

SYN scan adalah teknik scanning yang hanya mengirim **paket SYN** tanpa menyelesaikan proses handshake TCP.

Contoh:

```bash
sudo hping3 -S -p 22 target.com
```

Perintah ini akan mengirim paket SYN ke **port 22 (SSH)**.

Respons server biasanya akan memberikan informasi seperti:

- **SYN-ACK** → Port kemungkinan **terbuka**
- **RST** → Port **tertutup**
- **Tidak ada respons** → Bisa jadi **difilter oleh firewall**

Teknik ini sering digunakan dalam **pengujian keamanan jaringan** untuk mengetahui port mana saja yang terbuka.

Namun penting untuk diingat bahwa **port scanning terhadap sistem yang tidak memiliki izin dapat dianggap ilegal**. Oleh karena itu teknik ini sebaiknya hanya dilakukan pada:

- Lab testing
- Sistem milik sendiri
- Sistem yang memberikan izin eksplisit

#### Response Time Measurement

HPING3 juga dapat digunakan untuk mengukur **waktu respons jaringan**, yang sering disebut **RTT (Round Trip Time)**.

RTT adalah waktu yang dibutuhkan paket untuk:

1. Dikirim dari client
2. Sampai ke server
3. Kembali lagi ke client

Contoh penggunaan:

```bash
sudo hping3 -S target.com
```

HPING3 akan mengirim paket secara berulang dan menampilkan waktu respons dari setiap paket.

Contoh output sederhana:

```
len=46 ip=192.168.1.1 ttl=54 DF id=0 sport=80 flags=SA seq=0 win=64240 rtt=28.5 ms
```

Penjelasan beberapa bagian penting:

- **ttl=54** → TTL dari paket balasan
- **flags=SA** → SYN-ACK
- **rtt=28.5 ms** → waktu round trip

Informasi ini membantu administrator untuk:

- Mengukur latency jaringan
- Mendeteksi jaringan lambat
- Menganalisis kualitas koneksi

#### Malformed Traffic Simulation

HPING3 juga memungkinkan kita membuat **paket yang tidak normal atau tidak standar**, yang sering disebut **malformed packets**.

Paket seperti ini bisa digunakan untuk menguji bagaimana sistem keamanan menangani traffic yang aneh atau tidak valid.

Contoh skenario pengujian:

- Paket dengan kombinasi flag yang tidak biasa
- Paket dengan ukuran aneh
- Paket dengan header yang dimodifikasi

Tujuannya bukan untuk menyerang sistem, tetapi untuk melakukan **defensive testing**, yaitu memastikan bahwa:

- Firewall tetap stabil
- Server tidak crash
- IDS mampu mendeteksi traffic abnormal

Dalam dunia keamanan jaringan, pengujian seperti ini penting untuk memastikan sistem tetap **tahan terhadap berbagai jenis traffic yang tidak biasa**.

Karena itulah HPING3 menjadi salah satu tool yang sangat populer dalam bidang **network security testing dan penetration testing**.

### c. Konsep Port dalam Jaringan

#### Pengertian Dasar Port

Sebelum memahami konsep **port scanning**, penting untuk terlebih dahulu memahami apa itu **port dalam jaringan**.

Dalam konteks networking, **port** adalah sebuah **virtual endpoint** yang digunakan untuk komunikasi antara perangkat dalam jaringan. Port berfungsi sebagai titik masuk dan keluar bagi data yang dikirim melalui jaringan.

Setiap port diidentifikasi menggunakan **angka unik** yang berada dalam rentang:

```
0 – 65535
```

Port bekerja bersama dengan **IP address** untuk memastikan data yang dikirim melalui jaringan dapat sampai ke **aplikasi atau layanan yang tepat** di dalam sebuah mesin.

Tanpa adanya port, sebuah komputer tidak akan bisa menentukan **aplikasi mana yang harus menerima data yang masuk**.

#### Hubungan antara IP Address dan Port

Dalam komunikasi jaringan, **IP address** dan **port** memiliki peran yang berbeda tetapi saling melengkapi.

- **IP Address** digunakan untuk mengidentifikasi **perangkat atau mesin** di dalam jaringan.
- **Port** digunakan untuk mengidentifikasi **layanan atau aplikasi** yang berjalan di dalam mesin tersebut.

Sebagai contoh, sebuah server dapat menjalankan beberapa layanan sekaligus, seperti:

- Web server
- SSH server
- FTP server
- Database server

Semua layanan tersebut berada di **mesin yang sama** dan memiliki **IP address yang sama**, tetapi masing-masing menggunakan **port yang berbeda** agar sistem dapat membedakan traffic yang masuk.

Contohnya:

```
192.168.1.10:80
```

Penjelasan:

- **192.168.1.10** → IP address server
- **80** → port yang digunakan oleh layanan web (HTTP)

Jika seseorang membuka website dari server tersebut, permintaan akan diarahkan ke **port 80**, tempat web server sedang berjalan.

#### Analogi Sederhana Port

Agar lebih mudah dipahami, kita bisa menggunakan analogi sebuah **gedung perkantoran**.

Bayangkan sebuah gedung besar:

- **Alamat gedung** → adalah **IP Address**
- **Nomor ruangan di dalam gedung** → adalah **port**

Misalnya:

- Ruang **22**
- Ruang **80**
- Ruang **443**

Setiap ruangan memiliki fungsi yang berbeda. Begitu juga dengan port.

Ketika seseorang mengirim paket jaringan ke sebuah server, paket tersebut tidak hanya dikirim ke **alamat gedung**, tetapi juga ke **ruangan tertentu**.

Contohnya:

```
IP Address : 203.0.113.10
Port       : 443
```

Artinya data dikirim ke:

- Gedung dengan alamat **203.0.113.10**
- Ruangan **443**

Di ruangan tersebut biasanya terdapat **aplikasi tertentu yang sedang “mendengarkan” (listening)** untuk menerima koneksi.

#### Layanan dan Port Standar

Dalam praktiknya, banyak layanan jaringan menggunakan **nomor port standar** yang sudah dikenal secara luas. Hal ini memudahkan perangkat dan aplikasi untuk mengetahui **port mana yang harus digunakan untuk layanan tertentu**.

Beberapa contoh layanan dan port standarnya adalah sebagai berikut.

**SSH — Port 22**

SSH digunakan untuk **remote login ke server secara aman**.

Contoh koneksi:

```bash
ssh user@server-ip
```

Secara default, SSH akan mencoba terhubung ke **port 22**.

**FTP — Port 21**

FTP digunakan untuk **transfer file melalui jaringan**.

Server FTP biasanya mendengarkan koneksi pada **port 21**.

**Telnet — Port 23**

Telnet digunakan untuk **remote access ke sistem**, tetapi saat ini sudah jarang digunakan karena **tidak terenkripsi**.

Sebagai penggantinya, biasanya digunakan **SSH**.

**HTTP — Port 80**

HTTP digunakan untuk **komunikasi web biasa (tanpa enkripsi)**.

Ketika seseorang membuka website seperti:

```
http://example.com
```

Browser secara otomatis akan menghubungi **port 80** pada server.

**HTTPS — Port 443**

HTTPS adalah versi **HTTP yang menggunakan enkripsi SSL/TLS**.

Ketika kita membuka website seperti:

```
https://example.com
```

Browser akan terhubung ke **port 443**.

**DNS — Port 53**

DNS digunakan untuk **menerjemahkan domain name menjadi IP address**.

Misalnya:

```
google.com → 142.250.x.x
```

Proses ini biasanya menggunakan **port 53**.

**SFTP — Port 22**

SFTP (Secure File Transfer Protocol) sebenarnya berjalan **di atas SSH**. Karena itu, SFTP juga menggunakan **port 22**.

#### Well-Known Ports

Dalam dunia networking, terdapat kategori port yang disebut **well-known ports**.

Well-known ports adalah port dengan nomor:

```
0 – 1023
```

Port-port ini telah ditetapkan secara standar untuk layanan tertentu oleh organisasi **IANA (Internet Assigned Numbers Authority)**.

Contoh well-known ports:

- 21 → FTP
- 22 → SSH
- 23 → Telnet
- 53 → DNS
- 80 → HTTP
- 443 → HTTPS

Karena sudah menjadi standar yang dikenal luas, banyak aplikasi dan sistem jaringan **secara otomatis menggunakan port tersebut secara default**.

#### Mengapa Memahami Port Itu Penting

Memahami konsep port sangat penting dalam berbagai bidang seperti:

- **Administrasi jaringan**
- **Troubleshooting koneksi**
- **Konfigurasi firewall**
- **Keamanan jaringan**
- **Port scanning**

Sebagai contoh, ketika sebuah layanan tidak dapat diakses, salah satu penyebabnya bisa saja:

- Port yang digunakan **tertutup**
- Port **diblokir oleh firewall**
- Layanan tidak berjalan pada port tersebut

Dengan memahami bagaimana port bekerja, kita dapat lebih mudah melakukan **diagnosis dan analisis masalah jaringan**.

### d. Port Scanning

#### Pengertian Port Scanning

**Port scanning** adalah proses mengirimkan berbagai permintaan jaringan (network requests) ke sejumlah **port pada sebuah host atau server** dengan tujuan untuk mengetahui **status dari port tersebut**.

Status port yang biasanya ingin diketahui antara lain:

- **Open (terbuka)** → port menerima koneksi dan ada layanan yang berjalan
- **Closed (tertutup)** → port tidak digunakan oleh layanan apa pun
- **Filtered (difilter)** → port kemungkinan diblokir oleh firewall atau sistem keamanan

Dengan melakukan port scanning, kita dapat memahami **layanan apa saja yang berjalan pada suatu sistem**, serta bagaimana sistem tersebut merespons koneksi yang masuk.

Dalam praktiknya, teknik ini sangat umum digunakan dalam **administrasi jaringan**, **troubleshooting**, maupun **keamanan sistem**.

#### Mengapa Port Scanning Dilakukan

Setiap server biasanya menjalankan beberapa layanan jaringan, misalnya:

- Web server
- SSH server
- Database
- File transfer server

Masing-masing layanan tersebut biasanya berjalan pada **port tertentu**.

Dengan melakukan port scanning, kita bisa mengetahui:

- Port mana saja yang **terbuka**
- Layanan apa yang **kemungkinan berjalan**
- Apakah ada port yang **tidak seharusnya terbuka**

Informasi ini sangat penting untuk memahami **permukaan serangan (attack surface)** dari sebuah sistem.

Semakin banyak port yang terbuka, biasanya semakin banyak juga layanan yang berjalan, dan itu berarti **potensi risiko keamanan juga meningkat**.

#### Cara Kerja Port Scanning

Secara sederhana, port scanning bekerja dengan cara:

1. Mengirim paket jaringan ke sebuah port
2. Menunggu respons dari server
3. Menganalisis respons tersebut

Dari respons yang diterima, kita bisa menentukan status port tersebut.

Sebagai contoh, ketika sebuah scanner mengirim paket **TCP SYN** ke suatu port, kemungkinan responsnya bisa seperti berikut:

**Jika port terbuka**

Server biasanya akan membalas dengan paket:

```id="5qokap"
SYN-ACK
```

Ini menunjukkan bahwa server siap untuk melanjutkan koneksi TCP.

**Jika port tertutup**

Server biasanya akan membalas dengan:

```id="m0zn6p"
RST
```

Artinya tidak ada layanan yang berjalan pada port tersebut.

**Jika port difilter**

Dalam beberapa kasus, server atau firewall **tidak memberikan respons sama sekali**, atau paket diblokir di tengah jaringan.

Ini biasanya berarti port tersebut **dilindungi oleh firewall atau sistem filtering**.

#### Contoh Konsep Port Scanning

Bayangkan kita ingin mengetahui apakah sebuah server memiliki layanan **SSH** yang aktif.

SSH biasanya berjalan pada **port 22**.

Sebuah tool scanning akan mencoba mengirim paket ke port tersebut.

Contoh menggunakan HPING3:

```bash id="in49yd"
sudo hping3 -S -p 22 target.com
```

Penjelasan:

- `-S` → Mengirim paket TCP dengan **flag SYN**
- `-p 22` → Mengirim ke **port 22**
- `target.com` → alamat server target

Jika server merespons dengan **SYN-ACK**, maka kemungkinan besar:

- Port **22 terbuka**
- Layanan **SSH sedang berjalan**

Jika server membalas dengan **RST**, maka port tersebut **tertutup**.

Jika tidak ada respons, kemungkinan:

- Port **diblokir firewall**
- Server **mengabaikan paket tersebut**

Dengan melakukan scanning pada banyak port sekaligus, kita bisa membuat **peta layanan yang berjalan pada server tersebut**.

#### Penggunaan Port Scanning oleh Administrator Jaringan

Port scanning tidak selalu berkaitan dengan aktivitas menyerang sistem. Justru dalam banyak kasus, teknik ini digunakan oleh **administrator jaringan** untuk menjaga keamanan sistem mereka sendiri.

Beberapa tujuan umum penggunaan port scanning oleh administrator antara lain:

- Memastikan hanya port yang diperlukan saja yang terbuka
- Mengidentifikasi layanan yang tidak sengaja terbuka
- Menguji konfigurasi firewall
- Memverifikasi kebijakan keamanan jaringan

Sebagai contoh, sebuah server produksi biasanya hanya membutuhkan beberapa port saja, seperti:

- **22** → SSH
- **80** → HTTP
- **443** → HTTPS

Jika hasil scanning menunjukkan ada port lain yang terbuka, administrator bisa segera menyelidiki **layanan apa yang berjalan di sana**.

#### Port Scanning dalam Reconnaissance (Tahap Pengintaian)

Selain digunakan untuk tujuan administrasi, port scanning juga sering digunakan dalam tahap awal serangan siber yang disebut **reconnaissance**.

Reconnaissance adalah tahap di mana seseorang mencoba **mengumpulkan informasi sebanyak mungkin tentang target** sebelum melakukan serangan lebih lanjut.

Dengan port scanning, seorang penyerang bisa mengetahui:

- Layanan apa saja yang berjalan pada server
- Versi software yang kemungkinan digunakan
- Port mana yang berpotensi memiliki celah keamanan

Sebagai contoh, jika sebuah server memiliki port berikut yang terbuka:

- 22 → SSH
- 80 → HTTP
- 3306 → MySQL

Maka penyerang mungkin akan mencoba mencari **kerentanan pada layanan-layanan tersebut**.

Karena itu, administrator biasanya membatasi port yang terbuka hanya pada layanan yang benar-benar diperlukan.

#### Pentingnya Port Scanning dalam Keamanan Jaringan

Dalam dunia keamanan jaringan, port scanning merupakan salah satu teknik **paling dasar namun sangat penting**.

Teknik ini digunakan dalam berbagai aktivitas seperti:

- **Security auditing**
- **Penetration testing**
- **Network monitoring**
- **System hardening**

Dengan memahami hasil port scanning, seorang administrator dapat:

- Mengurangi permukaan serangan
- Menutup port yang tidak diperlukan
- Memperkuat konfigurasi firewall
- Mengidentifikasi potensi risiko lebih awal

Karena itu, port scanning bukan hanya alat bagi penyerang, tetapi juga merupakan **alat penting bagi defender untuk melindungi sistem mereka**.

### e. Praktik 1 — ICMP Ping (Mode Dasar)

#### Pengantar Mode ICMP pada HPING3

Mode **ICMP** merupakan salah satu mode paling sederhana dalam HPING3. Pada mode ini, HPING3 bekerja dengan cara yang mirip dengan perintah **`ping`** yang biasa digunakan untuk mengecek apakah suatu perangkat di jaringan dapat dijangkau atau tidak.

Perbedaannya adalah, meskipun terlihat seperti `ping`, HPING3 tetap memberikan kontrol yang lebih besar terhadap paket yang dikirim. Artinya, kita tidak hanya bisa mengirim paket ICMP biasa, tetapi juga dapat mempelajari **detail respons jaringan** seperti:

- **Round Trip Time (RTT)**
- **TTL (Time To Live)**
- Struktur respons ICMP

Mode ini sangat cocok digunakan sebagai **latihan awal** sebelum mencoba fitur HPING3 yang lebih kompleks.

#### Perintah Dasar ICMP Ping dengan HPING3

Contoh perintah yang digunakan dalam praktik ini adalah sebagai berikut:

```bash
sudo hping3 -1 -c 4 192.168.0.108
```

Perintah tersebut akan mengirimkan beberapa paket ICMP ke alamat IP **192.168.0.108** untuk melihat apakah host tersebut merespons.

#### Penjelasan Flag yang Digunakan

Agar lebih mudah dipahami, mari kita bahas satu per satu parameter yang digunakan pada perintah tersebut.

**`sudo`**

HPING3 memerlukan akses **root atau administrator** karena tool ini bekerja menggunakan **raw packets**. Sistem operasi biasanya tidak mengizinkan pengguna biasa membuat raw packet karena alasan keamanan.

Oleh karena itu perintah selalu diawali dengan:

```bash
sudo
```

**`hping3`**

Ini adalah program utama yang digunakan untuk membuat dan mengirim paket jaringan secara manual.

**`-1`**

Flag ini digunakan untuk mengaktifkan **mode ICMP**.

Dengan kata lain, HPING3 akan mengirim **ICMP Echo Request**, sama seperti yang dilakukan oleh perintah `ping`.

**`-c 4`**

Flag ini menentukan **jumlah paket yang akan dikirim**.

Pada contoh di atas:

```bash
-c 4
```

Artinya HPING3 hanya akan mengirim **4 paket ICMP** lalu berhenti.

Perilaku ini mirip dengan **default ping pada Windows**, yang juga mengirim 4 paket secara otomatis.

**`192.168.0.108`**

Ini adalah **alamat IP target** yang akan menerima paket ICMP.

Target ini biasanya merupakan:

- komputer lain di jaringan
- server
- virtual machine
- perangkat jaringan seperti router

#### Contoh Output yang Dihasilkan

Jika target dapat dijangkau dan tidak memblokir ICMP, maka HPING3 akan menampilkan respons seperti berikut:

```
HPING 192.168.0.108 (eth0 192.168.0.108): icmp mode set, 28 headers + 0 data bytes
len=28 ip=192.168.0.108 ttl=64 id=12345 icmp_seq=0 rtt=1.2 ms
len=28 ip=192.168.0.108 ttl=64 id=12346 icmp_seq=1 rtt=1.1 ms
len=28 ip=192.168.0.108 ttl=64 id=12347 icmp_seq=2 rtt=1.3 ms
len=28 ip=192.168.0.108 ttl=64 id=12348 icmp_seq=3 rtt=1.0 ms
```

Mari kita pahami beberapa bagian penting dari output tersebut.

**len**

Menunjukkan **ukuran paket** yang diterima.

**ip**

Menunjukkan **alamat IP dari host yang merespons**.

**ttl (Time To Live)**

TTL menunjukkan **jumlah hop maksimum** yang dapat dilalui paket sebelum dibuang oleh router.

Nilai TTL juga sering digunakan untuk:

- memahami jalur jaringan
- memperkirakan sistem operasi target
- mempelajari perilaku routing

**icmp_seq**

Ini adalah **nomor urutan paket ICMP**.

Karena kita mengirim 4 paket, maka urutannya biasanya:

```
0
1
2
3
```

**rtt (Round Trip Time)**

RTT menunjukkan **waktu yang dibutuhkan paket untuk pergi ke target dan kembali lagi**.

Biasanya diukur dalam **milidetik (ms)**.

Contoh:

```
rtt=1.2 ms
```

Artinya paket hanya membutuhkan sekitar **1.2 milidetik** untuk melakukan perjalanan pulang-pergi.

Semakin kecil nilai RTT, biasanya berarti **koneksi jaringan semakin cepat**.

#### Tujuan Praktik Ini

Praktik ICMP ping ini bertujuan untuk membantu kita memahami beberapa konsep dasar dalam jaringan, seperti:

- cara kerja **ICMP Echo Request dan Echo Reply**
- bagaimana **RTT dihitung**
- bagaimana nilai **TTL muncul dalam paket**
- bagaimana HPING3 menampilkan informasi jaringan

Meskipun terlihat sederhana, latihan ini merupakan langkah penting untuk memahami **cara kerja paket jaringan secara lebih mendalam**.

#### Catatan Penting untuk Pengguna Windows

Jika target yang digunakan adalah **komputer Windows**, sering kali perintah ini tidak akan mendapatkan respons.

Hal ini terjadi karena secara default **Windows memblokir ICMP Echo Request** melalui firewall.

Agar komputer Windows dapat merespons ping, kita perlu mengaktifkan rule tertentu di firewall.

Langkahnya adalah sebagai berikut:

1. Buka **Windows Defender Firewall**
2. Pilih **Advanced Settings**
3. Masuk ke **Inbound Rules**
4. Cari rule berikut:

```
Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)
```

5. Klik kanan pada rule tersebut
6. Pilih **Enable Rule**

Setelah rule tersebut diaktifkan, komputer Windows akan mulai **merespons paket ICMP**, sehingga HPING3 dapat menerima balasan dengan normal.

### f. Praktik 2 — TCP SYN Scan (Handshake Probing)

#### Pengantar TCP SYN Scan

Pada praktik ini kita mulai menggunakan salah satu teknik yang sangat umum dalam analisis jaringan, yaitu **TCP SYN Scan**. Teknik ini sering disebut juga sebagai **handshake probing**, karena kita mencoba memulai proses koneksi TCP dengan cara mengirim **paket SYN**, lalu mengamati bagaimana server meresponsnya.

Dalam protokol TCP, sebelum dua perangkat dapat saling bertukar data, mereka harus melewati proses yang disebut **TCP Three-Way Handshake**. Proses ini terdiri dari tiga langkah utama:

1. Client mengirim paket **SYN**
2. Server membalas dengan **SYN-ACK**
3. Client mengirim **ACK**

Barulah setelah tiga langkah ini selesai, koneksi TCP dianggap **terbentuk sepenuhnya**.

Pada teknik **SYN scan**, kita hanya melakukan **langkah pertama**, yaitu mengirim paket SYN untuk melihat apakah server merespons atau tidak. Dari respons tersebut, kita bisa mengetahui **status port pada server target**.

Teknik ini sangat umum digunakan dalam:

- analisis jaringan
- troubleshooting
- security auditing
- penetration testing

#### Perintah TCP SYN Scan dengan HPING3

Berikut adalah contoh perintah yang digunakan untuk melakukan SYN scan menggunakan HPING3:

```bash id="ulxij6"
sudo hping3 -S -p 80 -c 4 <target_IP>
```

Perintah ini akan mengirim paket TCP dengan **flag SYN** ke **port 80** pada server target sebanyak **4 kali**.

#### Penjelasan Flag yang Digunakan

Mari kita pahami setiap bagian dari perintah tersebut.

**`sudo`**

HPING3 membutuhkan akses **root atau administrator** karena program ini bekerja menggunakan **raw packets**. Tanpa hak akses ini, sistem operasi biasanya tidak mengizinkan pembuatan paket TCP secara manual.

**`hping3`**

Ini adalah tool utama yang digunakan untuk membuat dan mengirim paket jaringan yang dapat dikustomisasi.

**`-S`**

Flag ini digunakan untuk mengaktifkan **SYN flag** pada paket TCP.

SYN adalah sinyal yang digunakan untuk **memulai koneksi TCP**. Dengan mengirim paket SYN, kita sebenarnya sedang mencoba **memulai proses handshake** dengan server.

**`-p 80`**

Parameter ini menentukan **port tujuan**.

Dalam contoh ini, paket dikirim ke:

```id="q8qcb8"
port 80
```

Port 80 biasanya digunakan oleh **layanan HTTP (web server)**.

**`-c 4`**

Flag ini menentukan jumlah paket yang akan dikirim.

Dalam contoh ini:

```id="ek30eh"
-c 4
```

Artinya HPING3 akan mengirim **4 paket SYN** ke server target.

**`<target_IP>`**

Ini adalah alamat IP dari server yang ingin kita uji.

Contohnya bisa berupa:

```id="tyub77"
192.168.1.10
```

atau alamat server lain yang ingin dianalisis.

#### Cara Menginterpretasikan Respons Server

Setelah paket SYN dikirim, server biasanya akan memberikan respons tertentu. Respons ini memberikan informasi penting tentang **status port yang kita uji**.

Berikut beberapa kemungkinan respons yang dapat muncul.

**SYN-ACK (SA)**

Jika server membalas dengan:

```id="m3tavp"
SA
```

Artinya server mengirim **SYN-ACK**.

Ini menunjukkan bahwa:

- port **terbuka**
- layanan sedang **aktif**
- server siap melanjutkan proses handshake

Dengan kata lain, server bersedia menerima koneksi TCP pada port tersebut.

**RST-ACK (RA)**

Jika server membalas dengan:

```id="2k94gn"
RA
```

Artinya server mengirim **RST-ACK**.

Ini menunjukkan bahwa:

- port **tertutup**
- tidak ada layanan yang berjalan pada port tersebut

Server secara eksplisit menolak koneksi.

**Tidak Ada Respons**

Jika tidak ada respons sama sekali, biasanya berarti:

- paket diblokir oleh **firewall**
- port difilter oleh sistem keamanan
- server mengabaikan paket tersebut

Dalam kondisi ini, status port sering dikategorikan sebagai **filtered**.

#### Contoh Output yang Berhasil

Jika server merespons semua paket yang dikirim, HPING3 biasanya akan menampilkan ringkasan seperti berikut:

```
4 packets transmitted, 4 received, 0% packet loss
```

Mari kita pahami arti dari output tersebut.

**4 packets transmitted**

HPING3 mengirim **4 paket SYN** ke server target.

**4 received**

Server memberikan **4 respons balik**.

**0% packet loss**

Tidak ada paket yang hilang selama proses komunikasi.

Hal ini menunjukkan bahwa koneksi jaringan ke server target **stabil dan dapat dijangkau dengan baik**.

#### Informasi Tambahan yang Bisa Dianalisis

Selain status port, output HPING3 juga memberikan informasi lain yang berguna, seperti:

- **RTT (Round Trip Time)**
  Waktu yang dibutuhkan paket untuk pergi ke server dan kembali lagi.

- **TTL (Time To Live)**
  Memberikan gambaran tentang jumlah hop yang dilewati paket.

- **Latency jaringan**
  Membantu memahami performa koneksi ke server target.

Dengan informasi tersebut, kita tidak hanya mengetahui **apakah port terbuka**, tetapi juga dapat menganalisis **kualitas koneksi jaringan** ke server tersebut.

#### Mengapa Teknik Ini Penting

Teknik **TCP SYN Scan** merupakan salah satu metode paling dasar dalam analisis jaringan dan keamanan.

Metode ini sering digunakan untuk:

- mengetahui port mana yang terbuka
- mendeteksi layanan aktif pada server
- menganalisis konfigurasi firewall
- memahami perilaku koneksi TCP

Karena teknik ini hanya mengirim paket SYN tanpa menyelesaikan seluruh proses handshake, metode ini juga sering dianggap **lebih ringan dan lebih cepat** dibandingkan koneksi TCP penuh.

Itulah sebabnya SYN scan menjadi salah satu teknik yang sangat populer dalam **network diagnostics dan security analysis**.

### g. Praktik 3 — ACK Probe (Menguji Firewall)

#### Pengantar Teknik ACK Probe

Pada praktik ini kita menggunakan teknik yang disebut **ACK Probe**, yaitu metode untuk menganalisis bagaimana sebuah **firewall memproses paket TCP**.

Tujuan utama teknik ini bukan sekadar mengecek apakah sebuah port terbuka atau tertutup, melainkan untuk memahami **jenis firewall yang digunakan oleh sistem target**.

Secara khusus, teknik ini digunakan untuk mengetahui apakah firewall bersifat:

- **Stateful firewall**
- **Stateless firewall**

Perbedaan keduanya sangat penting dalam dunia keamanan jaringan karena menentukan **seberapa cerdas firewall tersebut dalam menganalisis traffic jaringan**.

Dengan menggunakan HPING3, kita dapat mengirim paket TCP yang berisi **ACK flag**, lalu melihat bagaimana sistem target merespons paket tersebut.

#### Perintah ACK Probe dengan HPING3

Contoh perintah yang digunakan untuk melakukan ACK probe adalah sebagai berikut:

```bash
sudo hping3 -A -p 80 -c 4 <target_IP>
```

Perintah ini akan mengirimkan paket TCP dengan **flag ACK** ke **port 80** pada server target sebanyak **4 kali**.

Tujuannya adalah untuk melihat apakah firewall **menerima, menolak, atau mengabaikan paket tersebut**.

#### Penjelasan Flag yang Digunakan

Mari kita bahas setiap parameter yang digunakan pada perintah tersebut.

**`sudo`**

HPING3 memerlukan hak akses **root atau administrator** karena tool ini bekerja menggunakan **raw packets**. Tanpa akses ini, sistem operasi biasanya tidak mengizinkan program membuat paket TCP secara manual.

**`hping3`**

Program utama yang digunakan untuk membuat dan mengirim paket jaringan yang dapat dikustomisasi.

**`-A`**

Flag ini digunakan untuk mengaktifkan **ACK flag** pada paket TCP.

ACK adalah singkatan dari **Acknowledgment**, yaitu sinyal yang biasanya digunakan dalam TCP untuk **mengonfirmasi penerimaan data dalam sebuah koneksi yang sudah ada**.

Dengan kata lain, paket ACK biasanya muncul **setelah koneksi TCP terbentuk**.

**`-p 80`**

Parameter ini menentukan **port tujuan** paket.

Dalam contoh ini, paket dikirim ke:

```
port 80
```

Port 80 biasanya digunakan oleh **layanan HTTP (web server)**.

**`-c 4`**

Parameter ini menentukan jumlah paket yang akan dikirim.

```
-c 4
```

Artinya HPING3 akan mengirim **4 paket ACK** ke server target.

**`<target_IP>`**

Ini adalah alamat IP dari server yang ingin diuji.

#### Logika di Balik Teknik ACK Probe

Untuk memahami teknik ini dengan benar, kita perlu mengingat kembali bagaimana **koneksi TCP normal bekerja**.

Dalam koneksi TCP, paket ACK biasanya muncul **setelah proses handshake selesai**.

Urutan normalnya adalah:

1. Client mengirim **SYN**
2. Server membalas **SYN-ACK**
3. Client mengirim **ACK**

Setelah langkah ketiga selesai, barulah koneksi TCP dianggap aktif.

Artinya, dalam komunikasi normal:

```
ACK tidak muncul tanpa koneksi sebelumnya
```

Namun pada teknik **ACK Probe**, kita sengaja mengirim **ACK tanpa pernah membuat koneksi sebelumnya**.

Hal ini membuat paket tersebut terlihat **tidak wajar** dari sudut pandang jaringan.

Respons firewall terhadap paket inilah yang kemudian memberikan informasi penting tentang **bagaimana firewall bekerja**.

#### Interpretasi Respons Firewall

Setelah paket ACK dikirim, kita perlu memperhatikan bagaimana target merespons.

Respons tersebut biasanya memberikan petunjuk mengenai jenis firewall yang digunakan.

**Tidak Ada Respons / Paket Di-drop**

Jika tidak ada respons sama sekali, kemungkinan besar paket tersebut **diblokir atau di-drop oleh firewall**.

Hal ini biasanya menunjukkan bahwa firewall tersebut adalah **stateful firewall**.

Firewall stateful mampu:

- melacak status koneksi TCP
- mengetahui apakah sebuah paket merupakan bagian dari sesi yang valid
- memblokir paket yang tidak sesuai dengan konteks koneksi

Karena paket ACK yang kita kirim **tidak berasal dari koneksi yang valid**, firewall akan menganggapnya sebagai paket mencurigakan dan langsung **membuangnya**.

**Respons RST (Reset)**

Jika server merespons dengan paket:

```
RST
```

atau

```
RST-ACK
```

biasanya ini menunjukkan bahwa firewall bersifat **stateless**.

Firewall stateless biasanya hanya memeriksa:

- alamat IP
- nomor port
- beberapa flag dasar

Namun firewall jenis ini **tidak melacak status koneksi TCP secara penuh**.

Akibatnya, paket ACK yang tidak valid masih bisa mencapai server, dan server kemudian merespons dengan **RST** untuk menolak koneksi yang tidak dikenal.

#### Perbedaan Stateful dan Stateless Firewall

Untuk memperjelas, berikut perbedaan utama antara kedua jenis firewall tersebut.

**Stateful Firewall**

Firewall jenis ini lebih cerdas karena mampu:

- melacak status koneksi TCP
- memahami urutan paket dalam sebuah sesi
- memblokir paket yang tidak sesuai dengan state koneksi

Akibatnya, paket ACK yang tidak memiliki sesi sebelumnya biasanya **langsung di-drop**.

**Stateless Firewall**

Firewall stateless lebih sederhana karena hanya memeriksa aturan dasar seperti:

- alamat IP
- port
- jenis protokol

Firewall ini **tidak mengetahui konteks koneksi**, sehingga paket yang tidak valid masih bisa melewati firewall dan akhirnya ditolak oleh server.

#### Mengapa Teknik Ini Penting dalam Keamanan Jaringan

Teknik ACK probe sangat berguna dalam tahap **reconnaissance** dalam keamanan jaringan.

Reconnaissance adalah tahap pengumpulan informasi tentang sistem target sebelum melakukan analisis keamanan lebih lanjut.

Dengan teknik ini, seorang analis keamanan dapat mengetahui:

- jenis firewall yang digunakan
- bagaimana firewall memproses paket yang tidak biasa
- seberapa ketat sistem keamanan jaringan tersebut

Informasi seperti ini sangat berguna dalam **penetration testing** karena membantu memahami **arsitektur pertahanan jaringan target** sebelum melakukan pengujian yang lebih kompleks.

### h. Praktik 4 — UDP Probe

#### Pengantar UDP Probe

Selain TCP dan ICMP, HPING3 juga dapat digunakan untuk mengirim paket menggunakan **protokol UDP**. Dalam praktik ini, kita akan melakukan **UDP probe**, yaitu mengirim paket UDP ke sebuah port tertentu untuk melihat bagaimana server meresponsnya.

Teknik ini sering digunakan untuk menguji layanan yang berjalan menggunakan UDP, seperti:

- **DNS**
- **DHCP**
- **SNMP**
- **NTP**

Sebagai contoh dalam praktik ini, kita akan mencoba menguji **port 53**, yaitu port standar yang digunakan oleh **DNS (Domain Name System)**.

Berbeda dengan TCP, protokol UDP memiliki karakteristik yang lebih sederhana karena **tidak menggunakan mekanisme handshake**. Hal ini membuat analisis respons UDP sedikit lebih sulit dibandingkan TCP.

#### Perintah UDP Probe dengan HPING3

Contoh perintah yang digunakan untuk melakukan UDP probe adalah sebagai berikut:

```bash
sudo hping3 --udp -p 53 -c 4 <target_IP>
```

Perintah ini akan mengirim **paket UDP sebanyak 4 kali** ke **port 53** pada server target.

Tujuannya adalah untuk melihat apakah port tersebut:

- terbuka
- tertutup
- atau difilter oleh firewall

#### Penjelasan Flag yang Digunakan

Mari kita pahami setiap parameter yang digunakan pada perintah tersebut.

**`sudo`**

HPING3 memerlukan hak akses **root atau administrator** karena tool ini bekerja menggunakan **raw packets**. Sistem operasi biasanya membatasi akses ini untuk mencegah penyalahgunaan paket jaringan.

**`hping3`**

Program utama yang digunakan untuk membuat dan mengirim paket jaringan yang dapat dikustomisasi.

**`--udp`**

Flag ini digunakan untuk menentukan bahwa paket yang dikirim menggunakan **protokol UDP**.

Secara default, HPING3 biasanya menggunakan **TCP**, sehingga kita perlu secara eksplisit menyebutkan bahwa kita ingin menggunakan UDP.

**`-p 53`**

Parameter ini menentukan **port tujuan** paket.

Dalam contoh ini:

```
port 53
```

Port ini biasanya digunakan oleh **layanan DNS** untuk menangani permintaan resolusi domain.

**`-c 4`**

Parameter ini menentukan jumlah paket yang akan dikirim.

```
-c 4
```

Artinya HPING3 akan mengirim **4 paket UDP** ke server target.

**`<target_IP>`**

Ini adalah alamat IP dari server yang ingin diuji.

#### Cara Kerja Analisis UDP

Karena UDP tidak memiliki mekanisme handshake seperti TCP, cara kita menginterpretasikan responsnya sedikit berbeda.

Pada TCP, kita bisa dengan mudah melihat status port berdasarkan respons seperti:

- SYN-ACK
- RST

Namun pada UDP, server sering kali **tidak memberikan respons sama sekali**, bahkan jika port tersebut terbuka.

Oleh karena itu, analisis UDP biasanya dilakukan dengan melihat **respons ICMP yang dikirim oleh sistem target**.

#### Interpretasi Respons UDP

Setelah paket UDP dikirim, beberapa kemungkinan respons dapat terjadi.

**ICMP “Port Unreachable”**

Jika server mengirim respons ICMP seperti:

```
ICMP Port Unreachable
```

artinya:

- port yang kita tuju **tidak memiliki layanan yang berjalan**
- sistem target secara eksplisit menyatakan bahwa port tersebut **tertutup**

Ini adalah respons yang cukup umum ketika kita mengirim paket UDP ke port yang tidak digunakan.

**Tidak Ada Respons**

Jika tidak ada respons sama sekali, ada dua kemungkinan yang bisa terjadi:

1. **Port terbuka**

   Beberapa layanan UDP memang **tidak memberikan respons** terhadap paket yang tidak valid atau tidak sesuai dengan protokol aplikasinya.

   Misalnya, server DNS mungkin hanya merespons jika paket yang dikirim adalah **query DNS yang valid**.

2. **Port difilter oleh firewall**

   Firewall juga dapat memblokir paket UDP tanpa memberikan respons apa pun.

Karena kedua kondisi tersebut menghasilkan hasil yang sama (tidak ada respons), maka dalam analisis keamanan jaringan kondisi ini sering dikategorikan sebagai:

```
open | filtered
```

Artinya port tersebut **mungkin terbuka atau mungkin difilter**.

#### Perbedaan Penting antara TCP dan UDP

Agar lebih jelas, berikut perbedaan utama antara kedua protokol ini dalam konteks scanning.

**TCP**

TCP memiliki mekanisme:

- handshake
- konfirmasi paket
- pengelolaan koneksi

Karena itu, respons TCP biasanya **lebih mudah dianalisis**.

**UDP**

UDP bersifat:

- connectionless
- tidak memiliki handshake
- tidak memiliki konfirmasi bawaan

Akibatnya, hasil scanning UDP sering kali **lebih ambigu** karena server tidak selalu memberikan respons.

#### Mengapa UDP Probe Penting

Meskipun lebih sulit dianalisis, UDP probe tetap sangat penting dalam analisis jaringan karena banyak layanan penting menggunakan protokol ini.

Contoh layanan yang menggunakan UDP antara lain:

- **DNS (53)**
- **NTP (123)**
- **SNMP (161)**
- **DHCP (67/68)**

Dengan melakukan UDP probe, administrator jaringan dapat:

- mengetahui apakah layanan UDP tertentu aktif
- menguji konfigurasi firewall
- mendeteksi layanan yang tidak seharusnya terbuka

Karena itulah teknik ini sering digunakan dalam:

- **network diagnostics**
- **security auditing**
- **penetration testing**

Meskipun hasilnya tidak selalu jelas seperti pada TCP, UDP scanning tetap merupakan bagian penting dalam **analisis keamanan jaringan secara menyeluruh**.

## 3. Hubungan Antar Konsep

Berikut adalah alur pemahaman yang menghubungkan semua konsep di atas:

```
[HPING3 sebagai Tool]
        │
        ├──► Mengirim paket RAW (ICMP / TCP / UDP / raw IP)
        │           │
        │           ▼
        │   [Konsep PORT]
        │   Port = endpoint virtual untuk layanan di sebuah mesin
        │   (SSH:22, HTTP:80, DNS:53, dst.)
        │           │
        │           ▼
        │   [Port Scanning]
        │   Mendeteksi port terbuka/tertutup/difilter
        │           │
        ├──► [TCP SYN Scan] → Temukan port terbuka
        ├──► [ACK Probe]   → Identifikasi tipe firewall (stateful vs stateless)
        └──► [UDP Probe]   → Uji layanan berbasis UDP (misal: DNS)
```

Setiap teknik memanfaatkan kemampuan HPING3 untuk **mengkustomisasi paket** di level yang lebih rendah dibanding tool biasa. Hasilnya memberikan informasi yang jauh lebih kaya untuk tujuan administrasi jaringan maupun pengujian keamanan (penetration testing).

---

## 4. Kesimpulan

HPING3 adalah tool yang jauh lebih powerful daripada `ping` biasa. Dengan HPING3, pengguna dapat:

1. **Mengirim paket ICMP** untuk uji konektivitas dasar dan analisis TTL.
2. **Melakukan TCP SYN scan** untuk mendeteksi port yang terbuka pada target.
3. **Mengirim ACK probe** untuk mengidentifikasi apakah firewall bersifat _stateful_ atau _stateless_.
4. **Menguji port UDP** seperti DNS untuk mengetahui status layanan berbasis UDP.

Pemahaman konsep **port** sangat penting sebagai fondasi sebelum melakukan port scanning. Setiap teknik yang didemonstrasikan memiliki kegunaan nyata baik dalam **administrasi jaringan** maupun **ethical hacking / penetration testing**.

> 🔐 **Selalu gunakan tool ini secara bertanggung jawab** — hanya di lingkungan lab yang terkontrol atau dengan izin eksplisit dari pemilik sistem.

---

_Catatan dibuat berdasarkan subtitle video pembelajaran HPING3._
