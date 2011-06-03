## PostgreSQL Client

The [[node-postgres|https://github.com/brianc/node-postgres]] project provides a non-blocking PostgreSQL client for [[node.js|nodejs.org]], with pure JavaScript and native `libpq` bindings, and we use it to connect to SpacialDB. If you have `npm` installed then simply:

```console
$ npm install pg

> pg@0.5.0 install /Users/kashif/node/nack/node_modules/pg
> node-waf configure build || true

Checking for program g++ or c++          : /usr/bin/g++ 
Checking for program cpp                 : /usr/bin/cpp 
Checking for program ar                  : /usr/bin/ar 
Checking for program ranlib              : /usr/bin/ranlib 
Checking for g++                         : ok  
Checking for node path                   : not found 
Checking for node prefix                 : ok /usr/local/Cellar/node/0.4.8 
Checking for program pg_config           : /usr/local/bin/pg_config 
'configure' finished successfully (0.179s)
Waf: Entering directory `/Users/kashif/node/nack/node_modules/pg/build'
[1/2] cxx: src/binding.cc -> build/default/src/binding_1.o
[2/2] cxx_link: build/default/src/binding_1.o -> build/default/binding.node
Waf: Leaving directory `/Users/kashif/node/nack/node_modules/pg/build'
'build' finished successfully (1.187s)
```

should build and install the bindings assuming you have `libpq` or `postgresql` installed.

## Simple example

By default, the `require('pg')` will use the Javascript bindings, and to use the `libpq` bindings just do the following in say your `index.coffee` file:

```coffeescript
pg = require('pg').native
```

`node-postgres` supports both an *event emitter* style API and a *callback* style API, the callback style being  more concise and generally preferred. But the evented API is also handy and you can mix the two if you need.

But very simply to use the bindings, you need your SpacialDB connection string to `connect` to SpacialDB and then send a `query` and display the `result`:

```coffeescript
conString = "postgres://krasul:mypasswd@beta.spacialdb.com:9999/spacialdb0_krasul"

pg.connect conString, (err, client) ->
  client.query "SELECT NOW() as when", (err, result) ->
     console.log "Row count: %d", result.rows.length
     console.log "Current year: %d", result.rows[0].when.getYear()
```