# Satellite-Based Precipitation Downscaling for Greece

A machine learning pipeline for generating high-resolution daily precipitation maps over Greece by fusing satellite-derived environmental variables with ground station observations.

## Overview

This project combines multiple satellite data sources (MODIS cloud properties, IMERG precipitation estimates, column water vapor) with meteorological station records and topographic features to train an XGBoost regression model. The trained model performs pixel-wise inference to produce daily precipitation maps at 1 km resolution for the period 2010–2023.

## Pipeline

The workflow is organized into six sequential notebooks:

| # | Notebook | Description |
|---|----------|-------------|
| 1 | `01_Export_MODIS_Water_Vapor.ipynb` | Exports daily MODIS Column Water Vapor (MCD19A2) for Greece via Google Earth Engine |
| 2 | `02_IMERG_Precipitation_Processing.ipynb` | Aggregates IMERG precipitation within 24h windows, extracts station values, resamples to 1 km |
| 3 | `03_MODIS_Wind_Resampled_Combined.ipynb` | Resamples MODIS cloud properties (COT, CER, CWP) and wind data to a consistent extent, resolution, and CRS |
| 4 | `04_Extract_Nearest_Pixels.ipynb` | Extracts nearest raster pixel values for each weather station using spatial proximity search |
| 5 | `05_Merge_Preprocessing.ipynb` | Merges all environmental datasets (satellite, station, topographic) into a single ML-ready dataframe |
| 6 | `06_Model_Training_Inference.ipynb` | Trains XGBoost with feature engineering and hyperparameter tuning; runs daily pixel-wise inference |

## Data Sources

| Dataset | Source | Resolution | Variables |
|---------|--------|------------|-----------|
| Cloud Properties | MODIS (COT, CER, CWP) | 1 km | Cloud Optical Thickness, Cloud Effective Radius, Cloud Water Path |
| Column Water Vapor | MODIS MCD19A2 | 1 km | Near-IR water vapor |
| Precipitation | GPM IMERG | 0.1° → 1 km | Daily accumulated precipitation |
| Wind | ERA5 / Reanalysis | Resampled to 1 km | Wind speed and direction |
| Topography | SRTM DEM | 90 m → 1 km | Elevation, Slope, Aspect, TWI, TPI, Landforms |
| Ground Stations | Hellenic National Meteorological Service | Point | Daily precipitation (mm) |

## Feature Engineering

- **COT-CER Ratio**: Proxy for cloud microphysical regime
- **CWP²**: Captures nonlinear relationship between cloud water content and precipitation
- **Log-transformed target**: Addresses the skewed distribution of precipitation values

## Model

- **Algorithm**: XGBoost Regressor
- **Tuning**: RandomizedSearchCV with cross-validation
- **Preprocessing**: QuantileTransformer for feature normalization
- **Evaluation metrics**: R², RMSE, MAE on validation and test sets
- **Output**: Daily GeoTIFF precipitation maps (1 km, EPSG:2100)

## Repository Structure

```
satellite-precipitation-downscaling/
├── README.md
├── .gitignore
├── notebooks/
│   ├── 01_Export_MODIS_Water_Vapor.ipynb
│   ├── 02_IMERG_Precipitation_Processing.ipynb
│   ├── 03_MODIS_Wind_Resampled_Combined.ipynb
│   ├── 04_Extract_Nearest_Pixels.ipynb
│   ├── 05_Merge_Preprocessing.ipynb
│   └── 06_Model_Training_Inference.ipynb
├── data/                  # Not tracked (see .gitignore)
└── models/                # Not tracked (see .gitignore)
```

## Requirements

- Python 3.9+
- Google Earth Engine Python API
- xgboost, scikit-learn, rasterio, xarray, numpy, pandas

## Usage

1. Run notebooks 01–04 to acquire and process satellite data
2. Run notebook 05 to merge all datasets
3. Run notebook 06 to train the model and generate precipitation maps

> **Note**: Notebook 01 requires Google Earth Engine authentication. Notebooks 02–04 assume data has been downloaded locally.

## Author

Nikos Kordalis — [GitHub](https://github.com/nikkordalis)
