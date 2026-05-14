# SDS210 Wildfire Mapping Project

## Recent Wildfire Detections: A Reproducible Interactive Mapping Workflow

This repository contains a reproducible workflow for retrieving recent active fire detections from the **NASA FIRMS API**, cleaning the data with Python, exploring global spatial and temporal wildfire patterns, and publishing the result as a set of **interactive web maps**.

The project was developed for the SDS210 *Programming with Spatial Data* course and focuses on building a transparent, notebook-based geospatial workflow rather than only producing a final map.

---

## Project question

**How can recent wildfire detections be retrieved from the NASA FIRMS API, cleaned, analysed, and visualised in interactive web maps?**

The analysis breaks this down into four focused research questions:

| #  | Question |
|----|----------|
| Q1 | Where are the most severe active wildfires (highest Fire Radiative Power)? |
| Q2 | How does thermal intensity (FRP) vary across fire events and continents? |
| Q3 | Are wildfires more commonly detected during daytime or nighttime passes, and does this differ by region? |
| Q4 | How does detection confidence relate to fire intensity (FRP)? |

---

## Study scope

- **Study area:** Global extent (`AREA = "world"` in the retrieval notebook; a bounding box can be substituted for a regional study)
- **Time window:** The most recent few days of detections, set by `DAYS` in the retrieval notebook (currently 5)
- **Data source:** NASA FIRMS API
- **Sensor product:** VIIRS S-NPP active fire detections, Near Real-Time (`VIIRS_SNPP_NRT`)
- **Geometry type:** Point detections from latitude and longitude coordinates
- **Main attributes:** acquisition date, acquisition time, confidence, brightness temperature, fire radiative power (FRP), latitude, longitude

---

## Repository structure

```text
sds210-wildfire-mapping-project/
├── data/
│   ├── raw/                 # Original FIRMS API download / cached CSV (firms_viirs_global_5d.csv)
│   └── processed/           # Cleaned outputs (firms_viirs_cleaned.csv, firms_viirs_cleaned.gpkg)
├── notebooks/
│   ├── data_retrieval.ipynb
│   ├── data_cleaning.ipynb
│   ├── exploratory_analysis.ipynb
│   └── interactive_web_map.ipynb
├── outputs/
│   ├── figures/             # Exploratory plots for Q1–Q4
│   └── maps/                # Exported interactive HTML maps
├── environment.yml          # Python package dependencies (or requirements.txt)
└── README.md
```

The intended workflow is sequential: retrieval → cleaning → analysis → interactive mapping.

---

## Workflow overview

### 1. Data retrieval — `data_retrieval.ipynb`

Retrieves active fire detections from the NASA FIRMS API. The API request is built from four main parameters:

- `MAP_KEY`: personal FIRMS API key
- `SOURCE`: satellite product, for example `VIIRS_SNPP_NRT`
- `AREA`: area of interest — `"world"` for global coverage, or a `west,south,east,north` bounding box for a region
- `DAYS`: number of recent days requested

A reusable `fetch_firms_data()` function handles both the live API call and a local fallback: the downloaded CSV is cached in `data/raw/`, and if no API key is supplied (or the request fails) the notebook loads the cached file instead. This keeps the later notebooks reproducible without repeatedly querying the API.

### 2. Data cleaning — `data_cleaning.ipynb`

Prepares the raw FIRMS detections for spatial analysis. It drops records with missing coordinates or FRP, removes duplicate detections, parses acquisition date and time into a single UTC datetime, standardises the `confidence` labels, assigns a coarse `continent` label to each detection, and adds a normalised FRP column for marker sizing. Point geometries are created from `longitude` and `latitude`, and the table is converted into a GeoDataFrame with the coordinate reference system `EPSG:4326`.

The cleaned dataset is exported to `data/processed/` as both a CSV and a GeoPackage. These processed files are the main input for the analysis and map notebooks.

### 3. Exploratory analysis — `exploratory_analysis.ipynb`

Loads the cleaned fire-detection dataset and addresses the four research questions:

- **Q1** — a ranked view of the highest-FRP detections
- **Q2** — FRP distributions by continent, on a log scale to handle the heavy right skew
- **Q3** — a day/night detection breakdown by region
- **Q4** — FRP summarised across the low / nominal / high confidence classes

This step checks whether the data looks plausible before building the final maps. Figures are saved to `outputs/figures/`.

### 4. Interactive mapping — `interactive_web_map.ipynb`

Builds two complementary interactive web maps with `folium` / Leaflet:

- **Date-picker map** (`wildfire_interactive_map.html`) — each acquisition date is rendered as a separate, toggleable layer, collected under a grouped layer control. Because the control is non-exclusive, several days can be switched on at once and compared side by side on a single map.
- **Time-slider map** (`wildfire_timeslider_map.html`) — the same detections animated through time with play / step / loop controls, for day-by-day playback.

In both maps, marker **colour** encodes detection confidence (low / nominal / high) and marker **radius** encodes Fire Radiative Power. Popups expose the full metadata for each detection (date and time, FRP, confidence, brightness, continent, satellite pass, coordinates). For browser performance, each day is independently capped at its highest-FRP detections.

The maps are exported as standalone HTML files so they can be opened directly in a browser or shared as lightweight project outputs.

---

## How to retrieve the wildfire data yourself

Follow these steps to reproduce the FIRMS data retrieval.

### Step 1: Request a FIRMS MAP_KEY

Go to the NASA FIRMS API page and request a free `MAP_KEY` using your email address:

<https://firms.modaps.eosdis.nasa.gov/api/map_key/>

Keep the key private. Do not commit it directly to GitHub — load it from an environment variable or an untracked configuration file instead, so it is never part of the repository history.

### Step 2: Choose a FIRMS data source

For recent wildfire mapping, a useful starting point is:

```python
SOURCE = "VIIRS_SNPP_NRT"
```

Other available sources can be checked through the FIRMS API documentation.

### Step 3: Define the area

The FIRMS area endpoint accepts either the keyword `world` for global coverage, or a bounding box in `west,south,east,north` order. This project uses the global extent:

```python
AREA = "world"

# Or, for a regional study, a bounding box — e.g. Southern Europe / Mediterranean:
# AREA = "-10,34,35,46"
```

### Step 4: Choose the time window

Set the number of recent days to download:

```python
DAYS = 5
```

The FIRMS area endpoint returns detections from today backwards depending on `DAYS`. Some endpoints limit how many days can be requested at once, so longer study periods may need repeated requests or downloaded archive data.

### Step 5: Build the API URL

```python
MAP_KEY = "your_map_key_here"
SOURCE  = "VIIRS_SNPP_NRT"
AREA    = "world"
DAYS    = 5

base_url = "https://firms.modaps.eosdis.nasa.gov/api/area/csv"
url = f"{base_url}/{MAP_KEY}/{SOURCE}/{AREA}/{DAYS}"
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
raw_df.to_csv("data/raw/firms_viirs_global_5d.csv", index=False)

raw_df.head()
```

### Step 7: Clean the data and convert it to spatial data

The `data_cleaning.ipynb` notebook applies the full cleaning pipeline (missing-value handling, deduplication, datetime parsing, confidence standardisation, continent labels) and builds the GeoDataFrame. The core geometry step is:

```python
import geopandas as gpd

fires_gdf = gpd.GeoDataFrame(
    cleaned_df,
    geometry=gpd.points_from_xy(cleaned_df["longitude"], cleaned_df["latitude"]),
    crs="EPSG:4326",
)

Path("data/processed").mkdir(parents=True, exist_ok=True)
fires_gdf.to_file("data/processed/firms_viirs_cleaned.gpkg", layer="fires", driver="GPKG")
```

### Step 8: Use the processed file in the analysis and map notebooks

All later notebooks load the cleaned file from `data/processed/` instead of querying the API again. This makes the project easier to reproduce and avoids unnecessary API calls.

---

## Main Python libraries

- `pandas` and `numpy` for tabular data handling
- `geopandas` and `shapely` for spatial data processing
- `requests` for API requests
- `matplotlib` and `seaborn` for exploratory plots
- `folium` (with `folium.plugins`) for the interactive web maps
- `pathlib` for robust, relative file paths

---

## Outputs

The project produces the following main outputs:

1. **Raw FIRMS CSV file** saved in `data/raw/`
2. **Cleaned spatial dataset** saved in `data/processed/` as CSV and GeoPackage
3. **Exploratory figures** for Q1–Q4 saved in `outputs/figures/`
4. **Two interactive wildfire maps** exported as HTML files in `outputs/maps/`:
   - `wildfire_interactive_map.html` — date-picker map for comparing days
   - `wildfire_timeslider_map.html` — animated time-slider map

---

## Limitations

FIRMS active fire detections show satellite-detected thermal anomalies, not manually confirmed fire perimeters. A detection represents a satellite observation at a specific time and location. Cloud cover, overpass timing, sensor resolution, false positives and delayed updates can all influence the result. For browser performance, the interactive maps also cap the number of markers drawn per day, keeping the highest-FRP detections. The maps should therefore be interpreted as a recent fire-detection overview, not as an official emergency-response product.

---

## Future enhancements

- Add administrative boundaries or country-level summaries via spatial joins
- Compare different FIRMS products, such as MODIS and VIIRS, on parallel layers
- Add weather or land-cover data for contextual analysis
- Pull a longer time window and aggregate to weekly layers for seasonal comparison
- Publish the HTML maps through GitHub Pages
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