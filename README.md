Prerequisites
-------------
* A recent Geoserver
* A recent PostgreSQL (I work with 9.1 and so should you)
* A recent PostGIS (This doc is done with 2.0 but should translate to 1.5 without too much trouble)

Steps to get your Staleness page up
-----------------------------------
### Preps
1. Download data for an area, for example from http://metro.teczno.com/. I've taken Amsterdam from there in pbf format: `amsterdam.osm.pbf`. Park it someplace convenient.
1. Create a PostGIS database 
1. Read the data into your fresh database:

  `osmosis --rb /osm/planet/amsterdam.osm.pbf --wp database=osm user=osm password=osm` (replace db name and credentials with your own)

## Generate age files

Now sneak some shapefiles out of there that we will use to feed the WMS:

`$  /usr/lib/postgresql/9.1/bin/pgsql2shp -u osm -f amsterdam_nodes.shp osm "SELECT geom, floor(extract(epoch from age(tstamp))/60/60/24) AS age FROM nodes WHERE akeys(tags) <> '{}'"
$  /usr/lib/postgresql/9.1/bin/pgsql2shp -u osm -f amsterdam_ways.shp osm "SELECT linestring, floor(extract(epoch from age(tstamp))/60/60/24) AS age FROM ways WHERE NOT tags?'building'"`

(Yes, you could have Geoserver talk directly to the database too. Go ahead and set it up that way, smartass.)

## Geoserver setup

1. Styles -> Add New Style -> Browse... -> select `osm_age_nodes.sld` -> OK -> Upload... -> Submit
1. Repeat for `osm_age_ways.sld`
1. Stores -> Add New Store -> Directory of spatial files (shapefiles)

        <img src="screenshots/GeoServer New Vector Data Source - Mozilla Firefox_2012-07-03_17-45-31.png" type="image/png" height="597" width="448"/>

1. Save.
1. Layers -> Add New Layer... -> Select `staleness_amsterdam` 

<img src="screenshots/GeoServer New Layer - Mozilla Firefox_2012-07-03_17-50-02.png" type="image/png" height="324" width="748"/>

For both layers:

1. Click 'Publish' 
1. Set 'EPSG:3785' as 'Declared CRS'
1. Set 'SRS Handling' to 'Reproject native to declared'.
1. Fill the Bounding Box extents by clicking 'Compute from data' and 'Compute from native bounds' respectively. This should give something like this:

<img src="screenshots/GeoServer Edit Layer - Mozilla Firefox_2012-07-03_17-50-35.png" type="image/png" height="403" width="481"/>

1. Note the Native Bounding Box extents, we'll need them later in the OpenLayers config.
1. Now click the 'Publishing' tab near the top.
1. Choose the `osm_age_nodes` / `osm_age_ways` style as the default style for the WMS service:

<img src="screenshots/GeoServer Edit Layer - Mozilla Firefox_2012-07-03_17-51-00.png" type="image/png" height="225" width="345"/>

1. Save. When you published both layers you're done with GeoServer config.

## OpenLayers setup

Open index.html in a text editor. Three things to edit here.

1. Look for the nodes and ways OpenLayers layer configurations, specifically the line
        `LAYERS: 'schaaltreinen:dc_nodes'`
	and the corresponding ways layer config.
	Change to your namespace:layername. The namespace is the Geoserver data store name. Following the example the names would be 'schaaltreinen:amsterdam_nodes' and 'schaaltreinen:amsterdam_ways'
1.  Look for the OpenLayers map extent definition:
	`var bounds = new OpenLayers.Bounds(......`
	and replace the values with the xmin,ymin,xmax,ymax from the layer config in Geoserver (doesn't really matter too much which one).
1. Look for
	`var metroname = "Amsterdam";`
	and change to your region name.
Save.
Load `index.html` in your web browser and enjoy.
