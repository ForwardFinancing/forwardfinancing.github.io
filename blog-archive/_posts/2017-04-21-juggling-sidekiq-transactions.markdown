---
layout: post
title:  "Sidekiq Gotchas: Juggling Multiple Database Commits in Sidekiq Transactions"
date:   2017-4-21 14:00:00 -0500
categories: sidekiq activerecord ruby rails
author: Kelvin Ma
---

## The Issue

In one of our applications, we have a feature that generates a schedule of payments for our merchants' accounts. In these accounts, balances change, payment amounts may fluctuate, fees are incurred, terms can alter —- lots of things that can throw off the duration of a daily payment schedule. To solve this, our initial solution was to handle the recalculation via Sidekiq Worker, handling all of the different factors for potentially hundreds of thousands of records asynchronously. The worker would purge any outdated records, and then regenerate a schedule based on the new criteria for the account. Pretty straightforward, right?

It looked something like this:

```
def rebalance_schedule(starting_date)
  delete_payments_after(starting_date)
  create_scheduled_payments_after(starting_date)
end

class ScheduleWorker
  ...
  def perform(id, starting_date)
    schedule = Schedule.find(id)
    rebalance_schedule(starting_date)
  end
end

```

Of course, a solution can never be that simple.

As soon as we started running this process, we noticed that schedules were showing duplicate items in the schedule. Not good!

## So, what happened?

Because of the transactional nature of the Sidekiq job, the duplication turned out to be side effect of introducing two database commits within the worker.

```
Within the context of the worker...

def rebalance_schedule(starting_date)
  delete_scheduled_payments_after(starting_date) <-- Schedule payments "deleted", but not committed
  create_scheduled_payments_after(starting_date) <-- Creates duplicate payments on top of existing, but not-yet-deleted items

  ** Duplicates get committed to database -- Bad! **
end
```

As a transaction, we should assume that there is only going to be one database commit. Makes sense, right?

Our solution ended up decoupling the computationally easy deletion from the worker and only running the computationally expensive recalculation and record creation in to the asynchronous job.

```
class ScheduleWorker
  ...
  def perform(id, starting_date)
    schedule = Schedule.find(id)
    create_scheduled_payments_after(starting_date)
  end
end

def rebalance_schedule(starting_date)
    ActiveRecord::Base.transaction do
      delete_future_payments(starting_date)
    end
    ScheduleWorker.perform_async(id, starting_date)
  end
end
```

TL;DR: Don't put multiple transactions into a single Sidekiq job!
