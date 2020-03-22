# ETL

Many of these notebooks are used to populate the data tables that populate the map.

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
