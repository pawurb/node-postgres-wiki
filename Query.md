## incomplete documentation

Not to be created directly from its constructor, the __Query__ is returned from [[Client#connect|Client#method-query-simple]]. It functions primarily as an EventEmitter allowing you to handle returned rows.

- events
  - [[row|Query#event-row]]
  - [[end|Query#event-end]]

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