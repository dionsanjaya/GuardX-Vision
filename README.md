# Entity-GuardX-Vision
# Sistem Pengenalan Wajah dan Plat Nomor untuk Pengendalian Akses

Proyek ini mengembangkan sistem berbasis **computer vision** untuk pengendalian akses dan analisis data visual. Sistem mencakup empat komponen utama:
1. **Deteksi Wajah untuk Buka Pintu**: Menggunakan pengenalan wajah untuk membuka pintu secara otomatis.
2. **Deteksi Plat Nomor untuk Buka Pintu**: Menggunakan pengenalan plat nomor kendaraan untuk akses pintu/gate.
3. **Deteksi Wajah + Counting**: Menghitung jumlah wajah unik dalam video untuk analisis kerumunan.
4. **Membedakan Gender, Counting, dan Umur**: Mengklasifikasi gender, menghitung jumlah per kategori, dan memperkirakan umur.

Dokumen ini menyediakan panduan untuk dua skenario: **prototyping** (skala kecil, menggunakan **Orange Pi 3 LTS**) dan **skala besar** (kantoran/industri, menggunakan **PC server** dengan beberapa **Orange Pi** sebagai perangkat edge). Juga dibahas opsi penggunaan **Orange Pi** hanya sebagai **trigger** dan alternatif perangkat dengan spesifikasi lebih rendah.

---

## Versi 1: Kebutuhan Prototyping
Versi ini dirancang untuk pengembangan skala kecil, cocok untuk eksperimen atau uji coba di lingkungan terbatas (misal, satu pintu atau gate).

### Tujuan
- Membuat prototipe sistem pengendalian akses berbasis wajah dan plat nomor.
- Mendeteksi wajah tunggal secara real-time (~0.5-1 detik) untuk buka pintu (database 5-10 wajah).
- Membaca plat nomor dengan akurasi >85%.
- Menghitung wajah unik dan mengklasifikasi gender/umur untuk analisis sederhana.
- Memberikan umpan balik suara (espeak) dan visual (LED) kepada pengguna.
- Menangani error seperti banyak wajah atau gambar buram.

### Prasyarat
- **Perangkat Lunak**:
  - Ubuntu Server 22.04 (ARM64) terpasang di Orange Pi 3 LTS.
  - Python 3.8+ (`python3`, `pip3`).
  - OpenCV (`python3-opencv`) untuk deteksi wajah/plat.
  - dlib (`pip install dlib`) untuk pengenalan wajah.
  - EasyOCR (`pip install easyocr`) untuk pengenalan plat nomor.
  - espeak (`espeak`) untuk notifikasi suara.
  - OPi.GPIO (`pip install OPi.GPIO`) untuk kontrol relay/LED.
  - Opsional: DeepFace (`pip install deepface`) untuk klasifikasi gender/umur.
- **Keterampilan**:
  - Pemrograman Python dasar (variabel, loop, fungsi).
  - Familiar dengan terminal Linux (SSH, instalasi paket).
  - Pemahaman dasar computer vision (deteksi objek, embeddings).
- **Dependensi**:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install build-essential cmake libopenblas-dev liblapack-dev libx11-dev libgtk-3-dev python3-opencv espeak v4l-utils -y
  pip3 install dlib easyocr OPi.GPIO
  ```

### Perangkat yang Dibutuhkan
- **Orange Pi 3 LTS (atau cari versi Orange Pi terbaru penerus 3LTS)**:
  - Quad-core Cortex-A53, 2GB RAM, 8GB eMMC.
  - Catu daya: 5V 3A (USB-C).
  - Heatsink disarankan (~Rp 20.000) untuk mencegah panas berlebih.
- **USB Webcam**:
  - Resolusi 720p, UVC-compliant (misal, Logitech C270, ~Rp 200.000-300.000).
  - Terhubung via port USB.
- **Modul Relay**:
  - Relay 5V satu kanal (~Rp 50.000) untuk simulasi kunci pintu.
  - Terhubung ke pin GPIO (misal, pin 18).
- **Speaker**:
  - Speaker 3.5mm atau USB (~Rp 50.000-100.000) untuk notifikasi suara.
- **LED** (opsional):
  - LED hijau (sukses), merah (gagal), kuning (proses) (~Rp 10.000-20.000).
  - Terhubung ke pin GPIO (misal, pin 22, 24, 26).
- **Jaringan** (opsional):
  - Wi-Fi atau Ethernet untuk SSH dan pembaruan.

### Instalasi
1. **Siapkan Orange Pi 3 LTS**:
   - Flash Ubuntu Server 22.04 ke eMMC atau microSD menggunakan Balena Etcher.
   - Boot dan konfigurasi jaringan (SSH atau monitor/keyboard).
   - Perbarui sistem:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
2. **Instal Dependensi**:
   ```bash
   sudo apt install python3 python3-pip libopencv-dev python3-opencv espeak v4l-utils -y
   pip3 install dlib easyocr OPi.GPIO
   ```
3. **Tes USB Webcam**:
   - Colokkan webcam, cek deteksi:
     ```bash
     ls /dev/video*
     ```
   - Output yang diharapkan: `/dev/video0`.
4. **Unduh Haar Cascade**:
   - Ambil `haarcascade_frontalface_default.xml` dari OpenCV GitHub (`opencv/data/haarcascades`).
   - Simpan di direktori proyek (misal, `/home/user/akses_vision/haarcascade_frontalface_default.xml`).

### Penggunaan
1. **Siapkan Database**:
   - Kumpulkan 5-10 gambar wajah (JPG/PNG, ~100x100 piksel) untuk deteksi wajah.
   - Kumpulkan gambar plat nomor untuk deteksi plat.
   - Hasilkan embeddings wajah menggunakan `dlib` dan simpan ke file pickle/CSV.
2. **Jalankan Sistem**:
   - Gunakan skrip Python (misal, `main_wajah.py`) untuk:
     - Menangkap feed webcam.
     - Mendeteksi wajah (OpenCV), verifikasi (dlib), dan buka pintu (relay).
     - Mendeteksi plat nomor (EasyOCR).
     - Menghitung wajah atau mengklasifikasi gender/umur (DeepFace).
3. **Umpan Balik**:
   - Suara: "Selamat datang, pintu terbuka" atau "Akses ditolak" via espeak.
   - Visual: LED hijau (sukses), merah (gagal).

### Penanganan Error
- **Banyak Wajah**: Tolak jika >1 wajah, notifikasi: "Silakan mendekat satu per satu."
- **Wajah/Plat Buram**: Tolak jika gambar <100x100 piksel atau buram, notifikasi: "Gambar tidak jelas, silakan mendekat."
- **Timeout**: Reset setelah 5 detik, notifikasi: "Sesi habis, coba lagi."
- **Log**: Simpan percobaan ke `/var/log/akses_vision.log`.

### Catatan Performa
- Orange Pi 3 LTS (2GB RAM) cukup untuk deteksi wajah/plat nomor (~1-2 detik per frame) dan klasifikasi sederhana.
- Untuk DeepFace (gender/umur), latensi bisa ~3-5 detik. Kurangi resolusi webcam ke 640x480 jika lag.

---

## Versi 2: Kebutuhan Skala Besar (Kantoran/Industri)
Versi ini dirancang untuk lingkungan kantoran atau industri dengan banyak pintu/gate, menggunakan **PC server** untuk pemrosesan ML berat dan beberapa **Orange Pi** sebagai perangkat edge untuk deteksi lokal, trigger, atau komunikasi via **MQTT**.

### Tujuan
- Mengelola akses di banyak pintu/gate (misal, 10-50 pintu) dengan database besar (100+ wajah/plat).
- Memproses deteksi wajah/plat nomor di server untuk akurasi tinggi dan latensi rendah (<1 detik).
- Menghitung wajah unik dan mengklasifikasi gender/umur untuk analisis keamanan/komersial skala besar.
- Mengintegrasikan Orange Pi sebagai perangkat edge untuk trigger pintu atau deteksi lokal.
- Menyediakan dashboard untuk monitoring dan logging.

### Prasyarat
- **Perangkat Lunak (Server)**:
  - OS: Ubuntu 22.04/24.04 (x86_64) di PC server.
  - Python 3.8+, OpenCV, dlib, EasyOCR, DeepFace, Paho-MQTT (`pip install paho-mqtt`).
  - Mosquitto MQTT broker (`sudo apt install mosquitto`).
  - Database: SQLite untuk prototyping, PostgreSQL untuk produksi.
  - Opsional: Flask/FastAPI untuk dashboard monitoring.
- **Perangkat Lunak (Orange Pi)**:
  - Ubuntu Server 22.04 (ARM64) di setiap Orange Pi.
  - Python 3.8+, OPi.GPIO, Paho-MQTT.
  - Opsional: OpenCV untuk deteksi lokal ringan.
- **Keterampilan**:
  - Pemrograman Python tingkat menengah (API, multithreading).
  - Administrasi server Linux (konfigurasi jaringan, database).
  - Pemahaman IoT (MQTT, komunikasi client-server).
  - Pengelolaan database (SQL).
- **Dependensi (Server)**:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install python3 python3-pip libopencv-dev python3-opencv mosquitto -y
  pip3 install dlib easyocr deepface paho-mqtt flask
  ```
- **Dependensi (Orange Pi)**:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install python3 python3-pip v4l-utils -y
  pip3 install OPi.GPIO paho-mqtt
  ```

### Perangkat yang Dibutuhkan
- **PC Server**:
  - Spesifikasi minimal: Intel i5/Ryzen 5 (dukungan AVX2, AVX512), 32GB RAM, GPU (misal, Nvidia RTX 3060) untuk ML berat, NVME 500GB.
  - Jaringan: Ethernet/Wi-Fi stabil untuk komunikasi dengan Orange Pi.
- **Orange Pi (Per Pintu/Gate)**:
  - **Jika untuk deteksi lokal + trigger**:
    - Orange Pi 3 LTS + case (2GB RAM, ~Rp 800.000) untuk deteksi wajah/plat ringan dan kontrol relay.
  - **Jika hanya untuk trigger**:
    - Orange Pi Zero2 + case (512MB/1GB RAM, ~Rp 500.000) cukup untuk menerima sinyal MQTT dan mengaktifkan relay/LED.
    - Spesifikasi lebih rendah (Quad-core Cortex-A53, 512MB RAM) hemat biaya untuk banyak pintu.
  - Catu daya: 5V 3A (3 LTS) atau micro USB (Zero2).
  - Heatsink untuk 3 LTS (~Rp 20.000).
- **USB Webcam (Per Pintu/Gate)**:
  - Resolusi 1080p untuk akurasi tinggi (~Rp 300.000-500.000).
  - UVC-compliant, terhubung via USB.
- **Modul Relay**:
  - Relay 5V satu kanal (~Rp 50.000) per pintu.
  - Terhubung ke GPIO (misal, pin 18).
- **Speaker**:
  - Speaker 3.5mm/USB (~Rp 50.000-100.000) per pintu untuk notifikasi.
- **LED** (opsional):
  - LED hijau/merah/kuning (~Rp 10.000-20.000) per pintu.
- **Jaringan**:
  - Router/switch dengan bandwidth tinggi untuk koneksi server ke 10-50 Orange Pi.
  - Latensi jaringan <100ms untuk MQTT.

### Instalasi
1. **Siapkan PC Server**:
   - Instal Ubuntu 22.04/24.04, perbarui sistem:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - Instal Mosquitto:
     ```bash
     sudo apt install mosquitto -y
     ```
   - Konfigurasi Mosquitto untuk autentikasi (user/password) dan TLS (opsional).
   - Instal dependensi ML:
     ```bash
     sudo apt install python3 python3-pip libopencv-dev python3-opencv -y
     pip3 install dlib easyocr deepface paho-mqtt
     ```
2. **Siapkan Orange Pi (3 LTS atau Zero2)**:
   - Flash Ubuntu Server 22.04 ke setiap Orange Pi.
   - Instal dependensi:
     ```bash
     sudo apt install python3 python3-pip v4l-utils -y
     pip3 install OPi.GPIO paho-mqtt
     ```
   - Jika 3 LTS untuk deteksi lokal, tambah OpenCV:
     ```bash
     sudo apt install python3-opencv -y
     ```
3. **Tes Koneksi MQTT**:
   - Di server, publish pesan:
     ```bash
     mosquitto_pub -h localhost -t "pintu/1/kontrol" -m "open"
     ```
   - Di Orange Pi, subscribe:
     ```bash
     mosquitto_sub -h [server_ip] -t "pintu/1/kontrol"
     ```
4. **Tes Webcam**:
   - Colokkan webcam ke Orange Pi (jika deteksi lokal) atau server, cek:
     ```bash
     ls /dev/video*
     ```

### Penggunaan
1. **Arsitektur Sistem**:
   - **PC Server**:
     - Proses deteksi wajah (dlib/DeepFace), plat nomor (EasyOCR), counting, dan klasifikasi gender/umur.
     - Simpan database besar (100+ wajah/plat) di PostgreSQL.
     - Publish sinyal (misal, "open") ke topic MQTT (misal, `pintu/1/kontrol`).
   - **Orange Pi**:
     - **Jika deteksi lokal (3 LTS)**: Tangkap feed webcam, deteksi wajah/plat ringan (OpenCV), kirim ke server untuk verifikasi, terima sinyal MQTT untuk trigger relay.
     - **Jika hanya trigger (Zero2)**: Subscribe ke topic MQTT, aktifkan relay/LED saat terima sinyal.
2. **Umpan Balik**:
   - Suara: "Selamat datang, pintu terbuka" atau "Akses ditolak" via espeak.
   - Visual: LED hijau (sukses), merah (gagal).
3. **Monitoring**:
   - Gunakan Flask/FastAPI untuk dashboard (akses via browser).
   - Log percobaan ke PostgreSQL dan file `/var/log/akses_vision.log`.

### Penanganan Error
- **Banyak Wajah**: Server tolak, kirim notifikasi via MQTT: "Silakan mendekat satu per satu."
- **Gambar Buram/Kecil**: Tolak, notifikasi: "Gambar tidak jelas, silakan mendekat."
- **Koneksi MQTT Gagal**: Retry 3 kali, log error.
- **Timeout**: Reset setelah 3 detik, notifikasi: "Sesi habis, coba lagi."

### Catatan Performa
- **PC Server**: GPU (RTX 3060) memungkinkan latensi <1 detik untuk 100+ wajah/plat.
- **Orange Pi 3 LTS**: Cocok untuk deteksi lokal ringan (~1-2 detik).
- **Orange Pi Zero2**: Ideal untuk trigger saja (latensi MQTT <0.1 detik).
- Skalabilitas: Tambah Orange Pi per pintu, server handle hingga 50 perangkat dengan jaringan stabil.

### Opsi Orange Pi untuk Trigger Saja
- **Orange Pi 3 LTS**:
  - Overkill untuk hanya trigger (2GB RAM, Quad-core Cortex-A53).
  - Pilih jika butuh deteksi lokal atau cadangan untuk ML ringan.
  - Harga: ~Rp 500.000.
- **Orange Pi Zero2**:
  - Cukup untuk trigger (512MB/1GB RAM, Quad-core Cortex-A53).
  - Jalankan skrip MQTT sederhana untuk terima sinyal dan aktifkan relay/LED.
  - Hemat biaya (~Rp 200.000), cocok untuk banyak pintu.
  - Kelemahan: Tidak cocok untuk deteksi wajah/plat karena RAM kecil.
- **Alternatif Lain**:
  - Orange Pi Zero Plus (512MB RAM, ~Rp 250.000): Mirip Zero2, tapi sedikit lebih mahal.
  - Raspberry Pi Zero W (~Rp 300.000): Kompatibel, tapi lebih sulit didapat.
- **Rekomendasi**: Gunakan **Zero2** untuk trigger saja, simpan **3 LTS** untuk pintu utama atau deteksi lokal.

---

## Kontribusi
- Fork repositori.
- Kirim pull request untuk perbaikan bug atau peningkatan.
- Kontak: [Info kontak Mr. Don].

## Lisensi
MIT License. Lihat `LICENSE` untuk detail.

## Penghargaan
- Komunitas OpenCV, dlib, dan EasyOCR.
- Forum Orange Pi (orangepi.org) untuk dukungan perangkat keras.
- Murtaza’s Workshop (YouTube) untuk tutorial deteksi wajah.
