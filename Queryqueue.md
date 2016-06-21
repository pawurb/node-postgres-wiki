__[[Clients|Client]]__ are responsible for creating __[[Queries|Query]]__ via the factory method Client#query.  The __Client__ can create a new query before the client is connected to the server or while other queries are executing.  Internally the __Client__ maintains a queue of __Query__ objects which are popped and executed as the preceding __Query__ completes.  When the __Client__'s internal query queue is emptied, the __Client__ raises the _drain_ event.  It can be useful to monitor this event, and when the __Client__ goes idle, disconnect it from the server.  

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