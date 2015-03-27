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

In practice prepared statements are usually used in sets; for example, posting an invoice might use 
prepared statements to reset inventory levels, customer amounts due, and salesman totals.  Adding and
deallocating them as a set makes sense.

Practical steps might be to:

- query for one of the prepared statements using `pg_prepared_statements`
- if not existent, create the set of prepared statements
- do work
- if the set will probably not be used again soon, `deallocate` them; otherwise, let them stay

Alternately, if one wishes to optimize performance, one could:

- keeps prepared statements in sets, create and deallocate each of them in a set
