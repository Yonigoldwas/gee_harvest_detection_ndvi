Harvest detection for corn and soy in Sentinel 2 MGRS tiles
Overview

This script detects harvested corn and soybean fields inside a single Sentinel 2 MGRS tile using NDVI change between two time windows.

You draw an area of interest in the GEE Code Editor. The script then

Finds all Sentinel 2 tiles touching that geometry for the chosen period

Selects the single tile whose footprints overlap your geometry the most

Runs the full analysis on the entire footprint of that chosen tile

Estimates harvested area for corn and soy within that tile and reports areas in hectares

Processing is intentionally done on the full tile footprint rather than clipping to the user geometry, so results are tile consistent and not influenced by the drawn shape.

Data sources

The script uses the following Earth Engine collections

Sentinel 2 surface reflectance harmonized collection

ID COPERNICUS/S2_SR_HARMONIZED

Sentinel 2 cloud probability collection

ID COPERNICUS/S2_CLOUD_PROBABILITY

USDA Cropland Data Layer for crop masks

ID USDA/NASS/CDL

User inputs

At the top of the script there is a user parameters section. You normally only need to adjust these values.

Time windows

preStart and preEnd

First time window before expected harvest

Should represent green crop conditions

postStart and postEnd

Second time window after expected harvest

Should represent mostly bare or very low vegetation cover

Make sure the windows do not overlap and both are within the same broad harvest period for your region.

NDVI thresholds

preMinNDVI

Minimum NDVI in the pre window for a pixel to be considered green crop

postMaxNDVI

Maximum NDVI in the post window for a pixel to be considered bare or harvested

ndviDropThr

Maximum allowed value of the change NDVI post minus NDVI pre

Should be a negative value representing the drop in NDVI from pre to post

There is a note in the script that the optimal NDVI change threshold needs more exploration. You are expected to tune these values for your region, season, and crop condition.

Cloud masking

cloudProbThresh

Threshold on Sentinel 2 cloud probability (range zero to one hundred)

Lower values give stricter masking and cleaner scenes but may remove more pixels

The script uses S2 cloud probability when available and falls back to QA60 bit masks when cloud probability is missing.

Patch filtering

minPatchPixels

Minimum connected pixel count for accepted harvest patches

Used to remove speckle and tiny isolated detections

Values correspond to ten meter pixels

CDL year and crop masking

cdlYear

Year of Cropland Data Layer used to label corn and soy

If requested year is not available, the script falls back to the nearest available year in the collection

maskNDVIToCrops

Boolean flag

When true, NDVI and NDVI change are considered only on pixels mapped as corn or soy in CDL

When false, NDVI is computed everywhere and CDL is used only later to separate corn and soy detections

Export settings

exportScale

Pixel size for exports in meters

exportFolder

Name of the folder in your Google Drive where images will be exported

Exports are currently commented out. You can enable them once you are satisfied with the results.

Geometry input and tile selection

Draw a polygon named geometry in the GEE Code Editor

The map is centered on this polygon

The script finds all Sentinel 2 scenes that intersect your geometry during the union of the pre and post windows

It lists all MGRS tiles for those scenes

For each tile it computes the area of the intersection between

the union of that tile footprint over the consolidated date range

the user drawn geometry

The tile with the largest intersecting area is selected

The chosen tile ID and the overlapping area in hectares are printed in the Console. The outline of the chosen tile is drawn on the map.

Although the geometry is used to choose a tile, all further NDVI and harvest calculations are performed over the full tile footprint, not the original geometry. The tile geometry is only used as a region for statistics and exports.

Cleanest scene and NDVI computation

For each time window (pre and post) and for the chosen tile, the script

Filters Sentinel 2 surface reflectance images by dates and tile ID

Filters S2 cloud probability images by the same dates and tile ID

Joins the two collections by system index

For each joined image, computes the fraction of clear pixels within the tile union geometry, based on the cloud probability threshold

Selects the image with the highest clear fraction when cloud probability is available

If no cloud probability image is present, selects the scene with the smallest CLOUDY_PIXEL_PERCENTAGE value

From the selected scene it then

Computes NDVI using B8 and B4

Builds an RGB image using B4, B3, B2

Applies either the cloud probability mask or QA60 cloud and cirrus bits

Adds metadata about the selected scene date, ID, mask method, and cloudiness

If no suitable scene is found in a window, an empty NDVI and RGB stack is returned so downstream logic does not fail.

Harvest detection logic

After computing NDVI for both windows on the full tile footprint, the script

Computes NDVI change as NDVI post minus NDVI pre

Optionally masks NDVI and NDVI change to CDL corn and soy pixels when maskNDVIToCrops is true

Builds three logical conditions for each pixel

Pre green condition NDVI pre greater than or equal to preMinNDVI

Post bare condition NDVI post less than or equal to postMaxNDVI

Strong drop condition NDVI change less than or equal to ndviDropThr

A pixel is considered a harvest candidate when all three conditions are true

The candidate mask is combined with CDL crop masks to produce

harvestCorn

harvestSoy

A minimum patch size filter is applied to each crop detection using connected pixel count

A class image is built

value zero no harvest

value one harvested corn

value two harvested soy

The final harvest class image is clipped by the tile geometry only for display or export region control. Internally the analysis already covers the full tile footprint.

Visualization layers

The script adds several layers to the map

Pre window RGB true color

Post window RGB true color

Pre window NDVI

Post window NDVI

NDVI change image

Harvest class image with separate colors for corn and soy

These layers are created from the full tile footprint with cloud masks applied.

Area statistics

To summarize harvested area in the chosen tile, the script

Builds a pixel area image in hectares

Applies masks for harvested corn and harvested soy

Sums area values over the tile geometry using ten meter scale

Prints harvested area for corn and soy in hectares to the Console

This gives you a quick estimate of harvested corn and soy area within the selected tile.

Exports

Two image exports are prepared and commented out

Harvest class image as an integer raster with classes zero one two

Diagnostic NDVI stack with NDVI pre, NDVI post, and NDVI change

To enable exports

Uncomment the Export.image.toDrive calls near the end of the script

Check that exportFolder exists in your Google Drive or adjust the folder name

Run the script and monitor the Tasks panel in GEE

Exports are bounded by the tile geometry for manageable file sizes but the internal processing remains tile wide.

How to run

Open the script in the Earth Engine Code Editor

Draw or import a polygon and rename it to geometry

Adjust the user parameters section at the top

Dates for pre and post windows

NDVI thresholds

Cloud probability threshold

Patch size threshold

CDL year

Optionally set maskNDVIToCrops to true if you want NDVI logic restricted to CDL corn and soy

Run the script

Inspect

Printed logs for chosen tile ID and dates of selected scenes

Map layers for NDVI, NDVI change, and harvest classes

Printed harvested areas in the Console

Once satisfied, enable exports and rerun to generate output rasters

Tuning guidance

Some practical tips for tuning

Time windows

Choose pre window when crops are fully green and close to peak canopy

Choose post window after harvest but before significant regrowth or residue green up

NDVI thresholds

Start with moderately strict thresholds for green and bare conditions

Inspect histograms or map values over representative fields to refine thresholds

NDVI drop threshold

Larger drops are more conservative and reduce false positives but may miss partial harvest or dry down only

Smaller drops increase sensitivity but risk cloud shadow or other noise triggering detections

Patch size

Set minPatchPixels according to minimum field size you care about

Because the script works at tile scale, you can reuse the same configuration over larger regions by simply changing the geometry and re running, while keeping a consistent detection logic per tile.
