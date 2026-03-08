# 📝 NMAP – Panduan Scanning Jaringan (Bagian 1)

## 1. Ringkasan Singkat

Video ini memperkenalkan **NMAP** (_Network Mapper_), alat pemindaian jaringan standar industri yang digunakan oleh para profesional keamanan siber. Materi mencakup langkah-langkah dari yang paling dasar hingga yang lebih canggih: mulai dari _ping scan_ untuk mengecek apakah host aktif, lalu berbagai teknik _TCP port scan_ (SYN, Connect, ACK, FIN), hingga persiapan menuju deteksi layanan, fingerprinting sistem operasi, dan penggunaan _NMAP Scripting Engine_ (NSE) di video berikutnya.

Semua demonstrasi dilakukan di lingkungan lab menggunakan **Kali Linux** sebagai mesin penyerang dan **OWASP Broken Web Application** sebagai target yang sengaja dibuat rentan.

---

## 2. Konsep Utama

### a. Persiapan dan Etika Penggunaan NMAP

Sebelum mulai melakukan proses **scanning menggunakan Nmap**, ada beberapa hal penting yang perlu dipahami terlebih dahulu. Persiapan ini bukan hanya berkaitan dengan aspek teknis, tetapi juga menyangkut **etika dan legalitas** penggunaan alat tersebut.

Nmap adalah tool yang sangat kuat untuk melakukan **network discovery dan port scanning**. Karena kemampuannya yang cukup dalam untuk memetakan sistem target, penggunaannya harus dilakukan secara bertanggung jawab.

#### Menjalankan Nmap dengan Hak Akses Root

Untuk mendapatkan hasil scan yang paling lengkap dan akurat, Nmap sebaiknya dijalankan menggunakan **hak akses root**. Di sistem operasi seperti **Kali Linux**, hal ini biasanya dilakukan dengan menggunakan perintah `sudo` atau dengan masuk sebagai user `root`.

Beberapa teknik scanning yang digunakan oleh Nmap bekerja pada **level jaringan yang lebih rendah (low-level packet crafting)**. Teknik seperti ini membutuhkan izin sistem yang lebih tinggi agar dapat membuat dan mengirim paket jaringan secara langsung.

Contoh menjalankan Nmap sebagai root:

```bash
sudo nmap 192.168.1.1
```

Perintah tersebut akan melakukan scanning terhadap host dengan alamat IP `192.168.1.1`.

Jika Nmap dijalankan tanpa hak akses root, beberapa metode scan mungkin tidak dapat digunakan secara optimal. Akibatnya, hasil scan bisa menjadi **kurang lengkap atau kurang akurat**.

#### Menyimpan Output Scan

Saat melakukan scanning, sangat disarankan untuk **menyimpan hasil output scan**. Hal ini penting terutama dalam konteks:

- **Dokumentasi**
- **Analisis lanjutan**
- **Pelaporan keamanan**
- **Audit atau penetration testing**

Nmap menyediakan beberapa format output yang bisa digunakan. Contoh sederhana untuk menyimpan output ke file:

```bash
sudo nmap 192.168.1.1 -oN hasil_scan.txt
```

Penjelasan perintah:

- `sudo` → menjalankan Nmap dengan hak akses root
- `nmap` → tool yang digunakan untuk scanning
- `192.168.1.1` → target yang akan di-scan
- `-oN` → menyimpan output dalam format **normal** ke file
- `hasil_scan.txt` → nama file tempat hasil scan disimpan

Dengan cara ini, hasil scanning dapat dibuka kembali kapan saja tanpa harus menjalankan scan ulang.

#### Menghormati Batas Lingkungan Lab

Hal lain yang tidak kalah penting adalah memahami **batasan lingkungan tempat kita melakukan scanning**.

Scanning sebaiknya hanya dilakukan pada:

- **Lab pribadi**
- **Mesin virtual yang disediakan untuk latihan**
- **Sistem yang memang kita miliki**
- **Target yang memberikan izin eksplisit**

Sebaliknya, kita **tidak boleh melakukan scanning terhadap sistem produksi atau sistem milik orang lain tanpa izin**. Hal ini dapat dianggap sebagai aktivitas yang mencurigakan atau bahkan serangan terhadap sistem tersebut.

Dalam praktik profesional seperti **penetration testing**, sebelum melakukan scanning biasanya akan ada **dokumen persetujuan resmi (authorization)** dari pemilik sistem.

⚠️ Penting untuk dipahami bahwa **melakukan port scanning tanpa izin merupakan tindakan ilegal di banyak yurisdiksi**. Oleh karena itu, selalu pastikan bahwa aktivitas scanning dilakukan dalam konteks yang sah dan diizinkan.

#### Memahami Status Port dalam Hasil Scan

Setelah Nmap melakukan scanning terhadap sebuah target, hasilnya akan menampilkan status dari setiap port yang diperiksa. Status ini memberikan gambaran mengenai kondisi port tersebut.

Beberapa status yang paling umum adalah sebagai berikut:

**`open`**

Status `open` menunjukkan bahwa port tersebut **aktif dan sedang menjalankan sebuah layanan**. Artinya, sistem target menerima koneksi pada port tersebut.

Contoh layanan yang sering ditemukan:

- Port `80` → Web server (HTTP)
- Port `443` → Web server dengan HTTPS
- Port `22` → SSH

Jika sebuah port berstatus `open`, maka biasanya ada **service yang dapat diakses atau dianalisis lebih lanjut**.

**`closed`**

Status `closed` berarti port tersebut **dapat dijangkau oleh Nmap**, tetapi **tidak ada layanan yang berjalan di port tersebut**.

Dengan kata lain:

- Host target aktif
- Port merespons
- Namun tidak ada aplikasi yang mendengarkan koneksi pada port tersebut

Informasi ini tetap berguna karena menunjukkan bahwa **host target memang hidup dan dapat dijangkau melalui jaringan**.

**`filtered`**

Status `filtered` menunjukkan bahwa Nmap **tidak dapat menentukan apakah port tersebut terbuka atau tertutup**. Hal ini biasanya terjadi karena adanya **firewall, packet filtering, atau sistem keamanan jaringan lainnya** yang memblokir atau mengabaikan paket scan.

Akibatnya, Nmap tidak menerima respons yang cukup untuk menentukan kondisi port tersebut secara pasti.

Dalam banyak kasus, status `filtered` menunjukkan bahwa sistem target memiliki **lapisan keamanan tambahan** yang melindungi port tersebut dari probing langsung.

### b. Ping Scan – Mengecek Apakah Host Aktif

Dalam proses **network scanning**, langkah pertama yang biasanya dilakukan adalah memastikan apakah sebuah host benar-benar **aktif (live)** di jaringan. Tidak ada gunanya melakukan scanning port jika mesin target ternyata tidak sedang menyala atau tidak terhubung ke jaringan.

Untuk tujuan ini, Nmap menyediakan teknik yang disebut **Ping Scan**.

Ping scan digunakan untuk **mendeteksi keberadaan host di jaringan tanpa melakukan scanning terhadap port sama sekali**. Dengan kata lain, teknik ini hanya menjawab satu pertanyaan sederhana:

> Apakah host ini aktif atau tidak?

Karena tidak melakukan pemeriksaan port, ping scan biasanya **lebih cepat dan lebih ringan** dibandingkan jenis scanning lainnya.

#### Perintah Ping Scan dengan Nmap

Untuk menjalankan ping scan menggunakan Nmap, kita dapat menggunakan flag `-sn`.

Contoh perintahnya adalah sebagai berikut:

```bash
sudo nmap -sn 172.17.199.151
```

Mari kita lihat arti dari setiap bagian perintah tersebut:

- `sudo`
  Digunakan untuk menjalankan Nmap dengan hak akses **root**, sehingga Nmap dapat menggunakan metode deteksi host yang lebih lengkap.

- `nmap`
  Program yang digunakan untuk melakukan proses scanning jaringan.

- `-sn`
  Singkatan dari **scan no port**. Artinya Nmap hanya melakukan **host discovery**, tanpa memeriksa port mana pun pada target.

- `172.17.199.151`
  Alamat IP dari host yang ingin kita cek apakah aktif atau tidak.

Ketika perintah ini dijalankan, Nmap akan mengirim beberapa jenis paket jaringan (seperti ICMP, ARP, atau TCP ping) untuk melihat apakah target memberikan respons.

Jika target merespons, maka Nmap akan menganggap host tersebut **aktif (up)**.

#### Contoh Output Ping Scan

Berikut contoh hasil output dari perintah ping scan:

```
Nmap scan report for owaspbwa.msnet (172.17.199.151)
Host is up (0.00046s latency).
MAC Address: 00:0C:29:XX:XX:XX (VMware)
```

Mari kita pahami informasi yang ditampilkan oleh Nmap tersebut.

#### Informasi Target Host

Baris pertama biasanya menunjukkan **nama host atau domain yang terkait dengan alamat IP target**.

```
Nmap scan report for owaspbwa.msnet (172.17.199.151)
```

Artinya:

- Target memiliki hostname **`owaspbwa.msnet`**
- IP address yang digunakan adalah **`172.17.199.151`**

Jika tidak ada hostname yang dapat ditemukan, Nmap biasanya hanya akan menampilkan alamat IP saja.

#### Status Host

Baris berikutnya menunjukkan apakah host tersebut aktif atau tidak.

```
Host is up (0.00046s latency).
```

Penjelasan:

- **Host is up**
  Menandakan bahwa mesin target **merespons paket yang dikirim oleh Nmap**, sehingga dapat dipastikan bahwa host tersebut aktif di jaringan.

- **Latency (0.00046s)**
  Menunjukkan **waktu respons** dari target terhadap paket yang dikirim. Nilai ini biasanya sangat kecil jika host berada dalam jaringan lokal.

Semakin kecil nilai latency, biasanya semakin **dekat atau cepat koneksi jaringan** ke target tersebut.

#### Informasi MAC Address

Baris berikutnya menampilkan informasi **MAC address** dari perangkat target.

```
MAC Address: 00:0C:29:XX:XX:XX (VMware)
```

MAC address adalah **alamat fisik dari network interface card (NIC)** pada perangkat tersebut.

Selain itu, Nmap juga sering mencoba **mengidentifikasi vendor perangkat berdasarkan prefix MAC address**. Dalam contoh ini, vendor yang terdeteksi adalah:

- **VMware**

Hal ini menunjukkan bahwa mesin target kemungkinan besar adalah **virtual machine yang berjalan di platform VMware**.

Informasi seperti ini cukup berguna dalam proses **network reconnaissance**, karena kita dapat mengetahui apakah target merupakan:

- mesin fisik
- mesin virtual
- atau bahkan perangkat jaringan tertentu

#### Kapan Ping Scan Digunakan

Ping scan sangat berguna ketika kita ingin melakukan **pemetaan awal jaringan**. Biasanya teknik ini digunakan untuk:

- Menemukan **host yang aktif** dalam sebuah jaringan
- Menghemat waktu dengan **menghindari scanning terhadap host yang tidak aktif**
- Membuat daftar target sebelum melakukan **port scanning yang lebih mendalam**

Contohnya, jika kita memiliki satu subnet seperti `192.168.1.0/24`, kita bisa terlebih dahulu menjalankan ping scan untuk menemukan host mana saja yang aktif. Setelah itu, barulah kita melakukan scanning port terhadap host-host tersebut.

Dengan cara ini, proses scanning menjadi **lebih efisien dan lebih terstruktur**.

### c. Default Scan – Melihat 1000 Port TCP Teratas

Ketika **Nmap dijalankan tanpa menggunakan flag khusus**, tool ini akan secara otomatis menjalankan metode scanning standar yang disebut **default scan**. Dalam mode ini, Nmap akan memeriksa **1000 port TCP yang paling umum digunakan** pada target.

Seribu port tersebut bukan dipilih secara acak. Nmap menggunakan **daftar port yang paling sering muncul dalam berbagai layanan jaringan**, sehingga kemungkinan menemukan service yang aktif menjadi lebih besar. Dengan pendekatan ini, proses scanning tetap **relatif cepat**, tetapi masih memberikan informasi yang cukup berguna.

#### Menjalankan Default Scan

Perintah untuk menjalankan default scan sebenarnya sangat sederhana. Kita hanya perlu menuliskan perintah Nmap diikuti dengan alamat IP target.

Contoh perintahnya:

```bash
sudo nmap 172.17.199.151
```

Mari kita pahami bagian dari perintah tersebut:

- `sudo`
  Menjalankan Nmap dengan hak akses **root**, yang memungkinkan teknik scanning tertentu bekerja lebih optimal.

- `nmap`
  Program yang digunakan untuk melakukan scanning jaringan.

- `172.17.199.151`
  Alamat IP dari host target yang akan dipindai.

Ketika perintah ini dijalankan, Nmap akan mencoba menghubungi **1000 port TCP paling umum** pada target dan melihat bagaimana respons dari setiap port tersebut. Berdasarkan respons yang diterima, Nmap kemudian menentukan status dari masing-masing port.

#### Contoh Hasil Scan pada Target OWASP BWA

Dalam contoh yang digunakan pada video, target yang dipindai adalah mesin **OWASP BWA (Broken Web Applications)**. Mesin ini memang sengaja dibuat untuk latihan keamanan aplikasi web.

Setelah scanning dilakukan, beberapa port ditemukan dalam keadaan **terbuka (open)**.

Berikut beberapa port yang terdeteksi:

| Port      | Layanan         |
| --------- | --------------- |
| 22        | SSH             |
| 80        | HTTP            |
| 139       | NetBIOS-SSN     |
| 143       | IMAP            |
| 443       | HTTPS           |
| 8080      | HTTP Alternatif |
| 8081      | HTTP Alternatif |
| 5001, 405 | Layanan lainnya |

Sekarang mari kita bahas secara singkat arti dari beberapa port tersebut.

#### Port 22 – SSH

Port `22` biasanya digunakan oleh layanan **SSH (Secure Shell)**. Layanan ini memungkinkan administrator atau pengguna untuk **mengakses server secara remote melalui terminal** dengan koneksi yang terenkripsi.

Jika port ini terbuka, kemungkinan besar server tersebut dapat diakses menggunakan SSH.

#### Port 80 – HTTP

Port `80` merupakan port standar untuk **HTTP (Hypertext Transfer Protocol)**. Ini adalah protokol yang digunakan oleh **web server** untuk menyajikan halaman web tanpa enkripsi.

Jika port ini terbuka, biasanya kita dapat membuka alamat IP target melalui browser.

Contoh:

```
http://172.17.199.151
```

#### Port 443 – HTTPS

Port `443` adalah port standar untuk **HTTPS**, yaitu versi aman dari HTTP yang menggunakan **enkripsi TLS/SSL**.

Jika port ini terbuka, berarti server kemungkinan menyediakan akses web yang aman.

Contoh akses melalui browser:

```
https://172.17.199.151
```

#### Port 139 – NetBIOS Session Service

Port `139` biasanya digunakan oleh **NetBIOS Session Service**, yang sering ditemukan pada sistem berbasis **Windows networking**. Layanan ini berkaitan dengan **file sharing dan komunikasi antar komputer dalam jaringan lokal**.

Dalam konteks keamanan jaringan, port ini sering diperiksa karena bisa memberikan informasi tambahan tentang sistem target.

#### Port 143 – IMAP

Port `143` digunakan oleh **IMAP (Internet Message Access Protocol)**, yaitu protokol yang digunakan oleh email client untuk **mengambil email dari mail server**.

Jika port ini terbuka, kemungkinan server tersebut juga menjalankan **layanan email**.

#### Port 8080 dan 8081 – HTTP Alternatif

Port `8080` dan `8081` sering digunakan sebagai **alternatif dari port 80 untuk layanan HTTP**. Biasanya port ini digunakan oleh:

- aplikasi web development
- proxy server
- panel administrasi
- aplikasi web yang berjalan di container atau environment khusus

Sebagai contoh, sebuah aplikasi web mungkin bisa diakses melalui:

```
http://172.17.199.151:8080
```

atau

```
http://172.17.199.151:8081
```

#### Port Lainnya

Selain port yang sudah disebutkan, terkadang juga ditemukan port lain seperti `5001` atau `405`. Port-port ini biasanya digunakan oleh **layanan aplikasi tertentu** yang tidak selalu standar.

Untuk mengetahui secara pasti layanan apa yang berjalan pada port tersebut, biasanya kita perlu melakukan **service detection** menggunakan teknik scanning tambahan di Nmap.

#### Menyimpan Hasil Scan ke File

Dalam praktik keamanan jaringan atau penetration testing, hasil scanning hampir selalu **disimpan ke dalam file**. Hal ini penting agar hasil analisis dapat didokumentasikan dan dipelajari kembali di kemudian hari.

Nmap menyediakan beberapa opsi untuk menyimpan output. Salah satu yang paling sederhana adalah menggunakan flag `-o`.

Contoh:

```bash
sudo nmap 172.17.199.151 -oN hasil_scan.txt
```

Penjelasan:

- `-oN` berarti **output normal**, yaitu format yang sama seperti yang tampil di terminal.
- `hasil_scan.txt` adalah nama file tempat hasil scan disimpan.

Dengan menyimpan hasil scan seperti ini, kita bisa:

- Membaca kembali hasil scanning tanpa menjalankan ulang proses scan
- Melakukan analisis lebih lanjut
- Menyertakan hasil scan dalam laporan keamanan

Pendekatan ini sangat umum digunakan dalam **audit keamanan, penetration testing, maupun dokumentasi teknis jaringan**.

### d. SYN Scan (`-sS`) – Stealth Scan

Salah satu teknik scanning yang paling terkenal dan paling sering digunakan dalam Nmap adalah **SYN scan**. Teknik ini sering disebut juga sebagai **stealth scan**, karena prosesnya tidak menyelesaikan koneksi TCP secara penuh.

Untuk memahami kenapa metode ini disebut “stealth”, kita perlu melihat sedikit bagaimana **koneksi TCP normal terbentuk**.

#### Sekilas tentang TCP Three-Way Handshake

Dalam komunikasi jaringan berbasis TCP, sebuah koneksi biasanya dibangun melalui proses yang disebut **three-way handshake**. Proses ini terdiri dari tiga langkah utama:

1. **Client mengirim paket `SYN`**
   Ini adalah permintaan awal untuk membuka koneksi ke server.

2. **Server merespons dengan `SYN-ACK`**
   Jika port pada server terbuka dan menerima koneksi, server akan mengirim balasan berupa `SYN-ACK`.

3. **Client mengirim `ACK`**
   Client mengirim paket `ACK` untuk mengonfirmasi koneksi. Setelah langkah ini selesai, koneksi TCP dianggap berhasil dibuat dan komunikasi data dapat dimulai.

Dalam bentuk sederhana, alurnya terlihat seperti ini:

```
Client → SYN → Server
Client ← SYN-ACK ← Server
Client → ACK → Server
```

Setelah tiga langkah ini selesai, koneksi TCP menjadi **fully established**.

#### Cara Kerja SYN Scan

Berbeda dengan koneksi TCP normal, **SYN scan tidak pernah menyelesaikan three-way handshake**.

Langkah-langkahnya kira-kira seperti berikut:

1. Scanner mengirim paket `SYN` ke port target.
2. Jika port **terbuka**, server akan merespons dengan `SYN-ACK`.
3. Namun scanner **tidak mengirim ACK balasan**.
4. Sebagai gantinya, scanner biasanya langsung menghentikan proses koneksi (sering dengan paket RST atau cukup membiarkan koneksi tidak selesai).

Karena koneksi tidak pernah selesai, server **tidak benar-benar menganggap koneksi itu berhasil dibuat**.

#### Dampak pada Logging Server

Pada banyak sistem, **log koneksi biasanya hanya dicatat ketika koneksi TCP berhasil terbentuk sepenuhnya**. Karena SYN scan tidak menyelesaikan handshake, aktivitas ini sering kali:

- Tidak tercatat sebagai koneksi penuh
- Lebih sulit terlihat dalam log standar

Inilah alasan mengapa teknik ini sering disebut sebagai **stealth scan**.

Namun penting untuk dipahami bahwa **“stealth” bukan berarti tidak bisa terdeteksi sama sekali**. Sistem keamanan modern seperti firewall, IDS, atau IPS masih dapat mengenali pola scanning tertentu.

#### Menjalankan SYN Scan dengan Nmap

Untuk menggunakan teknik SYN scan di Nmap, kita dapat menggunakan flag `-sS`.

Contoh perintahnya:

```bash
sudo nmap -sS 172.17.199.151
```

Penjelasan perintah:

- `sudo`
  SYN scan membutuhkan hak akses **root**, karena Nmap harus membuat dan mengirim paket TCP secara langsung.

- `nmap`
  Tool yang digunakan untuk melakukan scanning jaringan.

- `-sS`
  Mengaktifkan **SYN scan**, yaitu metode scanning berbasis paket SYN.

- `172.17.199.151`
  Alamat IP target yang akan dipindai.

Ketika perintah ini dijalankan, Nmap akan mengirim paket `SYN` ke banyak port pada target dan kemudian menganalisis respons yang diterima.

#### Interpretasi Respons Port

Respons dari server akan menentukan status port tersebut.

Beberapa kemungkinan respons yang biasanya terjadi adalah:

**Jika port terbuka (open)**

Server akan merespons dengan paket `SYN-ACK`.

Ini menunjukkan bahwa port tersebut menerima koneksi.

**Jika port tertutup (closed)**

Server akan merespons dengan paket `RST`.

Ini berarti port dapat dijangkau, tetapi tidak ada layanan yang berjalan di sana.

**Jika port difilter (filtered)**

Tidak ada respons yang jelas, atau paket diblokir oleh firewall.

Dalam kasus ini, Nmap biasanya menandai port sebagai `filtered`.

#### Mengapa Disebut Stealth Scan

Istilah **stealth scan** berasal dari karakteristik metode ini yang relatif lebih “halus” dibandingkan scanning TCP biasa.

Ada beberapa alasan utama:

Pertama, koneksi TCP **tidak pernah benar-benar selesai**. Karena handshake tidak lengkap, banyak sistem logging standar tidak mencatatnya sebagai koneksi penuh.

Kedua, Nmap biasanya mengirim paket scanning dengan **kecepatan yang terkontrol**. Scanner yang terlalu agresif biasanya akan mengirim ribuan paket dalam waktu sangat singkat. Pola seperti ini mudah dikenali oleh firewall atau sistem deteksi intrusi.

Sebaliknya, SYN scan dapat menambahkan **delay kecil di antara paket-paket yang dikirim**, sehingga pola scanning terlihat lebih “alami” dan tidak terlalu mencurigakan.

Meskipun demikian, dalam jaringan modern yang memiliki sistem keamanan canggih, teknik ini tetap dapat terdeteksi oleh **IDS (Intrusion Detection System)** atau **IPS (Intrusion Prevention System)** jika pola scanning dikenali.

Karena itu, SYN scan lebih tepat dianggap sebagai teknik scanning yang **lebih efisien dan relatif lebih tersembunyi**, bukan sepenuhnya tidak terdeteksi.

### e. Connect Scan (`-sT`) – Full TCP Handshake

Selain **SYN scan**, Nmap juga menyediakan metode scanning lain yang disebut **Connect scan**. Teknik ini menggunakan cara yang lebih “resmi” dalam membuat koneksi TCP, yaitu melalui **system call `connect()` yang disediakan oleh sistem operasi**.

Berbeda dengan SYN scan yang hanya memulai koneksi tanpa menyelesaikannya, **Connect scan benar-benar menyelesaikan seluruh proses TCP handshake**. Karena itu, metode ini sering disebut sebagai **full TCP handshake scan**.

Untuk memahami cara kerja teknik ini, kita perlu melihat kembali bagaimana **koneksi TCP normal terbentuk**.

#### Proses TCP Three-Way Handshake

Dalam komunikasi TCP, sebuah koneksi dibangun melalui proses yang disebut **three-way handshake**. Proses ini terdiri dari tiga tahap pertukaran paket antara client dan server.

Secara sederhana, prosesnya dapat digambarkan seperti berikut:

```
Client          Server
  |--- SYN ------->|   (Hello, apakah kamu ada?)
  |<-- SYN-ACK ----|   (Ya, aku di sini!)
  |--- ACK ------->|   (Baik, koneksi terbentuk)
```

Mari kita jelaskan langkah-langkah tersebut:

1. **Client mengirim paket `SYN`**
   Client mencoba membuka koneksi dengan mengirimkan paket SYN ke server.

2. **Server merespons dengan `SYN-ACK`**
   Jika port pada server terbuka, server akan membalas dengan paket SYN-ACK sebagai tanda bahwa ia siap menerima koneksi.

3. **Client mengirim `ACK`**
   Client mengirimkan paket ACK untuk menyelesaikan proses handshake. Setelah langkah ini selesai, koneksi TCP dianggap berhasil terbentuk.

Setelah koneksi terbentuk, client dan server dapat mulai **bertukar data melalui koneksi tersebut**.

#### Cara Kerja Connect Scan

Pada **Connect scan**, Nmap tidak membuat paket TCP secara manual seperti pada SYN scan. Sebaliknya, Nmap meminta sistem operasi untuk membuat koneksi menggunakan **fungsi `connect()`**.

Alurnya kira-kira seperti ini:

1. Nmap mencoba melakukan koneksi ke sebuah port menggunakan `connect()`.
2. Sistem operasi menjalankan **three-way handshake secara penuh**.
3. Jika koneksi berhasil, berarti port tersebut **terbuka**.
4. Setelah itu, Nmap biasanya langsung **menutup koneksi** dan melanjutkan scanning ke port berikutnya.

Karena koneksi benar-benar terbentuk, server akan melihatnya seperti **koneksi normal dari sebuah aplikasi**.

#### Menjalankan Connect Scan dengan Nmap

Untuk menjalankan Connect scan, kita dapat menggunakan flag `-sT`.

Contoh perintahnya:

```bash
sudo nmap -sT 172.17.199.151
```

Penjelasan perintah:

- `sudo`
  Menjalankan perintah dengan hak akses administrator. Meskipun Connect scan sebenarnya tidak memerlukan root, penggunaan `sudo` tetap umum dalam banyak contoh scanning.

- `nmap`
  Tool yang digunakan untuk melakukan scanning jaringan.

- `-sT`
  Mengaktifkan **TCP Connect scan**.

- `172.17.199.151`
  Target host yang akan dipindai.

Saat perintah ini dijalankan, Nmap akan mencoba membuat koneksi TCP ke setiap port yang dipindai. Jika koneksi berhasil, port tersebut akan dilaporkan sebagai **open**.

#### Mengapa Connect Scan Lebih Mudah Terdeteksi

Salah satu kekurangan Connect scan adalah **lebih mudah terdeteksi oleh sistem keamanan**.

Hal ini terjadi karena:

- Koneksi TCP benar-benar terbentuk
- Server melihat aktivitas tersebut sebagai koneksi yang valid
- Sistem logging server biasanya mencatat koneksi yang berhasil

Akibatnya, aktivitas scanning bisa lebih mudah terlihat dalam:

- log server
- firewall log
- sistem monitoring jaringan
- IDS (Intrusion Detection System)

Sebaliknya, pada **SYN scan**, koneksi tidak pernah selesai sehingga dalam beberapa sistem logging aktivitasnya tidak terlihat sebagai koneksi penuh.

#### Keuntungan Connect Scan

Meskipun lebih mudah terdeteksi, Connect scan tetap memiliki beberapa kelebihan.

Salah satu keuntungan utamanya adalah **tidak memerlukan hak akses root**. Karena teknik ini menggunakan fungsi jaringan standar dari sistem operasi, Nmap tidak perlu membuat paket TCP secara manual (raw socket).

Hal ini membuat Connect scan sangat berguna dalam situasi seperti:

- scanning dari **user biasa (non-root)**
- lingkungan sistem yang membatasi akses raw socket
- sistem operasi tertentu yang tidak mendukung teknik packet crafting

Dengan kata lain, **Connect scan adalah metode scanning yang lebih sederhana dan kompatibel**, meskipun tidak se-“stealth” metode SYN scan.

#### Perbedaan Hasil dengan SYN Scan

Dari sisi hasil scanning, **output Connect scan biasanya hampir sama dengan SYN scan**. Keduanya tetap memberikan informasi utama seperti:

- port yang terbuka (`open`)
- port yang tertutup (`closed`)
- port yang difilter (`filtered`)

Perbedaannya terletak pada **cara kerja di balik layar**.

SYN scan bekerja dengan memanipulasi paket TCP secara langsung tanpa menyelesaikan koneksi, sedangkan Connect scan memanfaatkan **mekanisme koneksi TCP standar dari sistem operasi**.

Karena itu, dalam praktik penetration testing atau network reconnaissance, pemilihan metode scan biasanya disesuaikan dengan:

- hak akses yang tersedia
- tingkat stealth yang diinginkan
- lingkungan jaringan yang sedang dianalisis.

### f. ACK Scan (`-sA`) – Memetakan Aturan Firewall

Berbeda dengan jenis scanning sebelumnya, **ACK scan** tidak digunakan untuk mencari port yang terbuka. Tujuan utama teknik ini adalah **menganalisis bagaimana firewall memperlakukan paket yang masuk**, sehingga kita bisa memahami apakah suatu port difilter atau tidak.

Dengan kata lain, ACK scan lebih fokus pada **pemetaan aturan firewall (firewall rule mapping)** daripada mencari layanan yang berjalan pada port tertentu.

Teknik ini sangat berguna dalam proses **network reconnaissance**, terutama ketika kita ingin mengetahui apakah sebuah sistem memiliki firewall dan bagaimana firewall tersebut memfilter paket jaringan.

#### Konsep Dasar di Balik ACK Scan

Untuk memahami cara kerja ACK scan, kita perlu mengingat kembali bagaimana **koneksi TCP normal dimulai**.

Dalam komunikasi TCP, koneksi biasanya dimulai dengan paket **SYN** sebagai bagian dari proses **three-way handshake**. Namun pada ACK scan, Nmap **tidak memulai koneksi dengan SYN**.

Sebaliknya, Nmap langsung mengirim **paket ACK** ke port target.

Secara normal, paket ACK seharusnya muncul **setelah koneksi TCP sudah terbentuk**. Jika sebuah sistem menerima paket ACK yang tidak sesuai konteks koneksi, biasanya sistem akan merespons dengan paket **RST (reset)**.

Respons inilah yang dimanfaatkan oleh Nmap untuk menganalisis perilaku firewall.

#### Cara Kerja ACK Scan

Proses scanning dengan metode ini dapat dijelaskan secara sederhana sebagai berikut:

1. Nmap mengirim **paket TCP dengan flag `ACK`** ke port target.
2. Jika paket tersebut **sampai ke sistem target**, sistem biasanya akan membalas dengan paket **RST**.
3. Jika **tidak ada respons sama sekali**, kemungkinan paket tersebut **diblokir oleh firewall** sebelum mencapai target.

Dari respons tersebut, Nmap dapat menarik kesimpulan tentang kondisi firewall.

#### Interpretasi Hasil ACK Scan

Ada dua kemungkinan utama yang biasanya muncul dalam hasil ACK scan.

**Jika mendapat respons `RST`**

Jika server merespons dengan paket `RST`, Nmap akan menandai port tersebut sebagai **unfiltered**.

Artinya:

- Paket yang dikirim Nmap berhasil mencapai target
- Firewall **tidak memblokir paket tersebut**

Namun perlu diperhatikan bahwa **unfiltered tidak berarti port tersebut terbuka**. Status ini hanya menunjukkan bahwa firewall tidak memblokir paket ACK.

**Jika tidak ada respons**

Jika tidak ada respons sama sekali, Nmap akan menandai port tersebut sebagai **filtered**.

Artinya kemungkinan besar:

- Firewall memblokir paket tersebut
- Paket tidak pernah mencapai host target

Dengan cara ini, ACK scan dapat membantu kita memahami **bagaimana firewall memfilter trafik jaringan**.

#### Menjalankan ACK Scan dengan Nmap

Untuk menggunakan teknik ini di Nmap, kita dapat menggunakan flag `-sA`.

Contoh perintahnya:

```bash id="4p6ub7"
sudo nmap -sA 172.17.199.151
```

Penjelasan perintah:

- `sudo`
  Digunakan untuk menjalankan Nmap dengan hak akses **root**, karena metode ini melibatkan manipulasi paket jaringan secara langsung.

- `nmap`
  Program yang digunakan untuk melakukan scanning jaringan.

- `-sA`
  Mengaktifkan **ACK scan**.

- `172.17.199.151`
  Alamat IP target yang akan dipindai.

Ketika perintah ini dijalankan, Nmap akan mengirim paket ACK ke berbagai port dan kemudian menganalisis respons yang diterima dari target.

#### Contoh Output ACK Scan

Berikut contoh output yang mungkin muncul dari hasil scanning:

```
All 1000 scanned ports on 172.17.199.151 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
```

Mari kita pahami arti dari output tersebut.

Baris pertama menunjukkan bahwa **1000 port yang dipindai berada dalam status yang tidak ditampilkan secara individual**.

Baris kedua memberikan informasi lebih detail:

```
Not shown: 1000 unfiltered tcp ports (reset)
```

Artinya:

- Semua port yang dipindai merespons dengan **paket RST**
- Karena itu Nmap menandainya sebagai **unfiltered**

Hal ini menunjukkan bahwa **paket ACK berhasil mencapai target dan mendapatkan respons**.

Namun penting untuk dipahami bahwa status **unfiltered bukan berarti port tersebut terbuka untuk layanan**. Status ini hanya menunjukkan bahwa firewall tidak memblokir paket ACK yang dikirim.

Untuk mengetahui apakah port benar-benar terbuka atau tidak, biasanya kita tetap perlu menggunakan teknik scanning lain seperti:

- **SYN scan (`-sS`)**
- **Connect scan (`-sT`)**

Dengan demikian, ACK scan lebih berfungsi sebagai **alat untuk memahami perilaku firewall**, bukan sebagai metode utama untuk menemukan layanan yang aktif pada sistem target.

### g. FIN/Null Scan (`-sF`) – Menguji Respons Stack Tidak Biasa

Selain teknik scanning yang umum seperti SYN scan atau Connect scan, Nmap juga memiliki metode yang lebih “tidak biasa”, salah satunya adalah **FIN scan**. Teknik ini digunakan untuk **mengamati bagaimana sebuah sistem merespons paket TCP yang tidak normal**.

FIN scan bekerja dengan cara mengirim **paket TCP dengan flag `FIN`**, meskipun sebelumnya **tidak pernah ada koneksi TCP yang dibuka**. Situasi ini sebenarnya tidak lazim dalam komunikasi jaringan normal, sehingga respons yang diberikan oleh sistem target dapat memberikan petunjuk tentang kondisi port.

Teknik ini sering digunakan untuk **mendeteksi port yang mungkin tersembunyi atau untuk melewati aturan firewall tertentu**.

#### Apa Itu Flag FIN dalam TCP

Dalam protokol TCP, flag **FIN (Finish)** biasanya digunakan untuk **menutup koneksi yang sudah ada**. Ketika sebuah aplikasi selesai berkomunikasi dengan server, ia akan mengirim paket dengan flag FIN sebagai tanda bahwa koneksi akan diakhiri.

Contoh alur normal penutupan koneksi TCP:

```
Client → FIN → Server
Server → ACK → Client
Server → FIN → Client
Client → ACK → Server
```

Namun dalam **FIN scan**, paket FIN dikirim **tanpa adanya koneksi sebelumnya**. Dari sudut pandang server, paket ini terlihat aneh karena tiba-tiba ada permintaan penutupan koneksi yang tidak pernah dibuka.

Respons server terhadap situasi ini menjadi dasar analisis bagi Nmap.

#### Cara Kerja FIN Scan

Ketika FIN scan dijalankan, Nmap akan mengirim **paket TCP dengan flag FIN** ke berbagai port pada target. Setelah itu, Nmap akan memperhatikan bagaimana target merespons.

Berdasarkan standar TCP yang dijelaskan dalam **RFC (Request for Comments)**, respons yang diharapkan adalah sebagai berikut:

**Jika port tertutup**

Jika port pada target dalam kondisi **tertutup**, sistem biasanya akan merespons dengan paket **RST (reset)**. Paket RST menandakan bahwa tidak ada koneksi yang valid untuk ditutup.

Dalam kasus ini, Nmap akan menandai port tersebut sebagai **closed**.

**Jika port terbuka**

Jika port dalam keadaan **terbuka**, banyak implementasi TCP akan **mengabaikan paket FIN yang tidak sesuai konteks**. Artinya server tidak mengirim respons apa pun.

Karena tidak ada respons, Nmap biasanya menandai port tersebut sebagai **open|filtered**.

#### Mengapa Statusnya "open|filtered"

Status `open|filtered` berarti Nmap **tidak dapat memastikan secara pasti** apakah port tersebut benar-benar terbuka atau justru diblokir oleh firewall.

Hal ini terjadi karena kedua kondisi tersebut menghasilkan perilaku yang sama:

- Jika port **terbuka**, server bisa saja mengabaikan paket FIN.
- Jika port **diblokir firewall**, paket mungkin tidak pernah sampai ke server sehingga tidak ada respons.

Karena kedua kemungkinan ini menghasilkan hasil yang sama (tidak ada respons), Nmap menggabungkannya dalam status **open|filtered**.

#### Peran Firewall dalam FIN Scan

Beberapa firewall dirancang untuk **memblokir paket SYN yang mencurigakan**, tetapi tidak selalu memfilter paket TCP yang menggunakan flag tidak biasa seperti FIN.

Akibatnya, dalam beberapa situasi:

- Paket SYN mungkin diblokir
- Tetapi paket FIN bisa saja lolos

Karena alasan inilah FIN scan terkadang digunakan sebagai **teknik alternatif untuk melewati aturan firewall tertentu**.

Namun perlu diingat bahwa banyak firewall modern sudah cukup canggih untuk **mendeteksi dan memblokir paket TCP yang tidak normal**, sehingga teknik ini tidak selalu berhasil.

#### Menjalankan FIN Scan dengan Nmap

Untuk menjalankan FIN scan menggunakan Nmap, kita dapat menggunakan flag `-sF`.

Contoh perintahnya:

```bash id="x9zv2t"
sudo nmap -sF 172.17.199.151
```

Penjelasan perintah:

- `sudo`
  Digunakan untuk menjalankan Nmap dengan hak akses root, karena teknik ini memerlukan kemampuan untuk membuat paket TCP secara langsung.

- `nmap`
  Tool yang digunakan untuk melakukan scanning jaringan.

- `-sF`
  Mengaktifkan metode **FIN scan**.

- `172.17.199.151`
  Target host yang akan dipindai.

Ketika perintah ini dijalankan, Nmap akan mengirim paket FIN ke berbagai port pada target dan kemudian mengamati respons yang diterima.

#### Kemungkinan Hasil Scan

Dalam FIN scan, ada dua hasil utama yang biasanya muncul.

**Status `closed`**

Jika port merespons dengan paket **RST**, Nmap akan menandai port tersebut sebagai `closed`.

Ini berarti:

- Port dapat dijangkau
- Namun tidak ada layanan yang berjalan pada port tersebut.

**Status `open|filtered`**

Jika **tidak ada respons sama sekali**, Nmap akan menandai port tersebut sebagai `open|filtered`.

Artinya ada dua kemungkinan:

- Port benar-benar **terbuka**
- Paket tersebut **diblokir oleh firewall**

Untuk memastikan kondisi sebenarnya, biasanya diperlukan **teknik scanning tambahan**, seperti SYN scan atau metode lain yang dapat memberikan informasi lebih jelas.

Dengan pendekatan ini, FIN scan menjadi salah satu teknik yang berguna dalam **menganalisis perilaku stack TCP pada sistem target**, sekaligus membantu mengidentifikasi kemungkinan konfigurasi firewall atau port yang tidak terlihat dengan metode scanning biasa.

## 3. Hubungan Antar Konsep

Proses scanning jaringan dengan NMAP mengikuti alur yang logis dan bertahap:

```
[1] Ping Scan (-sn)
     └── Konfirmasi host aktif
          │
[2] Default / Port Scan
     └── Identifikasi port terbuka (top 1000 TCP)
          │
[3] Pilih Teknik TCP Scan yang Tepat:
     ├── SYN Scan (-sS)   → Cepat, stealth, butuh root
     ├── Connect Scan (-sT) → Full handshake, tidak butuh root
     ├── ACK Scan (-sA)   → Memetakan aturan firewall
     └── FIN Scan (-sF)   → Menguji respons stack tidak biasa
          │
[4] Analisis Hasil
     └── Tentukan layanan mana yang akan dieksplorasi lebih lanjut
```

**Kaitan dengan eksploitasi:**
Setelah mengetahui port dan layanan yang terbuka, seorang _penetration tester_ dapat:

- Memilih layanan yang diketahui memiliki kerentanan (misalnya: SSH, SMB, NetBIOS).
- Menyesuaikan serangan berdasarkan keahlian dan informasi kerentanan yang dimiliki.
- Menggunakan hasil scan sebagai dasar untuk tahap berikutnya: _service version detection_, _OS fingerprinting_, dan _scripting engine_.

Perbedaan utama antar teknik scan bukan pada **hasilnya** (yang sering sama), melainkan pada **cara kerja di balik layar** — seberapa "bising" scan tersebut, apakah mudah terdeteksi firewall/IDS, dan apakah membutuhkan hak akses tertentu.

---

## 4. Kesimpulan

Video ini membangun fondasi pemahaman tentang penggunaan NMAP sebagai alat reconnaissance jaringan. Berikut poin-poin kunci yang perlu diingat:

1. **Selalu gunakan NMAP sebagai root** untuk hasil yang paling lengkap dan akurat.
2. **Ping scan** (`-sn`) adalah langkah pertama untuk memastikan host aktif sebelum melakukan scanning lebih dalam.
3. **SYN scan** (`-sS`) adalah pilihan utama — cepat, semi-stealth, dan paling banyak digunakan.
4. **Connect scan** (`-sT`) berguna saat tidak memiliki hak akses root, namun lebih mudah terdeteksi.
5. **ACK scan** (`-sA`) tidak mendeteksi port terbuka, tetapi sangat berguna untuk memahami perilaku firewall.
6. **FIN scan** (`-sF`) digunakan untuk menguji respons tidak biasa dari sistem target.
7. Hasil scan digunakan untuk **menentukan layanan mana yang akan menjadi target eksploitasi** selanjutnya.
8. Di video berikutnya, materi akan berlanjut ke **UDP scan**, **deteksi versi layanan**, **fingerprinting OS**, dan **NMAP Scripting Engine (NSE)**.

---

> 🔐 **Pengingat Etika:** Gunakan NMAP hanya pada sistem yang Anda miliki atau telah mendapat izin eksplisit untuk diuji. Penggunaan tanpa izin adalah pelanggaran hukum.
