node-postgres does not have any special APIs for dealing with transactions; just use sql.  

Below are two examples of how to run transactions.  Generally you'll probably use some kind of flow control library or abstract this behind a function or something.  I use nothing but node & pg to keep the examples as clear as possible.

Here is how you would do a transaction using the client pool:
```js
var pg = require('pg');
var rollback = function(client, done) {
  client.query('ROLLBACK', function(err) {
    //if there was a problem rolling back the query
    //something is seriously messed up.  Return the error
    //to the done function to close & remove this client from
    //the pool.  If you leave a client in the pool with an unaborted
    //transaction weird, hard to diagnose problems might happen.
    return done(err);
  });
};
pg.connect(function(err, client, done) {
  if(err) throw err;
  client.query('BEGIN', function(err) {
    if(err) return rollback(client, done);
    //as long as we do not call the `done` callback we can do 
    //whatever we want...the client is ours until we call `done`
    //on the flip side, if you do call `done` before either COMMIT or ROLLBACK
    //what you are doing is returning a client back to the pool while it 
    //is in the middle of a transaction.  
    //Returning a client while its in the middle of a transaction
    //will lead to weird & hard to diagnose errors.
    process.nextTick(function() {
      var text = 'INSERT INTO account(money) VALUES($1) WHERE id = $2';
      client.query(text, [100, 1], function(err) {
        if(err) return rollback(client, done);
        client.query(text, [-100, 2], function(err) {
          if(err) return rollback(client, done);
          client.query('COMMIT', done);
        });
      });
    });
  });
});
```

Here is how you would do a transaction using a single instance of a client:

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

***
[[◄ Back (Prepared Statements)|Prepared-Statements]] `      ` [[Next (FAQ) ►|FAQ]]