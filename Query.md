## incomplete documentation

Not to be created directly from its constructor, the __Query__ is returned from [[Client#query|Client#method-query-simple]]. It functions primarily as an EventEmitter allowing you to handle returned rows.

- events
  - [[row|Query#event-row]]
  - [[end|Query#event-end]]
  - [[error|Query#event-error]]

# Events

<div id="event-row"></div>
## Row

emitted by the query whenever a row is received from the PostgreSQL server upon query execution

<div id="event-error"></div>
## Error

emitted by the query when an error is encountered within the context of query execution.  

<div id="event-end"></div>
## End

Emitted by the query when all rows have been returned __or__ when an error has been encountered.  In either circumstance, the query's execution is finished and it is no longer interacting with the connection.

### todo: have this return the postgresql server status on insert/update/delete etc commands

#### Prepared statements

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