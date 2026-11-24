<p align="center">
  <img src="gambar/RimbaLink.png" alt="RimbaLink banner" width="85%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/MCU-ESP32-E7352C?style=flat-square&logo=espressif&logoColor=white" alt="ESP32">
  <img src="https://img.shields.io/badge/CAMERA-ESP32--CAM-374151?style=flat-square&logoColor=white" alt="ESP32-CAM">
  <img src="https://img.shields.io/badge/WIRELESS-LoRa-7C3AED?style=flat-square&logoColor=white" alt="LoRa">
  <a href="https://easyeda.com/"><img src="https://img.shields.io/badge/PCB%20DESIGN-EasyEDA-1769FF?style=flat-square&logo=easyeda&logoColor=white" alt="Designed with EasyEDA"></a>
  <img src="https://img.shields.io/badge/FIRMWARE-C%2FC%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white" alt="Embedded C/C++">
</p>

<p align="center">
  <img src="https://img.shields.io/github/last-commit/0x-irfan/RimbaLink?style=flat-square&logo=github&label=Last%20update" alt="Last commit">
  <img src="https://img.shields.io/github/repo-size/0x-irfan/RimbaLink?style=flat-square&logo=github&label=Repository%20size" alt="Repository size">
  <img src="https://img.shields.io/badge/STATUS-IN_DEVELOPMENT-F97316?style=flat-square" alt="In development">
</p>

---
Akan opensource pada waktunya🗿Klo lolos pendanaan

### 1. Desain Skematik

| Desain Schematic GateWay Dan Node |
|---|
| <p align="center"><img src="gambar/GW&NODE_SCH.png" alt="Overall schematic" width="720"></p> |

### 2. Desain PCB

Layout PCB disusun berdampingan agar perbandingan setiap desain lebih mudah dilihat.

<table>
  <tr>
    <td align="center"><strong>Node</strong><br><img src="gambar/NODE_PCB.png" alt="PCB Node" width="300"></td>
    <td align="center"><strong>Gateway</strong><br><img src="gambar/GW_PCB.png" alt="PCB Gateway" width="300"></td>
  </tr>
  <tr>
    <td align="center"><strong>LoRa</strong><br><img src="gambar/LORA_PCB.png" alt="PCB LoRa" width="300"></td>
    <td align="center"><strong>Power</strong><br><img src="gambar/POWER_PCB.png" alt="PCB Power" width="300"></td>
  </tr>
  <tr>
    <td align="center" colspan="2"><strong>ESP32-CAM</strong><br><img src="gambar/ESPCAM_PCB.png" alt="PCB ESP32-CAM" width="300"></td>
  </tr>
</table>

### 3. Visualisasi 3D PCB

Render 3D disusun berdampingan untuk memudahkan perbandingan bentuk dan penempatan komponen.

<table>
  <tr>
    <td align="center"><strong>Node</strong><br><img src="docs/images/3d-node.png" alt="3D Node" width="280"></td>
    <td align="center"><strong>Gateway</strong><br><img src="docs/images/3d-gateway.png" alt="3D Gateway" width="280"></td>
  </tr>
  <tr>
    <td align="center"><strong>LoRa</strong><br><img src="docs/images/3d-lora.png" alt="3D LoRa" width="280"></td>
    <td align="center"><strong>Power</strong><br><img src="docs/images/3d-power.png" alt="3D Power" width="280"></td>
  </tr>
  <tr>
    <td align="center" colspan="2"><strong>ESP32-CAM</strong><br><img src="docs/images/3d-espcam.png" alt="3D ESP32-CAM" width="280"></td>
  </tr>
</table>

### 4. Realisasi PCB

Dokumentasi berikut disiapkan untuk menampilkan hasil PCB yang telah dibuat. Foto sisi depan dan belakang dapat ditambahkan setelah proses perakitan atau fabrikasi.

<details>
<summary><strong>Node</strong></summary>

<p align="center">
  <img src="docs/images/real-node-front.png" alt="Node front" width="220">
  <img src="docs/images/real-node-back.png" alt="Node back" width="220">
</p>

<p align="center"><sub>Tampak depan · Tampak belakang</sub></p>

</details>

<details>
<summary><strong>Gateway</strong></summary>

<p align="center">
  <img src="docs/images/real-gateway-front.png" alt="Gateway front" width="220">
  <img src="docs/images/real-gateway-back.png" alt="Gateway back" width="220">
</p>

<p align="center"><sub>Tampak depan · Tampak belakang</sub></p>

</details>

<details>
<summary><strong>LoRa</strong></summary>

<p align="center">
  <img src="docs/images/real-lora-front.png" alt="LoRa front" width="220">
  <img src="docs/images/real-lora-back.png" alt="LoRa back" width="220">
</p>

<p align="center"><sub>Tampak depan · Tampak belakang</sub></p>

</details>

<details>
<summary><strong>ESP32-CAM</strong></summary>

<p align="center">
  <img src="docs/images/real-espcam-front.png" alt="ESP32-CAM front" width="220">
  <img src="docs/images/real-espcam-back.png" alt="ESP32-CAM back" width="220">
</p>

<p align="center"><sub>Tampak depan · Tampak belakang</sub></p>

</details>

<details>
<summary><strong>Power</strong></summary>

<p align="center">
  <img src="docs/images/real-power-front.png" alt="Power front" width="220">
  <img src="docs/images/real-power-back.png" alt="Power back" width="220">
</p>

<p align="center"><sub>Tampak depan · Tampak belakang</sub></p>

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
