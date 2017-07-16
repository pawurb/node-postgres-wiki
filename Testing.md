# Testing your own apps built using node-postgres

When you do integration testing within your own apps you'll eventually need to close your node-postgres clients otherwise the test process will never exit.  There are various strategies I've employed throughout the node-postgres test code to accomplish this.  While they all work they are not all very pretty due to having to bootstrap the testing & disconnecting of node-postgres while testing it. (chicken v. egg)  So here's what I recommend.

If you're using the __pg__ object to manage a pool of clients you can call `pg.end()` whenever you're finished testing.  This is currently a brute-force way to shut down your clients and may potentially kill any queries which have not finished running during your tests (_note: I plan on looking at better ways to terminate the client pool in the future_).  Internally it loops through all the clients and basically forces their sockets to disconnect.  Your mileage may vary with this approach.  

Another option is to set the client pool size to 1.  This ensures the __pg__ object will always return the same __client__ object whenever `pg.connect()` is called.  This way you can do the following in your 'cleanup' or 'teardown' area:

```javascript
    tearDown: function() {
      var connectionString = /*whatever connection you use for testing*/
      pg.connect(connectionString, function(err, client) { client.end() });
    }
```

If you're manually managing your own __Client__ instances without using __pg__ then you are free to end them whenever you want.  Generally I do the following:

```javascript
    var client = new Client(/*connectionInfo*/);
    client.query('bla bla');
    client.on('drain', client.end.bind(client)); //auto disconnect client after last query ends
```

_Note: as mentioned other places...do not create a new client instance for each http request you receive. You **will** exhaust available connections to your PostgreSQL server and you **will** be sorry._ 