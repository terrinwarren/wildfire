# wildfire

## Project Overview
This project builds a model that predicts wildfire risk in Butte county California using vegetation and environmental data from satellites. It generates maps showing where fires are most likely to occur and how that risk evolves throughout fire season. Predictions are validated against historical fire perimeters to assess accuracy, and outputs are designed to support emergency response prioritization. I chose Butte county for my initial investigation because according to [CalFire Statistics](https://www.fire.ca.gov/our-impact/statistics) the 2018 Camp fire in Butte County was the [most destructive](https://34c031f8-c9fd-4018-8c5a-4159cdff6b0d-cdn-endpoint.azureedge.net/-/media/calfire-website/our-impact/fire-statistics/top-20-destructive-ca-wildfires.pdf?rev=737a1073f76947b4a3bfb960b19f44c7&hash=7CA02D30D9BF46A32D5D98BD108BA26A) and [most deadly](https://34c031f8-c9fd-4018-8c5a-4159cdff6b0d-cdn-endpoint.azureedge.net/-/media/calfire-website/our-impact/fire-statistics/top-20-deadliest-ca-wildfires.pdf?rev=0d4612ff0cb447fb827fa0ac6c309d3d&hash=34718653A215C315C5E3CB5BB6A4E550) wildfire in CA. There have also been several other fires in this area with varying levels of severity. The pipeline is built to scale to other California counties in the future using similar methodology. 

## Stakeholders

Primary Stakeholders: CAL FIRE and Emergency Responders

Secondary Stakeholders: California Policymakers, Butte County Office of Emergency Services, General Public

## Anti-Goals

- This project does not predict the day a fire will ignite.
- This project does not include real-time data updates.
- This project does not incorporate any areas outside of California. 

## Primary Data Sources

1. [CAL FIRE Historical Wildfire Perimeters](https://gis.data.ca.gov/datasets/CALFIRE-Forestry::california-fire-perimeters-all/about)
   - County boundaries clipped using [CA County Data](https://data.ca.gov/dataset/ca-geographic-boundaries)

2. [Sentinel-2](https://dataspace.copernicus.eu/data-collections/copernicus-sentinel-missions/sentinel-2)

3. [GRIDMET](https://www.climatologylab.org/gridmet.html) 

## Time Range
May 1 - November 30 annually, 2016 - 2024. This window covers California's fire season for both Northern and Southern California. Source: [Western Fire Chiefs Association](https://wfca.com/wildfire-articles/california-fire-season-in-depth-guide/)

## Features
Using the findings and recommendations in [Spectral indices alone are not enough: A critical assessment of pre-fire wildfire risk mapping](https://www.sciencedirect.com/science/article/pii/S1574954125004443) we use the following features: 

### Sentinel-2 Derived Features from Bands (B4, B8, B11, B12)
| Feature | Formula | Measures | Fire Risk |
|---|---|---|---|
| NDVI (Normalized Difference Vegetation Index)| (B8 - B4)/(B8 + B4) | prescense of live green vegetation | fuel abundance |
| NDVI Anomaly | deviation from baseline | Unusual dryness | early warning signal |
| NDWI (Normalized Difference Water Index)| (B8 - B11)/(B8 + B11) | moisture of live vegetation | flammability of fuel |
| NBR (Normalized Burn Ratio)| (B8 - B12)/(B8 + B12) | density of vegetation | fuel load |

For a more detailed descriptions of the bands [see here](https://custom-scripts.sentinel-hub.com/sentinel-2/bands/). Additionally, the SCL (Scene Classification Layer) band is used to mask clouds and invalid pixels before computing indices.

### GRIDMET Climate Features
| Feature   | Measures |
|---|---|
| VPD (Vapor Pressure Deficit) | Atmospheric dryness | 
| BI (Burning Index) | Fire danger rating | 
| FM100 (Dead Fuel Moisture) | 100-hour fuel dryness | 
| PR (Precipitation)  | Recent rainfall | 
| VS (Wind Speed)  | Fire spread potential | 

### Topography Features (planned)
- Elevation
- Slope
- Aspect
- Source: USGS 3DEP



## Data Limitations
- Cal Fire perimeters exclude fires if the size falls under certain thresholds: CAL FIRE (including contract counties) does not consider perimeters < 10 acres in timber, < 50 acres in brush, or < 300 acres in grass, and/or < 3 impacted residential or commercial structures, and/or caused < 1 fatality.
- Sentinel-2 data may be missing or unusable in some months due to cloud coverage. We only consider images with cloud cover <20% and for each month amongst those images we consider at most one image which has minimal cloud coverage.
- GRIDMET data is coarser than Sentinel-2 data which we will eventually have to decide how to rectify.  

## References
- [Western Fire Chiefs Association — California Fire Season Guide](https://wfca.com/wildfire-articles/california-fire-season-in-depth-guide/)
- [USGS 3D Elevation](https://www.usgs.gov/3d-elevation-program/about-3dep-products-services)
- [Bilal(2025): Spectral indices alone are not enough](https://www.sciencedirect.com/science/article/pii/S1574954125004443)
- [GRIDMET Documentation](https://www.climatologylab.org/gridmet.html)
- [Sentinel-2 Band Descriptions](https://custom-scripts.sentinel-hub.com/sentinel-2/bands/)
- [NDVI Anomaly Detection Script](https://custom-scripts.sentinel-hub.com/sentinel-2/ndvi_anomaly_detection/)
- [Fuel Moisture Guides](https://gacc.nifc.gov/nrcc/predictive/fuels_fire-danger/psa_component_glossary.htm)


