# Building navigation applications using OpenStreetMap data

## Out of the box routing

When you provision a new SpacialDB instance you get [[PgRouting|http://www.pgrouting.org/]] installed and ready to go. In this tutorial we will be using data from OpenStreetMap but you can use any data you like. 

By the end of this you will have a routing demo done. Check out: [[http://spacialdb-routing.heroku.com]]. If you plan to deploy then before we start you need to know how to use it with [[Heroku]].

## Getting the road network data

We first want to download some OSM format data which we will convert into a network for routing. You can get this data in two ways:

### Using the openstreetmap.org website:

You simply pan to the region of interest and click the **Export** tab. Here select the export format to be **OpenStreetMap XML Data** and **Export**. This will start the download of the OpenStreetMap data, most probably saved as `map.osm`.

### Using the openstreetmap.org API:

The OpenStreetMap data API call is as follows:

    http://api.openstreetmap.org/api/0.6/map?bbox={x-lon-min},{y-lat-min},{x-lon-max},{y-lat-max} 

So lets download some data around Berlin by using `wget` (or `curl`)

    $ wget -O map.osm http://api.openstreetmap.org/api/0.6/map?bbox=13.415677,52.517816,13.420215,52.520088


## Importing road network data for routing

Assuming you have created a database already you will need a utility called `osm2pgrouting`. This command-line tool can take data in OpenStreetMap XML format and load it into a SpacialDB instance. Get it here: [[https://github.com/kashif/osm2pgrouting]] As of writing this the tool must be obtained from Kashif's github fork and requires PostGIS, Boost, and CMake to be installed on your system.

Once installed, simply run `osm2pgrouting` on the downloaded data. You can get the information about your database by running `spacialdb list` on the command-line. The file named `mapconfig.xml` is shipped with `osm2pgrouting`. It contains tag names that will be imported into the routing database. Learn  more about `osm2pgrouting` from this great [[workshop|http://workshop.pgrouting.org/]].

Use you database connection parameters to run:

```console
# SPACIALDB_USER=username
# SPACIALDB_DB=dbname
# SPACIALDB_PASS=passwd
# SPACIALDB_HOST=beta.spacialdb.com
# SPACIALDB_PORT=9999

$ osm2pgrouting -file map.osm \
  -conf /path/to/osm2pgrouting/mapconfig.xml \
  -dbname $SPACIALDB_DB \
  -user $SPACIALDB_USER \
  -host $SPACIALDB_HOST \
  -port $SPACIALDB_PORT \
  -passwd $SPACIALDB_PASS -clean
```

After this you should be able to view this data in [[QGIS|Instant Map]]. Connect to your database using the PostGIS connection, add the `ways` and `vertices_tmp` tables, and you should see something like:

![German Cities](/img/ways-and-vertices.png)

Lets do some routing. We now have access to quite a few routing functions. lets try `shortest_path` between two `ways` with say: `source_id=1` and `target_id=240`:

```sql
SELECT * FROM shortest_path('
  SELECT gid as id,
       source::integer,
       target::integer,
       length::double precision as cost
      FROM ways',
  1, 240, false, false);
```

This gives us the vertices of the route. If you want to visualise this, install the QGIS plugin called `RT SQL Layer` and call `dijkstra_sp` with the start and end `id` of the result from the previous query:

```sql
select * from dijkstra_sp('ways', 125, 271)
```

![German Cities](/img/qgis-route.png)

## Writing the Rails Application and deploying it on Heroku

Lets write an application in Ruby on Rails that makes use of SpacialDB as its routing engine. We will deploy this application to Heroku. Before starting you need to have created a new rails application via Heroku and have added a SpacialDB addon by running:

```console
$ heroku addons:add spacialdb:test
```

Form more details on Heroku check out the [[Heroku]] addon guide. 

You can get the whole application at [Routing Heroku Example](https://github.com/spacialdb/Routing-Heroku-Example) on SpacialDB's github account.

1. We start by creating an Openlayers client that renders OpenStreetMap tile as well as creates a form to allow selecting the routing tools. [openlayers client](https://github.com/spacialdb/Routing-Heroku-Example/commit/e0ccdb6312354d581815186429e9f13a5f068bba#diff-1). We also setup our [routes](https://github.com/spacialdb/Routing-Heroku-Example/commit/e0ccdb6312354d581815186429e9f13a5f068bba#diff-2) and a [map_controller](https://github.com/spacialdb/Routing-Heroku-Example/commit/e0ccdb6312354d581815186429e9f13a5f068bba#diff-0). 

2. Now lets setup an initializer to setup the [SpacialDB connection configuration](https://github.com/spacialdb/Routing-Heroku-Example/commit/9775d51e2158a3c9b9f1f84bd9929375313969e9#diff-4) as well as a [base class](https://github.com/spacialdb/Routing-Heroku-Example/commit/9775d51e2158a3c9b9f1f84bd9929375313969e9#diff-3) to connection to SpacialDB. 

3. Next we add the `route` method to the [controller](https://github.com/spacialdb/Routing-Heroku-Example/commit/46bce13d2f2f86d585db0bd4aa55a883e5a55a9e#diff-2) and process the start and end points to pass them to the [way model](https://github.com/spacialdb/Routing-Heroku-Example/commit/46bce13d2f2f86d585db0bd4aa55a883e5a55a9e#diff-3) to run the routing method. And thats it, you can go ahead and deploy to Heroku.

Check out: [[http://spacialdb-routing.heroku.com/]]

