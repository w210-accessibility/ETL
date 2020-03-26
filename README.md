# ETL

Transforming data from various public sources to be used by the Sidewaukee website.

## ArcGIS Data
To get csvs of intersections, sidewalk segments, and elevation/road grade data, you need to use ArcGIS to manipulate shapefiles. These steps assume you're using ArcGIS Pro unless otherwise stated.

### Shapefiles to Download
1. Roads Shapefile for the city/county
* `data.gov` usually has these. For Milwaukee County we pulled them from [here](https://catalog.data.gov/dataset/tiger-line-shapefile-2015-county-milwaukee-county-wi-all-roads-county-based-shapefile)
2. Contour line shapefile for elevation data
* These are sometimes available on `data.gov`. E.g. Milwaukee has [these](https://catalog.data.gov/dataset/tiger-line-shapefile-2015-county-milwaukee-county-wi-all-roads-county-based-shapefile), though we ultimately downloaded our elevation data from [here](http://mclio.maps.arcgis.com/apps/webappviewer/index.html?id=84c7b8d95af04cdda6b0c2ae26590531). 

### Remove Highways from Roads Shapefile
Interstates typically don't have sidewalks, so for our purposes, we wanted to remove those roads from our dataset. In ArcGIS, you can use the Select By Attributes feature to select only certain road segments based on their attributes (e.g. roads tagged as "I" for Interstate, or roads without a street name, which typically correspond to on/off ramps for highways) and then delete the selected segments. 

### Create Intersections Dataset
A great tutorial for creating intersection points from a roads shapefile is [here](https://www.esri.com/arcgis-blog/products/arcgis-desktop/analytics/more-adventures-in-overlay-creating-a-street-intersection-list/). The high-level steps for creating these in ArcGIS are below:

1. Run Unsplit Line
* This step combines all street segments with the same road name, which are sometimes already split at non-intersection points depending on how the roads shapefile is generated.
  * Dissolve Field: *street name field*
2. Run Intersect
* This step creates intersection points at every location where the roads shapefile overlaps itself. The tool will copy the feature ID from the intersecting roads shapefile into the new point shapefile so you can map back from the intersection point to the 4 intersecting road segments. 
  * Input: *output from step 1*
  * Output type: *point*
  * Attributes to Join: *Only feature IDs*
3. Add Attributes for Lat/Long
* Right click on the intersections layer and click Attribute Table. Click Add and add rows for Lat and Long (type: Double).
4. Run Calculate Geometry Attributes
* This step will add the lat/long coordinates to each intersection point.
  * Input features: *intersections layer*
  * Target field/Property: *Lat/Centroid Y-Coordinate*
  * Target field/Property: *Long/Centroid X-Coordinate*
5. Run Table to Excel
* This step will output the intersection features to an excel file. This will include the IDs of the intersecting streets as well as the Lat/Long of each intersection point.
  * Input Table: *intersections layer*



### Offset Roads Left/Right for Sidewalks
**This first step is in ArcGIS Pro.**
1. Add a new attribute to the cleaned up roads shapefile you created in step 2 of Create Intersections Dataset.
2. Click "Calculate" in the Attribute Table, and set this new attribute equal to the Object ID of this segment.
This step is necessary to make sure your left/right sidewalks are linked to the original road centerline's Object ID.

**For the following steps, you must use ArcMap. Using the same features in ArcGIS Pro leads to very bad performance issues and likely won't work.**

1. Add the roads shapefile from above as a layer
2. Go to Editor > Start Editing
3. Select all road segments and go to Editor > Copy Parallel
* Note: this will add the left/right sidewalk elements to the same layer as your original road segments. This is okay. We will fix this later.
* Assuming your city isn't very close to either pole, you can roughly convert feet to decimal degrees using the formula `364,567.2 feet = 1 degree`. 
  * Distance: *~20-35 ft offset, converted to decimal degrees*
  * Side: *Do left/right separately if your roadlines tend to be off-center like ours in order to get them to roughly align*
  * Remove self-intersecting loops: *unchecked*

4. Go to Editor > Save Edits and Editor > Stop Editing.
5. Go to Selection > Select by Attributes
* Select where OBJECT_ID <> the copy of the OBJECT ID you created at the start of this section
6. Right click on your layer in the Table of Contents and click Data > Export Data
* Save only the selected segments as a new layer. 
7. Go to Editor > Start Editing and delete the selected segments.
8. Repeat steps 2-7 for the other side of the road.

**These last steps are in ArcGIS Pro**
1. Add the left/right sidewalk features created in ArcMap.
2. Use Features to JSON to export these layers to GeoJSON files
* Input Features: *left/right sidewalk*
* Output to GeoJSON: *checked*
* Project to WGS_1984: *checked*


### Create Road Grades Dataset
These steps transform elevation contours into a road grade per road segment. 

#### ArcGIS Steps
1. Load contours and updated road segments shapefiles into ArcGIS
2. Run Intersect
* This step will create elevation points everywhere the contours cross road segments
  * Input Features: *Contours, roads layers*
  * Attributes to Join: *All attributes*
  * Output Type: *Point*

3. Use Table to Excel to export these points to an excel file

#### Jupyter Notebook
1. Run the `converting_elevation_to_road_grade.ipynb` notebook to convert the individual elevation points to road grades per sidewalk segment.


## Sidewalk Segment Process
**new_sidewalk_segments_data.ipynb** - old version of march15_data_loading.ipynb

**march15_data_loading.ipynb** - takes in right/left sidewalk geojson files, plus road grade files, from ArcGis
folder and uses them to populate SidewalkSegment table

**sidewalk_pano_bridge_table_loading.ipynb** -- takes in CSV spit out from march15_data_loading
notebook, plus pano/road bridge data, and uses it to populate SidewalkToPano table

^ we use code currently in mvp_endpoint to generate json files containing the three
lists of sidewalks (each list represents a particular passability)

**convert_json_to_geojson.ipynb** -- takes in json files spit out from mvp_endpoint
and turns them into GeoJson, as needed by Mapbox tileset uploader

## Missing curb ramp process

**make_fake_preds.ipynb** -- populates the Panofeature table with fake predictions.
These fake predictions should be replaced by real ones soon :)

**missing_curb_ramps.sql** -- Query to get the missing curb ramp locations from the data

**make_missing_curb_ramp_dataset.ipynb** -- Takes in the output of the above query
and manipulates it to make the missing curb ramp geojson, as needed by Mapbox tileset uploader 

