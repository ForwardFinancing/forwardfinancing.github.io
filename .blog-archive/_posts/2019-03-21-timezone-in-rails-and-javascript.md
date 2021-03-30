---
layout: post
title:  "Timezones in Rails and JavaScript"
date:   2019-03-21 4:00:00 -0400
categories: Timezones
author: Jason Ji
---

This post contains two parts:
  1. [Timezone in Rails App](#timezone-in-rails)
  2. [Timezone in the browser](#timezone-in-browser)

-----
<br>

## <a name="timezone-in-rails">Timezone in Rails</a>
<div class="information information-tip">
  Rails provides great tools for working with time zones but there's still a lot of things that can go wrong. This post aims to shed some light on these gotchas and provide solutions to the most common problems.
</div>

Rails almost has you covered all the time, but I've learnt the hard way that I can't get away without knowing when and how Rails is helping me. There are so many timezone factors we need to consider:
  - Database Timezone
  - Production Server Timezone
  - Local Dev Machine Timezone
  - Framework Timezone

So what tools do we have at our disposal as Rails developers?

```ruby
config.time_zone = 'Eastern Time (US & Canada)' # rails application timezone
config.active_record.default_timezone = :local  # controls the timezone datetime is converted to prior saving to DB
```
The most important one is the `config.time_zone` configuration in your __*config/application.rb*__ file. This sets the application timezone. ActiveRecord will help you convert from and to UTC and the timezone of your choice.

Another one is the `config.active_record.default_timezone` configuration in your __*config/application.rb*__ file, This controls which timezone DB entries are stored in. It can be either `:utc` or `:local`. And `:utc` is default. If you set it to `:local`, ActiveRecord will help you convert to and from system local and the timezone of your choice when setting and retrieving from the database.

<div class="information information-warning">
  In almost all scenarios, we don't want to change this value, and we want to keep UTC as the default timezone for Database.
</div>

If all you're doing is receiving user data through a form and use Active Record to persist it you're good to go. But what about actually doing something with the time information before persisting it? That's when it becomes tricky.

When parsing time information it's important to pay extra attention to timezone. It's always a good practice to use the methods that use the timezone specified in our rails application config. Let's look at a couple of methods that parse date and time:

**Methods**                           | **Description**
===================================== | ===============
`Time.current`                        | uses timezone from rails application config
`Time.now`                            | uses timezone from system locale
`Time.zone.xxx`                       | uses timezone from rails application config
`Time.strptime(xx, xx).in_time_zone`  | uses timezone from rails application config
`Time.strptime(xx, xx)`               | uses timezone from system locale
`Date.current`                        | uses timezone from rails application config
`Date.today`                          | uses timezone from system locale
`String.to_time`                      | uses timezone from system locale
`String.in_time_zone`                 | uses timezone from rails application config

<div class="information information-tip">
  <div class="title">
    Looking carefully, you'll discover that the naming of the methods that adopt rails application configured timezone has a set of common patterns:
  </div>
  <p class="body">
    <ol>
      <li>we should use the <code class="highlighter-rouge">.current</code> method whenever available</li>
      <li>we should use the <code class="highlighter-rouge">zone</code> object as an intermediate object whenever available</li>
      <li>we could use <code class="highlighter-rouge">.in_time_zone</code> on any value to reset its timezone to rails application config</li>
    </ol>
  </p>
</div>


## <a name="timezone-in-browser">Timezone in Browser (JS Date API)</a>

The JS Date API is tricky and extra attention is required when creating a Date object from string:

<div class="information information-warning">
  The Date constructor will always return a Date object in browser's local time, but the argument string is taken care of differently when it is provided in different formats.
</div>

**format**                | **Result**        | **Description**
==========                | ==========        | ===============
`new Date('2013-02-22')`  | 'Feb 21 2013 ...' | this treats argument as UTC time
`new Date('2013/02/22')`  | 'Feb 22 2013 ...' | this treats argument as local time
`new Date('02/22/2013')`  | 'Feb 22 2013 ...' | this treats argument as local time
`new Date('22 Feb 2013')` | 'Feb 22 2013 ...' | this treats argument as local time


<br>
## Summary

Timezone issues are hard to discover and tricky to deal with. We should always keep timezones in our minds when dealing with date and time.

The main takeaways from this article are:
1. In Rails, always use the methods that takes application configured timezone into consideration.
2. In Browser, pay extra attention to the string format of the argument supplied to JavaScript `Date()` constructor.

<style>
  .information {border: 1px solid #ccc;padding: 10px 10px 10px 20px;border-radius: 5px; color: #333; margin: 10px 0 1em 0;}
  .information-tip {background-color: #f3f9f4;border-color: #91c89c;}
  .information-tip .title {font-weight: bold; font-size: 92%;}
  .information-tip .title::before {content: "✓"; display: inline-block; margin-right: 5px; color: white; background: green; width: 1rem; height: 1rem; border-radius: 50%; line-height: 1rem; font-size: small; text-align: center; }
  .information-warning {background: #fff8f7; border-color: #d04437;}
  table tfoot {text-align: left;}
  code.highlighter-rouge {padding: 0.2em 0.4em;margin: 0;font-size: 82%;color: #e01e5a;background: rgba(230, 235, 237, 0.25);border-radius: 0.375em;border: solid 1px rgba(210, 215, 217, 0.75);}
</style>
