# Harvest Detection (Corn & Soy) Using Sentinel-2 MGRS Tiles

This repository contains a Google Earth Engine script that detects harvested corn and soybean fields based on NDVI change between two time windows. The workflow automatically selects the single Sentinel-2 MGRS tile that overlaps the user-drawn geometry the most, then processes the full tile footprint (no clipping) for consistent analysis.

---

## Features

* Selects the dominant overlapping MGRS tile from your drawn geometry  
* Computes NDVI for pre-harvest and post-harvest windows using the cleanest cloud-free scenes available  
* Applies crop masks from USDA CDL to isolate corn and soybean fields  
* Detects harvested pixels using NDVI thresholds and NDVI drop  
* Removes speckle using connected component filtering  
* Produces a harvest classification map for the full MGRS tile footprint  
* Summarizes harvested area of corn and soy in hectares  
* Optional exports of harvest class and NDVI diagnostic layers  

---

## Data Sources

* **Sentinel-2 SR (Harmonized)**  
  Collection: `COPERNICUS/S2_SR_HARMONIZED`

* **Sentinel-2 Cloud Probability**  
  Collection: `COPERNICUS/S2_CLOUD_PROBABILITY`

* **USDA Cropland Data Layer (CDL)**  
  Collection: `USDA/NASS/CDL`

---

## User Parameters

These parameters control the analysis and are defined at the top of the script.

### Time Windows
Defines the two periods used for NDVI comparison.

* `preStart` / `preEnd`  
  Dates when crop should still be green before harvest  
* `postStart` / `postEnd`  
  Dates after harvest when fields are expected to be bare

### NDVI Thresholds
Used to decide whether a pixel qualifies as harvested.

* `preMinNDVI`  
  Minimum NDVI in the pre window  
* `postMaxNDVI`  
  Maximum NDVI in the post window  
* `ndviDropThr`  
  NDVI difference threshold `post minus pre`

### Cloud Masking
* `cloudProbThresh`  
  Cloud probability cutoff from S2 cloud probability images  
  Lower values mean stricter cloud filtering

### Patch Filtering
* `minPatchPixels`  
  Minimum connected component size to reduce speckle (10 m pixels)

### CDL Crop Masking
* `cdlYear`  
  CDL year to be used for corn and soy maps  
* `maskNDVIToCrops`  
  Option to compute NDVI only on corn and soy pixels

### Export Options
* `exportScale`  
  Resolution for exported rasters  
* `exportFolder`  
  Drive folder name for exports

---

## Workflow

### 1. Draw Geometry
Draw a polygon named `geometry` in the GEE Code Editor.  
This geometry is used only to choose the MGRS tile.

### 2. Tile Selection
The script finds all Sentinel-2 scenes intersecting your geometry and identifies the MGRS tile with the largest footprint overlap.

### 3. Scene Selection and NDVI Computation
For both pre and post windows:

* Sentinel-2 SR scenes for the tile are filtered  
* Sentinel-2 cloud probability is joined  
* A clear-pixel fraction is computed  
* The cleanest scene is selected  
* NDVI and RGB layers are produced using cloud probability (or QA60) masking

If no valid scene exists, an empty placeholder NDVI image is returned.

### 4. Harvest Detection
* Compute NDVI drop `post minus pre`  
* Create masks for  
  * Green before  
  * Bare after  
  * Strong NDVI drop  
* Combine these to form a harvest candidate mask  
* Mask separately with CDL corn and CDL soy layers  
* Apply connected pixel filtering  
* Build a harvest class raster  
  * zero  no harvest  
  * one  corn  
  * two  soy

### 5. Area Statistics
Using the full tile footprint:

* Calculate pixel area in hectares  
* Sum area of harvested corn  
* Sum area of harvested soy  
* Print results to the console

### 6. Visualization
The script provides map layers for

* RGB before  
* RGB after  
* NDVI before  
* NDVI after  
* NDVI change  
* Harvest class

### 7. Exports
Two optional exports are included but commented out:

* Harvest class image  
* NDVI diagnostics stack

To export, uncomment the corresponding `Export.image.toDrive` blocks.

---

## How to Use

1. Open in Earth Engine Code Editor  
2. Draw a polygon and rename it to `geometry`  
3. Adjust user parameters at the top of the script  
4. Run the script  
5. View logs, inspect map layers, and tune thresholds  
6. Enable exports when satisfied with results

---

## Notes and Recommendations

* Tune `preMinNDVI`, `postMaxNDVI`, and `ndviDropThr` using local conditions  
* Ensure time windows properly represent green crop state and post-harvest bare state  
* Patch filtering helps remove isolated NDVI artifacts  
* Since analysis is done over the full tile, results remain comparable when reusing settings over different geometries

## Author

Remote sensing agronomy workflow for operational harvest detection using Google Earth Engine.

