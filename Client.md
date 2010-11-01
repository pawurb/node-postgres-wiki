Basically a facade on top of the connection to provide a _much_ more user friendly, "node style" interface for doing all the lovely things you like with PostgreSQL.

### new Client(Object config) : Client

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