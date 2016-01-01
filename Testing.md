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

# Internal Testing of node-postgres
In case you want to contribute....

## Unit tests

Unit tests do not depend on having access to a running PostgreSQL server.  They run in memory by stubbing out the layer below the object under test.  If the [[Client]] object is being tests, the lower level [[Connection]] object is stubbed out and controlled within the test.  This allows to simulate specifically timed error events and more complex interactions between the [[Client]] and the [[Connection]] such as when executing a [[Prepared Statement|Prepared-Statements]].  All unit test files exist under `test/unit/` and by convention end with `-tests.js`.  They can be executed either by `make test`, `make test-unit`, or `npm test pg`

## Integration tests

The integration tests operate on an actual PostgreSQL server instance.  You can supply credentials via a quick and dirty command line option parser which is executed before any test is run.  Also, the integration tests require populating your database with some test data.  The script is located at `node-postgres/script/create-test-tables.js`.  All integration tests files exist under `test/integration/` and by convention end with `-tests.js`.  Further instructions below...

## Running tests

You can run any test file directly by doing the `node test/unit/connection/inbound-parser-tests.js` or something of the like.  

However, you can specify command line arguments after the file and they will be picked up and used in the tests.  None of the arguments are used in  _unit_ tests since they have no external dependencies, so you're safe to just blast away with the command like above, but if you'd like to execute an _integration_ test, you should specify the database and user to use for testing, and optionally a password.

To do so you would do something like so:

    node test/integration/client/simple-query-tests.js pg://user:password@host:port/databaseName

If you'd like to execute all the unit or integration tests at one time, you can do so with the `Makefile`.

## Makefile
The make file is used as a shortcut to running all the tests.  Since many of the integration tests slam the database with load/connection-pool based testing, the tests are executed sequentially file-by-file.  This ensures each file finishes before the next begins, reduces test complexity, and raises test isolation.  The command line parameters passed to individual test files can be passed to the Makefile which will in turn pass them along to each test file.

### Run all unit tests

    make test-unit

or optionally, since `test-unit` is the default:

_(Note: This means integration tests under `test/integration` do **not** run with the default Makefile task)_

    make test

### Run all integration tests

    make test-integration connectionString=pg://user:password@host:port/databaseName

### Run all the tests! (unit, integration, native, and binary)

    make test-all connectionString=pg://user:password@host:port/databaseName

In short, I tried to make executing the tests as easy as possible. Hopefully this will encourage you to fork, hack, and do whatever you please as you've got a nice, big safety net under you.

### Test data

In order for the integration tests to not take ages to run, I've pulled out the script used to generate test data.  This way you can generate a "test" database once and don't have to up/down the tables every time an integration test runs.  To run the generation script, execute the script with the same command ine arguments passed to any other test script.

    node script/create-test-tables.js pg://user:password@host:port/databaseName

***
[[◄ Back (Query Queue)|Queryqueue]] `      ` [[Next (Todo) ►|Todo]]