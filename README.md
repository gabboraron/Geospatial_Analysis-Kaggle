# Geospatial Analysis *- Kaggle Course -*
***Create interactive maps, and discover patterns in geospatial data.***

This is my notes from the course, sometimes this will be the same as the original one, sometimes not, sometimes it will be the same as other, in the sourcelist mentioned articles. 

sources:
- [Kaggle - Geospatial Analysis](https://www.kaggle.com/learn/geospatial-analysis)
- [Kaggle - Pandas](https://www.kaggle.com/learn/pandas)
- [GeoPandas](https://geopandas.org/en/stable/)
- [Shapefile vs. GeoJSON vs. GeoPackage by Juho Häme](https://feed.terramonitor.com/shapefile-vs-geopackage-vs-geojson/)
- []()

## Your First Map
Original article: [Your First Map by Alexis Cook](https://www.kaggle.com/code/alexisbcook/your-first-map) 

There are many, many different geospatial file formats, such as shapefile, GeoJSON, KML, and GPKG:

> #### Shapefile:
> Shapefile is the most widely known format for distributing geospatial data. It is a standard first developed by ESRI almost 30 years ago, which is considered ancient in software development.
>
> The Shapefile in fact consists of several files: in addition to one file with the actual geometry data, another file for defining the coordinate reference system is needed, as well as a file for defining the attributes and a file to index the geometries. This makes operating Shapefiles slightly clunky and confusing. However, Shapefile has been around for so long that any GIS software supports handling it.
> #### GeoJSON
> GeoJSON is a subset of JSON (JavaScript object notation). It was developed 10 years ago by a group of enthusiastic GIS developers. The core idea is to provide a specification for encoding geospatial data while remaining decodable by any JSON decoder.
> 
> Being a subset of the immensely popular JSON, the parsing support is on a different level than with Shapefile. In addition to support from most GIS software, any web developer will be able to write a custom GeoJSON parser, opening new possibilites for integrating the data. 
> #### GeoPackage
> GeoPackage was first developed by Open Geospatial Consortium (OGC) 5 years ago, making it the official alternative for Shapefile. It is a subset of SQLite, which in turn is a lighweight SQL implementation designed for stand-alone databases. Similar to GeoJSON, this makes GeoPackage highly compatible by design, and accessible by non-GIS software as well.
>
> The minor nuisances of Shapefile, such as non-standardized format for CRS and legacy limitations on attribute fields, have been fixed in GeoPackage. Internally, GeoPackage uses Well-known binary (WKB) for storing the geometries, the same as Shapefile. Interestingly, GeoPackage also provides support for storing raster data. QGIS handling performance with the training data set was on par with Shapefile.
> 
> | Format 	| Shapefile |	GeoJSON |	GeoPackage |
> | -------|-----------|--------|------------|
> |Age (years) | 	30 | 	10 |	5|
> |Compatibility |	GIS |	GIS, any text editor |	GIS, SQL|
> |Relative size |	1.00 |	2.26 |	1.30 |
> |Compression ratio |	4.79:1 |	12.08:1 |	4.53:1|
> |QGIS performance |	Good | 	Bad |	Good|
> |Use case | 	Old standard |	Web, small data sets |	New standard |

All of these file types can be quickly loaded with the `gpd.read_file()`
*ex:*
```Python
# Read in the data
full_data = gpd.read_file("../input/geospatial-learn-course-data/DEC_lands/DEC_lands/DEC_lands.shp")

# View the first five rows of the data
full_data.head()

# get type of the database, it will be: "geopandas.geodataframe.GeoDataFrame"
type(full_data)

# select a subset of the whole data
data = full_data.loc[:, ["CLASS", "COUNTY", "geometry"]].copy()

# How many lands of each type are there?
data.CLASS.value_counts()

# Select lands that fall under the "WILD FOREST" or "WILDERNESS" category
wild_lands = data.loc[data.CLASS.isin(['WILD FOREST', 'WILDERNESS'])].copy()
wild_lands.head()

# quickly visualize the data 
wild_lands.plot()

# View the first five entries in the "geometry" column
wild_lands.geometry.head()

## we create three more GeoDataFrames, containing campsite locations (Point), foot trails (LineString), and county boundaries (Polygon)
# Campsites in New York state (Point)
POI_data = gpd.read_file("../input/geospatial-learn-course-data/DEC_pointsinterest/DEC_pointsinterest/Decptsofinterest.shp")
campsites = POI_data.loc[POI_data.ASSET=='PRIMITIVE CAMPSITE'].copy()

# Foot trails in New York state (LineString)
roads_trails = gpd.read_file("../input/geospatial-learn-course-data/DEC_roadstrails/DEC_roadstrails/Decroadstrails.shp")
trails = roads_trails.loc[roads_trails.ASSET=='FOOT TRAIL'].copy()

# County boundaries in New York state (Polygon)
counties = gpd.read_file("../input/geospatial-learn-course-data/NY_county_boundaries/NY_county_boundaries/NY_county_boundaries.shp")

## setting a value for ax ensures that all of the information is plotted on the same map.


# Define a base map with county boundaries
ax = counties.plot(figsize=(10,10), color='none', edgecolor='gainsboro', zorder=3)

# Add wild lands, campsites, and foot trails to the base map
wild_lands.plot(color='lightgreen', ax=ax)
campsites.plot(color='maroon', markersize=2, ax=ax)
trails.plot(color='black', markersize=1, ax=ax)
```

## Coordinate Reference Systems
Original article: [Coordinate Reference Systems by Alexis Cook](https://www.kaggle.com/code/alexisbcook/coordinate-reference-systems/tutorial)

> Coordinate reference systems are referenced by European Petroleum Survey Group (EPSG) codes.
>
> This GeoDataFrame uses EPSG 32630, which is more commonly called the "Mercator" projection. This projection preserves angles (making it useful for sea navigation) and slightly distorts area.
>
> However, when creating a GeoDataFrame from a CSV file, we have to set the CRS. EPSG 4326 corresponds to coordinates in latitude and longitude.

To create a GeoDataFrame from a CSV file, we needed to use both Pandas and GeoPandas:
- We begin by creating a DataFrame containing columns with latitude and longitude coordinates.
- To convert it to a GeoDataFrame, we use `gpd.GeoDataFrame().`
- The `gpd.points_from_xy()` function creates Point objects from the latitude and longitude columns.

When plotting multiple GeoDataFrames, it's important that they all use the same CRS. In the code cell below, we change the CRS of the facilities GeoDataFrame to match the CRS of regions before plotting it. The `to_crs()` method modifies only the "geometry" column: all other columns are left as-is.

```Python
# The "Latitude" and "Longitude" columns are unchanged
facilities.to_crs(epsg=32630).head()
```

In case the EPSG code is not available in GeoPandas, we can change the CRS with what's known as the "proj4 string" of the CRS. For instance, the proj4 string to convert to latitude/longitude coordinates.


All three types of geometric objects have built-in attributes that you can use to quickly analyze the dataset. For instance, you can get the x- and y-coordinates of a Point from the x and y attributes, respectively. And, you can get the length of a LineString from the length attribute. Or, you can get the area of a Polygon from the area attribute.

