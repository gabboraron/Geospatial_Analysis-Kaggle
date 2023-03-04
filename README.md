# Geospatial Analysis *- Kaggle Course -*
***Create interactive maps, and discover patterns in geospatial data.***

This is my notes from the course, sometimes this will be the same as the original one, sometimes not, sometimes it will be the same as other, in the sourcelist mentioned articles. 

sources:
- [Kaggle - Geospatial Analysis](https://www.kaggle.com/learn/geospatial-analysis)
- [Kaggle - Pandas](https://www.kaggle.com/learn/pandas)
- [GeoPandas](https://geopandas.org/en/stable/)
- [Shapefile vs. GeoJSON vs. GeoPackage by Juho Häme](https://feed.terramonitor.com/shapefile-vs-geopackage-vs-geojson/)
- [Data Renaming and Combining by Aleksey Bilogur](https://www.kaggle.com/code/residentmario/renaming-and-combining/tutorial)

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

## Interactive Maps
For this we will use `folium` package:

Several map styles available: https://github.com/python-visualization/folium/tree/main/folium/templates/tiles

```Python
import folium
from folium import Choropleth, Circle, Marker
from folium.plugins import HeatMap, MarkerCluster

# Create a map
m_1 = folium.Map(location=[42.32,-71.0589], tiles='openstreetmap', zoom_start=10)
  #  sets the initial center of the map. We use the latitude (42.32° N) and longitude (-71.0589° E) of the city of Boston. 
  # tiles changes the styling of the map; in this case, we choose the OpenStreetMap style.
  # zoom_start sets the initial level of zoom of the map, where higher values zoom in closer to the map.
  
# Display the map
m_1
```

Now, we'll add some data to the map! We won't focus on the data loading step. Instead, you can imagine you are at a point where you already have the data in a pandas DataFrame. 

We add markers to the map with `folium.Marker()`.

If we have a lot of markers to add, `folium.plugins.MarkerCluster()` can help to declutter the map. Each marker is added to a MarkerCluster object.
```Python
# Create the map
m_3 = folium.Map(location=[42.32,-71.0589], tiles='cartodbpositron', zoom_start=13)

# Add points to the map
mc = MarkerCluster()
for idx, row in daytime_robberies.iterrows():
    if not math.isnan(row['Long']) and not math.isnan(row['Lat']):
        mc.add_child(Marker([row['Lat'], row['Long']]))
m_3.add_child(mc)

# Display the map
m_3
```

### Bubble maps
A bubble map uses circles instead of markers. By varying the size and color of each circle, we can also show the relationship between location and two other variables.
We create a bubble map by using `folium.Circle()` to iteratively add circles. 

The arguments:
- `location` is a list containing the center of the circle, in latitude and longitude.
- `radius` sets the radius of the circle.
  *Note that in a traditional bubble map, the radius of each circle is allowed to vary. We can implement this by defining a function similar to the color_producer() function that is used to vary the color of each circle.*
- `color` sets the color of each circle.
  *The color_producer() function is used to visualize the effect of the hour on robbery location.*

```Python
# Create a base map
m_4 = folium.Map(location=[42.32,-71.0589], tiles='cartodbpositron', zoom_start=13)

def color_producer(val):
    if val <= 12:
        return 'forestgreen'
    else:
        return 'darkred'

# Add a bubble map to the base map
for i in range(0,len(daytime_robberies)):
    Circle(
        location=[daytime_robberies.iloc[i]['Lat'], daytime_robberies.iloc[i]['Long']],
        radius=20,
        color=color_producer(daytime_robberies.iloc[i]['HOUR'])).add_to(m_4)

# Display the map
m_4
```

### Heatmaps
To create a heatmap, we use `folium.plugins.HeatMap()`. This shows the density of crime in different areas of the city, where red areas have relatively more criminal incidents.

The arguments:
- `data` is a DataFrame containing the locations that we'd like to plot.
- `radius` controls the smoothness of the heatmap. Higher values make the heatmap look smoother (i.e., with fewer gaps).

```Python
# Create a base map
m_5 = folium.Map(location=[42.32,-71.0589], tiles='cartodbpositron', zoom_start=12)

# Add a heatmap to the base map
HeatMap(data=crimes[['Lat', 'Long']], radius=10).add_to(m_5)

# Display the map
m_5
```

### Choropleth maps
To understand how crime varies by police district, we'll create a choropleth map.

As a first step, we create a GeoDataFrame where each district is assigned a different row, and the "geometry" column contains the geographical boundaries.
```Python
# GeoDataFrame with geographical boundaries of Boston police districts
districts_full = gpd.read_file('../input/geospatial-learn-course-data/Police_Districts/Police_Districts/Police_Districts.shp')
districts = districts_full[["DISTRICT", "geometry"]].set_index("DISTRICT")
districts.head()

# Number of crimes in each police district
plot_dict = crimes.DISTRICT.value_counts() #It's very important that plot_dict has the same index as districts - this is how the code knows how to match the geographical boundaries with appropriate colors.
plot_dict.head()

# Create a base map
m_6 = folium.Map(location=[42.32,-71.0589], tiles='cartodbpositron', zoom_start=12)

# Add a choropleth map to the base map
Choropleth(geo_data=districts.__geo_interface__, #geo_data is a GeoJSON FeatureCollection containing the boundaries of each geographical area; we convert the districts GeoDataFrame to a GeoJSON FeatureCollection with the __geo_interface__ attribute.
           data=plot_dict, #data is a Pandas Series containing the values that will be used to color-code each geographical area. 
           key_on="feature.id", #key_on will always be set to feature.id. 
           fill_color='YlGnBu', #sets the color scale. 
           legend_name='Major criminal incidents (Jan-Aug 2018)' #labels the legend in the top right corner of the map.
          ).add_to(m_6)

# Display the map
m_6
```

## Manipulating Geospatial Data
Geocoding is the process of converting the name of a place or an address to a location on a map. If you have ever looked up a geographic location based on a landmark description with Google Maps, Bing Maps, or Baidu Maps, for instance, then you have used a geocoder!

We begin by instantiating the geocoder. Then, we need only apply the name or address as a Python string. (In this case, we supply "Pyramid of Khufu", also known as the Great Pyramid of Giza.) If the geocoding is successful, it returns a `geopy.location.Location` object with two important attributes:

`from geopy.geocoders import Nominatim`
- the "point" attribute contains the (latitude, longitude) location, and
- the "address" attribute contains the full address.

The value for the "point" attribute is a geopy.point.Point object, and we can get the latitude and longitude from the latitude and longitude attributes, respectively.

```Python
geolocator = Nominatim(user_agent="kaggle_learn")
location = geolocator.geocode("Pyramid of Khufu")

print(location.point)
print(location.address)
```

## Proximity Analysis - *Measure distance, and explore neighboring points on a map*
- measure the distance between points on a map, and
- select all points within some radius of a feature.

To measure distances between points from two different GeoDataFrames, we first have to make sure that they use the same coordinate reference system (CRS). We also check the CRS to see which units it uses (meters, feet, or something else). In this case, EPSG 2272 has units of feet.

It's relatively straightforward to compute distances in GeoPandas. The code cell below calculates the distance (in feet) between a relatively recent release incident in recent_release and every station in the stations GeoDataFrame.

```Python
# Select one release incident in particular
recent_release = releases.iloc[360]

# Measure distance from release to each station
distances = stations.geometry.distance(recent_release.geometry)
distances
```

If we want to understand all points on a map that are some radius away from a point, the simplest way is to create a buffer.

We use folium.GeoJson() to plot each polygon on a map. Note that since folium requires coordinates in latitude and longitude, we have to convert the CRS to EPSG 4326 before plotting.


