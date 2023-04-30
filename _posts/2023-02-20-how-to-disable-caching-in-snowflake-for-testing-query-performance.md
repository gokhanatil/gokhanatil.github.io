---
title: "How to Disable Caching in Snowflake for Testing Query Performance?"
layout: post
---

It's a common question to ask how to disable caching in Snowflake for testing. Although it's a very straightforward question, the answer is a bit complicated, and we need to understand the cache layers of Snowflake to answer this question. 

There are three cache layers in Snowflake:

1) Metadata cache: The Cloud Service Layer has a cache for metadata. It impacts compilation time and metadata-based operations such as SHOW command. The users may see slow compilation times when the metadata cache required by their query is expired. This cache cannot be turned off and is not visible to end-users if the metadata cache is used. 

2) Warehouse cache: Each node in a warehouse has an SSD storage. Snowflake caches recently accessed micro-partitions (from the Cloud storage) in this local SSD storage on the warehouse nodes. So running similar queries may use these cached micro-partitions instead of accessing remote storage. This cache cannot be turned off, but it's possible to see how much warehouse cache is used via the query profile:

![Reading from cache](/assets/readingfromcache.png)

<!--more-->

You may try to suspend/wait/resume the warehouse to clean the warehouse cache, but if the same nodes are assigned to the warehouse, your query may continue to use the cached data:

```sql
ALTER WAREHOUSE YOU_WH SUSPEND;
ALTER WAREHOUSE YOU_WH RESUME;
```
You may run the query on a new warehouse to avoid the warehouse cache, but it will be challenging if you need to re-run the query multiple times.

3) Result cache: Snowflake stores the results of queries on Cloud Storage. If a query is re-executed within 24 hours and the underlying wasn't changed ([check other requirements]()(https://docs.snowflake.com/en/user-guide/querying-persisted-results#retrieval-optimization)), Snowflake can return the result without executing the query. Each time the persisted result for a query is reused, Snowflake resets the 24-hour retention period for the result up to 31 days from the date and time the query was first executed. After 31 days, the result is purged, and the next time the query is submitted, a new result is generated and persisted.

The USE_CACHED_RESULT parameter can control this cache. The following command will disable it for the current session:

```sql
ALTER SESSION SET USE_CACHED_RESULT=FALSE;
```

You can check the query profile to see if a query uses the result cache:

![Using result cache](/assets/usingresultcache.png)

Please note that even result cache will be used for a query, the query still needs to be compiled! Sometimes, people may see that the total execution time is longer when the result cache is used because the metadata cache is expired, and the query execution time is already fast. 

So when testing the query performance, I suggest you ignore the metadata and warehouse cache. Just disable the result cache and run the query on a dedicated warehouse multiple times to get an average execution time. This will give you a better estimation of how the query will perform in the production environment.
