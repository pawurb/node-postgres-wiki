__incomplete documentation__

Not to be created directly from its constructor, the __Query__ is returned from [[Client#query|Client#method-query-simple]]. It functions primarily as an EventEmitter allowing you to handle returned rows.

- events
  - [[row|Query#event-row]]
  - [[error|Query#event-error]]
  - [[end|Query#event-end]]

## Events

<div id="event-row"></div>
### Row : (__object__ row)

Emitted by the query whenever a row is received from the PostgreSQL server upon query execution, the event listeners are passed the parsed, type-coerced row .

#### example
```javascript
    var query = client.query('SELECT name, age as user_age FROM users');
    query.on('row', function(row) {
      console.log('user "%s" is %d years old', row.name, row.user_age);
    });
```

<div id="event-error"></div>
### Error : (__object__ error)

Emitted by the query when an error is encountered within the context of query execution, the event listeners are passed the error event from the PostgreSQL server.

_note: if this event is not handled it will propagate to the global event loop.  This can potentially crash your node process._

#### example
```javascript
    var query = client.query('SELECT asdfasdfasdf');
    query.on('error', function(error) {
      //handle the error
    });
```

<div id="event-end"></div>
### End

Emitted by the query when all rows have been returned __or__ when an error has been encountered.  In either circumstance, the query's execution is finished and it is no longer interacting with the connection.

### todo: have this return the postgresql server status on insert/update/delete etc commands

## Prepared statements

I'm still working on the API for prepared statements.  Check out the tests for more up to date examples, but what I'm working towards is something like this:


     var client = new Client({
       user: 'brian',
       database: 'test'
     });
    
     var query = client.query({
       text: 'select * from person where age < $1',
       values: [21]
     });

     query.on('row', function(row) {
       console.log(row);
     });

     query.on('end', function() { client.end() });