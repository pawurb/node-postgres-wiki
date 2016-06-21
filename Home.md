* [[Installation]]
* API
    * [[pg]]
    * pg.[[Client]]
    * pg.[[Query]]
    * pg.[[Connection]]
* [[Prepared Statements]]
* [[Transactions]]
* [[FAQ]]
* [[Example App|Example]]
* [[Error Handling]]
* Internals
  * [[Query Queue|Queryqueue]]
* [[Extras]] – List of some handy modules
* [[Testing]]
* [[Todo]]

## tl; dr -

I've written many apps with node-postgres in production.  This is pretty much the smallest example I can come up with:

```js

var Pool = require('pg')
var pool = new Pool()

http.createServer(function(req, res) {
  pool.query('SELECT $1::text as name', ['brianc'], function(err, res) {
    res.writeHead({'content-type': 'text/html'})
    res.end('Hello from: ' + res.rows[0].name)
  })
})
```

The API of node-postgres supports a __lot__ of additional stuff if you dive in, but if you're just getting started I recommend just doing that above & only introducing more advanced stuff as needed.  node-postgres is designed to be very low level and has a lot of knobs to fiddle with, but 99% of the time I just wanna run some queries!  

There's also a really large ecosystem of [additional modules](https://github.com/brianc/node-postgres/wiki/Extras) - highly recommend checking some of those out for more advanced things or when building larger applications.