
### connect to self signed Postgresql server

Use following config to connect a Postgresql Server self signed:

```
var config = {
    database : 'database-name', //env var: PGDATABASE
    host     : "host-or-ip",    //env var: PGPORT
    port     : 5432,            //env var: PGPORT
    max      : 100,             // max number of clients in the pool
    idleTimeoutMillis: 30000,
    ssl : {
        rejectUnauthorized : false,
        ca   : fs.readFileSync("/path/to/server-certificates/maybe/root.crt").toString(),
        key  : fs.readFileSync("/path/to/client-key/maybe/postgresql.key").toString(),
        cert : fs.readFileSync("/path/to/client-certificates/maybe/postgresql.crt").toString(),
    }
};

```

For more information about `config.ssl` check [TLS (SSL) of nodejs](https://nodejs.org/dist/latest-v4.x/docs/api/tls.html)