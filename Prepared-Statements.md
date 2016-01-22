`node-postgres` provides three means of submitting a query:

- text, e.g. `query( "select name from emp where emp_id=123" )`
- paramaterized, e.g. `query( "select name from emp where emp_id=$1", [123] )`
- prepared, e.g. `query( {name:"emp_name", text:"select name from emp where emp_id=$1", values:[123]} )`

##Text Queries##

Plain text queries, for a single query instance, are faster and the most flexible.  They are also the
most insecure, providing no barrier to SQL injection attacks.

##Parameterized Queries##

A parameterized query allows you "pass arguments" to a query, providing a barrier to SQL injection attacks.

Parameters may not be DDL:

- `select name from emp where emp_id=$1` – legal
- `select $1 from emp where emp_id=$2` – illegal – column cannot be parameter
- `select name from $1 where emp_id=$2` – illegal – table cannot be parameter
- `select name from $1.emp where emp_id=$2` – illegal – schema cannot be parameter

Parameterized queries in postgres are planned for before each execution, so they provide safety
but not speed.

With tagged template string literals introduced in ECMAScript 6, parameterized queries can be written more easily with a simple tag function:

```javascript
function SQL(parts, ...values) {
  return {
    text: parts.reduce((prev, curr, i) => prev+"$"+i+curr),
    values
  };
}

client.query(SQL`select name from user where id=${userId} and password=${password}`, callback);
```

If you want it as a module, take a look at [sql-template-strings](https://www.npmjs.com/package/sql-template-strings).

##Prepared Statements##

A prepared statement lets postgres plan the statement so that each execution takes advantage of the
plan.  If you have statements that are executed repeatedly, then prepared statements are the fastest.
Prepared statements also provide a barrier to SQL injection attacks.

In postgres, the prepared statement is attached to the `session`.  This means that if you connect, prepare
a statement, and reconnect, the prepared statement will only be available if you are using the same
postgres session.  (Currently in node-postgres, if a new prepared statement is issued with the same name as an
existent prepared statement on the same client, the existing prepared statement will be used.  It is currently not possible to replace one named statement with another – we’re going to fix this: https://github.com/brianc/node-postgres/issues/345)

The following will help track:

- `select pg_backend_pid() as pid` – the postgres session
- `select name from pg_prepared_statements` – names of prepared statements in the session
- `deallocate 'emp_name'` – removes a specific prepared statement from the session
- `deallocate all` – removes all prepared statements from the session

Accumulated prepared statements can build cruft if not deallocated, but premature deallocation forces
postgres to replan the statement if reissued, losing the speed benefits.

###Using Prepared Statements###

pg tracks which statements have been prepared for a session - you do not need one query to prepare
and a second one to execute: the same query, if passed name, text and values, can be used each time
you wish to execute the query - pg will prepare the query first only if it needs to.

####`values` are required
It is not possible to simply create a prepared statement without executing it. e.g. the following will throw an error:
```javascript
client.query( {
    text :  "INSERT INTO my_table(field1,field2) VALUES ($1, $2)", 
    name : "insert-my-data"
});
```
The error is
> error: bind message supplies 0 parameters, but prepared statement "insert-my-data" requires 2

Of course, `values` are not required if there are no parameters in the query. But even in those instances, the query is executed when `client.query()` is invoked.

####Re-using prepared statements
Since caching is per-session, sometimes it is not possible to be sure that a given prepared statement exists in the cache. If all calls are made within the same `pg.connect` block, the statement will be cached because all calls use the same pg session.

However, in practical use (e.g. with Express), you could have a pool of pg connections being used by middleware serving hundreds of HTTP requests. The fail-safe way to use prepared statements is to always supply both the `text` and the `name`. When a statement named `name` is found to be cached, `text` is ignored and the cached statement is used. If `name` is not found in the cache, a new prepared statement is created using the `text` supplied. (as discussed in [this issue](https://github.com/brianc/node-postgres/issues/903))

***
[[◄ Back (API - pg.Connection)|Connection]] `      ` [[Next (Transactions) ►|Transactions]]