---
layout: post
title:  "PostgreSQL Performance and Diagnostic Tricks"
date:   2019-04-03
categories: database performance
author: Nathan Lilienthal
---

Here at Forward Financing we use [PostgreSQL](postgresql) heavily for our main
application databases. It's a modern, feature rich, and performant database.
But it's not always performant, and if you're not watching, it can come back to
bite you.

Starting the week of March 25, 2019, we noticed elevated timeout rates on one
of our main applications. This was quickly causing a bit of talk around office.

In this post we'll go over a few of the steps, related to our database, that we
took to resolve our issue. This application's DB has a number of good easy low
hanging fruit to pick off. These techniques aren't overly advanced, yet
resulted in a dramatic reduction in some queries, and ultimately response times
/ timeouts.

> 1. [Metrics](#metrics)
>     1. [Scout Metrics](#scout-metrics)
>     2. [Heroku Database Metrics](#heroku-database-metrics)
>     3. [`psql` Metrics](#psql-metrics)
> 2. [Improving Query Performance](#improving-query-performance)
>     1. [Missing indices](#missing-indices)
>     2. [N+1 Query](#remove-n1-query)
> 3. [Improving Database Health](#improving-database-health)

# Metrics

This is mostly a post about DB monitoring, diagnostics, and optimization.
However, this is the result of an investigation into a performance issue we
were facing with an application, which was causing both H12 (timeout) errors in
Heroku, but also just very poor response times.

- Server Respose Time
- Query Execution Time / Count
- Cache Hit Rates
- Table Bloat

## Scout Metrics

First, before we had identified any potential fixes, and we were just hearing
reports of pages timing out a lot more than normal, we could look at
[Scout][scout].

The image below is kinda a sneak peek at the performance gains we will achieve
throughout this post (so far).

![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/compare_admin_applications_show.png" | absolute_url}})

Here, the solid brown "total" is the optimized average response times, while
the dotted brown is the "total" for the previous slow code. It shows a -55%
change in mean response time!

## Heroku Database Metrics

Heroku Postgres's dashboard gives a few graphs that can help with your DB
sleuthing. Most relevant to this investigation were the following metrics:

- Most time consuming
- Slowest execution

Since the goal is to reduce the number of users impacted by timeouts, we
started by optimizing the most time consuming queries, and the queries used by
the pages timing out the most. Speeding up these queries puts less pressure on
the database, allowing other queries to flow.

Overly slow qeuries are also worth watching out for, and can indicate a query,
or table that needs work. Here's an example of a very slow query from our DB,
taking over 30 seconds to execute:

![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/original_fein_and_full_payload_on_accounts.png" | absolute_url}})

It's worth mentioning, in Heroku (other's too probably) our automatic backups
are `COPY` queries that are very slow. Possibly consider other backup strategies.

## `psql` Metrics

Seeing as PostgreSQL is a database it makes sense that it exposes its
statistics data as columns and tables. We'll start by looking at the
`pg_stat_statements` table, for looking at the execution time of our queries.

```sql
SELECT
  calls,
  total_time,
  total_time / calls as time_per,
  rows,
  rows / calls as rows_per,
  100.0 * shared_blks_hit /
    nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent,
  query
FROM pg_stat_statements
WHERE
  query NOT SIMILAR TO '%pg_%' AND
  calls > 500
ORDER BY time_per
--ORDER BY calls
--ORDER BY total_time
--ORDER BY rows_per
DESC LIMIT 20;
```


TODO

# Improving Query Performance

So, OK we've found some problems and we would like to actually speed things up
now. To simplify greatly the 3 steps to solving any performance issue are
these:

1. Use Metrics to Identify Hot/Relevant Spot(s)
2. Apply Optimization(s)
3. Monitor Optimization(s)

## Missing Indices

TODO: Short explanation of an index.

### Before (`accountid`, `stage_name`) Index on `opportunities`
![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/original_account_id_and_stage_name_on_opportunities.png" | absolute_url}})
```ruby
class AddAccountidAndStageNameIndicesToOpportunities < ActiveRecord::Migration[5.2]
  def change
    add_index :opportunities, :accountid
    add_index :opportunities, :stage_name
  end
end
```
### After (`accountid`, `stage_name`) Index on `opportunities`
![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/new_account_id_and_stage_name_on_opportunities.png" | absolute_url}})

---

### Before `account_id` on `addresses` Table
![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/original_account_id_on_addresses.png" | absolute_url}})
```ruby
class AddAccountAndContactIndicesToAddresses < ActiveRecord::Migration[5.2]
  def change
    add_index :addresses, :account_id
    add_index :addresses, :contact_id
  end
end
```
### After `account_id` on `addresses` Table
![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/new_account_id_on_addresses.png" | absolute_url}})

### Remove N+1 Query

Scout also detects and warns us about some N+1 queries.

![]({{"/assets/2019-04-03-psql-performance-and-diagnostic-tricks/n_plus_1.png" | absolute_url}})

Rails provides [`includes`][includes] just for the purpose of removing these kind of N+1
queries.

```diff
-<% owner.public_documents.each_with_index ... %>
+<% owner.public_documents
+        .includes(:document_overview)
+        .each_with_index ... %>
```

Turning:

```
DocumentOverview Load (20.3ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 1026634], ["LIMIT", 1]]
DocumentOverview Load (23.9ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 706717], ["LIMIT", 1]]
DocumentOverview Load (23.4ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 648758], ["LIMIT", 1]]
DocumentOverview Load (24.2ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 548840], ["LIMIT", 1]]
DocumentOverview Load (27.1ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 377232], ["LIMIT", 1]]
DocumentOverview Load (26.5ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 247995], ["LIMIT", 1]]
DocumentOverview Load (28.3ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 135709], ["LIMIT", 1]]
DocumentOverview Load (29.1ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 63550], ["LIMIT", 1]]
DocumentOverview Load (28.7ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 63548], ["LIMIT", 1]]
DocumentOverview Load (27.3ms)  SELECT  "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" = $1 LIMIT $2  [["document_id", 14441], ["LIMIT", 1]]
```

Into:

```
DocumentOverview Load (30.7ms)  SELECT "document_overviews".* FROM "document_overviews" WHERE "document_overviews"."document_id" IN ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10)  [["document_id", 1026634], ["document_id", 706717], ["document_id", 648758], ["document_id", 548840], ["document_id", 377232], ["document_id", 247995], ["document_id", 135709], ["document_id", 63550], ["document_id", 63548], ["document_id", 14441]]
```

Easy gains! This is why we keep all these protein bars around the office.

## Improving Database Health

While designing the data models and indices properly from the start is
generally the goal, there will be further steps, and possible optimizations
later. Database health therefor can mean a lot of things; A poor data model,
bad caching, bloated tables, unused indices, etc. Checking in on these aspects
of your DB from time to time may avoid more serious issues.

### Cache Hits

Checking the cache hit ratios can give you an indication of if PostgreSQL has
enough memory to properly cache its reads. Both the heap and index cache hit
rates should ideally be > 98%. Expensive queries can also affect this metric.

```sql
SELECT sum(heap_blks_hit) /
           (sum(heap_blks_hit) +
            sum(heap_blks_read)) AS cache_ratio,
       sum(idx_blks_hit) /
       sum(idx_blks_hit + idx_blks_read) AS index_ratio
FROM pg_statio_user_tables;

      cache_ratio       |      index_ratio
------------------------+------------------------
 0.80211381307796160475 | 0.99717843778473811314
```

Our indices are being cached quite well, but in generally our queries are
hitting IO more than they should. Possible gains from more memory, or improved
data model. Read more information about [caching in PostgreSQL][psql-caching].

### Unused Indices

```sql
SELECT
  pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size,
  idx_scan as index_scans,
  schemaname || '.' || relname AS table,
  indexrelname AS index
FROM pg_stat_user_indexes ui
JOIN pg_index i ON ui.indexrelid = i.indexrelid
WHERE NOT indisunique
ORDER BY
  pg_relation_size(i.indexrelid) / nullif(idx_scan, 0) DESC NULLS FIRST,
  pg_relation_size(i.indexrelid) DESC;

 index_size | index_scans |            table            |                             index
------------+-------------+-----------------------------+----------------------------------------------------------------
 3422 MB    |           0 | public.documents            | full_payload_docs_index
 525 MB     |           0 | public.versions             | index_versions_on_item_type_and_item_id
```

Look, a 3GB index that's never used! We should remove that.

TODO: Add some info about indices into JSONB keys.

### Going Further

There is a lot more information in the stats of PostgreSQL. [The
documentation][psql-monitoring-docs] is a good place to start reading more.
There's also Heroku's `pg-extras` repository which has [a lot of good
queries][pg-extras] for checking up on your DB, and [this][citusdata] post,
which includes a few of the most important statistics to query.


[postgresql]: https://www.postgresql.org/
[scout]: https://docs.scoutapm.com/
[includes]: https://apidock.com/rails/ActiveRecord/QueryMethods/includes
[psql-caching]: https://madusudanan.com/blog/understanding-postgres-caching-in-depth/
[psql-monitoring-docs]: https://www.postgresql.org/docs/11/monitoring.html
[pg-extras]: https://github.com/heroku/heroku-pg-extras/tree/master/commands
[citusdata]: https://www.citusdata.com/blog/2019/03/29/health-checks-for-your-postgres-database/
