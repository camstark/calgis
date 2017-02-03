## TopoJSON versions of City of Calgary GIS data

Various City of Calgary layers (including communities and wards) are available at http://data.calgary.ca

### Version 2.0

Following Mike Bostock's [Command-Line Cartography](https://medium.com/@mbostock/command-line-cartography-part-1-897aa8f8ca2c#.4ei0vh2ib) guide.

##### Getting the data and taking a look

Find the data you want in the catalogue and download using curl. Here's the call for the 2016 Census by Community dataset:
```
curl 'https://data.calgary.ca/api/geospatial/cje4-zd6c?method=export&format=GeoJSON' -o census-by-community-2016.geojson
```


Make sure you have d3-geo-projection installed via `npm install -g d3-geo-projection` and have a quick look at the default polygons as svg:
```
geoproject 'd3.geoAlbers().fitSize([500,500], d)' < census-by-community-2016.geojson | geo2svg -w 500 -h 500 > census-by-community-2016.svg
```
![community boundaries](https://camstark.github.io/calgis/census-by-community-2016.svg "Calgary Community Boundaries")


##### Simplifying boundaries

Calgary has lots of intricate and detailed community boundaries, but we're building a high-level thematic map, not a wayfinding tool. Capturing and presenting every jog can be distracting to the viewer, as well as slower to load in the browser. Command-Line Cartography suggests using the simplification tools in toposimplify, but here we use [Mapshaper](https://github.com/mbloch/mapshaper/wiki): `npm install -g mapshaper`.

Taking advantage of mapshaper's GeoJSON manipulation capbilities, create new uppercase attributes for NAME, COMM_CODE and CLASS to align with past data downloaded from the previous Open Data Catalogue.

In the same command, use the default weighted Visvalingam simplification retaining 4.2% of removable points and output a new simplified GeoJSON file.

```
mapshaper census-by-community-2016.geojson -each 'this.properties.NAME=this.properties.name, this.properties.COMM_CODE=this.properties.comm_code, this.properties.CLASS=this.properties.class' -simplify 4.2% keep-shapes -o community-2016-simple.json
```

The percentage option in the simplification was initially chosen interactively by loading the GeoJSON in to the [Mapshaper Web Interface](http://mapshaper.org/) to find a cut-off that simplifies as much as possible while retaining recognizable community shapes.

Take a look at the output to ensure the simplification is not too much or too little:
```
geoproject 'd3.geoAlbers().fitSize([500,500], d)' < community-2016-simple.json | geo2svg -w 500 -h 500 > community-2016-simple.svg
```

![community boundaries](https://camstark.github.io/calgis/community-2016-simple.svg "Simplified Calgary Community Boundaries")

##### Convert to TopoJSON for Web Maps

Make sure you have topojson installed via `npm install -g topojson` and convert your simplified file from GeoJSON to TopoJSON:

```
geo2topo collection=community-2016-simple.json > community-2016-simple-topo.json
```

To efficiently load for display in the browser, the original GeoJSON went from 3.1 MB to 870 KB through simplification. Then to 803 KB by conversion to TopoJSON. If we quantize and delta encode, its final size is 718 KB.

```
topoquantize 1e5 community-2016-simple-topo.json > community-2016-quantized-topo.json
```

There's a simple d3 map running off of this file at http://camstark.github.io/calgis/

### Version 1.0

Using $ ogr2ogr -f GeoJSON -t_srs crs:84 [name].geojson [name].shp

Simplifying using MapShaper: https://github.com/mbloch/mapshaper/wiki, e.g.:

$ mapshaper [name].geojson snap -simplify .05 -o [newname].geojson


Convert to TopoJSON using topojson: https://github.com/mbostock/topojson/wiki

$ topojson -o [name].json [name].geojson -p
