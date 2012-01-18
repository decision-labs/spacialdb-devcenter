# Importing SimpleGEO data into SpacialDB

Thanks to SpacialDB' Layers API you can import all your data into SpacialDB, including Polygons and Lines by following this two step process:

## Prerequisites

To get started you will need to Sign up with SpacialDB and have a `ruby` environment setup together with the following gems installed:

```console
$ gem install json typhoeus oauth launchy spacialdb
Building native extensions.  This could take a while...
Successfully installed json-1.6.5
Fetching: typhoeus-0.3.3.gem (100%)
Building native extensions.  This could take a while...
Successfully installed typhoeus-0.3.3
Fetching: oauth-0.4.5.gem (100%)
Successfully installed oauth-0.4.5
Successfully installed launchy-2.0.5
Successfully installed spacialdb-0.0.3
5 gems installed

# Only if you haven't signed up already
$ spacialdb signup
Sign up to Spacialdb.
Email:...
```

## Provision a new SpacialDB Database

Next Log into SpacialDB to create a new database.

```console
$ spacialdb login
nter your Spacialdb credentials.
Email or Username: krasul
Password:
Logged in successfully.

$ spacialdb create
{"name":"aetskhmm_krasul","port":9999,"username":"aetskhmm_krasul","host":"beta.spacialdb.com","password":"blah"}
```

## Get the import script

```console
$ wget https://raw.github.com/gist/1633494/migrate-simplegeo-to-spacialdb.rb
--2012-01-18 17:01:51--  https://raw.github.com/gist/1633494/migrate-simplegeo-to-spacialdb.rb
Resolving raw.github.com... 207.97.227.243
Connecting to raw.github.com|207.97.227.243|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6058 (5.9K) [text/plain]
Saving to: `migrate-simplegeo-to-spacialdb.rb'

100%[================================>] 6,058       --.-K/s   in 0s      

2012-01-18 17:01:52 (1.41 GB/s) - `migrate-simplegeo-to-spacialdb.rb' saved [6058/6058]
```

## Add SimpleGEO and SpacialDB credentials

Open the file and add your SimpleGEO credentials:

```console
$ head -14 migrate-simplegeo-to-spacialdb.rb
# ================================================================================
# Modify to add you SimpleGeo credentials:
# Adapt the following to your SimpleGeo credentials. Get it from the Simplegeo UI.
# ================================================================================
simplegeo_oauth_key="[token]"
simplegeo_oauth_secret="[secret]"

# =========================================================================
# Modify to add you SpacialDB credentials:
# Get from SpacialDB dashboard or see ~/.spacialdb/credentials after signup
# =========================================================================
spacialdb_login='[username@example.com]'
spacialdb_api_key='[462772f480f7c0fda28ef4d93610d4f7]'
database_name='[kgfpyqkw_username]'
```

For the SpacialDB API login and key check you `~/.spacialdb/credentials` file and the `database_name` should be the SpacialDB database we just provisioned. If you would like to see all your databases just run:

```console
$ spacialdb list
{"username"=>"aetskhmm_krasul", "password"=>"blahblah", "host"=>"beta.spacialdb.com", "name"=>"aetskhmm_krasul", "port"=>9999, "connection_string"=>"postgres://aetskhmm_krasul:a20e8a6b@beta.spacialdb.com:9999/aetskhmm_krasul"}
```

## Run the script!

Now we can run the script and it will go ahead and import all your SimpleGEO Layers into SpacialDB Layers and import the appropriate features into each layer. The SimpleGEO `id` will be renamed to `simplegeo_id` for each feature.

```console
$ ruby migrate-simplegeo-to-spacialdb.rb
...
================================================================================
:: Your new SpacialDB layer is ready to use ::

{:name=>"io_path_testlayer",
 :srid=>4326,
 :database=>"aetskhmm_krasul",
 :table_name=>"io_path_testlayer",
 :user=>"krasul",
 :api_url=>"https://api.spacialdb.com/1/users/krasul/layers/io_path_testlayer",
 :acl=>
  {:get=>"3fa5127715e3112504f69a16b36f18ac",
   :delete=>"blahblah",
   :post=>"blahblah",
   :put=>"blahblah"}}

Layer "io.path.testlayer" import complete! View and edit your layer here:
https://api.spacialdb.com/1/users/krasul/layers/io_path_testlayer?key=3fa5127715e3112504f69a16b36f18ac

================================================================================
```

This should also open up your browser for each imported layer and it should look something like:

```json
{
   "type":"FeatureCollection",
   "features":[
      {
         "type":"Feature",
         "geometry":{
            "type":"Point",
            "coordinates":[
               -122.42608,
               37.75965
            ]
         },
         "properties":{
            "cc2":"",
            "name":"Mission Dolores Park",
            "type":"place",
            "layer":"com.simplegeo.global.geonames",
            "admin1":"CA",
            "admin2":"075",
            "admin3":"",
            "admin4":"",
            "gtopo30":"60",
            "timezone":"America/Los_Angeles",
            "asciiname":"Mission Dolores Park",
            "elevation":"22",
            "population":"0",
            "country_code":"US",
            "feature_code":"PRK",
            "simplegeo_id":"5373629",
            "feature_class":"L",
            "last_modified":"2006-01-15",
            "alternatenames":""
         },
         "id":1
      },
      {
         "type":"Feature",
         "geometry":{
            "type":"Point",
            "coordinates":[
               -122.42553,
               37.75937
            ]
         },
         "properties":{
            "cc2":"",
            "name":"Golden Gate Lutheran Church",
            "type":"place",
            "layer":"com.simplegeo.global.geonames",
            "admin1":"CA",
            "admin2":"075",
            "admin3":"",
            "admin4":"",
            "gtopo30":"60",
            "timezone":"America/Los_Angeles",
            "asciiname":"Golden Gate Lutheran Church",
            "elevation":"23",
            "population":"0",
            "country_code":"US",
            "feature_code":"CH",
            "simplegeo_id":"5352852",
            "feature_class":"S",
            "last_modified":"2006-01-15",
            "alternatenames":""
         },
         "id":2
      }
   ]
}
```