# pg

The root __pg__ object returned by `require('pg')` exports a few internal constructors which might be helpful.  
It has a reference to the other components of node-postgres: 

1. [`pg.Client`](https://github.com/brianc/node-postgres/wiki/Client)
2. [`pg.Query`](https://github.com/brianc/node-postgres/wiki/Query)
3. [`pg.Pool`](https://github.com/brianc/node-pg-pool)

There are two common ways to interact with your PostgreSQL server via node-postgres.

First, and what you'll generally want to do is interact with your server via a [pool of clients](https://github.com/brianc/node-pg-pool)

```js
var Pool = require('pg').Pool;
var pool = new Pool({
  user: 'foo',
  password: 'bar',
  host: 'localhost',
  database: 'my_db',
  max: 10, // max number of clients in pool
  idleTimeoutMillis: 1000, // close & remove clients which have been idle > 1 second
});

pool.on('error', function(e, client) {
  // if a client is idle in the pool
  // and receives an error - for example when your PostgreSQL server restarts
  // the pool will catch the error & let you handle it here
});

// you can run queries directly against the pool
pool.query('SELECT $1::text as name', ['foo'], function(err, result) {
  console.log(result.rows[0].name); // output: foo
});

// the query object implements the promise API
pool.query('SELECT $1::text as name', ['foo'])
  .then(res => console.log(res.rows[0].name)); // output: foo

// the pool also supports checking out a client for
// multiple operations, such as a transaction

pool.connect(function(err, client, release) {
  // TODO - you'll want to handle the error in real code

  client.query('SELECT $1::text as name', ['foo'], function(err, result) {
    // you MUST return your client back to the pool when you're done!
    release();
    console.log(result.rows[0].name); // output: foo
  });
});

// again this api supports promises
// if you use something like [co](https://github.com/tj/co) 
// you can end up with much cleaner looking code

co(function * () {
  var client = yield pool.connect()
  try {
      yield client.query('BEGIN')
      var result = yield client.query('SELECT $1::text as name', ['foo'])
      yield client.query('INSERT INTO something(name) VALUES($1)', [result.rows[0].name])
      yield client.query('COMMIT')
      client.release()
  } catch(e) {
    // pass truthy value to release to destroy the client
    // instead of returning it to the pool
    // the pool will create a new client next time
    // this will also roll back the transaction within postgres
    client.release(true)
  }
})
```

If you want to interact with postgres without using the pool, you can instantiate a client directly & use it as follows:

```js
var Client = require('pg').Client;

var client = new Client();
client.connect();
client.query('SELECT $1::text as name', ['foo'], function(err, res) {
  console.log(res.rows[0].name);
  client.end();
});
```

***
[[◄ Back (Installation)|Installation]] `      ` [[Next (API - pg.Client) ►|Client]]