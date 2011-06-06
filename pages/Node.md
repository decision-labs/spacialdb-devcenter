# Connecting to SpacialDB from Node.js

In this article we learn to connect to SpacialDB using Node.js.

## Prerequisites

1. npm: If you are new to Node.js you will need the [[node package manager|http://npmjs.org/]] installed. This article uses version 1.0.9-1.

2. coffee-script: Since we will be writing the code in `coffee-script` you need to install [[coffee-script|http://jashkenas.github.com/coffee-script/]] by simply doing: `npm install -g coffee-script`

3. spacialdb account: you need an account on [[SpacialDB|http://spacialdb.com]]. <a class='button' href='mailto:info@spacialdb.com?try-node'> Get beta invite</a>

**Note**: If you find that you are installing something globally by using the `-g` option to `npm`, but cannot `require()` it the you might need to install it locally without the `-g` option to `npm`. In this case, the modules are installed in the `node_module/` folder of the current working directory of a particular project, and the packages binaries are installed in the `node_module/.bin/` folder. In the absence of a project, this folder is created in the home directory: `~/node_module/`, and so you need to make sure that your `PATH` and `NODE_PATH` environment variables are set appropriately. You might also want to use `npm link` for linking your current working code into  Node's  path,  so that you don't have to reinstall every time you make a change.

## PostgreSQL Client

The [[node-postgres|https://github.com/brianc/node-postgres]] project provides a non-blocking PostgreSQL client for [[node.js|nodejs.org]], with pure JavaScript and native `libpq` bindings, and we use it to connect to SpacialDB. If you have `npm` installed then simply:

```console
$ npm install pg

> pg@0.5.0 install /Users/kashif/node_modules/pg
> node-waf configure build || true

Checking for program g++ or c++          : /usr/bin/g++ 
Checking for program cpp                 : /usr/bin/cpp 
Checking for program ar                  : /usr/bin/ar 
Checking for program ranlib              : /usr/bin/ranlib 
Checking for g++                         : ok  
Checking for node path                   : ok /Users/kashif/.node_libraries 
Checking for node prefix                 : ok /usr/local/Cellar/node/0.4.7 
Checking for program pg_config           : /usr/local/bin/pg_config 
'configure' finished successfully (0.604s)
Waf: Entering directory `/Users/kashif/node_modules/pg/build'
[1/2] cxx: src/binding.cc -> build/default/src/binding_1.o
[2/2] cxx_link: build/default/src/binding_1.o -> build/default/binding.node
Waf: Leaving directory `/Users/kashif/node_modules/pg/build'
'build' finished successfully (2.827s)
pg@0.5.0 ../../node_modules/pg 
```

should build and install the bindings assuming you have `libpq` or `postgresql` installed.

## Simple example

We can use the [[node-coffee-project|https://github.com/sstephenson/node-coffee-project]] template written in CoffeeScript to set up our example project. So just clone it to:

```console
$ git clone https://github.com/sstephenson/node-coffee-project.git spacialdb-example
```

The first thing we do is to add `node-postgres` to the `package.json` as a dependency:

```javascript
, "dependencies":
  {"pg": ">=0.5.0"
  }
```
and running `npm install` in the root folder should install all the dependencies in a `node_module/` folder in the current working directory. So we can now go ahead and write some code.

By default, the `require('pg')` will use the Javascript bindings, and to use the `libpq` bindings just do the following in say your `index.coffee` file in the `src/` folder:

```coffeescript
pg = require('pg').native
```

`node-postgres` supports both an *event emitter* style API and a *callback* style API, the callback style being  more concise and generally preferred. But the evented API is also handy and you can mix the two if you need.

But very simply to use the bindings, you need your SpacialDB connection string to `connect` to SpacialDB and then send a `query` and display the `result`. So in our `index.coffee` we add:

```coffeescript
conString = "postgres://krasul:mypasswd@beta.spacialdb.com:9999/spacialdb0_krasul"

pg.connect conString, (err, client) ->
  client.query "SELECT NOW() as when", (err, result) ->
    console.log "Row count: %d", result.rows.length
    console.log "Current year: %d", result.rows[0].when.getYear()
```

Now we run: `cake build` to generate the Javascript in the `lib/` folder and can execute it with `node`.