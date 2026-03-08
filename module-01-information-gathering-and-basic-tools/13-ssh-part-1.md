# 📝 SSH (Secure Shell) — Bagian 1: Pengenalan dan Cara Kerja Dasar

---

## 1. Ringkasan Singkat

Video ini memperkenalkan **SSH (Secure Shell)**, sebuah protokol yang digunakan secara luas oleh profesional keamanan siber maupun administrator sistem untuk melakukan remote login, eksekusi perintah jarak jauh, transfer file yang aman, serta tunneling. Materi mencakup apa itu SSH, bagaimana cara kerjanya di balik layar, serta demonstrasi praktis menghubungkan dua mesin virtual (Kali Linux sebagai client dan OWASP Broken Web Application sebagai server) melalui terminal Linux.

---

## 2. Konsep Utama

### a. Apa itu SSH?

#### Pengertian SSH

**SSH (Secure Shell)** adalah sebuah protokol jaringan yang digunakan untuk melakukan komunikasi secara **aman antara dua komputer** melalui jaringan, biasanya melalui internet. SSH paling sering digunakan untuk **mengakses server dari jarak jauh (remote access)**.

Dengan SSH, seorang administrator atau developer dapat masuk ke sebuah server, menjalankan perintah, mengelola file, bahkan membuat koneksi jaringan khusus — semuanya dilakukan dengan **enkripsi yang kuat** sehingga komunikasi tidak bisa dibaca oleh pihak lain.

Beberapa fungsi utama SSH antara lain:

- **Remote login yang aman**
  Mengakses server dari komputer lain dengan koneksi yang terenkripsi.

- **Eksekusi perintah dari jarak jauh**
  Menjalankan perintah pada server tanpa harus berada di mesin tersebut.

- **Transfer file secara aman**
  Mengirim atau mengambil file melalui protokol seperti **SCP** atau **SFTP**.

- **Tunneling dan proxying**
  Membuat jalur komunikasi aman untuk aplikasi lain melalui koneksi SSH.

Karena kemampuannya ini, SSH menjadi salah satu **alat paling penting dalam administrasi server modern**, terutama pada sistem operasi Linux.

#### Port Default SSH

Secara default, SSH berjalan di atas protokol **TCP** menggunakan **port 22**.

Artinya, ketika sebuah klien mencoba terhubung ke server menggunakan SSH, koneksi tersebut biasanya akan diarahkan ke port 22 pada server.

Contoh penggunaan SSH dari terminal:

```bash
ssh user@server-ip
```

Misalnya:

```bash
ssh admin@192.168.1.10
```

Perintah ini berarti:

- `ssh` → menjalankan program SSH client
- `admin` → username di server tujuan
- `192.168.1.10` → alamat server yang ingin diakses

Jika koneksi berhasil, pengguna akan mendapatkan **shell terminal pada server tersebut**, seolah-olah sedang menggunakan komputer itu secara langsung.

#### Jaminan Keamanan dalam SSH

Salah satu alasan utama mengapa SSH digunakan secara luas adalah karena protokol ini dirancang dengan **mekanisme keamanan yang kuat**.

SSH menyediakan tiga jaminan keamanan utama yang merupakan versi modifikasi dari konsep **CIA Triad** dalam keamanan informasi.

Berikut adalah tiga aspek keamanan tersebut:

| Aspek Keamanan                    | Mekanisme dalam SSH                                                              |
| --------------------------------- | -------------------------------------------------------------------------------- |
| **Confidentiality (Kerahasiaan)** | Data dienkripsi sehingga tidak dapat dibaca oleh pihak lain                      |
| **Integrity (Integritas)**        | Data dilindungi menggunakan Message Authentication Code (MAC)                    |
| **Authentication (Autentikasi)**  | Verifikasi identitas menggunakan password, public key, host key, atau sertifikat |

Mari kita pahami satu per satu.

#### Confidentiality (Kerahasiaan)

**Confidentiality** berarti bahwa data yang dikirim melalui jaringan **tidak dapat dibaca oleh pihak yang tidak berwenang**.

SSH mencapai hal ini dengan menggunakan **enkripsi**. Ketika data dikirim antara klien dan server, data tersebut diubah menjadi bentuk terenkripsi sebelum melewati jaringan.

Sebagai contoh:

1. Anda mengetik sebuah perintah di terminal.
2. SSH akan **mengenkripsi perintah tersebut**.
3. Data terenkripsi dikirim melalui jaringan.
4. Server kemudian **mendekripsi data** untuk membaca perintahnya.

Jika seseorang mencoba menyadap koneksi jaringan, mereka hanya akan melihat **data terenkripsi yang tidak dapat dipahami**.

#### Integrity (Integritas)

Selain menjaga kerahasiaan, SSH juga memastikan bahwa **data tidak diubah selama proses pengiriman**.

Untuk melakukan ini, SSH menggunakan mekanisme yang disebut **Message Authentication Code (MAC)**.

MAC bekerja dengan cara:

1. Menghasilkan **kode hash khusus** dari pesan yang dikirim.
2. Kode ini dikirim bersama dengan pesan tersebut.
3. Penerima menghitung ulang kode hash dari pesan yang diterima.
4. Jika hasilnya sama, berarti pesan **tidak mengalami perubahan**.

Jika ada pihak yang mencoba memodifikasi data di tengah perjalanan, perbedaan hash akan terdeteksi dan pesan tersebut akan dianggap **tidak valid**.

#### Authentication (Autentikasi)

Aspek keamanan berikutnya adalah **authentication**, yaitu proses memastikan bahwa identitas kedua pihak yang berkomunikasi benar-benar valid.

SSH menyediakan beberapa metode autentikasi, di antaranya:

- **Password authentication**
  Pengguna memasukkan password untuk login ke server.

- **Public key authentication**
  Menggunakan pasangan **public key** dan **private key** untuk login tanpa password.

- **Host key verification**
  Memastikan bahwa klien benar-benar terhubung ke server yang tepat, bukan server palsu.

- **Certificate-based authentication**
  Menggunakan sertifikat digital untuk memverifikasi identitas.

Metode **public key authentication** sering dianggap lebih aman dibandingkan password karena tidak memerlukan pengiriman password melalui jaringan.

#### Catatan tentang CIA Triad

Dalam keamanan informasi secara umum, dikenal konsep **CIA Triad**, yang terdiri dari:

- **Confidentiality**
- **Integrity**
- **Availability**

Namun dalam konteks SSH, fokus utamanya adalah **keamanan komunikasi**, bukan ketersediaan layanan.

Karena itu, dalam pembahasan SSH sering digunakan tiga aspek berikut:

- **Confidentiality**
- **Integrity**
- **Authentication**

Dengan kata lain, **Availability** digantikan oleh **Authentication**, karena tujuan utama SSH adalah memastikan bahwa komunikasi antara klien dan server:

- tetap **rahasia**
- tidak **dimodifikasi**
- dan dilakukan oleh **pihak yang sah**.

### b. Implementasi SSH: OpenSSH

#### Apa itu OpenSSH

Dalam praktiknya, protokol **SSH** tidak digunakan secara langsung tanpa software. Kita membutuhkan sebuah **implementasi perangkat lunak** yang mampu menjalankan protokol tersebut. Implementasi SSH yang paling populer dan paling banyak digunakan adalah **OpenSSH**.

**OpenSSH (Open Secure Shell)** adalah kumpulan program open-source yang menyediakan berbagai tools untuk melakukan komunikasi aman menggunakan protokol SSH. Software ini tersedia secara default atau mudah diinstall di hampir semua sistem operasi modern seperti:

- Linux
- macOS
- Windows (melalui OpenSSH client bawaan atau Windows Subsystem)

Karena sifatnya open-source dan sangat stabil, OpenSSH menjadi **standar de facto** untuk koneksi SSH di dunia server dan DevOps.

Dalam OpenSSH terdapat dua komponen utama yang bekerja bersama:

- **SSH Client**
- **SSH Server**

#### SSH Client (`ssh`)

Program **`ssh`** adalah aplikasi yang digunakan oleh pengguna untuk **memulai koneksi ke server**.

Biasanya program ini dijalankan melalui terminal atau command line. Ketika kita menjalankan perintah `ssh`, program ini akan:

1. Membuka koneksi jaringan ke server
2. Melakukan proses **negosiasi enkripsi**
3. Melakukan **autentikasi pengguna**
4. Jika berhasil, memberikan **akses shell ke server**

Dengan kata lain, `ssh` adalah **alat yang kita gunakan dari sisi pengguna (client)** untuk masuk ke server.

#### SSH Server (`sshd`)

Di sisi server terdapat program bernama **`sshd`**, yang merupakan singkatan dari **SSH Daemon**.

Daemon adalah program yang berjalan di latar belakang dan **menunggu permintaan koneksi dari client**.

Fungsi utama `sshd` adalah:

- Mendengarkan koneksi pada **port 22**
- Menerima koneksi dari SSH client
- Melakukan **proses autentikasi**
- Membuat **session shell** untuk pengguna yang berhasil login

Secara sederhana, hubungan antara client dan server bisa dibayangkan seperti ini:

```
Client (ssh)  -------->  Server (sshd)
```

Client mengirim permintaan koneksi, lalu server memverifikasi identitas pengguna sebelum memberikan akses.

#### Contoh Perintah Dasar SSH

Berikut contoh perintah dasar untuk terhubung ke server menggunakan SSH:

```bash
ssh root@172.17.199.151
```

Format umum dari perintah ini adalah:

```
ssh <username>@<IP_address>
```

Mari kita uraikan bagian-bagiannya.

- **`ssh`**
  Program SSH client yang dijalankan dari terminal.

- **`root`**
  Username yang ingin digunakan untuk login ke server.

- **`172.17.199.151`**
  Alamat IP dari server tujuan.

#### Apa yang Terjadi Ketika Perintah Ini Dijalankan

Ketika Anda menjalankan perintah tersebut, beberapa proses akan terjadi secara berurutan:

1. SSH client mencoba membuka koneksi ke **server pada port 22**.
2. Client dan server melakukan **negosiasi algoritma enkripsi**.
3. Server mengirimkan **host key** untuk memverifikasi identitasnya.
4. Client meminta **autentikasi pengguna** (biasanya password atau SSH key).
5. Jika autentikasi berhasil, server memberikan **akses shell**.

Contoh tampilan yang mungkin muncul di terminal:

```
root@172.17.199.151's password:
```

Setelah password dimasukkan dengan benar, Anda akan masuk ke server dan melihat prompt seperti ini:

```
root@server:~#
```

Pada titik ini, Anda sudah berada di dalam sistem server dan dapat menjalankan berbagai perintah Linux seperti:

```bash
ls
pwd
top
apt update
```

Semua perintah tersebut sebenarnya dijalankan **di server**, bukan di komputer lokal Anda.

Inilah yang membuat SSH sangat powerful: Anda bisa **mengelola server dari mana saja**, selama memiliki akses jaringan dan kredensial yang benar.

### c. Apa yang Terjadi di Balik Layar Saat Koneksi SSH?

Ketika kita menjalankan perintah SSH di terminal, prosesnya terlihat sangat sederhana. Misalnya hanya dengan satu baris perintah seperti:

```bash
ssh user@server-ip
```

Namun sebenarnya, di balik layar terjadi **serangkaian proses keamanan yang cukup kompleks** sebelum kita benar-benar mendapatkan akses ke server. Semua proses ini berlangsung secara otomatis dalam hitungan milidetik.

Secara garis besar, tahapan koneksi SSH dapat digambarkan seperti berikut:

```
[1] TCP Handshake
        ↓
[2] Key Exchange & Server Authentication
        ↓
[3] Negosiasi Enkripsi (Shared Secret Keys)
        ↓
[4] User Authentication (password / public key)
        ↓
[5] Pembukaan Session & Channel
        ↓
[6] Forwarding, Tunneling & Host Key Verification
```

Mari kita bahas setiap tahap tersebut secara lebih rinci.

#### 1. TCP Handshake

Tahap pertama adalah **membentuk koneksi jaringan dasar menggunakan TCP**.

SSH berjalan di atas protokol **TCP (Transmission Control Protocol)**, biasanya pada **port 22**. Ketika perintah SSH dijalankan, client akan mencoba membuka koneksi TCP ke server.

Proses ini menggunakan mekanisme yang disebut **TCP three-way handshake**, yaitu:

1. Client mengirim paket **SYN** ke server.
2. Server merespons dengan **SYN-ACK**.
3. Client membalas dengan **ACK**.

Jika proses ini berhasil, berarti koneksi jaringan sudah terbentuk dan komunikasi bisa dilanjutkan ke tahap berikutnya.

Pada tahap ini **belum ada enkripsi SSH**, karena koneksi baru saja dibangun pada level jaringan.

#### 2. Key Exchange & Server Authentication

Setelah koneksi TCP berhasil, tahap berikutnya adalah **pertukaran kunci kriptografi (key exchange)**.

Pada tahap ini client dan server akan:

- Menentukan **algoritma kriptografi** yang akan digunakan
- Bertukar informasi untuk menghasilkan **shared secret**
- Memverifikasi **identitas server**

Biasanya SSH menggunakan algoritma seperti:

- Diffie-Hellman
- ECDH (Elliptic Curve Diffie-Hellman)

Algoritma ini memungkinkan dua pihak untuk menghasilkan **kunci rahasia bersama** meskipun komunikasi awal terjadi melalui jaringan publik.

Selain itu, server juga akan mengirimkan **host key** kepada client. Host key ini berfungsi untuk memastikan bahwa client benar-benar terhubung ke server yang benar, bukan server palsu yang mencoba melakukan **man-in-the-middle attack**.

Jika ini adalah pertama kali kita terhubung ke server, biasanya terminal akan menampilkan pesan seperti:

```bash
The authenticity of host '172.17.199.151' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Jika pengguna menjawab `yes`, maka host key server akan disimpan di file:

```
~/.ssh/known_hosts
```

Pada koneksi berikutnya, SSH akan memverifikasi bahwa host key server **tidak berubah**.

#### 3. Negosiasi Enkripsi (Shared Secret Keys)

Setelah proses key exchange selesai, client dan server akan memiliki **shared secret key** yang sama.

Kunci ini kemudian digunakan untuk membuat **kunci enkripsi sesi**. Mulai dari tahap ini, semua komunikasi antara client dan server akan:

- **dienkripsi**
- **dilindungi dari penyadapan**
- **dilindungi dari modifikasi data**

Dengan kata lain, siapa pun yang mencoba menyadap lalu lintas jaringan hanya akan melihat **data acak yang tidak bisa dibaca**.

Ini adalah tahap di mana **kerahasiaan komunikasi SSH benar-benar mulai berlaku**.

#### 4. User Authentication

Setelah jalur komunikasi aman terbentuk, server akan meminta pengguna untuk **membuktikan identitasnya**.

Proses ini disebut **user authentication**.

Beberapa metode autentikasi yang umum digunakan adalah:

**Password authentication**

Server meminta pengguna memasukkan password.

Contoh tampilan di terminal:

```bash
user@172.17.199.151's password:
```

Jika password benar, akses akan diberikan.

**Public key authentication**

Metode ini menggunakan pasangan kunci:

- **private key** (disimpan di komputer client)
- **public key** (disimpan di server)

Server akan memverifikasi bahwa client benar-benar memiliki private key yang sesuai dengan public key yang terdaftar.

Metode ini lebih aman karena **tidak mengirim password melalui jaringan**.

#### 5. Sessions & Channels

Setelah autentikasi berhasil, SSH akan membuka **session** antara client dan server.

Di dalam satu koneksi SSH, sebenarnya dapat dibuat beberapa **channel komunikasi** yang berbeda. Channel ini adalah jalur komunikasi logis di dalam koneksi SSH yang sudah terenkripsi.

Beberapa jenis channel yang umum digunakan adalah:

**Shell channel**

Digunakan untuk membuka **terminal interaktif** di server.

Contoh:

```bash
ssh user@server-ip
```

Setelah login, kita mendapatkan shell seperti:

```bash
user@server:~$
```

Di sinilah kita dapat menjalankan berbagai perintah Linux.

**Exec channel**

Digunakan untuk menjalankan **satu perintah langsung dari client** tanpa membuka shell interaktif.

Contoh:

```bash
ssh user@server-ip "ls -la"
```

Server akan menjalankan perintah `ls -la` lalu mengirim hasilnya kembali ke client.

**Subsystem channel (SFTP)**

Digunakan untuk menjalankan **subsystem khusus**, misalnya **SFTP** untuk transfer file secara aman.

Contoh penggunaan:

```bash
sftp user@server-ip
```

Ini akan membuka sesi transfer file berbasis SSH.

#### 6. Forwarding, Tunneling & Host Key Verification

Tahap terakhir adalah penggunaan fitur tambahan SSH seperti **port forwarding** dan **tunneling**.

Fitur ini memungkinkan SSH digunakan bukan hanya untuk login ke server, tetapi juga untuk **mengamankan koneksi aplikasi lain**.

Contoh penggunaan yang umum:

**Local Port Forwarding**

Mengakses layanan internal server melalui komputer lokal.

Contoh:

```bash
ssh -L 8080:localhost:80 user@server-ip
```

Artinya:

- Port `8080` di komputer lokal
- akan diteruskan ke
- port `80` pada server

Sehingga kita bisa membuka layanan web server melalui:

```
http://localhost:8080
```

Selain itu, SSH juga terus melakukan **host key verification** untuk memastikan bahwa identitas server tidak berubah selama koneksi berlangsung.

Jika host key server berubah secara tiba-tiba, SSH biasanya akan menampilkan peringatan keamanan karena hal tersebut bisa menjadi indikasi adanya **serangan man-in-the-middle**.

Dengan semua tahapan ini, SSH mampu menyediakan koneksi yang:

- aman
- terenkripsi
- terverifikasi

Meskipun dari sisi pengguna hanya terlihat sebagai **satu perintah sederhana di terminal**.

### d. Masalah Negosiasi Host Key

Ketika mencoba melakukan koneksi SSH ke sebuah server, terkadang koneksi tidak langsung berhasil. Salah satu error yang cukup sering muncul adalah masalah **negosiasi host key**.

Contoh pesan error yang mungkin muncul di terminal:

```
Unable to negotiate with server.
No matching host key type found. Their offer: ssh-rsa
```

Sekilas pesan ini mungkin terlihat membingungkan, tetapi sebenarnya error ini menunjukkan bahwa **client dan server tidak sepakat mengenai algoritma keamanan yang akan digunakan** saat proses koneksi SSH.

Untuk memahami masalah ini, kita perlu melihat bagaimana proses **negosiasi keamanan SSH** bekerja.

#### Apa Itu Host Key dalam SSH

Dalam SSH, **host key** adalah kunci kriptografi yang dimiliki oleh server. Kunci ini digunakan untuk **membuktikan identitas server kepada client**.

Saat client pertama kali terhubung ke server, server akan mengirimkan host key-nya. Client kemudian akan:

1. Memverifikasi host key tersebut.
2. Menyimpannya ke dalam file `known_hosts`.
3. Menggunakannya untuk memverifikasi koneksi pada pertemuan berikutnya.

Host key ini dibuat menggunakan **algoritma kriptografi tertentu**, misalnya:

- `rsa`
- `ecdsa`
- `ed25519`

Algoritma ini menentukan bagaimana kunci dibuat dan bagaimana proses verifikasi dilakukan.

#### Mengapa Error Negosiasi Bisa Terjadi

Pada tahap awal koneksi SSH, client dan server melakukan proses yang disebut **algorithm negotiation**. Di sini kedua pihak akan mencoba menyepakati algoritma yang sama untuk beberapa hal, termasuk:

- algoritma enkripsi
- algoritma pertukaran kunci
- algoritma host key

Masalah muncul ketika:

- server hanya mendukung **algoritma lama**, dan
- client modern **tidak lagi mengizinkan algoritma tersebut secara default**.

Contoh yang sering terjadi adalah algoritma **`ssh-rsa`**.

Pada banyak sistem modern, algoritma `ssh-rsa` sudah tidak lagi diaktifkan secara default karena dianggap **kurang aman dibandingkan algoritma baru**, seperti:

- `rsa-sha2-256`
- `rsa-sha2-512`
- `ed25519`

Jika server hanya menawarkan `ssh-rsa`, sedangkan client tidak mengizinkannya, maka negosiasi akan gagal dan muncul pesan seperti:

```
No matching host key type found. Their offer: ssh-rsa
```

Artinya: server hanya menawarkan `ssh-rsa`, tetapi client tidak memiliki algoritma tersebut dalam daftar yang diizinkan.

#### Cara Mengatasi Masalah Ini

Jika Anda tetap perlu terhubung ke server tersebut (misalnya server lama atau lab environment), Anda bisa **secara manual mengizinkan algoritma `ssh-rsa` pada client**.

Hal ini dapat dilakukan dengan menambahkan opsi khusus pada perintah SSH.

Contoh:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa root@172.17.199.151
```

Mari kita uraikan bagian dari perintah tersebut.

**`-o HostKeyAlgorithms=+ssh-rsa`**

Opsi ini memberi tahu SSH client untuk **menambahkan algoritma `ssh-rsa` ke daftar host key yang diperbolehkan**.

Tanda `+` berarti algoritma tersebut **ditambahkan**, bukan menggantikan seluruh daftar yang sudah ada.

**`-o PubkeyAcceptedAlgorithms=+ssh-rsa`**

Opsi ini mengizinkan penggunaan algoritma `ssh-rsa` juga untuk **public key authentication**.

Beberapa server lama masih menggunakan metode ini, sehingga client perlu diinstruksikan untuk menerima algoritma tersebut.

**`root@172.17.199.151`**

Ini adalah username dan alamat server yang ingin dihubungi.

Setelah opsi tersebut ditambahkan, SSH client akan mencoba kembali proses negosiasi dan biasanya koneksi akan berhasil.

#### Contoh Alur yang Terjadi

Sebelum perbaikan:

```bash
ssh root@172.17.199.151
```

Hasil:

```
Unable to negotiate with server.
No matching host key type found. Their offer: ssh-rsa
```

Setelah menambahkan opsi:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa root@172.17.199.151
```

Koneksi dapat berhasil karena client sekarang mengizinkan algoritma yang ditawarkan server.

#### Catatan Keamanan

Meskipun solusi ini bekerja, ada hal penting yang perlu diperhatikan.

Algoritma seperti **`ssh-rsa`** dianggap **lebih lemah dibandingkan algoritma modern**, terutama karena menggunakan metode tanda tangan lama berbasis **SHA-1**.

Karena alasan ini, banyak implementasi SSH modern mulai:

- menonaktifkan `ssh-rsa` secara default
- mendorong penggunaan algoritma yang lebih kuat seperti:
  - `ed25519`
  - `rsa-sha2-256`
  - `rsa-sha2-512`

Oleh karena itu, solusi terbaik dalam jangka panjang adalah:

- **memperbarui konfigurasi server**
- atau **menghasilkan host key baru dengan algoritma modern**

Sementara itu, menambahkan opsi `ssh-rsa` pada client sebaiknya hanya dilakukan ketika benar-benar diperlukan, misalnya saat bekerja dengan **server lama, sistem legacy, atau lingkungan lab**.

### e. Verifikasi Host Fingerprint (Known Hosts)

#### Mengapa SSH Memverifikasi Identitas Server

Ketika Anda mencoba terhubung ke sebuah server menggunakan SSH untuk pertama kalinya, biasanya terminal akan menampilkan pesan seperti ini:

```id="7ud5pr"
The authenticity of host '172.17.199.151' can't be established.
RSA key fingerprint is SHA256:xxxxxxx...
Are you sure you want to continue connecting (yes/no)?
```

Pesan ini bukanlah error. Justru ini adalah **mekanisme keamanan penting dalam SSH** yang bertujuan untuk memastikan bahwa Anda benar-benar terhubung ke server yang benar.

SSH tidak hanya mengautentikasi **pengguna**, tetapi juga memverifikasi **identitas server**. Proses ini dilakukan menggunakan sesuatu yang disebut **host fingerprint**.

#### Apa Itu Host Fingerprint

**Host fingerprint** adalah representasi singkat dari **host key** server dalam bentuk hash.

Host key sendiri adalah kunci kriptografi yang dimiliki oleh server dan digunakan untuk membuktikan identitasnya kepada client. Namun karena host key biasanya panjang dan sulit dibaca manusia, SSH menampilkan versi ringkasnya dalam bentuk **fingerprint**.

Contohnya:

```id="46w3no"
SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz123456789
```

Fingerprint ini dihasilkan dengan menjalankan fungsi hash terhadap host key server.

Dengan cara ini, pengguna dapat memverifikasi bahwa:

- server yang dihubungi benar
- koneksi tidak sedang disusupi pihak lain

#### Mengapa Pesan Ini Muncul

Pesan tersebut muncul karena **SSH client belum pernah terhubung ke server tersebut sebelumnya**.

Artinya:

- client belum memiliki catatan tentang host key server
- client tidak bisa memastikan apakah server tersebut benar atau palsu

Oleh karena itu SSH meminta konfirmasi dari pengguna sebelum melanjutkan koneksi.

#### Apa yang Terjadi Jika Menjawab "yes"

Jika Anda menjawab **yes**, maka SSH akan:

1. Menerima host key yang diberikan oleh server.
2. Menyimpannya ke dalam file khusus bernama:

```id="h3tqtp"
~/.ssh/known_hosts
```

File `known_hosts` berfungsi sebagai **database server yang pernah dipercaya oleh client**.

Setelah host key disimpan, koneksi SSH akan dilanjutkan seperti biasa hingga proses autentikasi pengguna.

#### Verifikasi pada Koneksi Berikutnya

Ketika Anda melakukan koneksi ke server yang sama di masa depan, SSH akan melakukan langkah tambahan:

1. Client membaca file `known_hosts`.
2. Client mencari apakah ada host key yang cocok dengan server tersebut.
3. Client membandingkan host key yang tersimpan dengan host key yang dikirim server.

Jika keduanya **identik**, koneksi akan dilanjutkan tanpa pertanyaan tambahan.

Namun jika host key **berbeda**, SSH akan menampilkan peringatan keamanan serius.

Hal ini penting karena perubahan host key bisa berarti:

- server telah diganti
- konfigurasi server berubah
- atau ada kemungkinan **serangan man-in-the-middle**

#### Contoh Alur Koneksi Pertama

Misalnya Anda menjalankan perintah berikut:

```bash id="t4k98v"
ssh root@172.17.199.151
```

Terminal kemudian menampilkan:

```id="p6sz86"
The authenticity of host '172.17.199.151' can't be established.
RSA key fingerprint is SHA256:xxxxxxx...
Are you sure you want to continue connecting (yes/no)?
```

Jika Anda mengetik:

```id="c3d4ku"
yes
```

SSH akan menyimpan host key server ke file `known_hosts`, lalu melanjutkan proses login.

Pada koneksi berikutnya ke server yang sama, pesan ini **tidak akan muncul lagi**, karena server tersebut sudah dikenal oleh client.

#### Mengapa Mekanisme Ini Penting

Sistem **known_hosts** adalah salah satu mekanisme penting untuk melindungi koneksi SSH dari serangan yang disebut **man-in-the-middle attack**.

Dalam serangan ini, penyerang mencoba:

1. Menyamar sebagai server asli.
2. Mengarahkan koneksi pengguna ke server palsu.
3. Mengintip atau memodifikasi data yang dikirim.

Dengan menyimpan host key server yang sah, SSH dapat memastikan bahwa setiap koneksi berikutnya benar-benar menuju **server yang sama seperti sebelumnya**.

Jika ada perubahan host key secara tiba-tiba, SSH akan segera memberikan peringatan sehingga pengguna dapat menyadari bahwa ada sesuatu yang tidak normal pada koneksi tersebut.

### f. Autentikasi dengan Public Key

#### Mengapa Public Key Authentication Digunakan

Selain menggunakan password, SSH juga mendukung metode autentikasi yang lebih aman yaitu **public key authentication**. Metode ini menggunakan konsep **kriptografi kunci asimetris**, yaitu sepasang kunci yang saling berhubungan:

- **Private key**
- **Public key**

Pendekatan ini dianggap lebih aman dibandingkan password karena **tidak ada password yang perlu dikirim atau diketik setiap kali login**. Selain itu, metode ini juga lebih tahan terhadap serangan seperti **brute force**.

Dalam sistem ini, autentikasi dilakukan dengan memverifikasi bahwa client benar-benar memiliki **private key** yang cocok dengan **public key** yang sudah didaftarkan di server.

#### Struktur Kunci dalam SSH

Public key authentication menggunakan dua kunci yang saling terkait.

**Private key**

Private key adalah kunci rahasia yang disimpan di komputer client. Kunci ini harus dijaga dengan sangat baik karena siapa pun yang memiliki private key dapat mencoba login ke server yang mempercayainya.

Biasanya private key disimpan di direktori berikut:

```id="kh5d76"
~/.ssh/
```

Contoh file private key:

```id="3cl32p"
~/.ssh/id_rsa
~/.ssh/id_ed25519
```

**Public key**

Public key adalah pasangan dari private key. Berbeda dengan private key, kunci ini **boleh dibagikan**.

Public key akan disimpan di server pada file:

```id="ntq8cy"
~/.ssh/authorized_keys
```

File ini berisi daftar public key dari pengguna yang diizinkan untuk login ke server.

Contoh isi file `authorized_keys`:

```id="6pqghh"
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIExampleKeyUserLaptop
```

Jika public key client ada di dalam file ini, server akan mengizinkan autentikasi menggunakan pasangan private key yang sesuai.

#### Alur Autentikasi Public Key

Proses autentikasi menggunakan public key tidak melibatkan pengiriman private key ke server. Sebagai gantinya, SSH menggunakan mekanisme **challenge–response** untuk membuktikan kepemilikan private key.

Alur prosesnya dapat digambarkan seperti berikut:

```
Server mengirim "challenge" (data acak)
        ↓
Client menandatangani challenge dengan private key
        ↓
Server memverifikasi tanda tangan menggunakan public key
        ↓
Jika cocok → akses diberikan (tanpa perlu password)
```

Mari kita lihat langkah-langkahnya secara lebih rinci.

#### 1. Server Mengirim Challenge

Setelah koneksi SSH terbentuk dan server mengetahui bahwa client ingin menggunakan public key authentication, server akan mengirimkan sebuah **challenge**.

Challenge ini biasanya berupa **data acak** yang dibuat khusus untuk sesi tersebut.

Tujuannya adalah untuk memastikan bahwa client benar-benar memiliki private key yang sesuai.

#### 2. Client Menandatangani Challenge

Client kemudian akan menggunakan **private key** miliknya untuk **menandatangani challenge tersebut**.

Proses ini disebut **digital signing**. Dengan menggunakan private key, client menghasilkan tanda tangan kriptografi dari data challenge.

Yang penting untuk dipahami adalah:

- **private key tidak pernah dikirim ke server**
- yang dikirim hanya **hasil tanda tangan digital**

#### 3. Server Memverifikasi Tanda Tangan

Setelah menerima tanda tangan dari client, server akan mengambil **public key** milik client dari file:

```id="y6i0jn"
~/.ssh/authorized_keys
```

Server kemudian menggunakan public key tersebut untuk **memverifikasi tanda tangan digital** yang dibuat oleh client.

Jika tanda tangan tersebut valid, berarti:

- client benar-benar memiliki private key yang sesuai
- identitas client dapat dipercaya

#### 4. Akses Diberikan

Jika proses verifikasi berhasil, server akan memberikan akses kepada pengguna tanpa perlu meminta password.

Pada titik ini koneksi SSH akan langsung masuk ke session shell, seperti:

```id="pc5qok"
user@server:~$
```

Pengguna dapat langsung menjalankan perintah di server tanpa perlu memasukkan password.

#### Contoh Penggunaan Login dengan Public Key

Jika public key sudah disimpan di server, pengguna cukup menjalankan perintah SSH biasa:

```bash id="ajof0y"
ssh user@172.17.199.151
```

SSH client akan secara otomatis:

1. mencari private key di direktori `~/.ssh`
2. mencoba melakukan autentikasi menggunakan kunci tersebut
3. melakukan proses challenge–response dengan server

Jika semuanya cocok, login akan berhasil **tanpa prompt password**.

#### Keuntungan Public Key Authentication

Metode ini memiliki beberapa keuntungan dibandingkan autentikasi berbasis password.

**Lebih aman**

Private key tidak pernah dikirim melalui jaringan, sehingga lebih sulit dicuri.

**Tahan terhadap brute force**

Penyerang tidak bisa menebak password karena autentikasi bergantung pada kepemilikan private key.

**Lebih praktis untuk otomatisasi**

Public key authentication sering digunakan dalam:

- deployment server
- automation scripts
- CI/CD pipelines
- akses antar server

Karena proses login dapat dilakukan **tanpa interaksi manual**.

Inilah alasan mengapa pada banyak sistem produksi, administrator biasanya **menonaktifkan login password SSH** dan hanya mengizinkan autentikasi menggunakan public key.

### g. Session & Channel dalam SSH

#### Gambaran Umum Session dalam SSH

Setelah proses koneksi SSH selesai—mulai dari TCP handshake, pertukaran kunci, hingga autentikasi pengguna—server akan membuka **session SSH**. Session ini merupakan koneksi aktif antara client dan server yang memungkinkan berbagai aktivitas dilakukan secara aman.

Menariknya, SSH tidak hanya membuka satu jalur komunikasi sederhana. Sebaliknya, SSH menggunakan konsep yang disebut **multiplexing channel**.

Artinya, **dalam satu koneksi SSH yang terenkripsi, kita bisa menjalankan beberapa jenis komunikasi sekaligus**. Setiap jenis komunikasi ini berjalan dalam sebuah **channel** yang terpisah secara logis, tetapi semuanya tetap berada di dalam satu tunnel SSH yang sama.

Secara konseptual, hubungan ini bisa dibayangkan seperti sebuah terowongan besar yang memiliki beberapa jalur di dalamnya:

```id="1x0nq9"
SSH Encrypted Tunnel
        ├── Shell Channel
        ├── Exec Channel
        └── SFTP Subsystem
```

Semua channel tersebut berjalan secara bersamaan, tetapi tetap aman karena seluruh koneksi berada di dalam **tunnel SSH yang sudah terenkripsi**.

#### Shell Channel

Channel yang paling sering digunakan dalam SSH adalah **Shell Channel**.

Channel ini digunakan untuk membuka **sesi terminal interaktif** pada server. Setelah login berhasil, pengguna akan mendapatkan shell yang memungkinkan mereka menjalankan berbagai perintah seolah-olah sedang bekerja langsung di server tersebut.

Contoh penggunaan:

```bash id="2q7hxt"
ssh user@server-ip
```

Setelah berhasil login, biasanya kita akan melihat prompt seperti:

```id="i7c8mb"
user@server:~$
```

Pada titik ini kita berada dalam **shell interaktif** di server. Pengguna dapat menjalankan berbagai perintah seperti:

```bash id="qg33ws"
ls
pwd
top
cat file.txt
```

Semua perintah tersebut dikirim melalui **shell channel**, dan hasilnya dikirim kembali ke client melalui channel yang sama.

Shell channel biasanya tetap terbuka selama pengguna masih berada dalam sesi SSH.

#### Exec Channel

Selain shell interaktif, SSH juga menyediakan **Exec Channel**.

Channel ini digunakan ketika kita ingin menjalankan **satu perintah langsung di server tanpa membuka sesi terminal penuh**.

Contoh penggunaan:

```bash id="yy7wxf"
ssh user@server-ip "ls -la"
```

Dalam perintah ini, SSH akan:

1. Menghubungi server.
2. Mengautentikasi pengguna.
3. Menjalankan perintah `ls -la`.
4. Mengirimkan hasilnya kembali ke client.
5. Menutup koneksi setelah perintah selesai.

Contoh hasil di terminal:

```id="j1s5od"
total 8
drwxr-xr-x 2 user user 4096 Jun 10 12:00 .
drwxr-xr-x 3 user user 4096 Jun 10 11:59 ..
```

Karena hanya menjalankan satu perintah, **exec channel biasanya langsung ditutup setelah tugas selesai**.

Metode ini sering digunakan dalam:

- automation script
- remote administration
- deployment tools
- CI/CD pipelines

#### Subsystem SFTP

SSH juga mendukung channel khusus yang disebut **subsystem**. Salah satu subsystem yang paling umum adalah **SFTP (Secure File Transfer Protocol)**.

SFTP memungkinkan pengguna melakukan **transfer file secara aman** antara client dan server menggunakan koneksi SSH.

Contoh membuka sesi SFTP:

```bash id="frw6m8"
sftp user@server-ip
```

Jika koneksi berhasil, biasanya akan muncul prompt seperti ini:

```id="n2vdps"
sftp>
```

Di dalam sesi ini, pengguna dapat menjalankan perintah untuk mengelola file di server, misalnya:

```bash id="2h2w7k"
ls
get file.txt
put image.png
```

Penjelasan perintah tersebut:

- `ls` → melihat daftar file di server
- `get file.txt` → mengunduh file dari server ke komputer lokal
- `put image.png` → mengunggah file dari komputer lokal ke server

Semua aktivitas ini tetap berjalan melalui **koneksi SSH yang terenkripsi**, sehingga data yang ditransfer tidak dapat disadap oleh pihak lain.

#### Multiplexing Channel dalam Satu Tunnel

Hal yang membuat SSH sangat efisien adalah kemampuannya menjalankan **beberapa channel sekaligus dalam satu koneksi terenkripsi**.

Sebagai contoh, dalam satu koneksi SSH kita dapat:

- membuka shell interaktif
- menjalankan perintah otomatis
- mentransfer file melalui SFTP
- melakukan port forwarding

Semua aktivitas tersebut berjalan dalam **satu tunnel SSH yang sama**.

Keuntungan pendekatan ini adalah:

- **lebih efisien** karena tidak perlu membuat koneksi baru
- **lebih cepat** karena proses enkripsi sudah dilakukan sekali
- **lebih aman** karena seluruh komunikasi berada dalam tunnel terenkripsi

Inilah salah satu alasan mengapa SSH menjadi protokol yang sangat fleksibel dan banyak digunakan untuk berbagai kebutuhan, mulai dari **administrasi server hingga otomatisasi sistem**.

### h. Demonstrasi Praktis: Mengendalikan Mesin Jarak Jauh

#### Gambaran Umum Demonstrasi

Salah satu cara terbaik untuk memahami bagaimana SSH bekerja adalah dengan melihat **penggunaan langsungnya dalam praktik**. Pada demonstrasi ini, SSH digunakan untuk mengendalikan sebuah mesin server dari komputer lain melalui jaringan.

Dalam skenario ini terdapat dua mesin yang berperan berbeda:

**Client**

Mesin yang digunakan untuk menjalankan perintah SSH.

- Sistem operasi: **Kali Linux**

**Server**

Mesin tujuan yang akan dikendalikan dari jarak jauh.

- Sistem: **OWASP Broken Web Application VM**
- Alamat IP: `172.17.199.151`

Tujuan dari demonstrasi ini adalah menunjukkan bahwa setelah koneksi SSH berhasil dibuat, kita dapat **menjalankan perintah di server langsung dari terminal client**, seolah-olah kita sedang menggunakan mesin server secara langsung.

#### 1. Menghubungkan Client ke Server

Langkah pertama adalah membuat koneksi SSH dari client ke server.

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa root@172.17.199.151
```

Perintah ini melakukan beberapa hal sekaligus.

**`ssh`**

Menjalankan program SSH client untuk membuat koneksi ke server.

**`root@172.17.199.151`**

Menunjukkan bahwa kita ingin login sebagai user **root** ke server dengan alamat IP `172.17.199.151`.

**`-o HostKeyAlgorithms=+ssh-rsa`**
**`-o PubkeyAcceptedAlgorithms=+ssh-rsa`**

Opsi ini menambahkan algoritma `ssh-rsa` ke daftar algoritma yang diperbolehkan oleh client. Hal ini biasanya diperlukan jika server menggunakan konfigurasi lama yang masih mengandalkan algoritma tersebut.

Jika koneksi berhasil dan autentikasi diterima, kita akan mendapatkan **prompt shell server** di terminal client.

Contoh prompt yang muncul:

```bash
root@server:~#
```

Pada titik ini, kita sebenarnya sudah berada di dalam sistem server.

#### 2. Memverifikasi Identitas Pengguna

Langkah berikutnya adalah memastikan kita login sebagai user yang benar.

Perintah yang digunakan:

```bash
whoami
```

Perintah `whoami` digunakan untuk menampilkan **nama pengguna yang sedang aktif di sistem**.

Output yang diharapkan:

```bash
root
```

Ini menunjukkan bahwa sesi SSH saat ini berjalan sebagai **user root di server**.

#### 3. Melihat Isi Direktori Server

Selanjutnya kita dapat melihat isi direktori saat ini menggunakan perintah:

```bash
ls
```

Perintah `ls` digunakan untuk menampilkan daftar file dan folder dalam direktori aktif.

Contoh hasilnya mungkin terlihat seperti:

```bash
index.html
config.php
logs
```

Yang penting untuk dipahami adalah bahwa daftar file tersebut **berasal dari server**, bukan dari komputer client.

#### 4. Membuat File Baru di Server

Sekarang kita akan membuat file baru di server menggunakan perintah:

```bash
touch test.txt
```

Perintah `touch` digunakan untuk:

- membuat file kosong baru
- atau memperbarui timestamp file yang sudah ada

Setelah perintah ini dijalankan, server akan memiliki file baru bernama:

```bash
test.txt
```

#### 5. Menambahkan Konten ke File

Setelah file dibuat, kita dapat menambahkan teks ke dalamnya menggunakan perintah berikut:

```bash
echo "ini adalah tes" >> test.txt
```

Penjelasan perintah ini:

**`echo "ini adalah tes"`**

Perintah `echo` digunakan untuk mencetak teks.

**`>> test.txt`**

Operator `>>` digunakan untuk **menambahkan output ke dalam file** tanpa menghapus isi sebelumnya.

Setelah perintah ini dijalankan, file `test.txt` akan berisi teks:

```bash
ini adalah tes
```

#### 6. Memverifikasi Isi File

Untuk memastikan bahwa file benar-benar berisi teks tersebut, kita dapat membaca isinya dengan perintah:

```bash
cat test.txt
```

Perintah `cat` digunakan untuk menampilkan isi file ke terminal.

Output yang muncul:

```bash
ini adalah tes
```

Ini menunjukkan bahwa perubahan yang kita lakukan melalui terminal client **langsung diterapkan di server**.

Jika seseorang membuka file tersebut langsung dari mesin server, hasilnya akan sama.

#### 7. Keluar dari Sesi SSH

Setelah selesai bekerja di server, kita dapat menutup koneksi SSH dengan perintah:

```bash
exit
```

Perintah ini akan:

1. Menutup session SSH.
2. Mengakhiri koneksi antara client dan server.
3. Mengembalikan kita ke terminal mesin lokal (client).

Prompt terminal biasanya akan kembali ke tampilan seperti:

```bash
kali@kali:~$
```

#### Kesimpulan dari Demonstrasi

Demonstrasi ini menunjukkan konsep utama dari SSH: **mengendalikan mesin jarak jauh secara aman melalui terminal**.

Semua perintah yang diketik di terminal client sebenarnya **dikirim melalui koneksi SSH ke server**, kemudian:

1. Server mengeksekusi perintah tersebut.
2. Hasilnya dikirim kembali ke client.
3. Terminal client menampilkan outputnya.

Proses ini terjadi **secara real-time** dan seluruh komunikasi dilindungi oleh **enkripsi SSH**, sehingga aman dari penyadapan jaringan.

Inilah yang membuat SSH menjadi alat yang sangat penting dalam dunia:

- administrasi server
- DevOps
- keamanan sistem
- pengelolaan infrastruktur jarak jauh.

## 3. Hubungan Antar Konsep

Seluruh konsep dalam materi ini membentuk alur pemahaman yang saling terkait:

```
Definisi SSH
    │
    ├── Port 22 (TCP) → transport layer
    │
    ├── CIA (Confidentiality, Integrity, Authentication)
    │       └── Dipenuhi oleh: enkripsi, MAC, dan mekanisme autentikasi
    │
    └── OpenSSH (Implementasi)
            ├── ssh (client)
            └── sshd (server daemon)
                    │
                    └── Proses Koneksi
                            ├── TCP Handshake
                            ├── Key Exchange & Negosiasi Algoritma ← [bisa gagal jika tidak cocok]
                            ├── Enkripsi tunnel
                            ├── User Authentication (password / public key)
                            └── Sessions & Channels (shell, exec, SFTP)
```

Dengan kata lain: **SSH bukan sekadar "remote terminal"** — ia adalah sistem keamanan berlapis yang menjamin privasi, integritas, dan identitas semua pihak yang berkomunikasi.

---

## 4. Kesimpulan

- **SSH (Secure Shell)** adalah protokol esensial untuk administrasi sistem jarak jauh yang aman, berjalan di **TCP port 22**.
- SSH menjamin **kerahasiaan** (enkripsi), **integritas** (MAC), dan **autentikasi** (password/public key).
- Implementasi paling umum adalah **OpenSSH**, menggunakan `ssh` di sisi client dan `sshd` di sisi server.
- Di balik satu perintah sederhana, SSH menjalankan proses kompleks: TCP handshake → key exchange → enkripsi → autentikasi user → pembukaan session & channel.
- Memahami proses ini penting agar bisa **mendiagnosis error** (seperti ketidakcocokan algoritma) dan memahami **risiko keamanan** yang mungkin muncul.
- SSH juga berfungsi sebagai **vektor serangan** yang diketahui oleh penyerang, sehingga penting untuk menguasainya dari kedua perspektif: administrasi **dan** keamanan.

---

> 📌 **Preview Part 2:** Video berikutnya akan membahas SSH ke mesin Windows, manipulasi aturan firewall, konfigurasi server untuk menerima koneksi SSH, serta penanganan error yang lebih dalam.
