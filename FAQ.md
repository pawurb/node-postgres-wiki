Thanks to [g40](https://github.com/g40)

### 1. How do I know what values are on the row object? ###

The row object has properties which align to the column names returned from the query.

Given a table users with columns 'name' and 'age' doing `select * from users` would return you a result object with an array of row objects.  Each row object would have the properties `name` and `age`.  Example:

```js
client.query('SELECT * FROM users', function(err, result) {
  console.log('name: %s and age: %d', result.rows[0].name, result.rows[0].age);
  //since the row object is just a hash, it can be accessed also as follows
  console.log('name: %s and age: %d', result.rows[0]['name'], result.rows[0]['age']);
});
```

### 2. Can I iterate across the columns in the recordset to dynamically display column names? ###

Why, yes.  Yes you can.

```js
client.query(..., function(err, result) { 
  var firstRow = result.rows[0];
  for(var columnName in firstRow) {
    console.log('column "%s" has a value of "%j"', columnName, firstRow[columnName]);
  }
});
```

### 3. Assuming a recordset is enumerated using the array accessor style used in 1, can we get the column names in the same fashion, i.e. is there a result.rows[i].columnName property? ###

This is possible using the `result.fields` array:
```js
client.query(..., function(err, result) {
  console.log("Returned columns:", result.fields.map(function(f) { return f.name; }).join(', '));
});
```

### 4. How do you get the count of columns in the result set ? ###

```js
client.query(..., function(err, result) {
  var columnCount = Object.keys(result.rows[0]).length;
});
```

This may also be accomplished using the `result.fields` array:
```js
client.query(..., function(err, result) {
  var columnCount = result.fields.length;
});
```

### 5. If pg returns query data in JSON format, for web service applications, it would make sense to return that directly to the client. If this assumption is correct what is the most efficient method? ###

```js
http.createServer(function(req, res) {
  //NOTE: pg connection boilerplate not present
  pg.query(..., function(err, result) {
    //NOTE: error handling not present
    var json = JSON.stringify(result.rows);
    res.writeHead(200, {'content-type':'application/json', 'content-length':Buffer.byteLength(json)}); 
    res.end(json);
  });
})
```
### 6. How do I use the Client instance directly? ###

Example code:

```js
        var client = new Client(connectionString);
        client.connect();
        // now enumerate ...
        enumerate(client, path, callback);
        //
        client.end();
```
This fails with:

```js
   var client = new Client(connectionString);
           ^
   ReferenceError: Client is not defined
```
When you import the postgres library you commonly do `require('pg')`.  This works and requires the 'root' of the library with various properties hanging off of it.  To directly instantiate a specific client instance instead of using the pool you can access the client constructor off the the imported pg object.

1.  `var Client = require('pg').Client;`

or for the native client

2.  `var Client = require('pg').native.Client;`

Thank you Brian. pg is excellent.


### 7. I just have a question and maybe a feature request that i am not able to think about how to implement or do it: i need to retrieve the inserted row or someway to reach it after the insert is done.

Yeah, you can do this as so: 

```js
//let's pretend we have a user table with the 'id' as the auto-incrementing primary key
var queryText = 'INSERT INTO users(password_hash, email) VALUES($1, $2) RETURNING id'
client.query(queryText, ['841l14yah', 'test@te.st'], function(err, result) {
  if(err) //handle error
  else {
    var newlyCreatedUserId = result.rows[0].id;
  }
});
```

### 8. Does node-postgres handle SQL injection?

Absolutely! The parameterized query support in node-postgres is first class. All escaping is done by the postgresql server ensuring proper behavior across dialects, encodings, etc...  For example, this will not inject sql:

```js
client.query("INSERT INTO user(name) VALUES($1)", ["'; DROP TABLE user;"], function (err, result) {
  // ...
});
```


### 9. Can I create a named prepared statement for use later on without performing a query? If not, does passing the same text again to a named statement get ignored and the cached version used? I don't want to have two codepaths in a function, one for first-use and one for every other. 

If a prepared statement has a `name`, it is only parsed once.  After that, `name` will re-use the prepared statement regardless of what `text` is.

### 10. Can we override the built in data converters between javascript and postgres data types? 

Yes, [here is a test that shows how it can be done.](https://github.com/brianc/node-postgres/blob/master/test/integration/client/huge-numeric-tests.js#L6) And for some examples of already registered converters, take a look at [the node-pg-types project](https://github.com/brianc/node-pg-types).


### 11. How do I build a `WHERE foo IN (...)` query to find rows matching an array of values?

node-postgres supports mapping simple JavaScript arrays to PostgreSQL arrays, so in most cases you can just pass it like any other parameter.

```javascript
client.query("SELECT * FROM stooges WHERE name = ANY ($1)", [ ['larry', 'curly', 'moe'] ], ...);
```

Note that `= ANY` is another way to write `IN (...)`, but unlike `IN (...)` it will work how you'd expect when you pass an array as a query parameter.

If you know the length of the array in advance you can flatten it to an `IN` list:

```
// passing a flat array of values will work:
client.query("SELECT * FROM stooges WHERE name IN ($1, $2, $3)", ['larry', 'curly', 'moe'], ...);
```

... but there's little benefit when `= ANY` works with a JavaScript array.

If you're on an old version of node-postgres or you need to create more complex PostgreSQL arrays (arrays of composite types, etc) that node-postgres isn't coping with, you can generate an array literal with dynamic SQL, but *be extremely careful of [SQL injection](https://en.wikipedia.org/wiki/SQL_injection) when doing this*. The following approach is safe because it generates a query string with query parameters and a flattened parameter list, so you're still using the driver's support for parameterised queries ("prepared statements") to protect against SQL injection:

```javascript
var stooge_names = ['larry', 'curly', 'moe'];
var offset = 1;
var placeholders = stooge_names.map(function(name,i) { 
    return '$'+(i+offset); 
}).join(',');
client.query("SELECT * FROM stooges WHERE name IN ("+placeholders+")", stooge_names, ...);
```

If you have other values and placeholders in your query you'll need to use a different `offset` value for the array placeholders. See [#129](https://github.com/brianc/node-postgres/issues/129) and [#82](https://github.com/brianc/node-postgres/issues/82) for extra discussion.

### 12. Why does node-postgres come with two bindings? One in Javascript and one "native" that uses libpq? Which one is fastest and why isn't a single binding enough?

node-postgres comes with two bindings because I wrote it back before the idea of "do one tiny thing in each module" was a popular idea.  I initially wrote the pure-javascript bindings.  People were complaining about adopting them because it wasn't a C binding so it wasn't fast.  To answer their critique I wrote libpq bindings.  I placed them in the same module because I could reuse 70% of the tests (all of the integration tests) so I could quickly know when the APIs diverged.

_note: sometime after v1.0 I plan on splitting the javascript, native, and integration tests into their own modules.  the node-postgres module itself will be a sort of 'meta package' for the other modules_

Last time I checked the native bindings were faster than the pure JavaScript bindings; however, there are performance gains still available to both through code refactors and this can/will change.  Either binding you use is fast enough to not end up being a significant factor in your application.  As for why isn't a single binding enough? A single binding is enough - __either one__ :wink:.  

Personally, I like the pure JavaScript bindings because it's JavaScript all the way down, but they both work equally and have full feature parity due to the extensive overlapping test suite.

### 13. What happens to open transactions when `pg.connect`'s `done` is called?

Nothing.  You are responsible for calling either `client.query('COMMIT')` or `client.query('ROLLBACK')`  If you call neither and call the `done()` callback the client will be returned to the pool with an open transaction, and I assume _bad things will happen_ in your application.

### 14. How do I install pg on Windows?

Problem: `npm install pg` fails with error message `Call to 'pg_config --libdir' returned exit status 1. while trying to load binding.gyp`

You need PostgreSQL installed on your system. The path to PostgreSQL bin directory must be included in the environment PATH variable. `pg_config` is stored in that bin directory.

Quick fix for PowerShell:

`$env:PATH+=";C:\Program Files\PostgreSQL\9.2\bin"`

`npm install pg`

### 15. (New Question) How can a quickly get a Client from Client pool?

pg.connect(): It takes time to reconnect ?

### 16. (New Question) Are queries asynchronous, or do they block? Can this behavior be overridden if desired?

### 17. What happens if I ask for a connection and the pool is already empty? will it throw an error or wait until a connection becomes available?

It will wait, and call your callback with a connection after one becomes available. This package uses the [generic-pool](https://github.com/coopernurse/node-pool) package to provide this behavior.

### 18. (New Question) Is there a way to check if I have an active connection?

### 19. (New Question) Do I correctly assume that calling done() or done(err) more than once (multiple times) has no side-effects?

### 20. I obtain `Promise { <pending> }` from a query
 
See https://github.com/brianc/node-postgres/issues/1130. The query returns a promise because of its asynchronous design. You should use a callback, the `then` method or await. Callback example:
 
 ```{js}
 pool.query('SELECT $1::int AS number', ['2'], function(err, res) {
   if(err) {
     return console.error('error running query', err);
   }
   
   console.log('number:', res.rows[0].number);
 });
 ```

***
[[◄ Back (Transactions)|Transactions]] `      ` [[Next (Example App) ►|Example]]