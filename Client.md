Your main interface point with the PostgreSQL server.  Client is used to create & dispatch queries to Postgres.  Client also emits events from Postgres for 'LISTEN/NOTIFY' processing and non-critical error and notice messages from the server.

- methods
  - [[connect|Client#method-connect]]
  - [[end|Client#method-end]]
  - [[query (simple)|Client#method-query-simple]]
  - [[query (prepared statement)|Client#method-query-prepared]]
  - [[pauseDrain|Client#wiki-pauseDrain]]
  - [[resumeDrain|Client#wiki-pauseDrain]]
- events
  - drain
  - error
  - notification
  - notice
  
## Constructors
_note: _Client_ instances created via the constructor do __not__ participate in connection pooling.  To take advantage of connection pooling (recommended) please use the [[pg]] object._
### new Client(_string_ url): _Client_
### new Client(_string_ domainSocketFolder): _Client_

Creates a new, unconnected client from a url based connection string `postgres://user:password@host:port/database` or from the location of a domain socket folder `/tmp` or `/var/run/postgres`.

Internally the connection string is parsed and a _config_ object is created with the same defaults as outlined below.  All parts of the connection string url are optional.  This is handy for use in managed hosting like [[Heroku|http://heroku.com]].

#### example
```javascript
    var client = new Client('postgres://brian:mypassword@localhost:5432/dev');
    var client = new Client('postgres://brian@localhost/dev'); //will use defaults
    var client = new Client(process.env.DATABASE_URL); //something like this should get you running with heroku
    var client = new Client('/tmp');  //looks for the socket file /tmp/.s.PGSQL.5432
```

### new Client(_object_ config) : _Client_

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
     - will support unix domain sockets in future
     - used to initialize underlying net.Stream()
  - _string_ __host__:
     - default value: `null`
     - host address of PostgreSQL server
     - used to initialize underlying net.Stream()

#### example

```javascript
    var client = new Client({
      user: 'brianc',
      password: 'boom!'
      database: 'test'
      host: 'example.com'
      port: 5313
    });
```

## Methods

### connect(_optional function_ callback) : _null_

Initializes __Client's__ internal __[[Connection]]__ object & net.Stream() instance.  Starts communication with PostgreSQL server including password negotiation.  If a callback is supplied it will be called with an instance of `Error` if an error was encountered during the connection procedure, otherwise it will be called with `null` for a single parameter after a connection to PostgreSQL server is established and the client is ready to dispatch queries.

_note: __Clients__ created via the __[[pg]]__#connect method are already connected and should __not__ have their #connect method called._
_____

### end() : _null_

Immediately sends a termination message to the PostgreSQL server and closes the underlying net.Stream().  

_note: __Clients__ created via the __[[pg]]__#connect method will be automatically disconnected or placed back into the connection pool and should __not__ have their #end method called._
_____

### _Simple queries_

### query(_string_ text, _optional function_ callback) : _[[Query]]_

Simply: Creates a query object, queues it for execution, and returns it.

In more detail: Adds a __[[Query]]__ to the __Client__'s internal [[query queue|QueryQueue]].  The query is executed as a simple query within PostgresSQL, takes no parameters, and it is parsed, bound, executed, and all rows are streamed backed to the __Client__ in one step within the PostgreSQL server.  For more detailed information [you can read the PostgreSQL protocol documentation](http://developer.postgresql.org/pgdocs/postgres/protocol-flow.html#AEN87085).

#### parameters

 - _string_ __text__: the query text
 - _optional function_ __callback__: optionally provided function which will be passed the error object (if the query raises an error) or the entire result set buffered into memory.  _note: do not provide this function for large result sets unless you're okay with loading the entire result set into memory_
  - _function_ callback(_object_ error, _object_ result) 
    - Called only if provided
    - __buffers all rows into memory before calling__
      - rows only buffered if callback is provided
      - can impact memory when buffering large result sets (i.e. do not provide a callback)
    - used as a shortcut instead of subscribing to the `row` query event
    - if passed, query will still raise the `row` and `end` events but will _no longer raise_ the `error` event
    - #### parameters
      - _object_ __error__:
        - `null` if there was no error
        - if PostgreSQL encountered an error during query execution, the message will be called here
      - _object_ __result__:
        - and object containing the following properties:
          - _array_ __rows__: 
            - an array of all rows returned from the query
            - each row is equal to one object passed to the Query#row callback


#### examples
##### simple query without callback
```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.connect();
    //query is executed once connection is established and
    //PostgreSQL server is ready for a query
    var query = client.query("SELECT name FROM users");
    query.on('row', function(row) {
      console.log(row.name);
    });
    query.on('end', client.end.bind(client)); //disconnect client manually
```

##### simple query with optional row callback
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

_____

### _Prepared statements_

### query(_object_ config, _optional function_ callback) : _[[Query]]_
### query(_string_ queryText, _array_ values, _optional function_ callback): _[[Query]]_

Creates a (optionally named) query object, queues it for execution, and returns it.

If either `name` or `values` is provided within the `config` object the query will be executed as a <a href="Query#prepared-statement">prepared statement</a>.  Otherwise, it will behave in the same manner as a <a href="#method-query-simple">simple query</a>.

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
    - __buffers all rows into memory before calling__
      - rows only buffered if callback is provided
      - can impact memory when buffering large result sets (i.e. do not provide a callback)
    - used as a shortcut instead of subscribing to the `row` query event
    - if passed, query will still raise the `row` and `end` events but will _no longer raise_ the `error` event
    - #### parameters
      - _object_ __error__:
        - `null` if there was no error
        - if PostgreSQL encountered an error during query execution, the message will be called here
      - _object_ __result__:
        - and object containing the following properties:
          - _array_ __rows__: 
            - an array of all rows returned from the query
            - each row is equal to one object passed to the Query#row callback

#### examples
##### prepared statement with config object

```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

    var query = client.query({
      text: 'SELECT name FROM users WHERE email = $1',
      values: ['brianc@example.com']
    });

    query.on('row', function(row) {
      //do something w/ yer row data
      assert.equal('brianc', row.name);
    });
```

##### prepared statement using string/array initialization
```javascript
 
    var client = new Client({user: 'brianc', database: 'test'});
    client.on('drain', client.end.bind(client)); //disconnect client when all queries are finished
    client.connect();

     var again = client.query("SELECT name FROM users WHERE email = $1", ['brianc@example.com']);

    again.on('row', function(row) {
      //do something else
      assert.equal('brianc', row.name);
    });
```

##### prepared statement with optional callback supplied
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

The proceeding examples used an 'unamed' prepared statement.  PostgreSQL server caches prepared statements by name on a per client basis.  If a name is supplied for the statement all following executions of the query can refer to it by name and the PostgreSQL server instance can skip the preparation step.

##### named prepared statement reuse 
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

### pauseDrain / resumeDrain <a name="pauseDrain"></a>

Pair of methods used to pause and resume __Client__ from raising it's `drain` event when it's query queue is emptied.  The `drain` event signifies the __Client__ has no more pending queries and can safely be returned back to a client pool.  Normally, `drain` will be emitted   These methods come in handy for doing async work between queries or within a transaction and disabling the __Client__ from alerting anyone it has gone idle.

#### example
```javascript
var client = new Client(/*connection params*/);
client.connect();
client.on('drain', function() {
  console.log('client has drained');
});
client.pauseDrain();
client.query("SELECT NOW() AS when", function(err, result) {
  console.log("first");
  setTimeout(function() {
    client.query("SELECT NOW() AS when", function(err, result) {
      console.log("second");
      client.resumeDrain(); //now client will emit drain
    });
  }, 1000);
});
//output: 
// first
// second
// client has drained
```
## Events

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
   //client1 will remain listening to channel 'boom' until it's 'end' is called
   setTimeout(function() {
      client2.query("NOTIFY boom", function() {
        client2.end();
      });
   }, 1000);
```


### notice : _object_ notice

Emitted from PostgreSQL server when non-critical events happen.  Libpq `printf`'s these out to stdout if the behavior is not overridden.  Yucky.  Thankfully node-postgres overrides the default behavior and emits an event (instead of printing to stdout) on the client which received the notice event.

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