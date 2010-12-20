# pg
__pg__ is an small _object_ which provides transparent __[[Client]]__ life-cycle management and client pooling.  

_You're absolutely free to bypass the pg class completely in favor of your own client pool implementation; however, it is very, very not recommended to create an individual, new __[[Client]]__ instance for each request to a web server.  It will work fine in development but once your web server receives more simultaneous requests than your PostgreSQL server can support, your new __[[Client]]__ instances will all emit the `error` event when they attempt to connect and you will be in...how you say...a world of hurt._

* Methods
  * [[connect|pg#method-connect]]
  * [[end|pg#method-end]]
* Properties
  * [[defaults|pg#properties-defaults]]

#### example
```javascript
    require('pg');
    
    var connectionString = "pg://brian:1234@localhost/postgres"
    pg.connect(connectionString, function(err, client) {
      client.query('SELECT name FROM users WHERE email = $1', ['brian@example.com'], function(err, result) {
        assert.equal('brianc', result.rows[0].name);
      });
    });
```

## Methods

### Connect(_string_ connectionString, _function_ callback)

### Connect(_object_ config, _function_ callback)

The connect method retrieves a __[[Client]]__ from the client pool.  If all clients are busy and the pool has available slots, it will create a new client passing the first argument to _connect_ directly to the __[[Client]]'s__ constructor.  In either case, the callback will only fire when the __[[Client]]__ is ready to issue queries or an error is encountered.  The callback will fire once and only once for each invocation of _connect_.  The first parameter passed to _connect_ currently functions as the key used in pooling clients; therefore, using two different connection strings will result in two separate pools being created.  _this might change in the future if it causes problems.  I'm considering creating pools based on a host/database combo from the connection information instead of the entire string or config object_

#### parameters

* _string_ __connectionString__
  * a connection string in the format _anything://user:password@host:port/database_
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