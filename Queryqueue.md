An advanced usage feature which might come in handy is using the internal query queue to queue up several queries in a row.  __[[Clients|Client]]__ are responsible for creating __[[Queries|Query]]__ via the factory method Client#query.  The __Client__ can create a new query before the client is connected to the server or while other queries are executing.  Internally the __Client__ maintains a queue of __Query__ objects which are popped and executed as the preceding __Query__ completes.  The error handling semantics here get really complicated and isn't recommended unless you're banging out a quick script and don't care about error handling or you really, really have some specific use case.  This "feature" is mostly a hold-over from a bad design decision I made years ago, but have left to maintain backwards compatibility.

#### example
```javascript
    var client = new Client(...);
    var query1 = client.query("SELECT * FROM NOW()"); //query is queued.  client is not connected
    query1.on('end', function() {
      console.log('query 1 completed');
    });
    var query2 = client.query("SELECT * FROM NOW()"); //also queued
    query2.on('end', function() {
      console.log('query 2 completed');
    });
    client.on('drain', function() {
      console.log("drained");
    });
    //at this point nothing has been printed to the console
    client.connect();
    //this will print the following:
    //query 1 completed
    //query 2 completed
    //drained
```

***
[[◄ Back (Example App)|Example]] `      ` [[Next (Testing) ►|Testing]]