Basically a facade on top of the connection to provide a _much_ more user friendly, "node style" interface for doing all the lovely things you like with PostgreSQL.

### new Client(Object config) : Client

##### parameters

- __config__: [_object_] can contain any of the following properties
  - __user__: [_string_] 
    - default value: `''`
    - PostgreSQL user
  - __database__: [_string_] database to use when connecting to PostgreSQL server
  - __port__: [_number_] port to use when connecting to PostgreSQL server
  - __host__: [_string_]