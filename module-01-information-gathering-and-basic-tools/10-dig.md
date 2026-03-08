# 📝 Catatan Belajar: Tool `dig` untuk DNS Lookup

## 1. Ringkasan Singkat

Video ini membahas penggunaan tool **`dig`** (Domain Information Groper) di Linux — sebuah tool DNS lookup yang fleksibel dan powerful. `dig` digunakan untuk melakukan query ke DNS server guna mendapatkan informasi tentang host address, mail server, name server, TXT record, reverse lookup, dan banyak lagi. Tool ini sangat berguna untuk DNS troubleshooting, reconnaissance, dan forensic inspection.

---

## 2. Konsep Utama

### a. Apa itu `dig`?

#### Pengertian `dig`

`dig` (Domain Information Groper) adalah tool command-line di Linux yang digunakan untuk melakukan query DNS secara detail. Tool ini sangat populer di kalangan system administrator, network engineer, dan security engineer karena mampu menampilkan informasi DNS secara lengkap dan terstruktur.

Jika dibandingkan dengan `nslookup`, `dig` dianggap lebih powerful dan fleksibel. Alasannya bukan hanya karena bisa melakukan query DNS biasa, tetapi juga karena memberikan akses ke detail teknis dari proses komunikasi DNS itu sendiri.

Secara sederhana, jika `nslookup` cocok untuk pengecekan cepat, maka `dig` cocok untuk analisis mendalam dan troubleshooting yang serius.

#### Kenapa `dig` Dianggap Lebih Powerful?

Ada beberapa alasan utama mengapa `dig` sering lebih disukai:

1. Menampilkan full DNS message details
   Output `dig` tidak hanya menampilkan jawaban (answer), tetapi juga seluruh struktur pesan DNS, termasuk:
   - Header
   - Question section
   - Answer section
   - Authority section
   - Additional section

   Artinya, kita bisa melihat bagaimana server DNS merespons query kita secara lengkap, bukan hanya hasil akhirnya saja.

2. Banyak opsi untuk mengontrol query
   `dig` memiliki berbagai parameter yang bisa digunakan untuk:
   - Menentukan jenis record (A, AAAA, MX, TXT, NS, dll.)
   - Memilih DNS server tertentu
   - Menggunakan TCP alih-alih UDP
   - Mengatur timeout dan retry
   - Menampilkan output yang lebih ringkas atau lebih detail

3. Cocok untuk troubleshooting dan analisis mendalam
   Karena output-nya lengkap, `dig` sangat berguna untuk:
   - Troubleshooting masalah DNS
   - Reconnaissance (misalnya dalam penetration testing)
   - Inspeksi forensik DNS
   - Mengecek propagasi DNS
   - Menganalisis authority dan delegation domain

#### Cara Kerja Dasar DNS dan Peran `dig`

Sebelum melihat contoh, penting memahami bahwa DNS bekerja di port 53 dan secara default menggunakan protokol UDP.

Alurnya seperti ini:

1. Client mengirimkan query ke DNS server melalui port 53 (biasanya via UDP).
2. Server DNS memproses permintaan tersebut.
3. Server mengirimkan respons yang berisi record yang diminta.

`dig` berfungsi sebagai client DNS yang memungkinkan kita melihat seluruh proses komunikasi tersebut secara transparan.

#### Contoh Penggunaan `dig`

Misalnya kita ingin mengetahui IP address dari domain example.com:

```
dig example.com
```

Output yang muncul akan cukup panjang. Secara umum akan terlihat seperti ini (disederhanakan untuk penjelasan):

```
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;example.com.        IN  A

;; ANSWER SECTION:
example.com.     300 IN  A   93.184.216.34
```

Mari kita pahami bagian-bagiannya.

Header
Bagian ini menunjukkan metadata dari query dan respons, seperti:

- Status (misalnya NOERROR berarti tidak ada error)
- Jumlah answer
- Flags (qr, rd, ra, dll.)

Question Section
Menunjukkan apa yang kita tanyakan ke server DNS.
Di sini terlihat bahwa kita meminta record tipe A (IPv4 address) untuk example.com.

Answer Section
Inilah jawaban dari server.
Dalam contoh di atas:

- example.com memiliki record A
- TTL-nya 300 detik
- IP address-nya adalah 93.184.216.34

Authority Section
Bagian ini biasanya muncul jika ada informasi mengenai authoritative name server untuk domain tersebut. Ini sangat berguna untuk memahami struktur delegasi DNS.

#### Menggunakan Tipe Record Tertentu

Kita juga bisa menentukan jenis record secara eksplisit.

Contoh untuk record MX:

```
dig example.com MX
```

Contoh untuk record NS:

```
dig example.com NS
```

Dengan cara ini, kita bisa mengeksplorasi konfigurasi DNS suatu domain secara menyeluruh.

#### Menggunakan DNS Server Tertentu

Kita juga bisa menentukan server DNS mana yang ingin digunakan:

```
dig @8.8.8.8 example.com
```

Di sini kita secara eksplisit meminta Google Public DNS (8.8.8.8) untuk menjawab query kita. Ini sangat berguna saat troubleshooting, misalnya untuk membandingkan hasil antara DNS lokal dan DNS publik.

#### Menggunakan TCP Alih-alih UDP

Secara default, DNS menggunakan UDP di port 53. Namun kita bisa memaksa `dig` menggunakan TCP:

```
dig example.com +tcp
```

Ini berguna jika:

- Respons DNS terlalu besar untuk UDP
- Sedang menganalisis firewall atau filtering
- Menguji perilaku DNS over TCP

#### Kapan Sebaiknya Menggunakan `dig`?

Gunakan `dig` ketika:

- Ingin melihat detail teknis dari respons DNS.
- Sedang melakukan troubleshooting DNS.
- Ingin memastikan authoritative server.
- Melakukan analisis keamanan atau penetration testing.
- Ingin memahami bagaimana DNS bekerja secara mendalam.

Secara keseluruhan, `dig` bukan sekadar tool untuk mencari IP address domain. Ia adalah alat inspeksi DNS yang memberikan visibilitas penuh terhadap komunikasi antara client dan server DNS di port 53, yang secara default menggunakan protokol UDP.

### b. Penggunaan Dasar `dig`

#### Sintaks Paling Sederhana

Penggunaan paling dasar dari `dig` sangat sederhana. Kita cukup menuliskan:

```bash
dig google.com
```

Perintah ini berarti kita meminta DNS server untuk mencari informasi tentang domain `google.com`.

Secara default, jika kita tidak menyebutkan tipe record, `dig` akan mencari record tipe **A**, yaitu record yang berisi alamat IPv4.

Artinya, perintah di atas secara implisit sama seperti:

```bash
dig google.com A
```

#### Apa yang Terjadi Saat Perintah Dijalankan?

Ketika perintah dijalankan, `dig` akan:

1. Mengirim query DNS ke server DNS yang terkonfigurasi di sistem kita (biasanya dari `/etc/resolv.conf` di Linux).
2. Menggunakan port 53.
3. Secara default memakai protokol UDP.
4. Menampilkan seluruh struktur respons DNS ke terminal.

Berbeda dengan tool sederhana yang hanya menampilkan IP address, `dig` memperlihatkan struktur lengkap respons DNS.

#### Struktur Output `dig`

Mari kita bahas bagian-bagian penting yang biasanya muncul dalam output.

Contoh output yang sudah disederhanakan:

```
;; QUESTION SECTION:
;google.com.    IN  A

;; ANSWER SECTION:
google.com.  300  IN  A  142.250.x.x

;; Query time: 7 msec
;; SERVER: 192.168.1.1#53
```

Sekarang kita bedah satu per satu.

#### Question Section

```
;; QUESTION SECTION:
;google.com.    IN  A
```

Bagian ini menunjukkan apa yang kita tanyakan ke DNS server.

Penjelasannya:

- `google.com.` → Domain yang diminta (titik di akhir menandakan fully qualified domain name).
- `IN` → Kelas DNS (Internet).
- `A` → Tipe record yang diminta (IPv4 address).

Artinya: kita meminta alamat IPv4 dari domain google.com.

#### Answer Section

```
;; ANSWER SECTION:
google.com.  300  IN  A  142.250.x.x
```

Inilah jawaban dari server DNS.

Penjelasannya:

- `google.com.` → Nama domain.
- `300` → TTL (Time To Live) dalam detik.
- `IN` → Kelas Internet.
- `A` → Tipe record.
- `142.250.x.x` → IP address hasil resolusi.

TTL 300 berarti hasil ini boleh di-cache selama 300 detik (5 menit) sebelum harus melakukan query ulang.

Jadi, inti dari bagian ini adalah: domain tersebut dipetakan ke IP address tertentu.

#### Query Time

```
;; Query time: 7 msec
```

Ini menunjukkan waktu yang dibutuhkan untuk mendapatkan respons dari server DNS.

Semakin kecil nilainya, semakin cepat respons server tersebut.
Dalam contoh ini, hanya membutuhkan 7 milidetik — cukup cepat karena kemungkinan server DNS berada di jaringan lokal atau dekat secara geografis.

Informasi ini sangat berguna saat troubleshooting performa DNS.

#### Server yang Digunakan

```
;; SERVER: 192.168.1.1#53
```

Bagian ini menunjukkan:

- `192.168.1.1` → Alamat DNS server yang menjawab query.
- `#53` → Port yang digunakan (default DNS adalah port 53).

Jika alamatnya 192.168.1.1, biasanya itu adalah router atau DNS lokal jaringan kita.
Kalau misalnya menggunakan 8.8.8.8, berarti kita menggunakan Google Public DNS.

Informasi ini penting untuk memastikan kita benar-benar menggunakan DNS server yang kita harapkan.

#### Header (Bagian yang Sering Terlewatkan)

Walaupun tidak ditampilkan di contoh sederhana tadi, biasanya di bagian atas output terdapat Header yang berisi:

- Versi `dig`
- Status query (misalnya NOERROR)
- Flag seperti:
  - `QR` → Query Response
  - `RD` → Recursion Desired
  - `RA` → Recursion Available

- Jumlah:
  - Query
  - Answer
  - Authority
  - Additional

Header ini sangat penting saat melakukan analisis mendalam, karena memberi tahu apakah query berhasil, apakah server mendukung recursion, dan apakah ada error seperti NXDOMAIN.

#### Kesimpulan Penggunaan Dasar

Perintah sederhana:

```bash
dig google.com
```

Sebenarnya memberikan banyak informasi, bukan hanya IP address.

Kita bisa melihat:

- Apa yang diminta (Question Section)
- Jawaban yang diberikan (Answer Section)
- TTL untuk caching
- Waktu respons (Query time)
- DNS server yang digunakan
- Detail teknis dari proses query

Dengan memahami struktur dasar ini, kita sudah memiliki fondasi yang kuat untuk menggunakan `dig` sebagai alat troubleshooting DNS secara lebih mendalam.

### c. Output Singkat dengan `+short`

#### Mengapa Perlu Output yang Lebih Ringkas?

Secara default, `dig` menampilkan informasi yang sangat lengkap: header, question section, answer section, query time, server, dan detail teknis lainnya.

Itu sangat berguna untuk troubleshooting atau analisis mendalam. Namun dalam banyak kasus sehari-hari, kita sebenarnya hanya ingin tahu satu hal sederhana:

“Apa IP address dari domain ini?”

Untuk kebutuhan seperti itu, output lengkap justru terasa terlalu panjang dan kurang praktis.

Di sinilah opsi `+short` menjadi sangat berguna.

#### Menggunakan `+short`

Untuk mendapatkan output yang langsung menampilkan jawaban tanpa informasi tambahan, kita bisa menggunakan:

```bash
dig +short google.com
```

Perintah ini tetap melakukan query DNS seperti biasa, tetapi `dig` hanya menampilkan bagian jawaban akhirnya saja.

#### Contoh Output

Jika dijalankan, hasilnya biasanya akan seperti ini:

```
142.250.x.x
```

Tidak ada header.
Tidak ada question section.
Tidak ada query time.
Tidak ada informasi server.

Hanya IP address saja.

Jika domain memiliki beberapa record A (misalnya load balancing), maka hasilnya bisa berupa beberapa baris IP address:

```
142.250.x.x
142.250.y.y
```

Artinya domain tersebut dipetakan ke lebih dari satu IP.

#### Apa yang Sebenarnya Terjadi di Balik Layar?

Walaupun output terlihat sederhana, proses di belakangnya tetap sama:

1. `dig` mengirim query DNS ke server.
2. Server membalas dengan struktur DNS lengkap.
3. `dig` memfilter hasilnya dan hanya menampilkan bagian Answer.

Jadi `+short` bukan berarti query-nya lebih sederhana — hanya output-nya saja yang disederhanakan.

#### Kapan Sebaiknya Menggunakan `+short`?

Gunakan `+short` ketika:

- Ingin cepat mengetahui IP address suatu domain.
- Membuat script atau automation (karena output-nya mudah diparsing).
- Tidak membutuhkan informasi teknis seperti TTL atau server DNS.
- Hanya ingin memverifikasi hasil resolusi DNS secara cepat.

Sebagai contoh, dalam shell script, jauh lebih mudah menangani output seperti ini:

```bash
IP=$(dig +short google.com)
echo $IP
```

Dibandingkan harus memfilter output panjang dari `dig` biasa.

#### Perbandingan Singkat

Tanpa `+short` → Output lengkap, cocok untuk analisis.
Dengan `+short` → Output minimalis, cocok untuk kebutuhan cepat dan scripting.

Dengan memahami opsi ini, kita bisa menyesuaikan `dig` sesuai kebutuhan: detail saat troubleshooting, ringkas saat butuh jawaban cepat.

### d. Reverse Lookup dengan Flag `-x`

#### Apa Itu Reverse Lookup?

Selama ini kita menggunakan DNS untuk menerjemahkan **nama domain menjadi IP address**.

Contohnya:

- `google.com` → `142.250.x.x`

Namun DNS juga mendukung proses kebalikannya, yaitu menerjemahkan **IP address menjadi nama domain**. Proses ini disebut **reverse lookup**.

Reverse lookup menggunakan record khusus yang disebut **PTR record** (Pointer Record).

#### Menggunakan Flag `-x` pada `dig`

Untuk melakukan reverse lookup dengan `dig`, kita bisa menggunakan flag `-x`:

```bash id="rv2kq1"
dig -x 142.250.22.238
```

Perintah ini memberi tahu `dig` bahwa kita ingin melakukan query PTR terhadap IP address tersebut.

Secara teknis, `dig` akan mengubah IP tersebut menjadi format khusus di bawah domain `in-addr.arpa`, lalu melakukan query PTR ke DNS.

#### Contoh Output dan Penjelasannya

Hasilnya biasanya akan menampilkan bagian seperti ini:

```id="jf9s8l"
;; QUESTION SECTION:
;238.22.250.142.in-addr.arpa.  IN  PTR

;; ANSWER SECTION:
238.22.250.142.in-addr.arpa.  300  IN  PTR  xxx.googleusercontent.com.
```

Mari kita pahami bagian pentingnya.

Question Section
Di sini terlihat bahwa `dig` sebenarnya tidak langsung menanyakan IP, tetapi:

```
238.22.250.142.in-addr.arpa
```

Itu adalah representasi terbalik dari IP `142.250.22.238`.

DNS memang menggunakan domain khusus `in-addr.arpa` untuk reverse lookup IPv4.

Answer Section
Bagian ini menunjukkan PTR record yang ditemukan.

Contohnya:

```
xxx.googleusercontent.com.
```

Artinya IP tersebut terdaftar memiliki nama domain tersebut sebagai reverse mapping-nya.

#### Kenapa Hasilnya Bukan `google.com`?

Untuk domain besar seperti Google, kita sering tidak mendapatkan `google.com` sebagai hasil reverse lookup.

Alasannya:

- Google memiliki banyak server (server pool).
- Satu domain bisa dipetakan ke banyak IP (load balancing).
- Satu IP biasanya merepresentasikan satu mesin/server tertentu.
- Reverse DNS sering mengarah ke nama host internal atau nama server tertentu, bukan ke domain utama.

Jadi ketika kita menjalankan:

```bash id="i8kz3m"
dig -x 142.250.22.238
```

IP tersebut kemungkinan besar adalah salah satu server di infrastruktur Google, bukan representasi langsung dari domain `google.com`.

Itulah sebabnya hasilnya bisa berupa nama seperti:

- `xxx.googleusercontent.com`
- atau nama host teknis lainnya

#### Kapan Reverse Lookup Digunakan?

Reverse lookup sering digunakan untuk:

- Analisis log server (misalnya melihat IP pengunjung lalu mencari hostname-nya).
- Investigasi keamanan.
- Troubleshooting jaringan.
- Verifikasi konfigurasi mail server (karena banyak mail server memeriksa PTR record).

Dalam konteks email server, misalnya, sangat umum dilakukan pengecekan apakah IP pengirim memiliki PTR record yang valid.

#### Kesimpulan

Dengan flag `-x`, `dig` dapat melakukan reverse lookup menggunakan PTR record.

```bash id="w4mtn0"
dig -x <ip-address>
```

Perintah ini mengubah IP address menjadi nama domain yang terkait, jika tersedia.

Namun perlu diingat, untuk layanan besar seperti Google, hasilnya biasanya menunjuk ke nama server spesifik dalam infrastruktur mereka, bukan langsung ke domain utama seperti `google.com`.

Memahami reverse lookup ini penting karena DNS bukan hanya sistem untuk “nama ke IP”, tetapi juga mendukung pemetaan dua arah yang sering digunakan dalam analisis jaringan dan keamanan.

### e. Query ke DNS Server Tertentu dengan `@`

#### Perilaku Default `dig`

Secara default, ketika kita menjalankan perintah seperti:

```bash
dig google.com
```

`dig` akan menggunakan DNS server yang sudah dikonfigurasi di sistem operasi.

Di Linux, konfigurasi ini biasanya berasal dari file `/etc/resolv.conf`. Dalam banyak kasus, DNS server yang digunakan adalah:

- Router lokal (misalnya `192.168.1.1`)
- DNS dari ISP
- Atau DNS yang sudah kita atur manual di sistem

Artinya, hasil query yang kita dapat sangat bergantung pada DNS server tersebut.

#### Menggunakan DNS Server Tertentu dengan `@`

Jika kita ingin menentukan DNS server secara eksplisit, kita bisa menggunakan simbol `@` diikuti alamat IP DNS server yang ingin digunakan.

Contoh menggunakan Google Public DNS:

```bash
dig google.com @8.8.8.8
```

Contoh menggunakan Cloudflare DNS:

```bash
dig google.com @1.1.1.1
```

Perintah ini berarti:

- Abaikan konfigurasi DNS sistem.
- Kirim query langsung ke server yang disebutkan setelah `@`.

Secara teknis, `dig` tetap melakukan query DNS ke port 53 (default) pada server tersebut, biasanya melalui UDP.

#### Apa yang Berbeda dari Hasilnya?

Struktur output tetap sama seperti biasa (header, question, answer, query time, server).

Namun, pada bagian akhir akan terlihat server mana yang menjawab:

```
;; SERVER: 8.8.8.8#53
```

atau

```
;; SERVER: 1.1.1.1#53
```

Ini memastikan bahwa respons benar-benar berasal dari DNS server yang kita tentukan.

#### Mengapa Perlu Mengganti DNS Server?

Ada beberapa alasan praktis mengapa fitur ini sangat penting.

1. Membandingkan hasil antar DNS server
   Kadang hasil resolusi bisa berbeda tergantung DNS server.
   Misalnya karena:
   - Perbedaan cache
   - Perbedaan propagasi DNS
   - Perbedaan kebijakan filtering

   Dengan `@`, kita bisa membandingkan:

   ```bash
   dig example.com @192.168.1.1
   dig example.com @8.8.8.8
   ```

   Jika hasilnya berbeda, berarti ada perbedaan di level DNS server.

2. Troubleshooting propagasi DNS
   Saat baru mengganti record domain, perubahan mungkin belum tersebar ke semua DNS server.
   Dengan mencoba beberapa server publik, kita bisa melihat apakah perubahan sudah sampai atau belum.

3. Menghindari cache lokal
   Router lokal sering menyimpan cache DNS. Kadang cache tersebut belum diperbarui.
   Dengan langsung query ke server publik seperti `8.8.8.8`, kita bisa memastikan mendapatkan data terbaru dari sisi mereka.

4. Menguji performa
   Kita juga bisa membandingkan `Query time` antara DNS lokal dan DNS publik untuk melihat mana yang lebih cepat.

#### Apakah DNS Publik Selalu Lebih Baik?

DNS server besar seperti:

- Google (`8.8.8.8`)
- Cloudflare (`1.1.1.1`)

Biasanya memiliki:

- Infrastruktur global
- Cache yang sangat luas
- Ketersediaan tinggi
- Respon cepat

Namun bukan berarti selalu lebih cepat dibanding router lokal. Jika router lokal memiliki cache yang masih valid, justru bisa lebih cepat karena tidak perlu keluar ke internet.

Jadi konteks jaringan sangat berpengaruh.

#### Kesimpulan

Dengan menggunakan simbol `@`, kita bisa menentukan DNS server secara spesifik:

```bash
dig domain.com @dns-server
```

Fitur ini sangat penting untuk:

- Membandingkan hasil resolusi
- Troubleshooting propagasi DNS
- Menghindari cache lokal
- Analisis performa dan debugging jaringan

Memahami penggunaan `@` membuat kita tidak lagi bergantung sepenuhnya pada konfigurasi DNS sistem, dan memberi kontrol penuh atas ke mana query DNS dikirim.

### f. Query Jenis Record Tertentu

#### Mengapa Perlu Menentukan Jenis Record?

Secara default, ketika kita menjalankan:

```bash
dig google.com
```

`dig` akan meminta record tipe **A**, yaitu alamat IPv4 dari domain tersebut.

Namun DNS sebenarnya menyimpan banyak jenis record yang berbeda, masing-masing memiliki fungsi spesifik. Karena itu, `dig` memungkinkan kita untuk secara eksplisit menentukan jenis record yang ingin kita minta.

Ini sangat penting ketika kita ingin menganalisis konfigurasi email, name server, verifikasi domain, atau struktur zona DNS.

Berikut adalah jenis-jenis record utama yang sering digunakan.

#### Record A (IPv4 Address)

Perintah:

```bash
dig google.com
```

atau

```bash
dig google.com A
```

Fungsi:
Menampilkan alamat IPv4 dari domain.

Di bagian **Answer Section**, kita akan melihat IP seperti:

```
google.com.  300  IN  A  142.250.x.x
```

Artinya domain tersebut dipetakan ke alamat IPv4 tertentu.

#### Record AAAA (IPv6 Address)

Perintah:

```bash
dig google.com AAAA
```

Fungsi:
Menampilkan alamat IPv6 dari domain.

Output di Answer Section akan berisi alamat dengan format IPv6, misalnya:

```
google.com.  300  IN  AAAA  2a00:1450:4001:...
```

Record ini penting di jaringan modern yang sudah mendukung IPv6.

#### Record MX (Mail Exchange)

Perintah:

```bash
dig google.com MX
```

Fungsi:
Menampilkan mail server yang bertanggung jawab menerima email untuk domain tersebut.

Di Answer Section, kita akan melihat sesuatu seperti:

```
google.com.  300  IN  MX  10 aspmx.l.google.com.
google.com.  300  IN  MX  20 alt1.aspmx.l.google.com.
```

Penjelasannya:

- Angka 10 dan 20 adalah **priority** (semakin kecil, semakin tinggi prioritasnya).
- `aspmx.l.google.com` adalah server email yang menangani domain tersebut.

Artinya, ketika seseorang mengirim email ke `@google.com`, server pengirim akan melihat MX record ini untuk mengetahui ke mana email harus dikirim.

#### Record NS (Name Server)

Perintah:

```bash
dig google.com NS
```

Fungsi:
Menampilkan name server yang berwenang (authoritative) untuk domain tersebut.

Contoh output:

```
google.com.  300  IN  NS  ns1.google.com.
google.com.  300  IN  NS  ns2.google.com.
```

Artinya domain tersebut dikelola oleh name server tersebut.

Biasanya ada beberapa NS untuk redundansi.

Jika kita melakukan query lebih lanjut terhadap masing-masing NS, kita bisa melihat alamat IPv4 dan IPv6 mereka.

Record NS sangat penting dalam memahami struktur delegasi DNS.

#### Record TXT (Text Record)

Perintah:

```bash
dig google.com TXT
```

Fungsi:
Menampilkan data teks yang disimpan dalam DNS.

TXT record sering digunakan untuk:

- Verifikasi domain (misalnya untuk layanan cloud atau SSL).
- SPF (Sender Policy Framework) untuk email.
- DKIM dan kebijakan keamanan email lainnya.
- Informasi kepemilikan domain.

Contoh output bisa berupa:

```
google.com.  300  IN  TXT  "v=spf1 include:_spf.google.com ~all"
```

Itu adalah konfigurasi SPF yang menentukan server mana yang diizinkan mengirim email atas nama domain tersebut.

TXT record sering digunakan dalam praktik DevOps dan administrasi email.

#### Record SOA (Start of Authority)

Perintah:

```bash
dig google.com SOA
```

Fungsi:
Menampilkan informasi otoritatif utama dari zona domain.

Contoh output:

```
google.com.  300  IN  SOA  ns1.google.com. dns-admin.google.com. 1234567890 900 900 1800 60
```

SOA record berisi:

- Primary name server
- Email administrator (dalam format khusus)
- Serial number zona
- Refresh, retry, expire, dan minimum TTL

Record ini sangat penting untuk memahami bagaimana zona DNS dikelola dan direplikasi antar server.

#### Record PTR (Reverse Lookup)

Perintah:

```bash
dig -x 142.250.22.238
```

Fungsi:
Melakukan reverse lookup (IP → Domain).

Jika tersedia, Answer Section akan menampilkan nama domain atau hostname yang terkait dengan IP tersebut.

PTR record sangat umum digunakan dalam:

- Validasi email server
- Analisis keamanan
- Investigasi jaringan

#### Kesimpulan

Dengan menentukan jenis record secara spesifik, kita bisa menggunakan `dig` bukan hanya untuk mengetahui IP address, tetapi juga untuk:

- Menganalisis konfigurasi email (MX, TXT, SPF).
- Mengetahui name server yang berwenang (NS).
- Memahami struktur zona DNS (SOA).
- Mendukung IPv6 (AAAA).
- Melakukan reverse lookup (PTR).

Kemampuan memilih record ini menjadikan `dig` sebagai alat yang sangat fleksibel dan powerful dalam analisis serta troubleshooting DNS.

### g. SOA Record

#### Apa Itu SOA (Start of Authority)?

SOA (Start of Authority) adalah salah satu record paling penting dalam sebuah zona DNS.

Jika kita membayangkan DNS seperti sebuah sistem administrasi domain, maka SOA record adalah “dokumen resmi” yang menjelaskan siapa yang bertanggung jawab atas domain tersebut dan bagaimana zona DNS tersebut dikelola.

Setiap zona DNS **wajib memiliki satu SOA record**. Tanpa SOA, zona tersebut tidak dianggap valid oleh sistem DNS.

#### Informasi Apa Saja yang Disimpan di SOA?

SOA record menyimpan beberapa informasi penting, di antaranya:

- Primary name server (server DNS utama untuk zona tersebut).
- Alamat email administrator domain.
- Serial number zona.
- Parameter replikasi seperti refresh, retry, expire, dan minimum TTL.

Dari semua informasi itu, yang paling sering diperhatikan adalah:

1. Server DNS primer
   Menunjukkan server utama yang menjadi sumber kebenaran (authoritative source) untuk zona tersebut.

2. Kontak administrator
   Biasanya ditampilkan dalam format khusus DNS.

#### Cara Melihat SOA Record dengan `dig`

Untuk meminta SOA record, kita bisa menjalankan:

```bash id="j4k8sp"
dig google.com SOA
```

Perintah ini memberi tahu `dig` untuk secara spesifik meminta record tipe SOA dari domain tersebut.

#### Contoh Output dan Penjelasannya

Di bagian **Answer Section**, kita akan melihat sesuatu seperti:

```id="x9pt2n"
google.com.  300  IN  SOA  ns1.google.com. dns-admin.google.com. 1234567890 900 900 1800 60
```

Mari kita uraikan bagian-bagiannya.

- `ns1.google.com.`
  Ini adalah primary name server untuk zona `google.com`.

- `dns-admin.google.com.`
  Ini adalah alamat email administrator dalam format DNS.

  Format aslinya sebenarnya:

  ```
  dns-admin@google.com
  ```

  Dalam DNS, tanda `@` diganti dengan titik pertama. Jadi:

  ```
  user@domain.com
  ```

  ditulis sebagai:

  ```
  user.domain.com.
  ```

- `1234567890`
  Ini adalah **serial number** zona.
  Serial number digunakan untuk menentukan apakah ada perubahan pada zona DNS.
  Secondary name server akan membandingkan serial ini untuk mengetahui apakah perlu melakukan sinkronisasi ulang.

- `900 900 1800 60`
  Ini adalah parameter teknis yang mengatur:
  - Refresh interval
  - Retry interval
  - Expire time
  - Minimum TTL

Parameter ini mengontrol bagaimana secondary DNS server memperbarui data dari primary server.

#### Kenapa SOA Record Penting?

SOA record sangat penting dalam beberapa situasi:

- Troubleshooting masalah propagasi DNS.
- Mengecek apakah perubahan zona sudah diupdate (melihat serial number).
- Memastikan siapa server utama yang authoritative.
- Analisis struktur zona DNS.

Dalam skenario profesional, saat terjadi masalah DNS, mengecek SOA adalah salah satu langkah awal untuk memastikan zona dikelola dengan benar.

#### Kesimpulan

SOA (Start of Authority) record adalah pusat informasi administratif sebuah zona DNS.

Dengan menjalankan:

```bash id="z6v1mu"
dig domain.com SOA
```

Kita bisa mengetahui:

- Server DNS primer.
- Kontak administrator domain.
- Serial number zona.
- Parameter replikasi DNS.

Memahami SOA record membantu kita melihat sisi “manajemen internal” dari sebuah domain, bukan hanya sekadar pemetaan nama ke IP address.

### h. Format Output Tambahan

Selain menentukan jenis record atau DNS server tertentu, `dig` juga menyediakan berbagai opsi tambahan untuk mengatur bagaimana query dilakukan dan bagaimana hasilnya ditampilkan. Opsi-opsi ini sangat membantu saat melakukan troubleshooting atau analisis DNS secara mendalam.

Berikut beberapa flag penting yang sering digunakan.

#### `+answer` — Menampilkan Hanya Bagian Jawaban

Secara default, `dig` menampilkan banyak informasi seperti header, question section, answer section, authority, additional, query time, dan server.

Jika kita hanya ingin melihat bagian **Answer Section** saja, kita bisa menggunakan kombinasi:

```bash
dig google.com +all +answer
```

Penjelasannya:

- `+all` → Mengaktifkan semua bagian output.
- `+answer` → Memfilter agar hanya bagian Answer yang ditampilkan.

Hasilnya akan langsung menampilkan record yang menjadi jawaban, misalnya:

```
google.com.  300  IN  A  142.250.x.x
```

Tanpa header, tanpa query time, tanpa informasi server.

Flag ini sangat berguna jika:

- Kita ingin output yang lebih bersih.
- Sedang membuat script.
- Tidak membutuhkan informasi teknis tambahan.

Secara praktis, ini seperti versi “semi-ringkas” dari `dig`, tetapi tetap mempertahankan format record DNS.

#### `+tcp` — Memaksa Query Menggunakan TCP

Secara default, DNS menggunakan protokol **UDP** di port 53 karena lebih ringan dan cepat. Namun dalam beberapa kondisi, DNS bisa menggunakan **TCP**, misalnya:

- Saat ukuran respons terlalu besar untuk UDP.
- Saat terjadi zone transfer.
- Saat troubleshooting firewall atau filtering jaringan.

Untuk memaksa `dig` menggunakan TCP, kita bisa menjalankan:

```bash
dig +tcp google.com
```

Dengan perintah ini, `dig` akan mengirim query DNS melalui koneksi TCP ke port 53.

Perlu diperhatikan:

Jika server tidak mendukung TCP untuk jenis query tersebut, atau ada firewall yang memblokir koneksi TCP ke port 53, maka bisa terjadi timeout.

Flag `+tcp` sangat berguna dalam analisis jaringan, terutama ketika ingin memastikan apakah suatu firewall hanya mengizinkan UDP tetapi memblokir TCP.

#### `+trace` — Melakukan Tracing Resolusi DNS

Salah satu fitur paling menarik dari `dig` adalah `+trace`.

Perintahnya:

```bash
dig +trace google.com
```

Berbeda dengan query biasa yang langsung meminta jawaban ke DNS resolver, `+trace` akan melakukan proses resolusi DNS secara bertahap mulai dari:

1. Root server.
2. TLD server (misalnya `.com`).
3. Authoritative name server untuk domain tersebut.
4. Hingga mendapatkan jawaban akhir.

Output-nya akan memperlihatkan setiap langkah resolusi tersebut.

Contohnya, kita akan melihat daftar root server terlebih dahulu, lalu referensi ke name server `.com`, kemudian referensi ke name server `google.com`, hingga akhirnya mendapatkan IP address.

Fitur ini sangat berguna bagi:

- Network administrator.
- Engineer yang ingin memahami alur resolusi DNS secara detail.
- Troubleshooting masalah delegasi atau propagasi DNS.

Dengan `+trace`, kita bisa melihat dengan jelas di mana proses resolusi mungkin mengalami masalah — apakah di level root, TLD, atau authoritative server.

#### Kesimpulan

Beberapa flag tambahan pada `dig` memberikan kontrol lebih terhadap cara query dilakukan dan bagaimana hasil ditampilkan:

- `+answer` → Fokus hanya pada bagian jawaban.
- `+tcp` → Memaksa penggunaan protokol TCP.
- `+trace` → Menelusuri proses resolusi DNS dari root hingga server akhir.

Memahami opsi-opsi ini membuat kita tidak hanya menggunakan `dig` sebagai alat pencari IP, tetapi sebagai alat analisis DNS yang lengkap dan fleksibel untuk berbagai kebutuhan teknis.

## 3. Hubungan Antar Konsep

Semua kemampuan `dig` di atas saling melengkapi dalam memahami cara kerja DNS:

1. **Query dasar** (`dig domain`) → memperkenalkan struktur output DNS (header, question, answer, authority).
2. **Jenis record** (A, MX, NS, TXT, PTR, SOA) → menunjukkan bahwa DNS bukan hanya soal mengubah domain ke IP, tapi menyimpan berbagai jenis informasi tentang suatu domain.
3. **Custom DNS server** (`@8.8.8.8`) → mengajarkan bahwa ada banyak DNS server dengan database berbeda, dan kita bisa memilih mana yang digunakan.
4. **Format output** (`+short`, `+answer`, `+trace`) → memberikan fleksibilitas sesuai kebutuhan: ringkas untuk jawaban cepat, verbose untuk debugging mendalam.
5. **Reverse lookup** (`-x`) → melengkapi kemampuan "forward lookup" (domain → IP) dengan "backward lookup" (IP → domain).

Secara keseluruhan, `dig` memberikan gambaran penuh tentang **bagaimana DNS digunakan dan bisa disalahgunakan** — sebuah pemahaman penting untuk keamanan siber dan administrasi jaringan.

---

## 4. Kesimpulan

`dig` adalah tool DNS lookup yang sangat powerful dan fleksibel di Linux. Kemampuan utamanya meliputi:

- Query berbagai jenis DNS record: **A, AAAA, MX, NS, TXT, SOA, PTR**.
- Menggunakan DNS server kustom dengan `@`.
- Menampilkan output singkat (`+short`) atau detail penuh (`+trace`).
- Melakukan reverse lookup dengan flag `-x`.

Untuk keperluan **ujian dan praktik keamanan jaringan**, memahami `dig` sangat penting karena mencakup semua tipe record DNS utama yang sering muncul — mulai dari resolusi IP, mail server, name server, hingga informasi teks dan zona DNS. Tool ini adalah senjata utama untuk DNS troubleshooting dan reconnaissance.

---

_Catatan dibuat berdasarkan video tutorial penggunaan tool `dig` di Linux._
