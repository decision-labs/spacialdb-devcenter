## What is SpacialDB?

SpacialDB is a cloud based Geospatial database.


# SpacialDB Cloud Database

[SpacialDB][1] is a Geospatial database service that allows you to create, operate and scale dedicated Geospatial databases in the cloud. Your SpacialDB databases can be used transparently in place of the Heroku-provided PostgreSQL database or a other cloud based databases such as Amazon RDS or Rackspace.

   [1]: http://www.spacialdb.com (SpacialDB)
   

## Deploying to Heroku

To use SpacialDB on Heroku, install the SpacialDB add-on:

```console
$ heroku addons:add spacialdb:test
```

**Note:** As database provisioning is carried out asynchronously, please wait until the creation and initialization of the database complete. You can monitor the progress of this process by calling `heroku config` and looking for the value of the `SPACIALDB_STATUS` variable. Alternatively you can check the provisioning status by clicking the SpacialDB add-on in the Add-ons menu of your application’s Resources.


## Connecting to SpacialDB

The SpacialDB add-on will store the database’s credentials in a [config
vars][2]: `SPACIALDB_URL`. The URL contains a string of the following syntax:


  [2]: http://docs.heroku.com/config-vars (Heroku Config Vars)

    postgres://{username}:{password}@{host}:{port}/{dbname}

The parts out of which the URLs are made of are also set for your convenience in the following environment variables:

```console
SPACIALDB_ADAPTER     # coming soon
SPACIALDB_USERNAME    # coming soon
SPACIALDB_PASSWORD    # coming soon
SPACIALDB_HOST        # coming soon
SPACIALDB_PORT        # coming soon
SPACIALDB_NAME        # coming soon
```

The following example demonstrates use of the connection URL with ActiveRecord's PostGIS Adapter and GeoRuby:

```ruby
require 'rubygems'
require 'pg'
require 'active_record'

def establish_active_record_connection(url)
  regex = Regexp.new("(.*):\\/\\/(.*):(.*)@(.*):([\\d]{3,5})\\/(.*)")
  matchdata = regex.match(url)

  if matchdata.length == 7
    ActiveRecord::Base.establish_connection(
      :adapter => "postgresql",
      :username => matchdata[2],
      :password => matchdata[3],
      :host => matchdata[4],
      :port => Integer(matchdata[5]),
      :database => matchdata[6]
    )
  end
end

begin
  establish_active_record_connection(ENV["SPACIALDB_URL"])
  ActiveRecord::Base.connection
  puts "Connection to SpacialDB established."
rescue => e
  abort "Failed to connect to database: #{e.message}"
end
```

## Local setup

To try out the basic connection locally you can try saving the above script and calling with the `SPACIALDB_URL` environment variable set. If you save the script to a file called `test_spacialdb_connection.rb`

```console
$ SPACIALDB_URL='postgres://{username}:{password}@beta.spacialdb.com:{port}/{database}' ruby test_spacialdb_connection.rb
Connection to SpacialDB established.
```

## Ruby on Rails

Coming soon...

## Sintra

Coming soon...

## Rack apps

Coming soon...


Further reading:

  * [SpacialDB devcenter](http://devcenter.spacialdb.com)
