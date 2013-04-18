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
```

The app should now be up and running and you can go to it by typing:

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

We add the migration of a `lnglat` geographic point column and create a spatial index on it:

```ruby
class CreatePhotos < ActiveRecord::Migration
  def change
    create_table :photos do |t|
      t.point :lnglat, :geographic => true

      t.timestamps
    end

    add_index :photos, :lnglat, spatial: true
  end
end
```


For the `Photos` model we now have to specify the RGeo factory as well as the `:nearby` scope which uses the Postgis `ST_DWithin()` to find photos around a given point and radius. Finally we also add the `gem 'rgeo-geojson'` to the `Gemfile` and set the [GeoJson](http://www.geojson.org/) generator for the Postgis columns:

```ruby
class Photo < ActiveRecord::Base
  attr_accessible :lnglat

  GEOG_FACTORY ||= RGeo::Geographic.spherical_factory(:srid => 4326)
  set_rgeo_factory_for_column(:lnglat, GEOG_FACTORY)

  RGeo::ActiveRecord::GeometryMixin.set_json_generator(:geojson)

  validates :lnglat, :presence => true

  scope :nearby, lambda { |radius_in_km, lng, lat|
    point = GEOG_FACTORY.point(lng, lat)
    where("ST_DWithin(lnglat, ST_GeomFromText('#{point}'), #{radius_in_km.to_f*1000} )")
  }
end
```

The controller needs to specify the fact that we only respond to Json requests and implement the CRUD methods for the Photos resource:

```ruby
class PhotosController < ApplicationController
  respond_to :json

  def index
    lat, lng = params[:lat], params[:lng]
    radius = params[:radius] || 5

    if lat and lng
      @photos = Photo.nearby(radius, lng.to_f, lat.to_f)
      respond_with({:photos => @photos})
    else
      respond_with({:message => "Invalid or missing lng/lat parameters"}, :status => 406)
    end
  end

  def show
    @photo = Photo.find(params[:id])
    respond_with(@photo)
  end

  def create
    @photo = Photo.new
    @photo.lnglat = "POINT(#{params[:lng]} #{params[:lat]}"
    @photo.save

    respond_with(@photo)
  end
end
```

Finally in `seeds.rb` we create some data:

```ruby
locations = {
  :sf   => [37.75, 122.68],
  :atx  => [30.30, 97.70],
  :pit  => [40.50, 80.22],
  ...
}

locations.values.each do |coordinate|
  photo = Photo.new
  photo.lnglat = "POINT(#{coordinate[1]} #{coordinate[0]})"
  photo.save
end
```

After we save, commit and push these changes to Heroku, we can then run the following to migrate and seed the data:

```console
$ heroku run bundle exec rake db:migrate db:seed
==  CreatePhotos: migrating ==
...
==  CreatePhotos: migrated (2.7966s) ==
```

We can test to see if the API works by sending it a request via `curl`:

```console
$ curl "http://young-reef-5849.herokuapp.com/photos.json?lng=110&lat=32&radius=1000"
{"photos":[
  {"created_at":"2013-04-14T16:30:38Z","id":6,"lnglat":{"type":"Point","coordinates":[106.6,35.05]},"updated_at":"2013-04-14T16:30:38Z"},
  {"created_at":"2013-04-14T16:30:39Z","id":9,"lnglat":{"type":"Point","coordinates":[118.4,33.93]},"updated_at":"2013-04-14T16:30:39Z"}
]}
```

### Paperclip gem and S3 storage

To continue a pre-requisite is an Amazon AWS account, which we will use for storing images via their S3 service, and Imagemagick installed on your system. In Rails we will use the `paperclip` gem to add file attachment capabilities to our Photos resource. We essentially follow the [original tutorial](https://devcenter.heroku.com/articles/ios-photo-sharing-geo-location-service) here, the only difference being that we now seed the data with an example image and we return the GeoJson representation of `lnglat` in our model:

```ruby
def as_json(options = nil)
  {
    :lnglat  => self.lnglat.as_json(options),

    :image_urls => {
      :original => self.image.url,
      :thumbnail => self.image.url(:thumbnail)
    },

    :created_at => self.created_at.iso8601
  }
end
```

With that the backend API should be complete and perhaps we can also add a `delete` method to remove photos.

## Further reading:

  * [Heroku Dev Center](https://devcenter.heroku.com/)
  * [SpacialDB devcenter](http://devcenter.spacialdb.com)
