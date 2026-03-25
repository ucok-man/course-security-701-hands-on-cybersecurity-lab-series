# 📝 Nmap – Bagian 2: UDP Scan, Kombinasi Scan, dan Version Detection

## 1. Ringkasan Singkat

Video ini merupakan kelanjutan dari seri Nmap (bagian kedua). Setelah sebelumnya membahas dasar-dasar Nmap dan TCP scan, kini video membahas tiga topik utama:

1. **UDP scan** — cara memindai port UDP yang sering diabaikan namun menyimpan layanan penting.
2. **Kombinasi TCP + UDP scan** — cara menggabungkan kedua teknik dalam satu perintah dengan filter port masing-masing.
3. **Service/Version Detection** — cara mendeteksi versi perangkat lunak yang berjalan di balik port terbuka, sekaligus cara memanfaatkan informasi tersebut untuk mencari kerentanan (vulnerability) menggunakan database seperti Exploit-DB.

---

## 2. Konsep Utama

### a. UDP Scan (`-sU`)

#### Kenapa UDP Scan Itu Penting

Saat melakukan scanning jaringan, banyak orang cenderung fokus ke port TCP dan sering mengabaikan UDP. Padahal, ada banyak layanan penting yang justru berjalan di atas protokol UDP, misalnya:

- **DNS** di port 53 → untuk resolusi domain
- **DHCP** di port 67 → untuk pembagian IP otomatis
- **NTP (Network Time Protocol)** di port 123 → untuk sinkronisasi waktu
- **SNMP** di port 161 → untuk monitoring perangkat jaringan

Artinya, kalau kamu tidak melakukan UDP scan, ada kemungkinan kamu melewatkan layanan penting yang sebenarnya aktif di target.

#### Tantangan dalam UDP Scan

Dibandingkan dengan TCP scan, UDP scan punya beberapa keterbatasan yang membuatnya terasa lebih “sulit” dan lambat.

Pertama, ada yang disebut **ICMP rate limiting**. Sistem operasi target biasanya membatasi jumlah respon ICMP (misalnya “port unreachable”) yang dikirim. Akibatnya, Nmap tidak selalu mendapatkan respon yang cukup untuk menentukan status port dengan cepat.

Kedua, ada kondisi yang disebut **silent drops**. Ketika kamu mengirim paket ke port UDP yang tertutup, target bisa saja tidak membalas sama sekali. Dalam situasi ini, Nmap tidak langsung tahu apakah port tersebut tertutup atau hanya tidak merespon, sehingga harus menunggu hingga timeout.

Karena dua hal ini, UDP scan:

- Lebih lambat
- Kurang konsisten hasilnya dibanding TCP scan

#### Strategi yang Disarankan

Karena scanning semua port UDP (0–65535) akan sangat memakan waktu dan tidak efisien, praktik terbaiknya adalah:

- Fokus hanya pada port-port UDP yang umum digunakan
- Target layanan yang memang kemungkinan besar aktif

Ini akan menghemat waktu sekaligus meningkatkan relevansi hasil scan.

#### Perintah Dasar UDP Scan

Untuk melakukan UDP scan di Nmap, kamu bisa menggunakan perintah berikut:

```bash
sudo nmap -sU -p 53,67,123 <IP_TARGET>
```

Penjelasan flag yang digunakan:

| Flag  | Keterangan                          |
| ----- | ----------------------------------- |
| `-sU` | Mengaktifkan mode UDP scan          |
| `-p`  | Menentukan port yang ingin dipindai |

Perlu diperhatikan bahwa UDP scan membutuhkan akses **superuser** (`sudo`). Hal ini karena Nmap perlu membuat raw socket untuk mengirim paket UDP secara langsung.

#### Contoh Output dan Cara Membacanya

Berikut contoh hasil scan ketika semua port yang dicek dalam kondisi tertutup:

```
PORT    STATE  SERVICE
53/udp  closed domain
67/udp  closed dhcps
123/udp closed ntp
```

Mari kita breakdown:

- **53/udp closed domain**
  - Port 53 (DNS) dalam kondisi **closed**, artinya tidak ada layanan DNS yang berjalan di port tersebut.

- **67/udp closed dhcps**
  - Port DHCP juga tertutup, jadi tidak ada service DHCP yang aktif di target.

- **123/udp closed ntp**
  - Port NTP tertutup, tidak ada layanan sinkronisasi waktu.

Dari output ini, kita bisa menyimpulkan bahwa tidak ada layanan UDP umum yang berjalan pada target untuk port yang kita scan.

Namun penting diingat, karena sifat UDP yang tidak selalu memberikan respon, status seperti `open|filtered` juga sering muncul di kasus lain. Itu berarti Nmap tidak bisa memastikan apakah port terbuka atau difilter oleh firewall.

#### Kesimpulan Singkat

UDP scan adalah teknik penting untuk menemukan layanan yang sering terlewat, tetapi:

- Lebih lambat dan tidak seandal TCP scan
- Rentan terhadap timeout dan minim respon
- Lebih efektif jika digunakan secara selektif pada port tertentu

Dengan memahami karakteristik ini, kamu bisa menggunakan UDP scan dengan lebih strategis dan efisien.

### b. Kombinasi TCP + UDP Scan

#### Konsep Dasar: Kenapa Perlu Digabung?

Dalam praktik nyata, layanan di sebuah server tidak hanya berjalan di satu jenis protokol saja. Ada yang menggunakan TCP, ada juga yang menggunakan UDP. Jika kamu hanya melakukan salah satu jenis scan, maka hasilnya tidak akan lengkap.

Di sinilah Nmap memberikan fleksibilitas: kamu bisa menjalankan **TCP scan dan UDP scan sekaligus dalam satu perintah**. Ini sangat berguna untuk mendapatkan gambaran yang lebih menyeluruh tentang layanan yang berjalan di target.

#### Flag yang Digunakan

Untuk melakukan kombinasi scan ini, ada beberapa flag penting yang perlu dipahami:

- `-sS` → digunakan untuk **TCP SYN scan** (sering disebut stealth scan)
- `-sU` → digunakan untuk **UDP scan**
- `T:` → prefix untuk mendefinisikan port TCP
- `U:` → prefix untuk mendefinisikan port UDP

Dengan prefix ini, kamu bisa mengatur port mana yang ingin dipindai untuk masing-masing protokol secara terpisah dalam satu command.

#### Contoh Perintah Kombinasi

Berikut contoh penggunaan kombinasi TCP dan UDP scan:

```bash
sudo nmap -sS -sU -p T:1-2000,U:53,67,123 <IP_TARGET>
```

Mari kita bahas bagian per bagian:

- `-sS` → mengaktifkan TCP SYN scan untuk mendeteksi port TCP yang terbuka
- `-sU` → mengaktifkan UDP scan untuk mendeteksi port UDP
- `-p` → digunakan untuk menentukan port yang ingin dipindai
- `T:1-2000` → artinya Nmap akan memindai port TCP dari 1 sampai 2000
- `U:53,67,123` → artinya Nmap hanya memindai port UDP tertentu, yaitu 53 (DNS), 67 (DHCP), dan 123 (NTP)

Dengan cara ini, kamu bisa mengombinasikan scanning luas untuk TCP (range besar) dan scanning selektif untuk UDP (port tertentu saja).

#### Kenapa UDP Tetap Dipilih Secara Selektif?

Seperti yang sudah dibahas sebelumnya, UDP scan memiliki karakteristik:

- Lebih lambat
- Sering tidak memberikan respon (silent)
- Bergantung pada ICMP yang bisa dibatasi (rate limiting)

Jika kamu memaksakan scan semua port UDP bersamaan dengan TCP, prosesnya bisa menjadi sangat lama dan tidak efisien.

Karena itu, pendekatan terbaik adalah:

- TCP → scan lebih luas (misalnya range besar seperti 1–2000)
- UDP → fokus ke port yang umum dan penting saja

#### Tips Praktis

Saat menggunakan kombinasi scan ini, ada beberapa hal yang perlu diperhatikan:

- Gunakan `sudo` karena kedua metode scan membutuhkan akses raw socket
- Jangan terlalu agresif dalam memilih port UDP, cukup yang relevan saja
- Siapkan waktu lebih lama dibanding TCP-only scan

Pendekatan kombinasi ini sangat powerful, karena kamu bisa mendapatkan visibilitas yang lebih lengkap terhadap target tanpa harus menjalankan dua scan terpisah.

### c. Service and Version Detection (`-sV`)

#### Kenapa Perlu Mengetahui Service dan Versi?

Setelah kamu berhasil menemukan port yang terbuka, sebenarnya itu baru langkah awal. Informasi “port terbuka” saja belum cukup. Yang lebih penting adalah:

- Layanan apa yang berjalan di port tersebut?
- Versi berapa dari layanan tersebut?

Kenapa ini penting? Karena versi software sering berkaitan langsung dengan **kerentanan (vulnerability)**. Dengan mengetahui versinya, kamu bisa:

- Mengidentifikasi potensi celah keamanan
- Menentukan langkah eksploitasi atau hardening selanjutnya
- Mendapat gambaran sistem target secara lebih spesifik

#### Cara Kerja `-sV` di Nmap

Saat kamu menggunakan flag `-sV`, Nmap tidak hanya berhenti di deteksi port terbuka. Ia akan melanjutkan dengan:

1. Mengirim berbagai **probe** (semacam request khusus) ke port yang terbuka
2. Menganalisis respons yang diberikan oleh layanan di port tersebut
3. Mencocokkan respons itu dengan database internal milik Nmap

Database ini dikenal sebagai **`nmap-service-probes`**, yang berisi ribuan pola respons dari berbagai layanan dan versi software.

Dari proses ini, Nmap bisa “menebak”:

- Nama layanan (misalnya: SSH, HTTP, FTP)
- Versi software (misalnya: OpenSSH 5.3, Apache 2.2.14)

#### Komponen Penting di Balik Version Detection

Ada beberapa konsep utama yang bekerja di balik fitur ini:

- **Nmap Service Probes**
  Ini adalah database utama yang berisi berbagai jenis probe dan signature (pola respons). Bisa dibilang ini adalah “otaknya” version detection.

- **Service Fingerprinting**
  Proses mencocokkan respons dari target dengan signature yang ada di database. Mirip seperti sidik jari—setiap layanan punya pola respons unik.

- **Intensity Control**
  Nmap memungkinkan kamu mengatur seberapa agresif probing yang dilakukan.
  Semakin tinggi intensitasnya:
  - Semakin akurat hasilnya
  - Tapi waktu scan juga jadi lebih lama

#### Perintah Dasar Version Detection

Untuk mengaktifkan fitur ini, kamu bisa menggunakan perintah berikut:

```bash id="b6jw22"
sudo nmap -sV <IP_TARGET>
```

Perintah ini akan:

- Melakukan scan port (default)
- Lalu langsung mencoba mendeteksi service dan versinya pada port yang terbuka

Biasanya tetap dijalankan dengan `sudo` agar hasilnya lebih maksimal.

#### Contoh Output dan Penjelasannya

Berikut contoh hasil scan dengan version detection:

```id="p93l0z"
PORT     STATE  SERVICE     VERSION
22/tcp   open   ssh         OpenSSH 5.3 (Debian/Ubuntu Linux)
80/tcp   open   http        Apache httpd 2.2.14
139/tcp  open   netbios-ssn Samba smbd
143/tcp  open   imap        Courier imapd
```

Mari kita bahas satu per satu:

- **22/tcp → OpenSSH 5.3 (Debian/Ubuntu Linux)**
  Artinya port SSH terbuka dan menggunakan OpenSSH versi 5.3.
  Bahkan, Nmap bisa memberikan hint bahwa sistemnya kemungkinan berbasis Debian/Ubuntu.

- **80/tcp → Apache httpd 2.2.14**
  Server web menggunakan Apache versi tertentu. Ini sangat berguna untuk mencari CVE yang relevan.

- **139/tcp → Samba smbd**
  Layanan file sharing berbasis SMB aktif.

- **143/tcp → Courier imapd**
  Layanan email IMAP berjalan di port ini.

Dari sini, kamu tidak hanya tahu port mana yang terbuka, tapi juga “siapa” yang ada di balik port tersebut.

#### Keunggulan Utama `-sV`

Salah satu kekuatan terbesar dari version detection adalah kemampuannya memberikan **konteks lebih dalam**.

Contohnya:

- Dari “OpenSSH 5.3” kamu bisa:
  - Mencari vulnerability spesifik
  - Menebak sistem operasi (misalnya Debian/Ubuntu Linux)

- Dari “Apache 2.2.14” kamu bisa:
  - Mengecek apakah versi tersebut sudah usang
  - Menemukan exploit yang sesuai

Dengan kata lain, `-sV` mengubah hasil scan dari sekadar daftar port menjadi **informasi yang bisa langsung ditindaklanjuti** dalam proses analisis keamanan.

### d. Memanfaatkan Informasi Versi untuk Mencari Vulnerability

#### Kenapa Informasi Versi Itu Sangat Berharga?

Setelah kamu mendapatkan informasi versi layanan menggunakan `-sV`, sebenarnya kamu sudah memegang “kunci” untuk melangkah lebih jauh dalam analisis keamanan.

Versi software sangat penting karena:

- Banyak vulnerability yang **spesifik terhadap versi tertentu**
- Exploit biasanya ditulis untuk versi yang jelas (misalnya: Apache 2.2.x, bukan sekadar Apache)
- Semakin spesifik versinya, semakin akurat pencarian celahnya

Dengan kata lain, dari sekadar “port terbuka”, kamu sekarang bisa mulai menjawab:

> “Apakah layanan ini punya celah yang sudah diketahui?”

#### Sumber untuk Mencari Vulnerability

Ada dua sumber utama yang umum digunakan oleh praktisi keamanan:

- **`searchsploit`**
  Tool berbasis CLI yang memungkinkan kamu mencari exploit dari database Exploit-DB secara offline. Cocok untuk workflow cepat di terminal.

- **exploitdb.com**
  Website resmi Exploit-DB yang berisi kumpulan exploit publik. Di sini kamu bisa mencari berdasarkan nama software, versi, atau CVE.

Keduanya menggunakan database yang sama, hanya berbeda cara aksesnya (CLI vs web).

#### Alur Kerja Praktis

Berikut langkah yang biasanya dilakukan setelah mendapatkan hasil dari Nmap:

```
1. Jalankan: sudo nmap -sV <TARGET>
2. Catat versi layanan yang terbuka, misal: Apache httpd 2.2.14
3. Cari di Exploit-DB: searchsploit apache 2.2.14
   atau buka exploitdb.com dan cari "Apache httpd 2.2"
4. Temukan exploit yang cocok dengan versi tersebut
5. Pelajari jenis eksploit (DoS, RCE, dll.), platform, dan CVE-nya
```

Mari kita pahami alurnya secara lebih dalam:

- Setelah scan, kamu menemukan misalnya: **Apache httpd 2.2.14**
- Lalu kamu cari di `searchsploit`:

  ```bash
  searchsploit apache 2.2.14
  ```

- Jika ada hasil, berarti versi tersebut **pernah memiliki vulnerability yang terdokumentasi**

Selanjutnya, kamu tidak langsung “menjalankan exploit”, tapi:

- Membaca deskripsinya
- Memahami jenis serangannya
- Melihat apakah relevan dengan target

#### Memahami Hasil dari Exploit-DB

Biasanya, setiap entri exploit di Exploit-DB memiliki beberapa informasi penting:

- **Nama**
  Menjelaskan target dan jenis celah, misalnya modul tertentu di Apache

- **Tipe**
  Jenis serangan, seperti:
  - DoS (Denial of Service)
  - RCE (Remote Code Execution)
  - LFI/RFI (File Inclusion)

- **Platform**
  Sistem operasi yang terdampak (Linux, Windows, dll.)

- **CVE**
  Identifier resmi dari vulnerability tersebut (misalnya CVE-2021-XXXX)

- **EDB-ID**
  ID unik di database Exploit-DB

Contoh sederhana:

- Nama: Apache httpd mod\_\* Denial of Service
- Tipe: DoS
- Platform: Linux/Unix
- CVE: CVE-XXXX-XXXX

Dari informasi ini, kamu bisa memahami:

- Apa dampaknya?
- Seberapa serius?
- Apakah target kamu cocok dengan kondisi exploit tersebut?

#### Kaitan dengan Bug Bounty dan Dunia Nyata

Teknik ini bukan hanya untuk pembelajaran atau lab, tapi juga digunakan di dunia nyata, terutama dalam konteks **bug bounty program**.

Alurnya biasanya seperti ini:

1. Peneliti menemukan service + versi menggunakan tools seperti Nmap
2. Mereka mencocokkan versi tersebut dengan database vulnerability
3. Jika ditemukan celah yang valid:
   - Dilaporkan ke perusahaan terkait
   - Mendapat reward (bug bounty)

4. Perusahaan kemudian:
   - Memverifikasi laporan
   - Merilis **security patch** untuk menutup celah tersebut

Jadi, kemampuan menghubungkan antara **version detection → vulnerability database** adalah skill inti dalam security testing.

#### Kesimpulan

Informasi versi bukan sekadar detail tambahan, tapi fondasi untuk:

- Mendeteksi vulnerability yang sudah diketahui
- Menghubungkan hasil scanning dengan exploit nyata
- Melangkah dari “reconnaissance” ke “analysis”

Semakin teliti kamu membaca versi layanan, semakin besar peluangmu menemukan celah yang relevan.

### e. Verifikasi Hasil Scan

#### Kenapa Perlu Verifikasi?

Hasil dari Nmap, termasuk service dan version detection (`-sV`), pada dasarnya adalah **hasil analisis berbasis probe dan pencocokan signature**. Artinya, meskipun sering akurat, hasil tersebut tetap bersifat _estimasi_.

Karena itu, dalam praktik yang lebih serius (misalnya penetration testing atau audit keamanan), penting untuk melakukan **verifikasi langsung di mesin target** jika kamu memiliki akses.

Tujuannya:

- Memastikan hasil scan benar-benar akurat
- Menghindari false positive (deteksi yang keliru)
- Memberikan validasi sebelum mengambil langkah lanjutan (misalnya eksploitasi atau pelaporan)

#### Cara Melakukan Verifikasi

Jika kamu memiliki akses ke server target (misalnya melalui SSH), kamu bisa langsung menjalankan perintah bawaan dari service tersebut untuk melihat versinya.

Contoh untuk Apache:

```bash id="t8f0rx"
apache2 -v
```

Perintah ini akan menampilkan versi Apache yang benar-benar terinstall dan berjalan di sistem.

#### Contoh Output

```id="v2m4hs"
Server version: Apache/2.2.14 (Ubuntu)
```

Dari output ini, kita bisa melihat:

- Software: Apache HTTP Server
- Versi: 2.2.14
- Sistem: Ubuntu

#### Membandingkan dengan Hasil Nmap

Misalnya sebelumnya kamu mendapatkan hasil dari Nmap seperti ini:

```
80/tcp open http Apache httpd 2.2.14
```

Jika dibandingkan:

- Nmap: Apache httpd 2.2.14
- Server langsung: Apache/2.2.14 (Ubuntu)

Keduanya menunjukkan versi yang sama. Ini berarti:

- **Version detection Nmap akurat**
- Informasi tersebut bisa dipercaya untuk analisis lebih lanjut

#### Insight Penting

Verifikasi seperti ini memberikan kepercayaan lebih terhadap hasil scanning. Dalam beberapa kasus, Nmap bisa saja:

- Salah mengidentifikasi versi (karena respons mirip)
- Tidak mendapatkan versi secara lengkap
- Menampilkan `unknown` atau `filtered`

Dengan melakukan pengecekan langsung:

- Kamu mendapatkan **ground truth** (data yang benar-benar valid dari sistem)
- Bisa memastikan apakah hasil scanning layak dijadikan dasar keputusan

#### Kesimpulan

Nmap adalah alat yang sangat powerful, tapi tetap penting untuk diingat bahwa:

- Hasilnya berbasis inferensi, bukan akses langsung
- Verifikasi manual adalah langkah penting jika memungkinkan

Dengan membiasakan verifikasi, kamu akan bekerja lebih akurat dan profesional dalam analisis jaringan maupun keamanan.

## 3. Hubungan Antar Konsep

Berikut adalah alur pemahaman yang menghubungkan seluruh konsep di atas:

```
[TCP Scan (Part 1)]
        ↓
[UDP Scan (-sU)]  ←  Melengkapi TCP scan karena banyak layanan penting di UDP
        ↓
[Kombinasi TCP + UDP (-sS -sU)]  ←  Scan komprehensif dalam satu perintah
        ↓
[Version Detection (-sV)]  ←  Dari port terbuka → identifikasi software & versi
        ↓
[Exploit Research (Exploit-DB / searchsploit)]  ←  Dari versi → cari kerentanan
        ↓
[Penetration Testing / Bug Bounty / Security Patching]
```

Intinya, setiap teknik scan adalah **lapisan informasi yang semakin dalam**:

- Port terbuka → **apa yang bisa diakses?**
- Versi layanan → **apakah ada celah keamanan yang diketahui?**
- Exploit database → **bagaimana celah itu bisa dieksploitasi?**

---

## 4. Kesimpulan

Nmap Part 2 memperluas kemampuan scanning dari sekadar mendeteksi port TCP ke teknik yang lebih komprehensif:

1. **UDP scan (`-sU`)** memungkinkan kita menemukan layanan penting seperti DNS dan DHCP yang tidak terdeteksi oleh TCP scan, meskipun lebih lambat karena sifat protokol UDP itu sendiri.

2. **Kombinasi scan (`-sS -sU`)** memungkinkan penggabungan TCP dan UDP scan dengan filter port independen, memberikan gambaran yang lebih lengkap tentang permukaan serangan (_attack surface_) suatu host.

3. **Version detection (`-sV`)** adalah salah satu fitur paling kritis Nmap dalam konteks keamanan siber — dari sini kita bisa mengetahui versi perangkat lunak yang berjalan, mencocokkannya dengan database exploit, dan berpotensi menemukan kerentanan yang dapat dimanfaatkan atau dilaporkan.

Video ini menutup dengan teaser bahwa **Part 3** akan mengeksplorasi lebih banyak jenis scan Nmap dan fitur-fitur lanjutan lainnya.

---

> 🔐 **Peringatan Etis:** Semua teknik yang dibahas hanya boleh digunakan pada sistem yang **Anda miliki sendiri** atau yang telah diberikan **izin eksplisit** untuk diuji. Penggunaan tanpa izin adalah ilegal.
