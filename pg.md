# pg
__pg__ is an _instance_ of __EventEmitter__ which provides __[[Client]]__ pooling.  It exports a 'helper function' to retrieve __[[Client]]__ instances from a pool of available clients.  You can bypass the __pg__ object all together and create __[[Client]]__ objects via their constructor; however, each __[[Client]]__ represents an open connection to your PostgreSQL server instance.  If you attempt to create and connect more __[[Client]]__ objects than supported connections to your PostgreSQL server you will encounter errors.  This becomes especially painful if you manually instantiate a new __[[Client]]__ object per each http request to a web server.  Once you receive more simultaneous requests to your web server than your PostgresSQL server can maintain you will be in a bad place...so it's recommended unless you have a particular case, use the __pg__ object to create clients, but avoid the two currently flawed border cases described in [issue #227](https://github.com/brianc/node-postgres/issues/227).

* Methods
  * [[connect|pg#method-connect]]
  * [[end|pg#method-end]]
* Properties
  * [[defaults|pg#properties-defaults]]
* Events
  * error

#### example
```javascript
    var pg = require('pg');
    var needToLookUpEmail = true;

    var connectionString = "pg://brian:1234@localhost/postgres"
    pg.connect(connectionString, function(err, client) {
      if (needToLookUpEmail)
        client.query('SELECT name FROM users WHERE email = $1', ['brian@example.com'], function(err, result) {
          assert.equal('brianc', result.rows[0].name);
        });
      /* if we don't run a query, release this client from the pool anyway so we don't eat connections */
      else client.emit('drain');
    });
```

## Methods

### Connect(_string_ connectionString, _function_ callback)

### Connect(_object_ config, _function_ callback)

### Connect(_function_ callback)

The connect method retrieves a __[[Client]]__ from the client pool, or if all pooled clients are busy and the pool is not full, the _connect_ method will create a new client passing its first argument directly to the __[[Client]]__ constructor.  In either case, your supplied callback will only be called when the __[[Client]]__ is ready to issue queries or an error is encountered.  The callback will be called once and only once for each invocation of _connect_.  The first parameter passed to _connect_ currently functions as the key used in pooling clients; therefore, using two different connection strings will result in two separate pools being created.

If called with only one _function_ argument, uses [[defaults|pg#properties-defaults]] for connection configuration.

When the client raises its __drain__ [[event|Client#wiki-drainEvent]], the client is automatically returned to the pool for reuse.

#### parameters

* _string_ __connectionString__
  * a connection string in the format _anything://user:password@host:port/database_
* _object_ __config__
  * an object with user, database, password, port, and host properties as described in [[Client|Client#constructor]].
* _function_ __callback__
  * called exactly once for one of the following reasons
    * new client is created _and_ connected to PostgreSQL
    * an existing client is returned to the internal client pool
    * an error is encountered during connection
  * callback parameters
    * _object_ __error_: error object
      * if there is no error, this will be null
    * _object_ __[[Client]]__ : postgres-node client object ready for queries
      * if there is an error, this object will be null

### end(_optional string_ poolKey)

Disconnects all clients within a pool if _poolKey_ is provided, or disconnects all clients in all pools.  Not very clean and can potentially interrupt query executions.  Primarily used during testing to allow the node process to shutdown after all the tests are executed.  I'm currently evaluating routes for cleaning up and shutting down client pools as gracefully as possible.

## pg.defaults

The __pg__ object has a set of defaults.
 
#### pg.defaults.user

The default user to use when connecting via tcp sockets (md5 or plaintext) if a user is not provided to the individual __Client__ instance.  Default value is `process.env.USER`

#### pg.defaults.database

The default database to use if a database is not provided to the individual __Client__ instance.  Default value is `process.env.USER`

#### pg.defaults.password

The default password to use when connecting via tcp sockets (md5 or plaintext) if a password is not provided to the individual __Client__ instance and PostgreSQL server requires a password. Default value is `null`

#### pg.defaults.host

The default host if a host is not provided to the individual __Client__ instance.  Can be a domain name, ip address, or path to unix socket folder.  Default value is `null`

#### pg.defaults.port

The default port if a port is not provided to the individual __Client__ instance.  In the case of a unix socket, the port becomes the extension to the socket file.  Default value is `5432`

#### pg.defaults.rows

Number of rows ro return at a time from a prepared statement's portal. 0 will return all rows at once.

#### pg.defaults.poolSize

Number of unique __Client__ objects to maintain in the pool.  If this value is set to 0, pooling will be disabled and pg#connect will always return a new client.

#### pg.defaults.poolIdleTimeout

Max milliseconds a client can go unused before it is removed from the pool and destroyed. Default value is `30000`

#### pg.defaults.reapIntervalMillis

Frequency to check for idle clients within the client pool

#### pg.defaults.binary

Binary result mode, defaults to `false`

## Events

### 'error' : _object_ error, _object_ client

Emitted whenever a pooled client emits an error.  An idle client will likely only emit an error when it loses connection to the PostgreSQL server instance, for example when your database crashes (oh no!).  The pooled client which emitted the error is automatically removed from the pool and supplied to the callback.