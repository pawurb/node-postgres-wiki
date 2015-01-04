- insert/update/select row count in query result callback 


_Though this would be extremely awesome off course, it is possible to obtain the behaviour by adding `RETURNING id` or even `RETURNING *` to the query. This works fine for single columns for me. I will test this for multiples and the handling of that case by this package. I am quite certain it must be possible in postgres. _
- SSL support
- investigate canceling queries
- refactor tests
 - use mocha
 - make it easier to get up and running
 - integration with travis-ci