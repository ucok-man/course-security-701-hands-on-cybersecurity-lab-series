# 📝 Catatan Belajar: Perintah cURL – Bagian 2 (Penggunaan Nyata & Analisis Mendalam)

---

## 1. Ringkasan Singkat

Video ini merupakan kelanjutan dari pengenalan perintah `cURL`. Pada bagian kedua ini, materi difokuskan pada **penggunaan nyata (real-life scenarios)** dari `cURL`, termasuk cara menggunakan berbagai flag/opsi secara bersamaan, menyimpan response body dan headers ke file terpisah, memahami apa yang terjadi di balik layar saat perintah dijalankan, serta relevansi `cURL` dalam dunia keamanan siber.

---

## 2. Konsep Utama

### a. Anatomi Perintah cURL Lengkap

Pada bagian ini kita akan membedah sebuah perintah `cURL` secara lengkap untuk memahami bagaimana setiap opsi bekerja. Tujuannya adalah agar kita tidak hanya sekadar menjalankan perintah, tetapi benar-benar memahami fungsi dari tiap bagian yang ada di dalamnya.

Berikut contoh perintah `cURL` yang digunakan:

```bash
curl -sS -L -H "Accept: application/json" \
  -o /tmp/body.txt \
  -D /tmp/headers.txt \
  https://httpbin.org/get
```

Perintah ini terlihat cukup panjang, tetapi sebenarnya setiap bagian memiliki tugas yang jelas. Jika kita pecah satu per satu, akan jauh lebih mudah dipahami.

#### Struktur Dasar Perintah

Secara umum, struktur perintah `cURL` terdiri dari tiga bagian utama:

1. **Program utama** (`curl`)
2. **Opsi atau flag** (misalnya `-sS`, `-L`, `-H`)
3. **Target URL** yang ingin diakses

Dalam contoh di atas, URL yang diakses adalah:

```
https://httpbin.org/get
```

Website **httpbin** sering digunakan untuk belajar HTTP karena dapat menampilkan kembali informasi request yang kita kirimkan. Dengan begitu kita bisa melihat secara langsung apa yang terjadi saat request dilakukan.

Sekarang mari kita bahas setiap opsi yang digunakan.

#### Opsi `-sS` (Silent Mode + Show Errors)

Bagian pertama dari perintah adalah:

```
-sS
```

Flag ini sebenarnya adalah gabungan dari dua opsi:

- `-s` (**silent**) → menjalankan `curl` tanpa menampilkan progress bar atau output tambahan.
- `-S` (**show error**) → tetap menampilkan pesan error jika terjadi masalah.

Biasanya jika kita hanya menggunakan `-s`, semua output termasuk error akan disembunyikan. Dengan menambahkan `-S`, kita tetap bisa melihat error tanpa menampilkan informasi yang tidak perlu.

Kombinasi ini sering dipakai dalam **script otomatis** agar output tetap bersih tetapi error tetap terlihat.

#### Opsi `-L` (Follow Redirect)

Selanjutnya ada opsi:

```
-L
```

Opsi ini berarti **follow redirect**.

Kadang sebuah URL tidak langsung memberikan response final, tetapi mengarahkan kita ke URL lain melalui HTTP redirect (status code seperti `301` atau `302`). Jika `curl` tidak menggunakan `-L`, maka ia hanya akan menampilkan response redirect tersebut tanpa mengikuti alamat baru.

Dengan `-L`, `curl` akan otomatis mengikuti redirect sampai mendapatkan response akhir.

Contohnya:

```
URL A → redirect → URL B → redirect → URL C → response final
```

Dengan `-L`, `curl` akan terus mengikuti alur tersebut sampai selesai.

#### Opsi `-H` (HTTP Header)

Berikutnya kita melihat bagian ini:

```
-H "Accept: application/json"
```

Flag `-H` digunakan untuk **menambahkan HTTP header** ke dalam request.

Header yang dikirim di sini adalah:

```
Accept: application/json
```

Header `Accept` memberi tahu server bahwa client **lebih memilih response dalam format JSON**.

Jadi secara sederhana, kita sedang mengatakan kepada server:

> “Jika memungkinkan, kirimkan response dalam format JSON.”

Header seperti ini sering digunakan ketika bekerja dengan **REST API**, karena kebanyakan API modern menggunakan format JSON sebagai format data utama.

#### Opsi `-o` (Output File)

Selanjutnya ada opsi:

```
-o /tmp/body.txt
```

Flag `-o` digunakan untuk **menyimpan response body ke file**.

Dalam kasus ini, isi response dari server akan disimpan ke file:

```
/tmp/body.txt
```

Artinya, body dari response tidak akan ditampilkan di terminal, melainkan langsung ditulis ke file tersebut.

Ini sangat berguna ketika:

- response sangat besar
- kita ingin memprosesnya nanti
- kita ingin menyimpannya sebagai log atau data

Contoh isi file `body.txt` dari httpbin biasanya terlihat seperti ini:

```json
{
  "args": {},
  "headers": {
    "Accept": "application/json",
    "Host": "httpbin.org"
  },
  "origin": "123.123.123.123",
  "url": "https://httpbin.org/get"
}
```

Response ini menunjukkan informasi tentang request yang kita kirim.

#### Opsi `-D` (Dump Headers)

Berikutnya ada opsi:

```
-D /tmp/headers.txt
```

Flag `-D` digunakan untuk **menyimpan HTTP response headers ke file**.

Headers yang diterima dari server akan disimpan ke:

```
/tmp/headers.txt
```

Contoh isi file tersebut mungkin terlihat seperti ini:

```
HTTP/2 200
content-type: application/json
content-length: 256
server: gunicorn
date: Tue, 10 Oct 2023 10:00:00 GMT
```

Dengan menggunakan `-o` dan `-D`, kita memisahkan dua bagian penting dari response HTTP:

- **Body** → disimpan di `body.txt`
- **Headers** → disimpan di `headers.txt`

Pemisahan ini sering digunakan dalam scripting atau debugging HTTP.

#### URL Target Request

Bagian terakhir dari perintah adalah URL tujuan:

```
https://httpbin.org/get
```

Endpoint `/get` di httpbin akan mengembalikan informasi tentang request GET yang kita kirim. Ini membuatnya sangat cocok untuk mempelajari bagaimana `curl` bekerja.

Ketika perintah `curl` dijalankan, alurnya kira-kira seperti ini:

1. `curl` mengirim request ke `https://httpbin.org/get`
2. Request tersebut menyertakan header `Accept: application/json`
3. Jika terjadi redirect, `curl` akan mengikutinya karena ada `-L`
4. Server mengirim response
5. Response header disimpan di `/tmp/headers.txt`
6. Response body disimpan di `/tmp/body.txt`

Dengan memahami setiap bagian dari perintah ini, kita bisa mulai melihat bahwa `curl` bukan hanya sekadar alat untuk mengambil data dari URL, tetapi juga **tool yang sangat fleksibel untuk mengontrol request HTTP secara detail**.

### b. Flag `-s` dan `-S` (Silent Mode dengan Error Reporting)

Saat menggunakan `curl`, secara default terminal akan menampilkan berbagai informasi tambahan saat proses request berlangsung. Salah satunya adalah **progress meter**, yaitu indikator yang menunjukkan seberapa banyak data yang sudah diunduh, kecepatan transfer, dan estimasi waktu selesai.

Informasi ini memang berguna ketika kita menjalankan perintah secara manual. Namun dalam banyak kasus—terutama ketika `curl` digunakan di dalam **script otomatis**—output seperti ini justru membuat log menjadi penuh dan sulit dibaca. Di sinilah flag `-s` dan `-S` menjadi sangat berguna.

#### Flag `-s` (Silent Mode)

Flag pertama adalah:

```
-s
```

Huruf `s` di sini berarti **silent**. Ketika flag ini digunakan, `curl` akan menjalankan request **tanpa menampilkan progress meter**.

Artinya, informasi seperti:

- persentase data yang diunduh
- jumlah data yang sudah diterima
- kecepatan transfer
- estimasi waktu

tidak akan muncul di terminal.

Sebagai contoh, jika kita menjalankan `curl` tanpa `-s`, outputnya biasanya terlihat seperti ini:

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  1024    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
```

Ini adalah **progress bar** yang menunjukkan perkembangan proses unduhan.

Jika kita menambahkan `-s`, output tersebut tidak akan ditampilkan sehingga terminal menjadi jauh lebih bersih.

Namun ada satu hal penting yang perlu diperhatikan: ketika `-s` aktif, **pesan error juga ikut disembunyikan**. Ini bisa menjadi masalah, terutama jika request gagal dan kita tidak mengetahui penyebabnya.

#### Flag `-S` (Show Error)

Untuk mengatasi masalah tersebut, kita bisa menggunakan flag:

```
-S
```

Huruf `S` besar di sini berarti **show error**.

Flag ini memastikan bahwa **pesan error tetap ditampilkan**, meskipun mode silent sedang aktif.

Jadi fungsinya adalah:

- tetap menyembunyikan progress meter
- tetapi **jangan sembunyikan pesan error**

Misalnya jika request gagal karena domain tidak ditemukan atau koneksi bermasalah, `curl` masih akan menampilkan pesan seperti:

```
curl: (6) Could not resolve host: example.invalid
```

Dengan begitu kita tetap bisa mengetahui bahwa terjadi kegagalan.

#### Kombinasi `-sS`

Dalam praktiknya, kedua flag ini hampir selalu digunakan bersama seperti ini:

```
-sS
```

Kombinasi ini memberikan keseimbangan yang ideal:

- output menjadi **bersih tanpa progress meter**
- **error tetap terlihat** jika terjadi masalah

Inilah alasan mengapa `-sS` sering digunakan dalam **script otomatis**, misalnya:

- script deployment
- automation pipeline
- monitoring script
- CI/CD job

Jika script dijalankan berkali-kali, progress meter hanya akan membuat log menjadi sangat panjang dan sulit dibaca. Dengan `-sS`, log hanya akan berisi informasi penting saja.

#### Ilustrasi Perbandingan Output

Perbedaan perilaku ini bisa dilihat dari contoh berikut.

Tanpa menggunakan `-sS`, `curl` akan menampilkan progress meter:

```
% Total    % Received % Xferd  Average Speed ...
```

Output ini terus diperbarui selama proses download berlangsung.

Sebaliknya, jika kita menggunakan `-sS`, terminal tidak akan menampilkan progress meter sama sekali. Terminal hanya akan menampilkan output utama atau pesan error jika terjadi masalah.

Dengan kata lain, selama request berhasil, terminal akan terlihat **bersih tanpa noise tambahan**. Namun jika terjadi kegagalan, pesan error tetap muncul sehingga kita bisa langsung mengetahui apa yang salah.

### c. Flag `-L` (Follow Redirects)

Dalam komunikasi HTTP, sering kali sebuah URL tidak langsung memberikan respons akhir. Sebaliknya, server bisa mengarahkan client ke URL lain terlebih dahulu melalui mekanisme yang disebut **redirect**.

Redirect biasanya digunakan dalam berbagai situasi, misalnya:

- Mengalihkan dari `http://` ke versi yang lebih aman `https://`
- Mengarahkan ke domain baru setelah perubahan alamat website
- Mengubah path URL ke endpoint yang sebenarnya
- Mengarahkan pengguna ke halaman login sebelum mengakses resource tertentu

Secara teknis, redirect terjadi ketika server mengirim **HTTP status code** seperti:

- `301 Moved Permanently`
- `302 Found`
- `307 Temporary Redirect`
- `308 Permanent Redirect`

Ketika hal ini terjadi, server juga akan menyertakan header `Location` yang berisi URL tujuan berikutnya.

#### Perilaku Default cURL Tanpa `-L`

Secara default, `curl` **tidak mengikuti redirect secara otomatis**. Artinya, jika server mengirim respons redirect, `curl` hanya akan menampilkan respons tersebut tanpa melanjutkan ke alamat berikutnya.

Sebagai contoh:

```bash
curl http://example.com
```

Jika server mengalihkan request ke `https://example.com`, maka `curl` hanya akan menerima respons redirect, misalnya:

```
HTTP/1.1 301 Moved Permanently
Location: https://example.com
```

Pada titik ini, `curl` berhenti dan **tidak melanjutkan request ke URL baru**.

Dalam beberapa situasi—terutama ketika kita ingin mendapatkan **respons akhir dari server**—perilaku ini tentu kurang praktis.

#### Menggunakan `-L` untuk Mengikuti Redirect

Untuk mengatasi hal tersebut, kita bisa menggunakan flag:

```bash
-L
```

Flag ini berarti **follow redirects**. Dengan opsi ini, `curl` akan secara otomatis mengikuti URL yang diberikan dalam header `Location` sampai menemukan respons final.

Contohnya:

```bash
curl -L http://example.com
```

Alur yang terjadi kira-kira seperti ini:

1. `curl` mengirim request ke `http://example.com`
2. Server merespons dengan redirect ke `https://example.com`
3. Karena ada flag `-L`, `curl` membaca header `Location`
4. `curl` otomatis mengirim request baru ke `https://example.com`
5. Server memberikan respons akhir

Bagi pengguna, proses ini terlihat seperti satu request saja, padahal sebenarnya terjadi beberapa request di belakang layar.

#### Contoh Kasus Nyata Redirect

Redirect dari HTTP ke HTTPS adalah salah satu contoh paling umum. Banyak website modern memaksa semua koneksi menggunakan HTTPS demi keamanan.

Misalnya:

```bash
curl http://github.com
```

Tanpa `-L`, `curl` hanya akan menerima respons redirect ke:

```
https://github.com
```

Namun jika kita menggunakan:

```bash
curl -L http://github.com
```

`curl` akan langsung mengikuti redirect tersebut dan menampilkan **isi halaman sebenarnya dari GitHub**.

#### Mengapa Flag Ini Penting

Flag `-L` sangat penting ketika kita bekerja dengan:

- **API endpoint** yang menggunakan redirect
- **short URL** seperti link tracking atau URL shortener
- **website modern** yang memaksa HTTPS
- **script otomatis** yang membutuhkan respons final

Tanpa `-L`, script bisa saja berhenti di respons redirect dan tidak pernah mendapatkan data yang sebenarnya kita butuhkan.

Karena alasan ini, flag `-L` sangat sering digunakan bersama opsi lain dalam perintah `curl`, terutama ketika kita tidak yakin apakah endpoint tersebut akan melakukan redirect atau tidak.

### d. Flag `-I` (Request Headers Only / HEAD Request)

Dalam HTTP, ada beberapa jenis metode request yang bisa digunakan untuk berkomunikasi dengan server. Dua yang paling umum adalah **GET** dan **HEAD**.

Secara default, ketika kita menggunakan `curl`, request yang dikirim adalah **GET request**. Artinya server akan mengembalikan **dua hal sekaligus**:

1. **Response headers** — informasi metadata tentang respons
2. **Response body** — isi utama dari respons, misalnya HTML, JSON, atau file

Namun ada situasi tertentu di mana kita **tidak membutuhkan isi body sama sekali**. Kita hanya ingin melihat informasi metadata dari server. Untuk kebutuhan seperti ini, `curl` menyediakan flag `-I`.

#### Apa yang Dilakukan Flag `-I`

Flag berikut:

```bash
-I
```

membuat `curl` mengirim **HTTP HEAD request** alih-alih GET request.

HEAD request bekerja hampir sama seperti GET, tetapi dengan satu perbedaan penting: **server hanya mengirimkan headers tanpa body**.

Dengan kata lain, kita hanya mendapatkan informasi seperti:

- `Content-Type`
- `Content-Length`
- `Server`
- `Date`
- `Cache-Control`
- dan berbagai header lainnya

tanpa perlu mengunduh isi halaman atau data sebenarnya.

#### Contoh Penggunaan `-I`

Misalnya kita ingin melihat metadata dari sebuah endpoint:

```bash
curl -I https://httpbin.org/get
```

Output yang muncul biasanya terlihat seperti ini:

```
HTTP/2 200
content-type: application/json
content-length: 256
server: gunicorn
date: Tue, 10 Oct 2023 10:00:00 GMT
```

Mari kita pahami beberapa bagian dari output tersebut:

- **HTTP/2 200**
  Menunjukkan versi HTTP yang digunakan dan status code `200` (request berhasil).

- **content-type: application/json**
  Memberi tahu bahwa body respons (jika ada) berbentuk JSON.

- **content-length: 256**
  Ukuran body respons dalam byte.

- **server: gunicorn**
  Software server yang digunakan untuk melayani request.

Karena ini adalah **HEAD request**, isi JSON yang sebenarnya **tidak akan ditampilkan**.

#### Mengapa HEAD Request Berguna

HEAD request sangat berguna ketika kita ingin melakukan pengecekan cepat terhadap server tanpa harus mengunduh data besar.

Beberapa contoh penggunaan praktisnya:

- Mengecek apakah sebuah URL **masih aktif**
- Melihat **Content-Type** sebelum mengunduh file
- Mengetahui ukuran file melalui `Content-Length`
- Melakukan **monitoring server**
- Debugging HTTP response

Karena body tidak dikirim, request ini biasanya **lebih cepat dan lebih ringan** dibandingkan GET request.

#### Interaksi `-I` dengan `-o`

Ada satu hal penting yang perlu diperhatikan ketika menggunakan `-I` bersamaan dengan flag lain seperti `-o`.

Ingat bahwa `-o` digunakan untuk **menyimpan response body ke file**. Namun ketika kita menggunakan `-I`, sebenarnya **tidak ada body yang dikirim oleh server**.

Contohnya:

```bash
curl -I -o /tmp/body.txt https://httpbin.org/get
```

Sekilas terlihat seperti kita sedang menyimpan body ke `body.txt`, tetapi karena request yang dikirim adalah HEAD request, maka **yang tersimpan di file tersebut hanyalah headers**.

Isi file kemungkinan akan terlihat seperti ini:

```
HTTP/2 200
content-type: application/json
content-length: 256
server: gunicorn
```

Tidak ada JSON atau data lain di dalamnya.

#### Perbandingan Dengan dan Tanpa `-I`

Perbedaan perilaku ini bisa dilihat dengan jelas dari dua contoh berikut.

Jika kita menggunakan `-I`:

```bash
# Dengan -I → hanya headers yang tersimpan
curl -I -o /tmp/body.txt https://httpbin.org/get
```

File `body.txt` hanya berisi **headers**.

Sebaliknya, jika kita tidak menggunakan `-I`:

```bash
# Tanpa -I → body dan headers dipisahkan
curl -o /tmp/body.txt -D /tmp/headers.txt https://httpbin.org/get
```

Perintah ini melakukan GET request sehingga server mengirim respons lengkap.

Hasilnya:

- **Response body** (misalnya JSON atau HTML) akan disimpan di
  `/tmp/body.txt`
- **Response headers** akan disimpan di
  `/tmp/headers.txt`

Dengan memahami perbedaan ini, kita bisa memilih kapan harus menggunakan **HEAD request (`-I`)** untuk pengecekan cepat, dan kapan harus menggunakan **GET request** untuk mengambil data lengkap dari server.

### e. Flag `-H` (Custom HTTP Header)

Dalam komunikasi HTTP, setiap request yang dikirim oleh client ke server biasanya disertai dengan **HTTP headers**. Header ini berisi berbagai informasi tambahan yang membantu server memahami bagaimana request tersebut harus diproses.

Beberapa contoh informasi yang biasanya dikirim melalui header antara lain:

- jenis data yang diharapkan oleh client
- format data yang dikirim
- informasi autentikasi
- informasi tentang client (browser, aplikasi, dan sebagainya)

Secara default, `curl` sudah mengirimkan beberapa header otomatis. Namun dalam banyak kasus—terutama saat bekerja dengan **API modern**—kita perlu menambahkan header secara manual. Untuk tujuan inilah flag `-H` digunakan.

#### Fungsi Flag `-H`

Flag berikut digunakan untuk menambahkan header ke dalam request:

```bash
-H "Header-Name: value"
```

Dengan kata lain, `-H` memungkinkan kita untuk **menentukan HTTP header sendiri** saat mengirim request menggunakan `curl`.

Header yang kita tambahkan bisa digunakan untuk berbagai tujuan, seperti:

- menentukan format data
- mengirim token autentikasi
- mengatur cache
- memberi tahu server jenis konten yang dikirim

Jika sebuah request membutuhkan beberapa header, kita juga bisa menggunakan flag `-H` **lebih dari satu kali**.

#### Contoh Header `Accept`

Pada contoh yang dibahas sebelumnya terdapat header berikut:

```bash
-H "Accept: application/json"
```

Header `Accept` memberi tahu server bahwa client **lebih memilih respons dalam format tertentu**.

Dalam contoh ini, kita menyatakan bahwa format yang diharapkan adalah:

```
application/json
```

Artinya kita memberi sinyal kepada server:

> “Jika memungkinkan, kirimkan data dalam format JSON.”

Hal ini sangat umum ketika berinteraksi dengan **REST API**, karena hampir semua API modern menggunakan JSON sebagai format data utama.

Sebagai ilustrasi, ketika request dikirim ke endpoint seperti:

```
https://httpbin.org/get
```

server akan mencoba mengembalikan data dalam format JSON jika header `Accept: application/json` diberikan.

#### Contoh Header Authorization

Salah satu penggunaan `-H` yang paling sering ditemui adalah untuk **autentikasi API**.

Banyak API modern menggunakan sistem **Bearer Token**. Dalam sistem ini, client harus mengirim token pada header `Authorization`.

Contohnya:

```bash
curl -H "Authorization: Bearer TOKEN123" https://api.example.com/data
```

Header yang dikirim dalam request ini adalah:

```
Authorization: Bearer TOKEN123
```

Server kemudian akan memverifikasi token tersebut sebelum mengizinkan akses ke resource yang diminta.

Jika token tidak valid atau tidak dikirim, server biasanya akan mengembalikan status seperti:

```
401 Unauthorized
```

#### Contoh Header `Content-Type`

Header lain yang sering digunakan adalah `Content-Type`.

Header ini memberi tahu server **jenis data yang dikirim oleh client**.

Contohnya:

```bash
curl -H "Content-Type: application/json" -X POST https://api.example.com/items
```

Dalam contoh ini, kita mengirim request `POST` dan memberi tahu server bahwa **body request** akan berformat JSON.

Header yang dikirim adalah:

```
Content-Type: application/json
```

Biasanya header ini digunakan bersamaan dengan data JSON yang dikirim menggunakan opsi seperti `-d` atau `--data`.

Sebagai contoh yang lebih lengkap:

```bash
curl -H "Content-Type: application/json" \
     -X POST \
     -d '{"name":"Laptop","price":1200}' \
     https://api.example.com/items
```

Di sini server mengetahui bahwa data yang dikirim adalah JSON sehingga bisa diproses dengan benar.

#### Menggunakan Beberapa Header Sekaligus

Dalam praktik nyata, sebuah request sering kali membutuhkan lebih dari satu header. `curl` memungkinkan kita menambahkan banyak header dengan menggunakan beberapa flag `-H`.

Contohnya:

```bash
curl \
  -H "Authorization: Bearer TOKEN123" \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  https://api.example.com/data
```

Request ini mengirim tiga header sekaligus:

1. **Authorization** untuk autentikasi
2. **Accept** untuk menentukan format respons
3. **Content-Type** untuk menentukan format data yang dikirim

Dengan memahami cara kerja flag `-H`, kita bisa mengontrol request HTTP dengan jauh lebih fleksibel. Ini sangat penting ketika bekerja dengan **API, microservices, dan sistem backend modern** yang sangat bergantung pada konfigurasi header untuk menentukan perilaku request dan respons.

### f. Flag `-o` (Output ke File / Response Body)

Secara default, ketika kita menjalankan `curl`, hasil respons dari server akan langsung ditampilkan di terminal. Respons ini biasanya berupa **response body**, yaitu isi utama dari data yang dikirim server, misalnya HTML, JSON, atau file lainnya.

Hal ini cukup nyaman ketika respons yang diterima kecil. Namun jika datanya besar, atau jika kita ingin menyimpannya untuk diproses nanti, menampilkan semuanya di terminal tentu tidak praktis. Untuk itulah `curl` menyediakan flag `-o`.

#### Fungsi Flag `-o`

Flag berikut digunakan untuk menyimpan **response body** ke dalam file tertentu:

```bash
-o <path/file>
```

Dengan menggunakan opsi ini, `curl` tidak lagi menampilkan body respons di terminal. Sebagai gantinya, isi respons akan **langsung ditulis ke file yang kita tentukan**.

Ini sangat berguna ketika kita:

- ingin menyimpan hasil respons untuk dianalisis nanti
- ingin mengunduh file dari internet
- ingin memproses data menggunakan tools lain
- menjalankan `curl` dalam script otomatis

#### Contoh Penggunaan Dasar

Misalnya kita menjalankan perintah berikut:

```bash
curl -o /tmp/body.txt https://httpbin.org/get
```

Perintah ini melakukan request ke endpoint:

```
https://httpbin.org/get
```

Endpoint tersebut biasanya mengembalikan respons dalam format JSON yang berisi informasi tentang request yang kita kirim.

Karena kita menggunakan flag `-o`, maka JSON tersebut tidak akan muncul di terminal. Sebagai gantinya, respons akan disimpan ke file:

```
/tmp/body.txt
```

#### Melihat Isi File yang Disimpan

Setelah request selesai, kita bisa membuka file tersebut menggunakan perintah seperti:

```bash
cat /tmp/body.txt
```

Jika endpoint mengembalikan data JSON, isi file tersebut mungkin terlihat seperti ini:

```json
{
  "args": {},
  "headers": {
    "Host": "httpbin.org",
    "User-Agent": "curl/7.88.1"
  },
  "origin": "123.123.123.123",
  "url": "https://httpbin.org/get"
}
```

Dari sini kita bisa melihat berbagai informasi tentang request yang dikirim, seperti:

- header yang dikirim oleh `curl`
- alamat IP client (`origin`)
- URL yang diminta

Karena data disimpan di file, kita bisa dengan mudah:

- memeriksanya menggunakan editor teks
- memprosesnya menggunakan tool seperti `jq`
- menggunakannya sebagai input untuk script lain

#### Kapan `-o` Biasanya Digunakan

Flag `-o` sangat sering digunakan dalam berbagai situasi praktis, misalnya:

1. **Mengunduh file**

   ```bash
   curl -o image.png https://example.com/image.png
   ```

   File gambar akan langsung disimpan sebagai `image.png`.

2. **Menyimpan respons API**

   ```bash
   curl -o response.json https://api.example.com/users
   ```

   Respons JSON dari API disimpan ke file sehingga bisa dianalisis nanti.

3. **Digunakan dalam script**

   Banyak script otomatis menggunakan `-o` untuk menyimpan respons sebelum diproses oleh program lain.

#### Perilaku Tanpa `-o`

Jika kita tidak menggunakan flag `-o`, `curl` akan langsung mencetak response body ke terminal.

Contohnya:

```bash
curl https://httpbin.org/get
```

Output JSON akan langsung muncul di layar terminal.

Perilaku ini cocok untuk **pengujian cepat**, tetapi kurang ideal jika data yang diterima besar atau jika kita ingin menyimpannya.

Dengan memahami cara kerja flag `-o`, kita bisa mengontrol bagaimana `curl` menangani response body—apakah ditampilkan langsung di terminal atau disimpan sebagai file untuk digunakan lebih lanjut.

### g. Flag `-D` (Dump Headers ke File)

Saat `curl` melakukan request ke sebuah server, respons yang dikembalikan sebenarnya terdiri dari dua bagian utama:

1. **Response headers** — metadata tentang respons
2. **Response body** — isi utama dari respons (HTML, JSON, file, dan sebagainya)

Secara default, kedua bagian ini biasanya ditampilkan bersamaan di terminal. Namun dalam banyak kasus—terutama saat debugging HTTP atau bekerja dengan API—kita ingin **memisahkan headers dari body** agar lebih mudah dianalisis. Di sinilah flag `-D` menjadi sangat berguna.

#### Fungsi Flag `-D`

Flag berikut digunakan untuk menyimpan **response headers** ke dalam file tertentu:

```bash
-D <path/file>
```

Huruf `D` di sini berasal dari kata **dump**, yang berarti menyalin atau mengekspor data ke tempat lain.

Dengan menggunakan `-D`, semua **HTTP response headers** yang diterima dari server akan disimpan ke file yang kita tentukan. Sementara itu, response body bisa tetap ditampilkan di terminal atau disimpan menggunakan flag lain seperti `-o`.

#### Mengapa Perlu Menyimpan Headers Terpisah

Headers sering berisi informasi penting tentang bagaimana server merespons request kita. Beberapa contoh header yang sering muncul antara lain:

- `Content-Type` → jenis data yang dikirim (JSON, HTML, dll.)
- `Content-Length` → ukuran body respons
- `Server` → software server yang digunakan
- `Date` → waktu respons dikirim
- `Cache-Control` → aturan caching
- `Set-Cookie` → cookie yang dikirim oleh server

Jika headers tercampur dengan body, terutama ketika body berupa JSON besar atau HTML panjang, informasi ini bisa menjadi sulit dibaca.

Dengan memisahkannya ke file sendiri, kita bisa melakukan **inspeksi HTTP response dengan lebih jelas**.

#### Contoh Penggunaan `-D`

Misalnya kita menjalankan perintah berikut:

```bash
curl -o /tmp/body.txt -D /tmp/headers.txt https://httpbin.org/get
```

Perintah ini melakukan dua hal sekaligus:

1. Menyimpan **response body** ke file:

```
/tmp/body.txt
```

2. Menyimpan **response headers** ke file:

```
/tmp/headers.txt
```

Dengan kata lain, respons HTTP dipisahkan menjadi dua file yang berbeda.

#### Melihat Isi Headers yang Disimpan

Setelah request selesai, kita bisa membaca isi file headers menggunakan perintah:

```bash
cat /tmp/headers.txt
```

Isi file tersebut biasanya terlihat seperti ini:

```
HTTP/2 200
content-type: application/json
content-length: 256
server: gunicorn
date: Tue, 10 Oct 2023 10:00:00 GMT
```

Mari kita pahami beberapa bagian dari output ini:

- **HTTP/2 200**
  Menunjukkan versi HTTP yang digunakan dan status code respons (`200` berarti request berhasil).

- **content-type: application/json**
  Memberi tahu bahwa body respons berupa JSON.

- **content-length: 256**
  Menunjukkan ukuran body respons dalam byte.

- **server: gunicorn**
  Menunjukkan software server yang digunakan untuk melayani request.

- **date**
  Waktu ketika respons dikirim oleh server.

Informasi seperti ini sering digunakan untuk **debugging HTTP**, menganalisis API, atau memahami bagaimana server memproses request.

#### Kombinasi `-D` dan `-o`

Dalam praktik nyata, `-D` hampir selalu digunakan bersama dengan `-o`.

Contohnya:

```bash
curl -o body.json -D headers.txt https://api.example.com/users
```

Hasilnya:

- **body.json** berisi data JSON dari API
- **headers.txt** berisi metadata respons HTTP

Pendekatan ini sangat berguna ketika kita ingin:

- menganalisis response headers
- melihat status code dari server
- memeriksa cookie atau caching
- memisahkan data utama dari metadata HTTP

Dengan memahami flag `-D`, kita bisa melihat dengan jelas bagaimana server merespons request kita tanpa harus mencampurkan informasi headers dengan isi body respons. Ini menjadikan proses debugging dan analisis HTTP jauh lebih rapi dan mudah dipahami.

### h. URL Target: `httpbin.org`

Pada contoh-contoh sebelumnya, kita sering menggunakan URL berikut sebagai target request:

```
https://httpbin.org/get
```

URL ini bukan sekadar website biasa. Ia berasal dari sebuah layanan bernama **httpbin**, yang memang dibuat khusus untuk **menguji dan mempelajari cara kerja HTTP request**.

Server ini bersifat **publik, gratis, dan aman digunakan untuk eksperimen**. Karena itu, httpbin sering dipakai dalam tutorial, dokumentasi API, maupun latihan menggunakan tools seperti `curl`.

#### Apa Itu httpbin

`httpbin.org` adalah layanan yang menyediakan berbagai endpoint HTTP yang dirancang untuk **menampilkan kembali informasi dari request yang kita kirim**.

Dengan kata lain, server ini akan memberi tahu kita:

- header apa saja yang dikirim oleh client
- parameter apa yang dikirim
- alamat IP client
- URL yang diakses
- metode HTTP yang digunakan

Semua informasi tersebut biasanya dikembalikan dalam bentuk **JSON**.

Hal ini sangat membantu ketika kita sedang belajar atau ingin memahami **bagaimana request HTTP sebenarnya bekerja**.

#### Contoh Request ke httpbin

Misalnya kita menjalankan perintah berikut:

```bash
curl https://httpbin.org/get
```

Perintah ini mengirim request HTTP GET ke endpoint `/get`.

Server kemudian akan mengembalikan respons JSON yang berisi informasi tentang request tersebut. Contoh hasilnya mungkin terlihat seperti ini:

```json
{
  "args": {},
  "headers": {
    "Host": "httpbin.org",
    "User-Agent": "curl/7.88.1",
    "Accept": "*/*"
  },
  "origin": "123.123.123.123",
  "url": "https://httpbin.org/get"
}
```

Mari kita lihat beberapa bagian penting dari respons ini.

- **args**
  Berisi query parameter yang dikirim dalam URL. Jika tidak ada parameter, maka nilainya kosong.

- **headers**
  Menampilkan semua header yang dikirim oleh client, termasuk `User-Agent`, `Accept`, dan lainnya.

- **origin**
  Menunjukkan alamat IP client yang melakukan request.

- **url**
  Menampilkan URL lengkap yang diakses.

Dengan melihat respons ini, kita bisa memahami dengan jelas **apa yang sebenarnya dikirim oleh `curl` ke server**.

#### Mengapa httpbin Sangat Berguna untuk Belajar

Ketika mempelajari HTTP atau tools seperti `curl`, sering kali kita ingin melihat **efek langsung dari perubahan yang kita lakukan pada request**.

Misalnya:

- menambahkan header baru
- mengirim parameter
- mengubah metode request
- menambahkan body request

Dengan menggunakan httpbin, kita bisa langsung melihat perubahan tersebut pada respons JSON yang dikembalikan.

Contohnya, jika kita menambahkan header:

```bash
curl -H "Accept: application/json" https://httpbin.org/get
```

Header tersebut akan muncul di bagian `headers` dalam respons JSON. Ini memudahkan kita untuk memverifikasi apakah request yang kita kirim sudah sesuai dengan yang diharapkan.

#### Endpoint Lain yang Disediakan httpbin

Selain `/get`, httpbin juga menyediakan banyak endpoint lain yang berguna untuk pengujian HTTP, misalnya:

- `/post` → untuk menguji request POST
- `/headers` → menampilkan header yang diterima server
- `/ip` → menampilkan alamat IP client
- `/user-agent` → menampilkan user-agent yang digunakan
- `/status/{code}` → mengembalikan status code tertentu

Karena fitur-fitur ini, httpbin sering digunakan oleh developer untuk:

- belajar HTTP request
- menguji API client
- debugging request `curl`
- memahami bagaimana header dan parameter bekerja

Dengan kata lain, `httpbin.org` adalah **alat belajar yang sangat praktis** untuk memahami cara kerja komunikasi HTTP antara client dan server tanpa perlu membuat server sendiri.

## 3. Apa yang Terjadi di Balik Layar (Behind the Scenes)

Saat perintah `cURL` dijalankan, terjadi serangkaian proses jaringan secara berurutan:

### 🔍 1. DNS Resolution

- `cURL` meminta OS untuk menerjemahkan nama domain (misal: `httpbin.org`) menjadi alamat IP.

### 🤝 2. TCP Handshake

- `cURL` membuka koneksi TCP ke IP yang diperoleh, biasanya ke **port 443** (HTTPS).
- Proses **three-way handshake** terjadi: `SYN → SYN-ACK → ACK`.

### 🔒 3. TLS Handshake

- Karena menggunakan HTTPS, setelah TCP terbuka, dilakukan **TLS handshake** untuk enkripsi.
- Ini memastikan komunikasi antara client dan server bersifat terenkripsi dan aman.

### 📤 4. HTTP Request

- `cURL` mengirimkan **HTTP GET request** (atau HEAD jika menggunakan `-I`).
- Headers tambahan seperti `Accept: application/json` turut disertakan.

### 🔀 5. Redirect Handling

- Jika server merespons dengan kode redirect (301, 302, dll.) dan flag `-L` aktif, `cURL` secara otomatis mengikuti redirect tersebut.

### 📥 6. Response

- Server mengembalikan **headers + body**.
- `cURL` menulis headers ke file yang ditentukan oleh `-D`, dan body ke file yang ditentukan oleh `-o`.
- Jika tidak ada flag penyimpanan, semua ditampilkan di terminal.

### 🔌 7. Connection Handling

- Bergantung pada konfigurasi server, koneksi TCP bisa **dipertahankan** (_keep-alive_) untuk digunakan ulang, atau **ditutup** segera setelah respons diterima.

---

## 4. Hubungan Antar Konsep

Semua flag dalam perintah `cURL` bekerja **secara sinergis** untuk membentuk satu request HTTP yang lengkap dan terkontrol:

```
cURL
 ├── Kontrol Output  → -s, -S (diam/tampilkan error)
 ├── Navigasi        → -L (ikuti redirect)
 ├── Tipe Request    → -I (HEAD) atau default (GET)
 ├── Header Custom   → -H (modifikasi request header)
 ├── Simpan Body     → -o (ke file)
 ├── Simpan Headers  → -D (ke file)
 └── Target          → URL (endpoint yang dituju)
```

Alur pemahaman yang utuh:

1. Kamu **membangun request** dengan header dan method yang tepat.
2. `cURL` **mengirim request** melalui proses jaringan (DNS → TCP → TLS → HTTP).
3. Respons **diterima dan disimpan** secara terpisah (body vs. headers) untuk analisis lebih lanjut.

---

## 5. Relevansi dalam Dunia Keamanan Siber

`cURL` bukan sekadar alat _download_ biasa — ia adalah **"Swiss Army knife"** untuk berinteraksi dengan layanan jaringan, mulai dari halaman web biasa hingga API yang kompleks.

Dalam konteks **Security+** dan keamanan siber secara umum:

| Kegunaan                        | Keterangan                                                                                                                |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| 📋 Analisis HTTP Headers        | Memeriksa header keamanan seperti `Strict-Transport-Security`, `X-Frame-Options`, dll.                                    |
| 🔐 Uji Koneksi Aman             | Menguji apakah endpoint HTTPS dikonfigurasi dengan benar.                                                                 |
| 🔑 Interaksi dengan Autentikasi | Mengirim token, cookie, atau credential untuk mengakses resource yang dilindungi.                                         |
| 🛡️ / ⚔️ Defensive & Offensive   | Digunakan oleh **defender** (analisis log, pengecekan konfigurasi) maupun **attacker** (reconnaissance, eksploitasi API). |

> 💡 `cURL` adalah **alat wajib** yang harus dikuasai oleh setiap profesional keamanan siber.

---

## 6. Kesimpulan

Pada video bagian kedua ini, kita telah mempelajari cara menggunakan `cURL` secara mendalam:

- **`-sS`**: Jalankan dalam mode senyap tapi tetap tampilkan error — ideal untuk skrip.
- **`-L`**: Ikuti redirect secara otomatis.
- **`-I`**: Kirim HEAD request untuk mendapatkan headers saja (hapus jika ingin body juga tersimpan).
- **`-H`**: Tambahkan custom header seperti `Accept: application/json`.
- **`-o`**: Simpan response body ke file.
- **`-D`**: Simpan response headers ke file terpisah.
- Di balik layar, `cURL` menjalankan proses DNS resolution → TCP handshake → TLS handshake → HTTP request → penerimaan respons.
- `cURL` sangat relevan dalam bidang keamanan siber sebagai alat analisis dan pengujian jaringan.

---

_Catatan ini dibuat berdasarkan subtitle video "CURL Part 2"._
