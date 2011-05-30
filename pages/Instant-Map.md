# Learn to quickly import geospatial data and view a map

Goal of this article is to take us from sign up to a working instance of SpacialDB. By the end we will know how to create a SpacialDB instance and import-data to it and see it on a map. Something like:

![German Cities](/img/German-Cities.png)

Sign up via the website or the command-line. For command-line use the SpacialDB Ruby Gem. If you are not familiar with Ruby or Gems (Ruby's package manager) then you just have to make sure you have ruby installed. Most flavours of *nixs come installed with ruby. If you are on windows you can get a single click installer from: [http://ruby-lang.org](http://ruby-lang.org).

To install the command-line client just do `gem install spacialdb` in you terminal or windows console.

For a complete reference of the Command-Line Usage check out the [[CLI-Usage]] page.

## Sign up

Via the website sign up at: [http://beta.spacialdb.com/signup/free](http://beta.spacialdb.com/signup/free)

Create a username and password. Username can only contain alpha-numeric values. 

CLI command:

```console
$ spacialdb signup

Sign up to Spacialdb.
Email: shoaib@spacialdb.com
Username: shoaib
Password: 
Password confirmation: 
Signed up successfully.
```


## Login

If you just signed up you are also automatically logged in. Otherwise, to login you need your email or username and password.

Via the website login at: [http://beta.spacialdb.com/login](http://beta.spacialdb.com/login)

CLI command:

```console
$ spacialdb login

Enter your Spacialdb credentials.
Email or Username: shoaib
Password: 
Logged in successfully.
```

## Creating a Geospatial database

Awesome! Signed up, logged-in; we are ready for our first geospatial database. If you have used PostGIS before you will have all the PostGIS goodness you are used to, with the added bonus of accessibility from anywhere and anytime. Not to mention some of the real-time APIs for mobile development that will be available soon.

Via the website after login you will be redirected to your dashboard at: [http://beta.spacialdb.com/dashboard](http://beta.spacialdb.com/dashboard). You will see the **New Database** button here. Go ahead and click it... and bam! you have a fresh install of a personal Geospatial database. You will see the connection parameters here.

CLI command:

```console
$ spacialdb create

{
  "host":"beta.spacialdb.com",
  "name":"spacialdb1_shoaib",
  "password":"13971abfeb",
  "port":9999,
  "username":"shoaib"
}
```

## Connecting via QGIS

QGIS is big! but get it. Why? Its one of the most feature rich open source desktop GIS packages out-there. And it will come in handy as you work with more and more geospatial data. We would recommend the last stable release: [http://www.qgis.org/en/download/current-software.html](http://www.qgis.org/en/download/current-software.html). Install it. After installing QGIS its time to connect and import some initial data.

## Download data

We want Shapefiles. Most prevalent geospatial data format currently available on the web... SpacialDB envisions changing that (but more on that later). For now lets get the following datasets:

* [boundary_with_populations_germany (zip)](http://j.mp/iwO6Ee) from GeoCommons
* [german_cities_ranked_by_population (zip)](http://j.mp/mc5sEc) from GeoCommons

## Download the PostGIS plugin

QGIS has a plugin manager. From there we get our hands on the Spit plugin. A great little utility for importing Shapefiles straight into PostGIS.

## Upload some data

Lets fire up the plugin to connect to your new database and import the Shapefiles we have just downloaded. It should slurps it right in.

## View the data.

Click **Add a PostGIS layer** button and you can connect to your new database.

