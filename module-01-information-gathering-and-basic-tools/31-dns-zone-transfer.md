# 📝 DNS Zone Transfer (AXFR) — Teknik Rekon & Keamanan DNS

## 1. Ringkasan Singkat

Video ini merupakan sesi lab praktik yang mendemonstrasikan bagaimana **DNS Zone Transfer** menggunakan protokol **AXFR** dapat disalahgunakan oleh penyerang. Lab dilakukan di lingkungan terkontrol menggunakan dua virtual machine: satu bertindak sebagai **DNS server yang dikonfigurasi secara keliru** (mesin victim Ubuntu dengan BIND 9), dan satu lagi bertindak sebagai **mesin penyerang** (Kali Purple). Tujuannya adalah menunjukkan bagaimana konfigurasi DNS yang salah dapat mengekspos seluruh zona internal kepada pihak yang tidak berwenang.

---

## 2. Konsep Utama

### a. DNS Zone Transfer & AXFR

- **DNS Zone Transfer** adalah mekanisme sinkronisasi yang digunakan antara _primary (master) DNS server_ dan _secondary DNS server_ agar data zona tetap konsisten.
- **AXFR** (_Authoritative Zone Transfer_) adalah jenis transfer zona penuh (_full zone transfer_) — server mengirimkan **semua record** dalam zona sekaligus.
- Dalam operasi normal, AXFR hanya boleh diizinkan antara server DNS yang terpercaya (misalnya, primary ke secondary).
- Jika konfigurasi `allow-transfer` dibiarkan terbuka (`any`), **siapa pun** dapat meminta transfer zona dan mendapatkan seluruh isi data DNS.

**Contoh perintah AXFR dari sisi penyerang:**

```bash
dig @<IP_VICTIM> example.com AXFR
```

Jika server salah konfigurasi, perintah di atas akan mengembalikan semua record DNS domain tersebut.

---

### b. Setup Lab: Dua Virtual Machine

Lab ini membutuhkan dua mesin yang saling terhubung dalam jaringan yang sama:

| Peran                    | Mesin         | Fungsi                                |
| ------------------------ | ------------- | ------------------------------------- |
| **Victim (Server)**      | Ubuntu 64-bit | DNS server dengan BIND 9              |
| **Attacker (Penyerang)** | Kali Purple   | Menjalankan perintah `dig` untuk AXFR |

- Kedua mesin harus berada di jaringan yang sama: bisa menggunakan mode **NAT** atau **Bridged** di VMware/VirtualBox.
- Pastikan konektivitas antar mesin terverifikasi dengan perintah `ping` sebelum memulai eksploitasi.

**Verifikasi koneksi:**

```bash
# Di mesin penyerang (Kali Purple)
ping <IP_VICTIM>
```

**Konfigurasi jaringan di VMware:**

> VM Settings → Network Adapter → Bridged atau NAT  
> Pastikan kedua mesin menggunakan mode jaringan yang sama.

---

### c. Instalasi dan Konfigurasi BIND 9 (di Mesin Victim)

**BIND 9** (_Berkeley Internet Name Domain_) adalah salah satu software DNS server paling populer di dunia. Digunakan sebagai DNS server otoritatif dalam lab ini.

**Langkah instalasi:**

```bash
# Update repositori
apt update

# Install BIND 9 (mode non-interaktif dengan flag -y)
apt install bind9 -y
```

---

### d. Konfigurasi Zona yang Salah (Misconfiguration)

Setelah BIND 9 terpasang, file konfigurasi lokal perlu diedit untuk mendefinisikan zona:

**Edit file konfigurasi BIND:**

```bash
nano /etc/bind/named.conf.local
```

**Blok zona yang ditambahkan (sengaja misconfigured):**

```
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
    allow-transfer { any; };   # ← INI ADALAH KESALAHAN KONFIGURASI!
};
```

> ⚠️ **`allow-transfer { any; }`** berarti **siapa pun** boleh meminta transfer zona penuh. Dalam produksi, nilai ini seharusnya dibatasi hanya pada IP secondary DNS server yang sah.

---

### e. Membuat Zone File

Zone file berisi pemetaan nama host ke alamat IP dan berbagai record DNS lainnya.

**Langkah membuat zone file:**

```bash
# Salin template zone file default
cp /etc/bind/db.local /etc/bind/db.example.com

# Edit zone file
nano /etc/bind/db.example.com
```

**Contoh isi zone file `db.example.com`:**

```
$TTL    604800
@   IN  SOA ns1.example.com. admin.example.com. (
              2024010101  ; Serial (format: YYYYMMDDNN)
              604800      ; Refresh
              86400       ; Retry
              2419200     ; Expire
              604800 )    ; Negative Cache TTL

@       IN  NS      ns1.example.com.
ns1     IN  A       192.168.x.x
www     IN  A       192.168.x.x
mail    IN  A       192.168.x.x
dev     IN  A       192.168.x.x
```

> 📌 **SOA Serial** sangat penting — nilai ini digunakan oleh secondary server untuk mendeteksi apakah zona sudah diperbarui.

---

### f. Restart dan Pengujian Lokal BIND 9

Setelah konfigurasi disimpan, BIND 9 harus di-restart agar konfigurasi baru dimuat:

```bash
# Restart layanan BIND 9
systemctl restart bind9

# Uji resolusi DNS secara lokal (dari mesin victim sendiri)
dig @127.0.0.1 example.com
dig @127.0.0.1 ns1.example.com
```

> ✅ Jika resolusi berhasil secara lokal, kita bisa melanjutkan ke tahap eksploitasi dari mesin penyerang. Jika gagal, perbaiki konfigurasi terlebih dahulu sebelum berpindah ke Kali Purple.

---

### g. Konfigurasi Firewall (Mengizinkan Port DNS)

Sebelum serangan dapat dilakukan dari mesin eksternal, firewall pada mesin victim harus mengizinkan lalu lintas DNS:

- **Port 53 UDP** → untuk query DNS biasa
- **Port 53 TCP** → untuk zone transfer (AXFR menggunakan TCP)

```bash
# Contoh menambahkan aturan allow di UFW (Ubuntu Firewall)
ufw allow 53/tcp
ufw allow 53/udp
```

> Tanpa aturan ini, permintaan dari mesin penyerang akan diblokir, dan zone transfer tidak akan berhasil.

---

### h. Eksekusi DNS Zone Transfer (AXFR) dari Mesin Penyerang

Dari mesin Kali Purple, uji koneksi dan lakukan zone transfer:

```bash
# Verifikasi koneksi ke victim
ping <IP_VICTIM>

# Uji resolusi DNS dari attacker ke server victim
dig @<IP_VICTIM> example.com

# Lakukan Zone Transfer (AXFR) — serangan utama
dig @<IP_VICTIM> example.com AXFR
```

**Hasil yang diperoleh penyerang:**
Jika konfigurasi `allow-transfer { any; }` aktif, BIND 9 akan mengirimkan **semua record** dalam zona, termasuk:

- Record `A` (host → IP)
- Record `NS` (name server)
- Record `MX` (mail server)
- Nama host internal seperti `dev`, `mail`, `www`

---

### i. Dampak dari Zone Transfer yang Bocor (Reconnaissance Goal)

Data yang diperoleh dari AXFR sangat berharga bagi penyerang karena mengekspos:

| Informasi          | Dampak                                         |
| ------------------ | ---------------------------------------------- |
| Nama host internal | Peta infrastruktur jaringan                    |
| Alamat IP server   | Target langsung untuk serangan lanjutan        |
| Mail server        | Target phishing atau serangan email            |
| Development host   | Kemungkinan sistem dengan keamanan lebih lemah |

> 🔐 Dalam kerangka **CompTIA Security+**, AXFR yang bocor dikategorikan sebagai **initial reconnaissance** — langkah pertama penyerang sebelum melakukan eksploitasi nyata.

---

## 3. Hubungan Antar Konsep

Konsep-konsep di atas membentuk alur serangan yang logis dan sistematis:

```
[Setup Lab]
    │
    ▼
[BIND 9 dipasang di mesin victim]
    │
    ▼
[Zone file dibuat dengan record internal]
    │
    ▼
[Konfigurasi BIND: allow-transfer { any; }] ← CELAH KEAMANAN
    │
    ▼
[Firewall diizinkan untuk port 53 TCP/UDP]
    │
    ▼
[Attacker (Kali Purple) mengirim permintaan AXFR]
    │
    ▼
[Server victim merespons dengan SELURUH isi zona]
    │
    ▼
[Attacker mendapatkan peta infrastruktur DNS lengkap]
    │
    ▼
[Data digunakan sebagai bahan reconnaissance → eksploitasi lanjutan]
```

Kerentanan ini terjadi **bukan karena celah pada protokol DNS itu sendiri**, melainkan karena **kesalahan konfigurasi administrator**. AXFR adalah fitur yang sah dan berguna — tetapi harus dibatasi hanya untuk secondary server yang dipercaya.

---

## 4. Kesimpulan

- **DNS Zone Transfer (AXFR)** adalah mekanisme sah untuk sinkronisasi antara primary dan secondary DNS server.
- Konfigurasi `allow-transfer { any; }` pada BIND 9 adalah **kesalahan kritis** yang memungkinkan siapa pun mendapatkan seluruh data zona DNS.
- Dengan perintah sederhana `dig @<IP_SERVER> <domain> AXFR`, penyerang dapat memetakan seluruh infrastruktur jaringan internal.
- Lab ini menekankan prinsip keamanan penting: **selalu uji di lingkungan terkontrol**, bukan di sistem produksi atau publik.
- Remediasi: ubah `allow-transfer { any; }` menjadi `allow-transfer { <IP_secondary_DNS>; };` untuk membatasi akses hanya pada server yang sah.

---

> ⚠️ **Peringatan Etis:** Teknik ini hanya boleh dipraktikkan di lingkungan lab yang terkontrol dan dengan izin eksplisit. Melakukan zone transfer terhadap DNS server publik atau milik orang lain tanpa izin adalah **ilegal** dan melanggar etika keamanan siber.
