# 📝 Recon-NG: Open Source Intelligence (OSINT) dengan Framework Reconnaissance

## 1. Ringkasan Singkat

Video ini membahas penggunaan **Recon-NG**, sebuah framework berbasis Python untuk melakukan _Open Source Intelligence_ (OSINT) secara otomatis. Materi mencakup cara instalasi Recon-NG di Kali Linux, konsep dasar OSINT dan informasi WHOIS, cara kerja _workspace_ dan _marketplace module_, serta dua praktik nyata: mengumpulkan data WHOIS dan menemukan subdomain beserta alamat IPv4 dari sebuah domain target (facebook.com) menggunakan API HackerTarget.

---

## 2. Konsep Utama

### a. Open Source Intelligence (OSINT)

Open Source Intelligence atau **OSINT** adalah teknik untuk mengumpulkan informasi dari sumber-sumber yang tersedia secara publik. Artinya, data yang diambil bukan hasil hacking atau akses ilegal, melainkan informasi yang memang bisa diakses oleh siapa saja melalui internet.

Dalam praktiknya, OSINT sering digunakan di bidang keamanan siber, terutama dalam proses **penetration testing**. Di sini, OSINT menjadi bagian dari tahap awal yang disebut _reconnaissance_ (pengintaian). Tujuan tahap ini adalah untuk memahami target sebanyak mungkin sebelum melakukan analisis atau pengujian lebih lanjut.

#### Peran OSINT dalam Reconnaissance

Sebelum melakukan serangan simulasi (ethical hacking), seorang pentester biasanya akan mengumpulkan informasi dasar seperti:

- Domain yang digunakan target
- Struktur jaringan (secara umum)
- Teknologi yang dipakai (misalnya framework atau server)
- Informasi karyawan atau organisasi

Semua ini dilakukan tanpa berinteraksi langsung dengan sistem target secara agresif. Jadi, OSINT bisa dibilang sebagai cara “mengintip dari luar” tanpa meninggalkan jejak yang mencurigakan.

#### Legalitas OSINT

Salah satu hal penting dari OSINT adalah sifatnya yang **legal**. Selama informasi yang dikumpulkan memang tersedia untuk publik, maka aktivitas ini tidak melanggar hukum. Namun, tetap perlu berhati-hati—cara penggunaan informasi tersebut setelah dikumpulkan bisa saja memiliki implikasi hukum jika disalahgunakan.

#### Contoh Sumber OSINT

Berikut beberapa sumber umum yang sering digunakan dalam OSINT:

- **Search engine** seperti Google atau Bing
  Digunakan untuk mencari informasi umum, dokumen publik, atau bahkan data sensitif yang tidak sengaja terekspos.

- **Media sosial** seperti LinkedIn atau Twitter
  Bisa memberikan insight tentang struktur organisasi, jabatan karyawan, hingga teknologi yang digunakan (misalnya dari postingan atau profil kerja).

- **Database WHOIS publik**
  Digunakan untuk mengetahui informasi kepemilikan domain, seperti nama registrant, email, dan tanggal pendaftaran.

- **DNS records**
  Memberikan informasi tentang konfigurasi domain, seperti alamat IP server, subdomain, dan layanan yang digunakan.

Dengan menggabungkan berbagai sumber ini, seorang analis bisa membangun gambaran yang cukup lengkap tentang target tanpa harus menyentuh sistemnya secara langsung.

### b. Recon-NG

**Recon-NG** adalah sebuah _framework reconnaissance_ berbasis Python yang dirancang untuk membantu proses pengumpulan data OSINT secara otomatis. Jika sebelumnya OSINT dilakukan secara manual (misalnya mencari satu per satu di Google atau media sosial), Recon-NG hadir untuk menyederhanakan dan mempercepat proses tersebut.

Framework ini sering dijuluki sebagai _“the Metasploit of reconnaissance”_. Maksudnya, Recon-NG memiliki konsep yang mirip dengan Metasploit Framework: modular, terstruktur, dan bisa diperluas dengan berbagai module sesuai kebutuhan. Jadi, alih-alih melakukan semuanya secara manual, kita cukup menjalankan module tertentu untuk mengumpulkan data spesifik.

#### Cara Kerja Recon-NG

Recon-NG bekerja dengan konsep utama:

- **Workspace** → tempat menyimpan proyek atau target yang sedang dianalisis
- **Module** → fitur atau script tertentu untuk mengumpulkan jenis informasi tertentu

Dengan pendekatan ini, semua hasil pengumpulan data akan tersimpan rapi dan bisa digunakan kembali tanpa harus mengulang dari awal.

Sebagai contoh alur sederhana:

1. Membuat workspace untuk target
2. Menambahkan domain atau keyword target
3. Menjalankan module (misalnya untuk mencari subdomain atau email)
4. Melihat hasil yang tersimpan secara otomatis di database internal

Pendekatan ini membuat proses reconnaissance menjadi jauh lebih sistematis dibanding metode manual.

#### Keunggulan Recon-NG

Beberapa alasan kenapa tool ini sering digunakan dalam penetration testing:

- **Menghemat waktu**
  Proses yang biasanya dilakukan manual bisa dijalankan otomatis hanya dengan beberapa perintah.

- **Terorganisir dengan baik**
  Semua data disimpan dalam workspace, sehingga mudah dilacak dan dianalisis kembali.

- **Modular dan fleksibel**
  Kita bisa memilih module sesuai kebutuhan, tanpa harus menjalankan semua fitur sekaligus.

- **Cocok untuk skenario profesional**
  Karena strukturnya rapi dan scalable, tool ini sering digunakan dalam workflow pentesting yang serius.

#### Instalasi di Kali Linux

Untuk mulai menggunakan Recon-NG di Kali Linux, langkah-langkahnya cukup sederhana:

```bash
# Update repository terlebih dahulu
sudo apt update

# Install Recon-NG
sudo apt install recon-ng

# Jalankan tool
recon-ng
```

#### Penjelasan Perintah

- `sudo apt update`
  Digunakan untuk memperbarui daftar package dari repository. Ini penting agar sistem mengetahui versi terbaru dari software yang akan diinstall.

- `sudo apt install recon-ng`
  Perintah untuk menginstall Recon-NG dari repository Kali Linux.

- `recon-ng`
  Menjalankan tool dan masuk ke dalam interactive shell milik Recon-NG.

Setelah menjalankan perintah terakhir, biasanya kamu akan masuk ke interface berbasis CLI yang memiliki prompt khusus, di mana kamu bisa mulai membuat workspace dan menjalankan berbagai module.

#### Catatan Penting

Jika saat instalasi terjadi error, biasanya penyebabnya adalah repository yang belum diperbarui. Oleh karena itu, pastikan selalu menjalankan:

```bash
sudo apt update
```

sebelum melakukan instalasi package baru. Ini adalah langkah kecil, tapi sering jadi penyebab utama kegagalan instalasi di Linux.

### c. WHOIS Information

**WHOIS** adalah sebuah protokol sekaligus database publik yang digunakan untuk menyimpan informasi terkait pendaftaran sebuah domain. Ketika seseorang membeli atau mendaftarkan domain (misalnya `example.com`), data pendaftaran tersebut akan dicatat dan bisa diakses melalui layanan WHOIS.

Dalam konteks keamanan siber dan OSINT, WHOIS menjadi salah satu sumber informasi yang sangat penting karena dapat memberikan gambaran awal tentang siapa yang memiliki sebuah domain dan bagaimana domain tersebut dikelola.

#### Informasi yang Bisa Didapat dari WHOIS

Melalui WHOIS, kita bisa menemukan berbagai informasi penting, seperti:

- **Lokasi pemilik domain**
  Memberikan gambaran geografis dari organisasi atau individu yang mendaftarkan domain.

- **Tanggal registrasi dan kedaluwarsa domain**
  Ini berguna untuk mengetahui:
  - Seberapa lama domain sudah aktif
  - Kapan domain akan habis masa berlakunya
    Informasi ini kadang digunakan untuk menilai kredibilitas (misalnya domain baru vs domain lama).

- **Informasi kontak**
  Bisa berupa:
  - Email
  - Nomor telepon
  - Alamat
    Namun, perlu diketahui bahwa saat ini banyak domain menggunakan fitur _privacy protection_, sehingga data ini bisa disamarkan.

- **Data teknis lainnya**
  Seperti nama registrar (penyedia layanan domain) dan informasi administratif lainnya.

#### Contoh Data WHOIS

Berikut contoh sederhana hasil data WHOIS:

```
Domain: facebook.com
Registrar: RegistrarNamaXYZ
Registered On: 1997-03-29
Expires On: 2031-03-30
Contact Email: domain@fb.com
Registrant Location: Menlo Park, CA, US
```

#### Penjelasan Hasil

Mari kita pahami satu per satu:

- **Domain: facebook.com**
  Menunjukkan nama domain yang sedang dicek.

- **Registrar: RegistrarNamaXYZ**
  Pihak yang menyediakan layanan pendaftaran domain tersebut.

- **Registered On: 1997-03-29**
  Tanggal pertama kali domain didaftarkan. Ini menunjukkan bahwa domain tersebut sudah sangat lama aktif.

- **Expires On: 2031-03-30**
  Tanggal kedaluwarsa domain. Jika tidak diperpanjang, domain bisa menjadi tidak aktif.

- **Contact Email: [domain@fb.com](mailto:domain@fb.com)**
  Email yang digunakan untuk kontak administratif domain.

- **Registrant Location: Menlo Park, CA, US**
  Lokasi pemilik atau organisasi yang mendaftarkan domain tersebut.

#### Peran WHOIS dalam OSINT

Dalam praktik OSINT, WHOIS sering digunakan sebagai langkah awal untuk:

- Mengidentifikasi pemilik domain
- Menemukan email yang bisa dijadikan pivot untuk pencarian lanjutan
- Mengetahui hubungan antar domain (misalnya domain lain dengan email registrant yang sama)

Dari satu data sederhana seperti email atau nama organisasi, seorang analis bisa melanjutkan pencarian ke sumber lain dan memperluas informasi yang didapat.

#### Legalitas Penggunaan WHOIS

WHOIS bersifat **publik**, sehingga siapa pun dapat mengakses informasi ini tanpa memerlukan izin khusus. Namun, tetap penting untuk menggunakan data tersebut secara etis dan sesuai hukum, terutama jika digunakan dalam konteks penetration testing atau investigasi keamanan.

### d. Workspace di Recon-NG

Dalam Recon-NG, **workspace** adalah konsep penting yang berfungsi sebagai lingkungan kerja terisolasi. Kalau dianalogikan, workspace ini mirip seperti _folder project_—tempat kamu menyimpan semua aktivitas dan hasil reconnaissance untuk satu target tertentu.

Dengan adanya workspace, kamu bisa mengerjakan beberapa target atau proyek secara terpisah tanpa takut datanya tercampur.

#### Fungsi Workspace

Setiap workspace memiliki:

- **Database sendiri**
  Semua data hasil pengumpulan (seperti domain, email, host, dll) akan disimpan di database khusus workspace tersebut.

- **Konfigurasi sendiri**
  Pengaturan atau parameter yang kamu gunakan di satu workspace tidak akan mempengaruhi workspace lain.

- **Isolasi antar proyek**
  Misalnya kamu sedang menganalisis dua domain berbeda, kamu bisa membuat dua workspace terpisah agar hasilnya tetap rapi dan mudah dianalisis.

Pendekatan ini sangat membantu dalam workflow profesional, karena data yang dikumpulkan biasanya cukup banyak dan kompleks.

#### Cara Membuat Workspace Baru

Untuk membuat workspace baru di Recon-NG, gunakan perintah berikut:

```
[recon-ng] > workspaces create whois-recon
```

#### Penjelasan Perintah

- `workspaces create`
  Digunakan untuk membuat workspace baru.

- `whois-recon`
  Nama workspace yang ingin dibuat. Biasanya nama ini disesuaikan dengan tujuan atau jenis reconnaissance yang dilakukan.

#### Apa yang Terjadi Setelah Perintah Dijalankan

Setelah menjalankan perintah tersebut:

- Workspace baru dengan nama `whois-recon` akan dibuat
- Recon-NG akan **langsung berpindah (switch)** ke workspace tersebut
- Semua aktivitas berikutnya (menambahkan target, menjalankan module, dll) akan tersimpan di workspace ini

Artinya, kamu tidak perlu melakukan langkah tambahan untuk masuk ke workspace—semuanya otomatis.

#### Kenapa Workspace Itu Penting

Tanpa workspace, semua hasil reconnaissance akan bercampur dalam satu tempat, yang bisa menyebabkan:

- Data sulit dianalisis
- Risiko kesalahan dalam interpretasi hasil
- Sulit melacak asal data

Dengan workspace, kamu bisa menjaga alur kerja tetap rapi, terstruktur, dan mudah dikembangkan, terutama ketika menangani banyak target sekaligus.

### e. Marketplace dan Module di Recon-NG

Salah satu kekuatan utama dari Recon-NG adalah sistem **marketplace** dan **module** yang membuat tool ini fleksibel dan bisa disesuaikan dengan kebutuhan.

Secara sederhana, marketplace di Recon-NG adalah tempat kumpulan module (fitur tambahan) yang bisa kamu install dan gunakan. Setiap module dibuat untuk tugas tertentu, jadi kamu tidak perlu melakukan semuanya secara manual.

#### Konsep Module di Recon-NG

**Module** adalah komponen inti yang menjalankan proses pengumpulan data. Setiap module biasanya fokus pada satu jenis informasi, misalnya:

- Mengambil data **WHOIS**
- Mencari **subdomain**
- Mengumpulkan **email** dari domain tertentu
- Mengidentifikasi **host** atau layanan terkait

Dengan pendekatan ini, kamu bisa memilih hanya module yang relevan dengan kebutuhanmu, tanpa harus menjalankan semua proses sekaligus.

Sebagai contoh:

- Jika kamu ingin mencari email dari domain → gunakan module khusus email
- Jika ingin melihat informasi domain → gunakan module WHOIS

#### Peran Marketplace

Marketplace berfungsi sebagai repositori tempat semua module tersedia. Dari sini, kamu bisa:

- Mengupdate daftar module terbaru
- Mencari module tertentu
- Menginstall module sesuai kebutuhan

Ini mirip seperti package manager di Linux atau npm di JavaScript—tinggal install apa yang kamu butuhkan.

#### Perintah Dasar Marketplace

Berikut beberapa perintah penting yang sering digunakan:

```bash id="7qqhhe"
# Update daftar modul yang tersedia
marketplace update

# Install semua modul sekaligus
marketplace install all

# Cari modul tertentu (contoh: modul whois)
marketplace search whois

# Install modul spesifik
marketplace install recon/domains-contacts/whois_pocs
```

#### Penjelasan Perintah

- `marketplace update`
  Digunakan untuk memperbarui daftar module dari repositori. Ini penting agar kamu mendapatkan module terbaru.

- `marketplace install all`
  Menginstall semua module yang tersedia di marketplace. Cocok jika kamu ingin langsung memiliki semua fitur.

- `marketplace search whois`
  Mencari module yang berkaitan dengan kata kunci "whois". Ini membantu saat kamu tidak tahu nama module secara spesifik.

- `marketplace install recon/domains-contacts/whois_pocs`
  Menginstall satu module tertentu, dalam hal ini module untuk mencari kontak domain melalui WHOIS.

#### Tentang API Key pada Module

Perlu diperhatikan bahwa beberapa module membutuhkan **API key** dari layanan pihak ketiga, seperti:

- layanan pencarian data
- database email
- platform intelligence tertentu

Tanpa API key, module tersebut mungkin:

- Tidak bisa dijalankan
- Atau hasilnya terbatas

Biasanya, kamu perlu mendaftar ke layanan terkait untuk mendapatkan API key, lalu mengkonfigurasinya di Recon-NG.

#### Catatan Penting

Saat menjalankan:

```bash id="72zfna"
marketplace install all
```

mungkin akan muncul beberapa error. Ini normal, karena:

- Ada module yang membutuhkan API key
- Ada dependency yang belum terpenuhi

Error tersebut tidak perlu dikhawatirkan jika kamu memang tidak menggunakan module tersebut. Fokus saja pada module yang benar-benar kamu butuhkan agar workflow tetap efisien dan tidak terlalu kompleks.

### f. Praktik 1 — Mengumpulkan WHOIS Information

Pada praktik ini, kita akan menggunakan Recon-NG untuk mengumpulkan informasi **WHOIS** dari domain `facebook.com` secara otomatis. Tujuannya adalah memahami bagaimana workflow Recon-NG berjalan, mulai dari setup hingga eksekusi module.

Pendekatan ini jauh lebih efisien dibandingkan melakukan pencarian manual, karena hasilnya bisa langsung dikumpulkan, ditampilkan, dan disimpan dalam satu sistem.

#### Alur Besar Proses

Secara garis besar, langkah yang akan kita lakukan adalah:

1. Masuk ke Recon-NG
2. Membuat workspace untuk proyek
3. Mencari dan memilih module WHOIS
4. Menentukan target
5. Menjalankan module
6. Melihat hasil yang otomatis tersimpan

#### Langkah-langkah Praktik

Berikut perintah yang digunakan:

```bash id="lq6f4t"
# 1. Jalankan Recon-NG
recon-ng

# 2. Buat workspace
workspaces create whois-recon

# 3. Cari modul whois di marketplace
marketplace search whois

# 4. Load modul WHOIS ke workspace
modules load recon/domains-contacts/whois_pocs

# 5. Set domain target sebagai source
options set SOURCE facebook.com

# 6. Lihat konfigurasi modul (opsional)
info

# 7. Jalankan modul
run
```

#### Penjelasan Tiap Langkah

- `recon-ng`
  Membuka interface utama Recon-NG dalam mode interaktif.

- `workspaces create whois-recon`
  Membuat workspace baru bernama `whois-recon`. Semua hasil dari praktik ini akan disimpan di sini.

- `marketplace search whois`
  Mencari module yang berkaitan dengan WHOIS. Ini membantu kita menemukan module yang tepat.

- `modules load recon/domains-contacts/whois_pocs`
  Memuat module WHOIS ke dalam workspace agar bisa digunakan.

- `options set SOURCE facebook.com`
  Menentukan target domain yang ingin dianalisis. Dalam hal ini, kita menggunakan `facebook.com`.

- `info`
  (Opsional) Menampilkan informasi tentang module, seperti deskripsi, opsi yang tersedia, dan kebutuhan tambahan (misalnya API key).

- `run`
  Menjalankan module untuk mulai mengumpulkan data.

#### Apa yang Terjadi Saat Module Dijalankan

Ketika perintah `run` dijalankan:

- Recon-NG akan mengakses data WHOIS dari domain `facebook.com`
- Informasi seperti:
  - Email kontak
  - Lokasi registrant
  - Data terkait domain lainnya
    akan diproses dan ditampilkan di terminal

Selain itu, hal penting yang sering terlewat:

- Semua hasil tersebut akan **otomatis disimpan ke database workspace**

Artinya, kamu tidak perlu mencatat manual hasilnya. Data bisa digunakan kembali untuk analisis lanjutan atau dipakai oleh module lain.

#### Insight Penting

Dari praktik ini, ada beberapa hal yang bisa dipahami:

- Recon-NG bekerja secara **modular dan terstruktur**
- Setiap langkah memiliki tujuan yang jelas (setup → target → eksekusi)
- Data yang dikumpulkan bisa menjadi titik awal untuk eksplorasi lebih lanjut (misalnya pivot ke email atau domain lain)

Dengan memahami alur ini, kamu sudah punya fondasi kuat untuk menggunakan Recon-NG dalam skenario OSINT yang lebih kompleks.

### g. Praktik 2 — Subdomain Enumeration dengan HackerTarget API

Pada praktik ini, kita akan melangkah lebih jauh dari sekadar WHOIS, yaitu melakukan **subdomain enumeration**. Tujuannya adalah menemukan sebanyak mungkin subdomain dari `facebook.com` beserta alamat IP (IPv4) yang terkait.

Ini adalah langkah penting dalam reconnaissance, karena satu domain besar biasanya memiliki banyak subdomain yang masing-masing bisa menjalankan layanan berbeda. Dari sinilah sering ditemukan celah keamanan yang menarik untuk dianalisis lebih lanjut.

#### Mengenal HackerTarget

**HackerTarget** adalah layanan online yang menyediakan berbagai API untuk kebutuhan reconnaissance, salah satunya adalah pencarian subdomain.

Di dalam Recon-NG, layanan ini sudah tersedia dalam bentuk module, sehingga kita bisa langsung menggunakannya tanpa perlu mengakses API secara manual.

Module ini berada dalam kategori:

- `recon/domains-hosts` → artinya module ini berfungsi untuk memetakan domain menjadi daftar host dan alamat IP

#### Alur Proses

Secara umum, langkah yang dilakukan hampir sama seperti praktik sebelumnya:

1. Menentukan workspace
2. Mencari module yang sesuai
3. Load module ke dalam workspace
4. Menentukan target domain
5. Menjalankan module
6. Melihat hasil

#### Langkah-langkah Praktik

Berikut perintah yang digunakan:

```bash id="z3u8d3"
# 1. Pastikan berada di workspace yang sesuai
# (bisa menggunakan workspace yang sama atau buat baru)

# 2. Cari modul hackertarget
marketplace search hackertarget

# 3. Load modul
modules load recon/domains-hosts/hackertarget

# 4. Set domain target
options set SOURCE facebook.com

# 5. Verifikasi konfigurasi
info

# 6. Jalankan modul
run
```

#### Penjelasan Tiap Langkah

- `marketplace search hackertarget`
  Digunakan untuk mencari module yang berkaitan dengan layanan HackerTarget.

- `modules load recon/domains-hosts/hackertarget`
  Memuat module HackerTarget agar bisa digunakan di workspace.

- `options set SOURCE facebook.com`
  Menentukan target domain yang akan dianalisis.

- `info`
  Menampilkan informasi module, termasuk deskripsi dan konfigurasi yang digunakan.

- `run`
  Menjalankan proses enumeration untuk mengumpulkan data subdomain dan IP.

#### Output yang Dihasilkan

Setelah module dijalankan, kamu akan mendapatkan:

- **Daftar subdomain** dari `facebook.com`
- **Alamat IP (IPv4)** untuk masing-masing subdomain
- Informasi tambahan terkait hosting atau infrastruktur

Dalam contoh percobaan, ditemukan hingga **501 subdomain**, yang menunjukkan betapa luasnya infrastruktur sebuah domain besar.

Contoh output sederhana:

```
[*] cloud.facebook.com --> 157.240.0.1
[*] static.facebook.com --> 157.240.0.2
[*] m.facebook.com --> 157.240.0.3
...
[*] 501 total (hosts) found.
```

#### Analisis Hasil

Dari output tersebut, kita bisa mengambil beberapa insight:

- Setiap subdomain mengarah ke alamat IP tertentu → ini bisa digunakan untuk analisis lebih lanjut (misalnya scanning port)
- Banyaknya subdomain menunjukkan kompleksitas sistem target
- Informasi hosting (misalnya penggunaan cloud seperti Azure) bisa memberi petunjuk tentang arsitektur backend

#### Insight Penting

Subdomain enumeration adalah salah satu teknik paling powerful dalam OSINT dan penetration testing karena:

- Membuka “permukaan serangan” yang lebih luas
- Mengungkap layanan tersembunyi atau kurang terproteksi
- Menjadi titik awal untuk eksplorasi lanjutan (port scanning, vulnerability scanning, dll)

Dengan menggabungkan hasil dari praktik ini dan praktik sebelumnya (WHOIS), kamu sudah mulai membangun gambaran yang cukup lengkap tentang target tanpa harus melakukan interaksi langsung yang agresif ke sistemnya.

### h. Implikasi Keamanan dari Hasil OSINT

Setelah kita mengumpulkan data menggunakan teknik OSINT—seperti WHOIS, subdomain, dan alamat IP—langkah berikutnya adalah memahami **implikasi keamanannya**. Data ini bukan hanya sekadar informasi, tetapi bisa menjadi dasar untuk analisis yang lebih dalam, terutama dalam konteks keamanan siber.

#### Memperluas Permukaan Serangan (Attack Surface)

Salah satu dampak utama dari hasil OSINT adalah terbukanya **permukaan serangan (attack surface)** yang lebih luas.

- Semakin banyak subdomain yang ditemukan, semakin banyak titik yang bisa dianalisis
- Setiap subdomain bisa memiliki:
  - Server berbeda
  - Konfigurasi berbeda
  - Tingkat keamanan yang berbeda

Dari sudut pandang attacker, ini adalah peluang untuk menemukan celah di bagian yang mungkin kurang diperhatikan.

#### Subdomain sebagai Entry Point

Tidak semua subdomain dikelola dengan standar keamanan yang sama. Dalam praktik nyata:

- Subdomain utama (seperti `www`) biasanya sangat aman
- Namun subdomain lain (misalnya `dev`, `test`, `staging`) sering kali:
  - Kurang diperbarui
  - Memiliki konfigurasi yang lemah
  - Bahkan terkadang terekspos ke publik tanpa sengaja

Subdomain seperti ini bisa menjadi **entry point**, yaitu titik awal untuk masuk ke dalam sistem yang lebih besar.

#### Analisis dari Nama Subdomain

Menariknya, hanya dari **nama subdomain**, seorang analis sudah bisa mendapatkan banyak insight.

Contohnya:

- Subdomain dengan kata **"cloud"**
  Bisa mengindikasikan penggunaan layanan cloud

- Subdomain seperti:
  - `api.domain.com`
  - `dev.domain.com`
  - `cdn.domain.com`
    Bisa memberi petunjuk tentang:
  - Struktur aplikasi
  - Lingkungan pengembangan
  - Sistem distribusi konten

Dari sini, analis bisa memperkirakan teknologi atau arsitektur yang digunakan oleh target.

#### Mengidentifikasi Teknologi dan Potensi Kerentanan

Jika dari hasil OSINT diketahui bahwa target menggunakan layanan tertentu, misalnya:

- Cloud provider seperti AWS atau Azure
- Layanan CDN atau hosting tertentu

Maka langkah berikutnya adalah:

- Mencari **kerentanan spesifik** dari teknologi tersebut
- Contoh:
  - Misconfiguration pada cloud storage (seperti bucket yang terbuka)
  - Endpoint API yang tidak dilindungi dengan baik

Ini yang membuat OSINT sangat powerful—dari informasi publik, kita bisa mengarah ke potensi celah teknis yang lebih dalam.

#### Perspektif Pentesting yang Sah

Penting untuk ditekankan bahwa dalam konteks **penetration testing yang legal**, semua informasi ini digunakan untuk tujuan yang konstruktif, yaitu:

- Memetakan seluruh attack surface milik klien
- Mengidentifikasi potensi risiko keamanan
- Memberikan rekomendasi perbaikan

Jadi, tujuan utamanya bukan untuk mengeksploitasi, tetapi untuk **membantu meningkatkan keamanan sistem** secara menyeluruh.

Dengan memahami implikasi ini, kamu tidak hanya sekadar mengumpulkan data, tetapi juga mulai berpikir seperti seorang analis keamanan—menghubungkan informasi kecil menjadi gambaran besar tentang potensi risiko yang ada.

## 3. Hubungan Antar Konsep

Berikut adalah alur pemahaman dari keseluruhan materi:

```
OSINT (konsep dasar)
     │
     ▼
Recon-NG (tool otomasi OSINT)
     │
     ├─── Workspace (manajemen proyek)
     │         │
     │         └─── Setiap proyek memiliki database tersendiri
     │
     ├─── Marketplace (repositori modul)
     │         │
     │         ├─── Modul WHOIS → mengumpulkan info registrasi domain
     │         └─── Modul HackerTarget → menemukan subdomain + IP
     │
     └─── Output OSINT → dianalisis untuk menyusun strategi pentest
```

Alurnya adalah: **Pahami apa itu OSINT** → **Gunakan Recon-NG sebagai alat otomasi** → **Kelola pekerjaan dalam Workspace** → **Pilih dan load Module yang sesuai** → **Jalankan dan analisis hasilnya** → **Gunakan temuan untuk perencanaan pentest lebih lanjut**.

---

## 4. Kesimpulan

- **Recon-NG** adalah framework OSINT yang powerful dan modular, sering dibandingkan dengan Metasploit karena kemudahan penggunaannya untuk tugas reconnaissance.
- Alat ini memungkinkan pengumpulan informasi publik seperti **WHOIS**, **subdomain**, dan **alamat IP** secara otomatis dan terorganisir.
- Fitur **Workspace** membantu memisahkan berbagai proyek reconnaissance, sementara sistem **Marketplace** memudahkan instalasi dan pengelolaan modul.
- Dalam skenario penetration testing, data yang dikumpulkan dari Recon-NG menjadi fondasi penting untuk memahami infrastruktur target sebelum melanjutkan ke fase eksploitasi.
- Selalu gunakan tool ini **hanya pada target yang telah mendapatkan izin resmi** (_authorized targets_) untuk menghindari pelanggaran hukum.

---

> 📌 **Catatan Penting:** Semua aktivitas reconnaissance yang ditunjukkan dalam video bersifat edukatif dan dilakukan terhadap domain publik (facebook.com) menggunakan data yang memang tersedia untuk umum. Penggunaan tool ini tanpa izin terhadap sistem milik orang lain adalah **ilegal**.
