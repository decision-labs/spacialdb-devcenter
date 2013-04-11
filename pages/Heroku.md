# SpacialDB on Heroku

## What is SpacialDB?

SpacialDB is a cloud based Geospatial database.


# SpacialDB Cloud Database

[SpacialDB][1] is a Geospatial database service that allows you to create, operate and scale dedicated Geospatial databases in the cloud. Your SpacialDB databases can be used transparently in place of the Heroku-provided PostgreSQL database.

   [1]: http://www.spacialdb.com (SpacialDB)


## Deploying to Heroku

To use SpacialDB on Heroku, make sure that you have a database provisioned with SpacialDB and you have signed in to Heroku via their toolbelt suite of applications:

```console
$ heroku login
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



### Rails app (the server)

We create a new app with:

```console
$ rails new geo-photo --database=postgresql
...
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
$ cd geo-photo
```

While we are at it, lets check it in and login to heroku:

```console
$ git init
...
$  git add .
$  git commit -m "Initial commit"
...
$ heroku login
Enter your Heroku credentials.
Email: kashif.rasul@gmail.com
Password (typing will be hidden):
Authentication successful.
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

So one solution is to install precompiled binary libraries with the `vendorbinaries` buildpack together with the default Heroku Ruby buildpack. In order to use these two buildpacks, we will use the `buildpack-multi`. Have a look at this blog post: [Building and Deploying Custom Binaries on Heroku](http://spin.atomicobject.com/2012/12/09/building-and-deploying-custom-binaries-on-heroku/) for more info.

Finally the `vendorbinaries` buildpack requires a url to download the binaries from, and we have provided the latest Proj (4.8.0) and Geos (3.3.8) binaries via a public S3 bucket.

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

Now we are ready to add the RGeo gem. However there is one complication, the location of the libraries is not in any default location, so we need to let the `gem` command know the exact location which in our case is the root of our app: `/app`. So we use bunder to pass the specify flags to this one specific gem namerly: Rgeo. Normally we would do `gem install rgeo -- --with-geos-dir=/app --with-proj-dir=/app` but thats not possible on Heroku. So can add the following config to our app which will let bundler know the options when installing the Rgeo gem:

```console
$ heroku config:set BUNDLE_BUILD__RGEO="--with-geos-dir=/app --with-proj-dir=/app"
Setting config vars and restarting young-reef-5849... done, v9
BUNDLE_BUILD__RGEO: --with-geos-dir=/app --with-proj-dir=/app
```

So lets add the gems in the `Gemfile`, commit and deploy it:

```console
$ cat Gemfile
...
gem 'rgeo'
gem 'activerecord-postgis-adapter'
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

We now just need to set the `DATABASE_URL` config of our app to connect to a Postgis enabled database from SpacialDB. The Rgeo postgis gem requires the `adapter: postgis` so our `DATABASE_URL` looks like:

```console
$ heroku config:set DATABASE_URL="postgis://<username>:<password>@spacialdb.com:9999/<database>"
...
```

## Further reading:

  * [SpacialDB devcenter](http://devcenter.spacialdb.com)
