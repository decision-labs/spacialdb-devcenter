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
Email: kashif@spacialdb.com
Username: kashif
Password: 
Password confirmation: 
Signed up successfully.
```

The `login` command will log you in with you credentials and place your private API key in `~/.spacialdb/credentials`.

```console
$ spacialdb login
Enter your Spacialdb credentials.
Email or Username: krasul
Password: 
Logged in successfully.
$ cat ~/.spacialdb/credentials
kashif@spacialdb.com
92b2d0974304202d68827055cc04344d
```

The `logout` command will remove your credential file.

<div name='database_management'></div>
## Database Management Commands

To create a geospatial database simply run:

```console
$ spacialdb db:create
{"name":"spacialdb0_krasul","port":9999,"username":"krasul","host":"beta.spacialdb.com","password":"mypasswd"}
```

which will return the connection string to a new geospatial database.

The `db` command then prints out the connection settings for all your databases:

```console
$ spacialdb db
{"username"=>"krasul", "user_id"=>4, "password"=>"mypasswd", "host"=>"beta.spacialdb.com", "updated_at"=>"2011-08-26 12:38:49 +0000", "created_at"=>"2011-08-26 12:38:49 +0000", "name"=>"spacialdb2_krasul", "port"=>9999, "id"=>99}
{"username"=>"krasul", "user_id"=>4, "password"=>"mypasswd", "host"=>"beta.spacialdb.com", "updated_at"=>"2011-05-25 14:37:41 +0000", "created_at"=>"2011-05-25 14:37:41 +0000", "name"=>"spacialdb0_krasul", "port"=>9999, "id"=>23}
{"username"=>"krasul", "user_id"=>4, "password"=>"mypasswd", "host"=>"beta.spacialdb.com", "updated_at"=>"2011-07-04 09:20:24 +0000", "created_at"=>"2011-07-04 09:20:24 +0000", "name"=>"spacialdb1_krasul", "port"=>9999, "id"=>71}
```

Finally you can delete a database by calling:

```console
$ spacialdb db:destroy --db spacialdb2_krasul

 !    WARNING: Potentially Destructive Action
 !    This command will affect the db: spacialdb2_krasul
 !    To proceed, type "spacialdb2_krasul" or re-run this command with --confirm spacialdb2_krasul

> spacialdb2_krasul
Destroying spacialdb2_krasul ... done
```
<div name='api_layer_management'></div>
## API Layer Management Commands

If you plan to use the SpacialDB API then you will need to create a Layer which will also provide you keys for the different CRUD operations on the layer.

To create a layer called `parks` on a particular database you need to:

```console
$ spacialdb layers:add parks --db spacialdb0_krasul
{"name":"parks","srid":4326,"id":19,"user_id":4,"database":"spacialdb0_krasul","table_name":"parks","acl":{"get":"32166ddfd2738bd9f6ce2a3ca221d926","delete":"1c960a75518cb1d5d8bddcdb8d2a8613","post":"35b419a11dae739eaf2593e589fe140a","put":"b529d93463b67e4551e82ea314e4b1e4"}}

```

You can also give specify the `table_name` of this layer in your database with the `--tablename` option and also the `srid` with the `--srid` option.

To get information about a particular layer simply type:

```console
$ spacialdb layers:info parks
{"user_id"=>4, "srid"=>4326, "table_name"=>"parks", "database"=>"spacialdb0_krasul", "name"=>"parks", "id"=>19, "acl"=>{"get"=>"32166ddfd2738bd9f6ce2a3ca221d926", "delete"=>"1c960a75518cb1d5d8bddcdb8d2a8613", "post"=>"35b419a11dae739eaf2593e589fe140a", "put"=>"b529d93463b67e4551e82ea314e4b1e4"}}
```

Finally to remove a layer from the API, type:

```console
$ spacialdb layers:remove parks

```