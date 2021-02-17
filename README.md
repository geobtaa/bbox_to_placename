# Populating Spatial Coverage for DCAT Data Portals

This repository is for populating spatial coverage for records only with a bounding box. It is a reverse version of **<a href='https://github.com/BTAA-Geospatial-Data-Project/geonames'>geonames</a>** repository. 

The scripts can also be used for querying any records formatted in GBL Metadata Template. However, an adjustment to the names of the harvested metadata fields may be required first:

- Download
- Slug
- Publisher
- Code
- Bounding Box

**How to use**

1. Inspect the csv file with metadata to check if it includes all the columns listed below
2. If the target state(s) hasn't included city or county bounding box files, you may need to 
   1. download county and city boundary files (GeoJSON or Shapefile) online
   2. run `city_boundary.ipynb` or `county_boundary.ipynb` to create boundary GeoJSONs first
      - if there exists regional data portals, you may need to run `merge_geojsons.ipynb` to merge them together
      - Note that manual changes are required for `city_boundary.ipynb` based on attributes. 
   3. run `city_bbox.ipynb` or `county_bbox.ipynb` to create bounding box GeoJSONs

3. Run `sjoin.ipynb` 

## Something you may want to know

#### Why create county bounding box GeoJSON?

A Bounding box is typically described as an array of two coordinate pairs: **SW** (the minimum longitude and latitude) and **NE** (the maximum longitude and latitude). Therefore, the rectangle area it represents always exceeds the real one and overlaps each other. It may cause problems especially for features sharing the same border like counties. 

If we spatial join the bounding box of records with accurate county boundaries, there's a great chance of returning place names which actually have no spatial relationship. In order to improve the accuracy, we need to use regular rectangle area for both join features and target features. 

#### How to determine spatial coverage?

<a href='https://geopandas.org/reference/geopandas.sjoin.html'>`geopandas.sjoin`</a> provides three match options: **intersects**, **contains** and **within**. The flow chart below demonstrates the decision-making process:

<img src="https://user-images.githubusercontent.com/66186715/107158001-e2e4ab80-694c-11eb-924f-d04937b8176d.png" width="700" />

​		
