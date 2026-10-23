
#  Meteoblue Multimodel Weather Forecast Scraper

A Python-based tool for scraping, storing, and analysing multi-model weather forecast data from [Meteoblue](https://www.meteoblue.com) for **Chandīpur, India** (21.4667°N, 86.9500°E). Built for use with Google Colab.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Weather Models Covered](#weather-models-covered)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Setup & Usage](#setup--usage)
- [Database Schema](#database-schema)
- [EDA & Visualisations](#eda--visualisations)
- [Output Files](#output-files)

---

## Overview

This notebook (`DRDO_MET_DATA_v2.ipynb`) automates the collection of hourly multi-model weather forecasts from the Meteoblue multimodel page for Chandīpur. It uses **Playwright** to intercept live JSON API responses from Meteoblue's internal forecast endpoint, processes and converts all values to SI units, and stores results in a normalised **SQLite database**.

An **Exploratory Data Analysis (EDA)** section provides ready-to-run visualisations for temperature, rainfall, wind speed, wind direction, humidity, and weather pictograms.

---

## Features

- **Browser-based scraping** using Playwright (handles JavaScript-rendered pages)
- **JSON interception** — captures raw forecast data directly from Meteoblue's internal API endpoint
- **SI unit conversion** at ingestion:
  - Temperature: °F → °C
  - Precipitation: inch/hr → mm/hr
  - Wind speed: mph → m/s
- **Weather icon classification** — maps Meteoblue pictogram codes to human-readable conditions (Sunny, Rainy, Overcast, etc.)
- **Normalised SQLite schema** with foreign key relationships across 6 tables
- **EDA visualisations** with dark-background Matplotlib charts
- **HTML pictogram table** rendering a Model × Hour weather icon grid (similar to the Meteoblue website UI)
- **CSV + SQLite export** for download from Google Colab

---

## Weather Models Covered

| Internal Name | Display Label |
|---|---|
| AIFS025 | AIFS-25 |
| GEM15 | GEM-15 |
| GFS05 | GFS-40 |
| ICON | ICON-12 |
| IFS025 | IFS-20 |
| IFSHRES | IFS-HRES |
| MFGLOBAL | ARPEGE-25 |
| NEMSGLOBAL | NEMS-30 |
| NEMSGLOBAL_E | NEMS2-30 |
| NEMSIN | NEMS-10 |
| UMGLOBAL10 | UKMO-10 |

---

## Project Structure

```
.
├── DRDO_MET_DATA_v2.ipynb       # Main notebook (scraping + EDA)
├── meteoblue_forecast.sqlite    # SQLite database (generated at runtime)
└── clean_meteoblue_forecast.csv # Flat CSV export (generated at runtime)
```

---

## Requirements

```
playwright
pandas
matplotlib
numpy
```

Install inside the notebook:

```bash
pip install playwright
playwright install chromium
playwright install-deps chromium
```

> **Note:** Designed and tested on **Google Colab**. The Colab file download cells use `google.colab.files` and will need adaptation for local environments.

---

## Setup & Usage

1. Open `DRDO_MET_DATA_v2.ipynb` in Google Colab.
2. Run **Step 1** to install Playwright and Chromium.
3. Run cells sequentially through **Step 13** to scrape, process, and store the data.
4. The final cells export `clean_meteoblue_forecast.csv` and `meteoblue_forecast.sqlite` for download.
5. Continue to the **EDA section** for analysis and visualisations.

The target URL is set at the top of the notebook:

```python
PAGE_URL = "https://www.meteoblue.com/en/weather/forecast/multimodel/chand%c4%abpur_india_1274740"
DB_PATH  = "meteoblue_forecast.sqlite"
```

---

## Database Schema

The SQLite database uses a **normalised, foreign-key-linked schema** with 6 tables:

```
scrape_runs
    └── forecast_meta  (run_id → scrape_runs.id)
            ├── temperature          (forecast_id → forecast_meta.id)
            ├── precipitation        (forecast_id → forecast_meta.id)
            ├── wind                 (forecast_id → forecast_meta.id)
            └── relative_humidity    (forecast_id → forecast_meta.id)
```

| Table | Key Columns |
|---|---|
| `scrape_runs` | `id`, `page_url`, `data_url`, `scraped_at_utc` |
| `forecast_meta` | `run_id`, `model_name`, `forecast_time_utc`, `x_timestamp_ms`, `cloud_condition`, `cloud_icon_url` |
| `temperature` | `forecast_id`, `temperature_c` (°C) |
| `precipitation` | `forecast_id`, `precipitation_mm_per_hour` (mm/hr) |
| `wind` | `forecast_id`, `wind_speed_ms` (m/s), `wind_direction_degrees` |
| `relative_humidity` | `forecast_id`, `relative_humidity_percent` (%) |

---

## EDA & Visualisations

The EDA section (after Step 13) includes:

- **Temperature** — multi-model time series with ensemble mean and alternating-day shading
- **Rainfall** — histogram of precipitation distribution + average by model
- **Wind Speed** — multi-model time series with night shading and ensemble mean
- **Wind Direction** — scatter plot with compass labels (N / NE / E … NW)
- **Relative Humidity** — multi-model time series with ensemble mean
- **Pictogram Table** — interactive HTML table showing Meteoblue weather icons per model per hour (filterable by date, model, and hour range), displayed in IST (UTC+5:30)

---

## Output Files

| File | Description |
|---|---|
| `meteoblue_forecast.sqlite` | Normalised database with all scraped runs |
| `clean_meteoblue_forecast.csv` | Flat file with one row per model × forecast timestamp |

---

## Location

| Field | Value |
|---|---|
| Location | Chandīpur, India |
| Latitude | 21.4667°N |
| Longitude | 86.9500°E |
| Timezone (display) | IST (UTC+5:30) |
| Storage timezone | UTC |
