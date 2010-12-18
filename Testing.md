## Unit tests

Unit tests do not depend on having access to a running PostgreSQL server.  They run in memory by stubbing out the layer below the object under test.  If the [[Client]] object is being tests, the lower level [[Connection]] object is stubbed out and controlled within the test.  This allows to simulate specifically timed error events and more complex interactions between the [[Client]] and the [[Connection]] such as when executing a [[Prepared Statement|Prepared-Statements]].  All unit test files exist under `test/unit/` and by convention end with "-tests.js."  

## Integration tests

The integration tests operate on an actual PostgreSQL server instance.  You can supply credentials via a quick and dirty command line option parser which is executed before any test is run.  Also, the integration tests require populating your database with some test data.  The script is located at `node-postgres/script/create-test-tables.js.`  All integration tests files exist under `test/integration/` and by convention end with "-tests.js."  Further instructions below...

## Running tests

You can run any test file directly by doing the `node test/unit/connection/inbound-parser-tests.js` or something of the like.  

However, you can specify command line arguments after the file and they will be picked up and used in the tests.  None of the arguments are used in  _unit_ tests since they have no external dependencies, so you're safe to just blast away with the command like above, but if you'd like to execute an _integration_ test, you outta specifiy your database, user to use for testing, and optionally a password.

To do so you would do something like so:

    node test/integration/client/simple-query-tests.js -u brian -d test_db 

If you'd like to execute all the unit or integration tests at one time, you can do so with the "Makefile"

## Makefile
the make file is used as a shortcut to running all the tests.  Since many of the integration tests slam the database with load/connection-pool based testing the tests are executed sequentially file-by-file.  This ensures each file finishes before the next begins and reduces test complexity and raises test isolation.  The command line parameters passed to individual test files can be passed to the Makefile which will in turn pass them along to each test file.

### Run all unit tests

    make test-unit

or optionally, since `test-unit` is the default

_note this means integration tests under `test/integration` do __not__ run with from default makefile task_

    make test

### Run all integration tests

    make test-integration user=brianc password=1234 database=home host=somehost.com port=123

### Run all the tests! (unit & integration)

    make test-all user=brianc password=1234 database=home host=somehost.com port=123

In short, I tried to make executing the tests as easy as possible. Hopefully this will encourage you to fork, hack, and do whatever you please as you've got a nice, big safety net under you.

### Test data

In order for the integration tests to not take ages to run, I've pulled out the script used to generate test data.  This way you can generate a "test" database once and don't have to up/down the tables every time an integration test runs.  To run the generation script, execute the script with the same command ine arguments passed to any other test script.

    node script/create-test-tables.js -u user -d database

Aditionally if you want to revert the test data, you'll need to "down" the database first and then re-create the data as follows:

    node script/create-test-tables.js -u user -d database --down
    node script/create-test-tables.js -u user -d database

### Command line arguments

Only used during testing, so the implementation is pretty much _crap_ but it works.

    -t, --test [unit <default>, integration, all]
        only used when executing `node test/run.js`
        the type of tests to run. 
        "unit" is the default and runs all unit tests. 
        "integration" runs integration tests
        "all" runs both unit and integration tests

    -u, --user 
        default is 'postgres'
        database user name used to connect to server in integration tests

    --password
        default is ''
        password for the user

    -d, --database
        default is 'postgres'
        database to use during testing

    -p, --port
        default is 5432
        database port

    -h, --host
        default is 'localhost'
        host of database

    --down 
        only applies when running the script to create test data.  
        Will run the 'down' migration, bringing the database back to before script ran