# 🚦 Traffic Sign Detection — Jetson Nano + TensorRT YOLOv8 + GPIO LEDs

Sistem deteksi rambu lalu lintas **real-time** berbasis **Jetson Nano** menggunakan
kamera CSI **IMX219**, model **YOLOv8** yang sudah dikonversi ke **TensorRT engine**,
serta indikator **3 LED** (STOP / SLOW / GO) yang dikendalikan melalui GPIO.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📌 Ringkasan

Program menerima frame dari kamera CSI, menjalankan inferensi YOLOv8 pada GPU
Jetson melalui TensorRT, lalu mengklasifikasikan rambu ke dalam 3 kelas:
`stop`, `slow`, `go`. Kelas dengan confidence tertinggi yang **stabil selama
beberapa frame berturut-turut** akan menyalakan LED yang sesuai.

Model sudah tersedia dalam bentuk **`best.engine`** (TensorRT) — cukup diletakkan
di folder `models/` dan program langsung siap dijalankan.

---

## 🧰 Spesifikasi Hardware

| Komponen              | Spesifikasi                                        |
|-----------------------|----------------------------------------------------|
| Board                 | NVIDIA Jetson Nano (4 GB)                          |
| Kamera                | Raspberry Pi Camera v2 (IMX219) via CSI            |
| LED Indikator         | 3 × LED (Merah = STOP, Kuning = SLOW, Hijau = GO)  |
| Resistor              | 3 × 330 Ω (current limiter)                        |
| Kabel Jumper          | Female-to-female / male-to-female                  |
| Power Supply          | 5 V ⎓ 4 A (barrel jack)                            |
| Storage               | microSD 32 GB+ (JetPack image)                     |

### Pemetaan Pin (GPIO.BOARD)

| Fungsi | Pin Fisik | Warna LED |
|--------|-----------|-----------|
| STOP   | 11        | Merah     |
| SLOW   | 13        | Kuning    |
| GO     | 15        | Hijau     |
| 3.3 V  | 1         | VCC LED (active-low) |
| GND    | 6 / 9     | Ground    |

> ⚠️ **Konfigurasi Active-LOW** — pin GPIO `LOW` menyalakan LED, `HIGH` mematikan.

---

## 💻 Spesifikasi Software

| Komponen         | Versi / Keterangan                          |
|------------------|---------------------------------------------|
| OS               | NVIDIA JetPack 4.6.x (Ubuntu 18.04, L4T)    |
| Python           | 3.6                                         |
| CUDA             | 10.2 (bawaan JetPack)                       |
| TensorRT         | 8.x (bawaan JetPack)                        |
| OpenCV           | 4.x (`python3-opencv`)                      |
| GStreamer        | 1.0 + plugin NVIDIA (`nvarguscamerasrc`)    |
| NumPy            | 1.19+                                       |
| Jetson.GPIO      | `python3-jetson-gpio`                       |
| PyGObject (gi)   | `python3-gi`, `gir1.2-gstreamer-1.0`        |

---

## ⚙️ Cara Kerja Sistem

```text
┌──────────────┐   NVMM    ┌──────────────┐   BGR    ┌──────────────┐
│  Kamera CSI  │ ────────▶ │  GStreamer   │ ───────▶ │  Preprocess  │
│   IMX219     │           │  Pipeline    │          │  (320×320)   │
└──────────────┘           └──────────────┘          └──────┬───────┘
                                                            │
                                                            ▼
┌──────────────┐   hasil   ┌──────────────┐   tensor  ┌──────────────┐
│  GPIO LEDs   │ ◀──────── │  Postprocess │ ◀──────── │  TensorRT    │
│ STOP/SLOW/GO │  deteksi  │  + NMS       │   output  │  YOLOv8      │
└──────────────┘           └──────────────┘           └──────────────┘
```

1. **Capture** — `nvarguscamerasrc` mengambil frame dari sensor IMX219.
2. **Preprocess** — resize letterbox ke 320×320, normalisasi, ubah ke CHW & RGB.
3. **Inference** — engine TensorRT dijalankan via `context.execute_v2()`.
4. **Postprocess** — decode output YOLOv8, filter confidence, NMS per kelas.
5. **Stability Check** — kelas harus stabil `STABLE_FRAMES` kali berturut-turut.
6. **GPIO Actuation** — LED kelas terpilih dinyalakan, lainnya dimatikan.
7. **Cleanup** — saat keluar, semua LED dimatikan & memori GPU dibebaskan.

---

## 🔌 Wiring LED

Detail lengkap ada di [`docs/wiring.md`](docs/wiring.md).

**Rangkaian LED Active-LOW (per LED):**

```
   3.3V (pin 1) ──▶ Resistor 330Ω ──▶ Anoda LED ──▶ Katoda ──▶ GPIO pin
                                                               (11/13/15)
```

- GPIO output `LOW`  → **LED menyala**
- GPIO output `HIGH` → **LED mati**

> Lihat panduan wiring di terminal:
> ```bash
> python3 src/traffic_sign_jetson.py --wiring
> ```

---

## 🛠️ Instalasi

### 1. Persiapan JetPack

Flash Jetson Nano dengan JetPack 4.6.x (mengandung CUDA 10.2 + TensorRT 8).
Setelah booting, jalankan:

```bash
sudo apt update && sudo apt upgrade
```

### 2. Clone repositori

```bash
git clone https://github.com/Furohito/Jetson_Nano.git
cd Jetson_Nano
```

### 3. Instal dependencies

```bash
sudo apt install -y python3-numpy python3-opencv python3-gi \
    gir1.2-gstreamer-1.0 gir1.2-gst-plugins-base-1.0 \
    python3-jetson-gpio python3-libnvinfer
```

Atau pakai `requirements.txt`:

```bash
pip3 install -r requirements.txt
```

### 4. Siapkan model

Letakkan file `best.engine` (sudah tersedia) pada:

```
~/traffic-sign-project/models/best.engine
```

Atau sesuaikan path-nya di `src/traffic_sign_jetson.py` pada variabel `ENGINE_PATH`.

### 5. Periksa lingkungan

```bash
python3 src/traffic_sign_jetson.py --check
```

Output yang diharapkan: semua `[OK]` tanpa `[ERR]`.

---

## ▶️ Menjalankan Program

### Mode deteksi penuh (LED + kamera + TensorRT)

```bash
python3 src/traffic_sign_jetson.py
```

Tekan `q` pada window untuk keluar, atau `Ctrl+C`.

### Mode uji LED saja (tanpa kamera & TensorRT)

```bash
python3 src/traffic_sign_jetson.py --test-led
python3 src/traffic_sign_jetson.py --test-led --pin 11       # uji 1 pin
python3 src/traffic_sign_jetson.py --test-led --repeat 0     # loop terus
python3 src/traffic_sign_jetson.py --test-led --off-only     # matikan semua
```

### Mode tanpa preview (headless)

```bash
python3 src/traffic_sign_jetson.py --no-preview
python3 src/traffic_sign_jetson.py --no-preview --max-frames 300
```

### Argumen CLI Lengkap

| Argumen         | Deskripsi                                          |
|-----------------|----------------------------------------------------|
| `--check`       | Periksa library, kamera, dan model                 |
| `--test-led`    | Uji LED tanpa inferensi                            |
| `--wiring`      | Tampilkan panduan wiring                           |
| `--active-low`  | LED menyala saat LOW (default)                     |
| `--active-high` | LED menyala saat HIGH                              |
| `--pin N`       | Uji satu pin (11/13/15), hanya dengan `--test-led` |
| `--hold S`      | Durasi tiap step uji LED (default 2.0 s)           |
| `--repeat N`    | Jumlah siklus uji LED, `0` = terus (default 1)     |
| `--off-only`    | Uji dengan semua LED mati saja                     |
| `--no-preview`  | Jalankan tanpa jendela video                       |
| `--max-frames N`| Berhenti setelah N frame                           |

---

## 📂 Struktur Repositori

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── src/
│   └── traffic_sign_jetson.py     # source code utama
├── models/
│   ├── README.md                  # info model
│   └── best.engine                # TensorRT engine (siap pakai)
├── docs/
│   ├── arsitektur.md              # arsitektur sistem
│   ├── wiring.md                  # dokumentasi rangkaian
│   └── pengujian.md               # hasil pengujian
├── images/                        # gambar & foto
│   ├── arsitektur.png
│   ├── wiring.png
│   ├── dokumentasi-praktikum.jpg
│   ├── screenshot-deteksi.png
│   └── screenshot-led-test.png
└── test-results/
    └── log-pengujian.txt          # log output program
```

---

## 📸 Dokumentasi Visual

| File | Deskripsi |
|------|-----------|
| `images/arsitektur.png`            | Diagram arsitektur sistem |
| `images/wiring.png`                | Skema rangkaian LED + Jetson |
| `images/dokumentasi-praktikum.jpg` | Foto setup Jetson + kamera + LED |
| `images/screenshot-deteksi.png`    | Screenshot deteksi rambu berjalan |
| `images/screenshot-led-test.png`   | Screenshot uji LED |

---

## 🧪 Hasil Pengujian

Ringkasan hasil pengujian dan benchmark ada di
[`docs/pengujian.md`](docs/pengujian.md) dan log mentah di
[`test-results/log-pengujian.txt`](test-results/log-pengujian.txt).

---

## 📜 Lisensi

MIT License — lihat file [`LICENSE`](LICENSE).

---

## 👥 Tim Pengembang

| No | Nama                        | NIM        |
|----|-----------------------------|------------|
| 1  | Abdurrahman Hakim Makarim   | 2023071010 |
| 2  | Nurfajrian Abdurrohman      | 2023071045 |
| 3  | Zaky Naufal Saleh           | 20240708   |
| 4  | Ziyan Faiq                  | 2024071014 |
| 5  | Daffa Rinaldi               | 2024071024 |