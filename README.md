# 🌡️ Indonesia Urban Heat Explorer

<p align="left">
  <!-- Status Badge -->
  <a href="https://ee-izzmhmd53.projects.earthengine.app/view/urban-heat-explorer">
    <img src="https://img.shields.io/badge/Access_Here-0F9D58?style=for-the-badge&logo=google&logoColor=white" height="38" alt="Access Here">
  </a>
  <a href="https://github.com/izzmhmd/urbanheatexplorer">
    <img src="https://img.shields.io/badge/Platform-Google_Earth_Engine-4285F4?style=for-the-badge&logo=googleearthengine&logoColor=white" height="38" alt="GEE">
  </a>
  <a href="https://github.com/izzmhmd/urbanheatexplorer/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge" height="38" alt="License">
  </a>
</p>

**Indonesia Urban Heat Explorer** adalah platform WebGIS interaktif berbasis *cloud computing* (Google Earth Engine) untuk pemantauan, analisis, dan visualisasi dinamika *Land Surface Temperature* (LST) serta fenomena *Urban Heat Island* (UHI) di seluruh wilayah administratif Indonesia.

---

## 📌 Fitur Utama

- **🗺️ Region of Interest (ROI) Selector**: Memungkinkan analisis hierarkis dari tingkat Nasional, Provinsi, Kabupaten/Kota, hingga Kecamatan (menggunakan batas administratif FAO GAUL/GADM).
- **📅 Automatic & Flexible Date Selector**: Pemilihan rentang waktu observasi LST berbasis bulanan (*monthly aggregation*) secara otomatis.
- **📊 Information Charts & Visual Analytics**:
  - *Time Series LST Chart*
  - *Mean LST & UHI Chart* per wilayah
  - *Area of Urban Heat* per wilayah
  - *Estimated Population Affected* oleh dampak UHI
- **📍 Location Point Inspection**: Klik langsung pada peta untuk menampilkan grafik deret waktu (time series) suhu di koordinat spesifik.
- **🎥 Map Visualizations & Animations**: Ekspor dan visualisasi dinamika animasi spasial suhu permukaan tanah.

---

## 📡 Dataset yang Digunakan

1. **MODIS Terra Land Surface Temperature (MOD11A1.061)**
   - *Resolusi Spasial*: 1 km
   - *Fungsi*: Ekstraksi LST dan analisis nilai ambang batas UHI skala regional.
2. **WorldPop Global Project Population Data**
   - *Resolusi Spasial*: 100 m
   - *Fungsi*: Estimasi jumlah populasi yang terdampak oleh paparan panas perkotaan.
3. **Batas Administratif (FAO GAUL / GADM)**
   - *Fungsi*: Pilihan cakupan area kajian tingkat Provinsi hingga Kecamatan.

---

## 🚀 Cara Mengakses Aplikasi

Kamu dapat langsung mencoba WebGIS App tanpa perlu menginstal aplikasi tambahan di PC/Laptop melalui tautan berikut:

👉 **[Akses Web App Urban Heat Explorer](https://ee-izzmhmd53.projects.earthengine.app/view/urban-heat-explorer)**

> 💡 **Saran Penggunaan**: Disarankan mengakses melalui browser di PC/Laptop dengan koneksi internet yang stabil untuk pengalaman visualisasi peta dan grafik yang maksimal.

---

## 🛠️ Penggunaan Kode (Google Earth Engine)

Jika kamu ingin mengembangkan atau menjalankan skrip ini secara lokal di **Google Earth Engine Code Editor**:

1. Buka [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
2. Buat skrip baru, lalu salin isi file `app.js` atau `main.js` dari repositori ini.
3. Klik **Run** untuk menjalankan aplikasi.
