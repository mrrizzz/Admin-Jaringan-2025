# Bab 4: Pengendalian Proses

Sebuah proses dalam sistem operasi terdiri dari dua bagian utama: ruang alamat (yang mencakup kode, data, dan stack) serta struktur data kernel (menyimpan informasi seperti status, prioritas, dan sumber daya). Proses bertanggung jawab atas pengelolaan sumber daya yang dibutuhkan untuk menjalankan program, termasuk memori, file terbuka, dan atribut lainnya. Dengan kata lain, proses dapat dianggap sebagai wadah yang mengorganisir sumber daya yang diperlukan untuk eksekusi suatu program.

## Komponen dalam Proses

1. **Thread**: Unit eksekusi dalam proses yang berbagi ruang memori serta sumber daya yang sama. Thread memungkinkan program menjalankan beberapa tugas secara paralel. Dibandingkan dengan proses, thread lebih ringan sehingga lebih efisien dalam pembuatan dan penghapusannya.
2. **PID (Process ID)**: Identifikasi unik yang diberikan kernel untuk setiap proses. PID digunakan dalam berbagai operasi sistem, seperti mengirim sinyal atau memantau proses.
3. **Namespaces**: Fitur yang memungkinkan proses memiliki PID yang sama dalam lingkungan yang terisolasi. Teknologi container memanfaatkan fitur ini untuk memberikan lingkungan yang terpisah bagi setiap container.
4. **PPID (Parent Process ID)**: Merupakan PID dari proses induk yang menghasilkan proses tertentu. Semua proses (kecuali proses utama sistem) memiliki proses induk, dan PPID membantu sistem mengenali hubungan hierarkis antarproses.
5. **UID (User ID) dan EUID (Effective User ID)**: UID mengidentifikasi pengguna yang menjalankan proses, sementara EUID menentukan izin akses yang dimiliki proses terhadap sumber daya seperti file, port jaringan, atau operasi sistem lainnya.

## Siklus Hidup Proses

Proses baru diciptakan menggunakan system call `fork`, yang menyalin proses induk. Proses anak yang dihasilkan memiliki PID unik namun tetap identik dengan induknya. Pada Linux, `fork` sebenarnya memanggil `clone`, yang menawarkan fitur lebih kompleks dan mendukung thread. Saat sistem melakukan booting, kernel secara otomatis menciptakan beberapa proses, termasuk `init` atau `systemd` (PID 1), yang bertanggung jawab menjalankan skrip startup. Seluruh proses lain berasal dari proses utama ini.

### Signals

Signal adalah notifikasi yang dikirim ke suatu proses untuk memberitahukan terjadinya suatu kejadian. Beberapa signal yang umum digunakan meliputi:

- **KILL**: Memaksa penghentian proses tanpa bisa diabaikan.
- **INT**: Dikirim saat <CTRL-C> ditekan untuk meminta proses berhenti.
- **TERM**: Meminta proses untuk berhenti dengan cara yang bersih.
- **HUP**: Biasanya digunakan untuk meminta daemon melakukan restart.
- **QUIT**: Mirip dengan TERM, tetapi akan menghasilkan core dump jika tidak ditangani.

Perintah `kill` digunakan untuk mengirim signal ke proses:

```bash
kill -9 PID  # Menghentikan proses secara paksa
```

Alternatif lain adalah `killall` dan `pkill`, yang memungkinkan penghentian proses berdasarkan nama atau pengguna:

```bash
killall firefox  # Menghentikan semua proses Firefox
pkill -u user  # Menghentikan semua proses milik user tertentu
```

## PS: Memantau Proses

Perintah `ps` adalah alat utama untuk memantau proses yang berjalan:

```bash
ps aux
```

Alat pemantauan lain yang lebih interaktif adalah `top` dan `htop`, yang memungkinkan pemantauan proses secara real-time.

## Nice dan Renice: Mengubah Prioritas Proses

Niceness adalah nilai numerik yang menunjukkan tingkat prioritas proses dalam persaingan penggunaan CPU. Nilai nice yang lebih tinggi berarti prioritas lebih rendah, sedangkan nilai lebih rendah (atau negatif) menunjukkan prioritas lebih tinggi. Perintah `nice` digunakan untuk menjalankan proses dengan nilai niceness tertentu:

```bash
nice -n 10 sleep 1000  # Menjalankan proses dengan niceness 10
```

Sementara `renice` memungkinkan perubahan nilai niceness dari proses yang sedang berjalan:

```bash
renice -n 5 -p PID  # Mengubah niceness proses menjadi 5
```

## The /proc Filesystem

Direktori `/proc` adalah pseudo-filesystem yang menyediakan informasi tentang status sistem dan proses yang berjalan. Setiap proses direpresentasikan sebagai direktori dengan nama sesuai PID-nya.

## Strace dan Truss

`strace` (Linux) dan `truss` (FreeBSD) digunakan untuk melacak system call serta signal yang terjadi dalam suatu proses:

```bash
strace -p PID  # Melacak system calls dari proses tertentu
```

## Runaway Processes

Proses yang tidak merespons sistem dan menggunakan CPU secara berlebihan disebut runaway processes. Untuk menghentikannya, gunakan `kill` dengan sinyal KILL:

```bash
kill -9 PID
```

Alat seperti `strace` atau `lsof` dapat membantu menganalisis penyebabnya.

## Proses Periodik

### cron: Menjadwalkan Perintah

`cron` adalah alat tradisional untuk menjalankan perintah secara terjadwal. Saat sistem dinyalakan, `cron` akan terus berjalan dan membaca file konfigurasi **crontab**, yang berisi daftar perintah serta waktu eksekusinya.

Format crontab terdiri dari lima kolom untuk menentukan menit, jam, hari, bulan, dan hari dalam seminggu:

```bash
*     *     *     *     *  perintah
-     -     -     -     -  
|     |     |     |     |
|     |     |     |     +----- Hari dalam minggu (0–6, Minggu=0)
|     |     |     +------- Bulan (1–12)
|     |     +--------- Tanggal (1–31)
|     +----------- Jam (0–23)
+------------- Menit (0–59)
```

Contoh:

```bash
30 2 * * * /path/to/command  # Menjalankan perintah setiap hari pukul 02:30
```

### Systemd Timer

Systemd timer adalah alternatif modern untuk `cron`. Dengan systemd, tugas dapat dijadwalkan berdasarkan waktu atau event tertentu:

```bash
systemctl list-timers  # Menampilkan daftar timer yang aktif
```

