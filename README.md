# wildfire

This project builds a model that predicts wildfire risk in Butte county California using vegetation and environmental data from satellites. It generates maps showing where fires are most likely to occur and how that risk evolves throughout fire season. Predictions are validated against historical fire perimeters to assess accuracy, and outputs are designed to support emergency response prioritization. I chose Butte county for my initial investigation because according to [CalFire Statistics](https://www.fire.ca.gov/our-impact/statistics) the 2018 Camp fire in Butte County was the [most destructive](https://34c031f8-c9fd-4018-8c5a-4159cdff6b0d-cdn-endpoint.azureedge.net/-/media/calfire-website/our-impact/fire-statistics/top-20-destructive-ca-wildfires.pdf?rev=737a1073f76947b4a3bfb960b19f44c7&hash=7CA02D30D9BF46A32D5D98BD108BA26A) and [most deadly](https://34c031f8-c9fd-4018-8c5a-4159cdff6b0d-cdn-endpoint.azureedge.net/-/media/calfire-website/our-impact/fire-statistics/top-20-deadliest-ca-wildfires.pdf?rev=0d4612ff0cb447fb827fa0ac6c309d3d&hash=34718653A215C315C5E3CB5BB6A4E550) wildfire in CA. There have also been several other fires in this area with varying levels of severity. 

The pipeline is built to scale to other California counties in the future using similar methodology. 

Primary Stakeholders: CAL FIRE and Emergency Responders

Secondary Stakeholders: Emergency Services (At both the State and County level)


Primary data sets come from:  
1. (CAL FIRE Historical Wildfire Perimeters GeoJSON file)[https://gis.data.ca.gov/datasets/CALFIRE-Forestry::california-fire-perimeters-all/about] 

Also used the data file on https://data.ca.gov/dataset/ca-geographic-boundaries to clip the data to the specific county 

2. (Sentinel-2)[https://dataspace.copernicus.eu/data-collections/copernicus-sentinel-missions/sentinel-2]

3. wget data with weather/environmental stuff

Based on brief investigation of these data sets, the tentative time range under consideration is May 1 - November 30 annually from 2016-present. This will include the 2018 Camp fire and both of the above data sets include that date range. In order to manage the size of the data, we will focus on fire season which falls between May and November annually(according to the Western Fire Chiefs Association)[https://wfca.com/wildfire-articles/california-fire-season-in-depth-guide/#:~:text=Fires%20are%20possible%20throughout%20the,June)%20and%20runs%20until%20October.] 

Using the findings and recommendations in (Spectral indices alone are not enough: A critical assessment of pre-fire wildfire risk mapping)[https://www.sciencedirect.com/science/article/pii/S1574954125004443] we use the following features: 

From the sentinel satellite data we can use the bands to calculate the following: 
1. NDVI — Normalized Difference Vegetation Index 
    this measures how much live green vegetation there is 
    *Fuel abundance* NDVI = (B8 - B4)/(B8 + B4)

2. NDVI Anomaly- compares greenness/dryness to prior years to see if an area is unusually green/dry 
    *Early indicator/risk signal* (see e.g. https://custom-scripts.sentinel-hub.com/sentinel-2/ndvi_anomaly_detection/)

3. NDWI — Normalized Difference Water Index measures moisture of live vegetation (only at the top level/canopy)
    *flammability of fuel* Note: there are actually two different versions of this, and for our purposes, we want the version proposed by Gao which is defined as NDWI = (NIR - SWIR)/(NIR + SWIR) = (B8 - B11)/(B8 + B12)

4. NBR — Normalized Burn Ratio measures density of vegetation (pre-fire fuel presence indicator)
    *fuel load* Calculated as NBR = (B8 - B12)/(B8 + B12)


I need bands B4, B8, B11, and B12 from the Sentinel Data set-- descriptions of the bands found (here)[https://custom-scripts.sentinel-hub.com/sentinel-2/bands/]. Additionally, I need SCL 

Meteorological/Drought features	
1. Land Surface Temperature (LST); 
2. Vapor Pressure Deficit (VPD); 
3. Drought indices (e.g., KBDI, SPEI); 
4. Rainfall anomaly; 
5. Wind speed	
Potential sources: Reanalysis and climate data: ERA5, CHIRPS; Remote sensing: MODIS temperature products

Fuel Type and Load	
1. Land cover classification; 
2. Fuel models; 
3. Biomass density	
Potential sources: Land cover maps: MODIS MCD12Q1, ESA WorldCover; National fuel databases

Topography	
1. Elevation; 
2. Slope; 
3. Aspect	
Potential sources: DEMs: SRTM, ASTER, or LiDAR-derived terrain data


References I looked at for whatever reason: 

https://wfca.com/wildfire-articles/california-fire-season-in-depth-guide/#:~:text=Fires%20are%20possible%20throughout%20the,June)%20and%20runs%20until%20October.

https://www.usgs.gov/3d-elevation-program/about-3dep-products-services

https://www.sciencedirect.com/science/article/pii/S1574954125004443#s0030

https://gacc.nifc.gov/nrcc/predictive/fuels_fire-danger/psa_component_glossary.htm

https://www.ala.org/magirt/publicationsab/ca