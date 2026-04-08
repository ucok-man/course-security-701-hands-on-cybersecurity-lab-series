Berikut adalah catatan studi lengkap berdasarkan konten video tersebut:

---

# 📝 Evil Twin Attack dengan Airgeddon

## 1. Ringkasan Singkat

Video ini membahas konsep dan praktik **Evil Twin Attack**, sebuah bentuk serangan _wireless Man-in-the-Middle (MitM)_ di mana penyerang membuat access point Wi-Fi palsu yang meniru jaringan legitim untuk mengelabui klien agar terhubung ke dalamnya. Tool utama yang digunakan adalah **Airgeddon**, sebuah skrip audit Wi-Fi berbasis Linux yang mengotomatisasi berbagai alat hacking jaringan nirkabel. Karena keterbatasan hardware, video ini menggabungkan demonstrasi langsung di Kali Linux dengan penjelasan konseptual berbasis slideshow.

---

## 2. Konsep Utama

### 📝 Evil Twin Attack dengan Airgeddon

#### Ringkasan Singkat

Materi ini membahas tentang **Evil Twin Attack**, yaitu salah satu teknik serangan jaringan nirkabel (_wireless_) yang termasuk dalam kategori _Man-in-the-Middle (MitM)_. Intinya, penyerang membuat jaringan Wi-Fi palsu yang terlihat identik dengan jaringan asli, sehingga korban tanpa sadar terhubung ke jaringan tersebut.

Dalam praktiknya, digunakan tool bernama Airgeddon, yaitu skrip berbasis Linux (umumnya dijalankan di Kali Linux) yang mengotomatisasi berbagai tools hacking Wi-Fi seperti **aircrack-ng**, **MDK4**, dan **dhcpd**.

Karena keterbatasan perangkat (terutama adapter Wi-Fi), demonstrasi dalam materi ini merupakan kombinasi antara praktik langsung dan penjelasan konsep melalui slideshow.

#### Konsep Utama

##### Evil Twin Attack — Definisi & Tujuan

Evil Twin Attack adalah teknik di mana penyerang membuat **access point (AP) palsu** yang meniru jaringan asli, baik dari sisi nama (SSID) maupun karakteristik teknisnya.

Beberapa hal yang ditiru:

- Nama jaringan (SSID) dibuat **persis sama**
- Channel Wi-Fi disamakan
- Tampilan login (captive portal) dibuat menyerupai aslinya

Tujuannya cukup luas, di antaranya:

- Mengintip lalu lintas jaringan (_traffic interception_)
- Mencuri username dan password melalui halaman login palsu
- Memanipulasi data yang lewat (_traffic manipulation_)
- Mengambil **WPA handshake** untuk cracking password secara offline
- Menjalankan serangan lanjutan seperti:
  - SSL stripping
  - DNS spoofing
  - Packet injection

Dengan kata lain, penyerang berusaha “menjadi perantara” antara korban dan internet.

##### Alur Serangan (Attack Flow)

Supaya lebih mudah dipahami, kita uraikan alurnya langkah demi langkah.

###### Tahap 1 — Recon & Selection (Pengintaian)

Penyerang mulai dengan mencari target Wi-Fi.

Contoh:

```
Target: SSID = "CoffeeShop_WiFi", Channel = 6
```

Di sini penyerang mengamati:

- Nama jaringan
- Channel yang digunakan
- Aktivitas klien yang terhubung

Tujuannya adalah agar nanti AP palsu bisa dibuat semirip mungkin.

###### Tahap 2 — AP Impersonation (Meniru Access Point)

Selanjutnya, penyerang membuat AP palsu:

```
Fake AP: SSID = "CoffeeShop_WiFi"
```

Ciri khasnya:

- Nama sama persis
- Sinyal dibuat lebih kuat

Karena sebagian besar perangkat akan otomatis memilih jaringan dengan sinyal terbaik, peluang korban terhubung ke AP palsu jadi lebih besar.

###### Tahap 3 — Client Deauthentication

Tahap ini opsional, tapi sering digunakan untuk mempercepat serangan.

Penyerang mengirimkan _deauthentication frame_ untuk memutus koneksi korban dari jaringan asli:

```
Tool: MDK4 / aireplay-ng
Frame: 802.11 Deauthentication
```

Akibatnya:

- Korban terputus dari Wi-Fi asli
- Perangkat akan mencoba reconnect
- Dan kemungkinan besar akan masuk ke AP palsu

###### Tahap 4 — Association & DHCP

Setelah korban terhubung ke AP palsu:

- Perangkat akan meminta IP address
- Penyerang menyediakan DHCP server

Contoh:

```
dhcpd → memberikan IP ke korban
```

Sekarang korban sudah “sepenuhnya” berada di jaringan milik penyerang.

###### Tahap 5 — Traffic Handling (MitM & Captive Portal)

Ini adalah inti serangan.

Ada dua skenario utama:

1. **MitM (Man-in-the-Middle)**
   Penyerang meneruskan koneksi ke internet sambil:
   - Mengamati data
   - Mengubah isi traffic jika perlu

2. **Captive Portal Palsu**
   Korban diarahkan ke halaman login palsu yang terlihat seperti asli.
   Misalnya diminta memasukkan password Wi-Fi.

Masalahnya:

- Data yang dimasukkan korban bisa langsung tertangkap
- Biasanya dalam bentuk **plaintext**

###### Tahap 6 — Post Capture

Setelah data terkumpul, penyerang bisa:

- Melakukan cracking password dari WPA handshake (offline)
- Menggunakan kembali kredensial yang dicuri
- Menganalisis traffic untuk informasi sensitif

##### Catatan Keamanan

Beberapa teknologi modern bisa mengurangi efektivitas serangan ini:

- HTTPS
- HSTS
- Certificate pinning
- 802.1X (enterprise Wi-Fi)

Namun, kesalahan konfigurasi dan faktor manusia (misalnya asal klik login) tetap menjadi celah utama.

#### Airgeddon — Tool Utama

Airgeddon adalah skrip berbasis Bash yang menggabungkan banyak tools menjadi satu workflow otomatis.

Kelebihannya:

- Tidak perlu menjalankan tool satu per satu
- Sudah ada pengecekan dependensi
- Proses serangan jadi lebih praktis

Cara instalasi:

```bash
sudo git clone https://github.com/v1s1t0r1sh3r3/airgeddon
cd airgeddon
sudo bash airgeddon.sh
```

Saat dijalankan, Airgeddon akan:

1. Mengecek tool yang dibutuhkan
2. Menawarkan instalasi jika ada yang kurang
3. Meminta memilih network interface
4. Menampilkan menu serangan

#### Wireless Adapter yang Diperlukan

Tidak semua Wi-Fi adapter bisa digunakan untuk serangan ini.

Minimal harus mendukung:

- **Monitor Mode**
- **Packet Injection**

Contoh adapter yang umum digunakan:

- Alfa (berbagai model)
- TP-Link TL-WN722N **versi 1 saja**
- Panda Wireless

Catatan penting:
Versi 2 dan 3 dari TL-WN722N **tidak mendukung** fitur yang dibutuhkan.

#### Langkah Operasional di Airgeddon

Berikut gambaran alur penggunaannya:

1. Pilih opsi:

   ```
   Evil Twin AP attack with sniffing and SSL strip
   ```

2. Tool akan scan Wi-Fi menggunakan **airodump-ng**
3. Pilih target setelah scan selesai
4. Pilih metode deauthentication
5. Lakukan MAC spoofing (opsional tapi disarankan)
6. Tentukan apakah sudah punya handshake
7. Gunakan timeout default
8. Pilih bahasa captive portal
9. Biarkan semua window yang muncul tetap terbuka

Hasil di sisi korban:

- Muncul dua jaringan dengan nama sama
- Salah satunya tanpa password (Evil Twin)
- Korban terhubung ke jaringan palsu
- Muncul halaman login palsu
- Password yang dimasukkan langsung tertangkap

#### Hubungan Antar Konsep

Alur besarnya bisa dipahami seperti ini:

- Penyerang menggunakan Airgeddon
  → melakukan scanning
  → membuat AP palsu
  → memutus koneksi korban
  → korban pindah ke AP palsu
  → penyerang mengontrol traffic atau menampilkan captive portal
  → data korban dikumpulkan

Semua ini terjadi karena:

- Perangkat memilih sinyal terkuat
- Tool otomatis menyederhanakan proses kompleks
- Korban mudah tertipu tampilan yang mirip asli

#### Kesimpulan

Evil Twin Attack adalah teknik yang cukup efektif karena memanfaatkan perilaku default perangkat Wi-Fi: memilih jaringan dengan sinyal paling kuat tanpa verifikasi mendalam.

Dengan bantuan Airgeddon di Kali Linux, serangan ini bisa dijalankan dengan relatif mudah karena banyak proses sudah diotomatisasi.

Faktor keberhasilan utama:

1. Adapter yang mendukung monitor mode dan packet injection
2. AP palsu yang sangat mirip (SSID + sinyal kuat)
3. Deauthentication untuk “memaksa” korban pindah
4. Captive portal yang meyakinkan

Dari sisi pertahanan:

### 📝 Evil Twin Attack dengan Airgeddon

#### Ringkasan Singkat

Materi ini membahas tentang **Evil Twin Attack**, yaitu salah satu teknik serangan jaringan nirkabel (_wireless_) yang termasuk dalam kategori _Man-in-the-Middle (MitM)_. Intinya, penyerang membuat jaringan Wi-Fi palsu yang terlihat identik dengan jaringan asli, sehingga korban tanpa sadar terhubung ke jaringan tersebut.

Dalam praktiknya, digunakan tool bernama Airgeddon, yaitu skrip berbasis Linux (umumnya dijalankan di Kali Linux) yang mengotomatisasi berbagai tools hacking Wi-Fi seperti **aircrack-ng**, **MDK4**, dan **dhcpd**.

Karena keterbatasan perangkat (terutama adapter Wi-Fi), demonstrasi dalam materi ini merupakan kombinasi antara praktik langsung dan penjelasan konsep melalui slideshow.

#### Konsep Utama

##### Evil Twin Attack — Definisi & Tujuan

Evil Twin Attack adalah teknik di mana penyerang membuat **access point (AP) palsu** yang meniru jaringan asli, baik dari sisi nama (SSID) maupun karakteristik teknisnya.

Beberapa hal yang ditiru:

- Nama jaringan (SSID) dibuat **persis sama**
- Channel Wi-Fi disamakan
- Tampilan login (captive portal) dibuat menyerupai aslinya

Tujuannya cukup luas, di antaranya:

- Mengintip lalu lintas jaringan (_traffic interception_)
- Mencuri username dan password melalui halaman login palsu
- Memanipulasi data yang lewat (_traffic manipulation_)
- Mengambil **WPA handshake** untuk cracking password secara offline
- Menjalankan serangan lanjutan seperti:
  - SSL stripping
  - DNS spoofing
  - Packet injection

Dengan kata lain, penyerang berusaha “menjadi perantara” antara korban dan internet.

##### Alur Serangan (Attack Flow)

Supaya lebih mudah dipahami, kita uraikan alurnya langkah demi langkah.

###### Tahap 1 — Recon & Selection (Pengintaian)

Penyerang mulai dengan mencari target Wi-Fi.

Contoh:

```
Target: SSID = "CoffeeShop_WiFi", Channel = 6
```

Di sini penyerang mengamati:

- Nama jaringan
- Channel yang digunakan
- Aktivitas klien yang terhubung

Tujuannya adalah agar nanti AP palsu bisa dibuat semirip mungkin.

###### Tahap 2 — AP Impersonation (Meniru Access Point)

Selanjutnya, penyerang membuat AP palsu:

```
Fake AP: SSID = "CoffeeShop_WiFi"
```

Ciri khasnya:

- Nama sama persis
- Sinyal dibuat lebih kuat

Karena sebagian besar perangkat akan otomatis memilih jaringan dengan sinyal terbaik, peluang korban terhubung ke AP palsu jadi lebih besar.

###### Tahap 3 — Client Deauthentication

Tahap ini opsional, tapi sering digunakan untuk mempercepat serangan.

Penyerang mengirimkan _deauthentication frame_ untuk memutus koneksi korban dari jaringan asli:

```
Tool: MDK4 / aireplay-ng
Frame: 802.11 Deauthentication
```

Akibatnya:

- Korban terputus dari Wi-Fi asli
- Perangkat akan mencoba reconnect
- Dan kemungkinan besar akan masuk ke AP palsu

###### Tahap 4 — Association & DHCP

Setelah korban terhubung ke AP palsu:

- Perangkat akan meminta IP address
- Penyerang menyediakan DHCP server

Contoh:

```
dhcpd → memberikan IP ke korban
```

Sekarang korban sudah “sepenuhnya” berada di jaringan milik penyerang.

###### Tahap 5 — Traffic Handling (MitM & Captive Portal)

Ini adalah inti serangan.

Ada dua skenario utama:

1. **MitM (Man-in-the-Middle)**
   Penyerang meneruskan koneksi ke internet sambil:
   - Mengamati data
   - Mengubah isi traffic jika perlu

2. **Captive Portal Palsu**
   Korban diarahkan ke halaman login palsu yang terlihat seperti asli.
   Misalnya diminta memasukkan password Wi-Fi.

Masalahnya:

- Data yang dimasukkan korban bisa langsung tertangkap
- Biasanya dalam bentuk **plaintext**

###### Tahap 6 — Post Capture

Setelah data terkumpul, penyerang bisa:

- Melakukan cracking password dari WPA handshake (offline)
- Menggunakan kembali kredensial yang dicuri
- Menganalisis traffic untuk informasi sensitif

##### Catatan Keamanan

Beberapa teknologi modern bisa mengurangi efektivitas serangan ini:

- HTTPS
- HSTS
- Certificate pinning
- 802.1X (enterprise Wi-Fi)

Namun, kesalahan konfigurasi dan faktor manusia (misalnya asal klik login) tetap menjadi celah utama.

#### Airgeddon — Tool Utama

Airgeddon adalah skrip berbasis Bash yang menggabungkan banyak tools menjadi satu workflow otomatis.

Kelebihannya:

- Tidak perlu menjalankan tool satu per satu
- Sudah ada pengecekan dependensi
- Proses serangan jadi lebih praktis

Cara instalasi:

```bash
sudo git clone https://github.com/v1s1t0r1sh3r3/airgeddon
cd airgeddon
sudo bash airgeddon.sh
```

Saat dijalankan, Airgeddon akan:

1. Mengecek tool yang dibutuhkan
2. Menawarkan instalasi jika ada yang kurang
3. Meminta memilih network interface
4. Menampilkan menu serangan

#### Wireless Adapter yang Diperlukan

Tidak semua Wi-Fi adapter bisa digunakan untuk serangan ini.

Minimal harus mendukung:

- **Monitor Mode**
- **Packet Injection**

Contoh adapter yang umum digunakan:

- Alfa (berbagai model)
- TP-Link TL-WN722N **versi 1 saja**
- Panda Wireless

Catatan penting:
Versi 2 dan 3 dari TL-WN722N **tidak mendukung** fitur yang dibutuhkan.

#### Langkah Operasional di Airgeddon

Berikut gambaran alur penggunaannya:

1. Pilih opsi:

   ```
   Evil Twin AP attack with sniffing and SSL strip
   ```

2. Tool akan scan Wi-Fi menggunakan **airodump-ng**
3. Pilih target setelah scan selesai
4. Pilih metode deauthentication
5. Lakukan MAC spoofing (opsional tapi disarankan)
6. Tentukan apakah sudah punya handshake
7. Gunakan timeout default
8. Pilih bahasa captive portal
9. Biarkan semua window yang muncul tetap terbuka

Hasil di sisi korban:

- Muncul dua jaringan dengan nama sama
- Salah satunya tanpa password (Evil Twin)
- Korban terhubung ke jaringan palsu
- Muncul halaman login palsu
- Password yang dimasukkan langsung tertangkap

#### Hubungan Antar Konsep

Alur besarnya bisa dipahami seperti ini:

- Penyerang menggunakan Airgeddon
  → melakukan scanning
  → membuat AP palsu
  → memutus koneksi korban
  → korban pindah ke AP palsu
  → penyerang mengontrol traffic atau menampilkan captive portal
  → data korban dikumpulkan

Semua ini terjadi karena:

- Perangkat memilih sinyal terkuat
- Tool otomatis menyederhanakan proses kompleks
- Korban mudah tertipu tampilan yang mirip asli

#### Kesimpulan

Evil Twin Attack adalah teknik yang cukup efektif karena memanfaatkan perilaku default perangkat Wi-Fi: memilih jaringan dengan sinyal paling kuat tanpa verifikasi mendalam.

Dengan bantuan Airgeddon di Kali Linux, serangan ini bisa dijalankan dengan relatif mudah karena banyak proses sudah diotomatisasi.

Faktor keberhasilan utama:

1. Adapter yang mendukung monitor mode dan packet injection
2. AP palsu yang sangat mirip (SSID + sinyal kuat)
3. Deauthentication untuk “memaksa” korban pindah
4. Captive portal yang meyakinkan

Dari sisi pertahanan:

- Gunakan autentikasi kuat seperti 802.1X
- Pastikan HTTPS + HSTS aktif
- Waspada jika ada dua jaringan dengan nama sama
- Hindari memasukkan password di halaman mencurigakan

Memahami alur ini penting, bukan hanya untuk menyerang, tapi terutama untuk mengenali dan mencegah serangan di dunia nyata.

- Gunakan autentikasi kuat seperti 802.1X
- Pastikan HTTPS + HSTS aktif
- Waspada jika ada dua jaringan dengan nama sama
- Hindari memasukkan password di halaman mencurigakan

Memahami alur ini penting, bukan hanya untuk menyerang, tapi terutama untuk mengenali dan mencegah serangan di dunia nyata.

### b. Alur Serangan (Attack Flow)

Untuk memahami Evil Twin Attack dengan baik, kita perlu melihatnya sebagai sebuah proses bertahap. Setiap tahap saling terhubung dan membentuk alur logis dari awal hingga akhir serangan. Mari kita bahas satu per satu secara runtut.

#### 🔍 Tahap 1 — Recon & Selection (Pengintaian)

Di tahap awal ini, penyerang belum melakukan serangan aktif. Fokusnya adalah **mengumpulkan informasi** tentang target.

Hal-hal yang diamati:

- Nama Wi-Fi (SSID)
- Channel yang digunakan
- Aktivitas perangkat yang terhubung

Contohnya:

```
Target: SSID = "CoffeeShop_WiFi", Channel = 6
```

Artinya penyerang sudah mengetahui bahwa jaringan target bernama _CoffeeShop_WiFi_ dan berjalan di channel 6.

Kenapa ini penting?
Karena pada tahap berikutnya, penyerang harus membuat jaringan palsu yang **semirip mungkin** dengan jaringan asli. Semakin mirip, semakin besar kemungkinan korban tertipu.

#### 📡 Tahap 2 — AP Impersonation (Peniruan Access Point)

Setelah informasi cukup, penyerang mulai membuat **access point palsu**.

Contoh:

```
Fake AP: SSID = "CoffeeShop_WiFi" (sama persis), Signal lebih kuat dari aslinya
```

Perhatikan dua hal penting:

- **SSID dibuat identik** → agar korban tidak curiga
- **Sinyal dibuat lebih kuat** → agar perangkat lebih memilih jaringan palsu

Sebagian besar perangkat (HP, laptop) punya perilaku default:

- Menghubungkan ke jaringan dengan sinyal paling kuat
- Atau jaringan yang pernah tersimpan sebelumnya

Di sinilah celahnya — penyerang memanfaatkan logika otomatis ini.

#### 🚫 Tahap 3 — Client Deauthentication (Opsional)

Tahap ini bersifat opsional, tapi sering digunakan untuk mempercepat proses.

Penyerang mengirimkan paket khusus bernama **deauthentication frame** untuk memutus koneksi korban dari Wi-Fi asli.

Contoh:

```
Tool: MDK4, aireplay-ng
Frame: 802.11 Deauthentication Frame → dikirim ke klien
```

Dampaknya:

- Korban tiba-tiba terputus dari Wi-Fi
- Perangkat akan mencoba reconnect secara otomatis
- Karena ada AP palsu dengan sinyal lebih kuat, perangkat bisa “salah pilih”

Tanpa tahap ini pun serangan tetap bisa berjalan, tapi biasanya lebih lambat karena menunggu korban pindah sendiri.

#### 🤝 Tahap 4 — Association & DHCP

Setelah korban terhubung ke AP palsu, masuk ke tahap berikutnya.

Di sini terjadi dua hal:

1. **Association** → perangkat korban resmi terhubung ke jaringan palsu
2. **DHCP assignment** → korban mendapatkan IP address

Contoh:

```
Penyerang menjalankan: dhcpd (DHCP daemon) → klien mendapat IP dari penyerang
```

Artinya:

- Bukan router asli yang memberi IP
- Tapi server milik penyerang

Pada titik ini, korban sudah “berada di dalam” jaringan penyerang.

#### 🌐 Tahap 5 — Traffic Handling (Penanganan Lalu Lintas)

Ini adalah inti dari serangan, di mana penyerang mulai benar-benar mendapatkan manfaat.

Ada dua pendekatan utama:

1. **Man-in-the-Middle (MitM)**
   Penyerang meneruskan koneksi ke internet sambil:
   - Mengamati data yang lewat
   - Mengubah data jika diperlukan

   Misalnya:
   - Melihat request HTTP
   - Menginject konten tertentu

2. **Captive Portal Palsu**
   Korban diarahkan ke halaman login palsu yang terlihat meyakinkan.

   Skenarionya:
   - Korban membuka browser
   - Muncul halaman “login Wi-Fi”
   - Diminta memasukkan password
   - Data langsung disimpan oleh penyerang

Karena tampilannya sering dibuat sangat mirip, banyak pengguna tidak menyadari bahwa itu jebakan.

#### 🗂️ Tahap 6 — Post Capture

Setelah data berhasil dikumpulkan, penyerang masuk ke tahap pemanfaatan.

Data yang biasanya didapat:

- Username & password
- Traffic jaringan
- WPA handshake

Selanjutnya bisa digunakan untuk:

- **Password cracking offline** menggunakan tool seperti Hashcat atau John the Ripper
- **Credential reuse** (menggunakan ulang password di layanan lain)
- Analisis lebih lanjut terhadap data korban

Tahap ini sering kali justru yang paling berbahaya, karena dampaknya bisa meluas ke banyak akun korban.

#### Catatan Keamanan

Walaupun serangan ini cukup efektif, ada beberapa teknologi yang bisa mengurangi risikonya:

- HTTPS (enkripsi komunikasi)
- HSTS (mencegah downgrade ke HTTP)
- Certificate pinning
- 802.1X (autentikasi enterprise)

Namun perlu diingat, faktor manusia tetap jadi titik lemah utama.
Misalnya:

- Mengklik sembarang halaman login
- Tidak mengecek keaslian jaringan

Karena itu, selain teknologi, **kesadaran pengguna** juga sangat penting dalam mencegah serangan seperti ini.

### c. Airgeddon — Tool Utama

#### Mengenal Airgeddon

Airgeddon adalah sebuah skrip berbasis Bash yang dirancang untuk melakukan audit keamanan jaringan Wi-Fi secara terintegrasi. Tool ini biasanya dijalankan di Kali Linux, karena sistem operasi tersebut sudah menyediakan banyak dependensi yang dibutuhkan untuk pengujian keamanan.

Hal yang membuat Airgeddon menarik adalah pendekatannya: bukan membuat tool baru dari nol, tetapi **menggabungkan berbagai tool populer** ke dalam satu alur otomatis.

Beberapa tool yang diintegrasikan antara lain:

- aircrack-ng → untuk analisis dan cracking Wi-Fi
- MDK4 → untuk serangan deauthentication
- dhcpd → untuk memberikan IP address ke klien
- curl → untuk komunikasi HTTP (misalnya captive portal)

Dengan integrasi ini, pengguna tidak perlu lagi menjalankan setiap tool secara terpisah.

#### Kelebihan Utama

Dari sisi penggunaan, Airgeddon menawarkan beberapa keunggulan praktis:

- **Otomatisasi penuh**
  Banyak langkah teknis yang biasanya kompleks disederhanakan menjadi menu interaktif.

- **Tidak perlu setup manual satu per satu**
  Tanpa Airgeddon, kamu harus mengatur airodump-ng, aireplay-ng, DHCP server, dan lain-lain secara manual. Di sini semuanya sudah dirangkai.

- **Pengecekan dependensi otomatis**
  Saat pertama dijalankan, Airgeddon akan mendeteksi apakah semua tool yang dibutuhkan sudah terpasang.

- **Instalasi tool tambahan langsung dari skrip**
  Jika ada yang belum terinstall, kamu akan ditawari untuk menginstallnya tanpa keluar dari program.

Dengan kata lain, Airgeddon bertindak seperti “orchestrator” yang mengatur semua komponen serangan dalam satu workflow yang rapi.

#### Cara Instalasi

Proses instalasinya cukup sederhana karena hanya perlu mengambil source code dari GitHub.

```bash
sudo git clone https://github.com/v1s1t0r1sh3r3/airgeddon
cd airgeddon
sudo bash airgeddon.sh
```

Penjelasan langkahnya:

- `git clone` → mengunduh repository Airgeddon ke komputer
- `cd airgeddon` → masuk ke folder hasil download
- `bash airgeddon.sh` → menjalankan skrip utama

Biasanya, skrip ini langsung bisa dijalankan tanpa proses build tambahan karena ditulis dalam Bash.

#### Apa yang Terjadi Saat Airgeddon Dijalankan

Ketika skrip dijalankan, Airgeddon akan memandu pengguna melalui beberapa tahap awal sebelum serangan dilakukan.

1. **Pengecekan dependensi**
   Skrip akan memeriksa apakah semua tool yang diperlukan sudah tersedia di sistem.

2. **Penawaran instalasi**
   Jika ada tool yang belum ada, kamu akan diminta konfirmasi untuk menginstallnya.

3. **Pemilihan network interface**
   Kamu harus memilih adapter Wi-Fi mana yang akan digunakan (terutama yang mendukung monitor mode).

4. **Menampilkan menu serangan**
   Setelah semua siap, Airgeddon akan menampilkan berbagai opsi serangan, termasuk Evil Twin Attack.

Dari sini, pengguna tinggal mengikuti menu interaktif yang tersedia, tanpa perlu mengingat perintah-perintah kompleks secara manual.

### d. Wireless Adapter yang Diperlukan

#### Kenapa Adapter Wi-Fi Itu Penting?

Dalam konteks Evil Twin Attack, **wireless adapter** bukan sekadar alat untuk konek internet seperti biasa. Adapter di sini berperan sebagai “sensor” sekaligus “pemancar” yang digunakan untuk:

- Mengamati semua aktivitas jaringan di udara
- Mengirim paket tertentu untuk memanipulasi koneksi korban

Masalahnya, **tidak semua adapter Wi-Fi punya kemampuan ini**. Kebanyakan adapter bawaan laptop hanya dirancang untuk penggunaan normal, bukan untuk kebutuhan analisis atau pengujian keamanan.

Karena itu, ada dua fitur utama yang wajib dimiliki.

#### Dua Fitur Wajib

##### Monitor Mode

Monitor mode memungkinkan adapter:

- Menangkap **semua paket Wi-Fi di udara**
- Tidak terbatas hanya pada jaringan yang sedang terhubung

Artinya, kamu bisa melihat:

- Jaringan di sekitar
- Perangkat yang terhubung
- Aktivitas komunikasi antar perangkat

Tanpa monitor mode, tahap awal seperti _reconnaissance_ (pengintaian) tidak bisa dilakukan dengan baik.

##### Packet Injection

Packet injection memungkinkan adapter:

- **Mengirim paket khusus ke jaringan**
- Termasuk paket seperti _deauthentication frame_

Contoh penggunaannya:

- Memutus koneksi korban dari Wi-Fi asli
- Mengganggu komunikasi jaringan
- Memicu korban untuk reconnect ke AP palsu

Tanpa fitur ini, kamu hanya bisa “mengamati”, tapi tidak bisa “mengintervensi”.

#### Rekomendasi Adapter

Berikut beberapa adapter yang umum digunakan karena kompatibilitasnya:

| Adapter                   | Keterangan                                                                      |
| ------------------------- | ------------------------------------------------------------------------------- |
| **Alfa** (berbagai model) | Sangat direkomendasikan karena stabil dan mendukung penuh kebutuhan audit Wi-Fi |
| **TP-Link TL-WN722N v1**  | Mendukung monitor mode & packet injection                                       |
| **Panda Wireless**        | Alternatif yang juga kompatibel                                                 |

Penjelasan tambahan:

- **Alfa**
  Biasanya jadi pilihan utama karena performanya stabil dan dukungan drivernya luas di Linux.

- **TP-Link TL-WN722N v1**
  Ini yang sering jadi “jebakan”. Hanya **versi 1** yang mendukung fitur penting.
  Versi 2, 3, dan 2C sudah menggunakan chipset berbeda yang **tidak mendukung monitor mode dan packet injection**.

- **Panda Wireless**
  Pilihan alternatif yang cukup populer dan kompatibel dengan banyak distro Linux.

#### Catatan Penting Saat Membeli

Kalau kamu memilih TP-Link TL-WN722N:

- **Pastikan benar-benar versi 1**
- Cek label fisik atau spesifikasi chipset sebelum membeli

Karena secara tampilan, semua versinya hampir sama, tapi kemampuan teknisnya sangat berbeda.

#### Kesimpulan Praktis

Singkatnya:

- Adapter menentukan apakah serangan bisa dilakukan atau tidak
- Minimal harus mendukung **monitor mode** dan **packet injection**
- Salah memilih adapter = banyak fitur tidak akan berjalan

Jadi sebelum masuk ke tahap praktik, memastikan hardware yang tepat itu adalah langkah paling fundamental.

### e. Langkah Operasional di Airgeddon (Slideshow Demo)

#### Gambaran Umum Proses

Pada bagian ini, instruktur menjelaskan bagaimana menjalankan serangan Evil Twin menggunakan Airgeddon. Karena tidak tersedia adapter Wi-Fi yang mendukung, penjelasan dilakukan secara konseptual melalui slideshow.

Walaupun begitu, alurnya tetap penting dipahami karena mencerminkan proses nyata saat tool dijalankan.

#### Langkah-Langkah Operasional

Mari kita bahas alurnya satu per satu agar jelas apa yang sebenarnya terjadi di balik setiap pilihan menu.

##### 1. Memilih Mode Serangan

Langkah pertama adalah memilih opsi:

```
Evil Twin AP attack with sniffing and SSL strip
```

Mode ini menggabungkan beberapa teknik sekaligus:

- Membuat access point palsu (Evil Twin)
- Melakukan sniffing (mengamati traffic)
- Mencoba SSL stripping (menurunkan HTTPS ke HTTP)

Artinya, ini adalah mode yang cukup “lengkap” untuk skenario MitM.

##### 2. Scanning Jaringan dengan airodump-ng

Setelah memilih mode, Airgeddon akan menjalankan proses scanning menggunakan **airodump-ng**.

Fungsinya:

- Menampilkan daftar Wi-Fi di sekitar
- Menunjukkan detail seperti SSID, channel, dan klien yang terhubung

Di tahap ini, kamu hanya perlu mengamati dan menentukan target.

##### 3. Menghentikan Scan

Ketika target sudah ditemukan:

```
Tekan Ctrl + C
```

Ini akan menghentikan proses scanning dan menampilkan daftar jaringan yang bisa dipilih.

##### 4. Memilih Target

Dari daftar yang muncul, pilih nomor jaringan target.

Di sinilah kamu menentukan:

- Wi-Fi mana yang akan ditiru
- Target utama dari serangan

##### 5. Memilih Metode Deauthentication

Selanjutnya, Airgeddon akan meminta metode deauthentication.

Biasanya:

- Pilih opsi 1 → metode standar

Fungsi tahap ini:

- Memutus koneksi korban dari Wi-Fi asli
- Memaksa mereka reconnect

Dengan begitu, peluang korban terhubung ke Evil Twin jadi lebih besar.

##### 6. MAC Address Spoofing

Airgeddon menawarkan opsi untuk melakukan **MAC spoofing**.

Tujuannya:

- Menyembunyikan identitas perangkat penyerang
- Menghindari deteksi dari log router

Disarankan untuk mengaktifkan opsi ini agar aktivitas lebih “tidak terlihat”.

##### 7. Handshake (Jika Belum Ada)

Jika ditanya apakah sudah punya file handshake:

- Pilih **No** jika belum

Artinya:

- Airgeddon akan mencoba menangkap handshake selama proses berlangsung

Ini berguna untuk serangan lanjutan seperti password cracking.

##### 8. Timeout Default

Airgeddon akan meminta nilai timeout.

Biasanya cukup:

```
Tekan Enter → gunakan default (20 detik)
```

Nilai ini mengatur durasi tertentu dalam proses otomatis (misalnya interval serangan).

##### 9. Menentukan Lokasi Penyimpanan

Selanjutnya, tentukan path untuk menyimpan file:

- Handshake
- Log hasil serangan

Jika tidak ingin repot:

- Gunakan path default saja

##### 10. Memilih Bahasa Captive Portal

Kamu bisa memilih bahasa untuk halaman captive portal.

Contoh:

- English

Tujuannya:

- Menyesuaikan dengan target agar terlihat lebih meyakinkan

Semakin natural tampilannya, semakin besar kemungkinan korban tertipu.

##### 11. Popup Window Otomatis

Setelah semua dikonfigurasi:

- Beberapa window akan terbuka otomatis

Ini biasanya berisi:

- Terminal untuk deauthentication
- Sniffing traffic
- DHCP server
- Captive portal

Penting:

- **Jangan menutup window ini**, karena masing-masing menjalankan bagian penting dari serangan.

#### Apa yang Dilihat oleh Korban

Dari sisi korban, pengalaman yang terjadi terlihat seperti ini:

- Muncul **dua jaringan dengan nama sama**
  Satu adalah jaringan asli, satu lagi adalah Evil Twin

- Salah satu jaringan biasanya:
  - Tidak memakai password
  - Atau terlihat lebih “mudah diakses”

- Setelah terhubung:
  - Tidak bisa akses internet seperti biasa
  - Browser akan diarahkan ke halaman login (captive portal)

- Korban diminta memasukkan password Wi-Fi
  Dan di sinilah masalah terjadi:
  - Data yang dimasukkan akan **tertangkap oleh Airgeddon**
  - Biasanya dalam bentuk **plaintext (tidak terenkripsi)**

#### Inti yang Perlu Dipahami

Kalau diringkas, langkah-langkah ini sebenarnya adalah orkestrasi dari beberapa proses:

- Scan jaringan
- Pilih target
- Putus koneksi korban
- Buat AP palsu
- Jalankan captive portal
- Tangkap data korban

Semua proses kompleks ini disederhanakan oleh Airgeddon menjadi alur menu yang mudah diikuti, sehingga bahkan pengguna yang belum terlalu berpengalaman pun bisa menjalankannya dengan panduan yang jelas.

## 3. Hubungan Antar Konsep

```
[Penyerang + Airgeddon]
       │
       ▼
[1. Recon] ──scan──► Temukan target SSID & channel
       │
       ▼
[2. AP Impersonation] ──buat──► Fake AP dengan SSID sama + sinyal kuat
       │
       ▼
[3. Deauthentication] ──kirim──► Frame deauth ke klien di AP asli
       │
       ▼
[4. Association + DHCP] ──klien konek ke──► Evil Twin, dapat IP dari penyerang
       │
       ▼
[5. Traffic Handling / Captive Portal]
       ├──► MitM: inspect & manipulasi traffic
       └──► Captive Portal: kumpulkan password/kredensial
       │
       ▼
[6. Post Capture]
       ├──► WPA Handshake → Password Cracking offline
       └──► Credential Reuse
```

Seluruh alur ini dimungkinkan oleh kombinasi antara:

- **Kelemahan protokol Wi-Fi** (klien memilih AP berdasarkan sinyal terkuat).
- **Tool otomatis** (Airgeddon mengintegrasikan dhcpd, MDK4, aircrack-ng, dll.).
- **Rekayasa sosial pasif** (captive portal yang menyerupai halaman login asli).

---

## 4. Kesimpulan

Evil Twin Attack adalah serangan _wireless MitM_ yang efektif karena memanfaatkan kelemahan alami protokol Wi-Fi — perangkat klien cenderung otomatis terhubung ke AP dengan sinyal terkuat. Dengan tool seperti **Airgeddon** di Kali Linux, proses serangan yang melibatkan banyak alat (airodump-ng, dhcpd, MDK4, captive portal) dapat diotomatisasi sepenuhnya.

Kunci kesuksesan serangan ini adalah:

1. Adapter Wi-Fi yang mendukung **monitor mode** dan **packet injection**.
2. AP palsu dengan SSID identik dan sinyal lebih kuat.
3. Deauthentication untuk mempercepat perpindahan klien.
4. Captive portal yang meyakinkan untuk mencuri kredensial.

Dari sisi **pertahanan (defender)**, serangan ini dapat dideteksi melalui monitoring anomali SSID ganda di jaringan, penggunaan protokol **802.1X/EAP** untuk autentikasi enterprise, dan kebijakan ketat **HTTPS + HSTS + certificate pinning** agar SSL stripping tidak efektif.
