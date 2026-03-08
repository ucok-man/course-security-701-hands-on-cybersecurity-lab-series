# 📝 Catatan Materi: PuTTY – SSH Client & Terminal Emulator

## 1. Ringkasan Singkat

Video ini memperkenalkan **PuTTY**, sebuah aplikasi ringan (_lightweight_) yang berfungsi sebagai SSH client, Telnet client, dan terminal emulator, yang sangat populer di lingkungan Windows. Demonstrasi dilakukan dengan menghubungkan Windows host ke sebuah server SSH yang berjalan di dalam virtual machine Kali Linux. Topik yang dibahas meliputi: pengunduhan dan instalasi PuTTY, cara membuat koneksi SSH, konsep _host key fingerprint_, serta gambaran umum fitur transfer file (PSCP/PSFTP) dan _port forwarding_.

---

## 2. Konsep Utama

### a. Apa Itu PuTTY?

#### Pengertian PuTTY

PuTTY adalah sebuah **terminal emulator** sekaligus **SSH/Telnet client** yang umumnya digunakan pada sistem operasi Windows. Aplikasi ini memungkinkan pengguna untuk membuka sesi terminal dan berkomunikasi dengan perangkat lain melalui jaringan.

Secara sederhana, PuTTY menyediakan **antarmuka berbasis teks (text-based interface)** yang memungkinkan pengguna menjalankan perintah langsung pada **command-line interface (CLI)** dari perangkat lain, seperti server Linux, router, atau switch jaringan.

Ketika seseorang menggunakan PuTTY, mereka sebenarnya sedang membuka sebuah **sesi terminal jarak jauh** ke perangkat lain. Semua perintah yang diketik di jendela PuTTY akan dikirim melalui jaringan ke perangkat tujuan, lalu hasilnya dikirim kembali dan ditampilkan di layar.

Hal ini sangat berguna dalam administrasi sistem, karena banyak server dan perangkat jaringan dikonfigurasi menggunakan **CLI**, bukan antarmuka grafis.

#### PuTTY Bukan Alat Konfigurasi

Hal penting yang perlu dipahami adalah bahwa PuTTY **bukan alat konfigurasi perangkat**.

PuTTY hanya berperan sebagai **klien (client)** yang menjadi perantara antara pengguna dan perangkat yang ingin dikendalikan.

Artinya:

- PuTTY **tidak mengubah konfigurasi apa pun secara langsung**
- PuTTY hanya **membuka koneksi** ke perangkat tujuan
- Semua konfigurasi tetap dilakukan melalui **perintah CLI pada perangkat tersebut**

Sebagai contoh:

Jika seorang administrator ingin mengkonfigurasi server Linux, mereka dapat membuka PuTTY lalu melakukan koneksi SSH ke server tersebut. Setelah login berhasil, barulah administrator dapat menjalankan berbagai perintah seperti:

```
sudo apt update
sudo systemctl restart nginx
```

Perintah-perintah tersebut sebenarnya **dijalankan di server**, bukan di PuTTY. PuTTY hanya menampilkan input dan output dari sesi tersebut.

#### Protokol yang Didukung PuTTY

PuTTY dapat digunakan untuk terhubung ke perangkat menggunakan beberapa protokol komunikasi jaringan. Protokol-protokol ini menentukan bagaimana data dikirim antara komputer pengguna dan perangkat tujuan.

Beberapa protokol yang umum digunakan di PuTTY antara lain:

**1. SSH (Secure Shell)**
SSH adalah protokol yang digunakan untuk mengakses perangkat secara **aman dan terenkripsi** melalui jaringan.

Semua data yang dikirim—termasuk username, password, dan perintah—akan dienkripsi sehingga tidak mudah disadap oleh pihak lain.

Contoh penggunaan:

- Mengakses server Linux di cloud
- Mengelola VPS
- Administrasi sistem jarak jauh

Karena tingkat keamanannya tinggi, SSH menjadi **standar utama dalam administrasi server modern**.

**2. Telnet**
Telnet adalah protokol yang lebih lama yang juga digunakan untuk mengakses CLI perangkat jaringan.

Namun, berbeda dengan SSH, Telnet **tidak menggunakan enkripsi**. Semua data dikirim dalam bentuk teks biasa (_plain text_). Akibatnya, informasi seperti username dan password dapat dengan mudah disadap oleh pihak yang berada di jaringan yang sama.

Karena alasan keamanan ini, Telnet saat ini **sudah jarang digunakan di lingkungan produksi** dan biasanya hanya dipakai untuk:

- simulasi jaringan
- pembelajaran
- perangkat lama yang belum mendukung SSH

**3. Serial Connection**
Selain koneksi jaringan, PuTTY juga dapat digunakan untuk koneksi **serial**.

Koneksi serial biasanya digunakan untuk mengakses perangkat jaringan secara langsung menggunakan **kabel console**. Metode ini sering dipakai ketika:

- perangkat jaringan baru pertama kali dikonfigurasi
- perangkat belum memiliki alamat IP
- akses jaringan tidak tersedia

Contohnya adalah ketika seorang teknisi menghubungkan laptop ke router Cisco menggunakan kabel console. PuTTY kemudian digunakan untuk membuka sesi terminal melalui port serial tersebut.

#### Posisi PuTTY dalam Arsitektur Jaringan

Untuk memahami bagaimana PuTTY bekerja dalam jaringan, kita bisa melihat ilustrasi berikut:

```
[Windows Host]
     |
     | (SSH / Telnet / Serial)
     |
  [PuTTY]
     |
     v
[Server / Router / Switch / Firewall]
```

Penjelasan alurnya sebagai berikut:

1. Pengguna menjalankan PuTTY di komputer Windows mereka.
2. PuTTY membuat koneksi menggunakan protokol tertentu, misalnya **SSH**.
3. Koneksi tersebut dikirim melalui jaringan menuju perangkat tujuan.
4. Perangkat tujuan (server, router, atau switch) menerima koneksi tersebut dan membuka sesi CLI.
5. Semua perintah yang diketik oleh pengguna akan diproses oleh perangkat tersebut dan hasilnya dikirim kembali ke PuTTY untuk ditampilkan.

Dengan cara kerja ini, administrator dapat mengelola sistem yang berada **di lokasi yang berbeda**, bahkan di negara yang berbeda, seolah-olah mereka sedang bekerja langsung di mesin tersebut.

### b. Kegunaan PuTTY di Dunia Nyata

PuTTY bukan hanya sekadar aplikasi terminal biasa. Dalam praktik sehari-hari, terutama di bidang **administrasi server** dan **jaringan komputer**, PuTTY menjadi salah satu alat yang sangat sering digunakan.

Secara umum, penggunaan PuTTY dapat dibagi menjadi dua skenario utama:

1. Mengakses server secara jarak jauh melalui jaringan.
2. Mengonfigurasi perangkat jaringan secara langsung melalui koneksi konsol.

Kedua skenario ini memiliki cara kerja yang berbeda, tetapi sama-sama memanfaatkan kemampuan PuTTY sebagai **terminal client**.

#### 1. Koneksi SSH/Telnet ke Server

Salah satu kegunaan paling umum dari PuTTY adalah untuk membuat koneksi **SSH atau Telnet** ke server yang berada di jaringan lain.

Dalam banyak kasus, server Linux atau Unix tidak memiliki antarmuka grafis (GUI) atau tidak diakses secara langsung melalui monitor dan keyboard. Sebagai gantinya, administrator mengelola server tersebut melalui **command-line interface (CLI)** dari jarak jauh.

Di sinilah PuTTY berperan.

Dengan menggunakan PuTTY, pengguna Windows dapat membuka koneksi ke server dan menjalankan perintah seolah-olah mereka sedang berada langsung di mesin tersebut.

Contoh skenario sederhana:

Misalnya ada sebuah server Linux dengan alamat IP:

```
192.168.1.10
```

Administrator dapat membuka PuTTY lalu melakukan koneksi menggunakan protokol SSH.

Setelah koneksi berhasil dibuat, server biasanya akan meminta **username dan password** untuk proses login.

Contoh tampilan login:

```
login as: admin
admin@192.168.1.10's password:
```

Setelah berhasil login, pengguna akan mendapatkan akses ke terminal server.

Contoh perintah yang bisa dijalankan:

```
ls
```

Perintah ini digunakan untuk melihat daftar file di direktori saat ini.

Contoh output:

```
backup
config
logs
nginx.conf
```

Contoh perintah lain:

```
sudo systemctl restart nginx
```

Perintah ini digunakan untuk **me-restart service Nginx** di server.

Dalam skenario ini:

- PuTTY hanya menjadi **jembatan koneksi**
- Semua perintah sebenarnya **dieksekusi di server**

Dengan cara ini, administrator dapat mengelola server dari mana saja tanpa harus datang langsung ke lokasi server tersebut.

Inilah alasan mengapa SSH menjadi **metode standar dalam administrasi server modern**, terutama untuk VPS, cloud server, dan infrastruktur data center.

#### 2. Konfigurasi Perangkat Jaringan (Networking)

Selain untuk server, PuTTY juga sangat umum digunakan oleh **network administrator** untuk mengonfigurasi perangkat jaringan seperti:

- router
- switch
- firewall

Berbeda dengan koneksi SSH yang dilakukan melalui jaringan, konfigurasi awal perangkat jaringan sering dilakukan menggunakan **console port**.

Console port adalah port khusus pada perangkat jaringan yang digunakan untuk **akses langsung ke CLI perangkat**, biasanya melalui koneksi serial.

Untuk melakukan koneksi ini, diperlukan:

- **kabel console (rollover cable)**
- **port serial atau USB-to-serial adapter**
- aplikasi terminal seperti **PuTTY**

Di PuTTY, koneksi ini menggunakan **mode Serial**, bukan SSH.

Langkah umum yang dilakukan biasanya seperti ini:

1. Laptop dihubungkan ke perangkat jaringan menggunakan kabel console.
2. PuTTY dibuka dan mode koneksi diatur menjadi **Serial**.
3. PuTTY terhubung ke port serial perangkat.
4. CLI perangkat jaringan langsung muncul di terminal.

Metode ini sangat penting terutama dalam kondisi berikut:

**Konfigurasi awal perangkat (Initial Configuration)**
Ketika perangkat jaringan baru pertama kali dipasang, biasanya perangkat tersebut belum memiliki alamat IP. Karena belum ada IP, perangkat tidak bisa diakses melalui SSH atau Telnet.

Satu-satunya cara mengaksesnya adalah melalui **console port**.

Melalui koneksi ini, administrator dapat melakukan konfigurasi awal seperti:

- mengatur hostname
- mengatur IP address
- mengaktifkan SSH
- mengatur password administrator

**Recovery perangkat**
Jika terjadi kesalahan konfigurasi atau masalah jaringan, administrator mungkin kehilangan akses melalui SSH.

Dalam kondisi seperti ini, koneksi console menjadi **cara darurat** untuk masuk ke perangkat dan memperbaiki konfigurasi.

**Konektivitas jaringan belum tersedia**
Pada tahap instalasi awal jaringan, perangkat mungkin belum terhubung dengan jaringan yang aktif. Oleh karena itu, konfigurasi harus dilakukan melalui koneksi console terlebih dahulu.

#### Contoh Skenario Jaringan

Berikut gambaran sederhana bagaimana PuTTY digunakan untuk mengakses CLI switch melalui koneksi console:

```
[Laptop]
  |-- (Kabel Console/Rollover) --> [Console Port Switch]
  |
  PuTTY (Mode: Serial)
  --> Akses CLI Switch untuk konfigurasi awal
```

Alurnya dapat dijelaskan sebagai berikut:

1. Laptop administrator terhubung ke switch menggunakan kabel console.
2. PuTTY dijalankan dengan pengaturan koneksi **Serial**.
3. PuTTY membuka komunikasi melalui port serial.
4. CLI dari switch langsung muncul di terminal.
5. Administrator dapat mulai mengetik perintah konfigurasi.

Contoh tampilan CLI pada switch:

```
Switch>
```

Administrator kemudian dapat masuk ke mode konfigurasi:

```
enable
configure terminal
```

Dengan metode ini, perangkat jaringan dapat dikonfigurasi bahkan **sebelum jaringan benar-benar aktif**.

Karena fleksibilitas inilah PuTTY menjadi salah satu tools paling penting bagi **system administrator dan network engineer**, terutama ketika bekerja dengan server Linux dan perangkat jaringan profesional.

### c. Cara Mengunduh dan Menginstal PuTTY

#### Mengunduh PuTTY dari Website Resmi

Langkah pertama untuk menggunakan PuTTY adalah mengunduh aplikasinya dari sumber resmi. Situs resmi PuTTY adalah **putty.org**, yang menyediakan tautan unduhan untuk berbagai versi sistem operasi.

Mengunduh dari situs resmi sangat penting karena memastikan bahwa file yang didapatkan:

- aman dari malware
- merupakan versi terbaru
- kompatibel dengan sistem operasi yang digunakan

Biasanya, halaman unduhan akan mengarahkan pengguna ke **mirror download** yang berisi berbagai file installer PuTTY.

#### Memilih Versi yang Sesuai dengan Arsitektur Sistem

Setelah masuk ke halaman unduhan, pengguna akan melihat beberapa pilihan file instalasi. Pilihan ini berkaitan dengan **arsitektur CPU** dari sistem operasi yang digunakan.

Memilih versi yang tepat penting agar aplikasi dapat berjalan dengan optimal.

Berikut penjelasan dari beberapa jenis arsitektur yang biasanya tersedia:

**64-bit (x86-64)**
Ini adalah versi yang paling umum digunakan saat ini. Hampir semua komputer modern menggunakan arsitektur **64-bit**.

Jika Anda menggunakan Windows 10, Windows 11, atau versi Windows modern lainnya, kemungkinan besar sistem Anda menggunakan arsitektur ini.

Versi ini biasanya memiliki nama seperti:

```
putty-64bit-installer.msi
```

**ARM**
Versi ini ditujukan untuk perangkat yang menggunakan **prosesor berbasis ARM**.

Arsitektur ARM biasanya ditemukan pada:

- beberapa laptop modern berbasis ARM
- perangkat embedded
- sistem tertentu seperti Windows on ARM

Jika komputer Anda menggunakan CPU ARM, maka Anda harus memilih installer yang secara khusus dibuat untuk arsitektur ini.

**32-bit**
Versi 32-bit disediakan untuk sistem operasi lama yang masih menggunakan arsitektur **x86 32-bit**.

Namun, saat ini versi ini sudah jarang digunakan karena sebagian besar komputer modern sudah menggunakan sistem 64-bit.

Biasanya file installer terlihat seperti:

```
putty-installer.msi
```

Jika ragu, memilih **versi 64-bit** biasanya merupakan pilihan yang benar untuk mayoritas komputer modern.

#### Proses Instalasi PuTTY

Setelah file installer berhasil diunduh, langkah berikutnya adalah menjalankan proses instalasi. Instalasi PuTTY tergolong sangat sederhana dan mengikuti pola instalasi standar pada Windows.

Langkah-langkahnya biasanya sebagai berikut.

Pertama, jalankan file installer yang telah diunduh. Windows akan membuka jendela instalasi PuTTY.

Selanjutnya pengguna akan diminta untuk menentukan **direktori instalasi**, yaitu lokasi di mana PuTTY akan dipasang di sistem.

Contohnya:

```
C:\Program Files\PuTTY\
```

Jika tidak ada kebutuhan khusus, biasanya direktori default ini sudah cukup.

Setelah itu proses instalasi dapat dilanjutkan dengan menekan tombol **Next** hingga masuk ke tahap pemasangan program.

Secara umum alurnya terlihat seperti berikut:

1. Klik **Next** pada halaman awal installer.
2. Pilih direktori instalasi.
3. Klik **Next** untuk melanjutkan.
4. Klik **Install** untuk memulai proses pemasangan.
5. Tunggu hingga proses selesai.
6. Klik **Finish** untuk menutup installer.

Setelah proses instalasi selesai, PuTTY akan tersedia di sistem dan bisa dijalankan melalui:

- **Start Menu**
- **Desktop Shortcut** (jika dibuat saat instalasi)
- pencarian Windows dengan mengetik **PuTTY**

Ketika pertama kali dijalankan, PuTTY akan menampilkan **jendela konfigurasi koneksi**, di mana pengguna dapat menentukan:

- alamat IP atau hostname server
- jenis protokol koneksi (SSH, Telnet, atau Serial)
- port yang digunakan

Dari jendela inilah pengguna biasanya memulai sesi koneksi ke server atau perangkat jaringan.

### d. Antarmuka PuTTY (GUI Overview)

#### Gambaran Umum Antarmuka PuTTY

Ketika PuTTY pertama kali dibuka, pengguna tidak langsung melihat terminal seperti pada aplikasi SSH lain. Sebaliknya, PuTTY akan menampilkan **jendela konfigurasi** terlebih dahulu.

Jendela ini digunakan untuk mengatur berbagai parameter koneksi sebelum sesi terminal dibuka. Melalui konfigurasi ini, pengguna dapat menentukan:

- alamat server tujuan
- jenis protokol yang digunakan
- port komunikasi
- berbagai pengaturan terminal lainnya

Desain antarmuka PuTTY cukup sederhana dan berbasis **panel pengaturan di sisi kiri** serta **form konfigurasi di sisi kanan**. Struktur ini memudahkan pengguna untuk menavigasi berbagai opsi yang tersedia.

Beberapa bagian utama yang biasanya terlihat pada jendela konfigurasi PuTTY antara lain:

| Bagian         | Fungsi                                                                           |
| -------------- | -------------------------------------------------------------------------------- |
| **Session**    | Digunakan untuk memasukkan alamat server, port, dan memilih tipe koneksi         |
| **Terminal**   | Mengatur tampilan terminal seperti ukuran layar, karakter, dan perilaku keyboard |
| **Connection** | Mengatur parameter koneksi yang lebih detail seperti SSH, Telnet, atau Serial    |

Setiap bagian ini memiliki fungsi yang berbeda dan dapat disesuaikan sesuai kebutuhan koneksi yang ingin dibuat.

#### Menu Session

Menu **Session** adalah bagian yang paling sering digunakan karena di sinilah pengguna menentukan tujuan koneksi.

Di dalam menu ini, terdapat beberapa field penting yang harus diisi sebelum membuka koneksi, seperti:

- **Host Name (or IP Address)**
- **Port**
- **Connection Type**

Biasanya, administrator server hanya perlu mengatur tiga parameter ini untuk memulai koneksi SSH.

#### Menu Terminal

Menu **Terminal** berfungsi untuk mengatur bagaimana terminal akan ditampilkan ketika koneksi berhasil dibuat.

Beberapa pengaturan yang dapat dilakukan di sini meliputi:

- ukuran buffer terminal
- perilaku keyboard
- karakter encoding
- pengaturan tampilan teks

Walaupun tersedia banyak opsi, sebagian besar pengguna jarang mengubah pengaturan ini karena konfigurasi default PuTTY biasanya sudah cukup nyaman digunakan.

#### Menu Connection

Menu **Connection** menyediakan pengaturan yang lebih teknis terkait dengan metode komunikasi antara PuTTY dan perangkat tujuan.

Di dalam menu ini terdapat berbagai sub-menu seperti:

- **SSH**
- **Telnet**
- **Serial**
- **Proxy**

Setiap sub-menu memungkinkan pengguna untuk menyesuaikan parameter spesifik dari protokol yang digunakan. Misalnya, pada pengaturan SSH pengguna dapat mengatur:

- metode autentikasi
- keep-alive connection
- konfigurasi enkripsi

Namun, dalam penggunaan dasar, pengaturan ini biasanya tidak perlu diubah.

#### Pengaturan Dasar untuk Koneksi SSH

Untuk membuat koneksi SSH sederhana menggunakan PuTTY, pengguna hanya perlu mengisi beberapa parameter dasar pada menu **Session**.

Berikut pengaturan yang umum digunakan:

**Host Name / IP Address**
Field ini digunakan untuk memasukkan alamat server yang ingin diakses.

Contoh:

```
192.168.1.10
```

Alamat ini bisa berupa:

- **IP Address** (misalnya `192.168.1.10`)
- **Hostname atau domain** (misalnya `server.example.com`)

**Port**
Port menentukan jalur komunikasi yang digunakan oleh protokol jaringan.

Untuk SSH, port default yang digunakan adalah:

```
22
```

Namun pada beberapa server, administrator mungkin mengubah port ini demi alasan keamanan.

**Connection Type**
Bagian ini menentukan jenis protokol yang akan digunakan untuk membuat koneksi.

Beberapa pilihan yang tersedia biasanya adalah:

- SSH
- Telnet
- Serial
- Raw

Untuk koneksi server yang aman, biasanya dipilih:

```
SSH
```

Setelah semua parameter diisi, pengguna cukup menekan tombol **Open** untuk memulai koneksi.

#### Contoh Tampilan Konfigurasi

Berikut contoh sederhana tampilan konfigurasi PuTTY ketika akan membuat koneksi SSH:

```
+--------------------------+
| PuTTY Configuration      |
|--------------------------|
| Host Name: 192.168.1.10  |
| Port:       22           |
| Connection Type: (o) SSH |
|                          |
|       [Open]             |
+--------------------------+
```

Penjelasan alurnya:

1. Pengguna memasukkan alamat server pada bagian **Host Name**.
2. Port SSH diatur ke **22**.
3. Jenis koneksi dipilih **SSH**.
4. Tombol **Open** ditekan untuk membuka sesi terminal.

Jika koneksi berhasil, PuTTY akan membuka **jendela terminal baru** yang menampilkan proses autentikasi login ke server. Setelah login berhasil, pengguna dapat mulai menjalankan berbagai perintah CLI pada server tersebut.

### e. Proses Koneksi SSH dengan PuTTY

#### Gambaran Umum Proses Koneksi SSH

SSH (Secure Shell) adalah protokol yang digunakan untuk mengakses sistem lain secara **aman melalui jaringan**. Dengan SSH, pengguna dapat masuk ke server jarak jauh dan menjalankan perintah melalui **command-line interface (CLI)**.

PuTTY berfungsi sebagai **klien SSH**, yaitu aplikasi yang digunakan dari komputer pengguna untuk membuat koneksi ke server.

Ketika koneksi SSH berhasil dibuat, pengguna akan mendapatkan **terminal jarak jauh** yang memungkinkan mereka mengelola server seolah-olah sedang berada langsung di mesin tersebut.

Berikut adalah langkah-langkah umum untuk membuat koneksi SSH menggunakan PuTTY.

#### 1. Membuka Aplikasi PuTTY

Langkah pertama adalah menjalankan aplikasi PuTTY yang sudah terinstal di komputer.

Ketika PuTTY dibuka, akan muncul jendela **PuTTY Configuration**. Jendela ini digunakan untuk mengatur parameter koneksi sebelum terminal dibuka.

Di sinilah pengguna menentukan alamat server, port koneksi, serta jenis protokol yang digunakan.

#### 2. Memasukkan IP Address Server Tujuan

Selanjutnya, pengguna harus memasukkan alamat server yang ingin diakses.

Alamat ini dimasukkan pada field **Host Name (or IP Address)**.

Contoh:

```id="3pqjt3"
192.168.1.10
```

Alamat tersebut dapat berupa:

- **IP address** (misalnya `192.168.1.10`)
- **hostname atau domain** (misalnya `server.example.com`)

PuTTY akan menggunakan alamat ini untuk mengetahui **server mana yang akan dihubungi** melalui jaringan.

#### 3. Memastikan Port dan Tipe Koneksi

Setelah alamat server dimasukkan, langkah berikutnya adalah memastikan pengaturan koneksi sudah benar.

Untuk SSH, pengaturan yang umum digunakan adalah:

```id="bplkh1"
Port: 22
Connection type: SSH
```

Penjelasan singkatnya:

**Port 22**
Port adalah jalur komunikasi yang digunakan oleh protokol jaringan. Secara default, SSH menggunakan port **22**.

Namun dalam beberapa konfigurasi server, port ini bisa saja diubah oleh administrator untuk meningkatkan keamanan.

**Connection Type: SSH**
Pengaturan ini memastikan bahwa PuTTY akan menggunakan protokol **Secure Shell** untuk membuat koneksi yang terenkripsi.

Jika semua pengaturan sudah benar, pengguna dapat melanjutkan dengan menekan tombol **Open**.

#### 4. Membuka Koneksi

Setelah tombol **Open** ditekan, PuTTY akan mencoba membangun koneksi ke server.

Jika alamat IP benar dan server sedang aktif, PuTTY akan membuka jendela terminal baru dan mulai melakukan proses **handshake SSH**.

Proses ini melibatkan pertukaran informasi keamanan antara klien (PuTTY) dan server.

#### 5. Peringatan Host Key Fingerprint

Jika ini adalah **pertama kalinya** komputer tersebut terhubung ke server tertentu, PuTTY akan menampilkan pesan keamanan yang berisi **host key fingerprint**.

Pesan ini biasanya terlihat seperti peringatan yang menanyakan apakah pengguna mempercayai identitas server tersebut.

Tujuan dari host key fingerprint adalah untuk memastikan bahwa pengguna benar-benar terhubung ke server yang tepat dan bukan ke server palsu.

Contoh pesan yang mungkin muncul:

```
The server's host key is not cached in the registry.
You have no guarantee that the server is the computer you think it is.
```

Jika pengguna yakin bahwa server tersebut benar, maka langkah yang dilakukan adalah menekan tombol **Accept**.

Setelah diterima, fingerprint server akan disimpan sehingga pada koneksi berikutnya pesan ini tidak muncul lagi.

#### 6. Memasukkan Username dan Password

Setelah proses verifikasi host key selesai, PuTTY akan menampilkan prompt login.

Contohnya seperti berikut:

```
login as:
```

Pengguna harus memasukkan **username** yang valid di server tersebut.

Contoh:

```id="y2g95s"
login as: admin
```

Setelah username dimasukkan, server akan meminta **password**.

```
admin@192.168.1.10's password:
```

Saat mengetik password di terminal SSH, biasanya **tidak ada karakter yang muncul** di layar (bahkan tanda bintang sekalipun). Ini adalah perilaku normal yang bertujuan untuk meningkatkan keamanan.

Jika username dan password benar, pengguna akan berhasil masuk ke sistem.

#### 7. Terminal Server Siap Digunakan

Setelah proses login berhasil, terminal server akan muncul.

Biasanya akan terlihat prompt seperti ini:

```id="vsdbr7"
admin@server:~$
```

Prompt ini menandakan bahwa pengguna sudah berada di dalam **shell server** dan dapat mulai menjalankan berbagai perintah Linux.

Contoh perintah yang bisa dijalankan:

```id="gjdphb"
ls
```

Perintah ini akan menampilkan daftar file pada direktori saat ini.

Contoh output:

```id="de3n98"
backup
config
logs
www
```

Contoh perintah lain:

```id="puedwy"
pwd
```

Perintah ini akan menampilkan lokasi direktori saat ini.

Output:

```id="6hy9kq"
/home/admin
```

Setelah tahap ini, koneksi SSH sudah sepenuhnya aktif dan pengguna dapat melakukan berbagai aktivitas administrasi server seperti:

- mengelola file
- menginstal aplikasi
- mengonfigurasi layanan
- memonitor sistem

Semua perintah yang diketik di jendela PuTTY akan dikirim ke server dan hasilnya akan ditampilkan kembali di terminal.

### f. Host Key Fingerprint

#### Apa Itu Host Key Fingerprint

Ketika pertama kali mencoba terhubung ke sebuah server menggunakan SSH melalui PuTTY, biasanya akan muncul sebuah peringatan yang menyatakan bahwa **host key server belum tersimpan di cache**.

Hal ini sering membuat pengguna baru merasa bingung atau khawatir, padahal sebenarnya ini adalah **mekanisme keamanan normal dalam protokol SSH**.

Dalam sistem SSH, setiap server memiliki sebuah **host key**, yaitu kunci kriptografi yang berfungsi sebagai identitas unik server tersebut. Dari host key ini kemudian dihasilkan sebuah nilai ringkas yang disebut **fingerprint**.

Fingerprint ini dapat dianggap sebagai **identitas digital** dari server.

Contohnya, PuTTY mungkin akan menampilkan pesan seperti berikut ketika koneksi pertama kali dilakukan:

```
The server's host key is not cached in the registry.
You have no guarantee that the server is the computer you think it is.
```

Pesan ini bukan berarti ada masalah, tetapi hanya memberi tahu bahwa **komputer Anda belum pernah mengenali server tersebut sebelumnya**.

#### Tujuan Mekanisme Host Key

Fungsi utama dari host key fingerprint adalah untuk mencegah serangan yang dikenal sebagai **Man-in-the-Middle (MITM)**.

Dalam serangan MITM, seorang penyerang mencoba **menyamar sebagai server tujuan** dan mencegat komunikasi antara pengguna dan server asli.

Tanpa mekanisme verifikasi identitas server, pengguna bisa saja terhubung ke server palsu tanpa menyadarinya. Jika hal ini terjadi, penyerang berpotensi mencuri informasi penting seperti:

- username
- password
- perintah yang dikirim
- data sensitif lainnya

Dengan adanya host key fingerprint, klien SSH seperti PuTTY dapat **memverifikasi identitas server** sebelum koneksi dilanjutkan.

Jika fingerprint yang diterima berubah secara tiba-tiba pada koneksi berikutnya, itu bisa menjadi indikasi bahwa sesuatu yang tidak normal sedang terjadi.

#### Proses Verifikasi Fingerprint

Ketika koneksi SSH dilakukan untuk pertama kali, PuTTY akan menampilkan fingerprint dari host key server dan meminta pengguna untuk memutuskan apakah koneksi tersebut dipercaya atau tidak.

Contoh tampilan peringatan:

```
The server's host key is not cached in the registry.
If you trust this host, press Accept to add the key to PuTTY's cache.
```

Pada tahap ini, secara ideal pengguna harus **memverifikasi fingerprint server** dengan sumber yang terpercaya, misalnya:

- dokumentasi server
- administrator sistem
- panel manajemen server

Jika fingerprint yang ditampilkan sesuai dengan yang seharusnya, maka pengguna dapat menekan tombol **Accept**.

Ketika tombol **Accept** dipilih, PuTTY akan:

1. Menyimpan host key server ke dalam **cache lokal**.
2. Mengaitkan host key tersebut dengan alamat server yang digunakan.
3. Menggunakan informasi ini untuk memverifikasi koneksi di masa mendatang.

#### Perilaku Pada Koneksi Berikutnya

Setelah host key disimpan, PuTTY tidak akan lagi menampilkan peringatan tersebut ketika pengguna terhubung ke server yang sama.

Hal ini karena PuTTY sudah mengenali identitas server tersebut dan dapat memverifikasi bahwa server yang dihubungi masih menggunakan host key yang sama.

Namun, jika pada suatu saat fingerprint server **tiba-tiba berubah**, PuTTY akan kembali menampilkan peringatan keamanan.

Contoh pesan yang mungkin muncul:

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Perubahan fingerprint ini bisa terjadi karena beberapa alasan, misalnya:

- server baru saja diinstal ulang
- konfigurasi SSH server berubah
- host key diganti oleh administrator
- atau dalam skenario terburuk, ada potensi serangan MITM

Karena itu, perubahan fingerprint harus selalu diperiksa dengan hati-hati sebelum menerima koneksi baru.

#### Analogi Fingerprint

Cara mudah memahami konsep host key fingerprint adalah dengan menggunakan analogi **tanda tangan digital**.

Bayangkan Anda bertemu seseorang untuk pertama kali dalam urusan penting. Orang tersebut menunjukkan **tanda tangan resmi** yang membuktikan identitasnya.

Anda memeriksa tanda tangan tersebut dan memutuskan bahwa orang itu memang benar orang yang dimaksud.

Pada pertemuan berikutnya, Anda akan langsung mengenali tanda tangan yang sama tanpa perlu memverifikasi ulang.

Namun jika suatu hari tanda tangan orang tersebut tiba-tiba berubah, tentu Anda akan merasa curiga dan ingin memastikan kembali identitasnya.

Dalam konteks SSH, fingerprint server berfungsi persis seperti **tanda tangan digital** tersebut. Ia membantu memastikan bahwa Anda selalu terhubung ke server yang benar dan bukan ke pihak yang mencoba menyamar.

### g. Fitur Tambahan PuTTY (Disebutkan Singkat)

#### Gambaran Umum Fitur Tambahan PuTTY

Selain berfungsi sebagai klien SSH untuk membuka terminal jarak jauh, PuTTY juga menyediakan beberapa **alat tambahan** yang sangat berguna dalam administrasi server dan jaringan.

Alat-alat ini biasanya ikut terinstal bersama PuTTY dan dirancang untuk melengkapi fungsi utama SSH, seperti:

- mentransfer file secara aman
- membuka sesi file transfer interaktif
- membuat tunnel jaringan melalui SSH
- melakukan koneksi menggunakan protokol lain seperti Telnet

Walaupun fitur-fitur ini tidak selalu digunakan oleh pengguna pemula, mereka sangat penting dalam pekerjaan **system administrator**, **developer**, maupun **network engineer**.

Berikut beberapa fitur tambahan yang umum tersedia dalam paket PuTTY.

#### PSCP (PuTTY Secure Copy)

**PSCP** adalah utilitas berbasis **command line** yang digunakan untuk mentransfer file antara komputer lokal dan server melalui protokol SSH.

PSCP merupakan implementasi dari protokol **SCP (Secure Copy Protocol)** yang memungkinkan transfer file dilakukan secara **terenkripsi**. Artinya, data yang dikirim melalui jaringan tidak dapat dengan mudah disadap oleh pihak lain.

PSCP biasanya dijalankan melalui **Command Prompt** atau terminal di Windows.

Contoh penggunaan untuk mengirim file dari komputer lokal ke server:

```
pscp file.txt user@192.168.1.10:/home/user/
```

Penjelasan perintah tersebut:

- `pscp` → program yang digunakan
- `file.txt` → file lokal yang ingin dikirim
- `user@192.168.1.10` → username dan alamat server
- `/home/user/` → direktori tujuan di server

Jika perintah berhasil dijalankan, file `file.txt` akan tersalin ke direktori `/home/user/` pada server.

Contoh lain untuk mengunduh file dari server ke komputer lokal:

```
pscp user@192.168.1.10:/home/user/log.txt .
```

Tanda titik (`.`) di akhir berarti file akan disimpan di **direktori lokal saat ini**.

PSCP sangat berguna ketika administrator ingin **mentransfer file dengan cepat tanpa membuka sesi terminal interaktif**.

#### PSFTP (PuTTY SFTP Client)

Selain PSCP, PuTTY juga menyediakan **PSFTP**, yaitu klien **SFTP (SSH File Transfer Protocol)** berbasis command line.

Berbeda dengan PSCP yang biasanya digunakan untuk transfer file sekali jalan, PSFTP menyediakan **mode interaktif** yang mirip dengan FTP.

Dengan PSFTP, pengguna dapat:

- menelusuri direktori server
- mengunggah file
- mengunduh file
- membuat direktori baru
- menghapus file

Contoh memulai sesi PSFTP:

```
psftp user@192.168.1.10
```

Setelah login berhasil, pengguna akan masuk ke mode interaktif:

```
psftp>
```

Beberapa perintah yang bisa digunakan di dalam PSFTP antara lain:

```
ls
```

Perintah ini menampilkan daftar file di server.

```
get file.txt
```

Perintah ini mengunduh file dari server ke komputer lokal.

```
put file.txt
```

Perintah ini mengunggah file dari komputer lokal ke server.

Dengan fitur ini, pengguna dapat mengelola file server dengan cara yang lebih fleksibel dibandingkan PSCP.

#### Port Forwarding (SSH Tunneling)

Fitur **Port Forwarding** memungkinkan PuTTY membuat **tunnel SSH**, yaitu jalur komunikasi aman yang meneruskan trafik dari port lokal ke port tertentu di server.

Teknik ini sering digunakan untuk mengakses layanan yang sebenarnya tidak terbuka secara langsung ke jaringan publik.

Misalnya, sebuah database hanya bisa diakses dari server lokal dan tidak dibuka ke internet. Dengan port forwarding, pengguna dapat membuat jalur aman melalui SSH untuk mengakses layanan tersebut dari komputer mereka.

Contoh konsepnya seperti ini:

```
Laptop (Port 3307)
        |
        | SSH Tunnel
        v
Server (Port 3306 MySQL)
```

Dalam skenario ini:

- komputer lokal menggunakan **port 3307**
- PuTTY membuat tunnel SSH
- trafik diteruskan ke **port 3306 di server**

Dengan cara ini, aplikasi di komputer lokal dapat mengakses database server melalui jalur SSH yang aman.

Fitur ini sangat sering digunakan dalam:

- akses database jarak jauh
- debugging aplikasi server
- mengamankan layanan internal

#### Telnet

Selain SSH, PuTTY juga mendukung koneksi menggunakan protokol **Telnet**.

Telnet adalah protokol lama yang digunakan untuk membuka sesi terminal jarak jauh, mirip dengan SSH. Cara penggunaannya di PuTTY juga hampir sama.

Langkah dasarnya biasanya seperti berikut:

1. Masukkan alamat server.
2. Tentukan port Telnet (biasanya `23`).
3. Pilih **Connection Type: Telnet**.
4. Klik **Open**.

Setelah itu, terminal Telnet akan terbuka dan pengguna dapat berinteraksi dengan sistem tujuan.

Namun ada satu perbedaan penting antara Telnet dan SSH.

Telnet **tidak menggunakan enkripsi**, sehingga semua data yang dikirim melalui jaringan berada dalam bentuk **plain text**. Hal ini membuat informasi seperti username dan password dapat dengan mudah disadap jika jaringan tidak aman.

Karena alasan keamanan ini, Telnet saat ini sudah jarang digunakan dalam sistem produksi modern dan lebih sering digunakan untuk:

- simulasi jaringan
- troubleshooting koneksi
- perangkat lama yang belum mendukung SSH

Meskipun begitu, dukungan Telnet di PuTTY tetap berguna dalam beberapa situasi khusus di bidang jaringan dan administrasi sistem.

## 3. Hubungan Antar Konsep

```
PuTTY (Terminal Emulator / SSH Client)
        |
        |-- SSH ---------> Server Linux/Unix (Port 22)
        |                       |
        |                   Login dengan username & password
        |                   (Setelah verifikasi Host Key Fingerprint)
        |
        |-- Telnet -------> Perangkat jaringan (tidak terenkripsi)
        |
        |-- Serial -------> Console Port Router/Switch
                            (Konfigurasi awal / recovery)
```

- **SSH** dan **Telnet** adalah dua protokol yang bisa digunakan PuTTY untuk terhubung ke perangkat remote; perbedaan utamanya adalah SSH **terenkripsi** sementara Telnet **tidak**.
- **Host key fingerprint** adalah bagian dari mekanisme keamanan SSH yang memastikan keaslian server.
- **PSCP dan PSFTP** adalah alat pelengkap PuTTY yang memungkinkan transfer file di atas koneksi SSH yang sama.
- **Serial/Console** adalah metode koneksi _out-of-band_ yang tidak memerlukan jaringan IP sama sekali, sangat penting dalam skenario konfigurasi awal perangkat jaringan.

---

## 4. Kesimpulan

PuTTY adalah alat yang ringan namun sangat powerful untuk:

- Mengakses server Linux/Unix dari Windows melalui **SSH** secara aman.
- Mengonfigurasi perangkat jaringan (router, switch, firewall) melalui koneksi **Serial** atau **Telnet**.
- Melakukan transfer file melalui **PSCP** atau **PSFTP**.
- Menerapkan **port forwarding** untuk kebutuhan tunneling jaringan.

Proses penggunaannya sangat sederhana: instal PuTTY → masukkan IP dan port → pilih protokol → buka koneksi → verifikasi host key → login. Dengan kemampuan ini, PuTTY menjadi salah satu tool wajib bagi administrator sistem dan jaringan yang bekerja di lingkungan Windows.
