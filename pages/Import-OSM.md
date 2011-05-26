# Importing OpenStreetMap data into SpacialDB

To import OpenStreetMap (OSM) data into SpacialDB make sure you have an account and an empty database. Note down the database information as you will need it soon. First though:

## Install osm2pgsql

`osm2pgsql` is a utility program that converts OpenStreetMap (`*.OSM`) data into a format that can be loaded into PostgreSQL. Unlike the `osm2pgrouting` utility, `osm2pgsql`  is a lossy conversion utility: it only adds features that have certain tags, as defined in the config file, and it converts nodes and ways to linestrings and polygons, which means  that you can't tell which linestring is connected to which. `osm2pgsql` is thus ideal for rendering applications as well as applications which require searching, rather than routing.

Installing `osm2pgsql` is easy for the different operating systems, just have a look at the [[Osm2pgsql wiki|http://wiki.openstreetmap.org/wiki/Osm2pgsql]]. In particular for Mac OS X:

```console
$ brew install osm2pgsql
==> Checking out http://svn.openstreetmap.org/applications/utils/export/osm2pgsq
==> ./autogen.sh
==> ./configure
==> make
/usr/local/Cellar/osm2pgsql/HEAD: 6 files, 240K, built in 81 seconds
```

should install it. Note that currently this installs the `default.style` config file into the `/usr/local/share/osm2pgsql/` folder, but you can always get it manually from here: [[default.style|http://svn.openstreetmap.org/applications/utils/export/osm2pgsql/default.style]]

## Download OpenStreetMap data from CloudMade, Geofabrik or Export Tab

The [[CloudMade Downloads|http://downloads.cloudmade.com/]] site contains extracts of OpenStreetMap data from different places around the world. The Just navigate to the continent and then city you are interested in and download the appropriate `*.osm.bz2` file.  For example:

```console
$ wget http://downloads.cloudmade.com/europe/western_europe/germany/berlin/berlin.osm.bz2
```

will download the data from Berlin. An alternative source of data is the [[Geofabrick Download Service|http://www.geofabrik.de/data/download.html]] similar to that of CloudMade.

If you need something local or an area which is not in the CloudMade repository, you can always go to the [[Export Tab of OpenStreetMap|http://www.openstreetmap.org/export]] and set the "Format to Export" to "OpenStreetMap XML Data". Then in the viewer you can go to the area you are interested in and drag a selection box on the map and press the "Export" button. You should receive a `map.osm` file.

You can also do this via the command-line. Below we download data from the Mitte neighbourhood in Berlin:

```console
$ wget -O map.osm http://api.openstreetmap.org/api/0.6/map?bbox=13.39910,52.5285,13.40594,52.5314
```

## Import data into SpacialDB

Once you have the data you will need to run the following command, replacing the SpacialDB database name and username with your own:

```console
$ osm2pgsql berlin.osm.bz2  -H beta.spacialdb.com -P 9999 -d spacialdb0_krasul -U krasul -W -S /usr/local/share/osm2pgsql/default.style --hstore -s

Password:
Using projection SRS 900913 (Spherical Mercator)
Setting up table: planet_osm_point
NOTICE:  table "planet_osm_point" does not exist, skipping
NOTICE:  table "planet_osm_point_tmp" does not exist, skipping
Setting up table: planet_osm_line
NOTICE:  table "planet_osm_line" does not exist, skipping
NOTICE:  table "planet_osm_line_tmp" does not exist, skipping
Setting up table: planet_osm_polygon
NOTICE:  table "planet_osm_polygon" does not exist, skipping
NOTICE:  table "planet_osm_polygon_tmp" does not exist, skipping
Setting up table: planet_osm_roads
NOTICE:  table "planet_osm_roads" does not exist, skipping
NOTICE:  table "planet_osm_roads_tmp" does not exist, skipping
Mid: pgsql, scale=100, cache=800MB, maxblocks=102401*8192
Setting up table: planet_osm_nodes
NOTICE:  table "planet_osm_nodes" does not exist, skipping
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "planet_osm_nodes_pkey" for table "planet_osm_nodes"
Setting up table: planet_osm_ways
NOTICE:  table "planet_osm_ways" does not exist, skipping
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "planet_osm_ways_pkey" for table "planet_osm_ways"
Setting up table: planet_osm_rels
NOTICE:  table "planet_osm_rels" does not exist, skipping
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "planet_osm_rels_pkey" for table "planet_osm_rels"

Reading in file: map.osm
Processing: Node(1k) Way(0k) Relation(0k)
Node stats: total(1681), max(1295161822)
Way stats: total(214), max(107822689)
Relation stats: total(29), max(1566054)

Going over pending ways
processing way (0k)

Going over pending relations

node cache: stored: 1681(100.00%), storage efficiency: 0.66%, hit rate: 100.00%
Stopping table: planet_osm_ways
Stopping table: planet_osm_nodes
Stopping table: planet_osm_rels
Committing transaction for planet_osm_polygon
Building index on table: planet_osm_ways
Building index on table: planet_osm_rels
Committing transaction for planet_osm_point
Sorting data and creating indexes for planet_osm_polygon
Sorting data and creating indexes for planet_osm_point
Committing transaction for planet_osm_roads
Committing transaction for planet_osm_line
Stopped table: planet_osm_ways
Stopped table: planet_osm_rels
Sorting data and creating indexes for planet_osm_roads
Sorting data and creating indexes for planet_osm_line
Stopped table: planet_osm_nodes
Completed planet_osm_polygon
Completed planet_osm_point
Completed planet_osm_line
Completed planet_osm_roads

```

This command will ask for your database password. Note that `osm2pgsql` needs the location of the `default.style` file and you might need to adjust it depending on your setup. In a short while, depending on the size of your data, the command should finish successfully.

## Indexing your hstore column

Since we imported the tags from OpenStreetMap into the `hstore` lets add an index on the column make searching fast.

```console
$ psql -h beta.spacialdb.com -p 9999 -d spacialdb0_krasul -U krasul -c \
"CREATE INDEX idx_planet_osm_point_tags ON planet_osm_point USING gist(tags);
CREATE INDEX idx_planet_osm_polygon_tags ON planet_osm_polygon USING gist(tags);
CREATE INDEX idx_planet_osm_line_tags ON planet_osm_line USING gist(tags);"

Password for user krasul: 
CREATE INDEX
```

Lets test it out by running a query for all Italian restaurants in our data. This time lets connect to SpacialDB via `pgsql`.

```console
$ psql -h beta.spacialdb.com -p 9999 -d spacialdb0_krasul -U krasul
Password for user krasul: 
psql (9.0.3)
Type "help" for help.
```

So to find Italian restaurants we will we will select records that have the `cuisine` tag set to `italian`. However, when we imported the OpenStreetMap data into SpacialDB we didn't create a column for cuisine. So this information would be in our `hstore` making our `WHERE` clause `tags @> 'cuisine=>italian'::hstore`.

```sql
spacialdb0_krasul=> 

SELECT name, ST_AsText(ST_Transform(way,4326)) AS pt_lonlattext 
FROM  planet_osm_point
WHERE tags @> 'cuisine=>italian'::hstore 
LIMIT 5;

        name         |              pt_lonlattext
---------------------+------------------------------------------
 12 Apostel          | POINT(13.3206158981458 52.5052346947542)
 Ristorante Papageno | POINT(13.3067926981477 52.5135816947541)
 il Ritrovo          | POINT(13.4581281981266 52.5095687947541)
 il pane e rose      | POINT(13.4343964981299 52.5294364947537)
 4 Elements          | POINT(13.4513549981276 52.5261730947538)
(5 rows)

```

`hstore` has a unique syntax. The official reference is [[here | http://www.postgresql.org/docs/current/static/hstore.html]].

## Local search by radius

Now lets take advantage of the location and find all restaurants that serve Italian food within 400m of a given location.

```sql
spacialdb0_krasul=>

SELECT name, amenity, ST_AsText(ST_Transform(way,4326)) 
FROM planet_osm_point 
WHERE ST_DWithin(way, ST_Transform('SRID=4326;POINT(13.401832 52.529539)', 900913 ), 400.0)
AND amenity = 'restaurant' AND tags @> 'cuisine=>italian'::hstore;

     name     |  amenity   |                st_astext
--------------+------------+------------------------------------------
 Sisal        | restaurant | POINT(13.4034218981342 52.5278459947538)
 VINO e LIBRI | restaurant | POINT(13.4042590981341 52.5299461947537)
(2 rows)

spacialdb0_krasul=> \q

```

## References:

* [[PostGIS in Action | http://www.manning.com/obe ]]
