# pg

The root __pg__ object returned by `require('pg')` serves two purposes.  
First, it has a reference to the other components of node-postgres: 

1. [`pg.Client`](https://github.com/brianc/node-postgres/wiki/Client)
2. [`pg.Query`](https://github.com/brianc/node-postgres/wiki/Query)
3. `pg.defaults`
4. [`pg.pools`](https://github.com/brianc/node-postgres/wiki/Pools)
5. `pg.types`

The second purpose is more important:

__pg__ is an _instance_ of __EventEmitter__ which provides a somewhat native implementation of __[[Client]]__ pooling.  It has a function to retrieve __[[Client]]__ instances from a pool of available clients.  You can bypass the __pg__ object all together and create __[[Client]]__ objects via their constructor (`new pg.Client()`); however, each __[[Client]]__ represents an open connection to your PostgreSQL server instance, and the initial connection handshake takes many times longer than a single query execution.  Also, If you attempt to create and connect more __[[Client]]__ objects than supported connections to your PostgreSQL server you will encounter errors.  This becomes especially painful if you manually instantiate a new __[[Client]]__ object per each http request to a web server.  Once you receive more simultaneous requests to your web server than your PostgresSQL server can maintain you will be in a bad place...so it's recommended unless you have a particular case, use the __pg__ object to create pooled clients, build your own client pool implementation, or use https://github.com/grncdr/node-any-db.

* Methods
  * [[connect|pg#connect]]
  * [[end|pg#end]]
  * [[cancel|pg#cancel]]
* Properties
  * [[defaults|pg#pgdefaults]]
* Events
  * error

## Methods


<a id="method-connect">
### connect([<em>string</em> connectionString], _function_ callback)
</a>


### connect([<em>object</em> config], _function_ callback)

### connect(<em>function</em> callback)

The _connect_ method retrieves a __[[Client]]__ from the client pool, or if all pooled clients are busy and the pool is not full, the _connect_ method will create a new client passing its first argument directly to the __[[Client]]__ constructor.  In either case, your supplied callback will only be called when the __[[Client]]__ is ready to issue queries or an error is encountered.  The callback will be called once and only once for each invocation of _connect_.  

Note: The first parameter passed to _connect_, either a string or config object (or nothing), currently functions as the key used in pooling clients; therefore, using two different connection strings will result in two separate pools being created.  The object, string, or nothing is passed to `JSON.stringify` for key uniqueness.  If nothing is passed you are relying on connection defaults (via [environment variables](http://www.postgresql.org/docs/9.3/static/libpq-envars.html) or `pg.defaults`) then `JSON.stringify({})` is used as the key.

__note: calling `done()` on a pooled client will cause it to close after idling for 30 seconds. If you do not call `done()` the client will never be returned to the pool and you will leak clients.  This is mega-bad so always call `done()`__

#### parameters

* _string_ __connectionString__
  * a connection string in the format `anything://user:password@host:port/database`
  * a socket path, like `/var/run/postgresql`
  * a socket path, with a specific database, like `/var/run/postgresql a_db_name`
  * a socket connection string `socket:/some/path/?db=database_name&encoding=utf8`
* _object_ __config__
  * an object with user, database, password, port, and host properties as described in [[Client|Client#constructor]].
* _function_ __callback__
  * called exactly once for one of the following reasons
    * new client is created _and_ connected to PostgreSQL
    * an existing client is returned to the internal client pool
    * an error is encountered during connection
  * callback parameters
    * _object_ __error__: error object
      * if there is no error, this will be null
    * _object_ __[[Client]]__ : postgres-node client object ready for queries
      * if there is an error, this object will be null
    * _function_ __done()__: done function
      * if there is an error, this will be a NOOP function `function() {}`
      * a truthy value passed to `done()` will cause the client to be destroyed and removed from the pool (in most cases you don't want this; see the examples below).


#### example
```javascript
    var pg = require('pg');
    var connectionString = "pg://brian:1234@localhost/postgres"
    pg.connect(connectionString, function(err, client, done) {
        client.query('SELECT name FROM users WHERE email = $1', ['brian@example.com'], function(err, result) {
          assert.equal('brianc', result.rows[0].name);
          done();  // client idles for 30 seconds before closing
        });
    });
```

### end()

Disconnects all idle clients within all active pools, and has all client pools terminate.  Any currently _open, checked out_ clients will still need to be returned to the pool before they will be shut down and disconnected.

#### example
```js
var pg = require('pg');

pg.connect(function(err, client, done) {
  done();
});

//your process will not exit immediately because the pool is holding open, idle connections to the server

pg.end();
//the pool will dispose of all clients and your process will terminate
```

### cancel(config, client, query)

Cancels the given query.

#### example
```js
var pg = require('pg');

pg.connect(conString, function(err, client, done) {
  var query = client.query('SELECT pg_sleep(1)', function(err, result) {
    if(err) {
      // this should print "error: canceling statement due to user request"
      console.error("%s", err);
    }
    done();
  });

  setImmediate(function() {
    pg.cancel(client.connectionParameters, client, query);
  });
});
```

## pg.defaults

The __pg__ object has a set of defaults.
 
#### pg.defaults.user

The default user to use when connecting via tcp sockets (md5 or plaintext) if a user is not provided to the individual __Client__ instance.  Default value is `process.env.USER`

#### pg.defaults.database

The default database to use if a database is not provided to the individual __Client__ instance.  Default value is `process.env.USER`

#### pg.defaults.password

The default password to use when connecting via tcp sockets (md5 or plaintext) if a password is not provided to the individual __Client__ instance and PostgreSQL server requires a password. Default value is `null`

#### pg.defaults.host

The default host if a host is not provided to the individual __Client__ instance.  Can be a domain name, ip address, or path to unix socket folder.  Default value is `localhost`

Implementation note: currently (v4.4.1, August 2015) the host `localhost` will connect to a TCP socket instead of Unix sockets for the non-native connector. Explicitly specify a value such as `/var/run/postgresql` or `/tmp` to force use of Unix domain sockets.

#### pg.defaults.port

The default port if a port is not provided to the individual __Client__ instance.  In the case of a unix socket, the port becomes the extension to the socket file.  Default value is `5432`

#### pg.defaults.ssl

Whether to try SSL/TLS to connect to server. Default value is `false`.

#### pg.defaults.rows

Number of rows to return at a time from a prepared statement's portal. 0 will return all rows at once.

#### pg.defaults.poolSize

Number of unique __Client__ objects to maintain in the pool.  Default value is `10`.  If you want to change the pool size you must modify this value __before__ creating a pool.  Example:

```js
var pg = require('pg');
pg.defaults.poolSize = 25;

//pool is created on first call to pg.connect
pg.connect(function(err, client, done) {
  done();
});

pg.defaults.poolSize = 2;
//pool still has a size of 25
pg.connect(function(err, client, done) {
  done();
});

//creating a new pool by using a different connection string
//will create this second pool with a size of 2
pg.connect('other-pool', function(err, client, done) {

});
```

#### pg.defaults.poolIdleTimeout

Max milliseconds a client can go unused before it is removed from the pool and destroyed. Default value is `30000` (30 seconds)

#### pg.defaults.reapIntervalMillis

Frequency to check for idle clients within the client pool. Default value is `1000` (1 second).

#### pg.defaults.binary

Binary result mode, defaults to `false`

#### pg.defaults.parseInt8

By default fields with type int8 are returned as strings because JavaScript cannot represent 64-bit numbers. In practice this causes a problem generally because the result of a `COUNT` operation is an int8. Since in reality most of us aren't going to be dealing with giant numbers and it sucks having all your `COUNT(*)` results come out as strings, you can opt-in to parsing int8 as integers.  Just be wary: you will lose data if you are dealing with actual 64bit numbers where more than 32bits are used.  You can always specify your own type parsers, but this is a handy shortcut.

## Events

### 'error' : _object_ error, _object_ client

Emitted whenever a pooled client emits an error.  An idle client will likely only emit an error when it loses connection to the PostgreSQL server instance, for example when your database crashes (oh no!).  The pooled client which emitted the error is automatically removed from the pool and supplied to the callback.

***
[[◄ Back (Installation)|Installation]] `      ` [[Next (API - pg.Client) ►|Client]]