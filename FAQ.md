Thanks to [g40](https://github.com/g40)

### 1. how do I know what values are on the row object ? ###

The row object has properties which align to the column names returned from the query.

Given a table users with columns 'name' and 'age' doing `select * from users` would return you a result object with an array of row objects.  Each row object would have the properties `name` and `age`.  Example:

```js
client.query('SELECT * FROM users', function(err, result) {
  console.log('name: %s and age: %d', result.rows[0].name, result.rows[0].age);
  //since the row object is just a hash, it can be accessed also as follows
  console.log('name: %s and age: %d', result.rows[0]['name'], result.rows[0]['age']);
});
```

### 2. can I iterate across the columns in the recordset to dynamically display column names ? ###

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

Not currently.  It would be helpful to access the column values by column name or index position, but it's not part of the node-postgres api for now.

### 4. How do you get the count of columns in the result set ? ###

```js
pg.query(..., function(err, result) {
  var columnCount = Object.keys(result.rows[0]).length;
});
```

### 5. If pg returns query data in JSON format, for web service applications, it would make sense to return that directly to the client. If this assumption is correct what is the most efficient method? ###

```js
http.createServer(function(req, res) {
  //NOTE: pg connection boilerplate not present
  pg.query(..., function(err, result) {
    //NOTE: error handling not present
    var json = JSON.stringify(result.rows);
    res.writeHead(200, {'content-type':'application/json', 'content-length':json.length}); 
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
client.query('INSERT INTO users(password_hash, email) VALUES($1, $2) RETURNING id', function(err, result) {
  if(err) //handle error
  else {
    var newlyCreatedUserId = result.rows[0].id;
  }
});
```

Or using the evented approach:

```js
client.query('INSERT INTO users(password_hash, email) VALUES($1, $2) RETURNING id')
  .on('row', function (row) {
    var newlyCreatedUserId = row.id;
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

Example:

You could use the same prepared statement to find a top-level parent in a hierarchy:

```js
var preparedStatement = {
  text: 'SELECT parent FROM foo WHERE id=$1',
  name: 'my prepared statement'
};
function getAncestor(id) {
  client.query(preparedStatement, [id], function (err, result) {
    if (result.rows[0].parent != null) {
      getAncestor(result.rows[0].parent);
    } else {
      console.log('The ancestor is ' + id);
    }
  });
}
getAncestor(childId);
```

This will find the parent, and then the parent-of-the-parent, and so on until parent is null.  Since the same `name` is passed each time, the prepared statement is only parsed once.

(Note that this could have been done in SQL more efficiently.  This example is to illustrate only.)