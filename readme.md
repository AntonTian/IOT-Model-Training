# 🗑️ SmartBin IoT Time-Series Predictor

Sebuah sistem *Internet of Things* (IoT) cerdas yang memprediksi sisa waktu (dalam jam) sebelum tempat sampah penuh. Proyek ini memadukan perangkat keras ESP32, model regresi *Machine Learning* (LightGBM), dan infrastruktur *Cloud* untuk memberikan visibilitas *real-time* dan analitik historis.

---

## 🏗️ Arsitektur Sistem (End-to-End)

Proyek ini dibangun menggunakan arsitektur berbasis *polling* dan *microservices*:
1. **Perangkat Keras (ESP32 + Sensor Ultrasonik):** Membaca jarak tumpukan sampah dan mengirimkannya ke Blynk IoT Cloud.
2. **Dashboard Real-Time (Blynk):** Antarmuka pengguna (*User Interface*) untuk melihat kapasitas saat ini, status tombol *pickup* (pengangkutan), dan prediksi waktu penuh.
3. **Otak Prediksi (Python Backend di Hugging Face):** Mengambil data dari Blynk via REST API, memproses *Feature Engineering* (Lag & Cyclical Time), menjalankan inferensi model AI, dan mengirim hasilnya kembali ke Blynk.
4. **Data Logger (InfluxDB Cloud):** Menyimpan seluruh data mentah dan hasil prediksi dalam format *Time-Series* untuk keperluan analisis masa lalu (*Historical Trends*).

---

## 📊 Performa Model AI

Model utama menggunakan **LightGBM Regressor** yang dioptimasi menggunakan metode Bayesian Optimization (Optuna). 

Untuk mencegah kebocoran data (*Data Leakage*), model dievaluasi menggunakan metode **Temporal Split** (Belajar dari masa lalu, diuji di masa depan) dikombinasikan dengan `GroupKFold` untuk memastikan model tangguh di berbagai lokasi tempat sampah yang berbeda.

**Spesifikasi Evaluasi Final (16 Hari Dataset):**
*   **Best Cross-Validation MAE:** 1.6717%
*   **Overall Test MAE:** 1.5928%
*   **High-Fill Test MAE (Kapasitas $\ge 75\%$):** 1.6012%
*   **Overall RMSE:** 2.0123%

> *Catatan: Error ~1.5% - 2% setara dengan deviasi ~1.5 - 2 cm pada dunia nyata, yang mana sudah menyentuh batas akurasi fisik sensor ultrasonik itu sendiri.*

---

## ⚙️ Persiapan & Instalasi

Pastikan Anda memiliki Python 3.9+ terinstal di sistem Anda. Disarankan menggunakan *Virtual Environment* (`venv` atau `conda`).

```bash
# 1. Clone repositori ini
git clone https://github.com/username-anda/smartbin-iot-predictor.git
cd smartbin-iot-predictor

# 2. Buat dan aktifkan Virtual Environment
python -m venv venv

# Windows:
.\\venv\\Scripts\\activate
# macOS/Linux:
source venv/bin/activate

# 3. Instal semua dependensi
pip install -r requirements.txt
```

---

## 📈 Tutorial: Menjalankan Optuna Dashboard

Kami menggunakan `optuna-dashboard` untuk memvisualisasikan sejarah *hyperparameter tuning* secara saintifik. Seluruh riwayat percobaan tersimpan dengan aman di dalam *database* SQLite lokal.

Untuk melihat grafik konvergensi dan tingkat kepentingan parameter (*Hyperparameter Importances*):

1. Pastikan *Virtual Environment* Anda **aktif** (muncul tanda `(venv)` di terminal).
2. Jalankan perintah berikut di terminal:
   ```bash
   optuna-dashboard sqlite:///optuna_smartbin_study.sqlite3
   ```
3. Buka *browser* Anda dan kunjungi tautan yang muncul (biasanya: `http://127.0.0.1:8080/`).
4. **Penting untuk Laporan:** Saat melihat bagan *History*, pastikan Anda menyalakan opsi **Log y scale** di sudut kanan atas agar detail optimasi (*micro-improvements*) algoritma TPE terlihat dengan jelas.

---

## 🚀 Cara Menjalankan Backend (Local / Hugging Face)

Skrip `app.py` menjalankan *Web Server* Flask (untuk tetap hidup di Hugging Face) dan *Background Thread* (untuk *polling* IoT).

1. Buka `app.py` dan pastikan Kredensial API sudah terisi:
   - `BLYNK_AUTH_TOKEN`
   - `INFLUX_URL`, `INFLUX_TOKEN`, `INFLUX_ORG`, `INFLUX_BUCKET`
2. Jalankan skrip:
   ```bash
   python app.py
   ```
3. Skrip akan otomatis berjalan 24/7, mengambil data dari Blynk, melakukan prediksi dengan `lightgbm_smartbin_16days_final.pkl`, dan mengarsipkannya ke InfluxDB.