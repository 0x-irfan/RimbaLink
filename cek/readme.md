# 🌲 RIMBALINK

**SISTEM PERINGATAN DINI KARHUTLA DAN PEMANTAUAN LAHAN BERBASIS NODE LORA SURYA DENGAN VERIFIKASI KAMERA, PETA RISIKO, DAN PRAKIRAAN**

## 📑 Daftar Isi

 1. Deskripsi Proyek

 2. [Latar Belakang dan Permasalahan](#-2-latar-belakang-dan-permasalahan)

 3. [Tujuan Sistem](#-3-tujuan-sistem)

 4. [Fitur Utama & Status Pengembangan](#-4-fitur-utama--status-pengembangan)

 5. [Arsitektur Sistem](#-5-arsitektur-sistem)

 6. [Spesifikasi Hardware](#-6-spesifikasi-hardware)

 7. [Protokol Komunikasi LoRa](#-7-protokol-komunikasi-lora)

 8. [Sistem RTC dan Penyimpanan Offline](#-8-sistem-rtc-dan-penyimpanan-offline)

 9. [Arsitektur API Gateway](#-9-arsitektur-api-gateway)

10. [Struktur Folder Backend](#-10-struktur-folder-backend)

11. [Struktur Database](#-11-struktur-database)

12. [Endpoint REST API](#-12-endpoint-rest-api)

13. [Format Data JSON](#-13-format-data-json)

14. [Panduan Instalasi API Gateway](#-14-panduan-instalasi-api-gateway)

15. [Panduan Deployment cPanel](#-15-panduan-deployment-cpanel)

16. [Arsitektur Dashboard](#-16-arsitektur-dashboard)

17. [Monitoring Node dan Gateway](#-17-monitoring-node-dan-gateway)

18. [Grafik dan Analisis Data](#-18-grafik-dan-analisis-data)

19. [Peta Interaktif](#-19-peta-interaktif)

20. [Analisis Fresnel Zone](#-20-analisis-fresnel-zone)

21. [Rekomendasi Titik Penempatan Node](#-21-rekomendasi-titik-penempatan-node)

22. [Verifikasi Kamera](#-22-verifikasi-kamera)

23. [Peta Risiko dan Peringatan Dini](#-23-peta-risiko-dan-peringatan-dini)

24. [Keamanan Sistem](#-24-keamanan-sistem)

25. [Panduan Implementasi dan Pengujian](#-25-panduan-implementasi-dan-pengujian)

26. [Troubleshooting](#-26-troubleshooting)

27. [Roadmap Pengembangan](#-27-roadmap-pengembangan)

28. [Kontribusi](#-28-kontribusi)

29. [Lisensi](#-29-lisensi)

30. [Kredit dan Referensi](#-30-kredit-dan-referensi)

## 📖 1. Deskripsi Proyek

**RimbaLink** adalah sistem Internet of Things (IoT) berbasis jaringan sensor nirkabel **LoRa (Long Range)** bertenaga surya yang dirancang untuk pemantauan kondisi mikroklimat lahan dan deteksi dini potensi kebakaran hutan dan lahan (karhutla).

Sistem terdiri dari beberapa **LoRa Sensor Node** (skala awal **1–5 node**, dirancang modular agar dapat diperluas untuk puluhan hingga ratusan node melalui multi-gateway) yang mengirimkan data lingkungan, kondisi daya, status aktuator relay, serta metadata hasil verifikasi kamera setiap **60 detik (1 menit)** menuju **ESP32 LoRa Gateway**.

Untuk menjamin keandalan data di wilayah terpencil yang rentan gangguan konektivitas, gateway dilengkapi modul **RTC (Real-Time Clock)** dan penyimpanan lokal **microSD Card** berbasis mekanisme *store-and-forward*. Ketika koneksi internet melalui **Ethernet W5500** terputus, seluruh paket telemetri tetap tercatat dengan stempel waktu yang konsisten di microSD, lalu disinkronkan secara otomatis dalam mode *batch* ke **HTTPS REST API (PHP 8.x & MySQL/MariaDB)** di hosting **cPanel** segera setelah koneksi pulih. Data kemudian divisualisasikan pada **Dashboard Web RimbaLink** yang mencakup pemantauan *real-time*, grafik historis teragregasi, kualitas link radio LoRa, peta interaktif, simulasi *Fresnel Zone*, verifikasi kamera, peta risiko, dan peringatan dini.

> **Catatan Transparansi Status Dokumentasi:** Dokumen ini berperan sebagai **cetak biru arsitektur teknis dan panduan implementasi utama (master blueprint)** RimbaLink. Fitur-fitur pada repositori diklasifikasikan secara eksplisit menggunakan label `IMPLEMENTED`, `IN DEVELOPMENT`, atau `PLANNED` pada bagian [Fitur Utama](#-4-fitur-utama--status-pengembangan). Jangan mengasumsikan modul tertentu telah lulus uji lapangan sebelum statusnya diperbarui disertai bukti pengujian.

## 🔍 2. Latar Belakang dan Permasalahan

Kebakaran hutan dan lahan (karhutla) sering terjadi di wilayah yang minim infrastruktur listrik maupun jaringan seluler yang stabil. Pendekatan pemantauan konvensional menghadapi sejumlah kendala teknis:

1. **Keterbatasan Catu Daya di Lapangan:** Lahan rawan karhutla umumnya tidak memiliki akses listrik PLN sehingga perangkat pemantau memerlukan sumber energi mandiri (panel surya dan baterai isi ulang).

2. **Konektivitas Internet Tidak Stabil:** Wilayah pinggiran hutan atau lahan gambut kerap mengalami *intermittent connectivity*. Sistem IoT tanpa *buffering* lokal akan kehilangan data krusial saat jaringan terputus.

3. **Risiko Interferensi & Paket Liar pada Kanal Radio:** Penggunaan spektrum bebas lisensi (ISM/SRD) berisiko menerima paket dari perangkat pihak ketiga apabila tidak ada validasi identitas jaringan, integritas, dan autentikasi paket.

4. **Tingginya False Alarm dari Sensor Tunggal:** Deteksi kebakaran yang hanya mengandalkan satu sensor (misalnya sensor gas/asap semata) rawan memicu alarm palsu akibat perubahan cuaca, kelembapan, atau *drift* sensor tanpa kalibrasi dan verifikasi visual.

5. **Penempatan Node Tanpa Perencanaan Link Radio (RF):** Penempatan sensor secara sembarangan tanpa mempertimbangkan topografi, elevasi, dan *Fresnel Zone* menyebabkan tingginya *packet loss* dan penurunan jangkauan LoRa.

## 🎯 3. Tujuan Sistem

1. Merancang sistem pemantauan parameter lingkungan lahan (suhu udara, kelembapan udara, suhu tanah, kelembapan tanah, dan indikasi gas/asap) serta manajemen energi surya secara berkala setiap 1 menit.

2. Menyediakan protokol komunikasi LoRa yang terstruktur, tervalidasi, dan mendukung autentikasi pesan untuk mencegah masuknya paket dari perangkat asing.

3. Membangun mekanisme *store-and-forward* berbasis RTC dan microSD pada ESP32 Gateway agar tidak terjadi kehilangan data maupun duplikasi data saat terjadi pemutusan koneksi internet.

4. Membangun arsitektur backend REST API berbasis PHP 8.x dan MySQL/MariaDB yang aman, idempoten, dan realistis untuk di-*deploy* pada lingkungan hosting cPanel ber-SSL/HTTPS.

5. Menyediakan rancangan dashboard web responsif dengan visualisasi data teragregasi, pemantauan kualitas link radio (RSSI/SNR per node), peta GIS interaktif, analisis *Fresnel Zone* berbasis data elevasi, integrasi verifikasi kamera, serta peringatan dini multi-parameter.

## ✨ 4. Fitur Utama & Status Pengembangan

Keterangan label status:

* 🟢 **`IMPLEMENTED`** — Sudah selesai dibangun, terdokumentasi, dan tersedia pada repositori.

* 🟡 **`IN DEVELOPMENT`** — Sedang dalam tahap perancangan detail, pengkodean, atau integrasi awal.

* 🔵 **`PLANNED`** — Direncanakan pada fase pengembangan lanjutan setelah modul inti stabil.

| Domain | Fitur | Status | Keterangan | 
| ----- | ----- | ----- | ----- | 
| **Dokumentasi** | Cetak Biru Arsitektur, Skema DB, Kontrak API & Protokol LoRa | 🟢 `IMPLEMENTED` | Terdefinisi pada `README.md` ini sebagai acuan pengembangan | 
| **Hardware Node** | Node Sensor Lingkungan + Panel Surya + Baterai 2S 18650 + Relay 4-Ch | 🟡 `IN DEVELOPMENT` | Integrasi pembacaan sensor, ADC tegangan baterai, dan LoRa SX1276 | 
| **Hardware Gateway** | ESP32 + SX1276 + Ethernet W5500 + RTC + microSD | 🟡 `IN DEVELOPMENT` | Integrasi bus SPI bersama, manajemen daya, dan TLS over Ethernet | 
| **Protokol LoRa** | Framing (`MAGIC_WORD`, `NETWORK_ID`, `NODE_ID`), CRC & HMAC | 🟡 `IN DEVELOPMENT` | Spesifikasi paket selesai; implementasi firmware & uji lapangan berjalan | 
| **Offline Storage** | Mekanisme *Store-and-Forward* JSONL pada microSD + NTP Sync | 🟡 `IN DEVELOPMENT` | Termasuk `event_id` idempoten dan pemulihan saat restart | 
| **Backend API** | REST API PHP 8.x (Telemetry Batch, Heartbeat, Sync, CRUD Node) | 🟡 `IN DEVELOPMENT` | Siap dibangun mengikuti struktur direktori & SQL di dokumen ini | 
| **Database** | Skema Relasional MySQL/MariaDB (10 Tabel + Indeks + Constraint) | 🟢 `IMPLEMENTED` | Skrip DDL `CREATE TABLE` siap dieksekusi di MySQL/MariaDB | 
| **Dashboard Core** | Overview, Monitoring Node & Gateway, CRUD Node (*Soft Delete*) | 🟡 `IN DEVELOPMENT` | UI responsif desktop/mobile dengan konversi waktu `Asia/Jakarta` | 
| **Analitik & Grafik** | Grafik Historis Multi-Rentang & Agregasi (`AVG`, `MIN`, `MAX`) | 🟡 `IN DEVELOPMENT` | Memisahkan kueri data mentah 1 menit dan data agregasi | 
| **Monitoring Link** | Kualitas Link LoRa per Node (RSSI, SNR, Estimasi *Packet Loss*) | 🟡 `IN DEVELOPMENT` | Terpisah dari metrik latensi API dan status Ethernet gateway | 
| **GIS & RF Planning** | Peta Interaktif, Simulasi *Fresnel Zone* Berbasis DEM & Rekomendasi Titik | 🔵 `PLANNED` | Membutuhkan integrasi sumber data elevasi (DEMNAS/SRTM) | 
| **Verifikasi Visual** | Modul Integrasi Hasil Verifikasi Kamera (Api/Asap & Bukti Visual) | 🔵 `PLANNED` | Penempatan kamera berstatus `[CONFIG_TBD]` | 
| **Peringatan Dini** | Fusion Multi-Sensor (NORMAL/WASPADA/SIAGA/BAHAYA) & Peta Risiko | 🔵 `PLANNED` | Ambang batas bersifat konfigurabel dan memerlukan kalibrasi lapangan | 
| **Prakiraan** | Modul Prakiraan Risiko Karhutla Berbasis Data Historis/Cuaca | 🔵 `PLANNED` | Bergantung pada ketersediaan data historis & integrasi API cuaca | 

## 🏗️ 5. Arsitektur Sistem

Alur data *end-to-end* pada RimbaLink bergerak dari node sensor di lapangan menuju gateway melalui modul radio LoRa SX1276, disimpan sementara pada microSD dengan stempel waktu RTC, kemudian dikirim via Ethernet W5500 melalui protokol HTTPS ke API Server PHP/MySQL untuk disajikan pada Dashboard Web.

### Diagram 1: Arsitektur Keseluruhan Sistem RimbaLink

```
flowchart TB
    subgraph FIELD["Lapisan Lapangan (LoRa Sensor Nodes - Panel Surya)"]
        N1["Node 001\nSX1276 + Sensor Lingkungan\nRelay 4-Ch + 2S 18650"]
        N2["Node 002\nSX1276 + Sensor Lingkungan\nRelay 4-Ch + 2S 18650"]
        NN["Node 00N (Skalabel)\nSX1276 + Sensor Lingkungan\nRelay 4-Ch + 2S 18650"]
        CAM["Unit Verifikasi Kamera\n[Arsitektur: CONFIG_TBD]"]
    end

    subgraph GW["Lapisan Edge (ESP32 LoRa Gateway)"]
        LORA_RX["Modul LoRa SX1276\n(Penerima RF & RSSI/SNR)"]
        ESP["ESP32 DevKit\nValidasi Protokol, HMAC, & Antrean"]
        RTC["Modul RTC\nStempel Waktu UTC + NTP Sync"]
        SD["Modul microSD Card\nStore-and-Forward JSONL"]
        ETH["Ethernet W5500\nTLS Client (HTTPS)"]
        PWR_GW["Catu Daya / Panel Surya Gateway\n& Monitoring 2S 18650 (Opsional)"]
    end

    subgraph CLOUD["Lapisan Server (cPanel Hosting - HTTPS)"]
        API["HTTPS REST API\napi.<domain-anda.com>\nPHP 8.x + Auth API Key/HMAC"]
        DB[("MySQL / MariaDB\n10 Tabel Relasional")]
        GIS_EXT["Sumber Data Elevasi (DEM)\n& Referensi Gambar Kamera"]
    end

    subgraph CLIENT["Lapisan Aplikasi (Dashboard Web)"]
        WEB["Dashboard Web RimbaLink\nDesktop & Mobile Responsive"]
        MOD1["Monitoring Node, Gateway & Link LoRa"]
        MOD2["Grafik Agregasi, Peta GIS & Fresnel Zone"]
        MOD3["Verifikasi Kamera, Peta Risiko & Alert"]
    end

    N1 -- "LoRa RF (60s)" --> LORA_RX
    N2 -- "LoRa RF (60s)" --> LORA_RX
    NN -- "LoRa RF (60s)" --> LORA_RX
    CAM -. "Metadata / Referensi Visual\n[Sesuai Opsi Arsitektur]" .-> ESP
    CAM -. "Upload Bukti Visual\n[Jika Server-Side/Direct]" .-> API

    LORA_RX --> ESP
    RTC <--> ESP
    ESP <--> SD
    PWR_GW --> ESP
    ESP <--> ETH

    ETH -- "HTTPS JSON (Batch & Heartbeat)" --> API
    API <--> DB
    API <--> GIS_EXT

    API <-- "REST JSON (Auth Token/Session)" --> WEB
    WEB --> MOD1
    WEB --> MOD2
    WEB --> MOD3

```

## ⚙️ 6. Spesifikasi Hardware

> **Penting:** Konfigurasi pin GPIO spesifik, tipe persis sensor (misalnya SHT31 vs BME280, DS18B20, MQ-2/MQ-135, DS3231, dll.), serta frekuensi kerja akhir ditandai sebagai placeholder (`<TBD>`) hingga ditetapkan pada skematik rangkaian final dan diverifikasi terhadap regulasi lokal.

### A. LoRa Sensor Node

Setiap node dirancang untuk beroperasi secara mandiri menggunakan energi surya dengan siklus transmisi telemetri setiap **60 detik (1 menit)**.

| No | Komponen / Subsistem | Fungsi Teknis | Catatan Konfigurasi & Kalibrasi | 
| ----- | ----- | ----- | ----- | 
| 1 | **MCU Node** | Pengendali akuisisi sensor, kontrol relay, & *framing* LoRa | `<MCU_NODE_TBD>` (mis. ESP32 / STM32 / ATmega sesuai kebutuhan daya & memori kriptografi) | 
| 2 | **Modul LoRa SX1276** | Transceiver radio jarak jauh berdaya rendah | Frekuensi harus mematuhi regulasi Indonesia (lihat catatan regulasi di bawah) | 
| 3 | **Sensor Suhu Udara** | Mengukur temperatur ambient ($^{\circ}\text{C}$) | `<TIPE_SENSOR_TBD>` — pastikan dilengkapi *radiation shield* di lapangan | 
| 4 | **Sensor Kelembapan Udara** | Mengukur kelembapan relatif udara ($\%\text{RH}$) | `<TIPE_SENSOR_TBD>` — rentang valid $0\text{--}100\%$ | 
| 5 | **Sensor Suhu Tanah** | Mengukur temperatur media tanah ($^{\circ}\text{C}$) | `<TIPE_SENSOR_TBD>` — probe tahan air (*waterproof*) | 
| 6 | **Sensor Kelembapan Tanah** | Mengukur kadar air tanah ($\%$) | `<TIPE_SENSOR_TBD>` — disarankan tipe kapasitif agar tahan korosi; wajib kalibrasi titik kering/basah | 
| 7 | **Sensor Gas / Asap** | Mendeteksi anomali konsentrasi gas/asap indikasi pembakaran | `<TIPE_SENSOR_TBD>` — **Jangan laporkan dalam satuan `ppm` tanpa kalibrasi kurva laboratorium/gas standar.** Gunakan nilai mentah (`gas_raw`, `smoke_raw`) terlebih dahulu | 
| 8 | **Panel Surya & Pemantau Status** | Sumber pengisian daya & deteksi status `STANDBY` / `CHARGING` | Spesifikasi daya panel `<WATT_PANEL_TBD>`; status dibaca dari pin charger/deteksi arus atau tegangan input | 
| 9 | **Baterai 2S 18650 & Sensor Tegangan** | Penyimpanan energi utama ($\approx 6{,}0\text{--}8{,}4\text{ V}$) | Gunakan *voltage divider* presisi + proteksi BMS 2S; kalibrasi rasio ADC sebelum operasi | 
| 10 | **Relay 4 Channel** | Kontrol perangkat eksternal (mis. pompa/sirine/katup/aktuator) | Mencatat status aktual `ON`/`OFF` (`true`/`false`) untuk `relay_1` s.d. `relay_4` | 
| 11 | **Integrasi Verifikasi Kamera** | Membawa/menghubungkan metadata verifikasi visual api/asap | Penempatan kamera berstatus **`[CONFIG_TBD]`** (lihat sub-bagian Arsitektur Kamera di bawah) | 

### B. ESP32 LoRa Gateway

Gateway berfungsi sebagai penghubung antara jaringan radio LoRa lokal dan API Server melalui jaringan kabel Ethernet.

| No | Komponen | Fungsi Teknis | Catatan Implementasi | 
| ----- | ----- | ----- | ----- | 
| 1 | **ESP32 DevKit** | Prosesor utama gateway (dual-core, mendukung mbedTLS/BearSSL) | `<Varian_ESP32_TBD>` (perhatikan kapasitas RAM/Flash untuk buffer TLS & antrean JSONL) | 
| 2 | **Modul LoRa SX1276** | Penerima paket dari seluruh node terdaftar & pembaca RSSI/SNR | Gunakan antarmuka SPI dengan pin `CS` terpisah dan *level shifting* / catu $3{,}3\text{ V}$ stabil | 
| 3 | **Ethernet W5500** | Konektivitas jaringan kabel LAN/Internet ber-TCP/IP hardware | Berbagi bus SPI atau menggunakan bus SPI terpisah (VSPI/HSPI) dari SX1276 & SD Card | 
| 4 | **Modul RTC** | Menjaga waktu UTC akurat saat internet/daya utama terputus | Disarankan tipe berpresisi tinggi dengan baterai koin cadangan (mis. `<TIPE_RTC_TBD>`, I2C) | 
| 5 | **Modul microSD Card** | Penyimpanan lokal *store-and-forward* saat offline | Format FAT32; perhatikan isolasi pin `CS` SPI dan stabilitas arus puncak saat *write* | 
| 6 | **Panel Surya & Baterai 2S 18650 (Opsional)** | Catu daya cadangan / mandiri untuk gateway | Jika diterapkan, gateway mencatat `solar_status` (`STANDBY`/`CHARGING`) dan `battery_voltage_v` | 

#### Parameter yang Wajib Dicatat Gateway

 1. `gateway_id` (Identitas unik gateway).

 2. Status `ONLINE` / `OFFLINE` (dihitung di sisi server dari `last_heartbeat_at` dan batas timeout).

 3. `uptime_seconds` (lama waktu aktif sejak boot terakhir).

 4. Status koneksi Ethernet (`UP`/`DOWN`, IP lokal, status link fisik).

 5. Status RTC (`OK`/`INVALID_TIME`/`BATTERY_LOW`/`ERROR`) dan sumber sinkronisasi waktu (`NTP`/`RTC`).

 6. Status microSD Card (`MOUNTED`/`ERROR`/`FULL`).

 7. Kapasitas total & sisa penyimpanan microSD (`sd_total_mb`, `sd_free_mb`).

 8. Jumlah antrean record yang belum tersinkronisasi (`offline_queue_count`).

 9. Status sinkronisasi data (`IDLE`/`SYNCING`/`ERROR`/`OFFLINE_BUFFERING`).

10. Status panel surya gateway (`STANDBY`/`CHARGING`/`NOT_APPLICABLE`).

11. Tegangan baterai 2S 18650 gateway (`battery_voltage_v`, bernilai `null` jika tidak diterapkan).

12. Latensi komunikasi HTTPS ke API Server (`api_latency_ms`) — **dicatat sebagai metrik jaringan IP/API tersendiri**.

13. **RSSI (dBm) dan SNR (dB) LoRa** — **dicatat per masing-masing node sebagai metrik kualitas link radio LoRa**, bukan sebagai metrik kualitas Ethernet gateway.

### C. Pemetaan Pin Hardware (Placeholder Konfigurasi)

Karena papan pengembangan dan modul yang digunakan dapat berbeda antar prototipe, seluruh penugasan pin harus didefinisikan pada file `include/pins_config.h` pada firmware dan diverifikasi terhadap *datasheet* sebelum penyalaan daya:

```
// Contoh struktur placeholder konfigurasi pin (ISI SESUAI SKEMATIK AKTUAL)
#define LORA_SCK_PIN      <GPIO_TBD>
#define LORA_MISO_PIN     <GPIO_TBD>
#define LORA_MOSI_PIN     <GPIO_TBD>
#define LORA_CS_PIN       <GPIO_TBD>
#define LORA_RST_PIN      <GPIO_TBD>
#define LORA_DIO0_PIN     <GPIO_TBD>

#define W5500_CS_PIN      <GPIO_TBD>
#define W5500_RST_PIN     <GPIO_TBD>
#define SD_CS_PIN         <GPIO_TBD>

#define RTC_SDA_PIN       <GPIO_TBD>
#define RTC_SCL_PIN       <GPIO_TBD>
#define VBAT_ADC_PIN      <GPIO_TBD>
#define SOLAR_STAT_PIN    <GPIO_TBD>

```

> **Peringatan Bus SPI pada ESP32:** Modul **SX1276**, **W5500**, dan **microSD Card** sama-sama menggunakan antarmuka SPI. Gunakan dua bus SPI terpisah pada ESP32 (`HSPI` dan `VSPI`) atau pastikan manajemen `CS` (*Chip Select*) dan *SPI transaction locking* (mutex FreeRTOS) diterapkan secara ketat agar tidak terjadi tabrakan bus (*bus contention*).

### D. Opsi Arsitektur Verifikasi Kamera (`[CONFIG_TBD]`)

RimbaLink **tidak mengasumsikan** bahwa modul kamera wajib terpasang di dalam LoRa Node atau di dalam ESP32 Gateway, mengingat keterbatasan *bandwidth* LoRa untuk pengiriman gambar utuh dan anggaran daya baterai node. Keputusan penempatan kamera merupakan **konfigurasi arsitektur yang harus ditentukan pada tahap implementasi**:

| Opsi | Penempatan Kamera | Mekanisme Kerja | Kelebihan | Keterbatasan | 
| ----- | ----- | ----- | ----- | ----- | 
| **Opsi A** | **Edge AI Camera Terpisah** (dekat area node) | Kamera memproses deteksi api/asap secara lokal lalu mengirim *ringkasan hasil* (status, flag deteksi, *confidence*, ID gambar) ke node/gateway, atau mengunggah gambar via jalur jaringan terpisah | Tidak membebani *airtime* LoRa dengan transfer citra mentah | Membutuhkan catu daya lebih besar untuk unit kamera | 
| **Opsi B** | **Kamera pada Lokasi Gateway / Menara Pantau** | Kamera terhubung ke jaringan lokal gateway (Ethernet/IP Camera) untuk memverifikasi sektor lahan secara visual saat sensor memicu peringatan | Mendukung unggah gambar langsung via koneksi internet gateway | Jarak pandang terbatas pada *line-of-sight* dari lokasi gateway | 
| **Opsi C** | **Kamera Mandiri Langsung ke API Server** | Kamera (seluler/Wi-Fi/satelit/link khusus) mengirim foto atau hasil inferensi langsung ke server, lalu dikaitkan ke `node_id` berdasarkan lokasi/sektor | Decoupled dari keterbatasan LoRa dan daya sensor node | Memerlukan konektivitas tambahan pada titik kamera | 

### E. Catatan Regulasi Frekuensi dan Daya Pancar LoRa di Indonesia

Penggunaan frekuensi radio dan daya pancar (EIRP) pada modul SX1276 **tidak boleh ditetapkan secara sembarangan**. Di Indonesia, alokasi spektrum untuk perangkat *Low Power Wide Area Network* (LPWAN) / *Short Range Device* (SRD) diatur oleh **Kementerian Komunikasi dan Digital (Komdigi / sebelumnya Kominfo) — Direktorat Jenderal Infrastruktur Digital (DJID / sebelumnya SDPPI)**, yang secara umum merujuk pada pita regional **AS923-2** (kisaran $920\text{--}923\text{ MHz}$) dengan batasan daya pancar, *dwell time*/*duty cycle*, dan sertifikasi perangkat tertentu.

* **Tindakan Wajib Sebelum Uji Lapangan:** Tetapkan parameter `LORA_FREQUENCY_HZ`, `LORA_TX_POWER_DBM`, `LORA_BANDWIDTH`, dan `LORA_SF` sebagai konfigurasi terpisah (`<FREQ_REGULASI_INDONESIA_TBD>`) dan verifikasi terhadap regulasi resmi terbaru yang berlaku di Indonesia sebelum mengoperasikan pemancar di luar ruang.

## 📡 7. Protokol Komunikasi LoRa

Untuk mencegah gateway memproses paket acak dari jaringan LoRa lain atau perangkat ilegal, RimbaLink menerapkan protokol paket biner terstruktur (*compact binary frame*) yang dilengkapi identifikasi jaringan, deteksi duplikasi, pemeriksaan integritas, serta autentikasi pesan.

> ⚠️ **Peringatan Keamanan Penting:** Field `MAGIC_WORD` dan `NETWORK_ID` **hanya berfungsi sebagai penanda framing dan penyaring awal paket** agar gateway cepat membuang paket dari jaringan lain. Keduanya **bukan mekanisme keamanan kriptografis**. Paket LoRa **tidak otomatis aman** hanya karena memiliki `MAGIC_WORD`. Perlindungan terhadap pemalsuan (*spoofing*) dan penyadapan wajib menggunakan **HMAC (Message Authentication Code)** dan/atau **enkripsi payload**.

### A. Struktur Paket LoRa RimbaLink

| Urutan | Nama Field | Tipe Data | Panjang (Byte) | Fungsi | Contoh Nilai (Ilustrasi) | 
| ----- | ----- | ----- | ----- | ----- | ----- | 
| 1 | `MAGIC_WORD` | `char[9]` / ASCII | 9 | Penanda awal paket milik protokol RimbaLink | `"RIMBALINK"` (`0x52 49 4D ...`) | 
| 2 | `NETWORK_ID` | `uint16_t` | 2 | Identitas unik jaringan/klaster RimbaLink | `0xA101` (`<NETWORK_ID_TBD>`) | 
| 3 | `PROTOCOL_VERSION` | `uint8_t` | 1 | Versi format protokol untuk kompatibilitas | `0x01` (v1) | 
| 4 | `DESTINATION_GATEWAY_ID` | `uint16_t` | 2 | ID numerik gateway tujuan yang berhak menerima | `0x0001` (map ke `RIMBA-GW-001`) | 
| 5 | `NODE_ID` | `uint16_t` | 2 | ID numerik unik node pengirim | `0x0001` (map ke `RIMBA-NODE-001`) | 
| 6 | `PACKET_SEQUENCE_NUMBER` | `uint32_t` | 4 | Nomor urut monoton untuk deteksi duplikasi & *packet loss* | `1001` | 
| 7 | `MESSAGE_TYPE` | `uint8_t` | 1 | Jenis pesan (`0x01`: Telemetri, `0x02`: Alert, `0x03`: ACK, `0x04`: Status) | `0x01` | 
| 8 | `NODE_TIMESTAMP` | `uint32_t` | 4 | Unix timestamp UTC dari node (`0` jika node tidak memiliki RTC) | `0` atau `1795429800` | 
| 9 | `PAYLOAD_LENGTH` | `uint8_t` | 1 | Panjang field `SENSOR_PAYLOAD` dalam byte | `28` | 
| 10 | `SENSOR_PAYLOAD` | `uint8_t[N]` | Variabel ($N$) | Data sensor terkompresi (skala *fixed-point*), status relay, baterai, & flag kamera | *Binary struct terenkripsi/plaintext* | 
| 11 | `AUTH_TAG` | `uint8_t[8]` | 8 | Tag autentikasi pesan (HMAC-SHA256 dipotong 64-bit / CMAC) | `0x8F3A1C92...` | 
| 12 | `CRC16` | `uint16_t` | 2 | Pemeriksaan integritas bit terhadap korupsi kanal radio (CRC-16/CCITT) | `0x4B2E` | 

#### Rincian Contoh Struktur `SENSOR_PAYLOAD` (Fixed-Point untuk Efisiensi Airtime LoRa)

* `air_temperature_x100` (`int16_t`, 2 byte): nilai $^{\circ}\text{C} \times 100$.

* `air_humidity_x100` (`uint16_t`, 2 byte): nilai $\%\text{RH} \times 100$.

* `soil_temperature_x100` (`int16_t`, 2 byte): nilai $^{\circ}\text{C} \times 100$.

* `soil_moisture_x100` (`uint16_t`, 2 byte): nilai $\% \times 100$.

* `gas_raw` (`uint16_t`, 2 byte): nilai mentah ADC/sensor gas.

* `smoke_raw` (`uint16_t`, 2 byte): nilai mentah ADC/sensor asap.

* `battery_mv` (`uint16_t`, 2 byte): tegangan baterai dalam milivolt (mV).

* `flags_status` (`uint8_t`, 1 byte): *bitmask* status panel surya (bit 0: `0=STANDBY`, `1=CHARGING`), status relay 1–4 (bit 1–4), flag validitas sensor.

* `camera_summary` (`uint8_t[4]`, 4 byte): *bitmask* status verifikasi kamera, flag `fire_detected`, `confidence_pct` ($0\text{--}100$, `255` jika N/A), dan indeks referensi lokal.

### B. Aturan Validasi, Autentikasi, dan Enkripsi

1. **Identitas Unik:** Setiap node memiliki `NODE_ID` unik, setiap gateway memiliki `GATEWAY_ID` unik, dan setiap instalasi jaringan memiliki `NETWORK_ID` tersendiri.

2. **Penyaringan Awal:** Gateway wajib memeriksa `CRC16` terlebih dahulu untuk membuang paket rusak, lalu memeriksa `MAGIC_WORD == "RIMBALINK"`, `NETWORK_ID`, dan `DESTINATION_GATEWAY_ID`.

3. **Daftar Putih (*Allowlist*) Node:** Gateway mencocokkan `NODE_ID` dengan daftar node terdaftar yang ditautkan ke gateway tersebut. Paket dari `NODE_ID` asing ditolak dan dicatat pada log keamanan lokal.

4. **Deteksi Duplikasi & Anti-Replay:** Gateway menyimpan nilai `last_sequence_number` per `NODE_ID` (dipersistensikan secara berkala ke NVS/SD). Paket dengan `PACKET_SEQUENCE_NUMBER <= last_sequence_number` ditandai sebagai duplikat atau indikasi *replay attack* (dengan penanganan khusus bila node baru mengalami *reboot* dan memulai ulang counter yang divalidasi dengan *boot counter* atau *session epoch*).

5. **Autentikasi Pesan (HMAC):** `AUTH_TAG` dihitung dari seluruh header (`MAGIC_WORD` hingga `PAYLOAD_LENGTH`) ditambah `SENSOR_PAYLOAD` menggunakan **HMAC-SHA256** (dipotong 8 atau 16 byte sesuai batas *payload* SF LoRa) dengan kunci unik per node (`NODE_PSK`).

6. **Opsi Enkripsi Payload & Manajemen Kunci:**

   * **Enkripsi (Direkomendasikan untuk produksi):** Enkripsi `SENSOR_PAYLOAD` menggunakan **AES-128-CTR** atau **ChaCha20** dengan *nonce* yang dibentuk dari kombinasi `NETWORK_ID + NODE_ID + BOOT_EPOCH + PACKET_SEQUENCE_NUMBER` (skema *Encrypt-then-MAC*).

   * **Manajemen Kunci:** Setiap node memiliki kunci rahasia unik (*per-device key*), **bukan satu kunci global untuk seluruh node**, sehingga jika satu node hilang/dicuri di hutan, kunci node lain tidak ikut kompromi. Kunci disimpan di NVS/flash terproteksi (*Flash Encryption* bila MCU mendukung) dan dapat dicabut (*revoke*) dari daftar putih gateway.

### Diagram 2: Flowchart Proses Pengiriman dan Validasi Data dari Node ke Gateway

```
flowchart TD
    A["Node Bangun / Timer 60 Detik"] --> B["Baca Sensor Lingkungan, Baterai, Panel Surya, Relay & Info Kamera"]
    B --> C["Naikkan PACKET_SEQUENCE_NUMBER"]
    C --> D["Susun Header: MAGIC_WORD, NETWORK_ID, PROTOCOL_VERSION, DST_GW, NODE_ID"]
    D --> E["Enkripsi SENSOR_PAYLOAD (Opsional: AES-128-CTR)"]
    E --> F["Hitung AUTH_TAG (HMAC-SHA256) & CRC16"]
    F --> G["Kirim Paket via LoRa SX1276"]

    G --> H["Gateway Terima Paket + Catat RSSI & SNR"]
    H --> I{"CRC16 Valid?"}
    I -- "Tidak" --> X1["Buang Paket & Tambah Counter CRC Error"]
    I -- "Ya" --> J{"MAGIC_WORD & NETWORK_ID Cocok?"}
    J -- "Tidak" --> X2["Abaikan Paket (Bukan Jaringan RimbaLink)"]
    J -- "Ya" --> K{"DESTINATION_GATEWAY_ID Cocok?"}
    K -- "Tidak" --> X3["Abaikan Paket (Untuk Gateway Lain)"]
    K -- "Ya" --> L{"NODE_ID Terdaftar di Allowlist?"}
    L -- "Tidak" --> X4["Tolak & Catat Log Keamanan"]
    L -- "Ya" --> M{"AUTH_TAG (HMAC) Valid?"}
    M -- "Tidak" --> X5["Tolak Paket (Autentikasi Gagal)"]
    M -- "Ya" --> N{"Sequence Number Baru / Bukan Replay?"}
    N -- "Duplikat" --> X6["Catat Duplikat & Jangan Simpan Ulang"]
    N -- "Valid" --> O["Dekripsi Payload (Jika Terenkripsi)"]
    O --> P["Beri Timestamp RTC (UTC) & Buat event_id Unik"]
    P --> Q["Simpan ke microSD Card (Store-and-Forward)"]

```

## ⏱️ 8. Sistem RTC dan Penyimpanan Offline

Mekanisme *store-and-forward* memastikan gateway tetap berfungsi penuh saat jaringan Ethernet/internet terputus. Prinsip dasarnya adalah **Write-Ahead Logging (WAL)**: setiap data valid yang diterima dari LoRa **wajib ditulis ke microSD terlebih dahulu** sebelum dikirim ke API Server.

### A. Standar Timestamp dan Sinkronisasi Waktu

Seluruh sistem menyimpan dan mempertukarkan waktu dalam format **UTC ISO-8601** (`YYYY-MM-DDTHH:MM:SSZ`) atau **Unix Timestamp UTC** secara konsisten. Konversi ke zona waktu lokal **`Asia/Jakarta` (WIB / UTC+7)** hanya dilakukan pada lapisan presentasi (Dashboard Web dan laporan pengguna).

| Field Waktu | Sumber | Fungsi | 
| ----- | ----- | ----- | 
| `node_sampled_at` | Node sensor (jika punya RTC) | Waktu aktual pembacaan sensor di node; bernilai `null` jika node tidak memiliki RTC | 
| `gateway_received_at` | RTC Gateway | Waktu paket LoRa diterima & divalidasi gateway (acuan utama urutan kronologis lapangan) | 
| `uploaded_at` | Gateway saat transmisi HTTPS | Waktu gateway mengirimkan batch ke server | 
| `server_received_at` | Jam Server Backend (UTC) | Waktu data berhasil diterima dan disimpan di MySQL | 

* **Sinkronisasi NTP:** Saat koneksi internet melalui W5500 aktif, gateway melakukan sinkronisasi waktu periodik (misalnya setiap 6 jam) ke server NTP dan memperbarui modul RTC jika selisih (*drift*) melebihi ambang batas (misalnya $> 2\text{ detik}$).

### B. Penanganan Kasus Tepi (*Edge Cases*) RTC & Sinkronisasi

1. **RTC Kehilangan Daya / Waktu Belum Valid (mis. kembali ke tahun 2000/1970):**

   * Gateway memeriksa flag *Oscillator Stop Flag (OSF)* RTC atau validitas tahun (`year < 2025`).

   * Jika waktu belum valid, gateway **tidak memalsukan waktu**; data tetap disimpan dengan `time_source = "UNSYNCED_UPTIME"`, `rtc_valid = false`, serta mencatat `gateway_boot_id` dan `uptime_ms`. Setelah internet/NTP kembali, server atau gateway dapat menandai data tersebut untuk rekonsiliasi waktu berdasarkan offset *uptime*.

2. **Perbedaan Timestamp (*Clock Skew*):**

   * Server membandingkan `gateway_received_at` terhadap `server_received_at`. Data offline yang datang terlambat (*late-arriving data*) tetap diterima dan disimpan sesuai `gateway_received_at` aslinya, sementara `server_received_at` mencatat waktu kedatangan aktual di server.

3. **Pencegahan Duplikasi Data (`event_id` Idempoten):**

   * Setiap record yang ditulis ke microSD diberi `event_id` unik yang bersifat deterministik atau UUID v4/v7 (misalnya format `GW001-N001-SEQ1001-1795429800` atau UUID unik yang disimpan permanen di baris JSONL).

   * Database memiliki `UNIQUE KEY` pada `event_id` serta kombinasi `(gateway_id, node_id, sequence_number, gateway_received_at)`. Jika gateway mengirim ulang batch yang sama karena ACK gagal diterima, server mengenali duplikasi dan mengembalikan status `ALREADY_EXISTS` tanpa menggandakan baris data.

4. **Gateway Restart di Tengah Proses Sinkronisasi:**

   * Gateway hanya memperbarui penanda (*checkpoint* / status tersinkronisasi) **setelah menerima HTTP `200 OK` dengan daftar `ack_event_ids` yang valid** dari API Server. Jika gateway mati di tengah pengiriman, record tetap berstatus belum tersinkronisasi dan akan dikirim ulang secara aman (idempoten) saat boot berikutnya.

### C. Struktur Penyimpanan microSD Card (Format JSON Lines / JSONL)

Format **JSON Lines (`.jsonl`)** dipilih karena setiap baris adalah satu objek JSON utuh (*append-only*), mudah dibaca per baris tanpa memuat seluruh file ke RAM ESP32 yang terbatas, dan lebih tahan terhadap korupsi parsial dibandingkan satu file JSON array besar.

```
/rimbalink/
├── config/
│   └── allowlist.json              # Cache daftar node terdaftar & parameter gateway
├── queue/
│   └── 2026/
│       └── 11/
│           ├── 2026-11-23.jsonl    # Data telemetri harian (append-only)
│           └── 2026-11-24.jsonl
├── state/
│   ├── sync_cursor.json            # Posisi file & byte offset terakhir yang sudah di-ACK server
│   ├── seq_tracker.json            # Sequence number terakhir per node
│   └── retry_failed.jsonl          # Record yang gagal validasi permanen (dead-letter queue)
├── synced/
│   └── 2026-11-22.jsonl            # Arsip file yang 100% barisnya telah menerima ACK (rotasi FIFO)
└── logs/
    └── gateway_2026-11-23.log      # Log diagnostik sistem & keamanan

```

Contoh isi `sync_cursor.json`:

```
{
  "active_file": "/rimbalink/queue/2026/11/2026-11-23.jsonl",
  "last_acked_offset": 14820,
  "pending_records": 42,
  "last_sync_utc": "2026-11-23T10:25:00Z"
}

```

> **Manajemen Kapasitas SD Card:** Saat sisa ruang microSD turun di bawah ambang konfigurabel (mis. $< 15\%$), gateway secara otomatis menghapus file arsip tertua di folder `/rimbalink/synced/` terlebih dahulu (*FIFO rotation*) tanpa menghapus file di `/rimbalink/queue/` yang belum ter-ACK.

### Diagram 3: Flowchart Penyimpanan Data Offline pada microSD Card

```
flowchart TD
    A["Paket LoRa Tervalidasi Masuk ke Antrean Gateway"] --> B["Cek Status RTC"]
    B --> C{"RTC Valid?"}
    C -- "Ya" --> D["Set gateway_received_at = RTC UTC\ntime_source = RTC"]
    C -- "Tidak" --> E["Set gateway_received_at = null\ntime_source = UNSYNCED_UPTIME\nCatat uptime_ms"]
    D --> F["Bangun Record JSONL + event_id Unik"]
    E --> F
    F --> G{"microSD Terpasang & Ruang Cukup?"}
    G -- "Ruang Menipis" --> H["Hapus Arsip Tertua di /synced/ (FIFO)"]
    H --> I["Append 1 Baris ke /queue/YYYY/MM/YYYY-MM-DD.jsonl + fsync"]
    G -- "Ya" --> I
    G -- "SD Error / Penuh Total" --> J["Catat Error SD ke Status Heartbeat\nSimpan di Ring Buffer RAM Sementara"]
    I --> K["Tambah Counter pending_records"]
    K --> L["Picu Task Sinkronisasi Jika Ethernet & Internet Tersedia"]

```

### Diagram 4: Flowchart Sinkronisasi Data ke API Server

```
flowchart TD
    A["Task Sinkronisasi Berjalan Periodik / Saat Ada Data"] --> B{"Link Ethernet W5500 UP & Internet Tersedia?"}
    B -- "Tidak" --> C["Tetap Mode OFFLINE_BUFFERING\nTunggu Siklus Berikutnya (Backoff)"]
    B -- "Ya" --> D["Baca sync_cursor.json & Ambil Maks N Record (mis. 10-25 Record) dari /queue/"]
    D --> E{"Ada Data Pending?"}
    E -- "Tidak" --> F["Status = IDLE"]
    E -- "Ya" --> G["Susun Payload Batch JSON + Hitung Signature HMAC API"]
    G --> H["Kirim HTTPS POST /api/v1/sync/upload\nUkur api_latency_ms"]
    H --> I{"Respons HTTP 200 OK & ACK JSON Valid?"}
    I -- "Timeout / 5xx / TLS Error" --> J["Jangan Ubah Cursor\nTerapkan Exponential Backoff & Coba Lagi"]
    I -- "4xx (Payload Cacat Permanen)" --> K["Pindahkan Record Bermasalah ke retry_failed.jsonl\nCatat Log Error"]
    I -- "Ya (200 OK)" --> L["Verifikasi ack_event_ids (Inserted + Duplicate)"]
    L --> M["Update sync_cursor.json (Offset Baru) secara Atomik (temp + rename)"]
    M --> N{"Seluruh Baris File Harian Selesai?"}
    N -- "Ya" --> O["Pindahkan File ke /synced/"]
    N -- "Tidak" --> D
    O --> D

```

## 🌐 9. Arsitektur API Gateway

Backend RimbaLink dibangun sebagai **HTTPS REST API** menggunakan **PHP 8.x** dan **MySQL/MariaDB** pada lingkungan **cPanel**, ditempatkan pada subdomain khusus seperti `https://api.<domain-anda.com>`, sedangkan Dashboard Web berada pada domain utama (`https://<domain-anda.com>`) atau subdomain terpisah (`https://dashboard.<domain-anda.com>`).

### Pertimbangan Teknis Penting: Ethernet W5500 + ESP32 + HTTPS/TLS

Terdapat kesalahan umum dalam implementasi IoT ESP32 dengan Ethernet W5500 yang **wajib dihindari**:

1. **Jangan menggunakan `WiFiClientSecure` untuk W5500:** `WiFiClientSecure` pada framework Arduino-ESP32 bergantung pada *socket stack* Wi-Fi/lwIP dan **tidak kompatibel secara langsung** dengan library `EthernetClient` (W5500 SPI).

2. **Opsi Implementasi TLS yang Tepat untuk ESP32 + W5500:**

   * **Opsi 1 (Direkomendasikan — ESP-IDF Native `esp_eth` + W5500):** Menggunakan driver native ESP-IDF untuk W5500 yang terintegrasi langsung dengan stack TCP/IP **lwIP** ESP32. Dengan cara ini, `esp_http_client` dan **mbedTLS** (dengan akselerasi hardware AES/SHA ESP32) dapat berjalan secara native melalui Ethernet.

   * **Opsi 2 (Arduino Framework + `SSLClient` BearSSL):** Membungkus objek `EthernetClient` (W5500) menggunakan pustaka **`SSLClient`** (berbasis BearSSL) untuk menyediakan lapisan TLS 1.2 di atas antarmuka `Client` Arduino. Perhatikan konsumsi RAM/stack dan ukuran *trust anchor* (Root CA).

   * **Opsi 3 (Arsitektur Lokal Proxy — Jika Terkendala Memori):** Jika firmware tidak mampu memuat TLS langsung, gunakan *local gateway/reverse proxy* di jaringan lokal yang aman; namun pada arsitektur standar RimbaLink, **Opsi 1 atau Opsi 2 adalah target utama**.

3. **Verifikasi Sertifikat Wajib Aktif pada Produksi:** **Jangan menonaktifkan verifikasi sertifikat TLS** (seperti `setInsecure()` atau melewati validasi Root CA) pada lingkungan produksi karena membuka celah serangan *Man-in-the-Middle (MitM)*. Tanamkan **Root CA Certificate** (misalnya ISRG Root X1 untuk Let's Encrypt / AutoSSL cPanel) pada firmware dan sediakan mekanisme pembaruan sertifikat.

### Diagram 6: Diagram Komunikasi API Gateway

```
sequenceDiagram
    autonumber
    participant GW as ESP32 Gateway (W5500 + TLS)
    participant API as PHP 8.x REST API (cPanel HTTPS)
    participant AUTH as Middleware Auth (API Key + HMAC)
    participant DB as MySQL / MariaDB

    Note over GW,API: 1. Pengiriman Heartbeat Berkala
    GW->>API: POST /api/v1/gateways/heartbeat (Headers: X-Gateway-Id, X-Api-Key, X-Timestamp, X-Signature)
    API->>AUTH: Validasi TLS, API Key, Timestamp Window & HMAC-SHA256
    AUTH-->>API: Valid (Gateway Terdaftar)
    API->>DB: INSERT gateway_heartbeats & UPDATE gateways.last_heartbeat_at
    DB-->>API: OK
    API-->>GW: 200 OK {server_time_utc, status: "ACK"}

    Note over GW,API: 2. Pengiriman Batch / Sinkronisasi Offline dari SD Card
    GW->>API: POST /api/v1/sync/upload (Batch JSON + event_id per record)
    API->>AUTH: Validasi Identitas Gateway, HMAC & Rate Limit
    AUTH-->>API: Valid
    API->>DB: BEGIN TRANSACTION
    loop Untuk setiap record dalam batch
        API->>DB: Validasi node_id terikat pada gateway_id
        API->>DB: INSERT IGNORE / ON DUPLICATE KEY (Idempoten via event_id)
        API->>DB: Simpan lora_link_metrics, relay_states, camera_verifications
    end
    API->>DB: INSERT sync_logs & Evaluasi Rule Peringatan Dini (alerts)
    API->>DB: COMMIT
    DB-->>API: Ringkasan (inserted, duplicates, rejected)
    API-->>GW: 200 OK {ack_event_ids: [...], duplicate_event_ids: [...]}
    Note over GW: Gateway perbarui sync_cursor.json di microSD setelah menerima ACK valid

```

## 📁 10. Struktur Folder Backend

Pada hosting **cPanel**, **dilarang keras** meletakkan file kredensial database, API secret, atau kode inti aplikasi di dalam direktori publik (*Document Root*). Arahkan *Document Root* subdomain `api.<domain-anda.com>` **hanya ke folder `api/public/`**.

```
/home/<cpanel-user>/
├── rimbalink-api/                      # Di luar public_html (tidak dapat diakses via browser)
│   ├── public/                         # <-- SET SEBAGAI DOCUMENT ROOT subdomain api.<domain-anda.com>
│   │   ├── index.php                   # Front controller tunggal (router REST API)
│   │   ├── .htaccess                   # Rewrite rule HTTPS, routing ke index.php, proteksi dotfiles
│   │   └── api/                        # (Opsional jika memilih endpoint berbasis file eksplisit)
│   │       └── v1/
│   │           ├── telemetry/
│   │           │   └── batch.php
│   │           ├── gateways/
│   │           │   ├── heartbeat.php
│   │           │   └── status.php
│   │           ├── nodes/
│   │           │   ├── index.php
│   │           │   └── detail.php
│   │           └── sync/
│   │               └── upload.php
│   ├── src/
│   │   ├── Controllers/
│   │   │   ├── GatewayController.php
│   │   │   ├── NodeController.php
│   │   │   ├── TelemetryController.php
│   │   │   ├── SyncController.php
│   │   │   ├── AlertController.php
│   │   │   └── MapFresnelController.php
│   │   ├── Middleware/
│   │   │   ├── GatewayAuthMiddleware.php   # Verifikasi API Key + HMAC + Anti-Replay Timestamp
│   │   │   ├── UserAuthMiddleware.php      # Autentikasi Dashboard (JWT / Session + RBAC)
│   │   │   ├── RateLimitMiddleware.php
│   │   │   └── CorsMiddleware.php
│   │   ├── Services/
│   │   │   ├── TelemetryIngestService.php
│   │   │   ├── SyncBatchService.php
│   │   │   ├── EarlyWarningService.php
│   │   │   └── FresnelAnalysisService.php
│   │   ├── Repositories/
│   │   │   ├── GatewayRepository.php
│   │   │   ├── NodeRepository.php
│   │   │   └── TelemetryRepository.php
│   │   ├── Validators/
│   │   │   ├── TelemetryValidator.php
│   │   │   └── NodeValidator.php
│   │   └── Helpers/
│   │       ├── ResponseHelper.php
│   │       └── Logger.php
│   ├── private/                        # Kredensial rahasia (permission chmod 600 / 750)
│   │   ├── config.php                  # Memuat konfigurasi dari .env / variabel rahasia
│   │   ├── .env                        # JANGAN PERNAH COMMIT KE GIT (.gitignore)
│   │   └── database.php                # Koneksi PDO Singleton (prepared statements wajib)
│   ├── database/
│   │   └── schema.sql                  # Skrip DDL MySQL lengkap
│   ├── logs/                           # Log aplikasi & audit keamanan (permission terbatas)
│   │   ├── api_error.log
│   │   └── security_audit.log
│   └── composer.json                   # Autoload PSR-4 & dependensi PHP 8.x
└── public_html/                        # Domain utama untuk Dashboard Web RimbaLink

```

> **Catatan Implementasi Routing:** Pendekatan terbaik di PHP 8.x pada cPanel adalah menggunakan **Front Controller** (`public/index.php` + `.htaccess` rewrite) yang merutekan seluruh permintaan `/api/v1/*` ke `src/Controllers/*`. Jika tim memilih struktur file eksplisit di `public/api/v1/...`, setiap file `.php` cukup berperan sebagai *thin entry point* yang memanggil `bootstrap.php` dan Controller di `../../src/`.

## 🗄️ 11. Struktur Database

Database menggunakan **MySQL 8.0+ / MariaDB 10.6+** dengan engine **InnoDB**, charset **`utf8mb4`**, dan penyimpanan seluruh kolom waktu (`DATETIME(3)` atau `TIMESTAMP`) dalam **UTC**.

### Diagram 5: Entity Relationship Diagram (ERD) RimbaLink

```
erDiagram
    gateways ||--o{ nodes : "mengelola"
    gateways ||--o{ telemetry : "menerima"
    gateways ||--o{ gateway_heartbeats : "melaporkan"
    gateways ||--o{ sync_logs : "mensinkronkan"
    gateways ||--o{ lora_link_metrics : "mengukur"
    nodes ||--o{ telemetry : "menghasilkan"
    nodes ||--o{ lora_link_metrics : "memiliki"
    nodes ||--o{ camera_verifications : "diverifikasi_oleh"
    nodes ||--o{ relay_states : "mengontrol"
    nodes ||--o{ alerts : "memicu"
    telemetry ||--o| lora_link_metrics : "menyertakan"
    telemetry ||--o| relay_states : "merekam"
    telemetry ||--o| camera_verifications : "menautkan"
    users ||--o{ alerts : "menangani"

    gateways {
        bigint id PK
        varchar gateway_code UK
        varchar network_id
        varchar name
        decimal latitude
        decimal longitude
        decimal antenna_height_m
        varchar api_key_hash
        varchar hmac_secret_enc
        int heartbeat_timeout_sec
        datetime last_heartbeat_at
        tinyint is_active
    }

    nodes {
        bigint id PK
        varchar node_code UK
        bigint gateway_id FK
        varchar name
        decimal latitude
        decimal longitude
        decimal ground_elevation_m
        decimal antenna_height_m
        int expected_interval_sec
        datetime last_seen_at
        datetime deleted_at
    }

    telemetry {
        bigint id PK
        varchar event_id UK
        bigint gateway_id FK
        bigint node_id FK
        bigint sequence_number
        datetime node_sampled_at
        datetime gateway_received_at
        datetime server_received_at
        varchar time_source
        decimal air_temperature_c
        decimal air_humidity_pct
        decimal soil_temperature_c
        decimal soil_moisture_pct
        int gas_raw
        int smoke_raw
        varchar solar_status
        decimal battery_voltage_v
    }

    gateway_heartbeats {
        bigint id PK
        bigint gateway_id FK
        datetime reported_at
        bigint uptime_seconds
        varchar ethernet_status
        varchar rtc_status
        varchar sd_status
        int sd_free_mb
        int offline_queue_count
        varchar sync_status
        varchar solar_status
        decimal battery_voltage_v
        int api_latency_ms
    }

    lora_link_metrics {
        bigint id PK
        bigint telemetry_id FK
        bigint gateway_id FK
        bigint node_id FK
        bigint sequence_number
        smallint rssi_dbm
        decimal snr_db
        datetime recorded_at
    }

    camera_verifications {
        bigint id PK
        bigint telemetry_id FK
        bigint node_id FK
        varchar verification_status
        tinyint fire_detected
        decimal confidence_score
        varchar image_reference
        datetime verified_at
    }

    relay_states {
        bigint id PK
        bigint telemetry_id FK
        bigint node_id FK
        tinyint relay_1
        tinyint relay_2
        tinyint relay_3
        tinyint relay_4
        datetime recorded_at
    }

    sync_logs {
        bigint id PK
        bigint gateway_id FK
        varchar batch_id UK
        int total_records
        int inserted_records
        int duplicate_records
        int rejected_records
        int api_latency_ms
        varchar status
        datetime synced_at
    }

    alerts {
        bigint id PK
        bigint node_id FK
        bigint telemetry_id FK
        varchar severity_level
        varchar category
        text message
        varchar status
        bigint acknowledged_by FK
        datetime triggered_at
    }

    users {
        bigint id PK
        varchar username UK
        varchar email UK
        varchar password_hash
        varchar role
        tinyint is_active
        datetime last_login_at
    }

```

### Ringkasan Tabel & Relasi

| No | Nama Tabel | Primary Key | Kunci Unik / Foreign Key Utama | Fungsi Utama | 
| ----- | ----- | ----- | ----- | ----- | 
| 1 | `gateways` | `id` | `UK(gateway_code)` | Master data gateway ESP32, koordinat, tinggi antena, hash API key, & waktu heartbeat terakhir | 
| 2 | `nodes` | `id` | `UK(node_code)`, `FK(gateway_id -> gateways.id)` | Master data node sensor LoRa, penugasan gateway, lokasi GIS, & *soft delete* (`deleted_at`) | 
| 3 | `telemetry` | `id` | `UK(event_id)`, `FK(gateway_id)`, `FK(node_id)` | Penyimpanan data sensor 1 menit, tegangan baterai, status surya, & multi-timestamp | 
| 4 | `gateway_heartbeats` | `id` | `FK(gateway_id)` | Riwayat kesehatan gateway (uptime, Ethernet, RTC, SD Card, antrean offline, latensi API) | 
| 5 | `lora_link_metrics` | `id` | `FK(telemetry_id)`, `FK(gateway_id)`, `FK(node_id)` | Riwayat kualitas link radio LoRa (RSSI & SNR) khusus per pasangan node-gateway | 
| 6 | `camera_verifications` | `id` | `FK(telemetry_id)`, `FK(node_id)` | Riwayat hasil verifikasi kamera (status, deteksi api/asap, *confidence*, referensi gambar) | 
| 7 | `relay_states` | `id` | `FK(telemetry_id)`, `FK(node_id)` | Riwayat status ON/OFF relay 1–4 pada setiap node | 
| 8 | `sync_logs` | `id` | `UK(batch_id)`, `FK(gateway_id)` | Audit setiap operasi sinkronisasi batch offline dari SD Card ke API Server | 
| 9 | `alerts` | `id` | `FK(node_id)`, `FK(telemetry_id)`, `FK(acknowledged_by -> users.id)` | Catatan peringatan dini (`NORMAL`, `WASPADA`, `SIAGA`, `BAHAYA`) dan tindak lanjut operator | 
| 10 | `users` | `id` | `UK(username)`, `UK(email)` | Manajemen pengguna Dashboard Web, *password hash*, dan kontrol akses berbasis peran (RBAC) | 

### Skrip SQL Lengkap (`database/schema.sql`)

Skrip di bawah dapat langsung dijalankan pada MySQL 8.0+ / MariaDB 10.6+ setelah membuat database target:

```
-- ============================================================================
-- RIMBALINK DATABASE SCHEMA (MySQL 8.0+ / MariaDB 10.6+)
-- Standar Waktu: Seluruh kolom DATETIME menyimpan waktu dalam UTC.
-- ============================================================================

SET NAMES utf8mb4;
SET time_zone = '+00:00';

-- 1. Tabel gateways
CREATE TABLE IF NOT EXISTS gateways (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    gateway_code VARCHAR(64) NOT NULL COMMENT 'Identitas unik publik gateway, mis. RIMBA-GW-001',
    network_id VARCHAR(16) NOT NULL COMMENT 'Identitas jaringan LoRa RimbaLink',
    name VARCHAR(120) NOT NULL COMMENT 'Nama deskriptif lokasi gateway',
    description TEXT NULL,
    latitude DECIMAL(10, 7) NULL COMMENT 'Koordinat lintang WGS84',
    longitude DECIMAL(10, 7) NULL COMMENT 'Koordinat bujur WGS84',
    ground_elevation_m DECIMAL(8, 2) NULL COMMENT 'Elevasi permukaan tanah (mdpl) dari DEM/survei',
    antenna_height_m DECIMAL(6, 2) NULL COMMENT 'Tinggi tiang antena di atas permukaan tanah (meter)',
    api_key_hash VARCHAR(255) NOT NULL COMMENT 'Hash SHA-256 / Argon2 dari API Key gateway',
    hmac_secret_enc VARCHAR(512) NOT NULL COMMENT 'Secret HMAC gateway terenkripsi di sisi server',
    heartbeat_timeout_sec INT UNSIGNED NOT NULL DEFAULT 180 COMMENT 'Batas timeout untuk status ONLINE/OFFLINE',
    last_heartbeat_at DATETIME NULL COMMENT 'Waktu UTC heartbeat terakhir diterima',
    is_active TINYINT(1) NOT NULL DEFAULT 1,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_gateways_code (gateway_code),
    INDEX idx_gateways_network (network_id),
    INDEX idx_gateways_heartbeat (last_heartbeat_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 2. Tabel nodes (mendukung soft delete via deleted_at)
CREATE TABLE IF NOT EXISTS nodes (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    node_code VARCHAR(64) NOT NULL COMMENT 'Identitas unik node, mis. RIMBA-NODE-001',
    gateway_id BIGINT UNSIGNED NOT NULL COMMENT 'Gateway yang berhak menerima paket node ini',
    name VARCHAR(120) NOT NULL COMMENT 'Nama lokasi/sektor node',
    description TEXT NULL,
    latitude DECIMAL(10, 7) NULL COMMENT 'Koordinat lintang WGS84',
    longitude DECIMAL(10, 7) NULL COMMENT 'Koordinat bujur WGS84',
    ground_elevation_m DECIMAL(8, 2) NULL COMMENT 'Elevasi tanah (mdpl)',
    antenna_height_m DECIMAL(6, 2) NULL COMMENT 'Tinggi antena node di atas tanah (meter)',
    installation_notes TEXT NULL COMMENT 'Catatan pemasangan, orientasi panel surya, dll.',
    expected_interval_sec INT UNSIGNED NOT NULL DEFAULT 60 COMMENT 'Interval kirim standar (60 detik)',
    offline_timeout_sec INT UNSIGNED NOT NULL DEFAULT 300 COMMENT 'Batas timeout status node ONLINE/OFFLINE',
    last_seen_at DATETIME NULL COMMENT 'Waktu UTC paket valid terakhir diterima',
    is_active TINYINT(1) NOT NULL DEFAULT 1,
    deleted_at DATETIME NULL COMMENT 'Soft delete timestamp agar data historis tidak hilang',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_nodes_code (node_code),
    INDEX idx_nodes_gateway (gateway_id),
    INDEX idx_nodes_deleted (deleted_at),
    INDEX idx_nodes_last_seen (last_seen_at),
    CONSTRAINT fk_nodes_gateway FOREIGN KEY (gateway_id)
        REFERENCES gateways(id) ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 3. Tabel telemetry
CREATE TABLE IF NOT EXISTS telemetry (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    event_id VARCHAR(100) NOT NULL COMMENT 'ID unik record dari gateway untuk menjamin idempotency',
    gateway_id BIGINT UNSIGNED NOT NULL,
    node_id BIGINT UNSIGNED NOT NULL,
    sequence_number BIGINT UNSIGNED NOT NULL COMMENT 'Nomor urut paket LoRa dari node',
    node_sampled_at DATETIME NULL COMMENT 'Waktu UTC pengukuran di node (NULL jika tanpa RTC node)',
    gateway_received_at DATETIME NULL COMMENT 'Waktu UTC diterima gateway via RTC',
    uploaded_at DATETIME NULL COMMENT 'Waktu UTC pengiriman oleh gateway',
    server_received_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT 'Waktu UTC disimpan server',
    time_source ENUM('NODE_RTC', 'RTC', 'NTP', 'UNSYNCED_UPTIME') NOT NULL DEFAULT 'RTC',
    gateway_uptime_ms BIGINT UNSIGNED NULL COMMENT 'Uptime gateway saat menerima paket (bantuan rekonsiliasi jika RTC invalid)',
    air_temperature_c DECIMAL(5, 2) NULL COMMENT 'Suhu udara (derajat Celsius)',
    air_humidity_pct DECIMAL(5, 2) NULL COMMENT 'Kelembapan udara (%RH)',
    soil_temperature_c DECIMAL(5, 2) NULL COMMENT 'Suhu tanah (derajat Celsius)',
    soil_moisture_pct DECIMAL(5, 2) NULL COMMENT 'Kelembapan tanah (%)',
    gas_raw INT UNSIGNED NULL COMMENT 'Nilai mentah sensor gas (bukan ppm sebelum kalibrasi)',
    smoke_raw INT UNSIGNED NULL COMMENT 'Nilai mentah indikasi asap (bukan ppm sebelum kalibrasi)',
    solar_status ENUM('STANDBY', 'CHARGING', 'UNKNOWN') NOT NULL DEFAULT 'UNKNOWN',
    battery_voltage_v DECIMAL(5, 3) NULL COMMENT 'Tegangan baterai 2S 18650 (Volt)',
    is_offline_synced TINYINT(1) NOT NULL DEFAULT 0 COMMENT '1 jika berasal dari sinkronisasi antrean SD Card',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_telemetry_event_id (event_id),
    UNIQUE KEY uk_telemetry_dedup (gateway_id, node_id, sequence_number, gateway_received_at),
    INDEX idx_telemetry_node_time (node_id, gateway_received_at),
    INDEX idx_telemetry_gw_time (gateway_id, gateway_received_at),
    INDEX idx_telemetry_server_time (server_received_at),
    CONSTRAINT fk_telemetry_gateway FOREIGN KEY (gateway_id)
        REFERENCES gateways(id) ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT fk_telemetry_node FOREIGN KEY (node_id)
        REFERENCES nodes(id) ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 4. Tabel gateway_heartbeats
CREATE TABLE IF NOT EXISTS gateway_heartbeats (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    gateway_id BIGINT UNSIGNED NOT NULL,
    reported_at DATETIME NOT NULL COMMENT 'Waktu UTC heartbeat dilaporkan gateway',
    server_received_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    uptime_seconds BIGINT UNSIGNED NOT NULL DEFAULT 0,
    ethernet_status ENUM('UP', 'DOWN', 'DEGRADED') NOT NULL DEFAULT 'UP',
    ip_address VARCHAR(45) NULL,
    rtc_status ENUM('OK', 'INVALID_TIME', 'BATTERY_LOW', 'ERROR') NOT NULL DEFAULT 'OK',
    ntp_synced TINYINT(1) NOT NULL DEFAULT 0,
    sd_status ENUM('MOUNTED', 'FULL', 'ERROR', 'NOT_PRESENT') NOT NULL DEFAULT 'MOUNTED',
    sd_total_mb INT UNSIGNED NULL,
    sd_free_mb INT UNSIGNED NULL,
    offline_queue_count INT UNSIGNED NOT NULL DEFAULT 0 COMMENT 'Jumlah record belum tersinkronisasi di SD Card',
    sync_status ENUM('IDLE', 'SYNCING', 'OFFLINE_BUFFERING', 'ERROR') NOT NULL DEFAULT 'IDLE',
    solar_status ENUM('STANDBY', 'CHARGING', 'NOT_APPLICABLE', 'UNKNOWN') NOT NULL DEFAULT 'NOT_APPLICABLE',
    battery_voltage_v DECIMAL(5, 3) NULL COMMENT 'Tegangan baterai 2S 18650 gateway jika diterapkan',
    api_latency_ms INT UNSIGNED NULL COMMENT 'Latensi round-trip HTTPS ke API Server (ms)',
    INDEX idx_hb_gateway_time (gateway_id, reported_at),
    CONSTRAINT fk_hb_gateway FOREIGN KEY (gateway_id)
        REFERENCES gateways(id) ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 5. Tabel lora_link_metrics (Kualitas link radio LoRa per node)
CREATE TABLE IF NOT EXISTS lora_link_metrics (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    telemetry_id BIGINT UNSIGNED NOT NULL,
    gateway_id BIGINT UNSIGNED NOT NULL,
    node_id BIGINT UNSIGNED NOT NULL,
    sequence_number BIGINT UNSIGNED NOT NULL,
    rssi_dbm SMALLINT NOT NULL COMMENT 'Received Signal Strength Indicator (dBm)',
    snr_db DECIMAL(5, 2) NOT NULL COMMENT 'Signal-to-Noise Ratio (dB)',
    recorded_at DATETIME NOT NULL COMMENT 'Waktu UTC penerimaan paket radio',
    UNIQUE KEY uk_lora_telemetry (telemetry_id),
    INDEX idx_lora_node_time (node_id, recorded_at),
    INDEX idx_lora_gw_node (gateway_id, node_id, recorded_at),
    CONSTRAINT fk_lora_telemetry FOREIGN KEY (telemetry_id)
        REFERENCES telemetry(id) ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_lora_gateway FOREIGN KEY (gateway_id)
        REFERENCES gateways(id) ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT fk_lora_node FOREIGN KEY (node_id)
        REFERENCES nodes(id) ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 6. Tabel camera_verifications
CREATE TABLE IF NOT EXISTS camera_verifications (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    telemetry_id BIGINT UNSIGNED NULL COMMENT 'Opsional jika terikat langsung pada record telemetri',
    node_id BIGINT UNSIGNED NOT NULL,
    verification_status ENUM('UNAVAILABLE', 'PENDING', 'VERIFIED', 'FAILED') NOT NULL DEFAULT 'UNAVAILABLE',
    fire_detected TINYINT(1) NULL COMMENT '1=Terdeteksi indikasi api/asap, 0=Tidak, NULL=Tidak tersedia',
    smoke_detected TINYINT(1) NULL,
    confidence_score DECIMAL(5, 4) NULL COMMENT 'Skor confidence model (0.0000 - 1.0000) jika tersedia',
    image_reference VARCHAR(512) NULL COMMENT 'Path/URL/ID referensi bukti visual',
    analysis_notes TEXT NULL COMMENT 'Keterangan hasil analisis otomatis/manual',
    verified_at DATETIME NULL COMMENT 'Waktu UTC proses verifikasi kamera',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_cam_node_time (node_id, verified_at),
    INDEX idx_cam_telemetry (telemetry_id),
    CONSTRAINT fk_cam_telemetry FOREIGN KEY (telemetry_id)
        REFERENCES telemetry(id) ON UPDATE CASCADE ON DELETE SET NULL,
    CONSTRAINT fk_cam_node FOREIGN KEY (node_id)
        REFERENCES nodes(id) ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 7. Tabel relay_states
CREATE TABLE IF NOT EXISTS relay_states (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    telemetry_id BIGINT UNSIGNED NOT NULL,
    node_id BIGINT UNSIGNED NOT NULL,
    relay_1 TINYINT(1) NOT NULL DEFAULT 0 COMMENT '0=OFF, 1=ON',
    relay_2 TINYINT(1) NOT NULL DEFAULT 0,
    relay_3 TINYINT(1) NOT NULL DEFAULT 0,
    relay_4 TINYINT(1) NOT NULL DEFAULT 0,
    recorded_at DATETIME NOT NULL,
    UNIQUE KEY uk_relay_telemetry (telemetry_id),
    INDEX idx_relay_node_time (node_id, recorded_at),
    CONSTRAINT fk_relay_telemetry FOREIGN KEY (telemetry_id)
        REFERENCES telemetry(id) ON UPDATE CASCADE ON DELETE CASCADE,
    CONSTRAINT fk_relay_node FOREIGN KEY (node_id)
        REFERENCES nodes(id) ON UPDATE CASCADE ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 8. Tabel sync_logs
CREATE TABLE IF NOT EXISTS sync_logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    gateway_id BIGINT UNSIGNED NOT NULL,
    batch_id VARCHAR(100) NOT NULL COMMENT 'ID unik batch sinkronisasi dari gateway',
    total_records INT UNSIGNED NOT NULL DEFAULT 0,
    inserted_records INT UNSIGNED NOT NULL DEFAULT 0,
    duplicate_records INT UNSIGNED NOT NULL DEFAULT 0,
    rejected_records INT UNSIGNED NOT NULL DEFAULT 0,
    api_latency_ms INT UNSIGNED NULL,
    status ENUM('SUCCESS', 'PARTIAL', 'FAILED') NOT NULL DEFAULT 'SUCCESS',
    error_summary TEXT NULL,
    synced_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_sync_batch_id (batch_id),
    INDEX idx_sync_gw_time (gateway_id, synced_at),
    CONSTRAINT fk_sync_gateway FOREIGN KEY (gateway_id)
        REFERENCES gateways(id) ON UPDATE CASCADE ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 9. Tabel users (Autentikasi Dashboard & RBAC)
CREATE TABLE IF NOT EXISTS users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(64) NOT NULL,
    email VARCHAR(150) NOT NULL,
    full_name VARCHAR(120) NOT NULL,
    password_hash VARCHAR(255) NOT NULL COMMENT 'Hash Argon2id atau bcrypt (password_hash PHP)',
    role ENUM('ADMIN', 'OPERATOR', 'VIEWER') NOT NULL DEFAULT 'VIEWER',
    is_active TINYINT(1) NOT NULL DEFAULT 1,
    last_login_at DATETIME NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_users_username (username),
    UNIQUE KEY uk_users_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 10. Tabel alerts
CREATE TABLE IF NOT EXISTS alerts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    node_id BIGINT UNSIGNED NOT NULL,
    telemetry_id BIGINT UNSIGNED NULL,
    severity_level ENUM('NORMAL', 'WASPADA', 'SIAGA', 'BAHAYA') NOT NULL,
    category ENUM('FIRE_RISK', 'GAS_SMOKE', 'HIGH_TEMP', 'LOW_BATTERY', 'NODE_OFFLINE', 'GATEWAY_OFFLINE', 'CAMERA_FIRE') NOT NULL,
    title VARCHAR(180) NOT NULL,
    message TEXT NOT NULL,
    status ENUM('OPEN', 'ACKNOWLEDGED', 'RESOLVED') NOT NULL DEFAULT 'OPEN',
    acknowledged_by BIGINT UNSIGNED NULL,
    acknowledged_at DATETIME NULL,
    resolved_at DATETIME NULL,
    triggered_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_alerts_node_status (node_id, status, triggered_at),
    INDEX idx_alerts_severity (severity_level, triggered_at),
    CONSTRAINT fk_alerts_node FOREIGN KEY (node_id)
        REFERENCES nodes(id) ON UPDATE CASCADE ON DELETE RESTRICT,
    CONSTRAINT fk_alerts_telemetry FOREIGN KEY (telemetry_id)
        REFERENCES telemetry(id) ON UPDATE CASCADE ON DELETE SET NULL,
    CONSTRAINT fk_alerts_user FOREIGN KEY (acknowledged_by)
        REFERENCES users(id) ON UPDATE CASCADE ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

## 🔌 12. Endpoint REST API

**Base URL:** `https://api.<domain-anda.com>/api/v1`

### Jenis Autentikasi

* **`Gateway Auth`**: Header wajib `X-Gateway-Id`, `X-Api-Key`, `X-Timestamp` (Unix UTC), dan `X-Signature` (`HMAC-SHA256(method + path + timestamp + raw_body, gateway_hmac_secret)`).

* **`Dashboard Auth`**: Header `Authorization: Bearer <TOKEN>` (JWT/Session token) dengan validasi peran RBAC (`ADMIN`, `OPERATOR`, `VIEWER`).

### Daftar Endpoint Lengkap

#### A. Gateway

| Method | Endpoint | Autentikasi | Fungsi | 
| ----- | ----- | ----- | ----- | 
| `POST` | `/api/v1/gateways/heartbeat` | Gateway Auth | Menerima laporan kesehatan, uptime, Ethernet, RTC, SD Card, baterai, dan latensi gateway | 
| `GET` | `/api/v1/gateways` | Dashboard Auth | Menampilkan daftar seluruh gateway beserta status `ONLINE`/`OFFLINE` | 
| `GET` | `/api/v1/gateways/{gateway_id}` | Dashboard Auth | Menampilkan detail konfigurasi, lokasi, dan ringkasan node pada gateway | 
| `GET` | `/api/v1/gateways/{gateway_id}/status` | Dashboard Auth | Menampilkan status operasional terbaru, antrean offline, dan riwayat heartbeat | 

#### B. Node

| Method | Endpoint | Autentikasi | Fungsi | 
| ----- | ----- | ----- | ----- | 
| `GET` | `/api/v1/nodes` | Dashboard Auth | Daftar node aktif (mendukung filter `gateway_id`, `status`, pagination) | 
| `POST` | `/api/v1/nodes` | Dashboard (`ADMIN`/`OPERATOR`) | Mendaftarkan node baru dan menautkannya ke gateway | 
| `GET` | `/api/v1/nodes/{node_id}` | Dashboard Auth | Detail node, koordinat, tinggi antena, dan status operasional | 
| `PUT` | `/api/v1/nodes/{node_id}` | Dashboard (`ADMIN`/`OPERATOR`) | Memperbarui nama, koordinat, gateway, tinggi antena, atau catatan pemasangan | 
| `DELETE` | `/api/v1/nodes/{node_id}` | Dashboard (`ADMIN`) | Menghapus node secara **soft delete** (`deleted_at = NOW()`) agar data historis utuh | 

#### C. Telemetry

| Method | Endpoint | Autentikasi | Fungsi | 
| ----- | ----- | ----- | ----- | 
| `POST` | `/api/v1/telemetry/batch` | Gateway Auth | Menerima 1 hingga $N$ record telemetri *real-time* dalam satu permintaan | 
| `GET` | `/api/v1/telemetry` | Dashboard Auth | Mengambil data telemetri lintas node dengan filter waktu, pagination, & agregasi | 
| `GET` | `/api/v1/telemetry/{node_id}` | Dashboard Auth | Riwayat telemetri satu node (mendukung \`granularity=raw | 
| `GET` | `/api/v1/telemetry/latest/{node_id}` | Dashboard Auth | Mengambil 1 record telemetri terakhir lengkap dengan relay, kamera, & RSSI/SNR | 

#### D. Sinkronisasi Offline

| Method | Endpoint | Autentikasi | Fungsi | 
| ----- | ----- | ----- | ----- | 
| `POST` | `/api/v1/sync/upload` | Gateway Auth | Mengunggah batch data dari antrean microSD Card dan mengembalikan ACK per `event_id` | 
| `GET` | `/api/v1/sync/status/{gateway_id}` | Dashboard Auth | Menampilkan riwayat sinkronisasi (`sync_logs`) dan sisa antrean offline gateway | 

#### E. Statistik Link LoRa

| Method | Endpoint | Autentikasi | Fungsi | 
| ----- | ----- | ----- | ----- | 
| `GET` | `/api/v1/nodes/{node_id}/link-quality` | Dashboard Auth | Statistik RSSI, SNR, jumlah paket diterima, estimasi *packet loss* (dari gap `sequence_number`), dan status link | 

#### F. Analisis, Peta, dan Peringatan Dini

| Method | Endpoint | Autentikasi | Fungsi | 
| ----- | ----- | ----- | ----- | 
| `GET` | `/api/v1/alerts` | Dashboard Auth | Daftar peringatan dini (filter `severity`, `status`, `node_id`, rentang tanggal) | 
| `GET` | `/api/v1/map/nodes` | Dashboard Auth | Data GeoJSON/JSON posisi gateway, node, status link, jarak, & ringkasan sensor untuk peta | 
| `GET` | `/api/v1/analysis/fresnel` | Dashboard Auth | Menghitung profil *First Fresnel Zone* antara `gateway_id` dan `node_id` (atau koordinat kandidat) dengan status ketersediaan data elevasi DEM | 

### Standar Pagination, Filter Waktu, dan HTTP Status Code

* **Pagination:** `?page=1&per_page=50` (maksimal `per_page=500` untuk mencegah beban memori berlebih).

* **Filter Waktu:** `?from=2026-11-22T00:00:00Z&to=2026-11-23T23:59:59Z` (wajib UTC ISO-8601).

* **Granularitas Grafik:** `?granularity=raw|hourly|daily|weekly|monthly`.

* **HTTP Status Code Standar:**

  * `200 OK`: Permintaan berhasil / batch selesai diproses.

  * `201 Created`: Resource baru (mis. node) berhasil dibuat.

  * `400 Bad Request`: JSON tidak valid atau parameter tidak sesuai format.

  * `401 Unauthorized`: API Key/Token tidak valid atau HMAC tidak cocok.

  * `403 Forbidden`: Peran pengguna tidak mencukupi atau `node_id` tidak terdaftar pada `gateway_id` tersebut.

  * `404 Not Found`: Resource tidak ditemukan.

  * `422 Unprocessable Entity`: Kesalahan validasi rentang nilai input.

  * `429 Too Many Requests`: Melewati batas *rate limit*.

  * `500 Internal Server Error`: Kesalahan internal server (detail sensitif tidak boleh bocor ke klien).

## 📦 13. Format Data JSON

> **Catatan Satuan Sensor:** Nilai pada contoh di bawah bersifat **ilustrasi format struktur data**, bukan hasil pengukuran lapangan. Nilai gas dan asap dicatat sebagai **`gas_raw`** dan **`smoke_raw`** (nilai ADC/indeks mentah) dan **tidak diklaim dalam satuan `ppm`** sebelum adanya kalibrasi resmi.

### 1. Request `POST /api/v1/sync/upload` (dan `POST /api/v1/telemetry/batch`)

```
{
  "batch_id": "SYNC-GW001-20261123T103500Z-001",
  "gateway_id": "RIMBA-GW-001",
  "uploaded_at": "2026-11-23T10:35:00Z",
  "is_offline_sync": true,
  "records": [
    {
      "event_id": "550e8400-e29b-41d4-a716-446655440001",
      "gateway_id": "RIMBA-GW-001",
      "node_id": "RIMBA-NODE-001",
      "sequence_number": 1001,
      "node_sampled_at": null,
      "gateway_received_at": "2026-11-23T10:30:00Z",
      "time_source": "RTC",
      "gateway_uptime_ms": 86420100,
      "air_temperature_c": 32.5,
      "air_humidity_pct": 75.2,
      "soil_temperature_c": 28.4,
      "soil_moisture_pct": 62.5,
      "gas_raw": 350,
      "smoke_raw": 120,
      "solar_status": "CHARGING",
      "battery_voltage_v": 7.82,
      "relay": {
        "relay_1": true,
        "relay_2": false,
        "relay_3": false,
        "relay_4": true
      },
      "camera": {
        "verification_status": "UNAVAILABLE",
        "fire_detected": null,
        "smoke_detected": null,
        "confidence": null,
        "verified_at": null,
        "image_reference": null
      },
      "lora": {
        "rssi_dbm": -85,
        "snr_db": 9.5
      }
    }
  ]
}

```

### 2. Response ACK `POST /api/v1/sync/upload` (`200 OK`)

```
{
  "success": true,
  "server_time_utc": "2026-11-23T10:35:01Z",
  "batch_id": "SYNC-GW001-20261123T103500Z-001",
  "summary": {
    "total": 1,
    "inserted": 1,
    "duplicates": 0,
    "rejected": 0
  },
  "ack_event_ids": [
    "550e8400-e29b-41d4-a716-446655440001"
  ],
  "duplicate_event_ids": [],
  "rejected_records": []
}

```

### 3. Request `POST /api/v1/gateways/heartbeat`

```
{
  "gateway_id": "RIMBA-GW-001",
  "reported_at": "2026-11-23T10:35:05Z",
  "uptime_seconds": 86425,
  "ethernet_status": "UP",
  "ip_address": "192.168.1.50",
  "rtc_status": "OK",
  "ntp_synced": true,
  "sd_status": "MOUNTED",
  "sd_total_mb": 15200,
  "sd_free_mb": 14980,
  "offline_queue_count": 0,
  "sync_status": "IDLE",
  "solar_status": "CHARGING",
  "battery_voltage_v": 8.05,
  "api_latency_ms": 142
}

```

### 4. Format Respons Error Standar (`4xx` / `5xx`)

```
{
  "success": false,
  "error": {
    "code": "NODE_NOT_BOUND_TO_GATEWAY",
    "message": "Node RIMBA-NODE-009 tidak terdaftar pada gateway RIMBA-GW-001.",
    "details": {
      "event_id": "550e8400-e29b-41d4-a716-446655440009"
    }
  },
  "server_time_utc": "2026-11-23T10:35:05Z"
}

```

## 🛠️ 14. Panduan Instalasi API Gateway

Ikuti langkah berikut untuk membangun API Gateway PHP 8.x secara lokal maupun sebelum diunggah ke server hosting:

### Langkah 1: Persiapan Lingkungan

* Pastikan **PHP 8.1 / 8.2 / 8.3** terpasang dengan ekstensi aktif: `pdo_mysql`, `json`, `openssl`, `mbstring`, `ctype`.

* Pastikan **MySQL 8.0+** atau **MariaDB 10.6+** tersedia.

### Langkah 2: Konfigurasi Environment Rahasia (`private/.env`)

Salin template konfigurasi dan isi dengan kredensial Anda (**jangan pernah memasukkan nilai asli ke Git**):

```
APP_ENV=production
APP_DEBUG=false
APP_TIMEZONE=UTC

DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=<NAMA_DATABASE_ANDA>
DB_USER=<USER_DATABASE_ANDA>
DB_PASS=<PASSWORD_DATABASE_KUAT_ANDA>

CORS_ALLOWED_ORIGIN=https://<domain-dashboard-anda.com>
HMAC_MAX_CLOCK_SKEW_SEC=300
JWT_SECRET=<SECRET_ACAK_MINIMAL_64_KARAKTER>

```

### Langkah 3: Koneksi PDO Aman (`private/database.php`)

```
<?php
declare(strict_types=1);

function getDatabaseConnection(array $config): PDO
{
    $dsn = sprintf(
        'mysql:host=%s;port=%d;dbname=%s;charset=utf8mb4',
        $config['db_host'],
        $config['db_port'],
        $config['db_name']
    );

    $pdo = new PDO($dsn, $config['db_user'], $config['db_pass'], [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES   => false, // Wajib false untuk proteksi SQL Injection sejati
    ]);

    $pdo->exec("SET time_zone = '+00:00'");
    return $pdo;
}

```

### Langkah 4: Verifikasi Autentikasi Gateway (`src/Middleware/GatewayAuthMiddleware.php`)

Logika wajib pada middleware autentikasi gateway:

1. Ambil header `X-Gateway-Id`, `X-Api-Key`, `X-Timestamp`, dan `X-Signature`.

2. Tolak permintaan jika selisih `abs(time() - (int)$timestamp) > 300` detik (pencegahan *replay attack*).

3. Ambil record gateway dari tabel `gateways` dengan *prepared statement*.

4. Bandingkan hash API Key menggunakan `hash_equals()` (tahan *timing attack*).

5. Hitung ulang `hash_hmac('sha256', $method . $path . $timestamp . $rawBody, $gatewayHmacSecret)` dan cocokkan menggunakan `hash_equals()`.

### Langkah 5: Impor Skema Database & Seed Gateway Awal

Jalankan skrip `database/schema.sql`, lalu daftarkan gateway pertama dan node awalnya menggunakan *hash* API Key yang dihasilkan secara acak (`openssl rand -hex 32`).

### Langkah 6: Pengujian Awal via cURL

```
# Contoh pengujian endpoint heartbeat (ganti placeholder sebelum dijalankan)
TIMESTAMP=$(date -u +%s)
BODY='{"gateway_id":"RIMBA-GW-001","reported_at":"2026-11-23T10:35:05Z","uptime_seconds":120,"ethernet_status":"UP","rtc_status":"OK","sd_status":"MOUNTED","offline_queue_count":0,"sync_status":"IDLE","api_latency_ms":95}'
SIGNATURE=$(printf "POST/api/v1/gateways/heartbeat%s%s" "$TIMESTAMP" "$BODY" | openssl dgst -sha256 -hmac "<GATEWAY_HMAC_SECRET_PLACEHOLDER>" | awk '{print $2}')

curl -X POST "https://api.<domain-anda.com>/api/v1/gateways/heartbeat" \
  -H "Content-Type: application/json" \
  -H "X-Gateway-Id: RIMBA-GW-001" \
  -H "X-Api-Key: <GATEWAY_API_KEY_PLACEHOLDER>" \
  -H "X-Timestamp: $TIMESTAMP" \
  -H "X-Signature: $SIGNATURE" \
  -d "$BODY"

```

## 🚀 15. Panduan Deployment cPanel

Ikuti langkah berikut untuk men-*deploy* API Gateway secara aman pada shared/cloud hosting berbasis **cPanel**:

1. **Membuat Subdomain API:**

   * Masuk ke cPanel $\rightarrow$ **Domains** $\rightarrow$ **Create A New Domain**.

   * Masukkan `api.<domain-anda.com>`.

   * Hilangkan centang *"Share document root"* dan atur **Document Root** ke: `/home/<cpanel-user>/rimbalink-api/public`.

2. **Mengaktifkan SSL/HTTPS:**

   * Buka menu **SSL/TLS Status** atau **Let's Encrypt SSL** di cPanel.

   * Jalankan **Run AutoSSL** untuk `api.<domain-anda.com>` dan verifikasi bahwa sertifikat aktif.

   * Aktifkan **Force HTTPS Redirect** pada menu **Domains** atau via `.htaccess`.

3. **Membuat Database & User MySQL:**

   * Buka **MySQL Databases** (atau **Manage My Databases**).

   * Buat database baru (mis. `<cpaneluser>_rimbalink`).

   * Buat user database baru dengan kata sandi acak kuat.

   * Tambahkan user ke database dengan hak akses yang diperlukan (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `INDEX`, `ALTER`; hindari hak berlebih yang tidak diperlukan saat produksi).

   * Buka **phpMyAdmin**, pilih database, dan impor `database/schema.sql`.

4. **Mengunggah Kode Backend di Luar Folder Publik:**

   * Unggah folder `src/`, `private/`, `logs/`, dan `public/` ke `/home/<cpanel-user>/rimbalink-api/`.

   * Atur *permission* file `private/.env` dan `private/config.php` menjadi `600` (hanya bisa dibaca oleh pemilik akun cPanel/PHP).

5. **Konfigurasi `public/.htaccess`:**

```
<IfModule mod_rewrite.c>
    RewriteEngine On

    # Paksa HTTPS
    RewriteCond %{HTTPS} !=on
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

    # Teruskan Header Authorization ke PHP (penting pada beberapa konfigurasi CGI/FastCGI cPanel)
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    # Arahkan seluruh request non-file ke Front Controller index.php
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.php [QSA,L]
</IfModule>

# Blokir akses langsung ke file tersembunyi (.env, .git, .htaccess)
<FilesMatch "^\.">
    Require all denied
</FilesMatch>

Options -Indexes

```

6. **Memilih Versi PHP:**

   * Buka **MultiPHP Manager** di cPanel dan pastikan subdomain `api.<domain-anda.com>` menggunakan **PHP 8.1 / 8.2 / 8.3**.

7. **Menghubungkan ESP32 Gateway ke API Server:**

   * Ekspor **Root CA Certificate** dari rantai sertifikat SSL `api.<domain-anda.com>` (mis. ISRG Root X1 untuk AutoSSL Let's Encrypt).

   * Simpan Root CA pada firmware ESP32 Gateway (`certs/root_ca.h`), atur host `api.<domain-anda.com>`, port `443`, `gateway_id`, API Key, dan HMAC Secret, lalu uji koneksi heartbeat via Ethernet W5500.

## 💻 16. Arsitektur Dashboard

Dashboard Web RimbaLink dirancang sebagai aplikasi web responsif (*desktop-first & mobile-friendly*) yang berkomunikasi secara asinkron dengan API Server melalui REST JSON.

* **Pemisahan Layer (*Decoupled*):** Frontend (HTML5/CSS3/JS modern atau SPA seperti React/Vue/Next.js sesuai pilihan tim) hanya memanggil endpoint `/api/v1/*` tanpa mengakses database secara langsung.

* **Konversi Zona Waktu:** Seluruh respons API menggunakan UTC (`Z`); frontend mengonversi tampilan waktu ke **`Asia/Jakarta` (WIB)** secara konsisten dengan label zona waktu yang jelas.

* **Pembaruan Data:** Mendukung *auto-refresh* berkala (mis. setiap 30–60 detik) dengan indikator waktu pembaruan terakhir agar tidak membebani shared hosting cPanel dengan koneksi *WebSocket* persisten.

## 📊 17. Monitoring Node dan Gateway

### A. Overview Dashboard

Halaman ringkasan utama menampilkan kartu KPI (*Key Performance Indicators*):

* Total node terdaftar, jumlah node **ONLINE**, dan jumlah node **OFFLINE**.

* Jumlah gateway **ONLINE** dan **OFFLINE**.

* Status peringatan dini tertinggi saat ini (`NORMAL` / `WASPADA` / `SIAGA` / `BAHAYA`).

* Nilai rata-rata terkini: suhu udara, kelembapan udara, kelembapan tanah, serta status indikasi gas/asap.

* Status sistem sinkronisasi global (jumlah antrean offline aktif di seluruh gateway).

### B. Monitoring & CRUD Node

Halaman detail node menampilkan:

* Identitas & lokasi: `node_code`, nama, koordinat (Lat/Lng), elevasi, tinggi antena, dan gateway terhubung.

* Status operasional: `ONLINE`/`OFFLINE` dan `last_seen_at` (WIB).

* Data sensor terkini: suhu udara, kelembapan udara, suhu tanah, kelembapan tanah, nilai mentah gas/asap.

* Daya & aktuator: tegangan baterai 2S 18650, status panel surya (`STANDBY`/`CHARGING`), dan status `ON`/`OFF` relay 1–4.

* Verifikasi visual: status verifikasi kamera, deteksi api/asap, *confidence* (jika ada), waktu verifikasi, dan tautan bukti gambar.

* Kualitas radio LoRa: RSSI (dBm), SNR (dB), dan riwayat paket.

**Fitur CRUD Node:** Tambah node, lihat detail, edit atribut/koordinat/tinggi antena/gateway tertaut, serta hapus node dengan **Soft Delete** (`deleted_at`) sehingga seluruh data historis telemetri tetap dapat diaudit.

### C. Monitoring Gateway

Halaman detail gateway menampilkan:

* `gateway_code`, status **ONLINE/OFFLINE** (dihitung secara dinamis: `NOW() - last_heartbeat_at <= heartbeat_timeout_sec`, dengan timeout yang dapat dikonfigurasi per gateway).

* Metrik jaringan IP terpisah: status Ethernet (`UP`/`DOWN`) dan **latensi komunikasi API (`api_latency_ms`)**.

* Metrik radio LoRa per node terhubung: daftar RSSI & SNR masing-masing node.

* Kesehatan perangkat: `uptime_seconds`, status RTC & sinkronisasi NTP, status microSD, kapasitas total/sisa microSD, jumlah antrean offline (`offline_queue_count`), status sinkronisasi, status panel surya gateway, tegangan baterai gateway (jika terpasang), dan grafik riwayat heartbeat.

### D. Status Link LoRa per Node

Panel khusus kualitas link radio menampilkan:

* **RSSI (dBm)** dan **SNR (dB)** terbaru serta rata-rata periode berjalan.

* **Jumlah paket diterima** dan **estimasi *packet loss*** yang dihitung dari selisih `sequence_number` yang diharapkan terhadap jumlah paket aktual pada jendela waktu tertentu (dengan pengecualian saat deteksi *reboot* node).

* **Last packet received** dan indikator status link (`BAIK`, `CUKUP`, `LEMAH`, `KRITIS`) berdasarkan **ambang batas yang dapat dikonfigurasi** mengacu pada sensitivitas penerima SX1276 sesuai *Spreading Factor (SF)* dan *Bandwidth (BW)* yang digunakan—bukan angka acak tanpa dasar teknis.

## 📈 18. Grafik dan Analisis Data

Dashboard menyediakan grafik interaktif (mis. Chart.js / ApexCharts / ECharts) untuk seluruh parameter utama:

1. Suhu udara ($^{\circ}\text{C}$)

2. Kelembapan udara ($\%\text{RH}$)

3. Suhu tanah ($^{\circ}\text{C}$)

4. Kelembapan tanah ($\%$)

5. Nilai mentah gas dan asap (`gas_raw`, `smoke_raw`)

6. Tegangan baterai 2S 18650 ($\text{V}$)

7. RSSI ($\text{dBm}$)

8. SNR ($\text{dB}$)

### Strategi Agregasi dan Performa Kueri

Karena 1 node mengirim $1\text{ paket/menit}$ ($1.440\text{ record/hari}$, $\approx 43.200\text{ record/bulan}$ per node), dashboard **membedakan data mentah dan data agregat** agar respons API tetap cepat:

| Filter Rentang Waktu | Granularitas Default | Metode Penyajian | 
| ----- | ----- | ----- | 
| **Per Jam** ($\le 6\text{ jam}$) | Data Mentah (`raw`, 1 menit) | Titik data asli + pagination untuk tabel ekspor | 
| **Per Hari** ($24\text{ jam}$) | Agregasi per 15 menit / per jam | `AVG`, `MIN`, `MAX`, dan `sample_count` | 
| **Per Minggu** ($7\text{ hari}$) | Agregasi per jam (`hourly`) | `AVG`, `MIN`, `MAX`, dan `sample_count` | 
| **Per Bulan** ($30\text{ hari}$) | Agregasi harian (`daily`) | `AVG`, `MIN`, `MAX`, dan `sample_count` | 
| **Rentang Khusus** | Otomatis menyesuaikan durasi | Memilih granularitas optimal agar titik grafik $\le 300\text{ titik}$ | 

Grafik menampilkan garis nilai rata-rata (`AVG`) disertai pita rentang (`MIN`–`MAX`) dan jumlah sampel (`COUNT`) pada *tooltip* agar lonjakan sesaat (seperti puncak suhu atau gas) tidak tertutupi oleh proses perataan.

## 🗺️ 19. Peta Interaktif

Peta interaktif (mis. berbasis **Leaflet** atau **MapLibre GL**) memvisualisasikan sebaran spasial seluruh perangkat RimbaLink:

* **Marker & Indikator Visual:** Ikon berbeda untuk Gateway dan Node, kode ID node, status `ONLINE`/`OFFLINE`, indikator kualitas link LoRa, status peringatan dini, dan *popup* ringkasan sensor terbaru.

* **Garis Jalur Komunikasi (Polyline):** Menghubungkan setiap node ke gateway induknya lengkap dengan label jarak geodesik (*Haversine/Vincenty*) dalam meter/kilometer dan pewarnaan berdasarkan status link RSSI/SNR.

* **Interaksi CRUD Spasial:**

  1. Menambah node baru dengan mengklik langsung titik koordinat pada peta.

  2. Memindahkan posisi node menggunakan fitur *drag marker* dengan dialog konfirmasi penyimpanan koordinat baru.

  3. Melihat detail node & membuka panel analisis *Fresnel Zone* langsung dari marker.

* **Estimasi Area Jangkauan:** Menampilkan lingkaran/poligon estimasi jangkauan indikatif (dengan label jelas bahwa jangkauan aktual sangat dipengaruhi topografi, vegetasi, antena, dan *link budget*).

## 📶 20. Analisis Fresnel Zone

RimbaLink menyediakan modul simulasi **Zona Fresnel Pertama (*First Fresnel Zone*)** untuk membantu perencanaan *line-of-sight* (LoS) radio antara gateway dan node.

### Rumus Dasar Radius Zona Fresnel Pertama

Radius zona Fresnel pertama $r_1$ (dalam meter) pada titik berjarak $d_1$ dari pemancar dan $d_2$ dari penerima dihitung sebagai berikut:

$$
r_1 = \sqrt{\frac{\lambda \cdot d_1 \cdot d_2}{d_1 + d_2}}
$$

dengan panjang gelombang $\lambda$ (dalam meter):

$$
\lambda = \frac{c}{f}
$$

Keterangan parameter:

* $r_1$: Radius zona Fresnel pertama pada titik evaluasi ($\text{m}$).

* $\lambda$: Panjang gelombang sinyal LoRa ($\text{m}$).

* $c$: Kecepatan cahaya di ruang hampa ($c \approx 2{,}99792458 \times 10^8\text{ m/s}$).

* $f$: Frekuensi kerja LoRa yang digunakan ($\text{Hz}$, mengikuti regulasi resmi Indonesia).

* $d_1$: Jarak dari antena gateway ke titik evaluasi pada jalur ($\text{m}$).

* $d_2$: Jarak dari titik evaluasi ke antena node ($\text{m}$), di mana total jarak jalur $D = d_1 + d_2$.

### Parameter Wajib Perhitungan & Keterbatasan Data Elevasi

> ⚠️ **PENTING:** Analisis *Fresnel Zone* **tidak boleh hanya mengandalkan koordinat Latitude dan Longitude**. Tanpa profil elevasi permukaan bumi dan tinggi antena, sistem **tidak boleh menghasilkan angka persentase clearance palsu**.

Perhitungan profil jalur yang layak wajib melibatkan:

1. Frekuensi kerja LoRa ($f$) dan panjang gelombang ($\lambda$).

2. Jarak total jalur ($D$) serta titik sampling sepanjang lintasan ($d_1, d_2$).

3. **Tinggi antena gateway** ($h_{\text{gw}}$) dan **tinggi antena node** ($h_{\text{node}}$) di atas permukaan tanah.

4. **Profil elevasi tanah (DEM)** sepanjang jalur dari sumber data topografi resmi seperti **DEMNAS (Badan Informasi Geospasial — Indonesia)**, **SRTM 30m**, atau **Copernicus DEM**.

5. Koreksi kelengkungan bumi (*Earth bulge*) dengan faktor refraksi standar $k = \frac{4}{3}$ pada jarak jauh, serta estimasi tinggi penghalang (kanopi vegetasi/bangunan) apabila tersedia.

### Kriteria Clearance

* Sebagai pedoman perencanaan link RF yang lazim, jalur radio ditargetkan memiliki **clearance minimal** $60\%$ **dari radius zona Fresnel pertama** ($0{,}6 \times r_1$) yang bebas dari penghalang.

* **Penandaan Status Estimasi:** Jika data DEM/elevasi untuk lokasi tertentu belum tersedia atau tinggi kanopi hutan belum terukur, antarmuka dashboard **wajib menampilkan label `ESTIMASI (TANPA PROFIL ELEVASI LENGKAP)`** dan hanya menampilkan radius teoritis $r_1$ tanpa mengklaim jalur bebas hambatan.

## 📍 21. Rekomendasi Titik Penempatan Node

Modul rekomendasi titik penempatan node (`PLANNED`) menggunakan pendekatan **Multi-Criteria Decision Analysis (MCDA)** berbasis bobot yang dapat dikonfigurasi untuk mengevaluasi koordinat kandidat di sekitar area pantau.

### Kriteria Evaluasi

| Kriteria | Pertimbangan Teknis | 
| ----- | ----- | 
| **Jangkauan & Jarak ke Gateway** | Berada dalam batas *link budget* realistis LoRa SX1276 dari gateway | 
| **Kualitas Fresnel Zone** | Persentase *clearance* terhadap profil DEM dan kelengkungan bumi | 
| **Elevasi Permukaan Tanah** | Titik lebih tinggi cenderung memberi visibilitas RF lebih baik | 
| **Potensi Penghalang** | Meminimalkan tutupan bukit, bangunan, atau kanopi sangat rapat | 
| **Ketersediaan Sinar Matahari** | Menghindari lokasi ternaungi total agar panel surya dapat mengisi baterai 2S 18650 | 
| **Area Prioritas Pantau** | Kedekatan dengan zona rawan karhutla (mis. lahan gambut kering/perbatasan lahan) | 
| **Akses Perawatan & Keamanan** | Kemudahan akses teknisi untuk kalibrasi/pembersihan panel serta keamanan fisik perangkat | 

### Format Output Rekomendasi

Setiap kandidat titik menampilkan:

* Koordinat kandidat (`latitude`, `longitude`).

* Jarak ke gateway ($\text{m}$ / $\text{km}$).

* Estimasi kualitas jalur LoRa dan kondisi *Fresnel Zone* (disertai catatan sumber DEM).

* Alasan rekomendasi (rincian skor per kriteria).

* Tingkat keyakinan analisis & keterbatasan data (mis. belum mencakup tinggi tajuk pohon aktual).

> **Validasi Lapangan Wajib:** Rekomendasi berbasis peta dan DEM merupakan alat bantu perencanaan awal dan **wajib diverifikasi melalui survei lapangan serta pengujian pengukuran RSSI dan SNR aktual** sebelum pemasangan permanen.

## 📷 22. Verifikasi Kamera

Modul verifikasi kamera menyimpan dan menampilkan hasil verifikasi visual untuk mengurangi risiko *false alarm* dari pembacaan sensor gas/suhu semata.

### Informasi yang Disimpan dan Ditampilkan

* Identitas `node_id` / sektor lahan terkait.

* `verified_at`: waktu verifikasi (disimpan UTC, ditampilkan WIB).

* `verification_status`: `UNAVAILABLE`, `PENDING`, `VERIFIED`, atau `FAILED`.

* Hasil deteksi: `fire_detected` (`true`/`false`/`null`) dan `smoke_detected`.

* `confidence_score`: skor keyakinan model deteksi jika tersedia (rentang $0{,}0000\text{--}1{,}0000$, bernilai `null` jika tidak tersedia).

* `image_reference`: path/URL aman menuju bukti citra visual (dengan proteksi akses terautentikasi).

* `analysis_notes`: catatan hasil inferensi otomatis maupun konfirmasi operator lapangan.

> **Catatan:** Arsitektur penempatan kamera berstatus **`[CONFIG_TBD]`** (lihat [Opsi Arsitektur Kamera](#d-opsi-arsitektur-verifikasi-kamera-config_tbd)). Nilai ambang batas `confidence_score` untuk keputusan otomatis tidak ditetapkan secara arbitrer dan harus ditentukan dari hasil evaluasi model (*precision/recall*) pada dataset lapangan.

## 🚨 23. Peta Risiko dan Peringatan Dini

### A. Konsep Peringatan Dini Multi-Parameter

RimbaLink menggabungkan berbagai indikator agar sistem tidak bergantung pada satu sensor tunggal:

1. **Kondisi Mikroklimat:** Kombinasi peningkatan suhu udara/tanah serta penurunan kelembapan udara dan kelembapan tanah.

2. **Indikasi Pembakaran:** Lonjakan nilai `gas_raw` dan/atau `smoke_raw` dibandingkan *baseline* harian node tersebut.

3. **Verifikasi Visual:** Konfirmasi `fire_detected` / `smoke_detected` dari modul kamera (jika tersedia).

4. **Persistensi Waktu:** Kondisi anomali bertahan selama beberapa siklus berturut-turut (mis. $N$ menit) untuk meredam *spike* sesaat.

#### Klasifikasi Level Peringatan

| Level | Indikasi Konseptual | Tindak Lanjut Sistem | 
| ----- | ----- | ----- | 
| 🟢 **`NORMAL`** | Seluruh parameter dalam rentang *baseline* normal dan tidak ada anomali gas/visual | Pemantauan rutin 1 menit | 
| 🟡 **`WASPADA`** | Kondisi lahan mengering / suhu meningkat di atas *threshold* awal tanpa indikasi asap/api | Tandai zona pada peta risiko & tingkatkan kewaspadaan operator | 
| 🟠 **`SIAGA`** | Anomali multi-sensor (mis. suhu naik + kelembapan turun + kenaikan `gas_raw`/`smoke_raw` persisten) | Picu notifikasi operator & minta verifikasi kamera / pengecekan lapangan | 
| 🔴 **`BAHAYA`** | Indikasi kuat multi-sensor disertai konfirmasi verifikasi kamera (`fire_detected = true`) atau lonjakan kritis tervalidasi | Peringatan prioritas tinggi, catat log insiden, & opsi aktivasi prosedur tanggap darurat / relay (sesuai SOP) | 

> ⚠️ **Kalibrasi Threshold:** Ambang batas suhu, kelembapan, gas mentah, dan asap **disimpan sebagai konfigurasi yang dapat diubah oleh administrator** dan **wajib dikalibrasi serta divalidasi pada kondisi lahan spesifik** (misalnya karakteristik lahan gambut berbeda dengan hutan dataran tinggi).

### B. Peta Risiko dan Prakiraan (`PLANNED`)

Untuk menjaga akurasi ilmiah, dashboard memisahkan secara tegas tiga kategori data:

1. **Data Hasil Pengukuran (*Observed Data*):** Pembacaan langsung sensor di node (`telemetry`).

2. **Data Hasil Inferensi (*Inferred Risk*):** Skor tingkat risiko spasial/level peringatan yang dihitung dari penggabungan sensor, histori kekeringan, dan verifikasi kamera.

3. **Data Prakiraan (*Forecast*):** Estimasi tren risiko jangka pendek yang hanya diaktifkan apabila tersedia seri data historis yang memadai dan/atau integrasi sumber prakiraan cuaca resmi (mis. API prakiraan cuaca BMKG/eksternal).

### Diagram 7: Flowchart Proses Monitoring dan Peringatan Dini

```
flowchart TD
    A["Data Telemetri Baru Masuk ke API Server"] --> B["Simpan Data Terukur ke Tabel telemetry, lora_link_metrics, relay_states"]
    B --> C["Muat Konfigurasi Threshold Terkalibrasi per Node/Zona"]
    C --> D["Evaluasi Parameter Mikroklimat\n(Suhu Udara/Tanah, Kelembapan Udara/Tanah)"]
    D --> E["Evaluasi Anomali Gas/Asap Mentah\nterhadap Baseline & Persistensi Waktu"]
    E --> F{"Hasil Verifikasi Kamera Tersedia?"}
    F -- "Ya" --> G["Gabungkan Skor Multi-Sensor + Hasil Verifikasi Visual"]
    F -- "Tidak" --> H["Hitung Skor Berdasarkan Multi-Sensor Saja\n(Tandai: Tanpa Verifikasi Visual)"]
    G --> I{"Tentukan Level Peringatan"}
    H --> I
    I -- "Normal" --> J["Status: NORMAL\nPerbarui Peta & Dashboard"]
    I -- "Waspada" --> K["Status: WASPADA\nCatat Alert & Sorot Zona Risiko"]
    I -- "Siaga" --> L["Status: SIAGA\nCatat Alert, Notifikasi Operator & Minta Verifikasi Kamera"]
    I -- "Bahaya" --> M["Status: BAHAYA\nPrioritas Tinggi, Notifikasi Darurat & Audit SOP"]

```

## 🔒 24. Keamanan Sistem

Keamanan RimbaLink diterapkan berlapis (*defense-in-depth*) mulai dari kanal radio hingga aplikasi web:

| Lapisan | Mekanisme Keamanan | Implementasi | 
| ----- | ----- | ----- | 
| **Transportasi Web** | **HTTPS / TLS 1.2+** | Wajib pada seluruh endpoint API & Dashboard; verifikasi Root CA aktif pada ESP32 Gateway (tanpa `setInsecure()`) | 
| **Autentikasi Gateway** | **API Key per Gateway + HMAC-SHA256** | Setiap gateway memiliki API Key & HMAC Secret unik; validasi menggunakan `hash_equals()` | 
| **Anti-Replay API** | **Timestamp Window Validation** | Header `X-Timestamp` diverifikasi dalam jendela toleransi waktu (mis. $\pm 300\text{ detik}$) | 
| **Idempotency** | **Unik `event_id` & `batch_id`** | Mencegah penggandaan data saat pengiriman ulang akibat koneksi terputus | 
| **Kanal Radio LoRa** | **Validasi Framing + Allowlist + HMAC (+ Opsi AES)** | Memeriksa `MAGIC_WORD`, `NETWORK_ID`, `DESTINATION_GATEWAY_ID`, `NODE_ID` terdaftar, `sequence_number`, & `AUTH_TAG` | 
| **Validasi Input** | **Strict JSON Schema & Type Checking** | Menolak payload tidak valid, tipe data salah, atau nilai di luar batas fisik sensor | 
| **Database** | **PDO Prepared Statements** | `PDO::ATTR_EMULATE_PREPARES => false` untuk mencegah *SQL Injection* secara menyeluruh | 
| **Ketersediaan API** | **Rate Limiting** | Pembatasan jumlah request per IP / per `gateway_id` untuk mencegah penyalahgunaan & *DoS* ringan | 
| **Akses Browser** | **Strict CORS Policy** | Hanya mengizinkan `Origin` resmi domain Dashboard RimbaLink | 
| **Pengguna Dashboard** | **Password Hashing + RBAC** | Hash menggunakan `password_hash()` (**Argon2id** atau **bcrypt**); peran `ADMIN`, `OPERATOR`, `VIEWER` | 
| **Manajemen Rahasia** | **Konfigurasi di Luar Document Root** | File `.env` dan kredensial disimpan di `private/` (bukan `public/`), dikecualikan via `.gitignore`, tanpa rahasia *hardcoded* di README | 
| **Audit & Pengelolaan Kunci** | **Audit Log & Rotasi Kunci** | Mencatat kegagalan autentikasi, perubahan CRUD node, & rotasi kunci per perangkat tanpa mengganggu perangkat lain | 

## 🧪 25. Panduan Implementasi dan Pengujian

Gunakan daftar periksa (*checklist*) 20 tahap berikut secara berurutan dari persiapan perangkat keras hingga pengujian integrasi penuh:

| Tahap | Aktivitas Implementasi & Pengujian | Kriteria Keberhasilan | 
| ----- | ----- | ----- | 
| **1** | **Persiapan Hardware** | Cek kontinuitas kabel, tegangan suplai $3{,}3\text{ V}$/$5\text{ V}$, polaritas baterai 2S 18650 + BMS, & pemasangan antena SX1276 (jangan nyalakan pemancar tanpa antena!) | 
| **2** | **Konfigurasi LoRa Node** | Pembacaan seluruh sensor lingkungan, ADC baterai 2S, status panel surya, dan uji sakelar relay 1–4 stabil pada *Serial Monitor* | 
| **3** | **Konfigurasi ID & `NETWORK_ID`** | Setiap node memiliki `NODE_ID` unik, `DESTINATION_GATEWAY_ID` benar, dan `NETWORK_ID` seragam | 
| **4** | **Konfigurasi ESP32 Gateway** | Firmware ESP32 boot stabil tanpa *brownout/watchdog reset*; task FreeRTOS berjalan lancar | 
| **5** | **Konfigurasi SX1276** | Frekuensi sesuai regulasi Indonesia (`<FREQ_REGULASI_INDONESIA_TBD>`), paket dari node diterima dengan `CRC16`, `MAGIC_WORD`, dan `AUTH_TAG` valid | 
| **6** | **Konfigurasi W5500** | Link Ethernet UP, memperoleh IP (DHCP/Statis), mampu membuka koneksi HTTPS/TLS terverifikasi ke API Server | 
| **7** | **Konfigurasi RTC** | RTC terbaca via I2C, waktu UTC tersinkronisasi melalui NTP saat online, dan tetap bertahan saat daya utama dicabut | 
| **8** | **Konfigurasi microSD Card** | Mount FAT32 berhasil, penulisan file `/rimbalink/queue/YYYY/MM/YYYY-MM-DD.jsonl` sukses dan terbaca ulang tanpa korupsi | 
| **9** | **Pembuatan Database MySQL** | Seluruh 10 tabel, *foreign key*, indeks, dan *unique constraint* pada `schema.sql` berhasil dibuat tanpa error | 
| **10** | **Subdomain API di cPanel** | `https://api.<domain-anda.com>` aktif dengan SSL valid dan *Document Root* menunjuk tepat ke `rimbalink-api/public` | 
| **11** | **Instalasi & Deployment PHP API** | File `private/.env` tidak dapat diakses dari browser (HTTP 403/404), koneksi PDO ke MySQL sukses | 
| **12** | **Uji Endpoint API (Postman / cURL)** | Uji `POST /gateways/heartbeat`, `POST /nodes`, `GET /nodes` dengan header auth valid (`200`/`201`) dan invalid (`401`/`403`) | 
| **13** | **Uji Pengiriman Batch** | Kirim 10 record melalui `POST /api/v1/telemetry/batch`; verifikasi masuk ke `telemetry`, `lora_link_metrics`, & `relay_states` | 
| **14** | **Uji Mode Offline** | Cabut kabel Ethernet W5500 selama 10–15 menit; verifikasi node tetap mengirim tiap 1 menit dan data tersimpan di microSD dengan `pending_records` bertambah | 
| **15** | **Uji Sinkronisasi Pemulihan** | Pasang kembali kabel Ethernet; verifikasi gateway mengirim antrean dalam batch, menerima ACK, memperbarui `sync_cursor.json`, dan tidak ada data ganda di DB (uji juga kirim ulang batch yang sama untuk membuktikan *idempotency*) | 
| **16** | **Uji Dashboard Web** | Halaman Overview, Monitoring Node, dan Monitoring Gateway tampil responsif pada desktop dan mobile dengan waktu `Asia/Jakarta` | 
| **17** | **Uji CRUD Node** | Tambah, ubah koordinat/tinggi antena, dan *soft delete* node; pastikan data historis telemetri node yang dihapus tetap utuh di DB | 
| **18** | **Uji Grafik & Agregasi** | Uji filter per jam, hari, minggu, bulan, dan rentang khusus; pastikan `AVG`/`MIN`/`MAX` akurat dan waktu muat tetap cepat | 
| **19** | **Uji Peta & Fresnel Zone** | Uji *click-to-add*, *drag marker*, garis link, jarak geodesik, dan profil *Fresnel Zone* (pastikan label estimasi tampil jika data DEM tidak tersedia) | 
| **20** | **Uji Kualitas Link LoRa** | Uji jarak bertahap di lapangan; catat RSSI, SNR, dan estimasi *packet loss* dari `sequence_number` per node secara terpisah dari latensi API gateway | 

## 🔧 26. Troubleshooting

| Gejala / Masalah | Kemungkinan Penyebab | Langkah Penanganan | 
| ----- | ----- | ----- | 
| **SX1276, W5500, atau microSD gagal inisialisasi bersamaan** | Konflik bus SPI (pin `CS` tidak di-HIGH saat init atau tidak ada *mutex locking*) | Set seluruh pin `CS` ke `HIGH` sebelum inisialisasi modul mana pun; pisahkan bus `HSPI` dan `VSPI` pada ESP32 | 
| **HTTPS dari ESP32 + W5500 gagal / handshake error** | Salah menggunakan `WiFiClientSecure`, Root CA tidak cocok/kedaluwarsa, atau RAM habis | Gunakan stack TLS kompatibel Ethernet (`esp_eth` + mbedTLS atau `SSLClient` BearSSL) dan perbarui Root CA AutoSSL | 
| **Data offline terduplikasi di database** | Gateway membuat `event_id` baru setiap kali *retry* | Pastikan `event_id` dibuat **satu kali saja saat ditulis ke microSD** dan tidak pernah diubah saat pengiriman ulang | 
| **Waktu telemetri melompat ke tahun 1970/2000** | Baterai koin RTC habis atau RTC belum diset via NTP | Periksa baterai RTC; pastikan firmware mendeteksi `rtc_valid = false` dan menandai `time_source = "UNSYNCED_UPTIME"` | 
| **API cPanel mengembalikan `401 Unauthorized` padahal token/header dikirim** | Konfigurasi FastCGI/Apache cPanel memotong header `Authorization` atau selisih jam gateway $> 5$ menit | Pastikan rule `RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]` ada di `.htaccess` dan sinkronkan waktu NTP gateway | 
| **API mengembalikan `403 Forbidden` pada telemetri** | `node_id` belum didaftarkan atau ditautkan ke `gateway_id` yang berbeda | Daftarkan node melalui `POST /api/v1/nodes` dan pastikan `gateway_id` sesuai | 
| **Gateway berstatus `OFFLINE` di dashboard padahal data masuk** | `heartbeat_timeout_sec` terlalu kecil atau endpoint heartbeat belum dijadwalkan | Sesuaikan interval kirim heartbeat (mis. 60 detik) lebih kecil dari `heartbeat_timeout_sec` (mis. 180 detik) | 
| **Paket dari node asing muncul di log** | Perangkat LoRa pihak ketiga beroperasi pada frekuensi dan SF yang sama | Perilaku normal pada spektrum bebas lisensi; pastikan gateway menolak paket saat validasi `MAGIC_WORD`, `NETWORK_ID`, `NODE_ID`, dan `HMAC` | 

## 🗺️ 27. Roadmap Pengembangan

* **Fase 1 — Fondasi Sistem & Konektivitas (Saat Ini — `IN DEVELOPMENT`):**

  * Finalisasi skematik pinout hardware Node & ESP32 Gateway.

  * Implementasi firmware protokol LoRa (framing + CRC16 + HMAC-SHA256) dan *store-and-forward* microSD + RTC.

  * Implementasi penuh PHP 8.x REST API, database MySQL, dan deployment cPanel HTTPS.

  * Pengujian keandalan mode offline dan sinkronisasi idempoten.

* **Fase 2 — Dashboard Operasional & Analitik (`IN DEVELOPMENT`** $\rightarrow$ **`PLANNED`):**

  * Penyelesaian Dashboard Web responsif (Overview, CRUD Node *soft delete*, Monitoring Gateway, Grafik Agregasi, Kualitas Link LoRa).

  * Pengujian lapangan awal (1–5 node) untuk pengumpulan *baseline* sensor dan kalibrasi *threshold* lingkungan.

* **Fase 3 — GIS Lanjut, Verifikasi Kamera & Peringatan Dini (`PLANNED`):**

  * Integrasi sumber data DEM (DEMNAS/SRTM) untuk analisis *Fresnel Zone* penuh dan rekomendasi titik penempatan node.

  * Penetapan arsitektur kamera (`[CONFIG_TBD]`) dan integrasi bukti visual ke modul peringatan dini.

  * Pengembangan peta risiko spasial dan evaluasi modul prakiraan berbasis data historis & cuaca.

* **Fase 4 — Skalabilitas Multi-Gateway (`PLANNED`):**

  * Dukungan manajemen puluhan node pada banyak gateway, rotasi kunci terkelola, dan optimalisasi partisi tabel `telemetry`.

## 🤝 28. Kontribusi

Kontribusi sangat terbuka untuk pengembangan firmware, backend API, analisis GIS/RF, maupun antarmuka dashboard:

1. *Fork* repositori ini dan buat branch fitur baru (`git checkout -b feature/<nama-fitur>`).

2. Pastikan tidak ada kredensial asli (API Key, password database, secret HMAC, atau kunci enkripsi) yang ikut ter-*commit*.

3. Ikuti standar kode: **PSR-12** untuk PHP, penamaan konsisten untuk SQL/JSON, serta perbarui dokumentasi apabila mengubah kontrak API atau protokol paket.

4. Sertakan deskripsi perubahan dan hasil pengujian lokal/lapangan pada *Pull Request*.

## 📄 29. Lisensi

Lisensi proyek ini belum ditetapkan secara final (`<LISENSI_PROYEK_TBD>`, mis. **MIT License** atau **Apache-2.0 License**). Tambahkan file `LICENSE` pada direktori utama repositori dan perbarui bagian ini beserta badge lisensi di bagian atas dokumen sesuai kebijakan tim pengembang atau institusi Anda.

## 📚 30. Kredit dan Referensi

Dokumentasi dan rujukan teknis resmi komponen yang digunakan pada arsitektur RimbaLink:

* **Espressif Systems — ESP32 & ESP-IDF (`esp_eth`, mbedTLS):** <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/>

* **Semtech — SX1276/77/78/79 LoRa Transceiver Datasheet:** <https://www.semtech.com/products/wireless-rf/lora-connect/sx1276>

* **WIZnet — W5500 Hardwired TCP/IP Ethernet Controller:** <https://docs.wiznet.io/Product/iEthernet/W5500/overview>

* **OPEnSLab-OSU — `SSLClient` (BearSSL TLS untuk Arduino `EthernetClient`):** <https://github.com/OPEnSLab-OSU/SSLClient>

* **PHP Official Documentation (PHP 8.x & PDO):** <https://www.php.net/manual/en/book.pdo.php>

* **MySQL 8.0 Reference Manual:** <https://dev.mysql.com/doc/refman/8.0/en/>

* **Leaflet — Open-Source JavaScript Library for Interactive Maps:** <https://leafletjs.com/>

* **Badan Informasi Geospasial (BIG) — DEMNAS (Digital Elevation Model Nasional Indonesia):** <https://tanahair.indonesia.go.id/demnas/>

* **ITU-R P.526 — Propagation by Diffraction (Referensi Teknis Zona Fresnel):** <https://www.itu.int/rec/R-REC-P.526/>

* **Regulasi Spektrum Frekuensi Radio Indonesia (Komdigi / DJID / SDPPI):** Periksa peraturan terbaru terkait standar teknis perangkat *Short Range Device* (SRD) / LPWAN LoRa di Indonesia sebelum pengoperasian lapangan melalui portal resmi <https://www.postel.go.id/>.
