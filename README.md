# Populating Place Name Based on Bounding Box

This repository is for populating spatial coverage for records only with a bounding box. It is a reverse version of **<a href='https://github.com/BTAA-Geospatial-Data-Project/geonames'>geonames</a>** repository. It implements `geopandas.sjoin` to find out where these records belong based on county bounding box. 

**How to use**

1. Create a new `code` folder under `data` folder, and move csv file into this folder. 
2. Determine which state(s) metadata falls into using source data portal.
3. Go to `geojson` folder to see if there exists county bounding box GeoJSON of target state(s). 
   - If metadata comes from one state, run `geojson_to_bbox.ipynb` if necessary. Copy this GeoJSON file to `data` folder > `code` folder. 
   - Otherwise, if metadata is pulled out of a regional data portal, copy all relevant state GeoJSON files to `data` folder > `code` folder, and then run `merge_geojson.ipynb` to merge them. 

3. Run `sjoin.ipynb` to produce `code_placename.csv`

## Something you may want to know

#### Why create county bounding box GeoJSON?

A Bounding box is typically described as an array of two coordinate pairs: **SW** (the minimum longitude and latitude) and **NE** (the minimum longitude and latitude). Therefore, the rectangle area it represents always exceeds the real one and overlaps each other. It may cause problem especially for features sharing the same border like counties. 

If we join the bounding box of records with accurate county boundaries, there's a great chance of returning place names which actually have no spatial relationship. In order to improve the accuracy, we need to use regular rectangle area for both join features and target features. 

#### How to determine spatial coverage?

<a href='https://geopandas.org/reference/geopandas.sjoin.html'>`geopandas.sjoin`</a> provides three match options: **intersects**, **contains** and **within**. The flow chart below demonstrates the decision-making process:

<img src="https://user-images.githubusercontent.com/66186715/106498107-884adb80-6484-11eb-9f7c-ebbe4e4ee0e4.png" width="400" />

​		
