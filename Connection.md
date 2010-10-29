The connection object is a 1 to 1 mapping to the [postgres client/server messaging protocol](http://developer.postgresql.org/pgdocs/postgres/protocol.html). The _Connection_ object is mostly used by the Client object (which...I haven't yet finished implementing) but you can do anything you want with PostgreSQL using the connection object if you're really into that.  I studied the protocol for a while implementing this and the documentation is pretty solid.  If you're already familiar you should be right at home.  Have fun looking up the [oids for  the datatypes in your bound queries](http://github.com/brianc/node-postgres/blob/master/script/list-db-types.js)

There are a few minor variations from the protocol:

- The connection only supports 'text' mode right now.
- Renamed 'passwordMessage' to 'password'
- Renamed 'startupMessage' to 'startup'
- Renamed 'errorResposne' to 'error'
- Renamed 'noticeResponce' to 'notice'

The reason for the renamings is 90% of the message names in the protocol do no contain "message" "request" "response" or anything similar, and I feel  it's a bit redundant to send a "passwordMessage message."  But then again...[I do say ATM machine](http://en.wikipedia.org/wiki/RAS_syndrome).

Anyways...using a connection directly is a pretty verbose and cumbersom affair.  Here's an example of executing a prepared query using the directly __Connection__ api in compliance with PostgreSQL.
 
_note: this works and is taken directly from an integration test; however, it doesn't include error handling_

    var con = new Connection({stream: new net.Stream()});

    con.connect('5432','localhost');

    con.once('connect', function() {

      con.startup({
        user: username,
        database: database
      });

      con.once('readyForQuery', function() {

        con.query('create temp table ids(id integer)');

        con.once('readyForQuery', function() {

          con.query('insert into ids(id) values(1); insert into ids(id) values(2);');

          con.once('readyForQuery', function() {

            con.parse({
              text: 'select * from ids'
            });
            con.flush();

            con.once('parseComplete', function() {
              con.bind();
              con.flush();
            });

            con.once('bindComplete', function() {
              con.execute();
              con.flush();
            });

            con.once('commandComplete', function() {
              con.sync();
            });

            con.once('readyForQuery', function() {
              con.end();
            });
          });
        });
      });
    });