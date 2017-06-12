### This code should be able to be copy/pasted to give you a sample app running on your machine

You'll need to do `npm i pg` before you run this example.  Make sure node-postgres is installed at minimum `6.0.0`.

```javascript
var http = require('http');
var Pool = require('pg').Pool;

// by default the pool will use the same environment variables
// as psql, pg_dump, pg_restore etc:
// https://www.postgresql.org/docs/9.5/static/libpq-envars.html

// you can optionally supply other values
var config = {
  //host: 'localhost',
  //user: 'foo',
  //password: 'bar',
  //database: 'my_db',
};

process.on('unhandledRejection', function(e) {
  console.log(e.message, e.stack)
})

// create the pool somewhere globally so its lifetime
// lasts for as long as your app is running
var pool = new Pool(config)

var server = http.createServer(function(req, res) {

  var onError = function(err) {
    console.log(err.message, err.stack)
    res.writeHead(500, {'content-type': 'text/plain'});
    res.end('An error occurred');
  };

  pool.query('INSERT INTO visit (date) VALUES ($1)', [new Date()], function(err) {
    if (err) return onError(err);

    // get the total number of visits today (including the current visit)
    pool.query('SELECT COUNT(date) AS count FROM visit', function(err, result) {
      // handle an error from the query
      if(err) return onError(err);
      res.writeHead(200, {'content-type': 'text/plain'});
      res.end('You are visitor number ' + result.rows[0].count);
    });
  });
});

pool
  .query('CREATE TABLE IF NOT EXISTS visit (date timestamptz)')
  .then(function() {
    server.listen(3001, function() {
      console.log('server is listening on 3001')
    })
  })

```

### Using promises

How to run a pool query using promises

```javascript
pool.query('SELECT $1::int AS number', ['3'])
  .then((res) => {
    console.log('number:', res.rows[0].number);
  })
  .catch((err) => {
    console.error('error running query', err);
  });
```

This is how you get a client from a pool using promises

```javascript
pool.connect()
  .then((client) => {
    client.query('SELECT $1::int AS number', ['4'])
      .then((res) => {
        client.release();
        console.log('number:', res.rows[0].number);
      })
      .catch((err) => {
        console.error('error running query', err);
      });
  })
  .catch((err) => {
    console.error('error fetching client from pool', err);
  });
```

### Using async/await

The `query()` and `connect()` methods return promises and therefore can be awaited. Here is how you run a pool query.

```javascript
async function queryExample() {
  try {
    var res = await pool.query('SELECT $1::int AS number', ['5']);
    console.log('number:', res.rows[0].number);
  } catch (err) {
    console.error('error running query', err);
  }
}

queryExample();
```

Finally this is how you would obtain a client from the pool using await.

```javascript
async function clientExample() {
  try {
    var client = await pool.connect();
    try {
      var res = await client.query('SELECT $1::int AS number', ['6']);
      console.log('number:', res.rows[0].number);
    } catch (err) {
      console.error('error running query', err);
    }
    client.release();
  } catch (err) {
    console.error('error fetching client from pool', err);
  }
}

clientExample();
```

***
[[◄ Back (FAQ)|FAQ]] `      ` [[Next (Internals - Query Queue) ►|Queryqueue]]