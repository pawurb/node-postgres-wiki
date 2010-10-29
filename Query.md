need to document

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