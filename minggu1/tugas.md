# Laporan Percobaan

## Mata Kuliah: Workshop Administrasi Jaringan

### Nama Dosen Pengampu
**Bapak Dr. Ferry Astika Saputra ST, M.Sc**

### Dikerjakan oleh:
- **Nama:** Muhammad Rafi Rizaldi  
- **NRP:** 3123600001  
- **Kelas:** 2 D4 IT A  

---
# Sistem DNS (Domain Name System)

## Pengertian Dasar
Sistem DNS adalah teknologi yang berfungsi mengkonversi nama domain menjadi alamat IP, memungkinkan komputer berkomunikasi satu sama lain di internet melalui identifikasi numerik yang tepat.

## Karakteristik Utama DNS

### Server DNS Terdistribusi
DNS beroperasi melalui jaringan server global yang dikelola oleh berbagai operator, meningkatkan kecepatan akses dan ketahanan sistem.

### Konsistensi Longgar
Meski terdistribusi di berbagai lokasi, server DNS tetap membentuk sistem terintegrasi dengan konsistensi data yang tercapai secara bertahap (*eventual consistency*).

### Skalabilitas
Arsitektur DNS dirancang untuk mengakomodasi pertumbuhan dengan penambahan kapasitas server, baik secara vertikal maupun horizontal.

### Keandalan Tinggi
Sebagai komponen infrastruktur internet yang kritis, DNS memiliki fitur failover, redundansi, dan perlindungan DDoS untuk menjamin ketersediaan tanpa gangguan.

### Kemampuan Dinamis
DNS mendukung pembaruan catatan secara langsung (misalnya melalui DDNS), memungkinkan perubahan konfigurasi tanpa menyebabkan gangguan layanan.

## Protokol Komunikasi DNS

### UDP Port 53
Mayoritas komunikasi DNS menggunakan protokol UDP karena efisiensi dan kecepatan dalam pertukaran data berukuran kecil.

### TCP Port 53
Digunakan dalam situasi khusus:
- Transfer zona (sinkronisasi data antar server DNS)
- Respons DNS yang melebihi batas ukuran paket UDP (512 byte)

## Proses Resolusi DNS

1. **Permintaan Pengguna**
   - Pengguna memasukkan nama domain (contoh: `www.example.com.au`) di browser
   - Browser memeriksa cache lokal untuk alamat IP yang tersimpan sebelumnya

2. **Konsultasi dengan DNS Resolver**
   - Jika tidak ditemukan di cache, browser menghubungi DNS Resolver (biasanya disediakan ISP atau layanan seperti Google DNS)
   - Resolver bertugas menemukan alamat IP dari domain yang diminta

3. **Pencarian Bertingkat**
   - Resolver berkomunikasi dengan Root Server untuk mengetahui pengelola ekstensi domain
   - Root Server mengarahkan ke TLD Server (Top-Level Domain Server) yang mengelola ekstensi tersebut

4. **Jenis-jenis TLD:**
   - **ccTLDs**: Domain tingkat atas kode negara (`.id`, `.us`, `.jp`)
   - **gTLDs**: Domain tingkat atas generik (`.com`, `.org`, `.gov`)
   - **Infrastructure TLD**: Untuk keperluan teknis internet (`.arpa`)
   - **IDN TLDs**: Domain tingkat atas dengan karakter non-ASCII (`.中国`, `.рф`)

5. **Komunikasi dengan Authoritative Server**
   - TLD Server memberikan informasi Authoritative Server yang menyimpan data lengkap domain
   - Authoritative Server menjawab dengan alamat IP yang sesuai

6. **Penyelesaian Permintaan**
   - Resolver menerima dan menyimpan alamat IP dalam cache untuk akses berikutnya
   - Browser mengakses alamat IP untuk menampilkan situs web yang diminta

## Komponen Sistem DNS

### 1. Namespace
Struktur hierarkis global untuk penamaan domain, terdiri dari:
- **Root** (.) di level tertinggi
- **Top-Level Domains** (`.com`, `.id`)
- **Second-Level Domains** (`example.com`)
- **Subdomain** (`blog.example.com`)

**Domain vs Zona:**
- **Domain**: Identitas unik dalam hierarki DNS
- **Zona**: Bagian namespace yang dikelola oleh otoritas tertentu

### 2. Nameserver
Server yang menyimpan dan mengelola data DNS untuk zona tertentu.

**Fungsi:**
- Menyimpan catatan DNS (A, MX, CNAME, dll.)
- Merespons permintaan DNS resolver

**Jenis:**
- **Authoritative Nameserver**: Menyediakan data resmi zona
- **Recursive Nameserver**: Mencari data dari nameserver lain jika diperlukan

### 3. Resolver dan Klien
Perangkat atau layanan yang mengirim permintaan DNS untuk mendapatkan alamat IP.

**Contoh:**
- **Resolver**: Google DNS (8.8.8.8), Cloudflare (1.1.1.1)
- **Klien**: Browser, aplikasi, sistem operasi

**Alur Kerja:**
1. Klien meminta resolver untuk menerjemahkan domain
2. Resolver berkomunikasi dengan nameserver (root → TLD → authoritative)
3. Hasil dikirimkan kembali ke klien

## Konfigurasi BIND9

### 1. Instalasi BIND9
```
apt-get install bind9
```

### 2. Konfigurasi Domain (kelompok07.com)

#### File `/etc/bind/named.conf`
Menambahkan referensi ke file konfigurasi jaringan internal.

#### File `/etc/bind/named.conf.options`
```
acl internal-network {
    192.168.200.0/24;
};

options {
    directory "/var/cache/bind";
    allow-query { localhost; internal-network; };
    allow-transfer { localhost; };
    recursion yes;
};
```

**Fungsi:**
- Mendefinisikan jaringan 192.168.200.0/24 sebagai "jaringan internal"
- Mengatur lokasi cache DNS
- Mengizinkan kueri dari localhost dan jaringan internal
- Membatasi transfer zona hanya dari localhost
- Mengaktifkan fungsi rekursi

#### File `/etc/bind/named.conf.internal-zones`
```
zone "com" {
    type master;
    file "/etc/bind/com.lan";
    allow-update { none; };
};

zone "200.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/200.168.192.db";
    allow-update { none; };
};
```

**Fungsi:**
- Mendefinisikan zona forward untuk domain "com"
- Mendefinisikan zona reverse untuk IP 192.168.200.0/24
- Menetapkan server sebagai master untuk kedua zona
- Menonaktifkan pembaruan dinamis

#### File `/etc/bind/com.lan`
```
$TTL 86400
@ IN SOA ns.com. admin.com. (
    2023080801  ; Serial
    3600        ; Refresh
    1800        ; Retry
    604800      ; Expire
    86400 )     ; Minimum TTL

@ IN NS ns.com.
@ IN A 192.168.200.2
ns IN A 192.168.200.2
kelompok07 IN A 192.168.200.2
```

**Fungsi:**
- Mengatur TTL (Time To Live) domain - 1 hari
- Mendefinisikan informasi SOA (Start of Authority)
- Menetapkan ns.com sebagai nameserver
- Menetapkan alamat IP untuk domain dasar, nameserver, dan subdomain

#### File `/etc/bind/200.168.192.db`
```
$TTL 86400
@ IN SOA ns.com. admin.com. (
    2023080801  ; Serial
    3600        ; Refresh
    1800        ; Retry
    604800      ; Expire
    86400 )     ; Minimum TTL

@ IN NS ns.com.
2 IN PTR kelompok07.com.
```

**Fungsi:**
- Mengkonfigurasi reverse lookup
- Mengarahkan IP 192.168.200.2 ke nama domain kelompok07.com

### 3. Konfigurasi Resolver
Mengatur komputer klien untuk menggunakan server DNS di alamat 192.168.200.2

### 4. Konfigurasi IPTables
```
iptables -A INPUT -p tcp --dport 53 -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE
```

**Fungsi:**
- Mengizinkan koneksi masuk ke port 53 (DNS) melalui TCP dan UDP
- Mengaktifkan IP masquerading untuk lalu lintas keluar melalui interface enp0s8
