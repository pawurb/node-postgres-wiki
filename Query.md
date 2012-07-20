Not to be created directly from its constructor, the __Query__ is returned from [[Client#query|Client#method-query-simple]]. It functions primarily as an EventEmitter allowing you to handle returned rows.

- events
  - [[row|Query#event-row]]
  - [[error|Query#event-error]]
  - [[end|Query#event-end]]

## Events

<div id="event-row"></div>
### Row : (__object__ row, __object__ result)

Emitted by the query whenever a row is received from the PostgreSQL server upon query execution.

The event listeners get two arguments:

- row: The parsed, type-coerced row. It is a simple key/value Object.
- result: A special [[Result|Query#result-object]] object that can be used to accumulate rows.

#### examples
```javascript
    var query = client.query('SELECT name, age as user_age FROM users');
    query.on('row', function(row) {
      console.log('user "%s" is %d years old', row.name, row.user_age);
    });
```

```javascript
    var query = client.query('SELECT name, age as user_age FROM users');
    query.on('row', function(row, result) {
      result.addRow(row);
    });
```

<div id="event-error"></div>
### Error : (__object__ error)

Emitted by the query when an error is encountered within the context of query execution, the event listeners are passed the error event from the PostgreSQL server.

If the __Query__ object was created with an optional callback function, the error event will __not__ fire.  This prevents you from having to handle the error both in the callback function and on the event listener.

_note: If this event (or any event with the name 'error') is not handled it will propagate to the global event loop.  This can potentially crash your node process.  This is standard node.js behavior.._

#### examples

##### no callback function

```javascript
    var query = client.query('SELECT asdfasdfasdf');
    query.on('error', function(error) {
      //handle the error
    });
```

##### callback function
```javascript
    var query = client.query('SELECT LAKJDLSKJF', function(err, result) {
      //err is the error returned from the PostgreSQL server
      //handle the error here
    });
    query.on('error', function() {
      //this code will never execute
      assert.ok(false, "This will never be called because you supplied the optional query callback function");
    })
```

<div id="event-end"></div>
### End

Emitted by the query when all rows have been returned __or__ when an error has been encountered.  In either circumstance, the query's execution is finished and it is no longer interacting with the connection.

The __end__ event listeners get one argument, which is the [[Result|Query#result-object]] object.

### examples

```javascript
    var query = client.query('select name from person');
    var rows = [];
    query.on('row', function(row) {
      //fired once for each row returned
      rows.push(row);
    });
    query.on('end', function(result) {
      //fired once and only once, after the last row has been returned and after all 'row' events are emitted
      //in this example, the 'rows' array now contains an ordered set of all the rows which we received from postgres
      console.log(result.rowCount + ' rows were received');
    })
```

<div id="result-object"></div>
### Result object

Available to the __row__ and __end__ events, shows the result of the query. It has the following properties:

- `command`: The sql command that was executed (e.g. "SELECT", "UPDATE", etc.)
- `rowCount`: The number of rows returned
- `oid`: The oid returned
- `rows`: An array of rows (if the `addRow` command is used)

It also has the following method:

- `addRow(row)`: Appends the `row` object to the `Result.rows` array. Normally this is the `row` object returned by the __row__ event, but it can be anything you want.

### example

```javascript
    var query = client.query('select name from person');
    query.on('row', function(row, result) {
      result.addRow(row);
    });
    query.on('end', function(result) {
      console.log(result.rows.length + ' rows were received');
    });
```

## Prepared statements

Need to document...
