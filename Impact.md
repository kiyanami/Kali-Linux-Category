# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Impact
# 🛠️ Impact Tools - Kali Linux 

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak pengujian dampak (*Impact*), pengujian ketahanan ketersediaan sistem, serta simulasi gangguan ketersediaan (*Denial of Service*) yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [SlowHTTPTest](#1-slowhttptest)

2. [MDK3 (Murder Digital King 3)](#2-mdk3-murder-digital-king-3)

3. [GoldenEye](#3-goldeneye)

4. [DHCPig](#4-dhcpig)

5. [Scapy](#5-scapy)

## 1. SlowHTTPTest

> **Ringkasan:** Perkakas DoS tingkat aplikasi yang sangat efisien. Alat ini mensimulasikan serangan *low-and-slow* dengan cara membuka koneksi HTTP ke server target dan mengirimkan data permintaan secara sangat lambat. Hal ini memaksa server untuk terus menahan alokasi memori *pool*-nya tetap terbuka hingga kehabisan soket, menyebabkan server web resmi menjadi macet (*crash*) tanpa perlu menggunakan bandwidth internet yang besar.

### 📄 Deskripsi Perangkat Lunak

SlowHTTPTest (`slowhttptest`) adalah sebuah instrumen pengujian ketahanan (*stress testing*) dan simulasi serangan *Application Layer Denial of Service* (Layer 7 DoS) berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman C++. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Impact**.

Secara arsitektural, berbeda dengan serangan *volumetric* DoS/DDoS tradisional yang membanjiri kapasitas lebar pita jaringan (*bandwidth exhaustion*) menggunakan ribuan paket per detik, SlowHTTPTest menerapkan paradigma serangan *Low-and-Slow HTTP DoS*. Instrumen ini mengeksploitasi spesifikasi protokol HTTP/1.x dengan membuka ratusan hingga ribuan koneksi TCP yang sah ke peladen web target, namun mengirimkan permintaan HTTP secara sangat lambat atau membaca respons dengan kecepatan sangat rendah. Pendekatan ini mengunci utas pekerja (*worker threads*) dan alokasi memori peladen web hingga kuota koneksi maksimum (*connection pool*) terlampaui, menyebabkan peladen tidak mampu melayani pengguna sah (*service unavailability*) hanya dengan konsumsi bandwidth minimal dari satu mesin penguji.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Evaluasi Ketahanan Ketersediaan Peladen Web (*Web Server Availability & Resilience Testing*):** Membantu administrator sistem dan penguji penetrasi (*penetration tester*) mengaudit batas toleransi konfigurasi peladen web (seperti Apache HTTP Server, Nginx, IIS, atau Tomcat) terhadap penahanan koneksi terbuka dalam durasi lama.

* **Validasi Efektivitas Proksi Balik dan WAF (*Reverse Proxy & WAF Mitigation Benchmarking*):** Digunakan oleh tim pertahanan (Blue Team) untuk menguji apakah mekanisme penyangga (*buffering*) pada Reverse Proxy (seperti Nginx/HAProxy), Load Balancer, maupun aturan perlindungan pada *Web Application Firewall* (WAF) mampu memutus koneksi lambat sebelum mencapai peladen aplikasi *backend*.

* **Kalibrasi Batas Waktu Koneksi (*Connection Timeout Tuning*):** Berfungsi sebagai instrumen pengujian empiris untuk menentukan nilai optimal parameter batas waktu pembacaan header (`client_header_timeout`), badan permintaan (`client_body_timeout`), serta batas koneksi per alamat IP pada infrastruktur web produksi.

### ⚙️ Fungsi Teknis (Functionality)

* **Simulasi Slowloris / Slow HTTP Headers (`-H`):** Membuka banyak koneksi simultan dan mengirimkan *HTTP Request Headers* secara parsial (potongan demi potongan) pada interval waktu tertentu tanpa pernah mengirimkan karakter penutup akhir header (`\r\n\r\n`), memaksa peladen terus menahan soket dalam status menunggu hingga batas koneksi maksimum tercapai.

* **Simulasi R-U-Dead-Yet (RUDY) / Slow HTTP POST Body (`-B`):** Mengirimkan header HTTP POST yang sah dengan deklarasi nilai `Content-Length` sangat besar (misalnya ratusan megabita), namun mengirimkan isi badan pesan (*request body*) dengan kecepatan sangat lambat (1 byte setiap beberapa detik), sehingga peladen terus mengalokasikan sumber daya untuk menunggu kelengkapan data formulir.

* **Simulasi Slow Read Attack (`-R`):** Mengirimkan permintaan HTTP yang lengkap dan sah untuk mengunduh sumber daya berukuran besar dari peladen, tetapi memanipulasi ukuran jendela penerima TCP (*TCP Advertised Receive Window Size*) menjadi sangat kecil, memaksa peladen mengantrekan data di memori dan menjaga soket tetap terbuka dalam waktu lama.

* **Simulasi Range Header Attack (`-X` / Apache Killer):** Mengeksploitasi kerentanan penanganan *HTTP Range Header* dengan meminta puluhan hingga ratusan rentang byte tumpang-tindih dari berkas yang sama dalam satu permintaan, memicu konsumsi CPU dan memori berlebih saat peladen merakit bagian-bagian berkas tersebut.

* **Pemantauan Ketersediaan Real-Time dan Pelaporan Visual (`-g` & `-o`):** Secara terus-menerus mengirimkan permintaan uji coba (*probe connections*) selama serangan berlangsung untuk mengukur tingkat ketersediaan peladen, serta menghasilkan laporan statistik terperinci dalam format HTML dan CSV (`-g -o`) yang memvisualisasikan grafik jumlah koneksi terbuka, koneksi tertutup, dan titik waktu ketika peladen mulai menolak layanan (*Service Unavailable*).

### 💻 Contoh Penggunaan CLI

```
# 1. Menguji ketahanan Slowloris (Header Attack) dengan 1000 koneksi
slowhttptest -H -c 1000 -i 10 -r 200 -t GET -u https://example.com -g -o slowloris_report

# 2. Menguji ketahanan Slow POST (R-U-Dead-Yet / RUDY)
slowhttptest -B -c 1000 -i 10 -r 200 -s 4096 -t POST -u https://example.com/login

# 3. Menguji Slow Read Attack dengan jendela TCP kustom
slowhttptest -R -c 1000 -a 10 -b 3000 -r 200 -t GET -u https://example.com/largefile.zip

```

> 💡 **Kesimpulan:**
> Secara akademis, SlowHTTPTest membuktikan bahwa ketersediaan (*Availability*) dalam triad keamanan informasi (CIA Triad) tidak hanya terancam oleh serangan berbasis volume lalu lintas masif, tetapi juga oleh asimetri konsumsi sumber daya pada tingkat logika protokol aplikasi (Layer 7). Keberadaannya di dalam kategori Impact Kali Linux menegaskan bahwa pertahanan peladen web modern menuntut konfigurasi penguatan (*hardening*) arsitektur koneksi yang presisi—meliputi penempatan Reverse Proxy penyangga di depan peladen aplikasi, penetapan batas waktu (*timeout*) yang ketat untuk penerimaan header dan body HTTP, pembatasan jumlah koneksi simultan per alamat IP (*connection rate limiting*), serta penggunaan arsitektur pemrosesan asinkron berbasis peristiwa (*event-driven*) dibandingkan model berbasis satu utas per koneksi (*thread-per-connection*).

## 2. MDK3 (Murder Digital King 3)

> **Ringkasan:** Perkakas eksploitasi dan serangan dampak khusus untuk jaringan nirkabel (Wi-Fi). MDK3 digunakan untuk mengirimkan paket *deauthentication* massal atau membuat *beacon flood* (ratusan nama Wi-Fi palsu), yang dapat memutuskan koneksi semua perangkat pengguna dari pemancar Wi-Fi asli di sekitar area tersebut secara instan.

### 📄 Deskripsi Perangkat Lunak

MDK3 (Murder Death Kill 3 / kerap disebut juga Murder Digital King 3) adalah sebuah instrumen pengujian ketahanan (*stress testing*) dan pembuktian konsep eksploitasi kelemahan protokol jaringan nirkabel IEEE 802.11 (Wi-Fi) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Impact**.

Secara arsitektural, MDK3 beroperasi pada Lapisan Fisik (Layer 1) dan Lapisan Datalink / MAC Sublayer (Layer 2) dengan memanfaatkan antarmuka jaringan nirkabel dalam mode pemantauan (*monitor mode*) dan injeksi paket mentah (*packet injection* melalui pustaka `osdep` dari proyek Aircrack-ng). Berbeda dengan alat pemecah kriptografi Wi-Fi yang menargetkan kekuatan kunci enkripsi (seperti `aircrack-ng`), MDK3 mengeksploitasi kelemahan desain pada bingkai manajemen (*management frames*) dan bingkai kontrol (*control frames*) standar protokol 802.11 yang pada implementasi tradisional tidak memiliki mekanisme otentikasi maupun integritas kriptografis, sehingga dapat melumpuhkan ketersediaan layanan jaringan nirkabel (*Wireless Denial of Service* / DoS).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Evaluasi Ketersediaan dan Ketahanan Titik Akses (*Wireless Infrastructure Resilience Testing*):** Membantu administrator jaringan dan penguji penetrasi (*penetration tester*) menguji batas toleransi perangkat keras Access Point (AP) dan pengendali nirkabel (*Wireless LAN Controller* / WLC) terhadap banjir bingkai manajemen yang berpotensi menyebabkan *freeze*, *reboot* paksa, atau penurunan kinerja jaringan.

* **Validasi Implementasi Protected Management Frames (IEEE 802.11w):** Digunakan oleh tim pertahanan (Blue Team) untuk memverifikasi secara empiris apakah infrastruktur jaringan Wi-Fi (khususnya WPA2/WPA3) telah mengaktifkan standar 802.11w (PMF) guna memblokir serangan pemutusan koneksi (*deauthentication/disassociation*).

* **Pemicu Penangkapan Handshake dan Serangan Evil Twin (Red Teaming):** Dimanfaatkan sebagai instrumen pendukung untuk memutus koneksi klien secara selektif dari AP sah agar klien melakukan koneksi ulang (memungkinkan penangkapan *4-way handshake*) atau berpindah ke titik akses tiruan (*Evil Twin*).

* **Pengujian Sistem Deteksi Intrusi Nirkabel (*WIDS/WIPS Benchmarking*):** Memungkinkan analis keamanan mengevaluasi kepekaan *Wireless Intrusion Detection/Prevention System* (WIDS/WIPS) dalam mendeteksi anomali seperti kemunculan ribuan SSID palsu (*beacon flooding*) maupun serangan banjir otentikasi.

### ⚙️ Fungsi Teknis (Functionality)

* **Beacon Flood Mode (`b`):** Memancarkan ribuan paket 802.11 *Beacon Frames* palsu dengan nama jaringan (ESSID) acak atau daftar kustom, alamat MAC acak, serta tipe enkripsi bervariasi. Fungsi ini membanjiri daftar pemindaian Wi-Fi pada perangkat klien dan berpotensi menyebabkan *crash* pada driver nirkabel maupun perangkat lunak pemindai spektrum.

* **Authentication DoS Mode (`a`):** Mengirimkan ribuan permintaan otentikasi (*Authentication Frames*) palsu dengan alamat MAC klien acak secara simultan ke satu atau banyak Access Point target untuk menghabiskan tabel status klien (*client association table*) AP, sehingga pengguna sah tidak dapat terhubung atau AP mengalami *hang*.

* **Deauthentication / Disassociation Amok Mode (`d`):** Memindai seluruh saluran (*channel hopping*) dan menyuntikkan paket *Deauthentication* serta *Disassociation* secara berkelanjutan ke seluruh stasiun klien dan AP di sekitar, memutus komunikasi nirkabel aktif secara massal dengan dukungan daftar putih/hitam alamat MAC (`-w` / `-b`).

* **TKIP Michael Countermeasures Exploitation (`m`):** Mengeksploitasi mekanisme perlindungan integritas protokol usang WPA-TKIP dengan menyuntikkan paket data terenkripsi yang rusak secara acak; jika AP mendeteksi dua kegagalan *Message Integrity Code* (MIC) dalam 60 detik, mekanisme keamanan standar protokol secara otomatis mematikan seluruh layanan AP selama 60 detik (*TKIP Shutdown*).

* **Basic Probing and ESSID Bruteforce Mode (`p`):** Mengirimkan paket *Probe Request* secara masif menggunakan kamus kata untuk mengungkap nama jaringan Wi-Fi tersembunyi (*Hidden SSID*) yang tidak menyiarkan nama di dalam bingkai beacon.

* **WIDS Confusion Mode (`w`):** Mengirimkan bingkai otentikasi dan asosiasi yang dimanipulasi ke sistem *Wireless Intrusion Detection System* (WIDS) untuk memicu alarm positif palsu (*false positives*) secara massal dan mengacaukan algoritma pelacakan penyusup.

* **MAC Filter Bruteforce Mode (`f`):** Menguji kerentanan kontrol akses berbasis penyaringan MAC (*MAC Address Filtering*) pada jaringan terbuka dengan mencoba seluruh kombinasi alamat MAC yang valid hingga menemukan alamat yang diizinkan terhubung.

* **WPA Downgrade Test (`g`):** Mengirimkan paket *deauthentication* khusus yang menargetkan sesi WPA2/WPA3 untuk menguji apakah perangkat klien maupun AP rentan menurunkan tingkat keamanannya ke protokol enkripsi yang lebih lemah.

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan Beacon Flood dengan daftar nama SSID buatan
sudo mdk3 wlan0mon b -f essids.txt -a -s 1000

# 2. Menjalankan Deauthentication Amok Mode pada saluran spesifik
sudo mdk3 wlan0mon d -c 6

# 3. Menjalankan Authentication DoS terhadap BSSID Target
sudo mdk3 wlan0mon a -a 00:11:22:33:44:55

```

> 💡 **Kesimpulan:**
> Secara akademis, MDK3 mendemonstrasikan bagaimana ketiadaan otentikasi kriptografis pada lapisan kontrol dan manajemen protokol komunikasi nirkabel warisan (legacy IEEE 802.11) menciptakan kerentanan ketersediaan (*Availability*) yang bersifat struktural. Keberadaannya di dalam kategori Impact Kali Linux menegaskan bahwa keamanan jaringan nirkabel modern tidak cukup hanya mengandalkan kerahasiaan data melalui enkripsi kata sandi; organisasi wajib bermigrasi penuh menuju standar WPA3 (atau minimal WPA2 dengan *Protected Management Frames* / IEEE 802.11w yang diatur ke mode *Required*), menonaktifkan dukungan protokol usang WPA-TKIP, serta menerapkan sensor WIDS/WIPS terintegrasi untuk mendeteksi dan memitigasi injeksi bingkai anomali di ruang udara secara *real-time*.

## 3. GoldenEye

> **Ringkasan:** Skrip pengujian beban berbasis Python yang dirancang untuk melakukan serangan DoS terhadap server HTTP/aplikasi web. GoldenEye membanjiri server dengan permintaan GET atau POST secara simultan guna menghabiskan kapasitas pekerja server web (*web server workers*).

### 📄 Deskripsi Perangkat Lunak

GoldenEye (`goldeneye`) adalah sebuah instrumen pengujian beban (*stress testing*) dan simulasi serangan *Application Layer Denial of Service* (Layer 7 DoS) berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Python. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Impact**.

Secara arsitektural, GoldenEye dirancang untuk mengevaluasi ketahanan dan ketersediaan (*availability*) peladen web melalui eksploitasi mekanisme komunikasi HTTP/HTTPS standar. Berbeda dengan serangan *Low-and-Slow* seperti Slowloris yang menahan koneksi tetap terbuka dengan lalu lintas sangat minim, GoldenEye mengombinasikan arsitektur pemrosesan paralel berbasis *multi-threading* dan *multi-processing* untuk membanjiri peladen target dengan permintaan HTTP GET maupun HTTP POST yang sah secara terus-menerus menggunakan fitur koneksi persisten (*HTTP Keep-Alive*) dan penghindaran tembolok (*Cache-Control bypass*). Pendekatan ini memaksa peladen aplikasi web mengeksekusi siklus pemrosesan penuh (seperti kueri basis data dan rendering skrip backend) pada setiap permintaan hingga kapasitas CPU, memori, dan *connection pool* peladen habis.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pengujian Ketahanan Infrastruktur Web (*Web Server & Application Stress Testing*):** Membantu administrator sistem dan penguji penetrasi (*penetration tester*) mengukur batas ambang kapasitas konkurensi (*concurrency threshold*) peladen web, penyeimbang beban (*load balancer*), dan basis data backend terhadap lonjakan lalu lintas HTTP yang agresif.

* **Validasi Efektivitas Sistem Mitigasi DDoS dan WAF (*Blue Team Mitigation Benchmarking*):** Digunakan oleh tim pertahanan untuk menguji keandalan aturan pembatasan laju permintaan (*rate limiting*), *Web Application Firewall* (WAF), serta jaringan pengiriman konten (CDN) dalam membedakan dan memblokir banjir permintaan otomatis dari lalu lintas pengguna sah.

* **Evaluasi Efisiensi Mekanisme Caching (*Cache-Bypass Resilience Auditing*):** Memungkinkan arsitek sistem menguji dampak beban komputasi pada peladen asal (*origin server*) ketika mekanisme *caching* statis (seperti Varnish atau CDN) berhasil dilewati oleh parameter kueri acak.

### ⚙️ Fungsi Teknis (Functionality)

* **Arsitektur Konkurensi Multi-Worker & Multi-Socket (`-w` & `-s`):** Mengalokasikan banyak proses pekerja independen (*workers*) melalui modul *multiprocessing* Python, di mana setiap pekerja membuka puluhan hingga ratusan soket koneksi TCP paralel menuju target guna memaksimalkan konsumsi sumber daya sistem operasi target.

* **Eksploitasi HTTP Keep-Alive dan Cache-Control Bypass:** Menyisipkan header `Connection: keep-alive` untuk mempertahankan terowongan TCP tetap terbuka demi mengirimkan banyak permintaan beruntun tanpa biaya jabat tangan ulang (*TCP handshake overhead*), sekaligus menambahkan string parameter acak pada URL dan header `Cache-Control: no-cache` guna memaksa peladen memproses setiap permintaan secara dinamis tanpa mengambil dari tembolok.

* **Dukungan Multi-Metode Permintaan HTTP (`-m`):** Menyediakan fleksibilitas pemilihan metode serangan antara `GET`, `POST` (mengirimkan muatan data acak berukuran variabel untuk membebani pemrosesan formulir backend), atau mode acak (`random`) yang mengombinasikan kedua metode tersebut secara dinamis.

* **Penyamaran Identitas Klien Dinamis (*User-Agent & Referer Randomization*):** Secara otomatis memutar (*randomize*) string User-Agent peramban web dan HTTP Referer pada setiap permintaan yang dikirimkan, bertujuan menyamarkan lalu lintas pengujian agar menyerupai permintaan organik dari beragam perangkat pengguna serta menghindari aturan pemblokiran berbasis tanda tangan User-Agent statis.

* **Dukungan SSL/TLS dan Mode Diagnostik (`-n` & `-d`):** Mampu menargetkan layanan terenkripsi HTTPS secara langsung (dengan opsi `-n` untuk mengabaikan validasi sertifikat SSL jika diperlukan) serta menyediakan mode debug (`-d`) untuk memantau status kode respons HTTP yang dikembalikan peladen secara terperinci.

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan Uji Beban HTTP GET dengan 10 workers dan 50 sockets per worker
goldeneye https://example.com -w 10 -s 50

# 2. Mengirimkan kombinasi permintaan GET & POST acak tanpa verifikasi SSL (-n)
goldeneye https://example.com -m random -w 20 -s 100 -n

```

> 💡 **Kesimpulan:**
> Secara akademis, GoldenEye mendemonstrasikan kerentanan struktural pada lapisan aplikasi web (Layer 7), di mana permintaan yang tampak sah secara sintaksis dalam protokol HTTP dapat dieksploitasi untuk menciptakan asimetri beban komputasi dan melumpuhkan ketersediaan layanan (*Availability*). Keberadaannya di dalam kategori Impact Kali Linux menegaskan bahwa pertahanan infrastruktur web modern tidak cukup hanya mengandalkan penambahan lebar pita jaringan; organisasi wajib menerapkan Rate Limiting berbasis IP dan sesi secara ketat, mengonfigurasi arsitektur Caching dan CDN yang tahan terhadap manipulasi parameter kueri, menerapkan tantangan verifikasi klien (JavaScript Challenge / CAPTCHA) saat terjadi anomali lonjakan trafik, serta memanfaatkan sistem Auto-Scaling dan WAF/DDoS Protection berlapis.

## 4. DHCPig

> **Ringkasan:** Perkakas serangan tingkat lanjut untuk protokol DHCP (*Dynamic Host Configuration Protocol*). DHCPig akan menghabiskan seluruh persediaan alamat IP (*IP pool exhaustion*) yang ada di dalam sebuah router/gateway lokal. Dampaknya, komputer atau perangkat baru milik karyawan yang mencoba terhubung ke jaringan tidak akan bisa mendapatkan alamat IP dan gagal tersambung ke internet.

### 📄 Deskripsi Perangkat Lunak

DHCPig (`dhcpig`) adalah sebuah instrumen pengujian ketahanan protokol jaringan (*protocol stress-testing tool*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman Python dengan memanfaatkan pustaka manipulasi paket tingkat rendah Scapy. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Impact**.

Secara arsitektural, DHCPig dirancang secara khusus untuk mengeksekusi serangan *DHCP Starvation* (*DHCP Pool Exhaustion* / Denial of Service) serta pencemaran tabel ARP pada jaringan lokal (*Local Area Network* / LAN). Instrumen ini beroperasi pada Lapisan Datalink (Layer 2) dan Lapisan Aplikasi (Layer 7 / UDP port 67–68) dengan mengirimkan ratusan hingga ribuan permintaan sewa alamat IP (*DHCP Discover* dan *DHCP Request*) yang dipalsukan (*spoofed*) menggunakan alamat MAC fisik yang diacak secara terus-menerus. Pendekatan ini menguras seluruh alokasi ruang alamat IP (*DHCP scope/pool*) yang tersedia pada peladen DHCP sah, sehingga perangkat klien baru tidak dapat memperoleh konfigurasi jaringan maupun terhubung ke jaringan.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Evaluasi Ketersediaan Layanan DHCP (*DHCP Server Availability & Resilience Auditing*):** Membantu administrator jaringan dan penguji penetrasi (*penetration tester*) menguji ketahanan peladen DHCP (baik berbasis Windows Server, router Cisco/MikroTik, maupun `isc-dhcp-server` Linux) terhadap serangan pengurasan kapasitas alokasi IP.

* **Validasi Keamanan Switch Layer 2 (*Port Security & DHCP Snooping Benchmarking*):** Digunakan oleh tim pertahanan (Blue Team) untuk memverifikasi secara empiris apakah mekanisme keamanan pada *managed switch*—seperti *Port Security* (pembatasan jumlah alamat MAC per port fisik) dan *DHCP Snooping Rate-Limiting*—telah dikonfigurasi dan berfungsi dengan benar.

* **Prasyarat Simulasi Serangan Rogue DHCP Server (Red Teaming):** Berfungsi sebagai langkah persiapan taktis dalam simulasi serangan *Man-in-the-Middle* (MitM) berbasis DHCP; dengan melumpuhkan peladen DHCP sah terlebih dahulu menggunakan DHCPig, penyerang dapat menyebarkan *Rogue DHCP Server* untuk membagikan konfigurasi *Default Gateway* dan DNS Server palsu kepada klien baru.

### ⚙️ Fungsi Teknis (Functionality)

* **Pengurasan Ruang Alamat IP Otomatis (*Automated DORA Handshake Exhaustion*):** Mengirimkan paket siaran *DHCP Discover* dengan alamat MAC sumber (*Source MAC*) dan parameter *Client Hardware Address* (`chaddr`) yang diacak pada setiap siklus, menangkap paket *DHCP Offer* dari peladen, lalu langsung membalas dengan *DHCP Request* hingga seluruh rentang IP di dalam *pool* peladen berstatus terpakai (*leased*).

* **Pelepasan Sewa IP Klien Sah (*Active Lease Release / DHCP Release Spoofing*):** Memindai jaringan lokal untuk mengidentifikasi perangkat klien yang sedang aktif, memalsukan identitas alamat IP dan alamat MAC mereka, lalu mengirimkan paket *DHCP Release* ke peladen DHCP guna memaksa peladen mencabut alokasi IP milik pengguna sah dan mengembalikannya ke *pool* agar dapat langsung dikuras kembali oleh DHCPig.

* **Pembanjiran Tabel ARP dan Pencemaran Tetangga (*ARP Exhaustion & Neighbor Flooding*):** Menyertakan fitur pengiriman paket permintaan ARP (*ARP Requests*) secara masif ke seluruh subnet lokal untuk membanjiri tabel ARP (*ARP cache*) pada gateway dan host sekitar, memperparah gangguan konektivitas pada segmen LAN.

* **Dukungan Multi-Protokol IPv4 dan IPv6 (`-6` / DHCPv6 Starvation):** Mampu menargetkan infrastruktur jaringan berbasis IPv4 standar maupun mengaktifkan mode IPv6 (`-6`) untuk menguras alokasi alamat dan prefiks pada peladen DHCPv6 modern.

* **Pengendalian Laju Paket, Identitas Vendor, dan Fuzzing (`-c`, `-i`, `-f`):** Menyediakan opsi pengaturan jumlah maksimal alamat IP yang dikuras (`-c`), jeda waktu antar-paket (`-i`), penyamaran identitas hostname klien agar menyerupai perangkat normal, serta mode *fuzzing* (`-f`) untuk mengirimkan muatan opsi DHCP anomali guna menguji stabilitas parser perangkat lunak peladen DHCP.

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan DHCP Starvation Attack pada interface eth0
sudo dhcpig eth0

# 2. Menguras alokasi IP dengan jeda antar paket 100ms
sudo dhcpig eth0 -i 100

# 3. Pengulasan alokasi DHCPv6 pada interface eth0
sudo dhcpig eth0 -6

```

> 💡 **Kesimpulan:**
> Secara akademis, DHCPig mendemonstrasikan bagaimana ketiadaan mekanisme otentikasi identitas pengirim pada protokol konfigurasi jaringan otomatis (*bootstrap protocols* seperti DHCP) menciptakan kerentanan ketersediaan (*Availability*) yang kritikal di dalam jaringan lokal. Keberadaannya di dalam kategori Impact Kali Linux menegaskan bahwa pertahanan jaringan LAN modern tidak boleh membiarkan port switch terbuka tanpa pembatasan; organisasi wajib mengaktifkan *Switch Port Security* (membatasi maksimum alamat MAC yang dipelajari per antarmuka fisik), mengonfigurasi *DHCP Snooping* dengan pembatasan laju paket (*rate limiting* dan verifikasi kesesuaian *Source MAC* dengan `chaddr`), serta menerapkan segmentasi VLAN dan autentikasi port berbasis IEEE 802.1X untuk mencegah injeksi paket palsu di ruang lingkup Layer 2.

## 5. Scapy

> **Ringkasan:** Meskipun merupakan framework manipulasi paket jaringan yang universal, di dalam konteks kategori Impact, Scapy sering kali digunakan untuk merancang skrip kustom guna melakukan *packet flooding*, memalsukan paket data (*packet spoofing*), atau mengirimkan struktur paket rusak (*malformed packets*) demi menguji apakah sistem target akan langsung mengalami *crash*.

### 📄 Deskripsi Perangkat Lunak

Scapy adalah sebuah pustaka (*library*) pemrograman dan program manipulasi paket jaringan interaktif berbasis *open-source* yang dikembangkan dalam bahasa pemrograman Python. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam dua kategori taktis utama, yaitu **Discovery** (secara spesifik pada sub-kategori *Network Sniffing*) dan **Impact**.

Secara arsitektural, Scapy melampaui keterbatasan alat jaringan tradisional (seperti Nmap, hping3, atau tcpdump) yang hanya mampu menjalankan skenario pengujian tetap yang telah diprogram sebelumnya oleh pengembangnya. Scapy menerapkan paradigma perakitan paket berlapis (*layered packet crafting*), yang memungkinkan peneliti keamanan merancang, menyusun, menyuntikkan (*inject*), menyadap (*sniff*), serta membedah (*dissect*) paket jaringan mentah dari awal mulai dari Lapisan Fisik/Datalink (Layer 2) hingga Lapisan Aplikasi (Layer 7) secara terprogram sesuai spesifikasi protokol maupun dalam bentuk paket anomali (*malformed packets*).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Simulasi Serangan Ketersediaan dan Eksploitasi Protokol (*Protocol Stress Testing & DoS Simulation*):** Membantu penguji penetrasi (*penetration tester*) membangun skrip pembuktian konsep (*Proof of Concept* / PoC) untuk serangan berbasis manipulasi paket mentah, seperti TCP SYN Flood, ICMP Smurf/Ping of Death, pembanjiran tabel MAC (*MAC Flooding*), atau DHCP Starvation.

* **Penyadapan Kustom dan Analisis Lalu Lintas Mendalam (*Custom Network Sniffing & Dissection*):** Digunakan untuk menangkap lalu lintas jaringan spesifik secara *real-time* maupun membedah berkas tangkapan paket (`.pcap`) secara terprogram guna mengekstrak artefak atau indikator kompromi tertentu.

* **Fuzzing Implementasi Tumpukan Protokol (*Protocol Fuzzing & Vulnerability Research*):** Memungkinkan peneliti keamanan menguji ketahanan parser protokol pada router, sistem operasi, atau perangkat *Internet of Things* (IoT) dengan mengirimkan kombinasi bidang header yang rusak atau tidak sesuai standar RFC.

* **Audit Aturan Firewall dan Evasi IDS/IPS (*Firewall & Intrusion Detection Benchmarking*):** Berfungsi sebagai instrumen presisi bagi tim pertahanan (Blue Team) dan praktisi Red Team untuk menguji bagaimana firewall dan sensor IDS/IPS merespons fragmentasi IP tumpang-tindih (*overlapping IP fragments*), manipulasi TCP Flags tidak lazim, atau pemalsuan alamat sumber (*IP/MAC spoofing*).

### ⚙️ Fungsi Teknis (Functionality)

* **Perakitan Paket Modular Berbasis Operator (`/` Layer Stacking):** Menggunakan sintaks intuitif Python dengan operator pembagian (`/`) untuk menumpuk lapisan protokol secara hierarkis (misalnya `Ether()/IP(dst="192.168.1.1")/TCP(dport=80, flags="S")/Raw(load="GET / HTTP/1.1")`), serta secara otomatis menghitung nilai bidang teknis yang kompleks seperti *checksum* dan panjang paket.

* **Injeksi dan Transmisi Paket Multi-Lapisan (`send`, `sendp`, `sr`, `sr1`, `srp`):** Menyediakan fungsi pengiriman paket pada Lapisan 3 (`send()` / `sr()` yang ditangani oleh tabel perutean kernel OS) maupun Lapisan 2 (`sendp()` / `srp()` yang dikirimkan langsung melalui antarmuka ethernet/Wi-Fi mentah), lengkap dengan fungsi korelasi otomatis antara paket permintaan (*stimulus*) dan paket balasan (*response*).

* **Mesin Penyadapan dan Pemfilteran Terprogram (`sniff`):** Mengintegrasikan fungsi penangkapan paket berbasis pustaka `libpcap` dengan dukungan filter *Berkeley Packet Filter* (BPF) serta fungsi *callback* (`prn=`) yang mengeksekusi logika analisis Python secara langsung pada setiap paket yang tertangkap.

* **Dukungan Ratusan Protokol dan Fuzzing Otomatis (`fuzz()`):** Mendukung spesifikasi ratusan protokol jaringan—mulai dari Ethernet, ARP, VLAN (802.1Q), IPv4/IPv6, ICMP, TCP, UDP, DNS, HTTP, hingga protokol nirkabel IEEE 802.11, Bluetooth, dan protokol industri—serta menyediakan pembungkus `fuzz()` untuk mengacak nilai bidang header yang tidak didefinisikan secara eksplisit.

* **Visualisasi Grafik dan Integrasi Forensik PCAP (`rdpcap`, `wrpcap`, `pdfdump`, `traceroute`):** Mampu membaca (`rdpcap`) dan menulis (`wrpcap`) berkas tangkapan standar Wireshark, memvisualisasikan struktur bit paket ke dalam diagram grafis PDF/PS (`pdfdump`), serta membuat grafik topologi rute jaringan interaktif (*3D/2D Traceroute Graph*).

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan konsol interaktif Scapy
sudo scapy

# 2. Contoh perakitan dan pengiriman paket SYN Flood sederhana di terminal Scapy:
# >>> send(IP(dst="192.168.1.50", src=Fuzz(IP()))/TCP(dport=80, flags="S"), loop=1)

# 3. Menjalankan skrip Python satu baris menggunakan Scapy untuk mengirimkan paket ICMP custom
python3 -c 'from scapy.all import *; send(IP(dst="192.168.1.1")/ICMP()/"TestPayload")'

```

> 💡 **Kesimpulan:**
> Secara akademis, Scapy merepresentasikan jembatan antara teori arsitektur protokol jaringan (model OSI dan TCP/IP) dengan implementasi rekayasa keamanan siber tingkat lanjut. Kemampuannya membebaskan peneliti dari batasan tumpukan protokol standar sistem operasi menjadikannya instrumen fundamental dalam riset kerentanan jaringan, pengujian ketahanan sistem terhadap gangguan ketersediaan (*Impact*), serta pengembangan perangkat lunak keamanan kustom. Keberadaannya di Kali Linux menegaskan bahwa pertahanan jaringan modern tidak cukup hanya mengandalkan pemblokiran pola lalu lintas standar; perangkat jaringan dan sensor keamanan wajib mampu memvalidasi integritas struktural paket secara ketat terhadap anomali protokol dan manipulasi lapisan rendah.
