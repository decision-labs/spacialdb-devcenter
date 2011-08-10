# API v1

<div name='toc'></div>
## Table of contents:

* [API Overview](#api_overview)
* [Layers resource](#layers_resource)
* [Publish a feature for a given layer](#create_feature)
* [Update a feature of a layer with given id](#update_feature)
* [Delete a feature from a layer given the feature id](#delete_feature)

---

<div name='api_overview'></div>
# API Overview

This describes the resources that make up the official SpacialDB API v1. If you have any problems please contact [support](mailto:info@spacialdb.com?subject=API+Support). SpacialDB has a RESTful query API that let you perform Create, Read, Update and Delete operations on you core Geospatial Layers. These layers need to conform to SpacialDB's standard schema. More details on standard schema can be found at [[spacialdb standard schema]] page.

---

<div name='layers_resource'></div>
## Layers resource 
The Layer resource maps to a SpacialDB table. It can store geospatial features and key-value attributes.

---

<div name='create_feature'></div>
## Publish a feature for a given layer

Creates a new feature.

### Url:

```bash
POST /users/:user/layers/:layername?key=<accessKey>
```

### Input:

* **geometry**
  - Optional GeoJSON Geometry Object

* **properties**
  - Optional JSON Object

* **key**
  - Required _String_, access key for your account/layer

Requires at least one of the above geometry or properties attribute.

```javascript
{
    "geometry": <GeoJSON Geometry Object>
    "properties": {
        <key1>: <value1>,
        <key2>: <value2>
    }
}
```

### Response:

#### Code/Header

```bash
200 - OK
```

####Body:

```javascript
{
    id: 42
}
```

[Back to Table of Contents](#toc)

---

<div name='update_feature'></div>
## Update a feature of a layer with given id

###Url:

```bash
PUT /users/:user/layers/:layername/:id?key=<accesKey>
```
###Input:

* **geometry**
 * Optional GeoJSON Geometry Object
* **properties**
 * Optional JSON OBject
* **key**
 * Required _String_, access key for your account/layer

Requires at least one of the above.

```javascript
{
    "geometry": <GeoJSON Geometry Object>
    "properties": {
        <key1>: <value1>,
        <key2>: <value2>
    }
}
```

###Response:

####Code:

```bash
204 - NO CONTENT
```

---

<div name='delete_feature'></div>
## Delete a feature from a layer given the feature id

###Url:

```bash
DELETE /users/:user/layers/:layername/:id?key=<accessKey>
```

####Input:
* **key**
 * Required _String_, access key for your account/layer

###Body:

###Response:

####Code:

```bash
204 - NO CONTENT
```

---

<div name='get_all_layer_features'></div>
## Get the all features in a layer

###Url:

```bash
GET /users/:user/layers/:layername?key=<accessKey>
```
  

###Input:
* **key**
 * Required _String_, access key for your account/layer

###Response:

####Code:

```bash
200 - OK
```


####Body:

```javascript
{
    "type": "FeatureCollection",
    "features": [
        {
            "id": 42
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [-0.925268, 51.552603]
            }
            "properties": {
                "office": "Awesome Company",
                "nice": "true"
            }

        }
    ]
}
```

---


<div name='get_feature'></div>
## Get a feature of a layer by id

###Url:

```bash
GET /users/:user/layers/:layername/:id?key=<accessKey>
```

###Input:
* **key**
 * Required _String_, access key for your account/layer

Supports all [extended request options](#extended_request_options).

###Response:

####Code:

```bash
200 - OK
```


####Body:

```javascript
{
    "id": 42
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-0.925268, 51.552603]
    }
    "properties": {
        "office": "Awesome Company",
        "nice": "true"
    }
}
```

---


<div name='get_feature_attr'></div>

## Get a single feature with only the requested attributes e.g. the id, properties

###Url:

```bash
GET /users/:user/layers/:layername/:id?key=<accessKey>&attr=id,properties
```

###Input:
* **key**
 * Required _String_, access key for your account/layer
* **attr**
 * Required
###Response:

####Code:

```bash
200 - OK
```


####Body:

```javascript
{
    "id": 42
    "properties": {
        key: "value",
        key2: "value2"
    }
}
```

---

## Get a collection of features with only the requested attributes e.g. id, properties

###Url:

```bash
GET /users/:user/layers/:layername?key=<accessKey>&attr=id,properties
```

###Body:

###Return:

####Code:

```bash
200 - OK
```


####Body:

```javascript
 {
    "type": "FeatureCollection",
    "features": [
        {
            "id": 42
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [-0.925268, 51.552603]
            }
            "properties": {
                "office": "Awesome Company",
                "nice": "true"
            }

        }
    ]
}
```

---

## Get all features intersecting with the given geometry

###Url:

```bash
POST /users/:user/layers/:layername/functions/intersects?key=<accessKey>
```

###Input:

* **key**
 * Required _String_, access key for your account/layer
* **geometry**
 * Required GeoJSON Geometry in the body

```javascript
{
  "geometry": <GeoJSON Geometry Object>
}
```

###Response:

####Code:

```bash
200 - OK
```


####Body:

```javascript
{
    "type": "FeatureCollection",
    "features": [
        {
            "id": 42
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [-0.925268, 51.552603]
            }
            "properties": {
                "office": "Awesome Company",
                "nice": "true"
            }
        }
    ]
}
```

---

## Get all features within (inclusion) given geometry

###Url:

    POST /users/:user/layers/:layername/functions/st_within?key=<accessKey>

###Input:

* **key**
 * Required _String_, access key for your account/layer
* **geometry**
 * Required GeoJSON Geometry in the body

```javascript
{
    "geometry": <GeoJSON Geometry Object>
}
```

###Response:

####Code:

```bash
200 - OK
```


####Body:

```javascript
{
    "type": "FeatureCollection",
    "features": [
        {
            "id": 42
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [-0.925268, 51.552603]
            }
            "properties": {
                "office": "Awesome Company",
                "nice": "true"
            }
        }
    ]
}
```

---

##Get all features with the given properties

###Url:

```bash
POST /users/:user/layers/:layername?key=<accessKey>
```

###Input:

* **key**
 * Required _String_, access key for your account/layer
* **operator**
 * Required _Key-Value-Pair_ which defines the operator to combine the properties. Valid values are be _or_ or _and_.
* **properties**
 * Requried _Object properties with Key-Value-Pairs_ which will be part of the WHERE condition

```javascript
{
    "operator": "or",
    "properties": {
        "office": "Awesome Company",
        "nice": "true"
    }
}
```

###Response:

####Code:

```bash
200 - OK
```


####Body:

```javascript
{
    "type": "FeatureCollection",
    "features": [
        {
            "id": 42
            "type": "Feature",
            "geometry": {
                "type": "Point",
                "coordinates": [-0.925268, 51.552603]
            }
            "properties": {
                "office": "Awesome Company",
                "nice": "true"
            }
        }
    ]
}
```

---

<div name='get_features_sorted'></div>
## Get all features sorted by a property key

* **key**
 * Required _String_, access key for your account/layer
* **sort**
 * Required _String_ which enables sorting
* **order**
 * Required _String_ which defines the order of the sorting. Could be _asc_ or _desc_.
* **sorton**
 * Required _String_ which defines the property key we sort on.


###Url:

```bash
POST /users/:user/layers/:layername?key=<accessKey>&sort
```

###Body:

```javascript
{
   "order": "asc",
   "sorton": "name"
}
```

###Response:

####Code:

```bash
200 - OK
```

####Body:

```javascript
{
    "type": "FeatureCollection",
    "features": [
        {
            "id": 1,
            "type": "Feature",
            "geometry": {
                "type": "Point"
                "coordinates": [42.0 23.0]
            }
            "properties": {
                name: "bar"
            }
        },
        {
            "id": 2,
            "type": "Feature",
            "geometry": {
                "type": "Point"
                "coordinates": [23.0 42.0]
            }
            "properties": {
                name: "foo"
            }
        }
    ]
}
```

---

<div name='count_features'></div>
## Count the features in a collection/layer

###Url:

```bash
GET /users/:user/layer/:layername?key=<accessKey>&count
```

###Input:

* **key**
 * Required _String_, access key for your account/layer
* **count**
 * Required, enables counting.

###Response:

####Code:

```bash
200 - OK
```

####Body:

```javascript
{
    count: 42
}
```

---

<div name='extended_request_options'></div>
# Extended request options

Extended request options let you do more fine tuned requests by supporting parameters for:

## Sorting

* [Get all features sorted by a property key](#get_features_sorted)

## Requested only specific attributes

* [Get a single feature with only the requested attributes e.g. the id, properties](#get_feature_attr)

## Counting

* [Count the features in a collection/layer](#count_features)

