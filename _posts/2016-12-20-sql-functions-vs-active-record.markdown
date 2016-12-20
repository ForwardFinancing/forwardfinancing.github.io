---
layout: post
title:  "Need for Speed: SQL vs. ActiveRecord"
date:   2016-12-20 17:00:00 -0400
categories: sql ruby rails efficiency speed
---

#### **By Kelvin Ma, Jr. Software Engineer, Forward Financing** ####

Here at Forward Financing, the ability to report on the state our portfolio is the backbone of our business and the primary tool for us to check on the health of our investments. The bulk of that load falls to our funding application, a Ruby on Rails application powered by Postgres. 

The Tech team here recently brought the application in-house, after it had spent several years under the stewardship of consultants domestic and abroad. When it came under our umbrella in Spring 2016, reports were generated via ActiveRecord query, iterating through our portfolio of about 3,500 advances at the time and nearly 300k payments from merchants tied to those advances. 

The code structure involved a Ruby module dedicated specifically for report calculations and a caching system on all associated records whose goal was to improve the processing time. Calculations for each advance would involve iterating through hundreds of its associated payments, along with any rejections or fees that may have been incurred along the way. Multiply that by a few thousand, and it's no surprise that a bottleneck was inevitable.

A six-hour bottleneck at its slowest, to be exact. With several key stakeholders spread out across the Northeast requiring at least one report every day, the slowness of the process necessitated late-night, early morning or weekend report generation to avoid potential side effects slowing down regular business operations.

Though the code itself, by nature of it being Ruby, was semantically easy to read and understand, ActiveRecord, no matter how much we refactored and eager-loaded, was proving to not be a sustainable solution as we continued to grow our portfolio and our associated records increased exponentially.

### Enter SQL ###

At first, my team and I discussed a variety of solutions, such as porting out the reporting to another application in a more math-friendly language such as Elixir, to incorporating an already-in-use Solr solution. The overhead involved in getting everything up and running proved to be too large, and without any guarantee of a meaningful performance increase.

In the end, we settled on a lower-level language solution — Custom SQL functions. By translating all of the calculations that were initially in the Ruby module to custom SQL functions, we were able to avoid iterating over hundreds of thousands records and perform a single query off the database. 

I first started by building out a query that would establish a base unit on which all of our other calculations would be based across the rest of the report. In this situation, it was the sum total of all of our payments for a scoped range set by the user:

```
CREATE OR REPLACE FUNCTION total_payments_for_range(
    payment_start_date date, 
    payment_end_date date
)
    RETURNS TABLE (
      total_payments_for_range numeric,
      advance_id integer
    ) AS $$
    SELECT
      COALESCE(sum(p.actual_holdback_amount), 0) as total_payments_for_range,
      id as advance_id
    FROM advances a
    LEFT JOIN (
      SELECT
        actual_holdback_amount,
        advance_id
      FROM payments p
      WHERE p.payment_date >= payment_start_date AND p.payment_date <= payment_end_date
    ) as p
    ON p.advance_id = a.id
    GROUP BY a.id
```

By structuring all of our subsequent calculations around this base query, we were able to bring down our total report time to around 27 seconds on the first iteration of the SQL report function. Further refactors to remove my myriad structural problems, mostly duplicated or redundant queries, I was finally able to get our current build down to... *drumroll please*

```
EXPLAIN ANALYZE SELECT * FROM generate_report(~~PARAMS~~);
                  QUERY PLAN                                                          
--------------------------------------------
 Function Scan on generate_report  (cost=0.25..10.25 rows=1000 width=1552) (actual time=3610.526..3610.817 rows=2995 loops=1)
 Planning time: 0.087 ms
 Execution time: 3611.286 ms
(3 rows)
```

Implementing the SQL query in to our existing Rails stack involved a trivial amount of overhead, setting up a sanitized query via Rails [`sanitize_sql_for_conditions`](http://api.rubyonrails.org/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_sql_for_conditions) and building out a custom report record `Struct` to handle the query table.

3.6 seconds. Down from 6 hours.

