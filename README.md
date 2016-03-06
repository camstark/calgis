#GeoJSON versions of City of Calgary GIS data
Using $ ogr2ogr -f GeoJSON -t_srs crs:84 [name].geojson [name].shp

Simplifying using MapShaper: https://github.com/mbloch/mapshaper/wiki, e.g.:

$ mapshaper [name].geojson snap -simplify .05 -o [newname].geojson


Convert to TopoJSON using topojson: https://github.com/mbostock/topojson/wiki

$ topojson -o [name].json [name].json
