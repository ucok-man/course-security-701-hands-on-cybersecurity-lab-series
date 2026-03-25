# 📝 NMAP - Bagian 3: OS Detection, Aggressive Scan, dan NSE (Nmap Scripting Engine)

## 1. Ringkasan Singkat

Video ini merupakan bagian ketiga dari seri tutorial NMAP. Setelah sebelumnya membahas dasar-dasar NMAP, TCP scan, ping scan, UDP scan, dan version scan, video ini melanjutkan dengan topik yang lebih lanjut, yaitu:

- **OS Detection** (deteksi sistem operasi target)
- **Aggressive Scan** (scan agresif yang menggabungkan banyak fitur sekaligus)
- **Nmap Scripting Engine (NSE)** dengan script `default` dan `vulners`
- **Alur kerja rekomendasi** dalam melakukan scanning menggunakan NMAP

---

## 2. Konsep Utama

### a. OS Detection (`-O`)

OS Detection adalah salah satu fitur di Nmap yang digunakan untuk mengidentifikasi sistem operasi yang berjalan pada target. Cara kerjanya menggunakan teknik yang disebut **fingerprinting**, yaitu dengan menganalisis bagaimana target merespons berbagai jenis paket jaringan.

Alih-alih bertanya langsung “kamu pakai OS apa?”, Nmap mengirimkan serangkaian request khusus, lalu membandingkan pola responsnya dengan database fingerprint yang sudah dimiliki. Dari situ, Nmap bisa memperkirakan OS yang digunakan.

#### Cara Menjalankan OS Detection

Untuk menggunakan fitur ini, kamu bisa menjalankan perintah berikut:

```bash
sudo nmap -O <IP_ADDRESS>
```

Perlu diperhatikan bahwa opsi `-O` biasanya membutuhkan hak akses root (sudo), karena Nmap perlu mengirim paket-paket tingkat rendah untuk mendapatkan fingerprint yang akurat.

#### Informasi yang Dihasilkan

Saat OS Detection dijalankan, ada beberapa informasi penting yang akan ditampilkan:

##### Perkiraan Versi OS

Nmap tidak selalu bisa memberikan jawaban yang 100% pasti. Sebagai gantinya, dia memberikan **perkiraan** dalam bentuk beberapa kemungkinan OS beserta tingkat kepercayaannya.

Misalnya:

```
Aggressive OS guesses: Linux 3.x (95%), Linux 4.x (90%)
```

Artinya:

- Kemungkinan terbesar target menggunakan Linux kernel 3.x dengan tingkat kepercayaan 95%
- Alternatif lain adalah Linux 4.x dengan confidence 90%

Semakin tinggi persentasenya, semakin besar kemungkinan hasil tersebut akurat. Tapi tetap harus diingat, ini bukan hasil absolut.

##### Network Distance

Selain OS, Nmap juga memberikan informasi tentang **jarak jaringan** antara mesin kamu dan target, yang disebut sebagai _Network Distance_.

Contohnya:

```
Network Distance: 1 hop
```

#### Penjelasan Network Distance

Network Distance dihitung berdasarkan jumlah **hop**, yaitu berapa kali paket harus melewati perangkat jaringan (biasanya router) sebelum sampai ke target.

- Jika nilainya **1 hop**, berarti:
  - Paket dari komputer kamu hanya melewati satu router sebelum sampai ke target
  - Ini biasanya menunjukkan target berada di jaringan yang sangat dekat (misalnya satu LAN atau lab yang sama)

Contoh alurnya:

- Komputer kamu (scanner) mengirim request
- Request masuk ke **default gateway** (router)
- Router meneruskan ke target (misalnya server atau VM seperti OWASP Broken Web App)

Total hanya **1 kali lompatan (hop)**.

Semakin besar angka hop:

- Semakin banyak router yang dilewati
- Semakin jauh posisi target secara jaringan

#### Contoh Output Lengkap

Berikut contoh sederhana hasil dari OS Detection:

```
Aggressive OS guesses: Linux 3.x (95%), Linux 4.x (90%)
Network Distance: 1 hop
```

Dari output ini, kamu bisa menyimpulkan:

- Target kemungkinan besar adalah mesin Linux
- Versinya ada di kisaran kernel 3.x atau 4.x
- Lokasinya cukup dekat secara jaringan (hanya 1 hop)

Dengan memahami hasil ini, kamu bisa melanjutkan proses reconnaissance dengan lebih terarah, misalnya menyesuaikan eksploit atau teknik lanjutan berdasarkan jenis OS yang terdeteksi.

### b. Aggressive Scan (`-A`)

Aggressive Scan adalah mode pemindaian di Nmap yang bisa dibilang sebagai paket lengkap dalam satu perintah. Dengan hanya menambahkan flag `-A`, kamu langsung mengaktifkan beberapa fitur penting sekaligus tanpa perlu menjalankannya satu per satu.

Mode ini sangat cocok digunakan saat kamu ingin mendapatkan gambaran menyeluruh tentang target, mulai dari sistem operasi, layanan yang berjalan, hingga jalur jaringan yang dilewati.

#### Cara Menjalankan Aggressive Scan

Untuk menggunakan mode ini, jalankan perintah berikut:

```bash
sudo nmap -A <IP_ADDRESS>
```

Contoh penggunaan langsung:

```bash
sudo nmap -A 192.168.1.10
```

Karena scan ini cukup “dalam” dan kompleks, biasanya membutuhkan akses root (`sudo`) agar hasilnya maksimal.

#### Apa Saja yang Dilakukan oleh `-A`?

Saat kamu menjalankan `-A`, sebenarnya Nmap sedang mengaktifkan beberapa fitur sekaligus. Berikut penjelasan masing-masing fitur agar kamu benar-benar paham apa yang terjadi di balik satu command ini.

##### OS Detection

Fitur ini mencoba menebak sistem operasi target menggunakan teknik fingerprinting. Nmap akan menganalisis respons dari target, lalu mencocokkannya dengan database OS yang sudah ada.

Hasilnya biasanya berupa perkiraan, misalnya:

- Linux kernel versi tertentu
- Windows dengan versi tertentu

Ini membantu kamu memahami environment target sebelum lanjut ke tahap berikutnya.

##### Version Detection (`-sV`)

Fitur ini fokus pada **service yang berjalan di port terbuka**.

Misalnya:

- Port 80 → HTTP (web server)
- Port 22 → SSH

Dengan `-sV`, Nmap tidak hanya bilang “ini HTTP”, tapi juga mencoba mengidentifikasi versinya, seperti:

- Apache 2.4.41
- OpenSSH 7.6

Informasi versi ini sangat penting karena sering digunakan untuk mencari celah (vulnerability) yang spesifik terhadap versi tertentu.

##### Script Scan (`-sC`)

Nmap memiliki fitur bernama **NSE (Nmap Scripting Engine)**, yaitu kumpulan script yang bisa digunakan untuk menggali informasi lebih dalam.

Dengan `-sC`, Nmap akan menjalankan script default, misalnya:

- Mengambil banner service
- Mengecek konfigurasi tertentu
- Mengidentifikasi potensi vulnerability ringan

Jadi, bukan cuma scan port, tapi juga mulai “ngobrol” lebih dalam dengan service yang ditemukan.

##### Traceroute

Fitur ini digunakan untuk melacak jalur yang dilalui paket dari komputer kamu ke target.

Hasilnya biasanya berupa daftar hop (router) yang dilewati. Ini berguna untuk:

- Memahami topologi jaringan
- Mengetahui seberapa jauh target dari sisi jaringan

#### Karakteristik Output

Karena menggabungkan banyak fitur, hasil dari `-A` biasanya sangat detail dan panjang. Dalam satu kali scan, kamu bisa mendapatkan:

- Daftar port terbuka
- Service dan versinya
- Perkiraan sistem operasi
- Informasi tambahan dari script
- Jalur jaringan (traceroute)

Ini membuat Aggressive Scan sangat powerful, tapi juga:

- Lebih lambat dibanding scan biasa
- Lebih “noisy” (lebih mudah terdeteksi oleh sistem keamanan)

#### Fitur Menarik: Kontribusi ke Database Nmap

Ada satu hal menarik dari Nmap.

Jika Nmap menemukan service yang merespons, tapi tidak dikenali oleh database fingerprint yang ada, biasanya akan muncul pesan yang meminta kamu untuk mengirimkan fingerprint tersebut ke tim Nmap.

Artinya:

- Nmap berkembang dari kontribusi komunitas
- Setiap pengguna bisa membantu meningkatkan akurasi deteksi di masa depan

Konsep ini sering disebut sebagai **community-driven improvement**, di mana tool menjadi semakin pintar karena digunakan dan diperbaiki bersama.

#### Kesimpulan Singkat

Aggressive Scan (`-A`) adalah pilihan tepat ketika kamu ingin:

- Menghemat waktu (tidak perlu menjalankan banyak command)
- Mendapatkan informasi lengkap dalam satu kali scan

Namun, gunakan dengan bijak, terutama di lingkungan yang sensitif, karena scan ini cukup intens dan mudah terdeteksi.

### c. Nmap Scripting Engine (NSE)

Nmap Scripting Engine (NSE) adalah salah satu fitur paling powerful di Nmap karena mampu **memperluas fungsi Nmap jauh lebih dari sekadar scanning port**. Dengan NSE, Nmap bisa menjalankan berbagai script otomatis untuk mengumpulkan informasi tambahan, bahkan sampai melakukan pengujian kerentanan.

Secara sederhana, kalau sebelumnya Nmap hanya “melihat” port dan service, dengan NSE Nmap bisa mulai **berinteraksi lebih dalam dengan target**.

#### Kemampuan Utama NSE

Beberapa hal yang bisa dilakukan dengan NSE antara lain:

- Enumerasi konten web (misalnya mencari endpoint atau file tertentu)
- Pemeriksaan sertifikat SSL
- Mengambil informasi detail dari service
- Melakukan pengujian kerentanan tertentu

Semua ini dilakukan menggunakan script yang sudah disediakan oleh Nmap, atau bahkan bisa kamu kembangkan sendiri.

#### c.1. Script `default`

Script `default` adalah kumpulan script bawaan Nmap yang dianggap **aman untuk proses discovery awal**.

Untuk menjalankannya, kamu bisa gunakan perintah:

```bash id="6p7lqf"
sudo nmap --script=default <IP_ADDRESS>
```

Atau versi singkatnya:

```bash id="xgq9ny"
sudo nmap -sC <IP_ADDRESS>
```

Kedua perintah tersebut sebenarnya sama, karena `-sC` adalah shortcut dari `--script=default`.

##### Kapan Menggunakan Script Default?

Script ini cocok digunakan saat:

- Kamu baru mulai eksplorasi target
- Ingin mendapatkan informasi tambahan tanpa risiko besar
- Tidak ingin terlalu “agresif” terhadap sistem target

Script default biasanya melakukan hal-hal seperti:

- Mengambil banner dari service
- Mengecek konfigurasi umum
- Mengidentifikasi informasi dasar yang berguna

Karena sifatnya aman, script ini sering menjadi langkah awal sebelum masuk ke tahap yang lebih dalam.

#### c.2. Script `vulners` (Vulnerability Scan)

Berbeda dengan script default, `vulners` digunakan khusus untuk **mendeteksi potensi kerentanan (vulnerability)** pada service yang ditemukan.

Perintahnya:

```bash id="nmp6mq"
sudo nmap --script=vulners <IP_ADDRESS>
```

##### Karakteristik Scan `vulners`

- Prosesnya bisa cukup lama (misalnya hingga beberapa menit, tergantung target)
- Output yang dihasilkan sangat detail
- Fokus pada menemukan celah keamanan berdasarkan versi service

##### Cara Kerja Script `vulners`

Agar kamu lebih paham, mari kita lihat alur kerjanya secara bertahap:

1. **Deteksi Service dan Versi**
   Nmap terlebih dahulu mengidentifikasi service yang berjalan di port tertentu.
   Contoh:
   - Port 22 → OpenSSH versi 5.3

2. **Ekstraksi CPE (Common Platform Enumeration)**
   Dari informasi versi tersebut, Nmap akan menghasilkan CPE.
   CPE adalah format standar untuk merepresentasikan software dan versinya.

3. **Query ke Database Vulners**
   CPE tersebut digunakan untuk mencari data ke database online di vulners.com.

4. **Pencocokan dengan CVE**
   Database akan mengembalikan daftar CVE (Common Vulnerabilities and Exposures) yang relevan dengan versi tersebut.

5. **Menampilkan Hasil ke User**
   Nmap kemudian menampilkan:
   - Daftar vulnerability
   - Skor CVSS
   - Link referensi exploit atau detail vulnerability

Dengan proses ini, kamu bisa langsung tahu apakah service yang berjalan memiliki celah keamanan yang sudah dikenal.

##### Memahami Skor CVSS

Setiap vulnerability biasanya memiliki skor CVSS yang menunjukkan tingkat keparahannya.

Berikut interpretasinya:

- **0.0 – 3.9** → Low (rendah)
- **4.0 – 6.9** → Medium (sedang)
- **7.0 – 8.9** → High (tinggi), contoh: 7.8 atau 8.1
- **9.0 – 10.0** → Critical (kritis), contoh: 9.8

Semakin tinggi skor, semakin serius dampaknya dan biasanya semakin prioritas untuk diperbaiki.

##### Catatan Penting

Walaupun sangat powerful, penggunaan script seperti `vulners` harus dilakukan dengan hati-hati.

Scan jenis ini:

- Lebih agresif
- Bisa dianggap sebagai aktivitas mencurigakan
- Bahkan bisa melanggar hukum jika dilakukan tanpa izin

Karena itu, pastikan kamu hanya menjalankan scan ini pada:

- Mesin milik sendiri
- Lab testing
- Target yang memang sudah memberikan izin eksplisit

Dengan memahami NSE, kamu tidak hanya sekadar scanning, tapi sudah mulai masuk ke tahap **analisis dan security assessment** yang lebih dalam.

### d. Alur Kerja Rekomendasi NMAP (NMAP Scan Progression)

Bagian ini menjelaskan alur kerja yang direkomendasikan saat menggunakan Nmap, terutama agar proses scanning dilakukan secara **terstruktur, efisien, dan tidak langsung terlalu agresif**.

Pendekatan ini penting, karena dalam praktik nyata kamu tidak langsung melakukan scan berat. Sebaliknya, kamu mulai dari yang ringan, lalu secara bertahap meningkatkan kedalaman analisis berdasarkan informasi yang ditemukan.

Berikut adalah tahapan yang umum digunakan.

#### Langkah 1: Quick Discovery

Tujuan utama di tahap ini adalah **mengetahui host mana yang aktif** dalam suatu jaringan.

Perintah yang digunakan:

```bash id="sxtzqk"
nmap -sn <network_range>
```

Ini disebut sebagai **ping scan**.

Apa yang dilakukan:

- Mengecek apakah host hidup (up) atau tidak
- Tidak melakukan port scanning

Contoh:

- Jika kamu scan `192.168.1.0/24`, Nmap akan memberi tahu IP mana saja yang aktif di jaringan tersebut

Tahap ini penting karena:

- Kamu tidak membuang waktu scan ke host yang tidak aktif
- Lebih cepat dan ringan

#### Langkah 2: Full TCP Service Detection

Setelah tahu host mana yang aktif, langkah berikutnya adalah melakukan scan lebih dalam ke target tertentu.

Perintahnya:

```bash id="u3g2d6"
nmap -sV -p- <IP_ADDRESS>
```

Penjelasan:

- `-p-` → scan **semua port TCP** (1–65535)
- `-sV` → mendeteksi versi service yang berjalan

Apa yang kamu dapatkan:

- Daftar port terbuka
- Service yang berjalan di port tersebut
- Versi dari service tersebut

Contoh hasil:

- Port 22 → OpenSSH 7.6
- Port 80 → Apache 2.4.41

Tahap ini sangat krusial karena:

- Memberikan gambaran lengkap permukaan serangan (attack surface)
- Menjadi dasar untuk langkah berikutnya seperti enumerasi dan vulnerability scanning

#### Langkah 3: HTTP Enumeration (jika HTTP ditemukan)

Jika dari hasil sebelumnya ditemukan:

- Port 80 (HTTP) atau
- Port 443 (HTTPS)

Maka kamu bisa lanjut ke tahap **enumerasi web**.

Perintahnya:

```bash id="m0c9cz"
nmap --script=http-enum <IP_ADDRESS>
```

Apa yang dilakukan:

- Mencari direktori atau endpoint umum
- Mengidentifikasi resource yang tersedia di web server

Contoh:

- `/admin`
- `/login`
- `/backup`

Tahap ini membantu kamu memahami:

- Struktur aplikasi web
- Area yang berpotensi sensitif

Tidak semua target perlu langkah ini, hanya jika memang ada layanan HTTP/HTTPS.

#### Langkah 4: Vulnerability Check

Setelah mengetahui service dan versinya, kamu bisa mulai mencari apakah ada **kerentanan yang sudah diketahui**.

Perintahnya:

```bash id="1q5f4y"
nmap --script=vulners <IP_ADDRESS>
```

Apa yang dilakukan:

- Mencocokkan versi service dengan database vulnerability
- Menampilkan daftar CVE yang relevan
- Memberikan skor CVSS dan referensi exploit

Hasilnya biasanya:

- Sangat detail
- Membutuhkan waktu lebih lama

Catatan penting:

- Scan ini lebih agresif
- Harus dilakukan hanya pada target yang kamu punya izin

#### Prinsip Utama dalam Nmap Scan

Selain urutan langkah, ada beberapa prinsip penting yang perlu kamu pegang saat menggunakan Nmap.

##### Begin Light

Mulai dari scan yang ringan terlebih dahulu.

Tujuannya:

- Menghindari beban berlebih pada target
- Mengurangi kemungkinan terdeteksi oleh sistem keamanan

##### Escalate Carefully

Naikkan intensitas scan secara bertahap.

Jangan langsung:

- Scan semua port
- Jalankan script vulnerability

Tapi lakukan berdasarkan kebutuhan dan temuan sebelumnya.

##### Save Everything

Selalu simpan hasil scan.

Kenapa penting:

- Bisa dianalisis ulang
- Bisa dibandingkan di kemudian hari
- Berguna untuk dokumentasi

Biasanya menggunakan opsi seperti:

- `-oN` (normal output)
- `-oA` (semua format)

##### Validate Manually

Hasil dari Nmap bukanlah kebenaran mutlak.

Artinya:

- Bisa ada false positive
- Bisa ada informasi yang kurang akurat

Karena itu, kamu tetap perlu:

- Verifikasi manual
- Cross-check dengan tools lain

##### Nmap sebagai Reconnaissance Tool

Perlu dipahami bahwa Nmap berfungsi sebagai alat **reconnaissance (pengumpulan informasi awal)**.

Artinya:

- Nmap memberikan petunjuk (leads)
- Bukan kesimpulan akhir

Contoh:

- Nmap bilang ada kemungkinan vulnerability → kamu tetap harus validasi
- Nmap mendeteksi service → kamu bisa eksplorasi lebih lanjut

Dengan mengikuti alur ini, proses scanning jadi lebih:

- Terarah
- Efisien
- Aman secara praktik

Dan yang paling penting, kamu tidak hanya “menjalankan tool”, tapi benar-benar memahami **strategi di balik setiap langkah scanning**.

## 3. Hubungan Antar Konsep

Ketiga konsep utama dalam video ini saling membangun dalam satu alur kemampuan NMAP:

```
OS Detection (-O)
    ↓
    Memberikan gambaran tentang sistem operasi target.

Aggressive Scan (-A)
    ↓
    Menggabungkan OS Detection + Version Detection + Script Default + Traceroute
    dalam satu perintah yang efisien. Ini adalah "titik tengah" yang seimbang
    antara kecepatan dan kelengkapan informasi.

NSE - Script Default (--script=default / -sC)
    ↓
    Memperluas kemampuan Aggressive Scan untuk mengumpulkan informasi
    lebih spesifik per layanan secara aman.

NSE - Script Vulners (--script=vulners)
    ↓
    Langkah terjauh dan paling agresif — mengidentifikasi CVE aktual
    yang dapat dieksploitasi pada layanan yang ditemukan.
```

Alur ini mencerminkan prinsip **reconnaissance bertahap** dalam penetration testing: mulai dari informasi umum → detail layanan → kerentanan spesifik. Setiap langkah membangun di atas temuan langkah sebelumnya.

---

## 4. Kesimpulan

Video ketiga seri NMAP ini membahas tiga teknik lanjutan yang sangat berguna:

1. **OS Detection (`-O`)** membantu mengidentifikasi sistem operasi target berdasarkan kisaran versi dan informasi jarak jaringan (network distance/hop).

2. **Aggressive Scan (`-A`)** adalah cara paling efisien untuk mendapatkan gambaran menyeluruh tentang target dalam satu perintah, karena menggabungkan OS Detection, Version Detection, Script Default, dan Traceroute.

3. **Nmap Scripting Engine (NSE)** — terutama script `vulners` — secara signifikan meningkatkan kemampuan NMAP menjadi alat pemindai kerentanan yang aktif mencari dan menghubungkan CVE dari database online seperti `vulners.com`.

Seluruh kemampuan ini harus digunakan dalam alur kerja yang terstruktur: dari _quick discovery_, _full TCP scan_, _HTTP enumeration_, hingga _vulnerability check_ — dengan selalu mengutamakan **etika**, **izin eksplisit**, dan **validasi manual** atas setiap temuan.

---

> 📌 _Catatan dibuat berdasarkan video NMAP Part 3. Semua teknik di atas hanya untuk keperluan edukasi dan pengujian pada sistem yang Anda miliki atau memiliki izin resmi._
