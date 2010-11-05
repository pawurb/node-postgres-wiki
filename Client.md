Your main interface point with the PostgreSQL server, the __Client__ is basically a facade on top of the [[Connection]] to provide a _much_ more user friendly, "node style" interface for doing all the lovely things you like with PostgreSQL.

- methods
  - [[connect|Client#method-connect]]
  - [[end|Client#method-end]]
  - [[query (simple)|Client#method-query-simple]]
  - [[query (prepared statement)|Client#method-query-prepared]]
- events
  - [[drain|Client#event-drain]]
  - [[error|Client#event-error]]
  
## Constructor

### new Client(_object_ config) : _Client_

Creates a new instance of a Client configured via supplied configuration object.  In normal instantiation the client will _not_ be connected automatically.

##### parameters

- _object_ __config__: can contain any of the following optional properties
  - _string_ __user__:
    - default value: `null`
    - PostgreSQL user
  - _string_ __database__:
    - default value: `null`
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
  - [_[[Connection]]_] __connection__:
    - default value: `new Connection(config)`
    - the __[[Connection]]__ object used by client.  Only really provided as a config option to aid in testing.  Will be used in the future when connection pooling is implemented

##### example

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

<div id="method-connect">&nbsp;</div>
### connect() : _null_

Initializes underlying net.Stream() and startup communication with PostgreSQL server including password negotiation.

<div id="method-end">&nbsp;</div>
### end() : _null_

Immediately sends a termination message to the PostgreSQL server and closes the underlying net.Stream().

<div id="method-query-simple">&nbsp;</div>
### query(_string_ text) : _[[Query]]_

Simply: Creates a query object, queues it for execution, and returns it.

In more detail: Adds a __[[Query]]__ to the __Client__'s internal [[query queue|QueryQueue]].  The query is executed as a simple query within PostgresSQL, takes no parameters, and it is parsed, bound, executed, and all rows are streamed backed to the __Client__ in one step within the PostgreSQL server.  For more detailed information [you can read the PostgreSQL protocol documentation](http://developer.postgresql.org/pgdocs/postgres/protocol-flow.html#AEN87085).

##### parameters

 - _string_ __text__: the query text

##### example

```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.connect();
    //query is executed once connection is established and
    //PostgreSQL server is ready for a query
    var query = client.query("select name from user")
    query.on('row', function(row) {
      console.log(row.name);
    });
    query.on('end', client.end.bind(client));
```

<div id="method-query-prepared">&nbsp;</div>
### query(_object_ config) : _[[Query]]_

Creates a (optionally named) query object, queues it for execution, and returns it.

If either `name` or `values` is provided within the `config` object the query will be executed as a <a href="Query#prepared-statement">prepared statement</a>.  Otherwise, it will behave in the same manor as a <a href="#method-query-simple">simple query</a>.

##### parameters
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

##### example

```javascript
    var client = new ...
    var query = client.query({
      text: 'select name from user where email = $1',
      name: 'get user by email',
      values: ['brianc@example.com']
    });
    query.on('row', function() {
      //do something w/ yer row data
    });

    var again = client.query({
      name: 'get user by email',
      values: ['brianc@example.net']
    });

    again.on('row', function() {
      //do something else
    });

    again.on('end', client.end.bind(client));

```

## Events

<div id="event-error">&nbsp;</div>
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
<div id="event-drain">&nbsp;</div>
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
