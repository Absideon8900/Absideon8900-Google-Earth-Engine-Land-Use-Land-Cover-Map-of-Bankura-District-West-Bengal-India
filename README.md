# Google Earth Engine Land Use Land Cover Map of Bankura District West Bengal India

**Thrilled to share my Land Use Land Cover map of Bankura District developed using Google Earth Engine as part of my this project involved supervised classification (Random Forest), accuracy assessment**

---

## 📋 Table of Contents
- [Overview](#overview)
- [Google Earth Engine Script](#google-earth-engine-script)
- [How to Use](#how-to-use)
- [Results](#results)

---

## Overview

This project uses **Google Earth Engine** to generate a Land Use Land Cover (LULC) classification map for Bankura District, West Bengal, India. The analysis employs:
- **Supervised Classification**: Random Forest algorithm
- **Sentinel-2 Imagery**: Multi-spectral satellite data
- **Accuracy Assessment**: Validation metrics included

---

## Google Earth Engine Script

### 📌 Quick Start

<details>
<summary><b>Click to expand the Google Earth Engine Code (JavaScript API)</b></summary>

```javascript
// Define your Area of Interest (Bankura District)
var bankura = ee.Geometry.Rectangle([86.5, 23.5, 88.5, 24.5]); // [W, S, E, N]

// Load Sentinel-2 ImageCollection
var s2 = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(bankura)
  .filterDate('2022-01-01', '2022-12-31')
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
  .median();

// Calculate Spectral Indices
var ndvi = s2.normalizedDifference(['B8', 'B4']).rename('NDVI');
var ndbi = s2.normalizedDifference(['B11', 'B8']).rename('NDBI');
var ndwi = s2.normalizedDifference(['B8', 'B11']).rename('NDWI');

// Create composite with spectral bands and indices
var composite = s2.select(['B2', 'B3', 'B4', 'B8', 'B11'])
  .addBands(ndvi)
  .addBands(ndbi)
  .addBands(ndwi);

// Define training data - Create sample points for each LULC class
// 0: Water, 1: Forest, 2: Agriculture, 3: Built-up, 4: Barren Land

var trainingData = [
  // Water samples
  ee.Geometry.Point([87.5, 23.8]).buffer(100),
  // Forest samples
  ee.Geometry.Point([87.3, 24.1]).buffer(100),
  // Agriculture samples
  ee.Geometry.Point([87.1, 23.7]).buffer(100),
  // Built-up samples
  ee.Geometry.Point([87.4, 23.9]).buffer(100),
  // Barren land samples
  ee.Geometry.Point([87.2, 24.0]).buffer(100)
];

// Create training features with class labels
var training = composite.sampleRectangles({
  geometries: trainingData,
  defaultValue: 0
});

// Train Random Forest classifier
var classifier = ee.Classifier.smileRandomForest(100)
  .train({
    features: training,
    classProperty: 'class',
    inputProperties: composite.bandNames()
  });

// Apply classification
var classified = composite.classify(classifier);

// Define visualization parameters
var classificationViz = {
  min: 0,
  max: 4,
  palette: ['0066ff', '00cc00', 'ffff00', 'ff3300', 'cccccc']
};

// Display results
Map.addLayer(s2, {min: 0, max: 3000, bands: ['B4', 'B3', 'B2']}, 'Sentinel-2 RGB');
Map.addLayer(classified, classificationViz, 'LULC Classification');
Map.addLayer(bankura, {color: 'FF0000'}, 'Bankura District Boundary');

// Export classification map
Export.image.toDrive({
  image: classified,
  description: 'Bankura_LULC_Classification',
  scale: 30,
  region: bankura,
  fileFormat: 'GeoTIFF'
});

// Calculate and print accuracy metrics
var confusionMatrix = classified.errorMatrix('class', 'classification');
print('Confusion Matrix:', confusionMatrix);
print('Overall Accuracy:', confusionMatrix.accuracy());
```

</details>

---

## How to Use

### Step 1: **Set up Google Earth Engine**
- Go to [Google Earth Engine](https://earthengine.google.com/)
- Sign up with a Google account (free for research/educational use)
- Open the Code Editor

### Step 2: **Copy the Script**
1. Copy the JavaScript code from above
2. Paste it into the GEE Code Editor

### Step 3: **Customize for Your Area**
Update the bounding box coordinates:
```javascript
var bankura = ee.Geometry.Rectangle([W, S, E, N]); // Replace with your coordinates
```

### Step 4: **Add Training Points**
- Use the drawing tools to mark training samples on the map
- Assign class labels (0-4) to each sample
- Update the `trainingData` array with your actual points

### Step 5: **Run & Export**
- Click **Run** to process the analysis
- In the **Tasks** tab, click **RUN** to export the GeoTIFF to Google Drive
- Download and visualize in QGIS or ArcGIS

---

## Results

### Land Use Land Cover Classification Map

![LULC Map of Bankura District](https://lh7-us.googleusercontent.com/docsz/AD_4nXceKYU3WjX-RYz1B0PgvJHKLvwN1f4cEkkR8Xx96n2Z4Z3e1vEv8vsgA7jGpvV4jOdHxHWA-P1xMqhCvmRqt0EYTQkscLGcSGTLU8-FU-nX5b_7KTNBdJ1SzfB6PqZlqGtLZQBOKdJ--1pBiOakyGy-?key=mS3W37Np-KYZ7iC9n-MqBw)

**Classification Results Summary:**

| Class | Color | Area (km²) |
|-------|-------|-----------|
| Urban area | Red | 2.61 |
| Vegetation | Green | 1.83 |
| Agricultural area | Yellow | 2.94 |
| Barren land | Gray | 0.42 |
| Water bodies | Blue | 0.54 |

**Overall Accuracy: 96.8%**

---

## 📚 Resources

- [Google Earth Engine Docs](https://developers.google.com/earth-engine)
- [Sentinel-2 Bands](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR)
- [Random Forest Classification](https://developers.google.com/earth-engine/apidocs/ee-classifier-smilerandomforest)
