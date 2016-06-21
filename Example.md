### This code should be able to be copy/pasted to give you a sample app running on your machine

You'll need to first "create table visit (date date)" in your postgres database

```javascript
var http = require('http');
var Pool = require('pg').Pool;

// by default the pool will use the same environment variables
// as psql, pg_dump, pg_restore etc:
// https://www.postgresql.org/docs/9.5/static/libpq-envars.html

// you can optionally supply other values
var config = {
  host: 'localhost',
  user: 'foo',
  password: 'bar',
  database: 'my_db',
};

var pool = new Pool(config)

var server = http.createServer(function(req, res) {

  var onError = function() {
    res.writeHead(500, {'content-type': 'text/plain'});
    res.end('An error occurred');
  };

  pool.query('INSERT INTO visit (date) VALUES ($1)', [new Date()], function(err) {
    if (err) return onError();

    // get the total number of visits today (including the current visit)
    pool.query('SELECT COUNT(date) AS count FROM visit', function(err, result) {
      // handle an error from the query
      if(err) return onError();
      res.writeHead(200, {'content-type': 'text/plain'});
      res.end('You are visitor number ' + result.rows[0].count);
    });
  });
});

server.listen(3001)
```

***
[[◄ Back (FAQ)|FAQ]] `      ` [[Next (Internals - Query Queue) ►|Queryqueue]]