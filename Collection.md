# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Collection
# 🛠️ Collection Tools - Kali Linux Documentation

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak pengumpulan data (*Collection*) dan pengintersepsian informasi yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [Ettercap & ettercap-text-only](#1-ettercap--ettercap-text-only)

   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-ettercap-vs-ettercap-text-only)

2. [Mitmproxy](#2-mitmproxy)

3. [HTTrack](#3-httrack)

4. [Evilginx2](#4-evilginx2)

5. [Fluxion & Wifipumpkin3](#5-fluxion--wifipumpkin3)

   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-fluxion-vs-wifipumpkin3)

## 1. Ettercap & ettercap-text-only

> **Ringkasan:** Perkakas komprehensif untuk serangan terencana di jaringan lokal (*Man-in-the-Middle*). Ettercap digunakan untuk menyadap lalu lintas data antarperangkat, mengisolasi koneksi, serta mengumpulkan data teks sensitif seperti kata sandi tidak terenkripsi yang melintasi jaringan.

### 📄 Deskripsi Perangkat Lunak

* **Ettercap (`ettercap` / `ettercap-graphical`):** Sebuah *suite* keamanan jaringan *open-source* untuk pengujian *Man-in-the-Middle* (MitM) pada jaringan berbasis *switched LAN* yang dilengkapi dengan antarmuka pengguna grafis (*Graphical User Interface* / GUI berbasis GTK+). Secara arsitektural, Ettercap bekerja dengan mengintervensi alur perutean paket di Lapisan Datalink (Layer 2) dan Lapisan Jaringan (Layer 3), sehingga seluruh komunikasi antara dua titik target (misalnya antara klien dan *default gateway*) mengalir melewati mesin penguji sebelum diteruskan ke tujuan asli.

* **ettercap-text-only:** Varian paket dan mode eksekusi murni berbasis antarmuka baris perintah (*Command-Line Interface* / CLI) serta antarmuka teks interaktif (*Ncurses UI*) dari mesin inti Ettercap. Secara struktural, `ettercap-text-only` menghilangkan seluruh dependensi pustaka grafis (X11/GTK+), menjadikannya sangat ringan, stabil, dan dapat dieksekusi secara jarak jauh melalui sesi SSH maupun diintegrasikan ke dalam skrip otomasi audit jaringan tanpa memerlukan lingkungan desktop.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pengumpulan Kredensial dan Data Sensitif (*Cleartext Credential Collection*):** Membantu penguji penetrasi (*penetration tester*) mencegat dan memanen nama pengguna, kata sandi, serta token sesi yang dikirimkan melalui protokol jaringan tanpa enkripsi (seperti HTTP, FTP, Telnet, POP3, IMAP, SMTP, SNMP, dan autentikasi basis data) di dalam jaringan internal.

* **Simulasi Pembelokan Lalu Lintas dan Manipulasi Paket (*Traffic Redirection & Packet Tampering*):** Digunakan oleh praktisi Red Team untuk mendemonstrasikan bagaimana aktor ancaman dapat memalsukan resolusi nama domain (DNS *Spoofing*), menyuntikkan skrip ke dalam halaman web HTTP yang sedang diakses korban, atau melakukan serangan penurunan tingkat enkripsi (SSL *Stripping*).

* **Evaluasi Keamanan Switch dan Sistem Deteksi Intrusi (*Blue Team & Switch Hardening Auditing*):** Memungkinkan administrator jaringan menguji efektivitas mekanisme pertahanan lapisan bawah pada switch terkelola—seperti *Dynamic ARP Inspection* (DAI), *DHCP Snooping*, dan *Port Security*—serta menguji kepekaan sistem deteksi intrusi (IDS) dalam mengenali anomali pemalsuan identitas jaringan.

### ⚙️ Fungsi Teknis (Functionality)

* **Eksekusi Multi-Vektor Serangan Man-in-the-Middle (`-M`):** Mengintegrasikan berbagai teknik pembelokan lalu lintas jaringan secara otomatis, meliputi:

  * **ARP Poisoning:** Mengirimkan paket *ARP Reply* palsu secara berkala untuk mengaitkan alamat MAC mesin penguji dengan alamat IP korban dan gateway.

  * **ICMP Redirect & DHCP Spoofing:** Memanipulasi tabel perutean klien atau bertindak sebagai peladen DHCP palsu untuk menetapkan mesin penguji sebagai gateway utama dan peladen DNS.

  * **Port Stealing:** Teknik mitigasi khusus pada lingkungan switch yang memblokir ARP *poisoning* dengan membanjiri tabel CAM switch menggunakan alamat MAC target.

* **Pembedahan Protokol dan Ekstraksi Kredensial Otomatis (*Active & Passive Protocol Dissection*):** Menguraikan ratusan protokol jaringan secara *real-time* untuk mengekstrak kredensial otentikasi secara otomatis di layar maupun berkas log, serta memetakan topologi host aktif, port terbuka, dan sistem operasi di dalam segmen LAN.

* **Mesin Penyaringan dan Injeksi Paket Dinamis (`etterfilter`):** Menyediakan bahasa pemrograman filter khusus yang dikompilasi menggunakan utilitas `etterfilter`. Fungsi ini memungkinkan analis memeriksa, memblokir (*drop*), mengganti (*replace*), atau menyisipkan muatan byte baru (*inject*) ke dalam badan paket jaringan saat melintas (misalnya menyuntikkan tag `<script>` ke dalam lalu lintas HTTP atau mengganti berkas biner yang diunduh korban).

* **Ekosistem Plugin Ekstensif (`-P`):** Dilengkapi dengan puluhan plugin modular siap pakai untuk memperluas kapabilitas serangan dan analisis, seperti `dns_spoof` (pemalsuan jawaban DNS), `arp_cop` (mendeteksi aktivitas ARP mencurigakan), `repoison_arp`, `chk_poison`, serta `find_conn`.

* **Fleksibilitas Mode Operasi Unified vs. Bridged (`-T`, `-C`, `-G`, `-B`):** Mendukung empat antarmuka operasi—yaitu mode teks murni (`-T`), antarmuka interaktif berbasis Curses (`-C`), antarmuka grafis penuh (`-G`), serta mode *Bridged Sniffing* (`-B`) yang menggunakan dua kartu jaringan fisik untuk mencegat lalu lintas secara transparan di lapisan fisik tanpa mengirimkan paket pemalsuan protokol apa pun.

### 📊 Perbandingan Singkat & Kesimpulan Akademis: Ettercap vs ettercap-text-only

| Aspek Komparasi | Ettercap (`ettercap` / GUI) | ettercap-text-only (CLI / Ncurses) | 
| ----- | ----- | ----- | 
| **Antarmuka & Dependensi** | Antarmuka grafis GTK+ (memerlukan lingkungan desktop X11/Wayland). | Antarmuka baris perintah (`-T`) dan terminal interaktif Curses (`-C`) tanpa dependensi GUI. | 
| **Kegunaan Optimal** | Pemetaan visual daftar host, pemantauan koneksi interaktif, dan pembelajaran laboratorium. | Eksekusi via SSH jarak jauh, konsumsi sumber daya minimal, dan otomasi skrip audit. | 

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan Ettercap mode teks murni (sniffing pasif pada eth0)
sudo ettercap -T -q -i eth0

# 2. Eksekusi ARP Poisoning (MitM) antara target IP 192.168.1.50 dan Gateway 192.168.1.1
sudo ettercap -T -q -i eth0 -M arp:remote /192.168.1.50// /192.168.1.1//

# 3. Menggunakan plugin DNS Spoofing bersama ARP Poisoning
sudo ettercap -T -q -i eth0 -P dns_spoof -M arp:remote /192.168.1.50// /192.168.1.1//

```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Ettercap dan `ettercap-text-only` membuktikan kerentanan fundamental pada desain protokol jaringan lokal warisan (seperti ARP dan DHCP) yang beroperasi tanpa mekanisme otentikasi kriptografis. Keberadaannya di dalam kategori Collection Kali Linux menegaskan bahwa pertahanan jaringan lokal modern menuntut penerapan pengamanan berlapis pada infrastruktur Layer 2 dan Layer 7, meliputi konfigurasi *DHCP Snooping* dan *Dynamic ARP Inspection* (DAI) pada switch, penggunaan enkripsi *end-to-end* yang dipaksakan (seperti HTTPS dengan HSTS, SSH, dan DNSSEC/DoH), serta pemantauan anomali lalu lintas ARP secara berkelanjutan.

## 2. Mitmproxy

> **Ringkasan:** Alat interseptor HTTP/HTTPS berbasis konsol yang bertindak sebagai proksi. Dalam fase pengumpulan data, alat ini memungkinkan penguji penetrasi untuk mencegat, melihat, dan memanipulasi lalu lintas aplikasi web atau ponsel untuk mengambil data API rahasia atau token sesi.

### 📄 Deskripsi Perangkat Lunak

`mitmproxy` adalah sebuah rangkaian (*suite*) proksi pencegat interaktif (*interactive intercepting proxy*) berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Python untuk memantau, membedah, dan memodifikasi lalu lintas protokol HTTP/1, HTTP/2, HTTP/3 (QUIC), HTTPS, serta WebSockets. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Collection**.

Secara arsitektural, `mitmproxy` beroperasi sebagai perantara *Man-in-the-Middle* (MitM) yang membongkar enkripsi saluran SSL/TLS secara dinamis dengan menerbitkan sertifikat kriptografis X.509 *on-the-fly* yang ditandatangani oleh otoritas sertifikat (*Certificate Authority* / CA) lokal `mitmproxy`. Berbeda dengan proksi berbasis desktop konvensional seperti Burp Suite, ekosistem `mitmproxy` menyediakan tiga antarmuka terpisah untuk satu inti mesin yang sama—yaitu `mitmproxy` (antarmuka konsol terminal interaktif), `mitmweb` (antarmuka grafis berbasis peramban web), dan `mitmdump` (utilitas aliran data murni baris perintah yang dapat diprogram).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pengumpulan Kredensial, Token Sesi, dan Lalu Lintas API (*Traffic & Credential Collection*):** Membantu penguji penetrasi (*penetration tester*) mencegat, merekam, dan memanen kredensial otentikasi, kunci API (*API keys*), JSON Web Tokens (JWT), maupun data sensitif yang dikirimkan oleh aplikasi klien maupun peramban web.

* **Audit Keamanan Aplikasi Seluler dan IoT (*Mobile & Thick-Client Security Auditing*):** Digunakan sebagai instrumen utama dalam pengujian keamanan aplikasi Android dan iOS untuk membedah komunikasi backend REST/GraphQL API, menguji ketahanan implementasi SSL/TLS *Certificate Pinning*, serta mendeteksi kebocoran data telemetri.

* **Otomasi Manipulasi Lalu Lintas dan Pengujian Ketahanan (*Programmable Traffic Tampering & Fuzzing*):** Memungkinkan analis keamanan dan pengembang menulis skrip Python khusus guna memodifikasi permintaan atau respons HTTP secara otomatis di tengah transmisi, menyuntikkan muatan eksploitasi, atau menyimulasikan kegagalan peladen.

* **Analisis Lalu Lintas Malware dan Forensik Jaringan (*Blue Team & Malware Sandbox Analysis*):** Berfungsi sebagai proksi transparan (*transparent proxy*) di dalam lingkungan isolasi (*sandbox*) untuk mendekripsi dan mengamati pola komunikasi Command and Control (C2) berbasis HTTPS yang dilakukan oleh sampel perangkat lunak berbahaya.

### ⚙️ Fungsi Teknis (Functionality)

* **Tiga Antarmuka Operasi Terpadu (`mitmproxy`, `mitmweb`, `mitmdump`):**

  * **mitmproxy:** Menyediakan antarmuka konsol berbasis teks (TUI - *Text User Interface*) dengan navigasi pintasan papan ketik bergaya Vim untuk menginspeksi headers, cookies, dan badan pesan secara cepat di terminal tanpa memerlukan lingkungan grafis.

  * **mitmweb:** Menyediakan antarmuka grafis berbasis web yang menyerupai Chrome DevTools untuk memvisualisasikan aliran permintaan HTTP secara intuitif.

  * **mitmdump:** Bertindak serupa dengan `tcpdump` untuk lalu lintas HTTP/HTTPS, memungkinkan perekaman aliran trafik ke dalam berkas biner serta pemutaran ulang (*replay*) secara otomatis melalui baris perintah.

* **Ekstensibilitas Terprogram via Python Addons API (`-s`):** Merupakan keunggulan arsitektural utama `mitmproxy`, yang mengizinkan analis memuat skrip addon Python untuk mencegat setiap siklus peristiwa (*event hooks* seperti `request()`, `response()`, dan `websocket_message()`) guna memodifikasi isi paket, menghitung ulang tanda tangan kriptografis (HMAC/Hash) secara otomatis, atau mengektrak data spesifik secara *real-time*.

* **Dukungan Multi-Mode Penangkapan Lalu Lintas (`--mode`):** Mendukung berbagai topologi jaringan fleksibel, meliputi mode proksi reguler (*Regular Proxy*), *Transparent Proxy* (bekerja bersama pengalihan aturan `iptables`/`nftables` di tingkat kernel tanpa memerlukan konfigurasi proksi pada aplikasi klien), *Reverse Proxy*, *Upstream Proxy*, serta mode proksi WireGuard bawaan untuk mencegat lalu lintas perangkat seluler secara langsung melalui terowongan VPN.

* **Perekaman, Pemutaran Ulang, dan Modifikasi Sesi (*Flow Recording & Client/Server Replay*):** Mampu menyimpan seluruh riwayat percakapan HTTP/HTTPS ke dalam berkas aliran terstruktur (`-w`), memutar ulang permintaan klien (*client replay*) untuk menguji konsistensi otorisasi, maupun bertindak sebagai peladen tiruan (*server replay*) untuk menyimulasikan respons API tanpa koneksi ke peladen asli.

* **Dekripsi Protokol Modern dan Ekspor Kunci Sesi (`SSLKEYLOGFILE`):** Mendukung protokol web generasi terbaru termasuk HTTP/2, HTTP/3 (QUIC), dan gRPC, serta mampu mengekspor kunci simetris sesi TLS ke dalam berkas `SSLKEYLOGFILE` agar berkas tangkapan paket mentah (`.pcap`) dapat didekripsi dan dibedah lebih lanjut di dalam Wireshark.

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan TUI interaktif mitmproxy pada port 8080
mitmproxy -p 8080

# 2. Menjalankan mitmweb dengan antarmuka peramban web
mitmweb --listen-port 8080

# 3. Menjalankan mitmdump untuk merekam lalu lintas ke file aliran data
mitmdump -w traffic_log.mitm -p 8080

# 4. Menjalankan mitmdump dengan skrip addon Python kustom
mitmdump -s custom_script.py -p 8080

```

> 💡 **Kesimpulan:**
> Secara akademis, `mitmproxy` merepresentasikan jembatan antara analisis protokol jaringan interaktif dan rekayasa otomasi perangkat lunak dalam domain keamanan siber. Kemampuannya untuk mendekripsi saluran TLS modern sekaligus menyediakan antarmuka pemrograman Python yang fleksibel menjadikannya instrumen esensial dalam mengevaluasi integritas komunikasi klien-peladen. Keberadaannya di dalam kategori Collection Kali Linux menegaskan bahwa enkripsi saluran transportasi (HTTPS/TLS) saja tidak cukup melindungi aplikasi apabila klien tidak memvalidasi keaslian rantai sertifikat dengan ketat; pertahanan modern menuntut implementasi *TLS Certificate Pinning* pada aplikasi klien sensitif, otorisasi berbasis peladen yang kuat, serta perlindungan terhadap pembajakan perutean jaringan lokal.

## 3. HTTrack

> **Ringkasan:** Aplikasi pembaca situs web luring (*offline browser utility*). HTTrack digunakan untuk mengunduh seluruh direktori, gambar, kode sumber, dan berkas dari situs web target ke komputer lokal, mempermudah pencarian berkas konfigurasi atau informasi sensitif yang terekspos.

### 📄 Deskripsi Perangkat Lunak

HTTrack adalah sebuah utilitas penyalin situs web (*website copier*) dan peramban luring (*offline browser utility*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Collection**.

Secara arsitektural, HTTrack dirancang untuk mengunduh seluruh struktur situs web dari internet ke dalam direktori penyimpanan lokal secara rekursif. Instrumen ini tidak sekadar mengambil berkas HTML mentah, melainkan membangun ulang struktur hierarki direktori situs secara utuh—mengunduh berkas gaya (CSS), skrip sisi klien (JavaScript), aset gambar, serta sumber daya media lainnya—dan memodifikasi seluruh tautan eksternal menjadi tautan relatif lokal agar situs target dapat dijelajahi secara luring (*offline mirroring*) persis seperti saat diakses secara daring.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Analisis Kode Sumber Sisi Klien Secara Luring (*Offline Client-Side Source Auditing*):** Membantu penguji penetrasi (*penetration tester*) mengumpulkan seluruh aset publik aplikasi web target untuk diinspeksi secara luring tanpa menimbulkan kebisingan log (*access log noise*) berulang pada peladen web maupun memicu aturan pembatasan laju (*rate limiting*) pada WAF. Analis dapat mencari kunci API tersembunyi, komentar pengembang sensitif, atau titik akhir (*endpoints*) yang tertanam di dalam berkas JavaScript.

* **Kloning Halaman untuk Simulasi Rekayasa Sosial (*Phishing Page Cloning & Credential Harvesting*):** Digunakan oleh praktisi Red Team untuk menduplikasi antarmuka halaman autentikasi atau portal korporat target secara presisi, yang kemudian dimodifikasi dan diintegrasikan ke dalam framework simulasi phishing (seperti Gophish atau SET) untuk menguji kesadaran keamanan pengguna.

* **Pengarsipan Bukti Digital dan Forensik Web (*Web Evidence Preservation*):** Berfungsi sebagai instrumen akuisisi forensik bagi investigator untuk mengarsipkan kondisi utuh sebuah situs web (misalnya situs berbahaya atau halaman yang mengalami *defacement*) pada titik waktu tertentu sebagai barang bukti statis sebelum kontennya dihapus oleh pelaku.

### ⚙️ Fungsi Teknis (Functionality)

* **Perayapan Rekursif dan Penulisan Ulang Tautan (*Recursive Spidering & Link Rewriting*):** Menelusuri setiap pranala (*hyperlink*), referensi gambar, berkas CSS, dan pemanggilan skrip di dalam dokumen HTML secara otomatis, lalu mengonversi URL absolut menjadi jalur relatif lokal sehingga navigasi antar-halaman tetap berfungsi penuh secara luring.

* **Pembaruan Inkremental dan Pemulihan Sesi (*Mirror Updating & Session Resume*):** Menyimpan metadata struktur unduhan (indeks cache lokal) yang memungkinkan HTTrack melanjutkan proses pengunduhan yang terputus akibat gangguan jaringan, maupun melakukan sinkronisasi ulang (*update*) dengan hanya mengunduh berkas yang mengalami perubahan di peladen target.

* **Mesin Penyaringan Granular (*Inclusion/Exclusion Scan Rules*):** Menyediakan aturan filter berbasis pola wildcard (seperti `+*.png` atau `-*.zip`), pembatasan kedalaman tautan (*maximum mirroring depth* `-r`), serta pembatasan ukuran berkas maksimum guna mengendalikan cakupan pengumpulan data agar tidak keluar dari domain target.

* **Pengendalian Koneksi, Bandwidth, dan Identitas Klien (`-c`, `-A`, `-F`):** Mengizinkan pengaturan jumlah koneksi paralel (*simultaneous connections*), pembatasan kecepatan transfer data (*bandwidth throttling* dalam byte/detik) agar tidak membebani peladen target, serta kustomisasi string User-Agent dan dukungan autentikasi melalui proksi HTTP/SOCKS.

* **Ketersediaan Antarmuka CLI dan Web GUI (`httrack` & `webhttrack`):** Menyediakan eksekusi murni berbasis baris perintah (`httrack`) yang sangat efisien untuk skrip otomasi terminal, sekaligus menyertakan utilitas `webhttrack` yang menjalankan peladen web lokal untuk konfigurasi proyek penyalinan secara visual melalui peramban.

### 💻 Contoh Penggunaan CLI

```
# 1. Mengunduh mirror situs web secara penuh via CLI
httrack "http://example.com" -O "./example_mirror" "+*.example.com/*"

# 2. Pengunduhan dengan pembatasan kedalaman perayapan (depth 2) dan pembatasan bandwidth
httrack "http://example.com" -O "./example_mirror" -r2 -A250000

# 3. Menjalankan antarmuka grafis WebHTTrack
webhttrack &

```

> 💡 **Kesimpulan:**
> Secara akademis, HTTrack membuktikan bahwa seluruh logika dan aset presentasi sisi klien (*client-side assets*) yang dipublikasikan melalui protokol HTTP dapat diarsipkan, direkonstruksi, dan dianalisis secara menyeluruh oleh pihak eksternal. Keberadaannya di dalam kategori Collection Kali Linux menegaskan prinsip fundamental keamanan aplikasi web bahwa pengembang tidak boleh pernah menyimpan rahasia bisnis, kredensial, kunci API, maupun validasi keamanan kritis di dalam kode sumber sisi klien (HTML/JavaScript), serta perlu menerapkan pembatasan laju permintaan (*rate limiting*) dan deteksi anomali perayapan otomatis pada peladen web.

## 4. Evilginx2

> **Ringkasan:** Kerangka kerja serangan phishing berbasis proksi tingkat lanjut yang digunakan untuk melewati pengamanan autentikasi dua faktor (2FA). Alat ini bekerja dengan cara mencegat dan mengumpulkan *session cookie* pengguna saat mereka masuk ke layanan palsu, sehingga penyerang dapat mereplikasi sesi tersebut.

### 📄 Deskripsi Perangkat Lunak

Evilginx2 adalah sebuah framework serangan *Adversary-in-the-Middle* (AitM) berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Go (Golang) untuk melakukan simulasi phishing tingkat lanjut dan pengumpulan token sesi (*session hijacking*). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Collection**.

Secara arsitektural, berbeda dengan alat phishing konvensional (seperti SET atau Gophish) yang hanya mengkloning halaman HTML statis untuk menangkap nama pengguna dan kata sandi, Evilginx2 bertindak sebagai *Reverse Proxy* HTTP/HTTPS penuh yang berdiri di antara peramban korban dan peladen web asli (*legitimate identity provider*). Dengan meneruskan lalu lintas secara transparan menggunakan sertifikat SSL/TLS yang sah (via Let's Encrypt) dan menulis ulang domain secara dinamis, Evilginx2 mampu mencegat kredensial sekaligus *session cookies* yang diterbitkan setelah korban menyelesaikan proses Otentikasi Multi-Faktor (MFA/2FA).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Simulasi Pengumpulan Token Sesi dan Bypassing MFA (*AitM Session Token Collection*):** Membantu praktisi Red Team mendemonstrasikan bagaimana aktor ancaman dapat memintas mekanisme otentikasi dua faktor berbasis kode sekali pakai (TOTP, SMS, atau Push Notification) dengan mencuri *session cookie* terautentikasi secara langsung saat korban masuk melalui tautan phishing.

* **Evaluasi Ketahanan Metode Otentikasi (*MFA Mechanism Benchmarking*):** Digunakan oleh arsitek keamanan korporat untuk membandingkan kerentanan metode MFA tradisional terhadap metode otentikasi kriptografis berbasis asal domain (*origin-bound authentication*), seperti standar FIDO2/WebAuthn (Passkeys/YubiKey).

* **Audit Deteksi Infrastruktur Phishing dan Anomali Sesi (*Blue Team Threat Hunting*):** Memungkinkan tim pertahanan menguji efektivitas sistem pemantauan log transparansi sertifikat (*Certificate Transparency*), deteksi *typosquatting* DNS, serta aturan deteksi pembajakan sesi (*Conditional Access Policies* / Token Binding) pada platform identitas seperti Microsoft Entra ID atau Okta.

### ⚙️ Fungsi Teknis (Functionality)

* **Mesin Reverse Proxy Dinamis dan Penulisan Ulang Domain (*On-the-Fly Domain & Content Rewriting*):** Menerima permintaan HTTPS dari peramban korban pada domain penguji (misalnya `https://target-corp.com`), meneruskannya ke peladen asli (`https://microsoftonline.com`), serta menulis ulang seluruh referensi domain di dalam HTTP Headers, Cookies, dan badan HTML/JavaScript secara *real-time* agar korban tetap berada di dalam jalur proksi selama seluruh alur login berlangsung.

* **Ekosistem Templat Konfigurasi Modular (*Phishlets*):** Menggunakan berkas konfigurasi berbasis format YAML yang disebut *Phishlets*. Setiap Phishlet mendefinisikan daftar subdomain yang diproksikan, aturan filter reguler (*regex*) untuk mengganti string domain di dalam kode JavaScript, serta parameter nama *session cookie* spesifik yang harus ditangkap dan diekstraksi setelah proses login berhasil.

* **Peladen DNS Otoritatif dan Manajemen Sertifikat SSL Otomatis:** Mengintegrasikan peladen DNS internal untuk menyelesaikan kueri subdomain phishlet secara otomatis, serta terhubung langsung dengan API Let's Encrypt untuk menerbitkan dan memperbarui sertifikat TLS valid secara otomatis sehingga peramban korban menampilkan ikon gembok keamanan tanpa peringatan sertifikat.

* **Pembuatan Tautan Umpan Unik dan Penyaringan Lalu Lintas (*Lures, Obfuscation & Cloaking*):** Menyediakan fitur *Lures* untuk menghasilkan URL terenkripsi atau acak per target kampanye, dilengkapi dengan mekanisme penyaringan IP/User-Agent (*redirectors* dan daftar hitam bot) guna mengalihkan pemindai keamanan otomatis (seperti *sandbox* email atau perayap Google) ke halaman web yang tidak berbahaya.

* **Ekstraksi dan Ekspor Token Sesi Siap Pakai (*Session Cookie Harvesting & JSON Export*):** Secara otomatis menangkap, mendekode, dan menyimpan seluruh kredensial serta struktur cookies sesi korban ke dalam basis data lokal, lengkap dengan format ekspor JSON yang dapat langsung diimpor ke dalam peramban penguji (melalui ekstensi pengelola cookie) untuk mengambil alih sesi akun korban tanpa perlu memasukkan kata sandi maupun kode MFA.

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan konsol Evilginx2
evilginx

# 2. Konfigurasi domain dan IP di dalam konsol Evilginx2
config domain example-phish.com
config ip 192.168.1.100

# 3. Mengaktifkan phishlet dan mengonfigurasi lure URL
phishlets enable o365
lures create o365
lures get-url 0

```

> 💡 **Kesimpulan:**
> Secara akademis, Evilginx2 membuktikan bahwa mekanisme otentikasi dua faktor konvensional (seperti SMS OTP, aplikasi autentikator TOTP, dan notifikasi push) tidak lagi kebal terhadap serangan phishing karena protokol tersebut tidak mengikat bukti kriptografis otentikasi pada nama domain asli peladen. Keberadaannya di dalam kategori Collection Kali Linux menegaskan urgensi peralihan standar otentikasi organisasi menuju protokol FIDO2/WebAuthn (Passkeys) yang secara kriptografis memverifikasi asal domain (*origin binding*) dan kebal terhadap serangan *Adversary-in-the-Middle*, disertai penerapan kebijakan akses bersyarat (*Conditional Access*) berbasis kepatuhan perangkat (*device compliance*) dan pembatasan alamat IP sesi.

## 5. Fluxion & Wifipumpkin3

> **Ringkasan:** Alat bantu serangan berbasis nirkabel (WiFi). Keduanya digunakan untuk membuat jalur akses internet palsu (*Rogue Access Point*) guna memaksa pengguna terhubung ke jaringan buatan *pentester*, lalu mengumpulkan data kredensial atau riwayat berselancar mereka.

### 📄 Deskripsi Perangkat Lunak

* **Fluxion:** Sebuah instrumen pengujian keamanan Wi-Fi *open-source* berbasis skrip Bash yang dirancang untuk melakukan serangan *Social Engineering Captive Portal* secara otomatis pada jaringan WPA/WPA2-Personal. Fluxion bekerja dengan memutus koneksi klien dari AP asli (deauthentication) dan membuat AP tiruan (*Evil Twin*) dengan SSID yang sama untuk memicu halaman login palsu yang memvalidasi kata sandi Wi-Fi terhadap *handshake* yang ditangkap.

* **WiFi-Pumpkin3 (`wifipumpkin3`):** Sebuah kerangka kerja (*framework*) audit nirkabel berbasis Python 3 yang dirancang untuk menyimulasikan *Rogue Access Point* tingkat lanjut dan serangan *Man-in-the-Middle* (MitM) pada jaringan nirkabel. WiFi-Pumpkin3 menyediakan fitur penyadapan lalu lintas, pembuatan *Captive Portal* terintegrasi, serta modifikasi lalu lintas web secara *real-time*.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pengumpulan Kredensial Wi-Fi dan Otentikasi Web (*Wireless Credential Collection*):** Membantu penguji penetrasi menguji ketahanan pengguna terhadap serangan *Rogue AP* dan memanen frasa sandi WPA/WPA2 atau kredensial autentikasi web melalui halaman *Captive Portal* yang meyakinkan.

* **Pencegatan dan Analisis Lalu Lintas Nirkabel (*MitM Traffic Interception*):** Digunakan untuk menguji sejauh mana lalu lintas data pengguna pada jaringan Wi-Fi publik dapat disadap, didekripsi, dan diamati oleh peretas yang mengoperasikan titik akses palsu.

* **Audit Kesadaran Keamanan Pengguna (*User Security Awareness Assessment*):** Memungkinkan tim Red Team dan administrator jaringan mengukur tingkat kewaspadaan karyawan dalam mengenali sinyal Wi-Fi palsu serta halaman peringatan keamanan peramban.

### ⚙️ Fungsi Teknis (Functionality)

* **Serangan Evil Twin & Captive Portal Otomatis (Fluxion):** Menggabungkan pemindaian jaringan nirkabel, penangkapan paket *4-way handshake*, pembuatan AP tiruan tanpa enkripsi, serta penyajian halaman *Captive Portal* otomatis yang memverifikasi input kata sandi korban secara *real-time* menggunakan `cowpatty` / `aircrack-ng`.

* **Manajemen Rogue AP Multi-Modul (WiFi-Pumpkin3):** Mengintegrasikan peladen DHCP (`dnsmasq`), peladen DNS palsu, serta mesin proksi HTTP/HTTPS untuk memberikan koneksi internet aktif kepada korban sembari menyadap seluruh paket data yang melintas.

* **Pencegatan Bingkai Manajemen & Deauthentication:** Menggunakan injeksi paket 802.11 (*deauthentication frames*) untuk memaksa perangkat korban terputus dari Access Point sah dan secara otomatis terhubung kembali ke *Rogue AP* yang memiliki sinyal lebih kuat.

* **Modifikasi Lalu Lintas dan Injeksi Muatan (WiFi-Pumpkin3 Plugins):** Dilengkapi dengan plugin modular untuk melakukan *DNS Spoofing*, *Bypass Captive Portal*, serta penyuntikan skrip ke dalam sesi penjelajahan web korban.

### 📊 Perbandingan Singkat & Kesimpulan Akademis: Fluxion vs WiFi-Pumpkin3

| Aspek Komparasi | Fluxion | WiFi-Pumpkin3 (`wifipumpkin3`) | 
| ----- | ----- | ----- | 
| **Fokus Serangan Utama** | Ekstraksi kunci Wi-Fi (WPA/WPA2) melalui kombinasi *deauth* dan portal pembaruan router palsu. | Pembangunan *Rogue AP* lengkap dengan koneksi internet aktif untuk intersepsi lalu lintas MitM dan *Captive Portal*. | 
| **Koneksi Internet Korban** | Korban terisolasi dari internet dan dipaksa memasukkan kata sandi Wi-Fi yang divalidasi terhadap *handshake*. | Korban tetap mendapatkan akses internet normal sementara seluruh lalu lintasnya disadap dan dimanipulasi. | 
| **Arsitektur Bahasa** | Skrip Bash yang mengorkestrasi alat CLI Linux (`aircrack-ng`, `hostapd`, dll.). | Framework modular berbasis Python 3 dengan antarmuka konsol interaktif. | 

### 💻 Contoh Penggunaan CLI

```
# 1. Menjalankan Fluxion melalui skrip utama
sudo ./fluxion.sh

# 2. Menjalankan antarmuka konsol WiFi-Pumpkin3
sudo wifipumpkin3

# 3. Konfigurasi dasar di dalam WiFi-Pumpkin3 (mengatur interface dan SSID)
wp3 > set interface wlan0
wp3 > set ssid "Free_WiFi_Hotspot"
wp3 > start

```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Fluxion dan WiFi-Pumpkin3 membuktikan bahwa keamanan komunikasi nirkabel tidak cukup hanya bergantung pada kekuatan algoritma enkripsi kriptografis (seperti WPA2-AES), karena faktor interaksi manusia dan ketiadaan otentikasi bingkai manajemen (*management frames*) dapat dieksploitasi untuk memintas proteksi tersebut. Keberadaan keduanya di dalam kategori Collection Kali Linux menegaskan urgensi peralihan standar keamanan jaringan nirkabel menuju WPA3-Personal/Enterprise—yang secara wajib menerapkan *Protected Management Frames* (IEEE 802.11w) untuk memblokir serangan *deauthentication* serta pertukaran kunci SAE (*Simultaneous Authentication of Equals*)—disertai penggunaan VPN terenkripsi saat terhubung ke jaringan Wi-Fi publik.
