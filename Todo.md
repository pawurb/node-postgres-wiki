  - Typed result set support in client
    - __done__ date handling
    - edge cases
      - [numeric 'NaN' result](http://www.postgresql.org/docs/8.4/static/datatype-numeric.html)
      - float Infinity, -Infinity
  - Query results returned
    - some way to return number of rows inserted/updated etc
    (supported in protocol and handled in __Connection__ but not sure
    where on the __Client__ api to add this functionality)
      - how about ```query.on('end', function (numUpdated) { ... });``` ?
        - need to consider multiple statements (each return a status message, `end` handler only fires once.
0 maybe instead of numUpdated pass an array of status objects, one for each command
      - or as a third parameter to the callback ```client.query(query, function (err, result, numUpdated) { ...```?
        - thinking of appending to to the result object directly
  - Error handling
    - __done__ removal of listeners on errors
    - __done__ passing errors to callbacks?
  - remove test dependency on script/create-test-tables.js?
  - more integration testing
  - __done__ connection pooling
  - __done__ copy data?
  - testing transactions
