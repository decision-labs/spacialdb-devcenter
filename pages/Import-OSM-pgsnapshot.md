# Importing OpenStreetMap data into SpacialDB (pgsnapshot)

The pg\_snapshot database schema is one of the osm database schemes. It's useful if you want to work on the tags or do spatial requests on nodes or ways. It's not made for tile rendering.
To import OpenStreetMap (OSM) data into SpacialDB make sure you have an account and an empty database. Note down the database information as you will need it soon. First though:

## Install osmosis

Installing `osmosis` should be easy. For instructions just take a look at the [[Osmosis wiki page|http://wiki.openstreetmap.org/wiki/osmosis]].

## Download OpenStreetMap data from CloudMade, Geofabrik or Export Tab

The [[CloudMade Downloads|http://downloads.cloudmade.com/]] site contains extracts of OpenStreetMap data from different places around the world. Just navigate to the continent and then city you are interested in and download the appropriate `*.osm.bz2` or `*.osm.pbf` file.  For example:

```console
$ wget http://downloads.cloudmade.com/europe/western_europe/germany/berlin/berlin.osm.bz2
```

will download the data from Berlin. An alternative source of data is the [[Geofabrick Download Service|http://www.geofabrik.de/data/download.html]] similar to that of CloudMade.

If you need something local or an area which is not in the CloudMade repository, you can always go to the [[Export Tab of OpenStreetMap|http://www.openstreetmap.org/export]] and set the "Format to Export" to "OpenStreetMap XML Data". Then in the viewer you can go to the area you are interested in and drag a selection box on the map and press the "Export" button. You should receive a `map.osm` file.

You can also do this via the command-line. Below we download data from the Mitte neighbourhood in Berlin:

```console
$ wget -O map.osm http://api.openstreetmap.org/api/0.6/map?bbox=13.39910,52.5285,13.40594,52.5314
```

## Create schemas

```console
cd osmosis-0.39/script
psql -H beta.spacialdb.com -P 9999 -d spacaildb0_krasul -U krasul -f pgsnapshot_schema_0.6.sql
psql -H beta.spacialdb.com -P 9999 -d spacaildb0_krasul -U krasul -f pgsnapshot_schema_0.6_linestring.sql
```

## Import data into SpacialDB

Once you have the data you will need to run the following command, replacing the SpacialDB database name and username with your own:

```console
cd osmosis-0.39/bin
./osmosis --read-xml file="pathToTheOsmXmlFile" --write-pgsql host=beta.spacialdb.com:9999 user=spacialdbUserName database=yourSpacialdbDatabase password=yourSpacialdbPassword
# if you want to use pbf files (faster)
./osmosis --read-pbf file="pathToTheOsmXmlFile" --write-pgsql host=beta.spacialdb.com:9999 user=spacialdbUserName database=yourSpacialdbDatabase password=yourSpacialdbPassword
```

Lets test it out by running a query for all Italian restaurants in our data. This time lets connect to SpacialDB via `pgsql`.

```console
$ psql -h beta.spacialdb.com -p 9999 -d spacialdb0_krasul -U krasul
Password for user krasul:
psql (9.0.3)
Type "help" for help.
```

So to find Italian restaurants we will we will select records that have the `cuisine` tag set to `italian`. This information would be in our `hstore` making our `WHERE` clause `tags @> hstore('cuisine','italian')`.

```sql
spacialdb0_krasul=>

SELECT tags, ST_AsText(ST_Transform(nodes,4326)) AS pt_lonlattext
FROM nodes
WHERE tags @> hstore('cuisine','italian')
LIMIT 5;
```

`hstore` has a unique syntax. The official reference is [[here | http://www.postgresql.org/docs/current/static/hstore.html]].

## Local search by radius

Now lets take advantage of the location and find all restaurants that serve Italian food within 400m of a given location.

```sql
spacialdb0_krasul=>

SELECT tags, ST_AsText(ST_Transform(geom,4326))
FROM nodes
WHERE ST_DWithin(geom,'SRID=4326;POINT(13.401832 52.529539)'), 400.0)
AND tags @> hstore('amenity','restaurant') AND (tags @> hstore('cuisine','italian');


spacialdb0_krasul=> \q

```

## References:

* [[PostGIS in Action | http://www.manning.com/obe ]]
