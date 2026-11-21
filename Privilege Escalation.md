# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/tree/main)
# Privilege Escalation
# 🛠️ Privilege Escalation Tools - Kali Linux 

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak pasca-eksploitasi (*post-exploitation*) dan peningkatan hak akses (*privilege escalation*) yang terdapat dalam distribusi **Kali Linux**.

---

## 📋 Daftar Isi

1. [PEASS - LinPEAS & WinPEAS](#1-peass---linpeas--winpeas)
   - [LinPEAS (Linux)](#bagain-i-linpeas-linux-privilege-escalation-awesome-script)
   - [WinPEAS (Windows)](#bagian-ii-winpeas-windows-privilege-escalation-awesome-script)
2. [Lynis](#2-lynis)
3. [Unix-privesc-check](#3-unix-privesc-check)
4. [BloodyAD](#4-bloodyad)
5. [Metasploit-Framework](#5-metasploit-framework)

---

## 1. PEASS - LinPEAS & WinPEAS

> **Ringkasan:** Skrip pemindai otomatis yang sangat populer. LinPEAS digunakan untuk sistem operasi berbasis Linux/Unix, sedangkan WinPEAS digunakan untuk Windows. Fungsi utamanya adalah mencari segala kemungkinan jalur eskalasi hak akses di sistem lokal dan menampilkan hasilnya dengan visualisasi warna untuk mempermudah analisis celah keamanan.

### Bagian I: LinPEAS (Linux Privilege Escalation Awesome Script)

#### 📄 Deskripsi Perangkat Lunak
LinPEAS adalah sebuah skrip audit keamanan *open-source* berbasis Bourne-Again Shell (`bash`/`sh`) yang dirancang khusus untuk memindai lingkungan sistem operasi berbasis Unix, Linux, dan macOS.

Secara arsitektural, LinPEAS beroperasi tanpa memerlukan dependensi eksternal maupun kompilator di mesin target—memanfaatkan utilitas asli bawaan sistem (*native binaries*) untuk mengekstrak konfigurasi sistem secara menyeluruh. Hasil pemindaian disajikan menggunakan sistem penandaan warna ANSI (*color-coded highlighting*), di mana temuan dengan probabilitas eskalasi hak akses tertinggi (seperti potensi akses `root` 99%) ditandai dengan warna merah/kuning terang untuk mempermudah triase analis.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Audit Konfigurasi Sistem Linux (*Linux Hardening Audit*):** Membantu administrator sistem dan tim pertahanan (Blue Team) mengidentifikasi kelemahan izin akses berkas, layanan jaringan internal, serta konfigurasi kernel sebelum dieksploitasi oleh pihak tidak berwenang.
* **Simulasi Eskalasi Hak Akses Lokal (*Red Teaming*):** Digunakan oleh penguji penetrasi (*penetration tester*) setelah memperoleh akses pengguna dengan hak istimewa rendah (seperti `www-data` atau pengguna biasa) untuk menemukan vektor kenaikan hak akses menuju pengguna administratif tertinggi (`root`).

#### ⚙️ Fungsi Teknis (Functionality)
LinPEAS mengeksekusi ratusan pemeriksaan sistemik yang dikelompokkan ke dalam domain audit berikut:
* **Identifikasi Kerentanan Kernel dan OS:** Memeriksa versi kernel Linux dan distribusi sistem operasi untuk mencocokkannya dengan eksploitasi lokal yang diketahui (seperti Dirty COW atau Dirty Pipe).
* **Audit Hak Istimewa Sudo dan SUID/SGID:** Menganalisis konfigurasi `/etc/sudoers` (`sudo -l`), memindai berkas biner dengan bit SUID/SGID aktif yang dapat disalahgunakan (referensi GTFOBins), serta memeriksa kapabilitas POSIX (*Linux Capabilities*).
* **Pemeriksaan Tugas Terjadwal (*Cron Jobs & Systemd Timers*):** Mengidentifikasi skrip otomatis yang dieksekusi oleh `root` namun memiliki izin tulis (*writable permissions*) bagi pengguna non-administratif atau memuat jalur pustaka (`PATH`) yang rentan dibajak.
* **Pencarian Kredensial dan Kunci Rahasia:** Memindai berkas konfigurasi aplikasi, variabel lingkungan (*environment variables*), riwayat perintah shell (`.bash_history`), serta kunci privat SSH yang tersimpan secara tidak aman dalam teks biasa.
* **Deteksi Kontainer dan Lingkungan Terisolasi:** Mengidentifikasi apakah sesi berjalan di dalam wadah terisolasi (seperti Docker, LXC, atau Kubernetes) serta memeriksa potensi kesalahan konfigurasi untuk meloloskan diri dari kontainer (*container breakout*).

#### 💻 Contoh Penggunaan CLI (LinPEAS)
```bash
# 1. Menjalankan LinPEAS secara langsung dari memori via curl
curl -sL https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# 2. Menjalankan skrip LinPEAS lokal dan menyimpan output ke file teks
chmod +x linpeas.sh
./linpeas.sh -a > linpeas_output.txt
```

---

### Bagian II: WinPEAS (Windows Privilege Escalation Awesome Script)

#### 📄 Deskripsi Perangkat Lunak
WinPEAS adalah instrumen audit pasca-eksploitasi *open-source* yang dikembangkan dalam bahasa C# (.NET Framework) serta skrip Batch (`.bat`), yang dirancang secara khusus untuk memetakan vektor eskalasi hak istimewa pada sistem operasi Microsoft Windows.

Secara arsitektural, WinPEAS tersedia dalam bentuk berkas biner terkompilasi (`.exe`) untuk berbagai versi .NET maupun skrip batch mandiri. Desain berbasis C# memungkinkan WinPEAS berinteraksi langsung dengan Windows API, Windows Registry, dan WMI (*Windows Management Instrumentation*) secara cepat dan terstruktur, serta dapat dimuat langsung di dalam memori (*in-memory execution* via PowerShell/Beacon) untuk meminimalkan jejak deteksi cakram keras.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Evaluasi Keamanan Infrastruktur Windows (*Windows Security Assessment*):** Membantu tim keamanan mengaudit kepatuhan konfigurasi registri, layanan sistem (*Windows Services*), dan kebijakan grup (*Group Policy*) terhadap standar keamanan industri.
* **Peningkatan Akses Pasca-Eksploitasi (*Red Teaming*):** Memfasilitasi penguji penetrasi dalam memetakan jalur eskalasi dari akun domain/lokal standar menuju hak akses administratif penuh (`Administrator` atau `NT AUTHORITY\SYSTEM`).
* **Pengujian Deteksi Endpoint (*EDR/AV Behavioral Testing*):** Digunakan oleh tim Blue Team untuk menguji kemampuan sensor *Endpoint Detection and Response* (EDR) dalam mendeteksi aktivitas enumerasi sistem massal yang dilakukan oleh biner .NET.

#### ⚙️ Fungsi Teknis (Functionality)
WinPEAS mengimplementasikan mekanisme inspeksi mendalam terhadap arsitektur internal Windows melalui lima pilar fungsional:
* **Audit Layanan dan Izin Direktori (*Windows Services Enumeration*):** Memindai seluruh layanan latar belakang untuk mendeteksi kerentanan *Unquoted Service Paths* (jalur eksekusi tanpa tanda kutip yang mengandung spasi), izin modifikasi berkas biner layanan yang lemah, serta izin registri layanan yang dapat diubah oleh pengguna biasa.
* **Pemeriksaan Token Hak Istimewa (*Token Privileges Analysis*):** Mengevaluasi hak akses token pengguna aktif, khususnya mendeteksi keberadaan privilese berbahaya seperti `SeImpersonatePrivilege` atau `SeAssignPrimaryTokenPrivilege` yang dapat dieksploitasi menggunakan teknik keluarga Potato (JuicyPotato, PrintSpoofer, GodPotato).
* **Inspeksi Registri dan Kebijakan Sistem:** Memeriksa kunci registri sensitif seperti pengaturan `AlwaysInstallElevated` (yang mengizinkan paket `.msi` berjalan dengan hak SYSTEM), konfigurasi `AutoLogon`, serta berkas `Unattend.xml` yang kerap menyimpan kredensial administrator.
* **Enumerasi Kredensial Tersimpan (*Credential & DPAPI Discovery*):** Memindai Windows Credential Manager, registri PuTTY/WinSCP, berkas konfigurasi peramban, serta sertifikat *Data Protection API* (DPAPI) yang dapat diakses oleh pengguna saat ini.
* **Pemetaan Jaringan dan Perangkat Lunak Terpasang:** Mengidentifikasi antarmuka jaringan, aturan Windows Firewall, proses yang berjalan dengan hak tinggi, serta daftar aplikasi pihak ketiga beserta versinya untuk mencocokkan kerentanan lokal yang diketahui.

#### 💻 Contoh Penggunaan CLI (WinPEAS)
```cmd
:: 1. Menjalankan WinPEAS biner (.exe) via Command Prompt dan mengalihkan output ke file
winPEASx64.exe quiet cmd > winpeas_log.txt

:: 2. Eksekusi WinPEAS langsung di memori via PowerShell (In-Memory Execution)
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/peass-ng/PEASS-ng/master/winPEAS/winPEASexe/winPEAS/bin/Obfuscated%20Releases/winPEASx64.exe')"
```

#### 📊 Perbandingan Singkat PEASS

| Aspek Komparasi | LinPEAS | WinPEAS |
| :--- | :--- | :--- |
| **Platform Target** | Unix, Linux, dan macOS. | Microsoft Windows (Desktop & Server). |
| **Arsitektur Bahasa** | Skrip Shell (`bash`/`sh` tanpa dependensi). | Biner C# (.NET) dan skrip Batch (`.bat`). |
| **Fokus Audit Utama** | Bit SUID/SGID, konfigurasi sudo, Cron jobs, dan kapabilitas kernel. | Izin layanan Windows, token privilese (`SeImpersonate`), dan kunci registri. |

> 💡 **Kesimpulan:**
> Secara akademis, LinPEAS dan WinPEAS membuktikan bahwa sebagian besar keberhasilan eskalasi hak istimewa lokal tidak berasal dari eksploitasi memori tingkat rendah (*zero-day*), melainkan dari akumulasi kelalaian administratif dan salah konfigurasi izin akses (*misconfigurations*). Keberadaan rangkaian PEASS di Kali Linux menegaskan pentingnya penerapan prinsip *Least Privilege* secara ketat, audit konfigurasi berkala, serta pemantauan anomali enumerasi sistem pada seluruh aset komputasi organisasi.

---

## 2. Lynis

> **Ringkasan:** Alat audit keamanan internal dan pemindai kerentanan untuk sistem berbasis Unix/Linux. Meskipun sering digunakan oleh administrator sistem untuk memperkuat pertahanan (*hardening*), dalam pengujian penetrasi alat ini dipakai untuk mendeteksi kelemahan konfigurasi sistem yang bisa dimanfaatkan untuk naik ke akses `root`.

### 📄 Deskripsi Perangkat Lunak
Lynis adalah sebuah instrumen audit keamanan, pemindaian kerentanan, dan evaluasi penguatan sistem (*system hardening*) berbasis *open-source* yang dikembangkan secara modular menggunakan skrip *POSIX-compliant shell*. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Privilege Escalation**.

Secara arsitektural, Lynis beroperasi langsung di dalam mesin target (*host-based auditing*) pada sistem operasi berbasis Unix, Linux, macOS, dan BSD. Berbeda dengan pemindai kerentanan jaringan eksternal (seperti Nmap atau OpenVAS) yang mengevaluasi port terbuka dari luar, Lynis melakukan inspeksi mendalam dari dalam sistem (*white-box/local inspection*). Alat ini tidak memerlukan instalasi agen latar belakang secara permanen maupun dependensi pustaka kompilasi yang rumit, melainkan mengeksekusi serangkaian pengujian modular terhadap konfigurasi kernel, hak akses berkas, layanan aktif, serta kebijakan otentikasi sistem.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Identifikasi Vektor Eskalasi Hak Istimewa (*Privilege Escalation Assessment*):** Membantu penguji penetrasi (*penetration tester*) dan tim Red Team memetakan kelemahan konfigurasi lokal (seperti salah konfigurasi izin berkas, layanan rentan, atau kebijakan sudo yang terlalu longgar) yang dapat dimanfaatkan untuk meningkatkan hak akses dari pengguna standar menuju `root`.
* **Audit Penguatan Sistem (*System Hardening & Compliance Auditing*):** Digunakan oleh administrator sistem dan tim pertahanan (Blue Team) untuk mengevaluasi sejauh mana konfigurasi server sesuai dengan standar keamanan internasional, seperti CIS Benchmarks, ISO 27001, PCI-DSS, dan HIPAA.
* **Analisis Pasca-Insiden dan Deteksi Anomali (*Post-Incident Review*):** Berfungsi sebagai instrumen verifikasi cepat untuk memeriksa integritas perangkat lunak keamanan lokal (seperti status firewall, antivirus/malware scanner, serta mekanisme pencatatan log sistem) setelah terjadinya insiden keamanan.

### ⚙️ Fungsi Teknis (Functionality)
Lynis mengimplementasikan otomatisasi audit keamanan sistem lokal tingkat lanjut melalui lima mekanisme fungsional utama:
* **Inspeksi Arsitektur Sistem dan Kernel (*Kernel & Bootloader Auditing*):** Memeriksa parameter konfigurasi kernel (`sysctl`), status modul keamanan Linux (SELinux, AppArmor), perlindungan memori (ASLR, NX bit), serta memeriksa apakah berkas bootloader (GRUB) dilindungi kata sandi untuk mencegah manipulasi akses fisik.
* **Audit Manajemen Identitas dan Otentikasi (*Authentication & Access Control Check*):** Mengevaluasi kebijakan kata sandi di `/etc/login.defs` dan modul PAM (*Pluggable Authentication Modules*), mendeteksi akun pengguna tanpa kata sandi, memeriksa kepemilikan UID 0 ganda, serta mengaudit konfigurasi keamanan layanan SSH (`/etc/ssh/sshd_config`).
* **Pemindaian Layanan Jaringan dan Perangkat Lunak (*Software, Web & Database Hardening*):** Mendeteksi perangkat lunak dan layanan yang terpasang (seperti Apache, Nginx, MySQL, PostgreSQL, Postfix, Docker) beserta versi paketnya, lalu mengaudit berkas konfigurasi masing-masing layanan untuk mencari celah kebocoran informasi atau pengaturan bawaan yang tidak aman.
* **Evaluasi Sistem Berkas dan Integritas (*File System & Permissions Analysis*):** Memindai titik kait (*mount points*) untuk memastikan penerapan opsi keamanan partisi (seperti `noexec`, `nosuid`, `nodev` pada direktori `/tmp`), memeriksa izin akses direktori kritis, serta mendeteksi keberadaan berkas *world-writable* dan biner SUID yang mencurigakan.
* **Sistem Penilaian dan Rekomendasi Mitigasi (*Hardening Index & Actionable Reporting*):** Menghasilkan laporan terstruktur ke dalam berkas log (`/var/log/lynis.log` dan `/var/log/lynis-report.dat`) yang mengklasifikasikan temuan ke dalam status *Warnings* (kelemahan kritis) dan *Suggestions* (saran penguatan), lengkap dengan perhitungan kuantitatif *Hardening Index* (skor 0–100) sebagai tolok ukur tingkat kematangan keamanan host.

### 💻 Contoh Penggunaan CLI
```bash
# 1. Menjalankan audit sistem lokal secara penuh
lynis audit system

# 2. Menjalankan audit non-interaktif (cepat) tanpa pewarnaan log untuk disimpan ke file
lynis audit system --quick --no-log-color > lynis_scan.log
```

> 💡 **Kesimpulan:**
> Secara akademis, Lynis menjembatani perspektif keamanan ofensif dan defensif melalui metodologi audit internal yang sistematis. Penempatannya di kategori Privilege Escalation Kali Linux menegaskan bahwa kelemahan konfigurasi internal (*misconfigurations*) merupakan faktor pemicu utama eskalasi hak akses, sekaligus membuktikan bahwa proses *system hardening* berbasis standar audit yang terukur adalah kunci fundamental untuk meminimalkan permukaan serangan lokal pada sistem berbasis Unix/Linux.

---

## 3. Unix-privesc-check

> **Ringkasan:** Skrip standar yang berjalan pada sistem UNIX/Linux untuk memeriksa kesalahan konfigurasi keamanan lokal. Alat ini membantu mendeteksi masalah seperti izin direktori tertulis yang terlalu longgar yang dapat dieksploitasi untuk mengganti biner sistem dengan kode manipulatif.

### 📄 Deskripsi Perangkat Lunak
Unix-privesc-check adalah sebuah instrumen audit keamanan lokal *open-source* yang dikembangkan dalam bentuk skrip shell mandiri untuk mengidentifikasi celah kesalahan konfigurasi (*misconfigurations*) yang dapat dimanfaatkan untuk melakukan eskalasi hak istimewa (*privilege escalation*) pada sistem operasi berbasis Unix dan Linux. Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Privilege Escalation**.

Secara arsitektural, Unix-privesc-check dirancang sebagai satu berkas skrip tunggal (*standalone script*) yang sangat ringan dan portabel. Alat ini menghindari ketergantungan pada pustaka eksternal atau kompilator bahasa pemrograman tingkat tinggi (seperti Python atau GCC) di mesin target, sehingga dapat langsung dijalankan pada lingkungan server Unix/Linux minimalis maupun warisan (*legacy systems*) dengan menggunakan perintah dasar POSIX shell standar.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Identifikasi Vektor Eskalasi Hak Akses Lokal (*Red Teaming*):** Membantu penguji penetrasi (*penetration tester*) yang telah memperoleh pijakan awal dengan akun pengguna berhak akses rendah (*unprivileged shell*) untuk memetakan jalur menuju hak akses administratif tertinggi (`root`) secara sistematis.
* **Audit Keamanan Izin Akses Sistem Berkas (*Blue Team & System Hardening*):** Digunakan oleh administrator sistem untuk memverifikasi penerapan prinsip *least privilege* pada direktori kritis, skrip otomatisasi, dan berkas konfigurasi layanan guna mencegah penyalahgunaan izin tulis oleh pengguna lokal.
* **Evaluasi Cepat pada Lingkungan Terbatas (*Minimalist Environment Assessment*):** Menjadi alternatif instrumen audit yang ideal ketika skrip pemindai modern yang lebih berat tidak dapat berjalan akibat keterbatasan sumber daya komputasi atau pembatasan lingkungan eksekusi pada target.

### ⚙️ Fungsi Teknis (Functionality)
Unix-privesc-check mengeksekusi serangkaian pemeriksaan hierarkis terhadap struktur sistem operasi melalui lima mekanisme fungsional utama:
* **Audit Keamanan Berkas Kritis dan Kredensial (*Sensitive File & Credential Inspection*):** Memeriksa izin baca dan tulis pada berkas autentikasi inti sistem seperti `/etc/passwd`, `/etc/shadow`, dan `/etc/sudoers`, serta mendeteksi keberadaan kata sandi yang tersimpan secara tidak aman di dalam berkas konfigurasi atau riwayat perintah pengguna.
* **Evaluasi Biner SUID dan SGID (*SetUID/SetGID Binary Enumeration*):** Memindai seluruh sistem berkas untuk mengidentifikasi program eksekusi yang memiliki bit SUID atau SGID aktif (khususnya yang dimiliki oleh `root`), serta memeriksa apakah pustaka bersama (*shared libraries*) yang dipanggil oleh program tersebut memiliki izin modifikasi yang lemah.
* **Inspeksi Tugas Terjadwal (*Cron Jobs & Scheduled Tasks Analysis*):** Menganalisis direktori dan berkas konfigurasi penjadwalan sistem (seperti `/etc/crontab` dan `/etc/cron.*`) untuk mendeteksi skrip yang dieksekusi secara berkala oleh `root` namun dapat dimodifikasi oleh pengguna non-administratif.
* **Pemeriksaan Proses Berjalan dan Layanan Jaringan (*Running Processes & Library Path Audit*):** Mengidentifikasi proses latar belakang yang berjalan dengan hak akses tinggi (`root`), memeriksa izin akses pada berkas biner dan skrip konfigurasi layanan tersebut, serta mengevaluasi keamanan variabel lingkungan seperti `PATH` dan `LD_PRELOAD` guna mendeteksi potensi pembajakan eksekusi (*library/binary hijacking*).
* **Mode Pemindaian Adaptif (*Standard vs. Detailed Execution Modes*):** Menyediakan dua mode operasi utama, yaitu mode *standard* untuk melakukan pemeriksaan cepat terhadap vektor eskalasi paling umum, serta mode *detailed* untuk melakukan audit rekursif mendalam terhadap seluruh izin direktori, berkas konfigurasi grup, dan direktori home pengguna di dalam sistem.

### 💻 Contoh Penggunaan CLI
```bash
# 1. Menjalankan pemeriksaan mode standar
unix-privesc-check standard > output_standard.txt

# 2. Menjalankan pemeriksaan mode detail (rekursif dan mendalam)
unix-privesc-check detailed > output_detailed.txt
```

> 💡 **Kesimpulan:**
> Secara akademis, Unix-privesc-check mendemonstrasikan bagaimana kelemahan tata kelola izin akses pada arsitektur *multi-user* Unix/Linux dapat meruntuhkan batasan isolasi antar-pengguna. Keberadaannya di Kali Linux menegaskan bahwa keamanan sistem operasi tidak hanya bergantung pada pembaruan patch kernel semata, melainkan menuntut disiplin ketat dalam pengaturan hak akses berkas (`chmod`/`chown`), pembatasan biner SUID, serta audit berkala terhadap skrip otomatisasi yang berjalan dengan hak akses `root`.

---

## 4. BloodyAD

> **Ringkasan:** Alat berbasis Python yang dirancang untuk berinteraksi dengan lingkungan Active Directory (AD) milik Windows. Alat ini membantu mengeksploitasi hak akses spesifik pada objek AD untuk meningkatkan kendali pengguna di level domain jaringan perusahaan.

### 📄 Deskripsi Perangkat Lunak
BloodyAD adalah sebuah framework dan instrumen baris perintah (*command-line*) *open-source* berbasis bahasa pemrograman Python yang dirancang khusus untuk melakukan audit, manipulasi, dan eskalasi hak istimewa (*privilege escalation*) pada infrastruktur Microsoft Active Directory (AD). Di dalam taksonomi resmi Kali Linux, perangkat lunak ini diklasifikasikan ke dalam kategori **Privilege Escalation**.

Secara arsitektural, BloodyAD beroperasi secara jarak jauh (*remote execution*) melalui protokol jaringan standar Active Directory—yaitu LDAP/LDAPS (*Lightweight Directory Access Protocol*) dan MS-RPC (*Microsoft Remote Procedure Call*)—serta mendukung berbagai mekanisme otentikasi (seperti teks biasa, Pass-the-Hash NTLM, tiket Kerberos, maupun sertifikat). Berbeda dengan alat pemetaan graf pasif seperti BloodHound yang berfokus pada pencarian jalur serangan, BloodyAD berfungsi sebagai instrumen eksekusi aktif yang langsung mengeksekusi perubahan pada atribut objek dan *Access Control Lists* (ACL) di dalam Domain Controller untuk mengeksploitasi jalur kerentanan yang ditemukan.

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Eksploitasi Jalur Eskalasi Hak Akses Active Directory (*Red Teaming*):** Membantu penguji penetrasi (*penetration tester*) mengeksekusi serangan berbasis penyalahgunaan izin ACL (seperti `GenericAll`, `WriteDacl`, `WriteOwner`, atau `ForceChangePassword`) guna meningkatkan hak akses dari akun pengguna domain standar menuju kelompok administratif tertinggi seperti `Domain Admins`.
* **Validasi Temuan Pemetaan Graf BloodHound:** Digunakan untuk memverifikasi secara empiris apakah hubungan delegasi dan izin objek yang ditandai rentan oleh pemindai AD benar-benar dapat dieksploitasi secara nyata di lingkungan produksi.
* **Audit Integritas ACL dan Pengujian Deteksi SIEM/EDR (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan menguji efektivitas pemantauan log peristiwa keamanan Windows (*Windows Event Logs*, khususnya Event ID terkait modifikasi objek direktori dan perubahan keanggotaan grup) dalam mendeteksi manipulasi konfigurasi Active Directory secara tidak sah.

### ⚙️ Fungsi Teknis (Functionality)
BloodyAD mengimplementasikan otomatisasi manipulasi objek direktori dan eksploitasi protokol AD tingkat lanjut melalui lima mekanisme fungsional utama:
* **Manipulasi Atribut dan Keanggotaan Objek (*Object & Group Manipulation*):** Menyediakan fungsi langsung (`add`, `set`, `remove`) untuk memodifikasi atribut objek di dalam database LDAP, seperti menambahkan pengguna ke grup berhak tinggi (`add groupMember`), mengubah kata sandi akun target tanpa mengetahui kata sandi lama (`set password`), hingga memodifikasi status kontrol akun (`userAccountControl`).
* **Eksploitasi Access Control Lists (*ACL & Ownership Abuse*):** Mengotomatisasi pengambilalihan kepemilikan objek (`set owner`) dan penyuntikan hak akses penuh pada *Discretionary Access Control List* (`add genericAll` atau `add dcsync`), yang memungkinkan akun biasa mengekstrak seluruh hash kata sandi domain melalui protokol replikasi direktori (*DCSync attack*).
* **Konfigurasi Delegasi Kerberos (*RBCD & Shadow Credentials Injection*):** Mendukung injeksi atribut `msDS-AllowedToActOnBehalfOfOtherIdentity` untuk melancarkan serangan *Resource-Based Constrained Delegation* (RBCD), serta penyuntikan sertifikat kriptografi ke atribut `msDS-KeyCredentialLink` (*Shadow Credentials*) guna memperoleh tiket TGT (*Ticket Granting Ticket*) Kerberos atas nama target.
* **Manipulasi DNS dan Objek Komputer (*ADIDNS & Machine Account Management*):** Mampu menambahkan atau memodifikasi rekaman Active Directory Integrated DNS (`add dnsRecord`) untuk membelokkan lalu lintas otentikasi jaringan (mendukung serangan NTLM Relay), serta membuat objek akun komputer baru (`add computer`) dengan memanfaatkan kuota bawaan `MachineAccountQuota`.
* **Inspeksi dan Pencarian Objek Terarah (*Targeted LDAP Querying & Writable Audit*):** Menyediakan modul pembacaan (`get`) yang mampu memfilter objek sensitif, membaca *Group Managed Service Accounts* (gMSA), serta memindai seluruh objek dan atribut di dalam domain yang memiliki izin tulis (`get writable`) bagi pengguna yang sedang terautentikasi.

### 💻 Contoh Penggunaan CLI
```bash
# 1. Menambahkan akun pengguna biasa ke dalam grup 'Domain Admins' via LDAP
bloodyAD -u 'john' -p 'Password123!' -d 'corp.local' --host '192.168.1.10' add groupMember 'Domain Admins' 'john'

# 2. Mengubah kata sandi pengguna target secara paksa (Force Change Password)
bloodyAD -u 'john' -p 'Password123!' -d 'corp.local' --host '192.168.1.10' set password 'targetuser' 'NewPass2026!'

# 3. Memeriksa atribut dan objek yang dapat ditulis (writable) oleh akun saat ini
bloodyAD -u 'john' -p 'Password123!' -d 'corp.local' --host '192.168.1.10' get writable
```

> 💡 **Kesimpulan:**
> Secara akademis, BloodyAD mendemonstrasikan bahwa kompleksitas struktur izin (*Access Control Lists*) dan hubungan delegasi di dalam ekosistem Active Directory sering kali menciptakan celah eskalasi hak istimewa secara logis tanpa memerlukan eksploitasi kode biner pada sistem operasi. Keberadaannya di Kali Linux menegaskan pentingnya penerapan prinsip *Least Privilege* pada delegasi administratif AD, audit rutin terhadap izin ACL objek sensitif, serta pemantauan *real-time* atas setiap perubahan atribut kritis di Domain Controller.

---

## 5. Metasploit-Framework

> **Ringkasan:** Platform pengujian penetrasi serbaguna yang juga menyediakan berbagai modul otomatis untuk eskalasi hak akses (seperti perintah `getsystem` atau modul `local_exploit_suggester`).

### 📄 Deskripsi Perangkat Lunak
Metasploit Framework adalah sebuah platform rekayasa eksploitasi dan pengujian penetrasi *open-source* berbasis bahasa pemrograman Ruby. Di dalam taksonomi resmi Kali Linux, instrumen ini diklasifikasikan secara lintas-kategori sebagai perangkat lunak multifungsi pada fase: *Initial Access*, *Execution*, *Privilege Escalation*, dan *Command and Control* (C2).

Secara arsitektural, Metasploit beroperasi sebagai kerangka kerja modular (*modular architecture*) yang memisahkan antara kode eksploitasi (*exploit*), muatan instruksi (*payload*), dan mekanisme penyamaran (*encoder*). Pendekatan abstraksi ini memungkinkan peneliti keamanan siber mengembangkan, menguji, serta mengeksekusi vektor serangan terhadap berbagai arsitektur sistem operasi dan layanan jaringan secara terstandardisasi melalui antarmuka baris perintah interaktif (`msfconsole`).

### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Validasi Kerentanan Sistem (*Vulnerability Verification*):** Membantu penguji penetrasi (*penetration tester*) membuktikan apakah kerentanan yang teridentifikasi pada fase pemindaian (*vulnerability scanning*) benar-benar dapat dieksploitasi secara nyata (*exploitable*), sehingga mengurangi temuan positif palsu (*false positives*).
* **Simulasi Siklus Serangan Lengkap (*End-to-End Adversary Simulation*):** Digunakan oleh praktisi Red Team untuk menyimulasikan seluruh rantai serangan siber (*Cyber Kill Chain*), mulai dari perolehan akses awal (*initial access*), eksekusi kode jarak jauh, eskalasi hak istimewa (*privilege escalation*), hingga pergerakan lateral di dalam jaringan korporat.
* **Pengembangan dan Pengujian IDS/IPS/EDR (*Blue Team Benchmarking*):** Memungkinkan analis pertahanan dan peneliti keamanan menguji keandalan aturan tanda tangan (*signatures*) dan deteksi heuristik pada *Intrusion Detection/Prevention System* (IDS/IPS) serta *Endpoint Detection and Response* (EDR) terhadap pola eksploitasi yang diketahui.

### ⚙️ Fungsi Teknis (Functionality)
Metasploit Framework mengimplementasikan otomasi siklus hidup eksploitasi tingkat lanjut melalui enam komponen fungsional modular utama:
* **Modul Eksploitasi (*Exploits*):** Kumpulan kode yang dirancang khusus untuk memanfaatkan celah keamanan spesifik pada perangkat lunak, layanan jaringan, atau sistem operasi target (seperti *buffer overflow*, kerentanan deserialisasi, atau kesalahan konfigurasi autentikasi) guna membajak alur eksekusi program.
* **Modul Muatan (*Payloads & Meterpreter*):** Potongan kode arbitrer yang dikirimkan dan dieksekusi di memori target setelah eksploitasi berhasil. Metasploit mengintegrasikan Meterpreter, sebuah payload canggih berbasis injeksi DLL di dalam memori (*in-memory execution*) yang tidak menyentuh cakram keras (*diskless*), sehingga meminimalkan jejak forensik dan deteksi antivirus.
* **Modul Penunjang (*Auxiliary*):** Berfungsi melakukan tugas-tugas non-eksploitasi yang krusial selama fase pengintaian dan analisis, meliputi pemindaian port, *fuzzing* protokol, *sniffing* lalu lintas jaringan, enumerasi layanan (SMB, SSH, HTTP), hingga serangan *denial-of-service* (DoS).
* **Modul Pasca-Eksploitasi (*Post-Exploitation*):** Menyediakan serangkaian skrip otomatis untuk beroperasi setelah akses berhasil didapatkan, seperti ekstraksi kredensial memori (*credential dumping*), pencatatan ketikan papan ketik (*keylogging*), eskalasi hak istimewa lokal (*local privilege escalation*), serta mekanisme *pivoting* untuk menjadikan mesin target sebagai jembatan menuju sub-jaringan internal yang terisolasi.
* **Modul Pengode dan Penghindar Deteksi (*Encoders & Evasion*):** Bertugas memodifikasi struktur biner payload menggunakan algoritma penyandian polimorfik guna menghilangkan karakter rusak (*bad characters*) sekaligus melewati sistem pemindaian antivirus statis.
* **Manajemen Basis Data Terintegrasi:** Terhubung langsung dengan basis data PostgreSQL untuk menyimpan, mengindeks, dan mengkorelasikan hasil pemindaian host, layanan, kerentanan, serta kredensial yang berhasil dikumpulkan selama operasi pengujian berlangsung.

### 💻 Contoh Penggunaan CLI
```bash
# 1. Menjalankan antarmuka utama Metasploit Console
msfconsole

# 2. Menggunakan modul Local Exploit Suggester setelah mendapatkan sesi Meterpreter
msf6 > use post/multi/recon/local_exploit_suggester
msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1
msf6 post(multi/recon/local_exploit_suggester) > run

# 3. Perintah eskalasi otomatis langsung di dalam sesi Meterpreter Windows
meterpreter > getsystem
```

> 💡 **Kesimpulan:**
> Secara akademis, Metasploit Framework merepresentasikan standardisasi rekayasa eksploitasi modern yang mengubah pengujian keamanan dari sekadar manipulasi kode manual yang rumit menjadi disiplin metodologis yang terstruktur dan dapat direproduksi. Keberadaannya di Kali Linux menegaskan pentingnya pendekatan pertahanan berlapis (*defense-in-depth*), manajemen tambalan perangkat lunak (*patch management*) yang disiplin, serta pemantauan anomali memori secara proaktif pada setiap infrastruktur komputasi.
