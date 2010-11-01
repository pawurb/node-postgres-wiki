Basically a facade on top of the connection to provide a _much_ more user friendly, "node style" interface for doing all the lovely things you like with PostgreSQL.

## Constructor

### new Client(_object_ config) : _Client_

Creates a new instance of a Client configured via supplied configuration object.  In normal instantiation the client will _not_ be connected automatically.

##### parameters

- __config__: [_object_] can contain any of the following optional properties
  - __user__: [_string_] 
    - default value: `null`
    - PostgreSQL user
  - __database__: [_string_] 
    - default value: `null`
    - database to use when connecting to PostgreSQL server
  - __password__: [_string_]
    - default value: `null`
    - user's password for PostgreSQL server
  - __port__: [_number_] 
    - default value: `5432`
    - port to use when connecting to PostgreSQL server
    - will support unix domain sockets in future
    - used to initialize underlying net.Stream()
  - __host__: [_string_]
    - default value: `null`
    - host address of PostgreSQL server
    - used to initialize underlying net.Stream()
  - __connection__: [_[[Connection]]_]
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

Initializes underlying net.Stream() and startup communication with PostgreSQL server.  Once the connection is finished, the __Client__ emits the _connect_ event.

<div id="method-end">
<h3>end() : _null_</h3>
</div>
Immediately sends a termination message to the PostgreSQL server and closes the underlying net.Stream().

<div id="method-query-simple">&nbsp;</div>
### query(_string_ text) : _[[Query]]_

Simply: Creates a query object, queues it for execution, and returns it.

In more detail: Adds a __[[Query]]__ to the __Client__'s internal [[query queue|QueryQueue]].  The query is executed as a simple query within PostgresSQL, takes no parameters, and it is parsed, bound, executed, and all rows are streamed backed to the __Client__ in one step within the PostgreSQL server.  For more detailed information [you can read the PostgreSQL protocol documentation](http://developer.postgresql.org/pgdocs/postgres/protocol-flow.html#AEN87085).

##### parameters

 - __text__: [_string_] the query text

##### example

```javascript
    var client = new Client({user: 'brianc', database: 'test'});
    client.connect();
    //query is executed once connection is established and
    //PostgreSQL server is ready for a query
    var query = client.query("select name from user")
    query.on('row', function(row) {
      console.log(row.fields[0]);
    });
    query.on('end', client.end.bind(client));
```
<div id="method-query-prepared">&nbsp;</div>
### query(_object_ config) : _[[Query]]_

Creates a (optionally named) query object, queues it for execution, and returns it.

If either `name` or `values` is provided within the `config` object the query will be executed as a <a href="/Query#prepared-statement">prepared statement</a>.  Otherwise, it will behave in the same manor as a <a href="#method-query-simple">simple query</a>.

##### parameters
- __config__: [_object_] can contain any of the following optional properties
  - __text__: [_string_] 
    - The text of the query
    - _example:_ `select name from user where email = $1`
  -__name__: [_string_]
    - The name of the prepared statement
    - Can be used to reference the same statement again later and is used internally to cache and skip the preparation step
  - __values__: [_array_]
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
