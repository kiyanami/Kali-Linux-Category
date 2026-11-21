# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Credential Access
# 🛠️ Credential Access Tools - Kali Linux 

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak pengaksesan dan pencurian kredensial (*Credential Access*) yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [Mimikatz (OS Credential Dumping)](#1-mimikatz-os-credential-dumping)
2. [Hashcat (Password Cracking)](#2-hashcat-password-cracking)
3. [John the Ripper - john / johnny (Password Cracking)](#3-john-the-ripper---john--johnny-password-cracking)
4. [Hydra - hydra / hydra-gtk (Brute Force)](#4-hydra---hydra--hydra-gtk-brute-force)
5. [Crunch (Password Profiling & Wordlists)](#5-crunch-password-profiling--wordlists)
6. [Trufflehog & Gitxray (Unsecured Credentials)](#6-trufflehog--gitxray-unsecured-credentials)
   * [Part I: Trufflehog](#part-i-trufflehog)
   * [Part II: Gitxray](#part-ii-gitxray)
   * [Tabel Perbandingan Akademis](#tabel-perbandingan-akademis-trufflehog-vs-gitxray)

---

## 1. Mimikatz (OS Credential Dumping)

> **Ringkasan:** Alat pasca-eksploitasi legendaris untuk lingkungan Windows. Mimikatz digunakan untuk mengekstrak kata sandi teks biasa, nilai hash, PIN, serta tiket Kerberos langsung dari memori proses LSASS (*Local Security Authority Subsystem Service*).

### 📄 Deskripsi Perangkat Lunak

Mimikatz adalah sebuah instrumen *open-source* pasca-eksploitasi (*post-exploitation tool*) yang dikembangkan dalam bahasa pemrograman C untuk mengeksplorasi mekanisme otentikasi dan keamanan memori pada sistem operasi Microsoft Windows. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Credential Access** (secara spesifik pada sub-kategori *OS Credential Dumping*), serta lintas-kategori pada **Defense Evasion** dan **Lateral Movement** (sub-kategori *Pass-the-Hash*).

Secara arsitektural, Mimikatz dirancang untuk berinteraksi langsung dengan proses inti sistem keamanan Windows, terutama LSASS (*Local Security Authority Subsystem Service* / `lsass.exe`). Ketika pengguna melakukan logon ke sistem Windows, LSASS menyimpan materi kredensial di dalam ruang memori RAM untuk memfasilitasi skema *Single Sign-On* (SSO) jaringan. Dengan memanfaatkan hak akses administratif dan privilese sistem (`SeDebugPrivilege`), Mimikatz mampu membedah struktur memori proses LSASS secara *in-memory* maupun melalui berkas *memory dump* (`lsass.dmp`) secara *offline* di Kali Linux untuk mengekstrak kredensial yang tersimpan.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Ekstraksi Kredensial Sistem Operasi (*OS Credential Dumping*):** Membantu penguji penetrasi (*penetration tester*) memanen kata sandi teks biasa (*plaintext passwords*), hash NTLM, kode PIN, serta tiket otentikasi Kerberos dari komputer yang telah terkompromi guna mengukur risiko eskalasi kompromi.
* **Simulasi Pergerakan Lateral dan Dominasi Domain (*Lateral Movement & AD Takeover*):** Digunakan oleh praktisi Red Team untuk mendemonstrasikan bagaimana pencurian materi otentikasi dari satu stasiun kerja (*workstation*) dapat dimanfaatkan untuk berpindah ke server lain (melalui teknik *Pass-the-Hash* atau *Pass-the-Ticket*) hingga mengambil alih kendali penuh atas Domain Controller Active Directory.
* **Pengujian Arsitektur Pertahanan Memori Windows (*Blue Team & EDR Benchmarking*):** Memungkinkan analis pertahanan menguji efektivitas kontrol keamanan modern Microsoft—seperti Credential Guard (berbasis virtualisasi perangkat keras/VBS), *Protected Process Light* (PPL) pada LSASS, serta aturan deteksi perilaku *Endpoint Detection and Response* (EDR)—dalam mencegah pembacaan memori proses keamanan.

### ⚙️ Fungsi Teknis (Functionality)

* **Ekstraksi Memori LSASS (`sekurlsa`):** Merupakan modul inti yang berfungsi membaca ruang memori proses `lsass.exe` (baik secara langsung pada host Windows maupun secara luring dari berkas minidump di Kali Linux) untuk mengekstrak hash NTLM, kredensial WDigest (teks biasa pada konfigurasi lawas), serta tiket Kerberos aktif (`sekurlsa::logonpasswords`).
* **Pemalsuan dan Manipulasi Tiket Kerberos (`kerberos`):** Mendukung pembuatan tiket otentikasi palsu secara kriptografis, meliputi *Golden Ticket* (pemalsuan *Ticket Granting Ticket* / TGT menggunakan hash akun `krbtgt` untuk akses persisten penuh di seluruh domain) serta *Silver Ticket* (pemalsuan *Ticket Granting Service* / TGS untuk mengakses layanan spesifik seperti CIFS/SMB atau HTTP).
* **Injeksi Kredensial Tanpa Kata Sandi Asli (*Pass-the-Hash / Overpass-the-Hash*):** Mengizinkan penguji menyuntikkan hash NTLM langsung ke dalam sesi logon baru (`sekurlsa::pth`) untuk mengautentikasi diri ke layanan jaringan jarak jauh tanpa perlu memecahkan (*cracking*) nilai hash kembali menjadi kata sandi teks biasa.
* **Ekstraksi Basis Data Kredensial Lokal dan Replikasi (`lsadump`):** Mampu membaca berkas registri lokal (SAM dan SYSTEM) secara luring untuk mengambil hash akun lokal, serta mengeksekusi teknik DCSync (`lsadump::dcsync`)—yaitu menyamar sebagai Domain Controller sah untuk meminta replikasi hash kata sandi akun domain manapun melalui protokol MS-DRSR tanpa perlu mengeksekusi kode di dalam server Domain Controller secara langsung.
* **Ekspor Sertifikat dan Kunci Kriptografi (`crypto`):** Berfungsi membongkar perlindungan API kriptografi Windows (CryptoAPI / CNG) untuk mengekspor kunci privat (*private keys*) sertifikat digital yang sebelumnya ditandai sebagai *non-exportable* oleh sistem operasi.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan Mimikatz pada target Windows (meminta hak akses debug & mengekstrak password)
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit

# 2. Menganalisis berkas LSASS dump offline di Kali Linux menggunakan Pypykatz (implementasi Python Mimikatz)
pypykatz lsa sub-dump minidump lsass.dmp

# 3. Melakukan serangan DCSync untuk mengekstrak hash pengguna spesifik dari Domain Controller
mimikatz.exe "privilege::debug" "lsadump::dcsync /domain:corp.local /user:Administrator" exit
```

> 💡 **Kesimpulan:**
> Secara akademis, Mimikatz merupakan instrumen paling berpengaruh dalam sejarah keamanan ekosistem Windows karena secara empiris membuktikan kelemahan mendasar pada penyimpanan materi otentikasi di dalam memori ruang pengguna (*user-mode memory*). Keberadaannya di Kali Linux—yang menyediakan berkas biner Mimikatz untuk ditransfer maupun dianalisis secara luring—menegaskan urgensi penerapan pertahanan berlapis pada arsitektur identitas modern, meliputi aktivasi Windows Defender Credential Guard, perlindungan RunAsPPL untuk proses LSASS, pembatasan ketat hak administratif lokal, serta penghapusan protokol otentikasi warisan seperti WDigest dan NTLMv1.

---

## 2. Hashcat (Password Cracking)

> **Ringkasan:** Mesin pemecah kata sandi (*password cracker*) berbasis pemrosesan grafis (GPU) tercepat di dunia. Alat ini digunakan untuk membongkar berkas hash (seperti MD5, SHA, bcrypt, NT, dll.) menggunakan berbagai metode seperti *brute-force*, *dictionary attack*, dan *rule-based attack*.

### 📄 Deskripsi Perangkat Lunak

Hashcat adalah sebuah instrumen pemulihan kata sandi (*password recovery utility*) *open-source* berkecepatan tinggi yang dirancang untuk memecahkan nilai hash kriptografis secara luring (*offline password cracking*). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Credential Access** (secara spesifik pada sub-kategori *Password Cracking*).

Secara arsitektural, Hashcat memelopori pemanfaatan komputasi heterogen paralel masif melalui GPGPU (*General-Purpose Computing on Graphics Processing Units*) menggunakan antarmuka seperti OpenCL, CUDA (NVIDIA), dan HIP (AMD), di samping dukungan terhadap CPU dan akselerator perangkat keras lainnya. Desain ini memindahkan beban kalkulasi fungsi hash satu arah (*one-way cryptographic hash functions*) dari arsitektur prosesor serial menuju ribuan inti pemrosesan paralel GPU, menjadikannya salah satu mesin kriptanalisis tercepat di dunia.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pemulihan Kredensial Pasca-Eksfiltrasi (*Offline Credential Cracking*):** Membantu penguji penetrasi (*penetration tester*) mengonversi nilai hash kata sandi yang diperoleh dari kebocoran basis data, memori proses LSASS (seperti hash NTLM), jabat tangan nirkabel (WPA/WPA2 PMKID/EAPOL), atau tiket Kerberos (Kerberoasting) kembali menjadi teks biasa (*plaintext*).
* **Audit Kebijakan Kata Sandi Korporat (*Password Policy & Strength Auditing*):** Digunakan oleh administrator keamanan dan tim pertahanan (Blue Team) untuk menguji ketahanan basis data kredensial pengguna internal terhadap serangan kamus modern, guna mengidentifikasi kata sandi yang lemah atau mudah ditebak sebelum dieksploitasi oleh aktor ancaman.
* **Forensik Digital dan Pemulihan Akses (*Digital Forensics & Data Recovery*):** Berfungsi sebagai instrumen investigasi untuk membuka proteksi kata sandi pada arsip terenkripsi (seperti ZIP, RAR, 7-Zip), dokumen perkantoran (PDF, Microsoft Office), maupun kontainer enkripsi cakram (seperti BitLocker, VeraCrypt, dan LUKS).

### ⚙️ Fungsi Teknis (Functionality)

* **Dukungan Ratusan Algoritma Hash dan Enkripsi (`-m`):** Menyediakan modul pemrosesan untuk lebih dari 350 jenis algoritma kriptografi dan protokol otentikasi—mulai dari fungsi hash mentah yang cepat (MD5, SHA-1, SHA-256, NTLM), hash kata sandi sistem operasi modern yang diberi garam (*salted hashes* seperti bcrypt, sha512crypt, Argon2), hingga struktur jabat tangan jaringan (NetNTLMv2, Kerberos 5 TGS-REP, dan WPA-PBKDF2-PMKID+EAPOL).
* **Multi-Mode Serangan Kriptanalisis (`-a`):** Mendukung beragam metodologi pencarian ruang kunci (*keyspace search*), meliputi:
  * **Straight / Dictionary Attack (`-a 0`):** Mencocokkan kandidat kata sandi secara langsung dari daftar kata (*wordlist*).
  * **Combinator Attack (`-a 1`):** Menggabungkan setiap kata dari dua kamus berbeda secara berpasangan.
  * **Brute-Force & Mask Attack (`-a 3`):** Menguji seluruh kombinasi karakter berdasarkan pola struktural spesifik (misalnya `?u?l?l?l?d?d?d?d` untuk pola 1 huruf kapital, 3 huruf kecil, dan 4 digit angka), yang secara drastis mempersempit ruang pencarian dibandingkan *brute-force* murni.
  * **Hybrid Attack (`-a 6` & `-a 7`):** Memadukan daftar kata kamus dengan penambahan atau awalan masker karakter secara dinamis.
* **Mesin Mutasi Berbasis Aturan (*Rule-Based Attack Engine* `-r`):** Mengintegrasikan bahasa pemrograman aturan internal yang memodifikasi setiap entri kamus secara *real-time* di dalam memori GPU (seperti mengubah huruf menjadi kapital, menyisipkan angka tahun di akhir kata, atau substitusi karakter leetspeak seperti `a -> @` dan `e -> 3`) guna meniru pola kebiasaan manusia dalam membuat kata sandi.
* **Manajemen Sesi Terdistribusi dan Pemulihan (*Session Checkpointing & Distributed Cracking*):** Dilengkapi fitur penyimpanan status pemrosesan (*restore points*) yang memungkinkan operasi pemecahan hash berdurasi panjang dijeda dan dilanjutkan kapan saja, serta mendukung koordinasi klaster multi-GPU dalam satu maupun banyak simpul komputasi (*nodes*).
* **Optimasi Termal dan Perangkat Keras (*Hardware Monitoring & Kernel Optimization*):** Menyediakan pemantauan suhu dan kecepatan kipas GPU secara *real-time* untuk mencegah kerusakan perangkat keras akibat beban komputasi penuh, serta menyertakan opsi kernel teroptimasi (`-O`) untuk memaksimalkan *throughput* kalkulasi hash per detik (H/s).

### 💻 Contoh Penggunaan CLI

```bash
# 1. Serangan Kamus (Dictionary Attack) terhadap Hash NTLM (Mode 1000)
hashcat -m 1000 -a 0 hashes_ntlm.txt /usr/share/wordlists/rockyou.txt

# 2. Serangan Masker (Mask Attack) untuk pola 1 Huruf Besar, 3 Huruf Kecil, 4 Angka (misal: Pass2026)
hashcat -m 0 -a 3 md5_hashes.txt ?u?l?l?l?d?d?d?d

# 3. Serangan Kamus berbasis Aturan (Rule-Based Attack) menggunakan aturan Best64
hashcat -m 1000 -a 0 hashes_ntlm.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

> 💡 **Kesimpulan:**
> Secara akademis, Hashcat mendemonstrasikan bagaimana kemajuan arsitektur komputasi paralel perangkat keras (GPGPU) dapat meruntuhkan asumsi keamanan fungsi hash kriptografi yang dirancang terlalu cepat (seperti MD5 atau NTLM). Keberadaannya di Kali Linux menegaskan bahwa pertahanan kredensial modern tidak cukup hanya mengandalkan pengacakan satu arah; organisasi wajib menerapkan algoritma derivasi kunci yang lambat secara komputasi dan boros memori (*memory-hard key derivation functions* seperti Argon2id, bcrypt, atau PBKDF2 dengan iterasi tinggi), disertai kebijakan panjang kata sandi (*passphrase*) yang memadai serta otentikasi multi-faktor (MFA).

---

## 3. John the Ripper - john / johnny (Password Cracking)

> **Ringkasan:** Alat pemecah kata sandi klasik berbasis CPU. John sangat fleksibel karena mampu mendeteksi jenis enkripsi hash secara otomatis dan memecahkan enkripsi kata sandi pada dokumen terproteksi (seperti ZIP, RAR, PDF, dan berkas kunci privat SSH).

### 📄 Deskripsi Perangkat Lunak

John the Ripper (JtR) (`john`) adalah sebuah instrumen audit kekuatan kata sandi dan pemulihan kredensial (*password cracking utility*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Credential Access** (secara spesifik pada sub-kategori *Password Cracking*) bersama dengan Johnny (`johnny`), yang merupakan lapisan antarmuka pengguna grafis (*Graphical User Interface* / GUI) lintas platform berbasis Qt untuk mengoperasikan *engine* JtR secara visual.

Secara arsitektural, John the Ripper (khususnya versi komunitas Jumbo yang terpasang di Kali Linux) dirancang dengan fleksibilitas ekstraksi dan pengenalan format otomatis (*auto-detection*). Berbeda dengan Hashcat yang sangat berorientasi pada akselerasi perangkat keras GPU dengan kode format numerik spesifik, John the Ripper menonjol dalam kompatibilitas pemrosesan berbasis CPU (memanfaatkan instruksi vektor SIMD seperti AVX2/AVX-512), dukungan puluhan utilitas konversi berkas (`*2john`), serta kemampuan membedah struktur teks mentah (seperti berkas `/etc/shadow` Linux) secara langsung tanpa memerlukan pra-pemrosesan manual.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pemulihan Kredensial Sistem dan Aplikasi (*Offline Credential Cracking*):** Membantu penguji penetrasi (*penetration tester*) mengonversi hash kata sandi yang diekstrak dari sistem operasi (Linux `/etc/shadow`, Windows SAM/NTLM, macOS), basis data, maupun aplikasi web kembali menjadi teks biasa (*plaintext*) guna membuktikan risiko kebocoran akses.
* **Audit Kepatuhan Kebijakan Kata Sandi (*Password Policy Compliance Auditing*):** Digunakan oleh administrator sistem dan tim pertahanan (Blue Team) untuk memindai akun-akun pengguna di lingkungan korporat yang masih menggunakan kata sandi lemah, *default*, atau mudah ditebak secara berkala.
* **Forensik Digital dan Pemulihan Berkas Terkunci (*File & Archive Forensics*):** Berfungsi sebagai instrumen investigasi untuk membuka proteksi kata sandi pada berbagai format berkas terenkripsi—seperti kunci privat SSH (`id_rsa`), arsip terkompresi (ZIP, RAR, 7z), dokumen perkantoran (PDF, Office), hingga dompet kripto—melalui rangkaian utilitas konverter bawaannya.

### ⚙️ Fungsi Teknis (Functionality)

* **Pengenalan Format Hash Otomatis & Ekosistem Konverter (`*2john`):** Mampu mendeteksi jenis algoritma hash secara otomatis berdasarkan struktur string input. Selain itu, JtR menyertakan puluhan skrip konverter khusus (seperti `unshadow`, `zip2john`, `rar2john`, `ssh2john`, `pdf2john`, dan `keepass2john`) yang mengekstrak struktur kriptografis dari berkas biner terenkripsi menjadi format teks yang siap diproses oleh mesin pemecah hash.
* **Multi-Mode Serangan Kriptanalisis (*Multi-Mode Cracking Engine*):** Mengintegrasikan empat mode pemrosesan berurutan untuk memaksimalkan tingkat keberhasilan pemulihan kata sandi:
  * **Single Crack Mode (`--single`):** Mode tercepat yang memanfaatkan informasi kontekstual dari baris kredensial itu sendiri (seperti nama pengguna, nama lengkap di kolom GECOS, atau direktori home) dan memutasikannya untuk menebak kata sandi pribadi pengguna.
  * **Wordlist Mode (`--wordlist`):** Mencocokkan kandidat kata sandi dari kamus kata eksternal (seperti `rockyou.txt` atau SecLists).
  * **Incremental Mode (`--incremental`):** Mode *brute-force* cerdas berbasis analisis frekuensi karakter Markov (*Markov chain character frequency*), yang menguji kombinasi karakter berdasarkan probabilitas kemunculan huruf/angka dalam bahasa manusia terlebih dahulu sebelum mencoba kombinasi acak yang tidak lazim.
  * **Mask Mode (`--mask`):** Mengizinkan definisi pola karakter eksplisit untuk mempersempit ruang pencarian kunci (*keyspace*).
* **Mesin Mutasi Berbasis Aturan (*Rule-Based Mangling Engine*):** Dilengkapi dengan mesin aturan dinamis (`--rules`) yang memodifikasi setiap kata di dalam kamus sebelum di-hash (misalnya menambahkan angka di akhir, membalik urutan huruf, atau kombinasi kapitalisasi) untuk meniru pola modifikasi kata sandi pengguna.
* **Manajemen Sesi dan Perekaman Progres (`john.pot` & `--restore`):** Secara otomatis menyimpan seluruh hash yang berhasil dipecahkan beserta kata sandi aslinya ke dalam berkas basis data lokal (`~/.john/john.pot`), serta mendukung penamaan sesi kerja (`--session`) agar proses pemecahan berdurasi panjang dapat dihentikan dan dilanjutkan kembali tanpa kehilangan progres.
* **Visualisasi dan Manajemen Audit via GUI (`johnny`):** Menyediakan antarmuka grafis melalui perangkat lunak Johnny, yang mempermudah analis memvisualisasikan progres persentase pemecahan hash, mengelompokkan akun yang telah terbuka, mengekspor hasil audit ke dalam format CSV, serta mengatur parameter aturan dan kamus tanpa perlu menghafal sintaks baris perintah yang kompleks.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menggabungkan berkas /etc/passwd dan /etc/shadow Linux lalu memecahkannya
unshadow /etc/passwd /etc/shadow > linux_hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt linux_hashes.txt

# 2. Mengekstrak hash dari berkas ZIP terproteksi kata sandi dan memecahkannya
zip2john rahasia.zip > zip_hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt

# 3. Menampilkan daftar hasil kata sandi yang berhasil dipecahkan (potfile)
john --show zip_hash.txt
```

> 💡 **Kesimpulan:**
> Secara akademis, John the Ripper dan Johnny merepresentasikan pendekatan kriptanalisis luring yang komprehensif karena menggabungkan kecerdasan heuristik berbasis konteks pengguna (*Single Crack* dan *Markov Incremental*) dengan dukungan format artefak yang sangat luas. Keberadaannya di Kali Linux menegaskan bahwa keamanan kredensial menuntut penerapan fungsi derivasi kunci yang lambat secara komputasi (*slow hashing* seperti Argon2id atau bcrypt), pemisahan hak akses berkas penyimpan hash (`/etc/shadow`), serta edukasi pengguna untuk menghindari penggunaan informasi pribadi sebagai dasar pembuatan kata sandi.

---

## 4. Hydra - hydra / hydra-gtk (Brute Force)

> **Ringkasan:** Alat otomatisasi serangan tebakan kata sandi daring (*online parallelized login cracker*). Hydra mendukung puluhan protokol jaringan populer (seperti SSH, FTP, Telnet, HTTP-FORM, SMB, RDP, dll.) untuk menguji kekuatan autentikasi sebuah layanan secara massal.

### 📄 Deskripsi Perangkat Lunak

THC-Hydra (`hydra`) adalah sebuah instrumen pengujian autentikasi jaringan daring (*online password cracking / login cracker*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Credential Access** (secara spesifik pada sub-kategori *Brute Force*) bersama dengan Hydra-GTK (`hydra-gtk`), yang merupakan antarmuka pengguna grafis (*Graphical User Interface* / GUI) berbasis GTK+ untuk mengonfigurasi dan mengeksekusi perintah Hydra secara visual.

Secara arsitektural, berbeda dengan pemecah kata sandi luring (*offline crackers* seperti Hashcat atau John the Ripper) yang mengkalkulasi hash secara lokal, Hydra beroperasi secara langsung terhadap layanan jaringan yang aktif (*live network services*). Alat ini menggunakan arsitektur pemrosesan paralel berbasis *multi-threading* (mendukung puluhan koneksi simultan secara asinkron) untuk mengotomatisasi percobaan autentikasi lintas lebih dari 50 protokol jaringan dan aplikasi web secara cepat.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Audit Kekuatan Autentikasi Layanan Jaringan (*Online Authentication Auditing*):** Membantu penguji penetrasi (*penetration tester*) mengidentifikasi layanan jarak jauh—seperti SSH, FTP, Telnet, RDP, SMB, dan basis data—yang masih menggunakan kredensial bawaan pabrik (*default credentials*) atau kata sandi lemah.
* **Simulasi Serangan Dictionary dan Password Spraying (*Red Teaming*):** Digunakan untuk memvalidasi risiko akses tidak sah melalui teknik *dictionary attack* (menguji banyak kata sandi pada satu akun) maupun *password spraying* (menguji satu kata sandi umum pada banyak akun pengguna untuk menghindari pemblokiran akun).
* **Pengujian Mekanisme Pertahanan Intrusi dan Rate Limiting (*Blue Team Benchmarking*):** Memungkinkan tim pertahanan mengevaluasi efektivitas kontrol keamanan perimeter—seperti Fail2Ban, kebijakan penguncian akun (*Account Lockout Policy*), sistem deteksi intrusi (IDS/IPS), serta aturan *Web Application Firewall* (WAF)—dalam mendeteksi dan memblokir lonjakan percobaan login gagal secara otomatis.

### ⚙️ Fungsi Teknis (Functionality)

* **Dukungan Multi-Protokol Jaringan dan Basis Data:** Mengintegrasikan modul klien bawaan untuk berinteraksi dengan puluhan protokol lapisan aplikasi dan transpor tanpa memerlukan perangkat lunak eksternal, meliputi SSH, FTP/FTPS, Telnet, SMB, RDP, VNC, POP3, IMAP, SMTP, SNMP, Cisco AAA, serta basis data relasional (MySQL, PostgreSQL, MSSQL, Oracle).
* **Modul Eksploitasi Autentikasi Web Dinamis (`http-get-form` / `http-post-form`):** Mampu melakukan *brute-force* terhadap formulir login berbasis web modern dengan memarsing parameter permintaan HTTP (POST body atau GET query), menyematkan nilai cookie sesi, serta mendeteksi keberhasilan atau kegagalan otentikasi berdasarkan pencocokan string respons server (*failure/success condition strings*).
* **Manajemen Konkurensi dan Pengendalian Kecepatan (`-t` & `-W`):** Menyediakan pengaturan granular terhadap jumlah utas koneksi paralel (*tasks/threads*) per target maupun keseluruhan, serta jeda waktu antar-percobaan (*wait time*), memungkinkan analis menyeimbangkan kecepatan pemindaian agar tidak menyebabkan *Denial of Service* (DoS) pada layanan target atau memicu alarm IDS.
* **Fleksibilitas Komposisi Kredensial (`-l`/`-L`, `-p`/`-P`, `-C`, `-e`):** Mendukung pengujian berbasis pasangan tunggal, daftar kamus terpisah untuk nama pengguna dan kata sandi, berkas kombinasi berformat `user:pass` (`-C`), serta pengujian kombinasi otomatis (`-e nsr`) yang secara otomatis mencoba kata sandi kosong (*null*), kata sandi yang sama dengan nama pengguna, dan kebalikan nama pengguna.
* **Pemulihan Sesi dan Visualisasi Terstruktur (`hydra.restore` & `hydra-gtk`):** Secara otomatis menyimpan status kemajuan serangan ke dalam berkas sesi apabila proses terhenti secara tiba-tiba, sementara Hydra-GTK memfasilitasi pemetaan opsi protokol, pengaturan port non-standar, penyisipan jalur proksi (HTTP/SOCKS), dan pelaporan hasil kredensial valid dalam satu antarmuka grafis terpadu.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Dictionary Attack terhadap layanan SSH menggunakan user tunggal 'admin'
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.1.50 ssh

# 2. Password Spraying terhadap protokol SMB (Windows Share) menggunakan daftar user
hydra -L users.txt -p Password2026! 192.168.1.100 smb

# 3. Brute-force formulir Web Login (HTTP POST Form)
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.1.50 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Login failed"
```

> 💡 **Kesimpulan:**
> Secara akademis, Hydra dan Hydra-GTK merepresentasikan tolok ukur pengujian ketahanan antarmuka otentikasi jaringan terhadap ancaman eksploitasi kredensial secara daring. Keberadaannya di Kali Linux menegaskan bahwa keamanan mekanisme login publik tidak cukup hanya mengandalkan kompleksitas kata sandi pengguna; organisasi wajib menerapkan pertahanan berlapis pada sisi server, meliputi Otentikasi Multi-Faktor (MFA), pembatasan laju permintaan (*rate limiting*), pemblokiran IP otomatis berbasis anomali log (seperti Fail2Ban), serta penghapusan autentikasi berbasis kata sandi demi penggunaan kunci kriptografis (seperti *SSH Key-Based Authentication*).

---

## 5. Crunch (Password Profiling & Wordlists)

> **Ringkasan:** Generator daftar kata (*wordlist generator*) yang memungkinkan *pentester* membuat daftar kombinasi karakter kustom berdasarkan pola, panjang karakter, atau simbol tertentu untuk digunakan dalam serangan pengujian kata sandi lanjutan.

### 📄 Deskripsi Perangkat Lunak

Crunch adalah sebuah utilitas generator daftar kata (*wordlist generator*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C untuk menghasilkan seluruh kemungkinan permutasi dan kombinasi karakter secara deterministik. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Credential Access** (secara spesifik pada sub-kategori *Password Profiling & Wordlists*).

Secara arsitektural, Crunch beroperasi menggunakan algoritma kombinatorial berbasis parameter panjang minimum (*min-length*), panjang maksimum (*max-length*), dan himpunan karakter (*character set* / *charset*) tertentu. Berbeda dengan kamus statis bawaan (seperti RockYou atau SecLists) yang bersumber dari kebocoran data historis, Crunch menciptakan ruang kunci (*keyspace*) secara dinamis dan menyeluruh (*exhaustive generation*), sehingga sangat krusial ketika target menggunakan pola kata sandi yang terstruktur namun tidak terdapat di dalam kamus publik standar.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber

* **Pembuatan Kamus Kata Sandi Tertarget (*Targeted Password Profiling*):** Membantu penguji penetrasi (*penetration tester*) membangun *wordlist* kustom berdasarkan informasi hasil pengintaian (OSINT), seperti nama perusahaan, tahun berdiri, kode wilayah, atau pola standar kata sandi bawaan router/perangkat IoT.
* **Integrasi Aliran Data On-the-Fly (*Piping to Password Crackers*):** Digunakan sebagai pemasok kandidat kata sandi secara langsung (*real-time standard output piping*) ke mesin pemecah hash maupun alat *brute-force* daring (seperti Hashcat, John the Ripper, Aircrack-ng, atau Hydra) tanpa harus menyimpan berkas kamus berukuran gigabita hingga terabita ke dalam cakram keras.
* **Analisis Kompleksitas Ruang Kunci (*Keyspace & Entropy Modeling*):** Memungkinkan peneliti keamanan dan tim pertahanan (Blue Team) menghitung secara empiris ukuran kombinasi (*keyspace size*) serta estimasi waktu yang dibutuhkan untuk menembus kebijakan panjang dan komposisi karakter kata sandi tertentu.

### ⚙️ Fungsi Teknis (Functionality)

* **Generasi Kombinatorial Berbasis Charset Kustom & Bawaan (`-f`):** Mengizinkan pengguna mendefinisikan himpunan karakter secara eksplisit melalui baris perintah maupun merujuk pada berkas definisi standar Kali Linux (`/usr/share/crunch/charset.lst`), seperti `lalpha` (huruf kecil), `ualpha` (huruf kapital), `numeric` (angka), `mixalpha-numeric-all-space` (seluruh karakter ASCII), hingga simbol Unicode.
* **Pembentukan Berbasis Pola Struktural (Pattern-Based Generation `-t`):** Menyediakan fitur masker pola menggunakan karakter pengganti (*placeholders*) spesifik—yaitu `@` untuk huruf kecil, `,` untuk huruf kapital, `%` untuk angka, dan `^` untuk simbol—yang memungkinkan analis mengunci posisi karakter tertentu (misalnya `Admin@2026%%`) guna memangkas ruang pencarian secara drastis.
* **Permutasi Kata dan Blok Karakter (`-p`):** Mendukung pengacakan urutan (*permutation*) dari sekumpulan kata atau token spesifik tanpa pengulangan, sangat efektif untuk mengombinasikan unsur-unsur identitas pribadi target (seperti nama depan, tanggal lahir, dan nama hewan peliharaan).
* **Manajemen Penyimpanan dan Pemecahan Berkas (Output Splitting & Compression `-b`, `-c`, `-z`):** Dilengkapi fitur untuk membagi keluaran daftar kata berukuran masif ke dalam beberapa berkas berdasarkan batas ukuran byte (`-b`, misalnya per 500 MB) atau jumlah baris (`-c`), serta mengompresi hasil secara langsung menggunakan format gzip, bzip2, lzma, atau 7z (`-z`) demi menghemat kapasitas penyimpanan.
* **Pengendalian Duplikasi dan Pemulihan Sesi (`-d` & `-r`):** Menyediakan filter pembatas karakter berulang secara berurutan (`-d`, misalnya mencegah kombinasi tidak realistis seperti `aaaa`) untuk mengurangi *noise* pengujian, serta fitur pemulihan proses (`-r`) apabila pembuatan *wordlist* berskala besar sempat dihentikan.

### 💻 Contoh Penggunaan CLI

```bash
# 1. Menghasilkan wordlist 8 karakter dengan pola tertentu (misal: Admin2026 -> Admin%%%%)
crunch 9 9 -t Admin2026 -o output_wordlist.txt

# 2. Permutasi kata spesifik tanpa pengulangan
crunch 0 0 -p Budi Rahasia 2026

# 3. Direct Piping: Menghasilkan kombinasi angka 8 digit langsung ke Hashcat tanpa simpan file
crunch 8 8 1234567890 | hashcat -m 0 hash.txt
```

> 💡 **Kesimpulan:**
> Secara akademis, Crunch merepresentasikan implementasi praktis dari teori kombinatorika dalam domain keamanan kredensial, yang mendemonstrasikan bagaimana ruang pencarian (*keyspace*) meningkat secara eksponensial seiring penambahan panjang karakter dan variasi tipe simbol ($N^{L}$). Keberadaannya di Kali Linux menegaskan bahwa pertahanan terhadap serangan berbasis tebakan terstruktur menuntut penerapan kebijakan panjang kata sandi minimal yang memadai, penghindaran pola kata sandi yang dapat diprediksi secara organisasi, serta pembatasan percobaan autentikasi di tingkat sistem.

---

## 6. Trufflehog & Gitxray (Unsecured Credentials)

> **Ringkasan:** Alat pemindai otomatis yang berfungsi mencari kunci rahasia (*secret keys*), token API, dan kredensial sensitif lainnya yang tidak sengaja tertinggal atau berkomitmen di dalam repositori kode publik (seperti GitHub).

---

### Part I: Trufflehog

#### 📄 Deskripsi Perangkat Lunak
Trufflehog (ditulis ulang secara menyeluruh dalam bahasa Go pada versi modernnya) adalah utilitas keamanan ofensif-defensif untuk menemukan, memverifikasi, dan menganalisis kredensial atau *secrets* yang tidak sengaja terkomit ke dalam riwayat *version control* maupun berbagai sumber data lainnya. Dalam konteks keamanan perangkat lunak dan *supply chain security*, kebocoran kredensial pada repositori sering kali terjadi meskipun berkas telah dihapus pada *commit* berikutnya, karena riwayat Git tetap menyimpan *snapshot* objek tersebut. Trufflehog dirancang untuk menelusuri riwayat *commit*, cabang (*branches*), sistem penyimpanan awan, hingga artefak biner/arsip guna meminimalkan *false positives* melalui validasi langsung.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Audit Keamanan Repositori & Riwayat Git:** Mengidentifikasi kunci API, *private key*, *access token*, atau kredensial basis data yang pernah dimasukkan ke repositori Git, GitHub, atau GitLab.
* **Pemindaian Multi-Sumber & Infrastruktur:** Selain repositori kode, perangkat ini berguna untuk memindai sistem berkas lokal (*filesystem*), *bucket* penyimpanan objek (seperti AWS S3 dan Google Cloud Storage), serta aliran input (*stdin*) atau konfigurasi multi-sumber.
* **Triase Risiko Berbasis Validasi:** Membedakan antara kredensial yang sekadar cocok secara pola/entropi dengan kredensial yang benar-benar aktif (*verified*) sehingga tim keamanan dapat memprioritaskan rotasi kunci secara rasional.

#### ⚙️ Fungsi Teknis (Functionality)
* **Deteksi Multi-Klasifikasi & Entropi:** Menggunakan ratusan hingga lebih dari 800 detektor tipe *secret* (seperti AWS, Stripe, Cloudflare, dan kunci privat) serta penyaringan berbasis Shannon *entropy* dan dekode iteratif berlapis (misalnya Base64 di dalam UTF-16) untuk mendeteksi kredensial yang tersandi atau tersembunyi.
* **Verifikasi Aktif (*Active Verification*):** Menguji kredensial yang terdeteksi secara langsung terhadap *endpoint*/API layanan terkait (atau mekanisme seperti verifikasi kunci privat SSH/TLS) untuk mengklasifikasikan hasil menjadi *verified*, *unverified*, atau *unknown*.
* **Analisis Hak Akses (*Permissions Analysis*):** Pada sejumlah tipe kredensial yang umum bocor, subperintah analisisnya dapat menelusuri identitas pemilik kredensial, sumber daya yang dapat diakses, serta tingkat izin (*permissions*) yang melekat pada kredensial tersebut.

#### 💻 Contoh Penggunaan CLI (Trufflehog)

```bash
# Pemindaian repositori GitHub publik dengan verifikasi aktif
trufflehog git https://github.com/target/repository.git --only-verified

# Pemindaian sistem berkas lokal
trufflehog filesystem /path/to/source/code
```

---

### Part II: Gitxray

#### 📄 Deskripsi Perangkat Lunak
Gitxray (singkatan dari Git X-Ray) adalah perangkat analisis keamanan berbasis Python yang memanfaatkan Public GitHub REST API (serta integrasi API pendukung seperti Code Search dan VirusTotal pada fitur tertentu) untuk keperluan OSINT, forensik digital, dan pengujian penetrasi pada repositori serta organisasi GitHub. Secara konseptual, Gitxray mengeksplorasi celah visibilitas manusia ketika melakukan inspeksi manual—termasuk metadata profil, kunci kriptografi publik, riwayat *workflow*, dan pola aktivitas kontributor—yang secara tidak sadar menyingkap informasi sensitif atau indikasi anomali integritas repositori.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Pengumpulan Intelijen & Jejak Kontributor (OSINT):** Mengumpulkan alamat email publik, menganalisis profil kontributor, serta mendeteksi informasi tak disengaja yang melekat pada identitas publik pengembang.
* **Investigasi Forensik & Audit Integritas:** Membantu menelusuri linimasa kejadian berdasarkan tanggal insiden, mendeteksi manipulasi tanggal *commit*, perubahan aset rilis setelah dirilis, atau penghapusan *workflow run* yang berpotensi menyamarkan jejak audit.
* **Deteksi Anomali Aktor & Repositori Tiruan:** Mengidentifikasi indikasi akun bersama/terkait (*associations*), pola *stargazer* mencurigakan, eksekusi *workflow* oleh pihak eksternal (*non-contributors*), serta repositori bernama serupa/duplikat yang berpotensi menyesatkan (*typosquatting* atau kloning).

#### ⚙️ Fungsi Teknis (Functionality)
* **Ekstraksi Kebocoran pada Data Kunci & Profil (`user_input`):** Memeriksa nama kunci PGP/SSH serta teks berlebih di luar blok Armored PGP key yang sering memuat *hostname*, nama pengguna lokal, *path* direktori shell, atau catatan sensitif akibat kesalahan salin-tempel.
* **Analisis Asosiasi & Pola Aktivitas (*association & Commit Time Analysis*):** Melakukan *fingerprinting* kunci pada profil dan tanda tangan *commit* beserta linimasa pembuatan akun/kunci untuk mendeteksi keterkaitan antar-akun, sekaligus menghitung distribusi persentase *commit* per jam (UTC) guna mengenali pola atau anomali aktivitas.
* **Inspeksi Workflow, Referensi Host, & Pelaporan:** Memantau penghapusan *workflow runs*, referensi rahasia/input pengguna dalam *workflow*, mendeteksi duplikasi nama repositori ber-reputasi tinggi, menelusuri referensi host HTTP(S) dengan pengecekan reputasi melalui VirusTotal, serta menghasilkan laporan terstruktur dalam format HTML, JSON, atau teks.

#### 💻 Contoh Penggunaan CLI (Gitxray)

```bash
# Menjalankan pemindaian OSINT dan forensik pada organisasi GitHub
python3 gitxray.py -t <GITHUB_API_TOKEN> -o target-org -out report.html
```

---

### 📊 Tabel Perbandingan Akademis: Trufflehog vs Gitxray

| Aspek Komparasi | Trufflehog | Gitxray |
| --- | --- | --- |
| **Fokus Utama** | Penemuan, validasi, dan analisis *secrets* / kredensial | OSINT repositori, forensik metadata, dan analisis perilaku/integritas |
| **Target Sumber** | Git, GitHub, GitLab, *filesystem*, S3, GCS, *stdin*, arsip/biner | Repositori dan organisasi GitHub (melalui GitHub API & integrasi pendukung) |
| **Mekanisme Kunci** | Detektor pola/entropi, dekode berlapis, pengujian API langsung, analisis izin | Analisis atribut PGP/SSH, korelasi kunci/waktu *commit*, audit *workflow*, cek reputasi |
| **Nilai Forensik/Keamanan** | Menentukan keberadaan dan status aktif kredensial yang bocor | Mengungkap jejak lingkungan, anomali riwayat, serta potensi manipulasi atau kloning |

> 💡 **Kesimpulan:**
> Secara akademis, kombinasi Trufflehog dan Gitxray membuktikan bahwa repositori kode modern menyimpan jejak digital sensitif yang melampaui sekadar kode sumber itu sendiri. Trufflehog menangani dimensi kerentanan fisik berupa *secrets* yang aktif, sementara Gitxray mengungkap dimensi intelijen berupa metadata, identitas pengembang, dan anomali perilaku komit. Penggunaannya di Kali Linux menegaskan pentingnya penerapan *Pre-commit Hooks*, pemindaian rahasia otomatis pada pipa CI/CD (*DevSecOps*), rotasi kredensial berkala, serta edukasi mengenai kebersihan operasional identitas publik pengembang.
