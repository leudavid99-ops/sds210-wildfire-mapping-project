# SDS210 Wildfire Mapping Project

## Recent Wildfire Detections: A Reproducible Interactive Mapping Workflow

This repository contains a reproducible workflow for retrieving recent active fire detections from the **NASA FIRMS API**, cleaning the data with Python, exploring spatial and temporal wildfire patterns, and publishing the result as an **interactive web map**.

The project was developed for the SDS210 *Programming with Spatial Data* course and focuses on building a transparent, notebook-based geospatial workflow rather than only producing a final map.

---

## Project question

**How can recent wildfire detections be retrieved from the NASA FIRMS API, cleaned, analysed, and visualised in an interactive web map for a selected study region?**

---

## Study scope

- **Study area:** Southern Europe / Mediterranean wildfire region
- **Time window:** Recent detections, usually the last 7 to 30 days depending on the API request
- **Data source:** NASA FIRMS API
- **Sensor product:** VIIRS active fire detections, for example `VIIRS_SNPP_NRT`
- **Geometry type:** Point detections from latitude and longitude coordinates
- **Main attributes:** acquisition date, acquisition time, confidence, brightness, fire radiative power (FRP), latitude, longitude

---

## Repository structure

```text
sds210-wildfire-mapping-project/
├── data/
│   ├── raw/                 # Original FIRMS API downloads or cached CSV files
│   └── processed/           # Cleaned GeoPackage / GeoJSON / CSV outputs
├── notebooks/
│   ├── 01_data_retrieval.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_exploratory_analysis.ipynb
│   └── 04_interactive_map.ipynb
├── maps/                    # Exported interactive HTML map
├── README.md
└── requirements.txt         # Python package dependencies, if used
```

The exact folder names may vary slightly, but the intended workflow is sequential: retrieval → cleaning → analysis → interactive mapping.

---

## Workflow overview

### 1. Data retrieval

The first notebook retrieves active fire detections from the NASA FIRMS API. The API request is built from four main parameters:

- `MAP_KEY`: personal FIRMS API key
- `SOURCE`: satellite product, for example `VIIRS_SNPP_NRT`
- `AREA_COORDINATES`: bounding box in the format `west,south,east,north`
- `DAY_RANGE`: number of recent days requested

The downloaded CSV response is loaded into a pandas DataFrame and saved locally in `data/raw/` so that the following notebooks can be rerun without repeatedly querying the API.

### 2. Data cleaning

The second notebook prepares the raw FIRMS detections for spatial analysis. It standardises column names and data types, parses acquisition dates and times, checks missing values, creates point geometries from `longitude` and `latitude`, and converts the table into a GeoDataFrame with the coordinate reference system `EPSG:4326`.

The cleaned dataset is exported to `data/processed/` as a reusable spatial file, such as GeoPackage or GeoJSON. This processed file is the main input for the exploratory analysis and map notebooks.

### 3. Exploratory analysis

The third notebook uses the cleaned fire-detection dataset to summarise recent wildfire patterns. Typical outputs include:

- number of detections in the selected period
- detections per day
- confidence class distribution
- FRP and brightness summaries
- simple spatial plots of detections across the study area

This step helps check whether the data looks plausible before building the final map.

### 4. Interactive mapping

The fourth notebook builds an interactive web map using `folium` / Leaflet. Fire detections are displayed as point markers or clustered points with popups containing key attributes such as acquisition date, confidence, brightness and FRP.

The final map is exported as an HTML file so it can be opened directly in a browser or shared as a lightweight project output.

---

## How to retrieve the wildfire data yourself

Follow these steps to reproduce the FIRMS data retrieval.

### Step 1: Request a FIRMS MAP_KEY

Go to the NASA FIRMS API page and request a free `MAP_KEY` using your email address:

<https://firms.modaps.eosdis.nasa.gov/api/map_key/>

Keep the key private. Do not commit it directly to GitHub.

### Step 2: Choose a FIRMS data source

For recent wildfire mapping, a useful starting point is:

```python
SOURCE = "VIIRS_SNPP_NRT"
```

Other available sources can be checked through the FIRMS API documentation.

### Step 3: Define the study area

The FIRMS area endpoint uses a bounding box in the order:

```text
west,south,east,north
```

Example for a broad Southern Europe / Mediterranean extent:

```python
AREA_COORDINATES = "-10,34,35,46"
```

You can adapt this bounding box to a smaller region if needed.

### Step 4: Choose the time window

Set the number of recent days to download:

```python
DAY_RANGE = 7
```

For the most recent data, the FIRMS area endpoint returns detections from today backwards depending on the selected `DAY_RANGE`. Some endpoints limit how many days can be requested at once, so longer study periods may need repeated requests or downloaded archive data.

### Step 5: Build the API URL

```python
MAP_KEY = "your_map_key_here"
SOURCE = "VIIRS_SNPP_NRT"
AREA_COORDINATES = "-10,34,35,46"
DAY_RANGE = 7

url = (
    f"https://firms.modaps.eosdis.nasa.gov/api/area/csv/"
    f"{MAP_KEY}/{SOURCE}/{AREA_COORDINATES}/{DAY_RANGE}"
)
```

### Step 6: Download the CSV with Python

```python
from pathlib import Path
import pandas as pd
import requests
from io import StringIO

response = requests.get(url, timeout=60)
response.raise_for_status()

raw_df = pd.read_csv(StringIO(response.text))

Path("data/raw").mkdir(parents=True, exist_ok=True)
raw_df.to_csv("data/raw/firms_raw.csv", index=False)

raw_df.head()
```

### Step 7: Convert the table to spatial data

```python
import geopandas as gpd

fires_gdf = gpd.GeoDataFrame(
    raw_df,
    geometry=gpd.points_from_xy(raw_df["longitude"], raw_df["latitude"]),
    crs="EPSG:4326"
)

Path("data/processed").mkdir(parents=True, exist_ok=True)
fires_gdf.to_file("data/processed/firms_cleaned.gpkg", layer="fires", driver="GPKG")
```

### Step 8: Use the processed file in the analysis and map notebooks

All later notebooks should load the cleaned file from `data/processed/` instead of querying the API again. This makes the project easier to reproduce and avoids unnecessary API calls.

---

## Main Python libraries

- `pandas` for tabular data handling
- `geopandas` for spatial data processing
- `requests` for API requests
- `matplotlib` for exploratory plots
- `folium` for the interactive web map
- `pathlib` for robust file paths

---

## Outputs

The project produces three main outputs:

1. **Raw FIRMS CSV file** saved in `data/raw/`
2. **Cleaned spatial dataset** saved in `data/processed/`
3. **Interactive wildfire map** exported as an HTML file in `maps/` or the project output folder

---

## Limitations

FIRMS active fire detections show satellite-detected thermal anomalies, not manually confirmed fire perimeters. A detection represents a satellite observation at a specific time and location. Cloud cover, overpass timing, sensor resolution, false positives and delayed updates can influence the result. The interactive map should therefore be interpreted as a recent fire-detection overview, not as an official emergency-response product.

---

## Future enhancements

- Add administrative boundaries or country-level summaries
- Compare different FIRMS products, such as MODIS and VIIRS
- Add weather or land-cover data for contextual analysis
- Classify detections by confidence and FRP intensity
- Publish the HTML map through GitHub Pages
- Extend the workflow into a small dashboard or web application

---

## References

- NASA FIRMS API documentation: <https://firms.modaps.eosdis.nasa.gov/api/>
- NASA FIRMS area API endpoint: <https://firms.modaps.eosdis.nasa.gov/api/area/>
- NASA FIRMS MAP_KEY page: <https://firms.modaps.eosdis.nasa.gov/api/map_key/>
- NASA FIRMS Python API tutorial: <https://firms.modaps.eosdis.nasa.gov/content/academy/data_api/firms_api_use.html>
- GeoPandas documentation: <https://geopandas.org/>
- Folium documentation: <https://python-visualization.github.io/folium/>
- Pandas documentation: <https://pandas.pydata.org/docs/>
- Requests documentation: <https://requests.readthedocs.io/>
- Leaflet documentation: <https://leafletjs.com/reference.html>
