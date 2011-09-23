Work in progress...

## Getting acquatied with the API

### Creating a layer

### Adding data to the layer

```console
$ curl -X POST -d 'properties={"name":"A Park in Berlin","location":"Berlin, Germany"}&\
geometry={"type":"Polygon","coordinates":[[[13.484,52.483],[13.483,52.467],[13.496, 52.464],[13.499,52.480],[13.484,52.483]]]}' \
 https://api.spacialdb.com/1/users/shoaib/layers/parks\?key\=24ccc5c742416dad2fb170b21ee66633
{"id":1}
```

### Getting data from layer

```console
$ curl -X GET https://api.spacialdb.com/1/users/shoaib/layers/parks\?key\=e158ff36d16c9ece0e49359b4903dc2c     
{ "type": "FeatureCollection", "features": [{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[13.484,52.483],[13.483,52.467],[13.496,52.464],[13.499,52.48],[13.484,52.483]]]},"properties":{"name":"A Park in Berlin","location":"Berlin, Germany"},"id":1}]}
```
### Filtering spatially

### Filtering by attributes

## Visulizing data in a javascript map client

### Using leaflet.js

Check out this [quick and dirty map](http://jsfiddle.net/sabman/n85Qr/12/) thrown together using the Layers API and a Standard Schema Layer.

work in progress ...

```javascript
var geojsonLayer = new L.GeoJSON();
geojsonLayer.on('featureparse', function(e) {
  e.layer.setStyle({ color:  '#000088', weight: 2, fill: true, fillColor: '#009933' });
});

$.getJSON(
    'https://api.spacialdb.com/1/users/shoaib/layers/parks\?key\=e158ff36d16c9ece0e49359b4903dc2c&callback=?',
    function(geojson) {
    $.each(geojson.features, function(i, feature) {
      geojsonLayer.addGeoJSON(feature);
    })
});

map.addLayer(geojsonLayer);
```

### Using polymaps
