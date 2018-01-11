# GDAL Snipets

#### Useful examples working with [GDAL](http://www.gdal.org/)

### gdaladdo

Add overviews to *.tif files in batch mode
```cpp
for i in `find *.tif`; do gdaladdo -r average $i 2 4 8 16 32; done
```

### ogrinfo

Create spatial index (*.qix) to shapefiles in batch mode
```cpp
for i in `find *.shp |cut -d '.' -f 1 ` ; do echo ogrinfo -sql "CREATE SPATIAL INDEX ON $i"  $i.shp; done
```

### gdalbuildvrt

Create virtual raster from a folder containing Geotiff files
```cpp
gdalbuildvrt file.vrt folder_name/*.tif
```

Create virtual raster from a folder containing PNG World files
```cpp
gdalbuildvrt file.vrt folder_name/*.png
```

### gdal2tiles

Create tiles to disk from GeoTiff file. Zoom level 6 to 18
```cpp
gdal2tiles.py file.tif floder_name  -z 6-18 -r antialias
```

### gdalwarp

PNG to GeoTiff and reprojection UTM 31N-ETRS89 to PseudoMercator (Google)
```cpp
gdalwarp -s_srs EPSG:25831 -t_srs EPSG:3857 /home/geostart/file.png /home/geostart/file.tif
```

Convert a Virtual Raster (.vrt) file to another format 
```cpp
gdalwarp -of GTiff source.vrt output.tif
```

### gdal_grid

Virtual file with ascii point data to GeoTiff
```cpp
gdal_grid  -of GTiff -ot Float64 -l point_layer /home/geostart/file.vrt /home/geostart/file.tif
```

### gdaldem

Create a hillsahe from a dem Geotiff
```cpp
gdaldem hillshade /home/geostart/filedem.tif   /home/geostart/filehillshade.tif
```

### ogr2ogr

Convert Shapefile to GeoPackege
```cpp
ogr2ogr -f GPKG /home/geostart/file.gpkg /home/geostart/file.shp
```
Convert  CSV with WKT geom field to GeoJSON
```cpp
ogr2ogr -f "GeoJson" "file.geojson" "file.csv" -sql "SELECT *, CAST(fieldname as geometry) FROM file"
```

Shapefile to GeoJSON force geometry creation
```cpp
ogr2ogr -f "GeoJson" "/home/geostart/file.geojson"  "/home/geostart/file.shp" -nlt POLYGON
```

Shapefile to PostGIS
```cpp
ogr2ogr -overwrite -f "PostgreSQL" -nln public.tableName PG:"host=127.0.0.1 user=postgres password=xxxxxx dbname=postgres_dbname"
/home/geostart/file.shp -lco GEOMETRY_NAME=geom  -skipfailures
```

Shapefile to PostGIS promote Multipolygon
```cpp
ogr2ogr -nlt PROMOTE_TO_MULTI  -overwrite -f "PostgreSQL" -nln public.table_name PG:"host=127.0.0.1 user=postgres password=xxxxxx dbname=postgres_dbname" /home/geostart/file.shp -lco GEOMETRY_NAME=geom -skipfailures
```

Oracle to PostGIS with reprojection
```cpp
ogr2ogr -s_srs EPSG:25831 -t_srs EPSG:3857 -overwrite -f "PostgreSQL" -nln public.table_name PG:"host=127.0.0.1 user=postgres password=xxxxxx dbname=postgres_dbname" OCI:"oracle_user/oracle_pass@(DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521)))(CONNECT_DATA = (SID =SID_ORACLE_MACHINE)))" -sql "SELECT A.SHAPE SHAPE, B.FIELD1 FIELD2,DELEGACIO,FIELD7,FIELD4,FIELD3 FROM TABLE1.FIELD4 A, TABLE1.FIELD5 B WHERE A.FIELD3=B.F31_PC AND A.TIPO='U'" -skipfailures
```

Oracle to PostGIS using SQL sentence
```cpp
ogr2ogr  -overwrite -f "PostgreSQL" -nln public.table_name PG:"host=127.0.0.1 user=postgres password=xxxxxx dbname=postgres_dbname" OCI:"oracle_user/oracle_pass@(DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521)))(CONNECT_DATA = (SID =SID_ORACLE_MACHINE)))" -sql "SELECT A.SHAPE SHAPE, B.FIELD1 FIELD2,DELEGACIO,FIELD7,FIELD4,FIELD3, C.FIELD8 FROM TABLE1.FIELD4 A, TABLE1.FIELD5 B, TABLE1.FIELD6 C WHERE A.FIELD3=B.F31_PC AND A.FIELD3 = C.F31_PC AND B.FIELD7=900" -skipfailures
```
Shapefile to Oracle with reprojection
```cpp
ogr2ogr -s_srs EPSG:25831 -t_srs EPSG:25831 -f OCI OCI:"USER/PASSWORD@(DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521)))(CONNECT_DATA = (SID =SID_ORACLE_MACHINE)))" "/home/geostart/file.shp"
```
Oracle to GeoJSON
```cpp
ogr2ogr  -append -f "GeoJSON" /home/geostart/file.geojson OCI:"USER/PASSWORD@(DESCRIPTION = (ADDRESS_LIST = (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521)))(CONNECT_DATA = (SID =SID_ORACLE_MACHINE))):PRG.PCIVIL_PAP" -skipfailures -mapFieldType All
```

Step 1: Convert XLSX to CVS

Step 2: Join CSV and Shapefile to create a GeoPackage
```cpp
ogr2ogr -f CSV /home/geostart/file.csv /home/geostart/file.xlsx
ogr2ogr -f GPKG -sql "select name_shapefile.*, joincsv.* from name_shapefile left join 'joincsv.csv'.joincsv on name_shapefile.fieldID = joincsv.fieldID" /home/geostart/file.gpkg /home/geostart/file.shp -skipFailures
```

### gdal_translate

MBTILES to GeoTiff
```cpp
gdal_translate -oo ZOOM_LEVEL=14  /home/geostart/file.mbtiles  /home/geostart/file.tif
```  
XYZ ASCII file to GeoTiff
```cpp
gdal_translate -of GTiff   /home/geostart/file.xyz  /home/geostart/file.tif
```

TIFF to XYZ
```cpp
gdal_translate  -s_srs EPSG:25831 -a_srs EPSG:3857 -outsize 25% 25% -a_nodata none -co COLUMN_SEPARATOR=; -co ADD_HEADER_LINE=YES -b 1 -of XYZ /home/geostart/file.tif /home/geostart/file_dem.xyz
```

GeoTiff to MBTILES
```cpp
gdal_translate  /home/geostart/file.tif  /home/geostart/file.mbtiles    -co NAME=name_file -co QUALITY=75 -co ZOOM_LEVEL_STRATEGY=UPPER -co TILE_FORMAT=JPEG  -co WRITE_BOUNDS=YES -of MBTILES                             

gdaladdo -r average /home/geostart/file.mbtiles 2 4 8 16
```
```cpp
gdal_translate  /home/geostart/file.tif  /home/geostart/file.mbtiles    -co NAME=name_file -co ZOOM_LEVEL_STRATEGY=UPPER -co TILE_FORMAT=PNG -co RESAMPLING=NEAREST  -co WRITE_BOUNDS=YES -of MBTILES                             

gdaladdo -r average /home/geostart/file.mbtiles 2 4 8 16
```
MBTILES to MBTILES cropping by bounding box and zoom level
```cpp
gdal_translate /home/geostart/file_in.mbtiles /home/geostart/file_out.mbtiles -co TILE_FORMAT=JPEG -co QUALITY=85 -of MBTILES -oo MINX=164729 -oo MINY=5167665 -oo MAXX=201866 -oo MAXY=5185641 -oo ZOOM_LEVEL=17 -co WRITE_BOUNDS=YES
```

### gdaltransform

Vertical transformation orthometric to ellipsoidal XYZ Ascii file
```cpp
gdaltransform -s_srs "+proj=longlat +datum=WGS84 +geoidgrids=/home/gdaldata/egm96_15.gtx +no_defs" -t_srs "+proj=longlat +datum=WGS84 +units=m +no_def" < /home/geostart/file.xyz > /home/geostart/file_elip91.xyz
```  
