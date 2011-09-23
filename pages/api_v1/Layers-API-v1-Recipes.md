Work in progress...

## Getting acquatied with the API

### Creating a layer

### Adding data to the layer

### Getting data from layer

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
