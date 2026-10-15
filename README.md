# 🌡️ Indonesia Urban Heat Explorer

<p align="left">
  <!-- Status Badges -->
  <a href="https://ee-izzmhmd53.projects.earthengine.app/view/urban-heat-explorer">
    <img src="https://img.shields.io/badge/Access_Here-0F9D58?style=for-the-badge&logo=google&logoColor=white" height="38" alt="Access Here">
  </a>
  <a href="https://github.com/izzmhmd/urbanheatexplorer">
    <img src="https://img.shields.io/badge/Platform-Google_Earth_Engine-4285F4?style=for-the-badge&logo=googleearthengine&logoColor=white" height="38" alt="GEE">
  </a>
  <a href="https://github.com/izzmhmd/urbanheatexplorer/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge" height="38" alt="License">
  </a>
  <a href="https://youtu.be/5ab2leqzgTw?si=wDzeLcqE98kgOjo5">
    <img src="https://img.shields.io/badge/Watch_Tutorial-FF0000?style=for-the-badge&logo=youtube&logoColor=white" height="38" alt="Watch Tutorial Video">
  </a>
</p>

**Indonesia Urban Heat Explorer** is an interactive cloud computing-based WebGIS platform (built on Google Earth Engine) for monitoring, analyzing, and visualizing Land Surface Temperature (LST) dynamics and Urban Heat Island (UHI) phenomena across all administrative levels in Indonesia.

---

## 📌 Key Features

- **🗺️ Region of Interest (ROI) Selector**: Enables hierarchical analysis from National, Provincial, Regency/City, down to District/Sub-district levels (utilizing FAO GAUL / GADM administrative boundaries).
- **📅 Automatic & Flexible Date Selector**: Automated monthly LST observation time-frame selection (*monthly aggregation*).
- **📊 Information Charts & Visual Analytics**:
  - *Time Series LST Chart*
  - *Mean LST & UHI Chart* per region
  - *Area of Urban Heat* per region
  - *Estimated Population Affected* by urban heat impacts
- **📍 Location Point Inspection**: Click directly on the map to display a time-series temperature chart for a specific coordinate.
- **🎥 Map Visualizations & Animations**: Export and visualize spatial dynamics of land surface temperature through animated maps.

---

## 📡 Datasets Used

1. **MODIS Terra Land Surface Temperature (MOD11A1.061)**
   - *Spatial Resolution*: 1 km
   - *Function*: LST extraction and regional-scale UHI threshold analysis.
2. **WorldPop Global Project Population Data**
   - *Spatial Resolution*: 100 m
   - *Function*: Estimation of the population affected by urban heat exposure.
3. **Administrative Boundaries (FAO GAUL / GADM)**
   - *Function*: Spatial boundaries selection from Provincial to District levels.

---

## 🚀 How to Access the Application

You can access and explore the WebGIS App directly without installing any additional software on your PC/Desktop via the following link:

👉 **[Access Urban Heat Explorer Web App](https://ee-izzmhmd53.projects.earthengine.app/view/urban-heat-explorer)**

> 💡 **Usage Recommendation**: Accessing via a desktop/laptop browser with a stable internet connection is highly recommended for an optimal map visualization and chart interaction experience.

---

## 🛠️ Code Usage (Google Earth Engine)

If you wish to develop or run this script locally in the **Google Earth Engine Code Editor**:

1. Open [Google Earth Engine Code Editor](https://code.earthengine.google.com/).
2. Create a new script, then copy and paste the contents of `app.js` or `main.js` from this repository.
3. Click **Run** to execute the application.
