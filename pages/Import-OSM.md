# Importing OSM data into SpacialDB

To import OpenStreetMap data into SpacialDB make sure you have an account and an empty database. Note down the database information as you will need it soon. First though:

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

## Import data into SpacialDB

Once you have the data you will need to run the following command, replacing the SpacialDB database name and username with your own:

```console
$ osm2pgsql berlin.osm.bz2  -H beta.spacialdb.com -P 9999 -d spacialdb0_krasul -U krasul -W -S /usr/local/share/osm2pgsql/default.style --hstore -s
```

This command will ask for your database password. Note that `osm2pgsql` needs the location of the `default.style` file and you might need to adjust it depending on your setup. In a short while, depending on the size of your data, the command should finish successfully.

