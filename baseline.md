# Nanda Devi Biosphere Reserve (NDBR) Temperature Analysis

## Overview

This project uses Google Earth Engine to analyze historical temperature data over the Nanda Devi Biosphere Reserve (NDBR) using ERA5 Land Daily Aggregated datasets. The focus is on the baseline temperature conditions between 1961 and 1990.

## Data Sources

- **NDBR Boundaries:** `projects/ee-20109451/assets/WDPA_WDOECM_May2025_Public_902492_shp-polygons`
- **ERA5 Land Daily Aggregated:** `ECMWF/ERA5_LAND/DAILY_AGGR`

## Study Period

- Baseline period: January 1, 1961 to December 31, 1990

## Methodolgy

### Importing Datasets and setting the period
```
var ndbrGeometry = ee.FeatureCollection("projects/ee-20109451/assets/WDPA_WDOECM_May2025_Public_902492_shp-polygons")
var era5_daily = ee.ImageCollection("ECMWF/ERA5_LAND/DAILY_AGGR")
var baselinePeriod = ee.DateRange('1961-01-01', '1990-12-31');

```
### Retrieving Baseline max temperature data

```
var band = "temperature_2m_max"
var era5_temp_maximum = era5_daily.select(band).filterDate(baselinePeriod)
function aggregateMaximumOverRegion(image){
  var scale = image.select(band).projection().nominalScale();
  var meanOverRegion = image.reduceRegion({
    reducer : ee.Reducer.mean(),
    geometry: ndbrGeometry.geometry(),
    bestEffort:true,
    crs : era5_daily.select(band).first().projection(),
    scale : scale
  })
  
  var meanMaxTemp = meanOverRegion.get(band)
  return image.set("Mean_Maximum_Temperature" , meanMaxTemp)  
  
}

var aggregatedCollection  = era5_temp_maximum.map(aggregateMaximumOverRegion)

```
### Retrieving Baseline minimum temperature data

```
var band = "temperature_2m_min"

var era5_temp_maximum = era5_daily.select(band).filterDate(baselinePeriod)

function aggregateMinimumOverRegion(image){
   var scale = image.select(band).projection().nominalScale();
  var meanOverRegion = image.reduceRegion({
    reducer : ee.Reducer.mean(),
    geometry: ndbrGeometry.geometry(),
    bestEffort:true,
    crs : era5_daily.select(band).first().projection(),
    scale : scale
  })
  
  var meanMinTemp = meanOverRegion.get(band)
  return image.set("Mean_Minimum_Temperature" , meanMinTemp)
  
  
}

var aggregatedCollection  = era5_temp_maximum.map(aggregateMinimumOverRegion)

```

### Setting features

```

var features = aggregatedCollection.map(function(image) {
  
  var meanTemp =  ee.Number(image.get('Mean_Maximum_Temperature'))
  var meanTempInCelcius = meanTemp.subtract(273.15)
  return ee.Feature(null, {
    'date': image.date().format('YYYY-MM-DD'),
    'mean_max_temp':meanTempInCelcius
  });
});

var features = aggregatedCollection.map(function(image) {
  
  var meanTemp =  ee.Number(image.get('Mean_Minimum_Temperature'))
  var meanTempInCelcius = meanTemp.subtract(273.15)
  return ee.Feature(null, {
    'date': image.date().format('YYYY-MM-DD'),
    'mean_min_temp':meanTempInCelcius
  });
});

```
### Exporting to CSV

```

Export.table.toDrive({
  collection: features,
  description: 'Mean_Max_Temp_NDBR_Baseline',
  fileFormat: 'CSV'
});
Export.table.toDrive({
  collection: features,
  description: 'Mean_Min_Temp_NDBR_Baseline',
  fileFormat: 'CSV'
});


```





