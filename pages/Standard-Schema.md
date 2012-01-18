# SpacialDB Standard Schema Documentation

The SpacialDB `Standard Schema` lets you store geometries and key-value attributes within a named layers. Although the geometry (PostGIS Geometry Type) and key-value attributes (HStore) can be accessed via a database connection, SpacialDB provides a very convenient and highly optimised JSON and JSONP REST API for accessing these tables. Its called the `Layers API`. The `Layers API` is designed to provide developers with a flexible schema that can be used by most applications without needing a database connection. For the specifics of the API see the [[Layers API v1 Documentation]] or [[Layers API v1 Recipes]] page.

In SpacialDB lingo these tables are called Layers and they are perfect for building **mobile** applications. Currently Layers are created and managed via the SpacialDB Command-Line interface. You can create them with a single command like `spacialdb layers:add NAME`. This will give you an operational geospatial database table and access to the API. For more on managing layers see [[API layer management | CLI-Usage#api_layer_management]] section in [[CLI-Usage]] page or type `spacialdb help layers` on the command-line. 

>  **To use this feature make sure you have updated your spacialdb gem to v0.3 or above.**

## Why use Standard Schema?

Thus SpacialDB `Standard Schema` abstracts the complexity of storing and querying geospatial data by treating each standard schema table as a Layer. The RESTful JSON and JSONP `Layers API` allows you to access these layers using your API keys. The basic syntax of the layer is as follows:

      /users/:user/layers/:layername?key=<accessKey>[&callback=?]

In cases where geospatial data is returned the response is in `GeoJSON`. Check out this [quick and dirty map](http://jsfiddle.net/sabman/n85Qr/12/) thrown together using the Layers API and a Standard Schema Layer.
