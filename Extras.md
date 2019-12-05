node-postgres is by design pretty light on abstractions.  These are some handy modules we've been using over the years to complete the picture:


### Components

Standalone PostgreSQL packages.

- [brianc/node-pg-native](https://github.com/brianc/node-pg-native) - Simple interface abstraction on top of [libpq](https://github.com/brianc/node-libpq)
- [brianc/node-pg-types](https://github.com/brianc/node-pg-types) - Type parsing for node-postgres
- [LinusU/pg-error-constants](https://github.com/LinusU/pg-error-constants) - error constants for more robust query error handling.


### Extensions

- [brianc/node-pg-cursor](https://github.com/brianc/node-pg-cursor) - Query cursor extension for node-postgres
- [brianc/node-pg-copy-streams](https://github.com/brianc/node-pg-copy-streams) - COPY FROM / COPY TO for node-postgres. Stream from one database to another, and stuff.
- [brianc/node-pg-query-stream](https://github.com/brianc/node-pg-query-stream) - Query results from node-postgres as a readable (object) stream
- [emilbayes/pg-ipc](https://github.com/emilbayes/pg-ipc) - IPC over PostgreSQL `LISTEN`/`NOTIFY`/`UNLISTEN` exposed as an `EventEmitter`
- [kibae/pg-logical-replication](https://github.com/kibae/pg-logical-replication) - PostgreSQL Location Replication client - logical WAL replication streaming
- [numminorihsf/pg-ka-fix](https://github.com/numminorihsf/pg-ka-fix) - TCP keep-alive extension for node-postgres.
- [recursivefunk/pg-gen](https://github.com/recursivefunk/pg-gen) - Use ES6 Generators to paginate through large Postgres result sets


### API wrappers

- [coderhaoxin/pg-then](https://github.com/coderhaoxin/pg-then) A tiny wrapper of `pg` for promise api.
- [grncdr/node-any-db](https://github.com/grncdr/node-any-db) - Thin and less-opinionated database abstraction layer for node.
- [haoliangyu/pg-reactive](https://github.com/haoliangyu/pg-reactive) - a lightweight [RxJS 5](https://github.com/ReactiveX/rxjs) wrapper for `node-postgres`
- [holdfenytolvaj/pogi](https://github.com/holdfenytolvaj/pogi) - convenient DbHandler over pg, just what you need :) (typescript,async,jsonb,you name it) .
- [iceddev/pg-transact](https://github.com/iceddev/pg-transact) - A nicer API on node-postgres transactions
- [jadbox/pg-rxjs](https://github.com/jadbox/pg-rxjs) Another tiny wrapper like `pg-then` but for [RxJS](https://github.com/Reactive-Extensions/RxJS)
- [kriasoft/node-pg-client](https://github.com/kriasoft/node-pg-client) - Promise-based wrapper for `node-postgres` designed for easy use with ES7 async/await.
- [langpavel/node-pg-async](https://github.com/langpavel/node-pg-async) - Tiny but powerful Promise based PostgreSQL client designed for easy use with ES7 async/await.
- [sehrope/node-pg-db](https://github.com/sehrope/node-pg-db) - Simpler interface, named parameter support, transaction management and event hooks.
- [Suor/pg-bricks](https://github.com/Suor/pg-bricks) - A higher level wrapper around node-postgres to handle connection settings, sql generation, transactions and ease data access.
- [vitaly-t/pg-promise](https://github.com/vitaly-t/pg-promise) - Use node-postgres via [Promises/A+](https://promisesaplus.com/).


### Database interfaces

- [MassiveJS](https://github.com/dmfay/massive-js) - A simple relational data access tool that has full JSONB document support for Postgres.


### SQL template tags

- [131/sql-template](https://github.com/131/sql-template) - pg compliant query builder (using ES6 template string)
- [sequencework/sql](https://github.com/sequencework/sql) - SQL template tag with convenient querying functions
- [XeCycle/pg-template-tag](https://github.com/XeCycle/pg-template-tag) - Write queries with ES6 tagged template literals, a "poor man's query builder".


### Query builders

- [brianc/node-sql](https://github.com/brianc/node-sql) - SQL generation for node.js
- [CSNW/sql-bricks](https://github.com/CSNW/sql-bricks) - Transparent, Schemaless SQL Generation
- [datalanche/node-pg-format](https://github.com/datalanche/node-pg-format) - Safely and easily create dynamic SQL queries with this Node implementation of [PostgreSQL format()](http://www.postgresql.org/docs/9.3/static/functions-string.html#FUNCTIONS-STRING-FORMAT).
- [hiddentao/squel](https://hiddentao.github.io/squel/) - SQL query string builder for Javascript


### Other

- [acarl/pg-restify](https://github.com/acarl/pg-restify) - Creates a generic REST API for a postgres database using restify.
- [archfirst/joinjs](https://github.com/archfirst/joinjs) - A simple library to map the results of complex joins to nested JavaScript objects (alternative to full-blown ORMs).
- [nuodata/nuodata-db-api](https://github.com/nuodata/nuodata-db-api) - REST API for a PostgreSQL database.
- [typed-typings/typed-pg](https://github.com/typed-typings/typed-pg) - pg type definition for [TypeScript](http://www.typescriptlang.org).
- [vitaly-t/pg-minify](https://github.com/vitaly-t/pg-minify) - Minifies PostgreSQL scripts.
- [pg-generator](http://www.pg-generator.com) - Template Based Scaffolding for PostgreSQL. Command line utility which generates files for each table and schema of a PostgreSQL database.
- [pg-structure](http://www.pg-structure.com) - Node.js library to get structure of a PostgreSQL database automatically as a detailed object.
