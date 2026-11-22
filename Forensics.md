# [Kali Linux Categories](https://github.com/kiyanami/Kali-Linux-Category/blob/main/Kali%20Linux%20Category.md)
# Forensics
# 🛠️ Forensics Tools - Kali Linux Documentation

Dokumentasi ini berisi penjelasan mendalam mengenai berbagai perangkat lunak forensik digital, akuisisi bukti, pengukiran berkas (*file carving*), rekayasa balik *firmware*, dan pendeteksian *rootkit* (*Digital Forensics and Incident Response* / DFIR) yang terdapat dalam distribusi **Kali Linux**.

## 📋 Daftar Isi

1. [Autopsy & The Sleuth Kit Suite (Sleuth Kit Suite)](#1-autopsy--the-sleuth-kit-suite-sleuth-kit-suite)
   * [Part I: The Sleuth Kit (TSK) Suite](#part-i-the-sleuth-kit-tsk-suite)
   * [Part II: Autopsy](#part-ii-autopsy)
   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-tsk-vs-autopsy)
2. [Guymager & dc3dd (Forensic Imaging Tools)](#2-guymager--dc3dd-forensic-imaging-tools)
   * [Part I: Guymager](#part-i-guymager)
   * [Part II: dc3dd](#part-ii-dc3dd)
   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-guymager-vs-dc3dd)
3. [Foremost & Scalpel (Forensic Carving Tools)](#3-foremost--scalpel-forensic-carving-tools)
   * [Part I: Foremost](#part-i-foremost)
   * [Part II: Scalpel](#part-ii-scalpel)
   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-foremost-vs-scalpel)
4. [Binwalk & Binwalk3](#4-binwalk--binwalk3)
   * [Part I: Binwalk (v2)](#part-i-binwalk-v2)
   * [Part II: Binwalk3 (v3)](#part-ii-binwalk3-v3)
   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-binwalk-vs-binwalk3)
5. [Chkrootkit & Rkhunter](#5-chkrootkit--rkhunter)
   * [Part I: chkrootkit](#part-i-chkrootkit-check-rootkit)
   * [Part II: rkhunter](#part-ii-rkhunter-rootkit-hunter)
   * [Perbandingan Singkat & Kesimpulan Akademis](#perbandingan-singkat--kesimpulan-akademis-chkrootkit-vs-rkhunter)

---

## 1. Autopsy & The Sleuth Kit Suite (Sleuth Kit Suite)

> **Ringkasan:** Autopsy adalah antarmuka grafis (GUI) berbasis web/desktop yang berjalan di atas The Sleuth Kit. Alat ini merupakan platform investigasi forensik digital komprehensif yang digunakan oleh penegak hukum maupun militer untuk menganalisis citra (*image*) harddisk, memeriksa riwayat web, mengekstrak geolokasi, hingga memulihkan berkas yang dihapus.

### Part I: The Sleuth Kit (TSK) Suite

#### 📄 Deskripsi Perangkat Lunak
The Sleuth Kit (TSK) Suite adalah sebuah koleksi pustaka pemrograman dan kumpulan utilitas baris perintah (*Command-Line Interface* / CLI) berbasis *open-source* yang dikembangkan dalam bahasa C dan C++. 

Secara arsitektural, TSK beroperasi langsung pada lapisan struktur biner mentah dari citra cakram (*disk images*, seperti format RAW/dd, E01/EnCase, dan AFF) tanpa memerlukan sistem operasi host untuk melakukan *mounting* secara tradisional. Hal ini menjaga integritas barang bukti (*read-only non-destructive analysis*) sekaligus memungkinkan pembedahan sistem berkas (*file systems*) secara mendalam hingga ke tingkat alokasi blok, metadata inode/MFT, dan ruang tidak teralokasi (*unallocated space*).

### Part II: Autopsy

#### 📄 Deskripsi Perangkat Lunak
Autopsy adalah sebuah platform forensik digital menyeluruh (*end-to-end digital forensics platform*) yang berfungsi sebagai lapisan antarmuka grafis (*Graphical User Interface* / GUI) modern berbasis Java di atas mesin inti The Sleuth Kit.

Secara arsitektural, Autopsy mengintegrasikan TSK dengan berbagai modul analisis otomatis berbasis Java dan Python (Jython) untuk mengelola kasus investigasi berskala besar, mengindeks artefak sistem operasi, memvisualisasikan lini masa kejadian (*timeline*), serta menghasilkan laporan forensik berstandar hukum.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Investigasi Pasca-Insiden dan Analisis Kompromi (*Incident Response & Root Cause Analysis*):** Membantu tim Blue Team dan investigator forensik merekonstruksi jejak aktivitas aktor ancaman pada sistem yang disusupi—seperti mengidentifikasi waktu masuk awal, malware yang ditanamkan, berkas konfigurasi yang dimodifikasi, hingga bukti upaya penghapusan jejak (*anti-forensics*).
* **Pemulihan Berkas Terhapus dan Pengukiran Data (*Deleted File Recovery & Data Carving*):** Digunakan untuk memulihkan dokumen, gambar, atau artefak log yang telah dihapus oleh pengguna maupun penyerang dari sistem berkas target tanpa merusak struktur citra bukti asli.
* **Analisis Rekam Jejak Aktivitas Pengguna (*User Activity & Artifact Profiling*):** Memungkinkan investigator mengekstrak bukti perilaku pengguna pada perangkat target, meliputi riwayat penjelajahan web, komunikasi email, metadata dokumen, perangkat USB yang pernah terhubung, serta informasi registri Windows.
* **Validasi Keabsahan Bukti Digital (*Court-Admissible Evidence Processing*):** Berfungsi sebagai laboratorium forensik terstandardisasi yang mendokumentasikan rantai penyimpanan bukti (*Chain of Custody*), integritas hash kriptografis (MD5/SHA-256), serta laporan kronologis untuk keperluan audit internal maupun pembuktian di pengadilan.

#### ⚙️ Fungsi Teknis (Functionality)
* **Analisis Lapisan Citra dan Volume Partisi (`img_stat`, `mmls`, `mmstat`):** Membedah struktur tabel partisi cakram keras (seperti MBR dan GPT), mengidentifikasi sektor awal dan akhir setiap partisi, serta mendeteksi ruang tersembunyi antar-partisi (*hidden/unallocated volume spaces*).
* **Inspeksi Lapisan Sistem Berkas dan Metadata (`fsstat`, `fls`, `istat`, `ffind`):** Mendukung analisis berbagai sistem berkas populer (NTFS, FAT12/16/32, exFAT, Ext2/Ext3/Ext4, UFS, HFS+, APFS, dan ISO9660). Utilitas seperti `fls` mendaftar nama berkas (termasuk berkas yang telah dihapus), `istat` menampilkan metadata lengkap seperti stempel waktu MACB (*Modified, Accessed, Created/MFT Changed, Born*), dan `ffind` memetakan kembali nomor inode ke nama berkasnya.
* **Ekstraksi Konten Data dan Ruang Tidak Teralokasi (`icat`, `blkls`, `blkcat`, `tsk_recover`):** Mengekstrak isi biner mentah dari suatu inode atau entri MFT (`icat`) tanpa mengubah stempel waktu akses aslinya, mengekspor seluruh blok data tidak teralokasi (`blkls` / *unallocated space*) untuk keperluan *data carving*, serta memulihkan seluruh berkas terhapus secara massal ke direktori eksternal (`tsk_recover`).
* **Rekonstruksi Lini Masa Forensik (*Super Timeline Generation* / `fls` + `mactime`):** Mengumpulkan seluruh atribut stempel waktu sistem berkas dan log peristiwa untuk menyusun urutan kronologis aktivitas sistem secara presisi (`mactime`), memungkinkan investigator melihat seluruh kejadian sebelum, selama, dan sesudah insiden keamanan berlangsung.
* **Otomasi Pemrosesan Modul Ingest dan Pencarian Hash (Autopsy Ingest Modules & `hfind`):** Di dalam antarmuka Autopsy, mesin pemrosesan otomatis (*Ingest Modules*) mengeksekusi pencocokan nilai hash kriptografis terhadap basis data NSRL (`hfind`) untuk menyaring berkas sistem yang diketahui aman, melakukan pengindeksan teks penuh (Apache Solr), mengekstrak EXIF gambar, memarsing Windows Registry, serta mendeteksi indikator perangkat lunak berbahaya.
* **Pelaporan Forensik Terstruktur dan Multi-Format (Autopsy Reporting):** Mengonsolidasikan seluruh temuan yang ditandai investigator (*tagged artifacts*), hasil kueri kata kunci, grafik komunikasi, dan lini masa ke dalam laporan terstruktur berformat HTML, Excel (XLSX), CSV, maupun format pertukaran bukti digital standar CASE/UCO.

#### 📊 Perbandingan Singkat & Kesimpulan Akademis: TSK vs Autopsy

| Aspek Komparasi | The Sleuth Kit (TSK) Suite | Autopsy |
| :--- | :--- | :--- |
| **Antarmuka Pengguna** | Kumpulan utilitas baris perintah (CLI) modular berbasis C/C++. | Platform desktop grafis (GUI) terpadu berbasis Java dengan basis data SQLite/PostgreSQL. |
| **Kekuatan Khas** | Sangat cepat, ringan, presisi tinggi untuk inspeksi struktur bit rendah, dan mudah diotomatisasi via skrip. | Visualisasi kasus menyeluruh, pengindeksan kata kunci massal, korelasi multi-artefak otomatis, dan pembuatan laporan formal. |

#### 💻 Contoh Penggunaan CLI

```bash
# 1. Menampilkan struktur tabel partisi dari citra disk RAW
mmls evidence.dd

# 2. Mendaftar seluruh berkas terhapus pada partisi tertentu (offset sektor 2048)
fls -r -d -o 2048 evidence.dd

# 3. Membuat bodyfile stempel waktu dan menyusun super timeline
fls -r -m / -o 2048 evidence.dd > bodyfile.txt
mactime -b bodyfile.txt -d > timeline.csv

# 4. Memulihkan seluruh berkas terhapus secara otomatis dari citra disk
tsk_recover -e -o 2048 evidence.dd /tmp/recovered_files/

# 5. Menjalankan antarmuka GUI Autopsy
autopsy &
```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Autopsy dan The Sleuth Kit Suite membuktikan prinsip fundamental ilmu forensik digital (*Locard’s Exchange Principle* dalam ruang siber), bahwa setiap tindakan penghapusan atau modifikasi data pada media penyimpanan tidak serta-merta menghilangkan jejak bit fisik maupun metadata strukturnya hingga sektor tersebut tertimpa data baru. Keberadaannya di dalam kategori Forensics Kali Linux menegaskan bahwa kemampuan membedah lapisan abstraksi sistem berkas—tanpa bergantung pada API sistem operasi yang berpotensi dimanipulasi oleh *rootkit*—merupakan prasyarat mutlak dalam menjaga integritas, objektivitas, dan akuntabilitas investigasi keamanan siber modern.

---

## 2. Guymager & dc3dd (Forensic Imaging Tools)

> **Ringkasan:** Alat bantu pembuat citra forensik (*forensic imager*). Guymager menyediakan antarmuka grafis yang cepat untuk melakukan penggandaan media penyimpanan ke dalam format berkas forensik resmi (seperti `.E01` atau `.dd`), lengkap dengan kalkulasi nilai hash untuk menjamin integritas data bukti.

### Part I: Guymager

#### 📄 Deskripsi Perangkat Lunak
Guymager adalah sebuah instrumen akuisisi media penyimpanan forensik berbasis antarmuka grafis (*Graphical User Interface* / GUI) *open-source* yang dikembangkan menggunakan bahasa pemrograman C++ dan pustaka Qt.

Secara arsitektural, Guymager dirancang dengan mesin pemrosesan paralel berkecepatan tinggi (*multi-threaded pipeline architecture*) yang memisahkan utas pembacaan perangkat (*reader thread*), kalkulasi hash kriptografis, kompresi data, dan penulisan berkas (*writer thread*). Desain ini memaksimalkan *throughput* antarmuka I/O modern (seperti SATA, NVMe, dan USB 3.x) sekaligus menyajikan pelacakan status perangkat keras dan dokumentasi kasus secara visual.

### Part II: dc3dd

#### 📄 Deskripsi Perangkat Lunak
`dc3dd` adalah sebuah utilitas akuisisi forensik dan pemrosesan aliran data mentah berbasis baris perintah (*Command-Line Interface* / CLI) yang dikembangkan dalam bahasa C oleh U.S. Department of Defense Cyber Crime Center (DC3) sebagai pengembangan lanjutan dari utilitas standar Unix `dd`.

Secara arsitektural, berbeda dengan `dd` tradisional yang hanya menyalin aliran byte tanpa fitur keamanan forensik bawaan, `dc3dd` menyematkan kemampuan verifikasi integritas kriptografis *on-the-fly*, pencatatan log terperinci, pemecahan berkas otomatis, serta penghapusan data aman (*forensic wiping*) langsung ke dalam satu eksekusi aliran data (*single-pass I/O*).

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Akuisisi Citra Forensik Berstandar Pengadilan (*Court-Admissible Disk Imaging*):** Membantu investigator forensik digital dan aparat penegak hukum menggandakan cakram keras (HDD/SSD), kartu memori, maupun USB flash drive ke dalam format berkas standar industri (E01 / EnCase Expert Witness Format, AFF, atau RAW dd) beserta metadata kasus yang lengkap.
* **Kloning Media Fisik ke Perangkat Cadangan (*Physical Media Cloning*):** Digunakan untuk menduplikasi seluruh struktur partisi dan sektor media bukti secara identik ke cakram keras fisik lain sebagai salinan kerja (*working copy*) guna mencegah kerusakan pada media bukti asli.
* **Verifikasi Integritas dan Dokumentasi Rantai Bukti (*Chain of Custody Documentation*):** Memastikan keabsahan hukum barang bukti dengan mencatat nomor seri perangkat keras, kapasitas sektor, informasi pemeriksa, serta menghitung dan memverifikasi nilai hash kriptografis secara otomatis setelah proses akuisisi selesai.

#### ⚙️ Fungsi Teknis (Functionality)
* **Dukungan Multi-Format Citra Forensik (EWF/E01, AFF, RAW):** Mampu menghasilkan citra forensik dalam format terkompresi *Expert Witness Format* (`.E01`) yang menyimpan metadata kasus di dalam header berkas dan membagi ukuran berkas secara otomatis (*file splitting*), serta format mentah RAW (`.dd`) dan *Advanced Forensic Format* (`.aff`).
* **Kalkulasi dan Verifikasi Hash Paralel (MD5, SHA-1, SHA-256):** Menghitung hingga tiga algoritma hash kriptografis secara bersamaan di dalam memori saat data dibaca dari perangkat sumber, lalu melakukan pembacaan ulang terhadap berkas citra hasil (*post-acquisition verification*) untuk memastikan nilai hash sumber dan citra identik 100%.
* **Deteksi Perangkat Otomatis dan Identifikasi Metadata Perangkat Keras:** Memindai bus sistem Linux secara *real-time* untuk mendeteksi perangkat blok yang terhubung, menampilkan model perangkat, nomor seri (*serial number*), ukuran sektor fisik/logis, serta status proteksi tulis (*write-blocker status*).
* **Penanganan Sektor Rusak (*Bad Sector Management*):** Mengintegrasikan mekanisme lompatan dan pengisian blok nol (*zero-padding*) ketika membaca media penyimpanan yang mengalami kerusakan fisik ringan, sehingga proses akuisisi tidak terhenti total dan lokasi sektor rusak tercatat secara akurat.
* **Pembuatan Berkas Informasi Audit Otomatis (`.info` / `log=`):** Menghasilkan berkas teks ringkasan audit terstruktur di samping berkas citra yang mendokumentasikan waktu mulai/selesai, kecepatan transfer rata-rata, parameter perintah, informasi perangkat keras, dan hasil verifikasi hash.

#### 📊 Perbandingan Singkat & Kesimpulan Akademis: Guymager vs dc3dd

| Aspek Komparasi | Guymager (`guymager`) | dc3dd (`dc3dd`) |
| :--- | :--- | :--- |
| **Antarmuka Pengguna** | Grafis (GUI berbasis Qt) dengan deteksi perangkat otomatis. | Baris perintah (CLI) murni untuk terminal dan skrip otomasi. |
| **Format Keluaran Utama** | E01 (EnCase) dengan kompresi dan metadata, AFF, serta RAW (dd). | RAW (dd) mentah (mendukung pemecahan berkas `.000`, `.001` dan keluaran ganda). |
| **Kelebihan Khas** | Ramah pengguna, menyimpan metadata kasus langsung di berkas `.E01`, kompresi tinggi. | Sangat ringan untuk server *headless*, mendukung penulisan multi-tujuan, *forensic wiping*. |

#### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan GUI Guymager dengan hak akses root
sudo guymager &

# 2. Akuisisi disk ke format raw image dengan dc3dd + kalkulasi hash SHA-256 + log audit
sudo dc3dd if=/dev/sdb of=/evidence/usb_image.dd hash=sha256 log=/evidence/audit.log

# 3. Akuisisi simultan ke dua drive tujuan berbeda dengan verifikasi hash otomatis
sudo dc3dd if=/dev/sdb of=/mnt/drive1/img.dd hof=/mnt/drive2/img.dd hash=sha256 vf=/mnt/drive1/img.dd

# 4. Menghapus (wiping) media penyimpanan target dengan pola 0x00 sebelum akuisisi
sudo dc3dd wipe=/dev/sdc hwipe=/dev/sdc log=/evidence/wipe_audit.log
```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Guymager dan `dc3dd` merepresentasikan pilar utama prinsip non-intervensi dalam metodologi forensik digital: analisis forensik tidak boleh pernah dilakukan secara langsung pada media penyimpanan asli, melainkan wajib dieksekusi pada salinan bit-demi-bit yang telah terverifikasi secara kriptografis. Keberadaan keduanya di dalam kategori Forensics Kali Linux menegaskan bahwa keabsahan bukti digital di ranah hukum maupun audit insiden sangat bergantung pada integritas akuisisi awal—di mana nilai hash kriptografis yang identik antara media sumber dan citra hasil menjadi bukti matematis bahwa data tidak mengalami perubahan sedikit pun selama proses investigasi berlangsung.

---

## 3. Foremost & Scalpel (Forensic Carving Tools)

> **Ringkasan:** Perkakas pemulihan data berbasis baris perintah (*data carving tools*). Foremost awalnya dikembangkan oleh Angkatan Udara AS, bekerja dengan memindai biner media penyimpanan untuk mencari struktur header dan footer file tertentu (seperti biner gambar `.jpg`, dokumen `.pdf`, dll.) guna menyusun kembali berkas yang telah dihapus meskipun sistem berkasnya telah rusak.

### Part I: Foremost

#### 📄 Deskripsi Perangkat Lunak
Foremost adalah sebuah utilitas pemulihan berkas konsol (*console-based file recovery program*) berbasis *open-source* yang dikembangkan dalam bahasa pemrograman C oleh U.S. Air Force Office of Special Investigations (AFOSI) dan Center for Information Systems Security Studies and Research.

Secara arsitektural, Foremost memperlakukan berkas citra forensik (seperti format dd, Safeback, atau EnCase) maupun perangkat blok fisik sebagai aliran byte kontinu (*sequential byte stream*). Alat ini membaca data blok demi blok ke dalam memori dan mencocokkan struktur biner internalnya dengan basis data definisi tanda tangan berkas (*file signatures*) guna mengekstrak kembali berkas-berkas yang hilang dari sistem berkas target.

### Part II: Scalpel

#### 📄 Deskripsi Perangkat Lunak
Scalpel adalah sebuah mesin pengukir berkas (*file carving engine*) berkinerja tinggi berbasis *open-source* yang ditulis ulang dalam bahasa C sebagai pengembangan arsitektural dari kode sumber Foremost (v0.69) oleh Golden G. Richard III.

Secara arsitektural, Scalpel dirancang untuk mengatasi hambatan kinerja (*I/O bottleneck*) dan konsumsi memori tinggi yang dialami Foremost pada citra media penyimpanan berskala terabita. Scalpel menerapkan algoritma pemindaian dua fase (*two-pass carving architecture*) dengan manajemen antrean kerja asinkron (*asynchronous I/O*) dan dukungan akselerasi pemrosesan paralel.

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Pemulihan Barang Bukti pada Partisi Terformat (*Formatted & Corrupted Partition Recovery*):** Membantu investigator forensik digital memulihkan dokumen, gambar, arsip, maupun biner eksekusi dari media penyimpanan yang tabel partisinya rusak, terhapus, atau telah diformat ulang.
* **Ekstraksi Artefak dari Ruang Tidak Teralokasi (*Unallocated Space & Memory Dump Carving*):** Digunakan dalam analisis pasca-insiden untuk mengekstrak berkas tersembunyi atau muatan malware yang tertinggal di dalam ruang kosong cakram (`blkls` output) maupun berkas *memory dump* RAM (LiME / Volatility dump).
* **Audit Keamanan Penghapusan Data (*Data Sanitization Verification*):** Berfungsi sebagai instrumen verifikasi bagi organisasi untuk menguji apakah proses penghapusan data konvensional pada perangkat keras bekas masih menyisakan berkas sensitif yang dapat dipulihkan.

#### ⚙️ Fungsi Teknis (Functionality)
* **Mesin Pengukiran Berbasis Header, Footer, dan Struktur Internal (`-t`):** Mendukung puluhan format berkas bawaan—meliputi gambar (jpg, gif, png, bmp), dokumen (pdf, doc, xls, ppt, ole), arsip (zip, rar, gz), video/audio (avi, mpg, mp4, wav), serta berkas eksekusi Windows (exe/dll).
* **Kustomisasi Aturan Tanda Tangan (`foremost.conf` / `scalpel.conf`):** Menyediakan berkas konfigurasi terstruktur yang memungkinkan peneliti forensik mendefinisikan pola tanda tangan kustom dalam format heksadesimal maupun ASCII, lengkap dengan pengaturan sensitivitas huruf besar/kecil dan batas ukuran berkas maksimum.
* **Arsitektur Pemindaian Dua Fase (Scalpel Two-Pass Engine):** Pada fase pertama (*First Pass*), Scalpel memindai seluruh citra bukti untuk membangun indeks offset header/footer di memori. Pada fase kedua (*Second Pass*), Scalpel melakukan pembacaan acak (*random access seeks*) secara efisien untuk mengekstrak byte yang valid.
* **Mode Pemrosesan Cepat dan Blok Sektor (Foremost `-q` & `-b`):** Menyediakan Quick Mode (`-q`) yang hanya memindai awal setiap sektor cakram tempat header berkas secara alami disimpan, mempercepat pemindaian citra cakram besar.
* **Pembuatan Laporan Audit Terstruktur (`audit.txt`):** Secara otomatis menghasilkan berkas ringkasan investigasi (`audit.txt`) di direktori keluaran yang mencatat daftar seluruh berkas yang berhasil diukir, offset byte asal, ukuran berkas, serta parameter eksekusi.

#### 📊 Perbandingan Singkat & Kesimpulan Akademis: Foremost vs Scalpel

| Aspek Komparasi | Foremost (`foremost`) | Scalpel (`scalpel`) |
| :--- | :--- | :--- |
| **Arsitektur Pemrosesan** | Pemindaian aliran tunggal (*single-pass sequential carving*). | Pemindaian dua fase terindeks (*two-pass indexed carving*) dengan I/O asinkron. |
| **Perilaku Konfigurasi** | Mengaktifkan berbagai tipe berkas umum secara bawaan tanpa perlu edit config. | Seluruh tipe berkas dinonaktifkan secara default; analis wajib memilih di `scalpel.conf`. |
| **Keunggulan Khas** | Sangat cepat dan praktis untuk triase cepat (*quick carving*) pada media kecil-menengah. | Hemat memori, akselerasi I/O tinggi, sangat unggul untuk citra cakram terabita. |

#### 💻 Contoh Penggunaan CLI

```bash
# 1. Mengukir gambar JPG dan dokumen PDF menggunakan Foremost
foremost -i evidence.dd -o /tmp/foremost_out -t jpg,pdf

# 2. Mengukir seluruh format yang didukung Foremost dalam mode cepat (Quick Mode)
foremost -q -i evidence.dd -o /tmp/foremost_all

# 3. Menyiapkan konfigurasi Scalpel (mengaktifkan tipe berkas target di scalpel.conf)
sudo nano /etc/scalpel/scalpel.conf

# 4. Mengeksekusi pengukiran berkas menggunakan Scalpel
scalpel -c /etc/scalpel/scalpel.conf -o /tmp/scalpel_out evidence.dd
```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Foremost dan Scalpel membuktikan bahwa penghapusan entri direktori pada sistem berkas (*file system metadata deletion*) tidak menghancurkan identitas struktural data itu sendiri, karena setiap format berkas digital membawa tanda tangan biner deterministik (*magic bytes*) pada lapisan datanya. Keberadaannya di dalam sub-kategori Forensic Carving Tools Kali Linux menegaskan dua prinsip penting: bagi tim forensik (Blue Team), *file carving* adalah instrumen pemulihan bukti yang krusial ketika metadata sistem berkas telah dirusak; sementara bagi administrator keamanan, penghapusan data sensitif sebelum pemusnahan perangkat keras wajib menggunakan metode penimpaan menyeluruh (*cryptographic erasure* atau *multi-pass data wiping*) agar sisa pola byte tidak dapat direkonstruksi kembali.

---

## 4. Binwalk & Binwalk3

> **Ringkasan:** Alat analisis biner yang sangat populer untuk menganalisis, merekayasa balik (*reverse engineering*), dan mengekstrak berkas di dalam sebuah *firmware* perangkat (seperti biner router atau perangkat IoT). Binwalk membantu investigator mencari tanda tangan digital atau kode berbahaya tersembunyi di dalam berkas tunggal yang terkompresi.

### Part I: Binwalk (v2)

#### 📄 Deskripsi Perangkat Lunak
Binwalk (`binwalk`) adalah sebuah utilitas analisis *firmware* dan biner *open-source* klasik yang dikembangkan menggunakan bahasa pemrograman Python dengan memanfaatkan pustaka pencocokan pola `libmagic`. 

Secara arsitektural, Binwalk memindai aliran byte dari awal hingga akhir berkas biner untuk mendeteksi tanda tangan struktur data yang diketahui (*magic bytes*), seperti header kompresi (LZMA, Gzip, Zlib), citra kernel Linux, dan sistem berkas tertanam (SquashFS, JFFS2, CramFS, UBIFS), lalu memanggil utilitas eksternal untuk membongkarnya.

### Part II: Binwalk3 (v3)

#### 📄 Deskripsi Perangkat Lunak
Binwalk3 (`binwalk3`) adalah generasi terbaru dari Binwalk yang ditulis ulang sepenuhnya (*complete rewrite*) dari awal menggunakan bahasa pemrograman Rust oleh pengembang aslinya (ReFirm Labs / Craig Heffner).

Secara arsitektural, Binwalk3 mengatasi keterbatasan kecepatan pemindaian dan kerentanan keamanan pada parser Python lama dengan mengimplementasikan mesin pencocokan pola Aho-Corasick berkecepatan tinggi, pemrosesan paralel *multi-threading* penuh, serta parser tanda tangan internal yang aman secara memori (*memory-safe*).

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Analisis Keamanan Firmware dan Perangkat IoT (*Firmware Reverse Engineering*):** Membantu peneliti keamanan dan penguji penetrasi membongkar citra pembaruan *firmware* router, kamera IP, maupun sistem kontrol industri (ICS/SCADA) untuk mengekstrak sistem berkas akar (*root filesystem*), sehingga analis dapat mengaudit kredensial bawaan pabrik, kunci privat kriptografis, maupun kerentanan biner layanan.
* **Forensik Malware dan Deteksi Muatan Tersembunyi (*Malware & Steganography Forensics*):** Digunakan oleh tim Blue Team dan investigator forensik untuk mendeteksi berkas eksekusi, skrip, atau arsip berbahaya yang disisipkan di akhir (*appended*) berkas media biasa dalam teknik penyelundupan data maupun steganografi biner.
* **Identifikasi Enkripsi dan Kompresi pada Artefak Biner (*Entropy & Cryptographic Assessment*):** Memungkinkan analis forensik mengevaluasi apakah suatu blok data di dalam berkas biner merupakan kode instruksi mentah, data terkompresi, atau muatan yang telah dienkripsi secara penuh.

#### ⚙️ Fungsi Teknis (Functionality)
* **Pemindaian Tanda Tangan Biner (*Signature Scanning Engine*):** Memindai seluruh rentang offset byte di dalam berkas target untuk mengidentifikasi ratusan format struktur biner—meliputi arsip terkompresi, bootloader (U-Boot), header kernel, sistem berkas tertanam, sertifikat X.509, hingga kode mesin spesifik arsitektur prosesor.
* **Ekstraksi Otomatis dan Rekursif (`-e` & `-M` / `--matryoshka`):** Mengekstrak secara otomatis setiap segmen biner atau sistem berkas yang terdeteksi ke dalam direktori terstruktur, serta mendukung mode Matryoshka (`-M`) yang secara rekursif memindai dan membongkar kembali berkas hasil ekstraksi hingga ke lapisan terdalam.
* **Analisis Entropi Informasi Shannon (`-E` / *Entropy Analysis*):** Menghitung nilai Shannon Entropy (skala 0.0 hingga 1.0) pada setiap blok data di dalam berkas biner dan memvisualisasikannya dalam bentuk grafik. Lonjakan kurva entropi mendekati nilai maksimum (~1.0) secara matematis mengindikasikan keberadaan blok data terkompresi atau terenkripsi.
* **Identifikasi Arsitektur Instruksi CPU (`-A` / *Opcodes Scanning*):** Memindai pola kode operasi (*opcodes*) bahasa rakitan (Assembly) di dalam biner mentah untuk menentukan arsitektur prosesor target (seperti ARM, MIPS, x86, x86-64, PowerPC, atau RISC-V) beserta urutan byte-nya (*Little-Endian* vs *Big-Endian*).
* **Komparasi Heksadesimal Diferensial (`-W` / *Binary Diffing*):** Membandingkan dua berkas biner secara berdampingan pada tingkat byte guna menyoroti lokasi offset spesifik yang mengalami modifikasi kode.

#### 📊 Perbandingan Singkat & Kesimpulan Akademis: Binwalk vs Binwalk3

| Aspek Komparasi | Binwalk (v2) | Binwalk3 (v3) |
| :--- | :--- | :--- |
| **Arsitektur Bahasa** | Python (mengandalkan `libmagic` & skrip ekstraktor eksternal). | Rust murni (parser internal terkompilasi dengan *memory safety*). |
| **Kinerja & Kecepatan** | Pemrosesan sekuensial; relatif lambat pada biner gigabita. | Pemrosesan paralel *multi-threaded* dengan algoritma Aho-Corasick. |
| **Keamanan Eksekusi** | Berisiko terhadap kerentanan *path traversal* pada plugin lama. | Isolasi ekstraksi ketat (*path traversal protection* bawaan). |

#### 💻 Contoh Penggunaan CLI

```bash
# 1. Pindai tanda tangan struktur biner di dalam berkas firmware
binwalk router_firmware.bin

# 2. Ekstraksi otomatis seluruh isi sistem berkas dan arsip dari firmware
binwalk -e router_firmware.bin

# 3. Ekstraksi rekursif mendalam (Matryoshka Mode)
binwalk -e -M router_firmware.bin

# 4. Analisis grafik entropi untuk mendeteksi bagian terenkripsi/terkompresi
binwalk -E router_firmware.bin

# 5. Ekstraksi menggunakan Binwalk3 (Rust engine)
binwalk3 --extract router_firmware.bin
```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, Binwalk dan Binwalk3 mendemonstrasikan bahwa penggabungan berbagai komponen perangkat lunak ke dalam satu wadah biner monolitik (*monolithic binary packaging*) tidak menghilangkan batas-batas struktural (*structural boundaries*) maupun karakteristik matematis (entropi) dari masing-masing komponen di dalamnya. Keberadaannya di dalam kategori Forensics Kali Linux menegaskan bahwa keamanan perangkat tertanam (IoT/Embedded Systems) tidak dapat mengandalkan kerahasiaan struktur *firmware* (*security through obscurity*); produsen perangkat keras wajib menerapkan enkripsi citra *firmware* berbasis kunci perangkat keras, mekanisme *Secure Boot* dengan verifikasi tanda tangan digital kriptografis, serta penghapusan seluruh kredensial bawaan dan kunci privat dari sistem berkas produksi.

---

## 5. Chkrootkit & Rkhunter

> **Ringkasan:** Pemindai keamanan lokal otomatis. Keduanya bekerja mendeteksi keberadaan *rootkit*—program berbahaya yang memodifikasi biner inti sistem operasi Linux untuk menyembunyikan jejak peretas—dengan cara membandingkan biner sistem saat ini dengan tanda tangan *rootkit* yang sudah dikenal.

### Part I: chkrootkit (Check Rootkit)

#### 📄 Deskripsi Perangkat Lunak
`chkrootkit` adalah sebuah instrumen pendeteksi *rootkit* lokal berbasis *open-source* yang dikembangkan dalam bentuk skrip POSIX shell dan rangkaian program pembantu berbahasa C.

Secara arsitektural, `chkrootkit` dirancang sebagai alat pemindai ringan yang memeriksa sistem operasi berbasis Unix/Linux secara langsung untuk mencari tanda tangan (*signatures*) *rootkit* yang dikenal serta anomali perilaku pada biner inti sistem (seperti `ls`, `ps`, `netstat`, `sshd`, dan `login`). Dengan menyertakan utilitas pembantu C (seperti `chkproc`, `chklastlog`, `chkwtmp`, dan `ifpromisc`), `chkrootkit` mampu melakukan pemeriksaan silang (*cross-view detection*) antara keluaran utilitas sistem standar dengan struktur data internal kernel maupun sistem berkas `/proc`.

### Part II: rkhunter (Rootkit Hunter)

#### 📄 Deskripsi Perangkat Lunak
`rkhunter` (Rootkit Hunter) adalah sebuah instrumen pemindai keamanan komprehensif berbasis *open-source* yang dikembangkan menggunakan skrip POSIX shell untuk mendeteksi *rootkit*, *backdoor*, eksploitasi lokal, serta anomali konfigurasi pada sistem operasi berbasis Unix/Linux.

Secara arsitektural, berbeda dengan `chkrootkit` yang lebih berfokus pada pencocokan tanda tangan *rootkit* spesifik dan pemeriksaan silang `/proc`, `rkhunter` menerapkan pendekatan berlapis yang memadukan verifikasi integritas kriptografis berbasis basis data acuan (*baseline hash database*), pencocokan tanda tangan biner, analisis izin berkas, serta audit penguatan konfigurasi sistem (*system hardening checks*).

#### 🎯 Kegunaan (Use Case) dalam Keamanan Siber
* **Triase Forensik Sistem Langsung & Pemantauan Baseline (*Live Incident Triage & FIM*):** Membantu investigator tanggap insiden dan administrator sistem melakukan pemeriksaan cepat pada peladen Linux yang dicurigai disusupi, serta mendeteksi modifikasi tidak sah pada biner sistem operasi.
* **Deteksi Manipulasi Log, Antarmuka Jaringan, & Backdoor:** Mengidentifikasi kartu jaringan dalam mode *promiscuous*, pembersihan berkas log (`wtmp`/`lastlog`), direktori tersembunyi (`/dev/.udev` atau `/tmp`), modul kernel mencurigakan, serta pembajakan pustaka bersama via `LD_PRELOAD`.
* **Audit Konfigurasi Keamanan SSH dan Akun Sistem:** Berfungsi sebagai instrumen audit preventif untuk memeriksa kelemahan konfigurasi layanan kritis (seperti pengizinan login langsung root via SSH) serta mendeteksi akun pengguna dengan hak akses UID 0 tersembunyi.

#### ⚙️ Fungsi Teknis (Functionality)
* **Deteksi Proses Tersembunyi via Cross-View (`chkproc`):** Membandingkan daftar proses yang dilaporkan oleh perintah `ps` dengan entri direktori numerik pada `/proc` serta pemanggilan sistem `kill(pid, 0)` dan `getpriority()`. Perbedaan hasil secara langsung mengindikasikan keberadaan *rootkit*.
* **Verifikasi Hash Kriptografis & Manajer Paket (`rkhunter --propupd`):** Menyimpan rekaman atribut berkas (hash SHA-256, ukuran, izin akses) ke dalam basis data lokal (`rkhunter.dat`), serta memvalidasi keaslian biner secara langsung terhadap basis data manajer paket resmi distribusi (`dpkg` / `rpm`).
* **Pemeriksaan Antarmuka Mode Promiscuous & Log Wipers (`ifpromisc`, `chkwtmp`):** Menginspeksi bendera antarmuka jaringan melalui `ioctl()` untuk mendeteksi *packet sniffer*, serta memeriksa struktur biner log acct (`/var/log/wtmp`) dari blok byte nol yang mencurigakan.
* **Pemindaian Tanda Tangan Rootkit LKM & User-Space:** Memeriksa ratusan variasi *rootkit* (seperti Diamorphine, Reptile, Azazel, Adore, T0rn), mendeteksi modifikasi pada `/etc/ld.so.preload`, serta mencari biner yang telah dihapus namun masih berjalan di memori (*deleted binaries running in memory*).
* **Dukungan Analisis Luring / Root Alternatif (`-r` / `--rootdir`):** Mengarahkan seluruh pemeriksaan ke titik kait (*mount point*) media bukti eksternal yang terhubung ke mesin Kali Linux yang bersih, memastikan analisis akurat tanpa terpengaruh oleh *rootkit* aktif pada kernel target.

#### 📊 Perbandingan Singkat & Kesimpulan Akademis: chkrootkit vs rkhunter

| Aspek Komparasi | chkrootkit (`chkrootkit`) | rkhunter (`rkhunter`) |
| :--- | :--- | :--- |
| **Fokus Metodologi** | Pemeriksaan silang proses (`chkproc`), log biner (`wtmp`), dan tanda tangan rootkit. | Verifikasi hash baseline (`dpkg`/`rpm`), direktori tersembunyi, dan audit hardening. |
| **Ketergantungan Baseline** | Tidak memerlukan database baseline awal (bisa langsung dijalankan saat insiden). | Sangat optimal apabila database properti (`--propupd`) dibuat saat sistem bersih. |
| **Cakupan Audit** | Berfokus secara spesifik pada indikator keberadaan *rootkit* dan *sniffer*. | Lebih luas (rootkit, trojan, `LD_PRELOAD` hijacking, celah SSH/akun). |

#### 💻 Contoh Penggunaan CLI

```bash
# 1. Menjalankan pemindaian chkrootkit standar
sudo chkrootkit

# 2. Pemindaian chkrootkit pada disk target yang di-mount (Offline Mode)
sudo chkrootkit -r /mnt/target_system/

# 3. Memperbarui basis data properti/hash baseline rkhunter (pada sistem bersih)
sudo rkhunter --propupd

# 4. Menjalankan pemindaian rkhunter secara otomatis tanpa interaksi prompt
sudo rkhunter --check --sk

# 5. Menampilkan hanya hasil peringatan (Warnings Only) dari log rkhunter
sudo rkhunter --rwo
```

> 💡 **Kesimpulan Akademis:**
> Secara akademis, `chkrootkit` dan `rkhunter` mendemonstrasikan prinsip dasar pendeteksian intrusi berbasis integritas dan konsistensi pandangan sistem (*cross-view & integrity-based detection*), di mana upaya penyerang untuk menyembunyikan artefak pada satu lapisan abstraksi sistem operasi sering kali meninggalkan inkonsistensi pada lapisan lainnya. Keberadaannya di dalam kategori Forensics Kali Linux menegaskan dua kaidah penting: pertama, perekaman *baseline* kriptografis secara berkala sangat krusial untuk mendeteksi perubahan biner secara dini; dan kedua, karena *kernel-mode rootkit* tingkat lanjut mampu memanipulasi pemanggilan sistem (*syscalls*) pada sistem yang sedang berjalan, validasi forensik yang paling akurat harus selalu dilakukan secara luring (*offline*) dari media booting yang bersih atau melalui akuisisi citra memori RAM.
