# Hasil Pengujian

## 1. Uji Lingkungan (`--check`)

```bash
python3 src/traffic_sign_jetson.py --check
```

**Output:**

```
Python: 3.6.9 | CPU: aarch64
[OK] numpy: 1.19.5
[OK] cv2: 4.5.4
[OK] gi: terpasang
[OK] Jetson.GPIO: 2.0.17
[OK] tensorrt: 8.2.1.8
Info Jetson.GPIO: {'MODEL': 'Jetson Nano', 'JETPACK': '4.6.1'}
Izin export GPIO: True
[OK] Pipeline kamera dan engine TensorRT tersedia.
```

## 2. Uji LED (`--test-led`)

```bash
python3 src/traffic_sign_jetson.py --test-led --hold 1.0 --repeat 3
```

**Output:**

```
Uji LED: ACTIVE-LOW
SEMUA MATI | GPIO {11: 1, 13: 1, 15: 1}
STOP       | GPIO {11: 0, 13: 1, 15: 1}
SLOW       | GPIO {11: 1, 13: 0, 15: 1}
GO         | GPIO {11: 1, 13: 1, 15: 0}
SEMUA MATI | GPIO {11: 1, 13: 1, 15: 1}
Uji level GPIO selesai.
Perintah LED MATI telah dikirim.
```

📸 Lihat `images/screenshot-led-test.png`.

## 3. Uji Deteksi Real-time

```bash
python3 src/traffic_sign_jetson.py --no-preview --max-frames 1000
```

**Ringkasan hasil pada 1000 frame:**

| Metrik                     | Nilai          |
|----------------------------|----------------|
| Total frame diproses       | 1000           |
| Rata-rata FPS              | 18.4 FPS       |
| Latensi inferensi (mean)   | 42 ms          |
| Deteksi benar (STOP)       | 96.2 %         |
| Deteksi benar (SLOW)       | 91.5 %         |
| Deteksi benar (GO)         | 94.8 %         |
| False positive             | 3.1 %          |
| LED aktif < 0.5 s          | ✅ (stabil 3 frame) |

## 4. Benchmark Inferensi

| Presisi Engine | Latensi (ms) | FPS   | Memori GPU |
|----------------|--------------|-------|------------|
| FP32           | 78           | 12.8  | 320 MB     |
| **FP16**       | **42**       | **18.4** | 210 MB  |
| INT8           | 27           | 28.1  | 150 MB     |

> Engine yang dipakai di produksi: **FP16**.

## 5. Uji Ketahanan LED

| Skenario                       | Hasil             |
|--------------------------------|-------------------|
| Program keluar normal (`q`)    | LED semua mati ✅ |
| Program dihentikan (`Ctrl+C`)  | LED semua mati ✅ |
| Program error (frame drop)     | LED semua mati ✅ |
| Kamera dicabut saat berjalan   | LED semua mati ✅ |

## 6. Foto & Screenshot

| File | Keterangan |
|------|------------|
| `images/dokumentasi-praktikum.jpg` | Foto setup Jetson + kamera + LED saat praktikum |
| `images/screenshot-deteksi.png`   | Deteksi STOP dengan bounding box |
| `images/screenshot-led-test.png`  | LED menyala saat uji |

## 7. Log Mentah

Lihat [`test-results/log-pengujian.txt`](../test-results/log-pengujian.txt).