# 📝 SSH – Bagian 2: File Structure, SSH Server, dan SCP Protocol

## 1. Ringkasan Singkat

Video ini adalah lanjutan dari materi SSH (Secure Shell) bagian pertama. Topik yang dibahas meliputi:

- Struktur file konfigurasi SSH di Linux
- Cara mengatur dan menjalankan SSH server di Kali Linux
- Cara melakukan koneksi SSH dari Windows (PowerShell) ke mesin Linux
- Pengenalan dan praktik SCP (Secure Copy Protocol) untuk transfer file antar mesin

---

## 2. Konsep Utama

### a. Struktur File SSH di Linux

Dalam sistem Linux, **SSH (Secure Shell)** menyimpan berbagai file konfigurasi dan kunci keamanan di beberapa lokasi khusus. Secara umum, ada dua tempat utama yang digunakan:

1. **System-wide files** → konfigurasi global yang berlaku untuk seluruh pengguna di sistem.
2. **User-specific files** → konfigurasi dan kunci yang hanya berlaku untuk satu pengguna tertentu.

Memahami struktur ini penting karena ketika kita mengatur SSH — baik sebagai **client** maupun **server** — kita akan sering berinteraksi dengan file-file ini.

#### 1. System-Wide Files (Berlaku untuk Semua Pengguna)

File jenis ini berada di direktori:

```
/etc/ssh/
```

Direktori `/etc` pada Linux biasanya digunakan untuk menyimpan **file konfigurasi sistem**. Karena itu, folder `/etc/ssh/` berisi konfigurasi SSH yang memengaruhi seluruh sistem.

File di sini biasanya **dikelola oleh administrator sistem (root)**.

Untuk melihat isi folder ini, kita bisa menggunakan perintah berikut:

```bash
ls /etc/ssh/
```

Contoh outputnya kira-kira seperti ini:

```
ssh_config  sshd_config  ssh_host_rsa_key  ssh_host_rsa_key.pub
```

Di dalam folder tersebut terdapat beberapa file penting.

| File          | Fungsi                                                                                                       |
| ------------- | ------------------------------------------------------------------------------------------------------------ |
| `sshd_config` | File konfigurasi untuk **SSH server** (daemon). Mengatur bagaimana server menerima koneksi SSH.              |
| `ssh_config`  | File konfigurasi untuk **SSH client**. Mengatur perilaku saat pengguna melakukan koneksi SSH ke server lain. |

Mari kita pahami sedikit perbedaannya.

**`sshd_config`**

File ini digunakan oleh **sshd (SSH daemon)**, yaitu program yang berjalan di background untuk menerima koneksi SSH dari komputer lain.

Beberapa hal yang biasanya diatur di file ini antara lain:

- Port SSH (default: 22)
- Apakah **root login diperbolehkan**
- Metode autentikasi (password atau key)
- Pengguna mana saja yang boleh login

Contoh isi konfigurasi:

```
Port 22
PermitRootLogin no
PasswordAuthentication yes
```

Artinya:

- SSH berjalan di port **22**
- Login sebagai **root tidak diperbolehkan**
- Autentikasi menggunakan **password diizinkan**

**`ssh_config`**

Berbeda dengan sebelumnya, file ini digunakan ketika mesin kita **bertindak sebagai client**, yaitu ketika kita menjalankan perintah seperti:

```bash
ssh user@server
```

File ini mengatur perilaku koneksi, misalnya:

- port default
- metode autentikasi
- timeout koneksi
- penggunaan identity file

Contoh konfigurasi:

```
Host *
    ForwardAgent no
    ServerAliveInterval 60
```

Artinya:

- konfigurasi ini berlaku untuk semua host (`*`)
- agent forwarding dimatikan
- client akan mengirim sinyal keep-alive setiap **60 detik**

Dengan kata lain, **file di `/etc/ssh/` adalah konfigurasi global yang mempengaruhi seluruh sistem**.

#### 2. User-Specific Files (Khusus Per Pengguna)

Selain konfigurasi global, SSH juga menyimpan file yang **khusus untuk setiap pengguna**.

Lokasinya berada di:

```
~/.ssh/
```

Tanda `~` berarti **home directory pengguna saat ini**.

Contohnya:

```
/home/username/.ssh/
```

Folder `.ssh` diawali dengan tanda titik (`.`), yang berarti **folder tersembunyi (hidden)** di Linux.

Untuk melihatnya kita harus menggunakan opsi `-a` pada perintah `ls`.

Contoh langkahnya:

```bash
cd ~
ls -la
```

Outputnya akan memperlihatkan folder `.ssh`.

Masuk ke folder tersebut:

```bash
cd .ssh
ls
```

Contoh isi foldernya:

```
known_hosts  authorized_keys  id_rsa  id_rsa.pub
```

Di dalam folder ini terdapat beberapa file penting.

| File              | Fungsi                                                                      |
| ----------------- | --------------------------------------------------------------------------- |
| `known_hosts`     | Daftar server yang pernah kita hubungi dan sudah diverifikasi identitasnya. |
| `authorized_keys` | Daftar public key yang diizinkan untuk login ke akun ini.                   |
| `id_rsa`          | Private key milik pengguna.                                                 |
| `id_rsa.pub`      | Public key milik pengguna.                                                  |

Mari kita pahami peran masing-masing file tersebut.

**`known_hosts`**

File ini berisi daftar **fingerprint server** yang pernah kita hubungi melalui SSH.

Ketika kita pertama kali terhubung ke sebuah server, SSH biasanya menampilkan pesan seperti:

```
The authenticity of host 'server.example.com' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

Jika kita menjawab **yes**, maka fingerprint server tersebut akan disimpan di file `known_hosts`.

Tujuannya adalah untuk **mencegah serangan Man-in-the-Middle**. Jika suatu saat fingerprint server berubah, SSH akan memberi peringatan.

**`authorized_keys`**

File ini digunakan pada **server** untuk menentukan **public key mana saja yang diperbolehkan login ke akun tersebut**.

Contohnya isi file:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... user@laptop
```

Jika public key seseorang ada di file ini, maka dia bisa login menggunakan private key pasangannya tanpa perlu password.

Ini adalah dasar dari **SSH key authentication**.

**`id_rsa`**

File ini adalah **private key**, yaitu kunci rahasia milik pengguna.

Private key digunakan oleh SSH client untuk membuktikan identitas saat login ke server.

Karena sifatnya rahasia:

- file ini **tidak boleh dibagikan**
- biasanya memiliki permission ketat seperti:

```
600
```

**`id_rsa.pub`**

Ini adalah **public key** yang merupakan pasangan dari private key.

Public key boleh dibagikan dan biasanya akan ditempatkan di file:

```
~/.ssh/authorized_keys
```

di server yang ingin kita akses.

Contoh alur penggunaannya:

1. Kita membuat key pair dengan `ssh-keygen`
2. File `id_rsa` tetap di komputer kita
3. File `id_rsa.pub` kita kirim ke server
4. Server menyimpannya di `authorized_keys`
5. Kita bisa login tanpa password

#### Catatan Penting

Perlu diingat perbedaan peran antara client dan server dalam SSH:

- Saat komputer kita **menghubungi server**, SSH menggunakan **`ssh_config`**.
- Saat komputer kita **menerima koneksi SSH**, SSH menggunakan **`sshd_config`**.

Dengan memahami struktur file ini, kita bisa lebih mudah:

- mengatur keamanan SSH
- melakukan konfigurasi koneksi
- mengelola autentikasi berbasis key

Semua konfigurasi tersebut pada akhirnya selalu berkaitan dengan dua lokasi utama:

```
/etc/ssh/   → konfigurasi global sistem
~/.ssh/     → konfigurasi dan kunci milik pengguna
```

### b. Menyiapkan SSH Server di Linux (Kali Linux)

Agar sebuah mesin Linux bisa menerima koneksi SSH dari komputer lain, kita perlu menjalankan **SSH server**. Di Kali Linux, server ini biasanya menggunakan implementasi dari **OpenBSD Secure Shell Server**, yang secara praktik digunakan melalui paket **OpenSSH**.

Dengan mengaktifkan SSH server, komputer kita bisa diakses secara remote menggunakan perintah seperti:

```bash
ssh user@ip-address
```

Berikut langkah-langkah untuk menyiapkan dan menjalankan SSH server di Kali Linux.

#### 1. Memastikan Paket OpenSSH Server Terinstal

Langkah pertama adalah memastikan bahwa paket **OpenSSH server** sudah terpasang di sistem.

Kita bisa menginstalnya menggunakan `apt`, yaitu package manager pada sistem berbasis Debian seperti Kali Linux.

```bash
apt install openssh-server
```

Jika paket belum terpasang, sistem akan mengunduh dan menginstalnya dari repository Kali Linux.

Namun jika paket tersebut sudah ada di sistem, biasanya output akan terlihat seperti ini:

```
openssh-server is already the newest version
0 upgraded, 0 newly installed, 0 to remove
```

Artinya:

- Paket **openssh-server sudah terpasang**
- Tidak ada paket baru yang perlu diinstal
- Sistem siap menjalankan SSH server

Setelah paket tersedia, langkah berikutnya adalah menjalankan servicenya.

#### 2. Menjalankan (Start) Service SSH

Di Linux modern, service biasanya dikelola oleh **systemd**, yaitu sistem manajemen service dan proses.

Untuk menjalankan SSH server, kita bisa menggunakan salah satu dari dua perintah berikut.

Menggunakan perintah `service`:

```bash
service ssh start
```

Atau menggunakan `systemctl` (cara yang lebih modern):

```bash
systemctl start ssh
```

Kedua perintah tersebut memiliki tujuan yang sama, yaitu:

- Menjalankan **SSH daemon (`sshd`)**
- Membuat server mulai **mendengarkan koneksi SSH dari jaringan**

Setelah service berjalan, komputer kita sudah siap menerima koneksi SSH dari perangkat lain di jaringan.

Sebagai contoh, dari komputer lain kita bisa mencoba:

```bash
ssh kali@192.168.1.10
```

Jika konfigurasi benar, maka kita akan diminta memasukkan password pengguna tersebut.

#### 3. Mengecek Status Service SSH

Setelah menjalankan service, langkah yang baik adalah memastikan bahwa SSH benar-benar berjalan.

Kita bisa memeriksa statusnya menggunakan perintah berikut:

```bash
systemctl status ssh
```

Contoh output yang biasanya muncul:

```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service)
     Active: active (running)
```

Hal penting yang perlu diperhatikan adalah bagian ini:

```
Active: active (running)
```

Ini berarti:

- Service SSH **berhasil dijalankan**
- Server sedang aktif
- Sistem siap menerima koneksi SSH

Biasanya output status juga menunjukkan bahwa SSH server sedang **mendengarkan pada port 22**, yang merupakan port default SSH.

Jika kita ingin melihat port yang digunakan, kita juga bisa menggunakan perintah seperti:

```bash
ss -tulpn | grep ssh
```

Outputnya bisa terlihat seperti:

```
LISTEN 0 128 0.0.0.0:22
```

Artinya server SSH sedang **listening pada port 22** untuk koneksi masuk.

#### Catatan Penting

Di Kali Linux, implementasi server yang digunakan adalah **OpenBSD Secure Shell Server**. Walaupun demikian, pengguna biasanya tetap berinteraksi melalui paket **OpenSSH**, sehingga cara instalasi dan penggunaannya tetap sama seperti di distribusi Linux lainnya.

Dengan kata lain:

- Kita menginstal **openssh-server**
- Service yang berjalan adalah **sshd**
- Server tersebut berbasis **OpenBSD Secure Shell**

Bagi pengguna, perbedaannya hampir tidak terasa karena **perintah dan konfigurasi yang digunakan tetap sama**.

### c. Koneksi SSH dari Windows ke Linux

Setelah **SSH server aktif di mesin Linux**, langkah berikutnya adalah mencoba melakukan koneksi dari perangkat lain. Salah satu cara yang paling umum adalah menghubungkan **Windows ke Linux menggunakan SSH**.

Pada Windows modern (Windows 10 dan Windows 11), SSH client sudah tersedia secara bawaan di **PowerShell**, sehingga kita tidak perlu menginstal aplikasi tambahan seperti PuTTY.

#### Menghubungkan Windows ke Linux Menggunakan SSH

Sintaks dasar untuk melakukan koneksi SSH adalah:

```powershell
ssh username@ip_address
```

Artinya:

- **username** → nama pengguna di server Linux
- **ip_address** → alamat IP dari server Linux yang menjalankan SSH

Contoh penggunaan:

```powershell
ssh kali@172.26.207.156
```

Penjelasan:

- `kali` adalah **username di mesin Linux**
- `172.26.207.156` adalah **alamat IP server Linux**

Jika koneksi berhasil, PowerShell akan mencoba membuka sesi terminal ke server tersebut.

#### Proses Koneksi Pertama Kali

Saat kita **pertama kali terhubung ke server SSH**, sistem akan menampilkan pesan keamanan. Pesan ini muncul karena komputer kita **belum mengenali identitas server tersebut**.

Contoh pesan yang muncul:

```
The authenticity of host '172.26.207.156' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxx
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.26.207.156' to the list of known hosts.
```

Mari kita pahami bagian-bagian dari pesan ini.

**1. Authenticity of host can't be established**

Pesan ini berarti komputer kita **belum pernah terhubung ke server tersebut sebelumnya**, sehingga SSH tidak bisa memastikan bahwa server itu benar-benar server yang kita maksud.

Ini adalah mekanisme keamanan untuk mencegah serangan **Man-in-the-Middle**.

**2. Fingerprint key**

SSH kemudian menampilkan **fingerprint dari host key**.

Contohnya:

```
ED25519 key fingerprint is SHA256:xxxxxxxxx
```

Penjelasan:

- **ED25519** → jenis algoritma kriptografi yang digunakan
- **SHA256** → metode hashing yang digunakan untuk menghasilkan fingerprint
- fingerprint ini adalah **identitas unik dari server SSH**

Administrator biasanya dapat membandingkan fingerprint ini untuk memastikan bahwa server yang kita hubungi benar.

**3. Konfirmasi koneksi**

SSH akan bertanya:

```
Are you sure you want to continue connecting (yes/no)?
```

Jika kita yakin bahwa server tersebut benar, kita bisa mengetik:

```
yes
```

Setelah itu, SSH akan menampilkan pesan:

```
Warning: Permanently added '172.26.207.156' to the list of known hosts.
```

Artinya server tersebut akan disimpan dalam file:

```
~/.ssh/known_hosts
```

Pada koneksi berikutnya, SSH akan memeriksa fingerprint server terhadap data yang tersimpan di file tersebut. Jika fingerprint berubah, SSH akan memberikan peringatan keamanan.

#### Setelah Koneksi Berhasil

Setelah proses autentikasi selesai (biasanya dengan password atau SSH key), kita akan mendapatkan **akses terminal ke mesin Linux** langsung dari PowerShell.

Artinya kita bisa menjalankan berbagai perintah Linux seperti biasa.

Contohnya:

```bash
whoami
```

Perintah ini digunakan untuk melihat **username yang sedang aktif di sistem**.

Outputnya bisa seperti:

```
kali
```

Contoh lain:

```bash
pwd
```

Perintah ini menunjukkan **direktori kerja saat ini (present working directory)**.

Misalnya outputnya:

```
/home/kali
```

Kita juga bisa berpindah direktori:

```bash
cd /
```

Perintah ini akan membawa kita ke **root directory**, yaitu direktori paling atas dalam struktur filesystem Linux.

Jika kita ingin menghentikan sebuah service, kita bisa menggunakan `sudo` (dengan hak administrator):

```bash
sudo service mysql stop
```

Perintah ini akan **menghentikan service MySQL** jika kita memiliki hak akses yang sesuai.

Kita juga bisa membuat direktori baru:

```bash
mkdir testing_ssh
```

Perintah ini membuat folder baru bernama `testing_ssh`.

Kemudian membuat file kosong:

```bash
touch ssh_connection_test
```

Perintah `touch` digunakan untuk **membuat file baru** tanpa isi.

Selanjutnya kita bisa menulis isi ke file tersebut:

```bash
echo "Testing connection" > ssh_connection_test
```

Penjelasan:

- `echo` mencetak teks
- simbol `>` digunakan untuk **menuliskan output ke dalam file**

Setelah perintah dijalankan, file `ssh_connection_test` akan berisi:

```
Testing connection
```

Ini menunjukkan bahwa koneksi SSH benar-benar memberi kita **akses penuh ke shell Linux**, meskipun kita menjalankannya dari komputer Windows.

#### PowerShell vs Command Prompt

Di Windows terdapat dua terminal yang umum digunakan:

- **Command Prompt (CMD)**
- **PowerShell**

PowerShell memiliki keunggulan dalam hal tampilan output, karena biasanya menampilkan **warna (coloring)** pada teks tertentu. Hal ini membuat hasil perintah lebih mudah dibaca dibandingkan Command Prompt yang cenderung lebih sederhana.

Karena itu, ketika bekerja dengan SSH di Windows, **PowerShell sering menjadi pilihan yang lebih nyaman** untuk digunakan.

### d. SCP (Secure Copy Protocol)

Selain digunakan untuk membuka **remote terminal**, SSH juga dapat dimanfaatkan untuk **mentransfer file secara aman** antar komputer. Salah satu cara yang paling sederhana untuk melakukan hal ini adalah menggunakan **SCP (Secure Copy Protocol)**.

SCP adalah protokol yang dirancang khusus untuk menyalin file melalui jaringan dengan memanfaatkan **koneksi SSH yang terenkripsi**.

Beberapa karakteristik utama dari SCP:

- Berjalan **di atas protokol SSH**
- Menggunakan **port yang sama dengan SSH**, yaitu port **22**
- Seluruh proses transfer file **terenkripsi**, sehingga data tidak bisa dibaca oleh pihak lain di jaringan
- Dapat digunakan untuk memindahkan file **antar komputer yang berbeda**

Dengan kata lain, jika kita sudah bisa melakukan koneksi SSH ke sebuah server, biasanya kita juga bisa langsung menggunakan **SCP untuk mengirim atau mengambil file dari server tersebut**.

#### Sintaks Dasar SCP

Format dasar penggunaan SCP adalah:

```bash
scp <file_sumber> <username>@<ip_tujuan>:<path_tujuan>
```

Mari kita pahami bagian-bagian dari sintaks ini:

- **`scp`** → perintah untuk menjalankan Secure Copy Protocol
- **`<file_sumber>`** → file yang ingin dikirim
- **`<username>`** → nama pengguna di server tujuan
- **`<ip_tujuan>`** → alamat IP server tujuan
- **`<path_tujuan>`** → lokasi direktori di server tempat file akan disimpan

Secara sederhana, perintah ini berarti:

> Salin file dari komputer lokal ke komputer lain melalui koneksi SSH.

#### Contoh Praktik: Transfer File dari Windows ke Linux

Sebagai contoh, kita akan melakukan transfer file dari **Windows (PowerShell)** ke **Linux yang menjalankan SSH server**.

#### 1. Membuat File di Windows

Pertama, kita buat file sederhana di PowerShell.

```powershell
echo "Testing SCP Protocol" > scp_test.txt
```

Perintah ini akan membuat file bernama **`scp_test.txt`** dan langsung mengisi file tersebut dengan teks:

```
Testing SCP Protocol
```

Untuk memastikan isi file benar, kita bisa menggunakan perintah:

```powershell
type scp_test.txt
```

Outputnya akan menampilkan isi file:

```
Testing SCP Protocol
```

Jika output sesuai, berarti file sudah siap untuk ditransfer.

#### 2. Mengirim File Menggunakan SCP

Selanjutnya kita kirim file tersebut ke server Linux menggunakan perintah `scp`.

```powershell
scp scp_test.txt kali@172.26.207.156:~/
```

Penjelasan perintah ini:

- **`scp_test.txt`** → file yang akan dikirim
- **`kali`** → username di server Linux
- **`172.26.207.156`** → alamat IP server
- **`~/`** → direktori tujuan di server (home directory pengguna)

Setelah menekan **Enter**, sistem akan meminta password dari pengguna di server.

Contohnya:

```
kali@172.26.207.156's password:
```

Setelah password dimasukkan dengan benar, proses transfer akan dimulai. Jika berhasil, file akan langsung muncul di server Linux.

#### 3. Memverifikasi File di Sisi Linux

Sekarang kita cek apakah file tersebut benar-benar sudah sampai di server.

Pertama, lihat daftar file di home directory:

```bash
ls ~/ | grep scp
```

Perintah ini akan menampilkan file yang memiliki kata **scp** di namanya.

Contoh output:

```
scp_test.txt
```

Artinya file berhasil ditransfer.

Untuk memastikan isi file juga benar, kita bisa menggunakan perintah:

```bash
cat ~/scp_test.txt
```

Outputnya akan menampilkan isi file:

```
Testing SCP Protocol
```

Ini menandakan bahwa file berhasil dikirim dari **Windows ke Linux melalui SCP**.

#### Tips Penting Saat Menggunakan SCP

Salah satu hal yang perlu diperhatikan saat menggunakan SCP adalah **path tujuan**.

Dalam contoh sebelumnya kita menggunakan:

```bash
~/
```

Simbol `~` adalah **shortcut untuk home directory pengguna** di Linux.

Sebagai contoh:

```
~/ = /home/kali/
```

Ini membuat perintah lebih singkat dan mudah dibaca.

Namun ada hal penting yang perlu diperhatikan: **direktori tujuan harus sudah ada**.

Jika kita mencoba mengirim file ke direktori yang belum dibuat, SCP akan gagal.

Contohnya:

```bash
scp file.txt kali@server:/folder_baru/
```

Jika `folder_baru` belum ada di server, maka transfer tidak akan berhasil.

Karena itu, biasanya kita memastikan direktori sudah tersedia terlebih dahulu, misalnya dengan:

```bash
mkdir folder_baru
```

Setelah direktori dibuat, barulah kita melakukan transfer menggunakan SCP.

Dengan memahami cara kerja SCP, kita bisa dengan mudah melakukan **transfer file yang aman melalui SSH**, tanpa perlu menggunakan protokol tambahan seperti FTP. Ini sangat berguna saat bekerja dengan server Linux, terutama dalam proses deployment, backup file, atau pengelolaan data dari jarak jauh.

## 3. Hubungan Antar Konsep

```
[File Konfigurasi SSH]
        |
        ├── /etc/ssh/sshd_config  → Mengatur perilaku SSH SERVER
        ├── /etc/ssh/ssh_config   → Mengatur perilaku SSH CLIENT
        └── ~/.ssh/               → File kunci & known_hosts per pengguna
                                          |
                                          ↓
                              [SSH Server Aktif di Linux]
                               (port 22, service: sshd)
                                          |
                                          ↓
                        [Koneksi dari Windows via PowerShell]
                          ssh kali@<ip>  →  Host diverifikasi
                                              & ditambah ke known_hosts
                                          |
                                          ↓
                        [Akses Penuh ke Mesin Remote]
                         - Jalankan perintah Linux
                         - Kelola file & service
                                          |
                                          ↓
                        [Transfer File dengan SCP]
                         scp <file> user@ip:<path>
                         (berjalan di atas SSH, port 22)
```

Ketiga konsep — struktur file, SSH server, dan SCP — saling terhubung membentuk ekosistem SSH yang utuh: file konfigurasi menentukan **bagaimana** koneksi berjalan, SSH server memungkinkan **akses remote**, dan SCP memperluas kemampuan SSH untuk **transfer file** secara aman.

---

## 4. Kesimpulan

Pada materi SSH Bagian 2 ini, kita mempelajari:

1. **Struktur file SSH** terbagi menjadi dua: system-wide (`/etc/ssh/`) dan user-specific (`~/.ssh/`). Masing-masing memiliki peran yang berbeda untuk server maupun client.

2. **SSH server** di Linux (khususnya Kali Linux) dapat diaktifkan dengan menginstal paket `openssh-server` dan menjalankan service dengan `systemctl start ssh`. Server akan mendengarkan koneksi pada **port 22**.

3. **Koneksi dari Windows** ke Linux bisa dilakukan langsung melalui **PowerShell** dengan perintah `ssh username@ip`. Saat pertama kali terhubung, fingerprint host perlu dikonfirmasi dan akan disimpan di `known_hosts`.

4. **SCP (Secure Copy Protocol)** adalah cara mudah untuk mentransfer file antara dua mesin secara aman, karena menggunakan infrastruktur dan enkripsi yang sama dengan SSH. Sintaksnya: `scp <file> user@ip:<path_tujuan>`.

SSH bukan hanya tentang _remote login_, tapi juga tentang ekosistem pengelolaan akses jarak jauh yang aman, termasuk transfer file melalui SCP.
