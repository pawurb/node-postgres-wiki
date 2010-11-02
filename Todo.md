  - Typed result set support in client
    - date handling
    - edge cases
      - [numeric 'NaN' result](http://www.postgresql.org/docs/8.4/static/datatype-numeric.html)
      - float Infinity, -Infinity
  - Query results returned
    - some way to return number of rows inserted/updated etc
    (supported in protocol and handled in __Connection__ but not sure
    where on the __Client__ api to add this functionality)
      - how about ```query.on('end', function (numUpdated) { ... });``` ?
  - Error handling
    - disconnection, removal of listeners on errors
    - passing errors to callbacks?
  - more integration testing
  - connection pooling
  - copy data?
  - kiss the sky
