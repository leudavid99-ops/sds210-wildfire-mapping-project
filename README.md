# sds210-wildfire-mapping-project

# Recent Wildfire Detections: A Reproducible Interactive Mapping Workflow

## Project question
How can recent wildfire detections be retrieved from the NASA FIRMS API, cleaned, analysed, and visualised in an interactive web map for a selected study region?

## Study scope
- Study area: Southern Europe
- Time window: Last 30 days
- Data source: NASA FIRMS API
- Main variables: acquisition date, acquisition time, confidence, brightness, FRP, latitude, longitude

## Planned output
- A reproducible notebook that retrieves wildfire detections directly from the FIRMS API
- A cleaned GeoDataFrame of recent wildfire detections
- A small exploratory summary of temporal and spatial patterns
- An interactive web map with popups

## Workflow steps
1. **Data retrieval**: Use the `requests` library to query the NASA FIRMS API for wildfire detections in the specified region and time window.
2. **Data cleaning**: Convert the retrieved data into a GeoDataFrame, handle missing values, and ensure correct data types. 
3. **Exploratory analysis**: Generate summary statistics and visualizations to understand the temporal and spatial distribution of wildfire detections.
4. **Interactive mapping**: Create an interactive web map using `folium` to display wildfire detections with popups showing key information (e.g., confidence, brightness, FRP).
5. **Reproducibility**: Document the entire workflow in a Jupyter Notebook, ensuring that all steps can be easily reproduced by others.

## Future enhancements
- Incorporate additional data sources (e.g., weather data, land cover) for more comprehensive analysis
- Implement machine learning models to predict wildfire risk based on historical data
- Develop a web application for real-time wildfire monitoring and alerts    

## Conclusion
This project aims to provide a comprehensive workflow for retrieving, cleaning, analysing, and visualising recent wildfire detections from the NASA FIRMS API. By creating a reproducible notebook and an interactive web map, I hope to enhance understanding of wildfire patterns and support informed decision-making for wildfire management and mitigation efforts.

## References
- NASA FIRMS API documentation: https://firms.modaps.eosdis.nasa.gov/api/
- Geopandas documentation: https://geopandas.org/
- Folium documentation: https://python-visualization.github.io/folium/  
- Jupyter Notebook documentation: https://jupyter.org/documentation
- Matplotlib documentation: https://matplotlib.org/stable/contents.html
- Seaborn documentation: https://seaborn.pydata.org/
- Pandas documentation: https://pandas.pydata.org/docs/
- Requests documentation: https://docs.python-requests.org/en/latest/
- Geopy documentation: https://geopy.readthedocs.io/en/stable/
- Scikit-learn documentation: https://scikit-learn.org/stable/documentation.html
- OpenStreetMap documentation: https://www.openstreetmap.org/about
- Leaflet documentation: https://leafletjs.com/reference.html
- Mapbox documentation: https://docs.mapbox.com/

