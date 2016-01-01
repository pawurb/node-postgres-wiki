- insert/update/select row count in query result callback 


_Though this would be extremely awesome off course, it is possible to obtain the behaviour by adding `RETURNING id` or even `RETURNING *` to the query. This works fine for single columns for me. I will test this for multiples and the handling of that case by this package. I am quite certain it must be possible in postgres._

**vitaly-t:** This will break just about every project or library that relies on precise return results. Therefore, making it implicit is a no-no! Also, whenever needed, `returning` works perfectly already, so it's not needed either.

And I think you have done some of ones below, just need to update the page ;)

- SSL support
- investigate cancelling queries
- refactor tests
 - use mocha
 - make it easier to get up and running
 - integration with travis-ci

***
[[◄ Back (Testing)|Testing]]