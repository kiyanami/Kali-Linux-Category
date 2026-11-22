# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Discovery
# 🛠️ Discovery Tools - Kali Linux

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak pemetaan, pengintaian, dan penemuan aset (*Discovery*) yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [BloodHound & BloodHound.py (Active Directory)](#1-bloodhound--bloodhoundpy-active-directory)
   * [Part I: BloodHound (bloodhound)](#part-i-bloodhound-bloodhound)
   * [Part II: BloodHound.py (bloodhound-python)](#part-ii-bloodhoundpy-bloodhound-python)
   * [Perbandingan Ringkas](#perbandingan-ringkas-bloodhound-vs-bloodhoundpy)
2. [Netdiscover (System Network Configuration Discovery)](#2-netdiscover-system-network-configuration-discovery)
3. [Pspy - pspy / pspy-binaries (Process Discovery)](#3-pspy---pspy--pspy-binaries-process-discovery)
4. [Enum4linux / Enum4linux-ng (Network Share Discovery)](#4-enum4linux--enum4linux-ng-network-share-discovery)
   * [Perbandingan Singkat](#perbandingan-singkat-enum4linux-vs-enum4linux-ng)
5. [Wafw00f (Network Security Appliances)](#5-wafw00f-network-security-appliances)
6. [Wireshark & Tcpdump (Network Sniffing)](#6-wireshark--tcpdump-network-sniffing)
   * [Part I: Wireshark](#part-i-wireshark)
   * [Part II: Tcpdump](#part-ii-tcpdump)
   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-wireshark-vs-tcpdump)

---

## 1. BloodHound & BloodHound.py (Active Directory)

> **Ringkasan:** Alat analisis grafis yang digunakan untuk memetakan hubungan dan jalur serangan tersembunyi di dalam lingkungan Active Directory (AD) Windows. BloodHound sangat andal untuk menemukan cara tercepat menguasai Domain Controller berdasarkan hak akses pengguna yang saling terhubung.

Pada taksonomi Kali Linux (`kali.org/tools`), BloodHound (paket `bloodhound` / BloodHound Community Edition) bersama ingestor berbasis Python-nya (`bloodhound-python` dari paket `bloodhound.py`) merupakan perangkat analisis relasi hak akses dan jalur serangan (*attack path analysis*) pada lingkungan Active Directory (AD). 

keduanya bekerja sebagai satu alur komplementer: `bloodhound-python` bertugas mengumpulkan data mentah direktori dan sesi dari jarak jauh, sedangkan `BloodHound` memodelkan data tersebut menjadi graf relasional untuk menemukan jalur eskalasi menuju hak akses istimewa (seperti Domain Admin).

---

### Part I: BloodHound (bloodhound)

#### 📄 Deskripsi Perangkat Lunak

BloodHound adalah platform analisis graf berbasis teori graf (*graph theory*) yang memodelkan objek-objek Active Directory (seperti User, Group, Computer, OU, dan GPO) sebagai *nodes* serta hubungan izin, keanggotaan, dan sesi autentikasi sebagai *edges* terarah. 

Dalam perspektif keamanan sistem terdistribusi, kompromi hak akses tinggi pada AD jarang terjadi secara langsung, melainkan melalui rantai relasi tak terduga (*chokepoints* dan *attack paths*) yang terbentuk dari akumulasi konfigurasi izin dan sesi login. Pada Kali Linux modern, paket `bloodhound` menyediakan BloodHound Community Edition (CE) lengkap dengan layanan basis data pendukung (seperti Neo4j dan PostgreSQL) serta utilitas manajemen layanan (`bloodhound-setup`, `bloodhound-start`, `bloodhound-stop`).

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pemetaan Jalur Serangan (*Attack Path Mapping*):** Mengidentifikasi jalur terpendek maupun seluruh jalur potensial dari akun berhak akses rendah/menengah menuju Domain Admins atau aset bernilai tinggi.
* **Audit & Hardening Postur Keamanan AD (Blue Team):** Membantu administrator sistem dan tim defensif menemukan miskonfigurasi ACL (*Access Control List*), pendelegasian berlebih, dan jejak sesi privilej untuk memutus rantai eskalasi sebelum dieksploitasi.
* **Simulasi & Prioritisasi Eksploitasi (Red Team / Pentesting):** Membantu penguji penetrasi menentukan target bernilai tinggi secara terukur tanpa melakukan pemindaian buta yang bising di jaringan.

#### ⚙️ Fungsi Teknis (Functionality)

* **Ingesti & Pemodelan Graf Relasional:** Menerima berkas data terstruktur hasil koleksi (JSON/arsip) dan menyimpannya ke basis data graf sehingga relasi multi-lompatan (*multi-hop*) dapat dianalisis secara efisien.
* **Eksekusi Kueri Graf & Pathfinding:** Menjalankan kueri bawaan maupun kustom (misalnya mencari jalur terpendek ke Domain Admin, akun Kerberoastable, pemegang hak DCSync, atau ACL yang dapat disalahgunakan).
* **Visualisasi Interaktif Relasi Privilej:** Menampilkan topologi ketergantungan antar-objek AD secara visual agar analis dapat memahami konteks operasional dan dampak perubahan izin.

---

### Part II: BloodHound.py (bloodhound-python)

#### 📄 Deskripsi Perangkat Lunak

`bloodhound-python` (dari pustaka `bloodhound.py` yang dibangun di atas Impacket) adalah *remote ingestor* berbasis Python 3 yang mengkueri layanan jaringan Active Directory—seperti LDAP/LDAPS, SMB/RPC, dan DNS—dari host Linux tanpa memerlukan eksekusi agen lokal pada mesin Windows target. 

Secara metodologis, alat ini menerjemahkan mekanisme enumerasi direktori dan sesi AD ke dalam skema data yang kompatibel dengan BloodHound, sehingga pengujian atau audit dapat dilakukan langsung dari lingkungan Kali Linux.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pengumpulan Data AD dari Linux (*Cross-Platform Enumeration*):** Memungkinkan penguji penetrasi atau auditor di Kali Linux mengumpulkan informasi domain, objek, izin, dan sesi tanpa perlu menjalankan kolektor *native* Windows di dalam jaringan target.
* **Enumerasi Berbasis Kredensial Fleksibel:** Mendukung autentikasi menggunakan *password*, NT *hash* (*Pass-the-Hash*), kunci AES Kerberos, maupun tiket Kerberos tersimpan (`ccache`), sehingga dapat diadaptasi sesuai konteks kredensial yang tersedia.
* **Penyesuaian Jejak Jaringan (*Footprint Control*):** Memungkinkan pembatasan metode koleksi (misalnya hanya mengkueri Domain Controller lewat `DCOnly`) ketika kebutuhan audit menuntut minimasi lalu lintas ke workstation/server anggota.

#### ⚙️ Fungsi Teknis (Functionality)

* **Koleksi Objek & Topologi Domain (`Group`, `Acl`, `Trusts`, `ObjectProps`, `Container`, `DCOnly`):** Mengambil keanggotaan grup, daftar kontrol akses (ACL), relasi *domain trust*, properti objek, serta struktur OU/GPO/kontainer.
* **Enumerasi Hak Akses Jarak Jauh & Sesi (`LocalAdmin`, `RDP`, `DCOM`, `PSRemote`, `Session`, `LoggedOn`):** Mengidentifikasi hak akses administratif/remote serta sesi pengguna aktif atau sesi istimewa yang terdeteksi pada komputer target.
* **Ekspor Data Terstruktur untuk BloodHound:** Menghasilkan berkas JSON (atau arsip `.zip`) hasil enumerasi yang siap diimpor ke antarmuka BloodHound untuk dianalisis lebih lanjut.

---

### 📊 Perbandingan Ringkas: BloodHound vs BloodHound.py

| Aspek Komparasi | BloodHound (`bloodhound`) | BloodHound.py (`bloodhound-python`) |
| --- | --- | --- |
| **Peran Arsitektural** | Mesin analisis & visualisasi graf relasi AD | *Remote data ingestor* berbasis Python & Impacket dari Linux |
| **Input Utama** | Data hasil koleksi (JSON / arsip `.zip`) dari ingestor | Kredensial/hash/tiket AD, domain, dan nameserver/DC target |
| **Output Utama** | Graf *attack path*, kueri privilej, dan wawasan mitigasi | Kumpulan berkas JSON (atau arsip) terstruktur per kategori objek/sesi |
| **Cakupan Teknis** | Fokus pada teori graf, kueri relasi, dan *hardening* AD | Fokus pada kueri LDAP/SMB/DNS/Kerberos & kontrol cakupan koleksi |

### 💻 Contoh Penggunaan CLI

```bash
# 1. Mengumpulkan data Active Directory dari Linux menggunakan bloodhound-python
bloodhound-python -d corp.local -u 'jdoe' -p 'Password123!' -gc dc01.corp.local -c All -z

# 2. Mengumpulkan data menggunakan Pass-the-Hash (PtH)
bloodhound-python -d corp.local -u 'jdoe' --hashes '00000000000000000000000000000000:8846F7EAEE8FB117AD06BDD830B7586C' -ns 192.168.1.10 -c DCOnly

# 3. Menjalankan BloodHound GUI di Kali Linux
bloodhound-setup
bloodhound
```

---

## 2. Netdiscover (System Network Configuration Discovery)

> **Ringkasan:** Alat pemindai jaringan aktif maupun pasif yang memanfaatkan permintaan ARP (*Address Resolution Protocol*). Alat ini sangat berguna di dalam jaringan lokal untuk mendeteksi alamat IP dan MAC address komputer lain yang terhubung tanpa memicu banyak alarm.

### 📄 Deskripsi Perangkat Lunak

Netdiscover adalah sebuah instrumen pengintaian jaringan aktif dan pasif (*active/passive address reconnaissance tool*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C dengan memanfaatkan pustaka `libpcap` dan `libnet`. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Discovery** (secara spesifik pada sub-kategori *System Network Configuration Discovery*).

Secara arsitektural, Netdiscover beroperasi pada Lapisan Datalink (Layer 2) dan antarmuka jaringan lokal dengan mengeksploitasi cara kerja ARP (*Address Resolution Protocol*). Berbeda dengan pemindai jaringan lapisan IP (Layer 3) seperti Nmap yang kerap terhambat oleh aturan pemblokiran paket ICMP (*ping block*) atau *host-based firewall*, Netdiscover memanfaatkan sifat fundamental protokol ARP yang wajib dijawab oleh setiap perangkat aktif di dalam satu segmen jaringan lokal (LAN/VLAN) maupun jaringan nirkabel (Wi-Fi) agar komunikasi ethernet dapat berlangsung.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pemetaan Aset Jaringan Tanpa Layanan DHCP (*Non-DHCP Network Discovery*):** Membantu penguji penetrasi (*penetration tester*) mengidentifikasi alamat IP, alamat MAC, dan segmen subnet aktif ketika terhubung ke jaringan lokal yang tidak menyediakan server DHCP otomatis atau menerapkan isolasi host ketat.
* **Pengintaian Siluman Pasif (*Stealth Passive Reconnaissance*):** Digunakan oleh praktisi Red Team untuk memetakan perangkat di dalam jaringan tanpa mengirimkan satu pun paket permintaan (*zero-packet transmission*), sehingga menghindari deteksi dari sistem pencegahan intrusi (IPS) atau sistem pemantauan anomali lalu lintas ARP.
* **Deteksi Perangkat Asing dan Rogue Devices (*Blue Team Asset Auditing*):** Berfungsi sebagai instrumen audit cepat bagi administrator jaringan untuk mendeteksi keberadaan perangkat tidak sah (*rogue access points*, komputer penyusup, atau perangkat IoT tersembunyi) serta memverifikasi identitas produsen perangkat keras yang terhubung ke jaringan lokal.

### ⚙️ Fungsi Teknis (Functionality)

* **Mode Pemindaian Aktif (`-r` / Active ARP Scanning):** Mengirimkan paket permintaan ARP (*ARP Requests*) secara sistematis ke rentang alamat IP spesifik (notasi CIDR, misalnya `192.168.1.0/24`) untuk memaksa setiap host yang aktif membalas dengan paket *ARP Reply* yang berisi alamat MAC mereka.
* **Mode Pemindaian Pasif (`-p` / Passive Sniffing Mode):** Mengatur kartu jaringan ke dalam mode *promiscuous* untuk mendengarkan (*sniffing*) siaran paket ARP (*ARP broadcasts*) yang melintas secara alami di dalam jaringan tanpa menyuntikkan paket apa pun, memungkinkan pemetaan perangkat secara senyap.
* **Identifikasi Produsen Perangkat Keras Otomatis (OUI Vendor Lookup):** Mengintegrasikan basis data lokal *Organizationally Unique Identifier* (OUI) dari IEEE yang mencocokkan 3 byte pertama dari setiap alamat MAC yang ditemukan dengan nama pabrikan perangkat keras (seperti Cisco, Dell, Apple, atau Intel), membantu analis mengklasifikasikan jenis perangkat target (router, server, stasiun kerja, atau perangkat seluler).
* **Pengendalian Laju Paket dan Mode Pemindaian Cepat (`-s`, `-n`, `-f`):** Menyediakan parameter penyesuaian jeda waktu (*sleep time*) antar-pengiriman paket ARP serta mode pemindaian cepat (*fast scan*) yang hanya memeriksa alamat IP umum (gateway seperti `.1` atau `.254`) pada setiap jaringan kelas C untuk mempercepat penemuan subnet aktif.
* **Ekspor Hasil Terstruktur untuk Otomasi (`-P` & `-L`):** Mendukung mode keluaran ramah parser (*parsable output*) yang dapat langsung dialirkan ke berkas teks atau diproses oleh skrip otomasi lainnya guna membangun daftar target pemindaian kerentanan lanjutan.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Pemindaian aktif ARP pada subnet lokal tertentu
sudo netdiscover -r 192.168.1.0/24

# 2. Pemindaian pasif senyap (sniffing tanpa mengirim paket) pada antarmuka eth0
sudo netdiscover -p -i eth0

# 3. Pemindaian cepat dengan format output parsable untuk disimpan ke file
sudo netdiscover -r 10.0.0.0/16 -P > netdiscover_scan.txt
```

> 💡 **Kesimpulan:**
> Secara akademis, Netdiscover merepresentasikan pemanfaatan fundamental terhadap protokol resolusi alamat pada lapisan tautan data jaringan (Layer 2). Keberadaannya di Kali Linux membuktikan bahwa penyembunyian layanan di tingkat sistem operasi (*host firewall*) tidak dapat menyembunyikan keberadaan fisik sebuah perangkat di dalam jaringan lokal selama perangkat tersebut masih berpartisipasi dalam komunikasi ethernet standar. Hal ini menegaskan pentingnya penerapan segmentasi jaringan (VLAN), fitur keamanan switch seperti *Dynamic ARP Inspection* (DAI), serta autentikasi port berbasis IEEE 802.1X untuk melindungi infrastruktur jaringan lokal dari pengintaian internal.

---

## 3. Pspy - pspy / pspy-binaries (Process Discovery)

> **Ringkasan:** Perkakas baris perintah (*command-line*) yang dirancang untuk memantau proses Linux yang sedang berjalan secara *real-time* tanpa memerlukan hak akses `root`. Pspy membantu mendeteksi *cron job* (tugas otomatis) atau perintah rahasia yang dieksekusi pengguna lain saat sistem aktif.

### 📄 Deskripsi Perangkat Lunak

`pspy` adalah sebuah instrumen pemantauan proses sistem operasi Linux berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Go (Golang). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Discovery** (secara spesifik pada sub-kategori *Process Discovery*) beserta paket kompilasi lintas arsitekturnya (`pspy-binaries`).

Secara arsitektural, `pspy` dirancang sebagai utilitas portabel tanpa dependensi (*statically linked binary*) yang mampu mengintip (*snooping*) aktivitas eksekusi perintah dan proses di sistem Linux tanpa memerlukan hak akses administratif (`root`). Berbeda dengan utilitas pemantauan tradisional seperti `ps` atau `top` yang sering melewatkan proses berumur sangat pendek (*short-lived processes*), `pspy` mengombinasikan pemindaian direktori sistem berkas virtual `/proc` dengan subsistem peristiwa kernel Linux (`inotify` API). Pendekatan berbasis pemicu peristiwa (*event-driven*) ini memungkinkan `pspy` menangkap detail perintah yang dieksekusi oleh pengguna lain (termasuk `root` dan tugas terjadwal *cron jobs*) sesaat setelah proses tersebut lahir.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Identifikasi Tugas Terjadwal Tersembunyi (*Hidden Cron Jobs & Automation Discovery*):** Membantu penguji penetrasi (*penetration tester*) mendeteksi skrip atau perintah yang dijalankan secara otomatis oleh `root` (misalnya melalui crontab sistem yang tidak dapat dibaca oleh pengguna biasa atau *systemd timers*), guna menemukan celah eskalasi hak istimewa lokal (*local privilege escalation*).
* **Pencegatan Kredensial dari Argumen Baris Perintah (*Command-Line Credential Sniffing*):** Digunakan untuk memantau argumen proses secara *real-time* ketika skrip administratif atau aplikasi lain meneruskan informasi sensitif—seperti kata sandi basis data, kunci API, atau token autentikasi—secara langsung melalui parameter CLI (misalnya `mysql -u root -pRahasia123`).
* **Analisis Perilaku Aplikasi dan Deteksi Anomali (*Blue Team Behavioral Monitoring*):** Berfungsi sebagai instrumen diagnostik ringan bagi administrator sistem dan peneliti forensik untuk mengamati urutan eksekusi anak proses (*child processes*) pada kontainer terisolasi (Docker/Kubernetes) maupun server produksi tanpa harus memasang agen audit kernel yang kompleks seperti `auditd`.

### ⚙️ Fungsi Teknis (Functionality)

* **Pemantauan Berbasis Peristiwa Kernel (`inotify` File System Watcher):** Memasang pengamat (*watchers*) pada direktori kritis seperti `/usr/bin`, `/bin`, `/tmp`, `/etc`, serta sub-direktori di dalam `/proc`. Setiap kali berkas biner diakses atau dieksekusi, kernel memicu notifikasi `inotify` yang langsung dibaca oleh `pspy` untuk mencatat identitas proses sebelum proses tersebut sempat berakhir.
* **Pemindaian Rekursif Direktori Virtual `/proc` (`-p` & `-r`):** Melakukan pembacaan terhadap berkas metadata proses di `/proc/[PID]/cmdline` dan `/proc/[PID]/status` untuk mengekstrak nama perintah lengkap beserta seluruh argumennya, nomor identifikasi proses (PID), nomor proses induk (PPID), serta identifikasi pengguna (UID) yang menjalankannya.
* **Ketersediaan Biner Lintas Arsitektur (`pspy-binaries`):** Menyediakan paket berkas eksekusi mandiri dalam berbagai arsitektur prosesor (x86/32-bit, amd64/64-bit, ARM, dan ARM64), termasuk versi berukuran kecil (*small builds*) yang telah dikompresi, sehingga analis di Kali Linux dapat langsung mentransfer biner yang tepat sesuai arsitektur mesin target.
* **Penyesuaian Interval Pemindaian dan Pengendalian Beban CPU (`-i`):** Menyediakan opsi pengaturan interval pemindaian `/proc` dalam satuan milidetik (misalnya `-i 1000`), memungkinkan penguji menyesuaikan keseimbangan antara ketelitian penangkapan proses kilat dengan konsumsi sumber daya CPU agar tidak menimbulkan kecurigaan pada sistem pemantauan kinerja target.
* **Perekaman Log dan Filterisasi Keluaran (`-f` & `--color`):** Mendukung pencatatan seluruh aktivitas proses yang tertangkap ke dalam berkas log statis untuk analisis luring, serta membedakan informasi berdasarkan kode warna berdasarkan nilai UID guna mempercepat identifikasi proses yang dieksekusi oleh `root` (UID=0).

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan pspy 64-bit bawaan Kali Linux untuk pemantauan standar
./pspy64

# 2. Pemantauan dengan jeda interval 1000ms dan mencatat hasil ke log file
./pspy64 -i 1000 -f -l pspy_cron_audit.log

# 3. Menjalankan versi ringkas (small build) tanpa warna terminal untuk menghemat beban memory
./pspy64s -c
```

> 💡 **Kesimpulan:**
> Secara akademis, `pspy` membuktikan bahwa visibilitas terhadap tabel proses global di lingkungan Linux secara bawaan bersifat terbuka bagi seluruh pengguna lokal, sehingga proses yang berjalan singkat maupun argumen baris perintah bukanlah ruang yang terisolasi secara rahasia. Keberadaannya di Kali Linux menegaskan pentingnya penerapan praktik keamanan administrasi sistem yang disiplin, meliputi pembatasan izin baca pada `/proc` melalui opsi pemasangan `hidepid=2`, penghindaran penyertaan kata sandi dalam argumen baris perintah (beralih ke variabel lingkungan terproteksi atau berkas konfigurasi terenkripsi), serta pengamanan izin tulis pada skrip yang dijalankan oleh tugas terjadwal otomatis.

---

## 4. Enum4linux / Enum4linux-ng (Network Share Discovery)

> **Ringkasan:** Alat bantu berbasis Perl/Python untuk melakukan enumerasi informasi dari protokol Samba/SMB pada sistem Windows dan Linux. Alat ini digunakan untuk menemukan folder bersama (*network shares*), daftar pengguna, grup, serta kebijakan kata sandi.

### 📄 Deskripsi Perangkat Lunak

Enum4linux (dikembangkan dalam bahasa Perl) dan Enum4linux-ng (*Next Generation*, ditulis ulang sepenuhnya menggunakan Python 3) adalah instrumen enumerasi informasi jaringan berbasis *open-source* yang dirancang khusus untuk mengekstrak data sistem dari mesin berbasis Microsoft Windows maupun peladen Samba (implementasi SMB/CIFS pada Linux/Unix). Di dalam taksonomi resmi Kali Linux, kedua perangkat lunak ini diklasifikasikan ke dalam kategori **Discovery** (secara spesifik pada sub-kategori *Network Share Discovery*).

Secara arsitektural, keduanya berfungsi sebagai pembungkus (*wrapper*) terpadu yang mengotomatisasi serangkaian utilitas klien paket Samba—seperti `rpcclient`, `net`, `nmblookup`, dan `smbclient` (serta pustaka Impacket dan `ldap3` pada Enum4linux-ng). Instrumen ini mengeksploitasi protokol SMB (*Server Message Block*), MS-RPC (*Microsoft Remote Procedure Call*), NetBIOS, dan LDAP melalui skema sesi anonim (*Null Sessions*) maupun kredensial pengguna standar guna memetakan struktur internal target.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Penemuan Berbagi Pakai Jaringan (*Network Share Discovery*):** Membantu penguji penetrasi (*penetration tester*) mengidentifikasi direktori bersama (*shared folders*) yang terekspos di jaringan serta memeriksa hak akses baca/tulisnya guna menemukan berkas konfigurasi sensitif, dokumen rahasia, atau skrip cadangan yang tidak terlindungi.
* **Pemetaan Entitas Identitas dan Kebijakan Domain (*Identity & Policy Profiling*):** Digunakan pada fase pengintaian internal untuk menyusun daftar nama pengguna (*usernames*) dan grup domain yang sah sebagai bahan bakar serangan *password spraying* atau *brute-force* yang terukur tanpa memicu penguncian akun (*account lockout*).
* **Audit Kepatuhan Keamanan SMB/Samba (*Blue Team Configuration Auditing*):** Memungkinkan administrator sistem memverifikasi apakah konfigurasi peladen Windows maupun Samba masih mengizinkan akses informasi secara anonim (*Null Session enumeration*) atau mengekspos metadata sistem operasi secara berlebihan.

### ⚙️ Fungsi Teknis (Functionality)

* **Enumerasi dan Pengujian Izin Berbagi Pakai (*SMB Share Enumeration & Access Testing*):** Memindai seluruh daftar folder bersama (termasuk *administrative shares* seperti `C$`, `ADMIN$`, dan `IPC$`), kemudian menguji secara otomatis apakah sesi aktif memiliki hak akses daftar (*listing*), baca (*read*), atau tulis (*write*) pada setiap direktori tersebut.
* **Ekstraksi Pengguna dan Grup via RID Cycling (`-r` / `-U`):** Mengumpulkan daftar akun pengguna dan grup lokal/domain melalui kueri RPC standar, serta dilengkapi mekanisme *RID Cycling*—yaitu teknik enumerasi *Security Identifier* (SID) dan *Relative Identifier* (RID) secara berurutan (misalnya RID 500 hingga 1050) untuk mengungkap nama akun tersembunyi meskipun pembatasan enumerasi standar telah diaktifkan.
* **Inspeksi Kebijakan Kata Sandi dan Keamanan (*Password Policy Extraction* `-P`):** Mengambil parameter kebijakan keamanan domain atau lokal secara jarak jauh, meliputi panjang minimum kata sandi, kompleksitas karakter, masa berlaku kata sandi, serta ambang batas kegagalan login (*account lockout threshold*).
* **Identifikasi Sistem Operasi dan NetBIOS (*OS & NetBIOS Fingerprinting*):** Mengirimkan kueri NetBIOS Name Service (port UDP 137) dan negosiasi SMB untuk menentukan nama hostname, nama workgroup atau domain Active Directory, peran peladen (Domain Controller vs Workstation), serta versi spesifik sistem operasi atau perangkat lunak Samba.
* **Ekspor Data Terstruktur dan Integrasi Otomasi (`enum4linux-ng`):** Selain mengadopsi seluruh fitur pendahulunya dengan kode yang lebih bersih, Enum4linux-ng menambahkan dukungan keluaran terstruktur dalam format JSON dan YAML (`-oJ` / `-oY`), serta integrasi kueri LDAP yang memungkinkan hasil enumerasi langsung diimpor ke dalam basis data pelaporan maupun kerangka kerja otomasi pengujian penetrasi modern.

---

### 📊 Perbandingan Singkat: Enum4linux vs Enum4linux-ng

| Aspek Komparasi | Enum4linux (Klasik) | Enum4linux-ng (Next Generation) |
| --- | --- | --- |
| **Bahasa Pemrograman** | Perl (bergantung pada biner eksternal Samba). | Python 3 (memanfaatkan pustaka modern seperti Impacket dan `ldap3`). |
| **Format Keluaran** | Teks biasa (*plain text*) di konsol. | Teks terwarna di konsol serta ekspor terstruktur JSON dan YAML. |
| **Dukungan Protokol** | SMB, RPC, NetBIOS. | SMB, RPC, NetBIOS, ditambah dukungan kueri LDAP yang lebih baik. |

### 💻 Contoh Penggunaan CLI

```bash
# 1. Enumerasi penuh menggunakan Enum4linux klasik (akses anonim/null session)
enum4linux -a 192.168.1.50

# 2. Enumerasi menggunakan Enum4linux dengan kredensial yang valid
enum4linux -u 'jdoe' -p 'Password123!' -a 192.168.1.50

# 3. Enumerasi modern menggunakan Enum4linux-ng dan mengunduh hasilnya ke format JSON
enum4linux-ng -A 192.168.1.50 -oJ enum_result.json
```

> 💡 **Kesimpulan:**
> Secara akademis, Enum4linux dan Enum4linux-ng membuktikan betapa banyaknya informasi topologi dan identitas internal yang dapat diekstrak dari protokol berbagi berkas jaringan apabila kontrol otentikasi awal tidak diperketat. Keberadaannya di Kali Linux menegaskan pentingnya penerapan penguatan konfigurasi (*hardening*) pada lingkungan Windows dan Samba, meliputi penonaktifan akses anonim (`RestrictAnonymous = 2` pada Windows atau `restrict anonymous = 2` pada Samba), pemblokiran port SMB/NetBIOS (TCP/UDP 135, 137–139, 445) dari jaringan eksternal, serta penerapan prinsip *least privilege* pada setiap direktori berbagi pakai.

---

## 5. Wafw00f (Network Security Appliances)

> **Ringkasan:** Alat otomatis untuk mendeteksi dan mengidentifikasi produk Web Application Firewall (WAF) yang melindungi sebuah situs web. Dengan mengetahui jenis WAF yang aktif, *pentester* dapat merancang strategi yang tepat untuk melewatinya.

### 📄 Deskripsi Perangkat Lunak

Wafw00f adalah sebuah instrumen identifikasi dan *fingerprinting* perangkat keamanan jaringan berbasis *open-source* yang dikembangkan dalam bahasa pemrograman Python. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Discovery** (secara spesifik pada sub-kategori *Network Security Appliances*).

Secara arsitektural, Wafw00f dirancang khusus untuk mendeteksi keberadaan serta mengidentifikasi merek/jenis Web Application Firewall (WAF) yang melindungi sebuah aplikasi web target. Alat ini bekerja dengan cara mengirimkan serangkaian permintaan HTTP yang dirancang secara khusus—mulai dari permintaan normal yang sah hingga muatan serangan simulasi (seperti Cross-Site Scripting, SQL Injection, dan Directory Traversal)—lalu menganalisis karakteristik anomali pada kode status HTTP, HTTP Headers, Cookies, maupun isi halaman respons server untuk mencocokkannya dengan basis data tanda tangan (*signatures*) ratusan produk WAF komersial maupun *open-source* (seperti Cloudflare, AWS WAF, Akamai, Imperva Incapsula, F5 BIG-IP ASM, dan ModSecurity).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pemetaan Pertahanan Perimeter Aplikasi Web (*Web Perimeter Reconnaissance*):** Membantu penguji penetrasi (*penetration tester*) mengetahui apakah infrastruktur aplikasi target berada di balik perlindungan WAF sebelum meluncurkan pemindaian kerentanan otomatis yang agresif, sehingga mencegah pemblokiran alamat IP secara prematur.
* **Perencanaan Strategi Evasi Muatan (*WAF Bypass Strategy Formulation*):** Digunakan oleh praktisi Red Team untuk mengidentifikasi vendor WAF spesifik yang digunakan target, memungkinkan analis memilih teknik pengaburan muatan (*tamper scripts* pada SQLmap atau *encoding* khusus pada XSSer) yang paling efektif untuk melewati aturan penyaringan vendor tersebut.
* **Audit Konfigurasi dan Penyamaran Identitas WAF (*Blue Team Hardening Verification*):** Memungkinkan administrator keamanan dan tim pertahanan memverifikasi apakah perangkat WAF mereka membocorkan informasi merek atau versi secara berlebihan melalui header HTTP bawaan, guna mengevaluasi efektivitas konfigurasi penyembunyian identitas (*security through obscurity* / *header stripping*).

### ⚙️ Fungsi Teknis (Functionality)

* **Analisis Pasif Header dan Cookie HTTP (*Passive HTTP Metadata Inspection*):** Mengirimkan permintaan HTTP standar ke URL target dan memeriksa keberadaan indikator khas vendor pada *response headers* (seperti `Server: cloudflare`, `X-CDN: Imperva`, atau `X-Powered-By-Anquanbao`) serta pola penamaan cookie pelacakan sesi (seperti `__cfduid` atau `visid_incap`).
* **Probing Aktif dengan Muatan Serangan Simulasi (*Active Malicious Payload Probing*):** Apabila inspeksi pasif tidak menemukan identitas WAF secara eksplisit, Wafw00f mengirimkan beberapa permintaan HTTP yang menyisipkan vektor serangan ringan (seperti `?id=1' OR '1'='1` atau `<script>alert(1)</script>`), kemudian membandingkan perbedaan perilaku respons server (misalnya perubahan kode status dari `200 OK` menjadi `403 Forbidden`, `406 Not Acceptable`, atau `501 Not Implemented`) beserta pola halaman blokir (*block page signatures*).
* **Deteksi Generik Anomali WAF (*Generic WAF Detection Heuristic*):** Jika pola respons target tidak cocok dengan tanda tangan vendor mana pun di dalam basis data, Wafw00f menggunakan algoritma perbandingan diferensial antara respons permintaan normal dan permintaan berbahaya untuk menyimpulkan apakah target dilindungi oleh solusi WAF kustom atau tidak dikenal (*Generic Detection*).
* **Pemindaian Multi-Target dan Pendeteksian Menyeluruh (`-a` & `-i`):** Menyediakan opsi `--findall` (`-a`) untuk terus menguji seluruh tanda tangan WAF yang tersedia meskipun satu kecocokan telah ditemukan (berguna untuk mendeteksi arsitektur pertahanan berlapis seperti kombinasi Cloudflare CDN dengan ModSecurity di server origin), serta mendukung pembacaan daftar target massal dari berkas teks (`-i`).
* **Dukungan Proksi, Kustomisasi Header, dan Ekspor Terstruktur (`-p`, `-H`, `-o`):** Mengakomodasi pengarahan lalu lintas pengujian melalui proksi (seperti Burp Suite atau Tor), penyisipan *custom HTTP headers* (misalnya token otentikasi atau User-Agent khusus), serta pengeksporan hasil identifikasi ke dalam format JSON, CSV, atau teks untuk integrasi ke dalam alur kerja pelaporan otomatis.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Mendeteksi WAF pada satu URL target
wafw00f https://example.com

# 2. Menguji seluruh signature WAF yang ada (Find All WAFs)
wafw00f -a https://example.com

# 3. Memindai daftar URL dari berkas teks dan menyimpan hasil ke format JSON
wafw00f -i targets.txt -o output_waf.json
```

> 💡 **Kesimpulan:**
> Secara akademis, Wafw00f membuktikan bahwa setiap mekanisme penyaringan aktif pada lapisan aplikasi (Layer 7) meninggalkan jejak perilaku dan karakteristik protokol yang dapat diidentifikasi secara sistematis oleh pihak eksternal. Keberadaannya di Kali Linux menegaskan bahwa pertahanan perimeter modern tidak boleh bergantung pada penyembunyian identitas perangkat keamanan semata; organisasi wajib memastikan bahwa aturan penyaringan WAF dikonfigurasi secara ketat, diperbarui secara berkala terhadap teknik *obfuscation* terbaru, serta dipadukan dengan perbaikan kerentanan langsung pada level kode sumber aplikasi.

---

## 6. Wireshark & Tcpdump (Network Sniffing)

> **Ringkasan:** Alat penganalisis paket jaringan (*packet analyzer*) yang digunakan untuk menangkap dan menginspeksi lalu lintas data yang lewat di kartu jaringan secara mendalam, membantu menemukan data sensitif yang tidak terenkripsi.

Di dalam taksonomi resmi Kali Linux, baik Wireshark maupun Tcpdump diklasifikasikan secara berdampingan ke dalam kategori **Discovery** (secara spesifik pada sub-kategori *Network Sniffing*). Keduanya beroperasi menggunakan pustaka penangkapan paket tingkat rendah (`libpcap` pada sistem berbasis Unix/Linux) untuk menyadap, merekam, dan membedah (*dissect*) unit data protokol (*Protocol Data Units*) yang melintas di antarmuka jaringan secara *real-time* maupun luring (*offline*) dalam format standar PCAP/PCAPNG.

---

### Part I: Wireshark

#### 📄 Deskripsi Perangkat Lunak

Wireshark adalah sebuah penganalisis protokol jaringan (*network protocol analyzer*) grafis (GUI) berbasis *open-source* yang dikembangkan dalam bahasa C dan C++ (dengan antarmuka berbasis Qt).

Secara arsitektural, Wireshark dirancang untuk melakukan inspeksi paket mendalam (*Deep Packet Inspection* / DPI) di seluruh lapisan model referensi OSI dan TCP/IP—mulai dari frame fisik Lapisan 2 (Ethernet, IEEE 802.11 Wi-Fi) hingga muatan Lapisan 7 (HTTP, DNS, SMB, TLS, Kerberos). Perangkat ini menerjemahkan aliran bit mentah menjadi struktur pohon hierarkis yang mudah dibaca manusia melalui ribuan *protocol dissectors* bawaan.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Forensik Jaringan dan Investigasi Insiden (*Network Forensics & Incident Response*):** Membantu analis Blue Team merekonstruksi kronologi serangan siber dari berkas tangkapan paket (`.pcap`), mengidentifikasi vektor eksfiltrasi data, serta melacak komunikasi Command and Control (C2).
* **Pencegatan Kredensial dan Data Tidak Terenkripsi (*Cleartext Credential Auditing*):** Digunakan dalam pengujian penetrasi untuk membuktikan risiko pengiriman kredensial melalui protokol usang tanpa enkripsi (seperti HTTP, FTP, Telnet, POP3, dan SNMPv1/v2c).
* **Analisis Anomali Protokol dan Kinerja Jaringan:** Berfungsi sebagai instrumen diagnostik untuk mendeteksi serangan tingkat jaringan seperti ARP Spoofing, SYN Flood, pemindaian port agresif, serta kesalahan konfigurasi jabat tangan TLS/SSL.

#### ⚙️ Fungsi Teknis (Functionality)

* **Pembedahan Protokol Multi-Lapisan (*Comprehensive Protocol Dissection*):** Mengurai setiap kolom header paket secara granular (alamat MAC, alamat IP, flags TCP, nomor urut, dan muatan aplikasi) serta mendeteksi ketidaksesuaian standar protokol melalui fitur *Expert Information*.
* **Rekonstruksi Aliran Sesi (*Stream Reassembly* / Follow TCP-UDP-TLS Stream):** Menggabungkan kembali paket-paket yang terfragmentasi menjadi satu percakapan utuh, memungkinkan analis melihat halaman web, berkas yang diunduh, atau sesi perintah terminal persis seperti yang dilihat pengguna.
* **Ekstraksi Objek Berkas Otomatis (*Export Objects*):** Mampu mengekstrak berkas biner, gambar, atau dokumen yang ditransfer melalui protokol seperti HTTP, SMB, FTP, dan TFTP langsung dari rekaman lalu lintas jaringan ke dalam sistem berkas lokal untuk analisis malware lanjutan.
* **Sistem Penyaringan Ekspresif (*Capture & Display Filters*):** Memisahkan filter penangkapan berbasis *Berkeley Packet Filter* (BPF) dengan *Display Filter* yang sangat kaya untuk menyaring jutaan paket secara instan berdasarkan atribut spesifik (misalnya `http.request.method == "POST"` atau `ntlmssp.auth.username`).
* **Dekripsi Lalu Lintas Terenkripsi (*TLS/WPA2 Decryption*):** Mendukung dekripsi lalu lintas HTTPS/TLS menggunakan berkas kunci sesi (`SSLKEYLOGFILE`) maupun dekripsi lalu lintas nirkabel WPA/WPA2 apabila kata sandi jaringan dan paket *4-way handshake* tersedia.

---

### Part II: Tcpdump

#### 📄 Deskripsi Perangkat Lunak

Tcpdump adalah sebuah utilitas penyadap dan penganalisis paket jaringan berbasis baris perintah (*command-line interface* / CLI) yang dikembangkan dalam bahasa pemrograman C.

Secara arsitektural, Tcpdump merupakan instrumen dasar yang sangat ringan, cepat, dan minim ketergantungan (*lightweight CLI packet sniffer*). Karena tidak memerlukan lingkungan grafis (X11/Wayland), Tcpdump menjadi standar industri untuk penangkapan paket jarak jauh melalui sesi SSH pada peladen produksi, perangkat tertanam (*embedded devices*), maupun lingkungan kontainer yang memiliki keterbatasan sumber daya komputasi.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Penangkapan Lalu Lintas Jarak Jauh Tanpa GUI (*Headless Server Packet Capture*):** Memungkinkan penguji penetrasi dan administrator sistem merekam lalu lintas jaringan langsung dari server Linux yang tidak memiliki antarmuka grafis, lalu mengunduh berkas `.pcap` hasilnya untuk dibedah secara visual di Wireshark.
* **Verifikasi Konektivitas dan Validasi Eksploitasi (*Payload & Callback Verification*):** Digunakan oleh praktisi Red Team untuk memantau secara cepat apakah muatan eksploitasi buta (*blind injection*, seperti Blind SSRF, Blind RCE via `ping`/`curl`, atau eksfiltrasi DNS) berhasil memicu koneksi jaringan kembali ke mesin penguji.
* **Pemantauan Berkelanjutan dan Otomasi Skrip (*Continuous Traffic Logging & Ring Buffer*):** Berfungsi sebagai perekam lalu lintas otomatis di latar belakang pada sistem pemantauan keamanan untuk menangkap bukti insiden secara berkala tanpa membebani memori sistem.

#### ⚙️ Fungsi Teknis (Functionality)

* **Penyaringan Tingkat Kernel dengan BPF (*Berkeley Packet Filter*):** Menerjemahkan ekspresi filter logika (seperti `host`, `net`, `port`, `tcp`, `udp`, `icmp`, serta operator `and`, `or`, `not`) menjadi instruksi *bytecode* yang dieksekusi langsung di ruang kernel sistem operasi. Hal ini membuang paket yang tidak relevan sebelum disalin ke ruang pengguna, sehingga mencegah *packet drop* pada lalu lintas berkecepatan tinggi.
* **Perekaman dan Pembacaan Berkas PCAP (`-w` & `-r`):** Menyimpan paket mentah secara utuh tanpa pemotongan panjang muatan (`-s 0`) ke dalam berkas biner standar `.pcap` (`-w`), serta mampu membaca dan memfilter ulang berkas rekaman tersebut secara luring (`-r`).
* **Rotasi Berkas Otomatis (*Ring Buffer Rotation* `-C` & `-W`):** Menyediakan mekanisme manajemen penyimpanan otomatis dengan membagi berkas tangkapan berdasarkan batas ukuran megabita (`-C`) dan membatasi jumlah total berkas berputar (`-W`), mencegah kehabisan ruang cakram saat pemantauan jangka panjang.
* **Inspeksi Heksadesimal dan ASCII Secara Langsung (`-X` & `-A`):** Menampilkan isi muatan paket (*payload*) dalam representasi karakter ASCII (`-A`) atau kombinasi heksadesimal dan ASCII (`-X`) langsung di terminal, sangat berguna untuk memeriksa permintaan HTTP mentah atau kueri DNS secara instan.
* **Kontrol Resolusi Nama dan Tingkat Detail (`-n`, `-nn`, `-v`):** Menyediakan opsi untuk menonaktifkan resolusi DNS dan nama port (`-nn`) guna mempercepat pemrosesan sekaligus mencegah kebocoran paket DNS tambahan saat melakukan pengintaian pasif, serta pengaturan tingkat kerincian keluaran (`-v`, `-vv`, `-vvv`).

---

### 📊 Perbandingan Singkat & Kesimpulan Akademis: Wireshark vs Tcpdump

| Aspek Komparasi | Wireshark | Tcpdump |
| --- | --- | --- |
| **Antarmuka Pengguna** | Grafis (GUI berbasis Qt) dengan visualisasi hierarkis. | Baris perintah (CLI) murni di terminal. |
| **Konsumsi Sumber Daya** | Tinggi (memuat seluruh konteks aliran di memori). | Sangat rendah dan optimal untuk peladen *headless*. |
| **Kekuatan Utama** | Rekonstruksi aliran (*Follow Stream*), ekstraksi berkas, dan analisis protokol mendalam. | Penangkapan paket berkecepatan tinggi di server jarak jauh dengan filter BPF. |

### 💻 Contoh Penggunaan CLI

```bash
# 1. Tcpdump: Menangkap paket TCP port 80/445 pada eth0 dan menyimpan ke file pcap
sudo tcpdump -i eth0 -nn -s 0 -w capture.pcap tcp port 80 or port 445

# 2. Tcpdump: Membaca file pcap dan menampilkan payload ASCII di terminal
tcpdump -A -r capture.pcap 'tcp port 80 and (((ip[20:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# 3. Wireshark: Membuka file pcap hasil tangkapan Tcpdump untuk analisis visual GUI
wireshark capture.pcap &
```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Wireshark dan Tcpdump merupakan dua sisi mata uang yang saling melengkapi dalam disiplin analisis jaringan dan keamanan siber: Tcpdump unggul sebagai instrumen akuisisi data mentah yang gesit di titik pengamatan (*network tap*), sementara Wireshark unggul sebagai laboratorium bedah forensik dan rekonstruksi protokol. Keberadaan keduanya di Kali Linux menegaskan prinsip fundamental keamanan komunikasi data bahwa setiap paket yang melintas di jaringan tanpa enkripsi bersifat transparan bagi siapa pun yang memiliki akses ke media transmisi—sehingga mewajibkan penerapan protokol terenkripsi (*end-to-end encryption* seperti TLS 1.3, SSH, IPsec, dan DNS-over-HTTPS) serta segmentasi jaringan yang ketat untuk melindungi kerahasiaan informasi.
