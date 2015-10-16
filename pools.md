node-postgres can maintain client pools internally.  They are generally accessed via [[pg#wiki-method-connect]] but can be directly accessed & manipulated via `pg.pools`.

## <em>hash/object</em> pg.pools.all

A hash of all existing client pools.  A pool's key is built by calling `JSON.stringify` on the first parameter passed to `pg.connect` unless the first parameter is a callback, in which case `JSON.stringify({})` is used.

####example

```js
var pg = require('pg');
console.log(Object.keys(pg.pools.all)); //[]

//connect using defaults or environment variables
//will create a pool by the name of "{}"
pg.connect(function(err, client, done) {
  done();
  console.log(Object.keys(pg.pools.all)); //["{}"]
});
```

## pg.pools.getOrCreate(<em>object</em> key)

Gets a pool by the given `key` or creates it if it does not yet exist.  In either case it returns a modified instance of https://github.com/coopernurse/node-pool pre-configured to return [[Client]] instances.

#### example

```js
var pg = require('pg');

pg.defaults.poolIdleTimeout = 1000;

var showPoolInfo = function(pool){
  console.log('poolSize: %d, availableObjects: %d', pool.getPoolSize(), pool.availableObjectsCount()); 
};

pg.connect(function(err, client, done) {
  var pool = pg.pools.getOrCreate();
  
  // poolSize: 1, availableObjects: 0
  showPoolInfo(pool);

  done();

  // poolSize: 1, availableObjects: 1
  showPoolInfo(pool);

  setTimeout(function(){
    // when this function executes the client in the pool has been destroyed and removed from the pool
    // poolSize: 0, availableObjects: 0
    showPoolInfo(pool);
  }, 2000);

});
```