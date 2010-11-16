### live example

This app is running live [[right here|http://explodemy.com]]

```javascript
var Client = require('pg').Client;

var client = new Client({
  user: 'postgres',
  database: 'postgres',
  password: '*******'
});

var express = require('express');
var app = express.createServer();

app.get('/', ?(req, res) {

  var hitcount= client.query({
    name: 'hitcount',
    text: 'SELECT COUNT(date) as count FROM visit'
  });

  var when = client.query({
    name: 'last',
    text: 'SELECT date FROM visit ORDER BY date DESC LIMIT 1'
  });

  //because query execution is ordered we can handle row events in                                                                                                               
  //order with confidence                                                                                                                                                        
  hitcount.on('row', ?(countRow) {
    when.on('row', ?(whenRow) {            
      var lines = ["hello world, I've been hit " + countRow.count + " times, most recently at " + whenRow.date,
                  '<a href="http://github.com/brianc/node-postgres/wiki/Example">you can view the source on github</a>']
      res.send(lines.join('<br />'));
    });
  });

  //this query is executed in the background  after the response has                                                                                                             
  //been sent to the user. we don't actually care about the values                                                                                                               
  //returned so there's no need to handle events                                                                                                                                 
  client.query({
    name: 'update visits',
    text:"insert into visit(date) VALUES(NOW())"
  });
});

app.listen(3000, ?() {
  //this error handling is not very good
  //I'm working on the api
  client.on('error', ?(err) {
    console.log(err);
  });
  client.connect();
});
```