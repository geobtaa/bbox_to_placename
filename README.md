# Populating Spatial Coverage for DCAT Data Portals

This repository is for populating spatial coverage for records only with a bounding box. It is a reverse version of **<a href='https://github.com/BTAA-Geospatial-Data-Project/geonames'>geonames</a>** repository. 

The scripts can also be used for querying any records formatted in GBL Metadata Template. However, an adjustment to the names of the harvested metadata fields may be required first:

- Download
- Slug
- Publisher
- Code
- Bounding Box

### Environment Setup

We will be using **Anaconda 3** to edit and run scripts. Information on Anaconda installation can be found [here](https://docs.anaconda.com/anaconda/install/). Please note that all scripts are running on Python 3. 

Here are all dependencies needed to be installed properly: 

- [geopandas](https://geopandas.org/getting_started/install.html) 

- [shapely](https://pypi.org/project/Shapely/)

- [requests](https://requests.readthedocs.io/en/master/user/install/#install)

- [numpy](https://numpy.org/install/)

### How to Use

1. Inspect the csv file with metadata to check if it includes all the columns listed below
2. If the target state(s) hasn't included city or county bounding box files, you may need to 
   1. download county and city boundary files (GeoJSON or Shapefile) online
   2. run `city_boundary.ipynb` or `county_boundary.ipynb` to create boundary GeoJSONs first
      - if there exists regional data portals, you may need to run `merge_geojsons.ipynb` to merge them together
      - Note that manual changes are required for `city_boundary.ipynb` based on attributes. 
   3. run `city_bbox.ipynb` or `county_bbox.ipynb` to create bounding box GeoJSONs
3. Run `sjoin.ipynb` 



## Handling URL Errors

### Error Summary

| Error                        | Access Landing Page? | Access Download Link or ImageServer? | Solution                       |
| ---------------------------- | :------------------: | :----------------------------------: | ------------------------------ |
| 404 - Not Found              |          X           |                  X                   | Drop this record               |
| 500 - Internal Server Error  |          √           |                  X                   | Only keep the landing page URL |
| Wrong File Type              |          √           |                  √                   | Only keep the landing page URL |
| Could not access ImageServer |          √           |                  X                   | Only keep the landing page URL |
| Read Timeout                 |      It depends      |              It depends              | Manually check it              |



### More Explanations Based on Each Error

1. **404 - Not Found**

   - **Reason**

     We can’t access either the landing page or download link. 

   - **Solution** 

     Technically speaking, it’s a broken link, so just drop it. 

2. **500 - Internal Server Error**

   - **Reason** 

     We can access the landing page, but there’s something wrong with the download link on the server side.

   - **Solution** 

     Provide the data source without the download link. Let users decide if they want to download other types of data.

3. **Wrong File Type** -- [Vector Data] *Not a shapefile*

   - **Reason** 

     When querying the metadata, the script will treat any link ended with `.zip` as a shapefile. But, some of them are not shapefiles. 

   - **Solution** 

     Provide the data source without the download link. Let users decide if they want to download other types of data.

4. **Could not access ImageServer**  -- [Imagery Data] 

   - **Reason** 

     Currently, all TIFF data would throw this error. Here’s Esri’s [explanation](https://support.esri.com/en/technical-article/000012620). 

   - **Solution** 

     Provide the data source without the download link. Let users decide if they want to download other types of data.

5. **Read Timeout**

   - **Reason** 

      This is the most ambiguous one. In order to improve the efficiency of web scraping, **`timeout`** is necessary to prevent the script waiting forever. If it does not get       a response within a particular time period, just move to the next one. Failure to do so can cause the program to hang indefinitely. 

     The servers can become slow and unresponsive for many reasons. One reason might be the **gigantic file size**. According to the Python library [**Requests**](https://requests.readthedocs.io/en/master/user/advanced/), when making a request, the body of the response (the entire file) is downloaded immediately. But                  **`timeout`** is not a time limit on the *entire response download*; rather, an exception is raised if the server has not issued a response for **`timeout`** seconds (more precisely, this is the time before the server sends the first byte). 

   - **Solution**

     Try setting **`timeout`** as **3 seconds** first, and push all the records with timeout error into a new list. Then increase **`timeout`** up to **10 seconds** and loop      through the list, most of the links will get a response. If still get some unresponsive ones, manually check it in case accidently delete any valid records. Those            records will be flagged in the “**Title**” column using something like “*Manually check it!*”.



## Obtain File Size

The default content length is in **Byte**, currently it has been converted into **MB** and rounded to 4 decimal places.



## Something You May Want to Know 

### Why create county bounding box GeoJSON?

A Bounding box is typically described as an array of two coordinate pairs: **SW** (the minimum longitude and latitude) and **NE** (the maximum longitude and latitude). Therefore, the rectangle area it represents always exceeds the real one and overlaps each other. It may cause problems especially for features sharing the same border like counties. 

If we spatial join the bounding box of records with accurate county boundaries, there's a great chance of returning place names which actually have no spatial relationship. In order to improve the accuracy, we need to use regular rectangle area for both join features and target features. 

### How to determine spatial coverage?

<a href='https://geopandas.org/reference/geopandas.sjoin.html'>`geopandas.sjoin`</a> provides three match options: **intersects**, **contains** and **within**. The flow chart below demonstrates the decision-making process:

<img src="https://user-images.githubusercontent.com/66186715/107158001-e2e4ab80-694c-11eb-924f-d04937b8176d.png" width="700" />

