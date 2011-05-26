Thanks to [g40](https://github.com/g40)

### 1. how do I know what values are on the row object ? ###

The row object has properties which align to the column names returned from the query.

Given a table users with columns 'name' and 'age' doing `select * from users` would return you a result object with an array of row objects.  Each row object would have the properties `name` and `age`.  Example:

```js
client.query('SELECT * FROM users`, function(err, result) {
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

### 4. (Alert! Possibly an idiot question) How do you get the count of columns in the result set ? ###

### 5. (Alert! Possibly an idiot question) If pg returns query data in JSON format, for web serivce applications, it would make sense to return that directly to the client. If this assumption is correct what is the most efficient method? ###


Thank you Brian. pg is excellent.