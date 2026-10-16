# Deteksi Pose Tubuh Realtime (YOLOv8 + Streamlit)

Proyek akhir mata kuliah **Jaringan Syaraf Tiruan**: aplikasi web yang mendeteksi dan mengklasifikasikan pose tubuh manusia secara realtime dari kamera, menggunakan **YOLOv8-pose** dan **Streamlit**.

![Python](https://img.shields.io/badge/Python-3.11-blue) ![Streamlit](https://img.shields.io/badge/Streamlit-1.38-red) ![YOLOv8](https://img.shields.io/badge/Ultralytics-8.3-green)

## ✨ Fitur

- 📷 **Deteksi realtime** dari kamera via WebRTC (tanpa upload file)
- 🧍 **Klasifikasi 4 pose**: Berdiri, Duduk, Jongkok, Berbaring
- 🦴 **Visualisasi** 17 keypoint + skeleton + sudut sendi (lutut & pinggul)
- 📊 **Statistik 5 detik terakhir** dengan progress bar per pose
- ⚙️ **Pengaturan** confidence threshold, toggle skeleton/keypoint/sudut
- ⚡ **Inference thread terpisah** agar stream tetap halus
- 🌐 **TURN server Twilio** untuk koneksi WebRTC yang stabil lintas jaringan

## 🧠 Cara Kerja

1. Kamera mengirim frame ke browser via WebRTC.
2. Frame dikirim ke **thread inference** terpisah, diproses dengan **YOLOv8n-pose** (17 keypoint COCO, input 320px).
3. `pose_classifier.py` menghitung sudut lutut, pinggul, dan kemiringan badan, lalu **voting multi-sinyal** menentukan pose.
4. Frame teranotasi dikirim balik; dashboard menampilkan pose, sudut, jumlah orang, FPS, dan statistik 5 detik.

## 🛠️ Tech Stack

| Komponen | Teknologi |
|---|---|
| Web app | Streamlit |
| Deteksi pose | Ultralytics YOLOv8n-pose |
| Streaming kamera | streamlit-webrtc + aiortc |
| Video frame | PyAV / OpenCV |
| Klasifikasi pose | Heuristik geometri (sudut sendi + voting) |

## 📁 Struktur Proyek

```
DETEKSI-POSE-WITH-YOLO-tes/
├── app.py               # Aplikasi Streamlit utama
├── pose_classifier.py   # Klasifikasi pose (sudut sendi + voting)
├── requirements.txt     # Dependensi Python
├── yolov8n-pose.pt      # Model YOLOv8 pose (di-download otomatis bila tidak ada)
├── .devcontainer/       # Konfigurasi GitHub Codespaces
└── README.md
```

## 🚀 Menjalankan Lokal

**1. Clone & masuk folder**
```bash
git clone <url-repo-anda> && cd DETEKSI-POSE-WITH-YOLO-tes
```

**2. Setup environment**
```bash
python -m venv .venv
.venv\Scripts\activate        # Windows
# source .venv/bin/activate   # macOS / Linux

pip install -r requirements.txt
```

**3. Konfigurasi Twilio (wajib)**

Buat file `.streamlit/secrets.toml` di root proyek:

```toml
TWILIO_ACCOUNT_SID = "ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
TWILIO_AUTH_TOKEN  = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

> Tanpa credentials Twilio, app tetap berjalan dengan STUN server publik sebagai cadangan, tapi koneksi WebRTC bisa kurang stabil di jaringan tertentu (NAT/Corporate firewall). SID & token Twilio gratis diambil dari [console.twilio.com](https://console.twilio.com).

**4. Jalankan**
```bash
streamlit run app.py
```

Buka `http://localhost:8501`, izinkan akses kamera, tekan **Start**.

## ☁️ Deploy ke Web Portfolio

App butuh **koneksi kamera via WebRTC** — karenanya deploy ke **Streamlit Community Cloud** lebih disarankan daripada embedding di website statis (HTTPS wajib agar kamera browser berfungsi).

1. Push repo ke GitHub.
2. Buka [share.streamlit.io](https://share.streamlit.io) → **New app** → pilih repo & `app.py`.
3. Di **Settings → Secrets**, tambahkan `TWILIO_ACCOUNT_SID` dan `TWILIO_AUTH_TOKEN`.
4. Deploy. App otomatis ter-deploy ulang setiap push.

> **Catatan untuk web portfolio:** kalau mau *embed* langsung ke halaman portfolio, gunakan `st.set_page_config(page_title=...)` yang sudah ada, lalu taruh app di iframe `<iframe src="https://<app>.streamlit.app/?embed=true">`. Tanpa HTTPS, kamera tidak akan aktif di browser.

Alternatif (butuh container): GitHub Codespaces sudah dikonfigurasi di `.devcontainer/`.

## 🧪 Model & Dataset

- **Model:** YOLOv8n-pose (COCO Keypoints, 17 keypoint)
- **Klasifikasi:** 4 kelas pose berdasar sudut sendi (lutut, pinggul) + kemiringan torso + posisi relatif hip–knee
- **Threshold keypoint minimum:** 4 titik wajib terdeteksi, skor voting minimum 3

## 📜 Lisensi

Gunakan untuk keperluan edukasi / portofolio. Model YOLOv8 berlisensi [AGPL-3.0](https://github.com/ultralytics/ultralytics/blob/main/LICENSE).
