# SpacialDB on Heroku

## What is SpacialDB?

SpacialDB is a cloud based Geospatial database.


# SpacialDB Cloud Database

[SpacialDB][1] is a Geospatial database service that allows you to create, operate and scale dedicated Geospatial databases in the cloud. Your SpacialDB databases can be used transparently in place of the Heroku-provided PostgreSQL database.

   [1]: https://www.spacialdb.com (SpacialDB)


## Deploying to Heroku

To use SpacialDB on Heroku, make sure that you have a database provisioned with SpacialDB and you have signed in to Heroku via their toolbelt suite of applications:

```console
$ heroku login
Enter your Heroku credentials.
Email: kashif.rasul@gmail.com
Password (typing will be hidden):
Authentication successful.
...
```

In order to use the SpacialDB provided database, you simply need to add the appropriate `DATABASE_URL` setting to your Heroku app. This can be achieved by:

```console
$ cd my-heroku-app
$ heroku config:add DATABASE_URL=....
```

The `DATABASE_URL` string will depend on the actual driver you are using as well as the framework your application uses.

## Connecting to SpacialDB via Ruby on Rails (a tutorial)

In order to deploy a Rails app to Heroku which uses Postgis for its database via SpacialDB, we will need to use the [RGeo](https://github.com/dazuma/rgeo) set of gems. In this example we will extend this tutorial: [Building an iOS Photo-sharing and Geolocation Mobile Client and API](https://devcenter.heroku.com/articles/ios-photo-sharing-geo-location-service) by using a SpacialDB provisioned database with Postgis.



### Rails 3.2 app (the server)

We create a new app with:

```console
$ rails new geo-photo --database=postgresql
...
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
$ cd geo-photo
```

To begin with we need to add a Heroku specific configuration in `config/application.rb` and then we can check it in and deploy it:

```console
$ cat config/application.rb
...
    # Heroku requires this to be false
    config.assets.initialize_on_precompile = false
...
$ git init
...
$  git add .
$  git commit -m "Initial commit"
...
```

Now we are ready to deploy our initial empty app by first creating it and then pushing it:

```console
$ heroku create
Creating young-reef-5849... done, stack is cedar
http://young-reef-5849.herokuapp.com/ | git@heroku.com:young-reef-5849.git
Git remote heroku added
$ git push heroku master
...
-----> Ruby/Rails app detected
...

To app should now be up and running and you can go to it by typing:

```console
$ heroku open
```

### Add the RGeo gem

Now we are ready to add the RGeo gem in order to use the Postgis adapter with Rails. RGeo requires [Geos](http://geos.osgeo.org/) and [Proj](http://trac.osgeo.org/proj/) to be installed on the system we are deploying on.

So one solution is to install these precompiled  dependencies with the `vendorbinaries` buildpack together with the default Heroku Ruby buildpack. In order to use these two buildpacks, we will use the `buildpack-multi`. Have a look at this blog post: [Building and Deploying Custom Binaries on Heroku](http://spin.atomicobject.com/2012/12/09/building-and-deploying-custom-binaries-on-heroku/) for more information.

Lastly, the `vendorbinaries` buildpack requires a url to download the binaries from, and we have provided the latest Proj (4.8.0) and Geos (3.3.8) binaries compiled on the [Heroku platform](https://devcenter.heroku.com/articles/buildpack-binaries) via a public Amazon S3 bucket.

So lets get started, first the `buildpack-multi`, we configure heroku to use it and add the buildpacks we use in the `.buildpacks` file:

```console
$ heroku config:set BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
...
$ cat .buildpacks
https://github.com/peterkeen/heroku-buildpack-vendorbinaries.git
https://github.com/heroku/heroku-buildpack-ruby.git
```

Now we tell the `vendorbinaries` buildpack the urls to use in the `.vendor_urls` file:

```console
$ cat .vendor_urls
https://s3.amazonaws.com/spacialdb/heroku/geos-3.3.8.tar.gz
https://s3.amazonaws.com/spacialdb/heroku/proj-4.8.0.tar.gz
```

Now during the deploy process, this buildpack will download and untar the binaries into the root of our app, in this case into folders: `bin/`, `lib/` and `include/`. So lets check this in and deploy it:

```console
$ git add .buildpacks .vendor_urls
$ git commit -m "buildpacks"
$ git push heroku master
...
-----> Found a .vendor_urls file
       Vendoring https://s3.amazonaws.com/spacialdb/heroku/geos-3.3.8.tar.gz
       Vendoring https://s3.amazonaws.com/spacialdb/heroku/proj-4.8.0.tar.gz
...
```

Now we are ready to add the RGeo gem. However there is one complication, the libraries are extracted in a non-default location. So we need to let the `gem` command know the exact location, which in our case is the root of our app: `/app`.  Normally we would do `gem install rgeo -- --with-geos-dir=/app --with-proj-dir=/app` but thats not possible on Heroku. So we use `bundle` to pass the specify flags to this one gem namerly: Rgeo. We can add the following config to our app which will let the `bundle` command know the options when installing the Rgeo gem:

```console
$ heroku config:set BUNDLE_BUILD__RGEO="--with-geos-dir=/app --with-proj-dir=/app"
Setting config vars and restarting young-reef-5849... done, v9
BUNDLE_BUILD__RGEO: --with-geos-dir=/app --with-proj-dir=/app
```

Once this is done, we can add the gems in the `Gemfile`, the Postgis adapter `railtie` in `config/application.rb`, commit and deploy it:

```console
$ cat Gemfile
...
gem 'rgeo'
gem 'activerecord-postgis-adapter'
...
$ cat config/application.rb
...
require 'rails/all'
require 'active_record/connection_adapters/postgis_adapter/railtie'
...
$ bundle install
...
$ git add ...
$ git commit ...
$ git push heroku master
....
       Installing rgeo (0.3.20)
       Installing rgeo-activerecord (0.5.0)
       Installing activerecord-postgis-adapter (0.6.2)
...
```

### Connect to a Postgis enabled database

We now just need to set the `DATABASE_URL` config of our app to connect to a Postgis enabled database from SpacialDB. The [Rgeo postgis gem](http://dazuma.github.io/activerecord-postgis-adapter/rdoc/Documentation_rdoc.html) requires the `adapter: postgis` so our `DATABASE_URL` will look like:

```console
$ heroku config:set DATABASE_URL="postgis://<username>:<password>@spacialdb.com:9999/<database>"
...
```

### Create the Photo resource

First, lets generate a Photo resource for the API with a Postgis `Point` geometry column to store the latitude and longitude of the photo’s coordinates:

```console
$ rails generate resource Photo
...
```

And the migration:

```ruby
class CreatePhotos < ActiveRecord::Migration
  def change
    create_table :photos do |t|
      t.point :lnglat, srid: 4326
      t.index :lnglat, spatial: true

      t.timestamps
    end
  end
end
```

## Further reading:

  * [Heroku Dev Center](https://devcenter.heroku.com/)
  * [SpacialDB devcenter](http://devcenter.spacialdb.com)
