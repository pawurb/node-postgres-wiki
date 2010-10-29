Basically a facade on top of the connection to provide a _much_ more
user friendly, "node style" interface for doing all the lovely things
you like with PostgreSQL.

Now that I've got the __Connection__ api in place, the bulk and meat of
the work is being done on the __Client__ to provide the best possible
API.  Help? Yes please!

    var client = new Client({
      user: 'brian',
      database: 'postgres',
    });

    client.query("create temp table ids(id integer)");
    client.query("insert into ids(id) values(1)");
    client.query("insert into ids(id) values(2)");
    var query = client.query("select * from ids", function(row) {
      row.fields[0] // <- that equals 1 the first time. 2 the second time.
    });
    query.on('end', function() {
      client.end();
    });    