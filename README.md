
<p align="center">
  <img src="gambar/RimbaLink.png" alt="RimbaLink banner" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/MCU-ESP32-E7352C?style=flat-square&logo=espressif&logoColor=white" alt="ESP32">
  <img src="https://img.shields.io/badge/CAMERA-ESP32--CAM-374151?style=flat-square&logoColor=white" alt="ESP32-CAM">
  <img src="https://img.shields.io/badge/WIRELESS-LoRa-7C3AED?style=flat-square&logoColor=white" alt="LoRa">
  <a href="https://easyeda.com/"><img src="https://img.shields.io/badge/PCB%20DESIGN-EasyEDA-1769FF?style=flat-square&logo=easyeda&logoColor=white" alt="Designed with EasyEDA"></a>
  <img src="https://img.shields.io/badge/FIRMWARE-C%2FC%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white" alt="Embedded C/C++">
</p>

<p align="center">
  <a href="https://github.com/0x-irfan/RimbaLink/stargazers"><img src="https://img.shields.io/github/stars/0x-irfan/RimbaLink?style=for-the-badge&logo=github&label=STARS" alt="GitHub stars"></a>
  <a href="https://github.com/0x-irfan/RimbaLink/network/members"><img src="https://img.shields.io/github/forks/0x-irfan/RimbaLink?style=for-the-badge&logo=git&label=FORKS" alt="GitHub forks"></a>
  <a href="https://github.com/0x-irfan/RimbaLink/issues"><img src="https://img.shields.io/github/issues/0x-irfan/RimbaLink?style=for-the-badge&logo=github&label=ISSUES" alt="Open issues"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/github/last-commit/0x-irfan/RimbaLink?style=flat-square&logo=github&label=Last%20update" alt="Last commit">
  <img src="https://img.shields.io/github/repo-size/0x-irfan/RimbaLink?style=flat-square&logo=github&label=Repository%20size" alt="Repository size">
  <img src="https://img.shields.io/badge/STATUS-IN_DEVELOPMENT-F97316?style=flat-square" alt="In development">
</p>

<p align="center">
  <sub>🌱 Monitoring nature · 📡 Connecting the field · ☀️ Powered by sunlight</sub>
</p>

<p align="center">
  <a href="https://github.com/0x-irfan/RimbaLink">Repository</a> ·
  <a href="#arsitektur-sistem">Arsitektur</a> ·
  <a href="#pengembangan-hardware">Hardware</a> ·
  <a href="#status-pengembangan">Status</a>
</p>

---

## Tentang Proyek

**RimbaLink** adalah rancangan sistem Internet of Things (IoT) untuk mendukung pemantauan kondisi lahan dan peringatan dini kebakaran hutan dan lahan (karhutla). Sistem dirancang menggunakan node sensor berbasis ESP32, komunikasi jarak jauh LoRa, serta sumber daya tenaga surya agar node pemantauan dapat ditempatkan di area yang jauh dari infrastruktur listrik.

Informasi dari node dikirim menuju gateway untuk diteruskan ke sistem pemantauan. Kamera digunakan sebagai sarana verifikasi visual terhadap kondisi atau indikasi kejadian di lapangan. Data pemantauan juga direncanakan dapat ditampilkan bersama peta risiko dan informasi prakiraan.

> **Catatan:** README ini menjelaskan rancangan dan arah pengembangan RimbaLink. Detail sensor, ambang batas peringatan, protokol payload, platform dashboard, serta hasil pengujian akan dilengkapi sesuai implementasi aktual.

## Tujuan

- Mengembangkan sistem pemantauan lahan berbasis IoT dengan komunikasi LoRa.
- Merancang node sensor yang dapat beroperasi menggunakan energi surya.
- Mengirimkan data pemantauan dari node ke gateway secara nirkabel.
- Mendukung verifikasi visual melalui modul kamera.
- Menyajikan informasi pemantauan dalam konteks lokasi dan peta risiko.
- Menjadi platform pengembangan sistem peringatan dini karhutla yang modular.

## Arsitektur Sistem

<p align="center">
  <img src="docs/images/system-architecture.png" alt="Diagram arsitektur sistem RimbaLink" width="90%">
</p>

**Gambaran alur sistem:**

1. **Node pemantauan** membaca kondisi lingkungan melalui sensor yang terpasang.
2. **Mikrokontroler ESP32** mengolah data awal dan menyiapkan data untuk dikirim.
3. **Modul LoRa** mengirimkan data dari node menuju gateway.
4. **Gateway** menerima data dari node dan menjadi penghubung menuju sistem pemantauan.
5. **Modul kamera ESP32-CAM** mendukung pengambilan gambar untuk verifikasi visual sesuai skenario implementasi.
6. **Sistem pemantauan** mengolah dan menyajikan informasi, termasuk visualisasi lokasi, peta risiko, dan prakiraan apabila modul tersebut telah diintegrasikan.

## Pengembangan Hardware

RimbaLink dikembangkan dengan pendekatan modular. Pemisahan fungsi ke beberapa bagian PCB ditujukan agar perancangan, pengujian, dan perawatan perangkat lebih mudah dilakukan.

### 1. Desain Skematik

Bagian ini memuat dokumentasi hubungan rangkaian elektronik setiap modul.

| Modul | Dokumentasi |
|---|---|
| Node | ![Schematic Node](docs/images/schematic-node.png) |
| Gateway | ![Schematic Gateway](docs/images/schematic-gateway.png) |
| Rangkaian keseluruhan | ![Overall schematic](docs/images/schematic-overall.png) |

### 2. Desain PCB

Layout PCB dirancang untuk mengintegrasikan komponen pada setiap bagian sistem.

| Modul | Layout PCB |
|---|---|
| Node | ![PCB Node](docs/images/pcb-node.png) |
| Gateway | ![PCB Gateway](docs/images/pcb-gateway.png) |
| LoRa | ![PCB LoRa](docs/images/pcb-lora.png) |
| ESP32-CAM | ![PCB ESP32-CAM](docs/images/pcb-espcam.png) |
| Power | ![PCB Power](docs/images/pcb-power.png) |

### 3. Visualisasi 3D PCB

Render 3D digunakan untuk memeriksa penempatan komponen dan gambaran fisik PCB sebelum proses fabrikasi.

| Modul | Render 3D |
|---|---|
| Node | ![3D Node](docs/images/3d-node.png) |
| Gateway | ![3D Gateway](docs/images/3d-gateway.png) |
| LoRa | ![3D LoRa](docs/images/3d-lora.png) |
| ESP32-CAM | ![3D ESP32-CAM](docs/images/3d-espcam.png) |
| Power | ![3D Power](docs/images/3d-power.png) |

### 4. Realisasi PCB

Dokumentasi berikut disiapkan untuk menampilkan hasil PCB yang telah dibuat. Foto sisi depan dan belakang dapat ditambahkan setelah proses perakitan atau fabrikasi.

<details>
<summary><strong>Node</strong></summary>

| Tampak depan | Tampak belakang |
|---|---|
| ![Node front](docs/images/real-node-front.png) | ![Node back](docs/images/real-node-back.png) |

</details>

<details>
<summary><strong>Gateway</strong></summary>

| Tampak depan | Tampak belakang |
|---|---|
| ![Gateway front](docs/images/real-gateway-front.png) | ![Gateway back](docs/images/real-gateway-back.png) |

</details>

<details>
<summary><strong>LoRa</strong></summary>

| Tampak depan | Tampak belakang |
|---|---|
| ![LoRa front](docs/images/real-lora-front.png) | ![LoRa back](docs/images/real-lora-back.png) |

</details>

<details>
<summary><strong>ESP32-CAM</strong></summary>

| Tampak depan | Tampak belakang |
|---|---|
| ![ESP32-CAM front](docs/images/real-espcam-front.png) | ![ESP32-CAM back](docs/images/real-espcam-back.png) |

</details>

<details>
<summary><strong>Power</strong></summary>

| Tampak depan | Tampak belakang |
|---|---|
| ![Power front](docs/images/real-power-front.png) | ![Power back](docs/images/real-power-back.png) |

</details>

## Fitur dan Komponen Utama

| Bagian | Peran dalam sistem |
|---|---|
| ESP32 | Pengendali node dan pemrosesan data sensor |
| Sensor lingkungan | Mengukur parameter kondisi lingkungan sesuai sensor yang dipasang |
| LoRa | Komunikasi data jarak jauh antara node dan gateway |
| ESP32-CAM | Mendukung dokumentasi dan verifikasi visual |
| Sistem tenaga surya | Mendukung suplai energi untuk perangkat lapangan |
| Gateway | Menerima data node dan meneruskannya ke sistem pemantauan |
| Peta risiko dan prakiraan | Konteks visualisasi dan analisis informasi pemantauan |

## Alur Kerja

```mermaid
flowchart TD
    A[Node Sensor] --> B[ESP32 membaca data]
    B --> C[Pengiriman melalui LoRa]
    C --> D[Gateway menerima data]
    D --> E[Sistem pemantauan]
    E --> F[Visualisasi data dan lokasi]
    B -. indikasi perlu verifikasi .-> G[ESP32-CAM]
    G --> D
    E --> H[Peta risiko dan prakiraan]
```

*Diagram ini merupakan gambaran konseptual alur sistem; alur final mengikuti implementasi firmware dan integrasi perangkat.*

## Teknologi dan Perangkat

- **Mikrokontroler:** ESP32
- **Komunikasi nirkabel:** LoRa
- **Kamera:** ESP32-CAM
- **Catu daya:** Sistem tenaga surya
- **Perancangan elektronik:** Schematic dan PCB design
- **Firmware:** Embedded C/C++ (sesuaikan dengan implementasi pada repository)

## Struktur Dokumentasi yang Disarankan

Simpan dokumentasi visual proyek pada folder berikut agar tautan gambar di README dapat ditampilkan oleh GitHub.

```text
RimbaLink/
├── README.md
└── docs/
    └── images/
        ├── rimbalink-banner.png
        ├── system-architecture.png
        ├── schematic-node.png
        ├── schematic-gateway.png
        ├── schematic-overall.png
        ├── pcb-node.png
        ├── pcb-gateway.png
        ├── pcb-lora.png
        ├── pcb-espcam.png
        ├── pcb-power.png
        ├── 3d-node.png
        ├── 3d-gateway.png
        ├── 3d-lora.png
        ├── 3d-espcam.png
        ├── 3d-power.png
        └── real-*.png
```

Nama file gambar pada README harus sama persis dengan nama file yang diunggah ke repository.

## Status Pengembangan

| Tahapan | Status |
|---|---|
| Perancangan arsitektur sistem | Dalam pengembangan |
| Perancangan skematik dan PCB | Dalam pengembangan |
| Integrasi node dan komunikasi LoRa | Perlu diperbarui sesuai hasil implementasi |
| Integrasi gateway dan kamera | Perlu diperbarui sesuai hasil implementasi |
| Integrasi dashboard, peta risiko, dan prakiraan | Perlu diperbarui sesuai hasil implementasi |
| Pengujian lapangan | Perlu diperbarui sesuai hasil pengujian |

## Kontribusi

Saran, diskusi teknis, dan kontribusi untuk pengembangan RimbaLink sangat terbuka. Silakan buat *issue* untuk melaporkan masalah atau mengusulkan fitur, maupun *pull request* untuk mengajukan perubahan.

## Lisensi

Lisensi proyek belum dicantumkan. Tambahkan file `LICENSE` dan perbarui bagian ini setelah lisensi proyek ditentukan.

---

<p align="center">
  <strong>RIMBALINK</strong><br>
  <em>Technology for smarter environmental monitoring.</em>
</p>
