node-postgres is by design pretty light on abstractions.  These are some handy modules the community’s been using over the years to complete the picture:


### Components

Standalone PostgreSQL packages.

- [brianc/node-pg-native](https://github.com/brianc/node-pg-native) - Simple interface abstraction on top of [libpq](https://github.com/brianc/node-libpq).
- [brianc/node-pg-types](https://github.com/brianc/node-pg-types) - Parsing for PostgreSQL serialized formats.
- [datalanche/node-pg-format](https://github.com/datalanche/node-pg-format) - Safely and easily create dynamic SQL queries with this Node implementation of [PostgreSQL format()](https://www.postgresql.org/docs/current/functions-string.html#FUNCTIONS-STRING-FORMAT).
- [LinusU/pg-error-constants](https://github.com/LinusU/pg-error-constants) - Error constants for more robust query error handling.
- [vitaly-t/connection-string](https://github.com/vitaly-t/connection-string) - Alternative connection string parser implementing superset of MongoDB connection string spec.


### Extensions

- [brianc/node-pg-cursor](https://github.com/brianc/node-pg-cursor) - Query cursor extension for node-postgres.
- [brianc/node-pg-copy-streams](https://github.com/brianc/node-pg-copy-streams) - `COPY FROM`/`COPY TO` for node-postgres. Stream from one database to another, and stuff.
- [brianc/node-pg-query-stream](https://github.com/brianc/node-pg-query-stream) - Query results from node-postgres as a readable (object) stream.
- [emilbayes/pg-ipc](https://github.com/emilbayes/pg-ipc) - IPC over PostgreSQL `LISTEN`/`NOTIFY`/`UNLISTEN` exposed as an `EventEmitter`.
- [kibae/pg-logical-replication](https://github.com/kibae/pg-logical-replication) - PostgreSQL logical replication client.
- [recursivefunk/pg-gen](https://github.com/recursivefunk/pg-gen) - Paginate through large result sets with cursors.
- [postgres-extras](https://github.com/pawurb/node-postgres-extras) - PostgreSQL database performance insights.


### API wrappers

- [holdfenytolvaj/pogi](https://github.com/holdfenytolvaj/pogi)
- [joeandaverde/tinypg](https://github.com/joeandaverde/tinypg) - Simpler interface, named parameter support, queries from files, transaction management, events and hooks.
- [langpavel/node-pg-async](https://github.com/langpavel/node-pg-async)
- [sehrope/node-pg-db](https://github.com/sehrope/node-pg-db) - Simpler interface, named parameter support, transaction management and event hooks.
- [Suor/pg-bricks](https://github.com/Suor/pg-bricks) - A higher-level wrapper around node-postgres to handle connection settings, SQL generation, transactions and ease data access.
- [vitaly-t/pg-promise](https://github.com/vitaly-t/pg-promise)

[RxJS](https://github.com/ReactiveX/rxjs):

- [haoliangyu/pg-reactive](https://github.com/haoliangyu/pg-reactive)
- [jadbox/pg-rxjs](https://github.com/jadbox/pg-rxjs)

Transactions:

- [iceddev/pg-transact](https://github.com/iceddev/pg-transact)


### Database interfaces

- [grncdr/node-any-db](https://github.com/grncdr/node-any-db) - Thin and less-opinionated database abstraction layer for Node.
- [MassiveJS](https://massivejs.org/) - A simple relational data access tool that has full JSONB document support for PostgreSQL.


### SQL template tags

- [131/sql-template](https://github.com/131/sql-template)
- [felixfbecker/node-sql-template-strings](https://github.com/felixfbecker/node-sql-template-strings) - Supports multiple database drivers and named prepared statements.
- [sequencework/sql](https://github.com/sequencework/sql) - Tag with convenient querying functions.
- [XeCycle/pg-template-tag](https://github.com/XeCycle/pg-template-tag)


### Query builders

- [brianc/node-sql](https://github.com/brianc/node-sql)
- [CSNW/sql-bricks](https://github.com/CSNW/sql-bricks)
- [hiddentao/squel](https://hiddentao.github.io/squel/)


### Other

- [acarl/pg-restify](https://github.com/acarl/pg-restify) - Creates a generic REST API for a PostgreSQL database using restify.
- [archfirst/joinjs](https://github.com/archfirst/joinjs) - A simple library to map the results of complex joins to nested JavaScript objects (alternative to full-blown ORMs).
- [brandon-d-mckay/krauter](https://github.com/brandon-d-mckay/krauter) - An Express router that lets queries act as request handlers.
- [nuodata/nuodata-db-api](https://github.com/nuodata/nuodata-db-api) - REST API for a PostgreSQL database.
- [ozum/pg-generator](https://www.pg-generator.com/) - Template-based scaffolding for PostgreSQL. Command line utility which generates files for each table and schema of a PostgreSQL database.
- [ozum/pg-structure](https://www.pg-structure.com/) - Node library to get structure of a PostgreSQL database automatically as a detailed object.
- [vitaly-t/pg-minify](https://github.com/vitaly-t/pg-minify) - Minifies PostgreSQL scripts.