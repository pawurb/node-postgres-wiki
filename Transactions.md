node-postgres does not have any special apis for dealing with transactions.  Here is how you would do a transaction using a single instance of a client:

```js
var Client = require('pg').Client;

var client = new Client(/*your connection info goes here*/);
client.connect();

var rollback = function(client) {
  //terminating a client connection will
  //automatically rollback any uncommitted transactions
  //so while it's not technically mandatory to call
  //ROLLBACK it is cleaner and more correct
  client.query('ROLLBACK', function() {
    client.end();
  });
};

client.query('BEGIN', function(err, result) {
  if(err) return rollback(client);
  client.query('INSERT INTO account(money) VALUES(100) WHERE id = $1', [1], function(err, result) {
    if(err) return rollback(client);
    client.query('INSERT INTO account(money) VALUES(-100) WHERE id = $1', [2], function(err, result) {
      if(err) return rollback(client);
      //disconnect after successful commit
      client.query('COMMIT', client.end.bind(client));
    });
  });
});
```