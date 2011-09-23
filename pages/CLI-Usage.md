Running `spacialdb help` displays the following usage summary:

```console
$ spacialdb help
Usage: spacialdb COMMAND [command-specific-options]

Primary help topics, type "spacialdb help TOPIC" for more details:

  auth    # authentication (signup, login, logout)
  db      # manage databases (list, create, destroy)
  layers  # manage layers (list, create, destroy)

Additional topics:

  help     # list commands and display help
  version  # version
```
<div name='authentication'></div>
## Authentication Commands

The `signup` command simply signs you up to a free plan:

```console
$ spacialdb signup
Sign up to Spacialdb.
Email: shoaib@spacialdb.com
Username: shoaib
Password: 
Password confirmation: 
Signed up successfully.
```

The `login` command will log you in with you credentials and place your private API key in `~/.spacialdb/credentials`.

```console
$ spacialdb login
Enter your Spacialdb credentials.
Email or Username: shoaib
Password: 
Logged in successfully.
$ cat ~/.spacialdb/credentials
shoaib@spacialdb.com
92b2d0974304202d68827055cc04344d
```

The `logout` command will remove your credential file.

<div name='database_management'></div>
## Database Management Commands

<div name='database_create'></div>
### Create a geospatial database

To create a geospatial database simply run:

```console
$ spacialdb db:create
{
  "name": "spacialdb1_shoaib",
  "port": 9999,
  "username": "shoaib",
  "host": "beta.spacialdb.com",
  "password": "1589b20eca"
}

```

which will return the connection string to a new geospatial database.

<div name='database_list'></div>
### List geospatial databases

The `db` command then prints out the connection settings for all your databases:

```console
$ spacialdb db                               
[
  {
    "username": "shoaib",
    "password": "1589b20eca",
    "host": "beta.spacialdb.com",
    "name": "spacialdb0_shoaib",
    "connection_string": "postgres://shoaib:1589b20eca@beta.spacialdb.com:9999/spacialdb0_shoaib",
    "port": 9999
  }
]
```

<div name='database_delete'></div>
### Delete geospatial databases

Finally you can delete a database by calling:

```console
$ spacialdb db:destroy --db spacialdb1_shoaib

 !    WARNING: Potentially Destructive Action
 !    This command will affect the db: spacialdb1_shoaib
 !    To proceed, type "spacialdb1_shoaib" or re-run this command with --confirm spacialdb1_shoaib

> spacialdb1_shoaib
Destroying spacialdb1_shoaib ... done.
```
<div name='api_layer_management'></div>
## API Layer Management Commands

If you plan to use the SpacialDB API then you will need to create a `Layer` which will also provide you keys for the different `CRUD` operations on the layer.

<div name='api_layer_create'></div>
### Creating a layer

To create a layer called `parks` on a particular database you need to:

```console
$ spacialdb layers:add parks --db spacialdb0_shoaib                     
{
  "name": "parks",
  "srid": 4326,
  "database": "spacialdb0_shoaib",
  "table_name": "parks",
  "user": "shoaib",
  "api_url": "https://api.spacialdb.com/1/users/shoaib/layers/parks",
  "acl": {
    "get": "e158ff36d16c9ece0e49359b4903dc2c",
    "delete": "81f6c272ac1662f8921a41d2cb2d3995",
    "post": "24ccc5c742416dad2fb170b21ee66633",
    "put": "53e372377f0ed1338af39aeadb3d3728"
  }
}

```

This creates a layer that's accessible via an API and access tokens. So in the above example you can try posting to your layer using curl:

```console
$ curl -X POST -d \
  'properties={"name":"A Park in Berlin", "location":"Berlin, Germany"}&\
   geometry={"type":"Polygon", "coordinates":\
    [[ [13.484,52.483],[13.483,52.467],[13.496, 52.464],[13.499,52.480],[13.484,52.483] ]]}' \
   https://api.spacialdb.com/1/users/shoaib/layers/parks\?key\=24ccc5c742416dad2fb170b21ee66633
{"id":1}
```

By default the layer name will also be the name of a corrsponding SpacialDB table. You can override this and specify your own `table_name` of a layer in your database with the `--tablename` option. You can also override the `srid` with the `--srid` option.

For more details on the Layers API see the [[SpacialDB API docs|api_v1]] and [[SpacialDB API Recipes|api-v1-recipes]].

<div name='api_layer_info'></div>
### Layer info

To get information about a particular layer simply type:

```console
$ spacialdb layers:info parks
{
  "user": "shoaib",
  "acl": {
    "get": "e158ff36d16c9ece0e49359b4903dc2c",
    "delete": "81f6c272ac1662f8921a41d2cb2d3995",
    "post": "24ccc5c742416dad2fb170b21ee66633",
    "put": "53e372377f0ed1338af39aeadb3d3728"
  },
  "api_url": "https://api.spacialdb.com/1/users/shoaib/layers/parks",
  "srid": 4326,
  "table_name": "parks",
  "database": "spacialdb0_shoaib",
  "name": "parks"
}
```

Finally to remove a layer from the API, type:

```console
$ spacialdb layers:remove parks
Removing layer parks ... done.
```