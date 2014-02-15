### This code should be able to be copy/pasted to give you a sample app running on your machine

You'll need to first "create table visit (date date)" in your postgres database

```javascript
var http = require('http');
var pg = require('pg');

var conString = "postgres://postgres:1234@localhost/postgres";

var server = http.createServer(conString, function(req, res, next) {
  pg.connect(function(err, client, done) {
    var handleError = function(err) {
      if(!err) return false;
      done(client);
      next(err);
      return true;
    };
    client.query('INSERT INTO visit (date) VALUES ($1)', [new Date()], function(err, result) {
      if(handleError(err)) return;
      client.query('SELECT COUNT(date) AS count FROM visit', function(err, result) {
        if(handleError(err)) return;
        done();
        res.writeHead(200, {'content-type': 'text/html'});
        res.end('You are visitor number ' + result.rows[0].count);
      });
    });
  });
})

server.listen(3001)
```