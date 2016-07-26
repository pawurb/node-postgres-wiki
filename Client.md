Your main interface point with the PostgreSQL server.  Client is used to create & dispatch queries to Postgres.  Client also emits events from Postgres for 'LISTEN/NOTIFY' processing and non-critical error and notice messages from the server.

- methods
  - [[connect|Client#wiki-method-connect]]
  - [[end|Client#wiki-method-end]]
  - [[query (simple)|Client#wiki-method-query-simple]]
  - [[query (parameterized)|Client#wiki-method-query-parameterized]]
  - [[query (prepared statement)|Client#wiki-method-query-prepared]]
- events
  - [[drain|Client#wiki-event-drain]]
  - [[error|Client#wiki-event-error]]
  - [[notification|Client#wiki-event-notification]]
  - [[notice|Client#wiki-event-notice]]
  - [[end|Client#wiki-event-end]]
  
## Constructors
_note: **Client** instances created via the constructor do **not** participate in [[pg]]'s connection pooling.  To take advantage of connection pooling (recommended) please use either [pg-pool](https://github.com/brianc/node-pg-pool) or a pooling utility such as [pgbouncer](https://wiki.postgresql.org/wiki/PgBouncer)._
### new Client(): _Client_

This is the __preferred__ way to create a client - let the client read its connection parameters out of environment variables: the client will read host, database, user, password, etc from the same environment variables [used by postgres utilities](https://www.postgresql.org/docs/9.5/static/libpq-envars.html)

### new Client(<em>string</em> url): _Client_
### new Client(<em>string</em> domainSocketFolder): _Client_

Creates a new, unconnected client from a url based connection string `postgres://user:password@host:port/database` or from the location of a domain socket folder `/tmp` or `/var/run/postgres`.

Internally the connection string is parsed and a _config_ object is created with the same defaults as outlined below.  All parts of the connection string url are optional.  This is handy for use in managed hosting like [[Heroku|http://heroku.com]].

#### example
```javascript
    var client = new Client('postgres://brian:mypassword@localhost:5432/dev');
    var client = new Client('postgres://brian@localhost/dev'); //will use defaults
    var client = new Client(process.env.DATABASE_URL); //something like this should get you running with heroku
    var client = new Client('/tmp');  //looks for the socket file /tmp/.s.PGSQL.5432
```

Caution : 
Url strings don't allow to pass special characters like `#`
If you have some of them in your passord, don't use a connection string, use a config object and pass
it as { host: 'foo', password: 'blah#blah' }

### new Client(<em>object</em> config) : _Client_

Creates a new, unconnected instance of a Client configured via supplied configuration object.

#### parameters

- _object_ __config__: can contain any of the following optional properties
  - _string_ __user__:
     - default value: `process.env.USER`
     - PostgreSQL user
  - _string_ __database__:
     - default value: `process.env.USER`
     - database to use when connecting to PostgreSQL server
  - _string_ __password__:
     - default value: `null`
     - user's password for PostgreSQL server
  - _number_ __port__:
     - default value: `5432`
     - port to use when connecting to PostgreSQL server
     - used to initialize underlying net.Stream()
  - _string_ __host__:
     - default value: `localhost`
     - host address of PostgreSQL server (or a path such as `/var/run/postgresql` for Unix sockets)
     - note: `localhost` still uses TCP (instead of Unix) sockets for the non-native connector
     - used to initialize underlying net.Stream()
  - _bool_/_object_ __ssl__:
     - default value: `false`
     - whether to try SSL/TLS to connect to server
     - if you wish to alter any SSL connection parameters, while using the the postgres javascript client implementation, pass the same options as [tls.connect()](https://nodejs.org/api/tls.html#tls_tls_connect_port_host_options_callback). Default values for tls.connect() options are overridden by this module, pass them explicitly. Eg: to use SSL certificate verification, pass values to the `ca` parameter and set the `rejectUnauthorized` paramether to `true` 
  - _string_ __application_name__:
    - default value: `process.env.PGAPPNAME`
    - name displayed in the `pg_stat_activity` view and included in CSV log entries
  - _string_ __fallback_application_name__:
    - default value: `false`
    - fallback value for the `application_name` configuration parameter


#### tcp example

```javascript
    var client = new Client({
      user: 'brianc',
      password: 'boom!',
      database: 'test',
      host: 'example.com',
      port: 5313
    });
```

#### domain socket example

Will look for the Unix Domain Socket at `/tmp/.s.PGSQL.5313` and connect with the rest of the supplied credentials:

```javascript
    var client = new Client({
      user: 'brianc',
      password: 'boom!',
      database: 'test',
      host: '/tmp',
      port: 5313
    });
```

## Methods

<a name="method-connect"></a>
### connect(<em>optional function</em> callback) : _null_

Initializes __Client's__ internal __[[Connection]]__ object & net.Stream() instance.  Starts communication with PostgreSQL server including password negotiation.  If a callback is supplied it will be called with an instance of `Error` if an error was encountered during the connection procedure, otherwise it will be called with `null` for a single parameter after a connection to PostgreSQL server is established and the client is ready to dispatch queries.

_note: **Clients** created via the **[[pg#connect]]** method are already connected and should **not** have their #connect method called._
_____

<a name="method-end"></a>
### end() : _null_

Immediately sends a termination message to the PostgreSQL server and closes the underlying net.Stream().  

_note: **Clients** created via the **[[pg#connect]]** method will be automatically disconnected or placed back into the connection pool and should **not** have their #end method called._
_____

<a name="method-query-simple"></a>
### _Simple queries_

### query(<em>string</em> text, <em>optional function</em> callback) : _[[Query]]_

Simply: Creates a query object, queues it for execution, and returns it.

In more detail: Adds a __[[Query]]__ to the __Client__'s internal [[query queue|QueryQueue]].  The query is executed as a simple query within PostgresSQL, takes no parameters, and it is parsed, bound, executed, and all rows are streamed backed to the __Client__ in one step within the PostgreSQL server.  For more detailed information [you can read the PostgreSQL protocol documentation](http://developer.postgresql.org/pgdocs/postgres/protocol-flow.html#AEN87085).

#### parameters

 - _string_ __text__: the query text
 - _optional function_ __callback__: optionally provided function which will be passed the error object (if the query raises an error) or the entire result set buffered into memory.  _note: do not provide this function for large result sets unless you're okay with loading the entire result set into memory_
  - _function_ callback(_object_ error, _object_ result) 
    - Called only if provided
    - if passed, query will still raise the `row` and `end` events but will _no longer raise_ the `error` event
    - parameters
      - _object_ __error__:
        - `null` if there was no error
        - if PostgreSQL encountered an error during query execution, the message will be called here
      - _object_ __result__:
        - the result of the query, containing the same properties as the [[Result object|Query#result-object]] in `end` event of `Query`.

#### examples

##### simple query with row callback
```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

    //query is executed once connection is established and
    //PostgreSQL server is ready for a query
    var query = client.query("SELECT name FROM users", function(err, result) {
      console.log(result.rows[0].name);
    })
```

##### simple query with promise
```js
  var client = new Client();
  client.query('SELECT NOW() as right_now')
    .then(res => console.log(res.rows[0].right_now))
    .then(() => client.end())
```

_____

<a name="method-query-parameterized"></a>
### _Parameterized Queries_

### query( <em>object</em> config, <em>optional function</em> callback) : _[[Query]]_
### query(<em>string</em> queryText, <em>array</em> values, <em>optional function</em> callback): _[[Query]]_

Creates an unnamed query object, queues it for execution, and returns it.

If `name` is provided within the `config` object the query will be executed as a [[prepared statement|Prepared-Statements]].  Otherwise, if `values` is provided within the `config` object the query will be executed as a [[parameterized query|Prepared-Statements#parameterized-queries]]. If Otherwise, it will behave in the same manner as a <a href="#method-query-simple">simple query</a>.

#### examples
##### parameterized query with config object

```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

    var query = client.query({
      text: 'SELECT name FROM users WHERE email = $1',
      values: ['brianc@example.com']
    }, function(err, result) {
      console.log(result.rows[0].name) // output: brianc
    });
```

##### parameterized query using string/array initialization
```javascript
 
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

    client.query("SELECT name FROM users WHERE email = $1", ['brianc@example.com'], function(err, result) {
      console.log(result.rows[0].name) // output: brianc
    });
```

##### parameterized query with optional callback supplied
```javascript
    
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

    //object config method
    var queryConfig = {
      text: 'SELECT name FROM users WHERE email = $1',
      values: ['brian@example.com']
    };
    client.query(queryConfig, function(err, result) {
      assert.equal('brianc', result.rows[0]);
    });

    //text/params method
    client.query('SELECT name FROM users WHERE email = $1', ['brian@example.com'], function(err, result) {
      assert.equal('brianc', result.rows[0].name);
    });
```

_____

<a name="method-query-prepared"></a>
### _Prepared statements_

### query(<em>object</em> config, <em>optional function</em> callback) : _[[Query]]_

(See [[Prepared Statements]] for a more detailed discussion of Prepared Statements in `node-postgres`.)

Creates a named query object, queues it for execution, and returns it.:

 * If and only if `name` is provided within the `config` object does query result in a prepared statement.
 * If `text` and `name` are provided within the `config`, the query will result in the creation of a <a href="Query#prepared-statement">prepared statement</a>.
 * If `values` and `name` provided within the `config`, the prepared statement will be executed.  (Note: if the prepared statement takes no parameters, use `values: []`.)

PostgreSQL server caches prepared statements by name on a per (postgres) session basis.  Subsequent queries may refer to the prepared statement by name, and the PostgresQL server instance can skip the preparation step.

#### examples
##### prepared statement reuse
```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

    var first = client.query({
      text: "SELECT email FROM users WHERE name = $1",
      values: ['brianc'],
      name: 'email from name'
    });
    first.on('row', function(row) {
      assert.equal("brian@example.com", row.email);
    });

    var second = client.query({
      name: 'email from name',
      values: ['brianc']
    });
    second.on('row', function(row) {
      assert.equal("brian@example.com", row.email);
    });

    //can still supply a callback method
    var third = client.query({name: 'email from name', values: ['brianc']}, function(err, result) {
      assert.equal('brian@example.com', result.rows[0].email);
    });
```

#### parameters
- _object_ __config__:  can contain any of the following optional properties
  - _string_ __text__: 
    - The text of the query
    - _example:_ `select name from user where email = $1`
  - _string_ __name__:
    - The name of the prepared statement
    - Can be used to reference the same statement again later and is used internally to cache and skip the preparation step
  - _array_ __values__:
    - The values to supply as parameters
    - Values may be any [[object type supported|Supportedtypes]] by the Client
- _optional function_ __callback__: callback function
  - _function_ callback(_object_ error, _object_ result) 
    - Called only if provided
    - used as a shortcut instead of subscribing to the `row` query event
    - if passed, query will still raise the `row` and `end` events but will _no longer raise_ the `error` event
    - parameters
      - _object_ __error__:
        - `null` if there was no error
        - if PostgreSQL encountered an error during query execution, the message will be called here
      - _object_ __result__:
        - the result of the query, containing the same properties as the [[Result object|Query#result-object]] in `end` event of `Query`.

_____


## Events

<a name="event-drain"></a>
### drain : 

Raised when the internal [[query queue|Queryqueue]] has been emptied and all queued queries have been executed.  Useful for disconnecting the client after running an undetermined number of queries.  

##### example
```javascript
    var client = new Client({user: 'brianc', database: 'postgres'});
    client.connect();
    var users = client.query("select * from user");
    var superdoods = client.query("select * from superman");
    client.on('drain', client.end.bind(client));
    //carry on doing whatever it was you wanted with the query results once they return
    users.on('row', function(row){ ...... });
```
_____

<a name="event-error"></a>
### error : _object_ error

Raised when the client recieves an error message from PostgreSQL _or_ when the underlying stream raises an error.  The single parameter passed to the listener will be the error message or error object.

##### example
```javascript
    var client = new Client({user: 'not a valid user name', database: 'postgres'});
    client.connect();
    client.on('error', function(error) {
      console.log(error);
    });                    
```

_____

<a name="event-notification"></a>
### notification : _object_ message

Used for "LISTEN/NOTIFY" interactions.  You can do some fun pub-sub style stuff with this.

##### example
```javascript
   var client1 = new Client(...)
   var client2 = new Client(...)
   client1.connect();
   client2.connect();
   client1.on('notification', function(msg) {
     console.log(msg.channel);  //outputs 'boom'
     client1.end();
   });
   client1.query("LISTEN boom");
   //need to let the first query actually complete
   //client1 will remain listening to channel 'boom' until its 'end' is called
   setTimeout(function() {
      client2.query("NOTIFY boom", function() {
        client2.end();
      });
   }, 1000);
```

_____

<a name="event-notice"></a>
### notice : _object_ notice

Emitted from PostgreSQL server when non-critical events happen, for example a `RAISE NOTICE` statement in a plpgsql function. When using connection pooling, be sure to attach the handler only once per client.

Libpq `printf`'s these out to stdout if the behavior is not overridden.  Yucky.  Thankfully node-postgres overrides the default behavior and emits an event (instead of printing to stdout) on the client which received the notice event.

##### example
```javascript
    var client = new Client(...)
    client.on('notice', function(msg) {
      console.log("notice: %j", msg);
    });
    //create a table with an id will cause a notice about creating an implicit seq or something like that...
    client.query('create temp table boom(id serial, size integer)');
    client.on('drain', client.end.bind(client));
```

_____
<a name="event-end"></a>
### end : 

Emitted when the connection is finished. It is useful when the pooling mechanism is external to pg.
##### example
```
    client.on('end', function(){console.log("Client was disconnected.");
```


***
[[◄ Back (API - pg)|pg]] `      ` [[Next (API - pg.Query) ►|Query]]