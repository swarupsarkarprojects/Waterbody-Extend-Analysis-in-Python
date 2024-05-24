
# Surface water extent time series analysis in Python


Waterbody extent analysis using Python involves the application of geospatial techniques to assess the coverage and characteristics of water bodies within a given area. Leveraging libraries such as GDAL, Fiona, and Rasterio, Python scripts can read, manipulate, and analyze raster and vector datasets containing information about water bodies. Through techniques like thresholding, segmentation, and classification, the extent of water bodies can be identified and quantified.


## API Reference

#### From Google Earth Engine for Python API

```http
  https://developers.google.com/earth-engine
```

| Parameter | Type     | Description                |
| :-------- | :------- | :------------------------- |
| `Earth Enigne API` | `Geospatial Imegary` | **Required**. Access the Geospatial Datasets |




## Follow the step for the Project

install all libary and import:

```bash
pip install earthengine-api
pip install geoplot

import geoplot
import geopandas
import geojson as gpd
import geemap
import pandas as pd
import matplotlib.pyplot as plt
import requests
import geojson
import ee
```

Authenticate and Initialize the earthengine package:

```bash
ee.Authenticate()
ee.Initialize()
```
## Open the geojson file and convert into featurecalss:

```bash
with open("C:/Users/Swarup/Desktop/Python/map.geojson") as json_file:
   gjson = json.load(json_file)

fc12 = ee.FeatureCollection(gjson)
```
## Create a DateRange for listed all spatial images:

```bash
startdate = 2021
enddate = 2021
year = ee.List.sequence(startdate, enddate)
month = ee.List.sequence(1, 12)
```

## Calculate NDWI index for extract Waterbody:

```bash
def addNDWI(img):
    ndwi = img.normalizedDifference(['B8','B3']).rename('ndwi')
    return img.addBands(ndwi)
```

## Create ImageCollection for Sentinel 2A:

```bash
datasets = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED') \
           .filterBounds(fc12) \
           .filterDate('2021-01-01','2021-12-31') \
           .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 10)) \
           .map(addNDWI)

datasets
```

## Add the NDWI image as a band Collection:

```bash
ndwi_collection = datasets.select('ndwi')
ndwi_collection
```

## Create a function to reduce all collection into DateRange:

```bash
monthly = ee.ImageCollection.fromImages(
    year.map(lambda y: month.map(lambda m: datasets.filter(ee.Filter.calendarRange(y, y, 'year'))
                                              .filter(ee.Filter.calendarRange(m, m, 'month'))
                                              .median()
                                              .set('year', y)
                                              .set('month', m))).flatten())

```

## Create a function for clip all datasets as per your ROI:

```bash
def clip_all(img):
    return img.clip(fc12)

```

## Listed all images in a Variable for Loop:

```bash
clip = monthly.map(clip_all)
list_of_images = clip.toList(clip.size())
n = list_of_images.size().getInfo()

list_of_images
```

## Listed all images in a Variable for Loop:

```bash
clip = monthly.map(clip_all)
list_of_images = clip.toList(clip.size())
n = list_of_images.size().getInfo()

n
```

## Create a Loop for Calculate Area of Waterbody and Classify into Waterbody and Non-Waterbody:

```bash
my_dic = {}

for i in range(n):
    img = ee.Image(list_of_images.get(i))
    ndwi = img.select('ndwi')
    classify = ndwi.gt(0)
    mask = classify.updateMask(classify)
    area = sum(mask*10)
    my_dic[i+1] = area

my_dic
```
## convert the Data into DataFrame:

```bash
df =pd.DataFrame(list(my_dic.items()), columns=['Month', 'Area_km2'])
```

## Plot the Data as per Scale:

```bash
df.plot(kind='line',
        x='Month',
        y='Area_km2',
        color='green')
```
## 🔗 Visit on Profile
[![Guihub](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://github.com/swarupsarkarprojects?tab=repositories)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/swarup-sarkar-9371251b4/)


