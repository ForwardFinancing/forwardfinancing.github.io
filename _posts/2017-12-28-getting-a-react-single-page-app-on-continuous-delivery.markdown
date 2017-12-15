---
layout: post
title:  "Getting a React Single Page App on Continuous Delivery"
date:   2017-12-15 12:00:00 -0400
categories: culture continuous-delivery
author: Patrick Hereford
---

Your pilot product just launched after weeks of writing code. Feedback is coming
back in droves. Bugs are piling up. From being a two-person team on this soft
launch, the team now grows to four based on demand and feedback.  
  
Your engineering workflow encourages QA on a staging environment that closely
mimics production. Before the pilot, there wasn't ever a time where the team of
two really needed the staging environment at the same time. Now, it seems like
multiple engineers require staging at the same time, causing bottlenecks and
delays in shipping features to the customers.

This happens at every organization as it begins to scale its engineering
team. Bottlenecks like these are commonplace and can create quite a bit of
friction in an engineer's productivity.  
  
## Continuous Delivery to the Rescue
To alleviate this friction, most organizations turn to Continuous Delivery.
This practice is generally used to decrease turnaround times on features. What
this does not solve is the concept of siloed environments for QA. What
complicates this further is that our build is for a static asset that is hosted
on a CDN. Our build generates a manifest file and then one JavaScript and one
CSS compiled asset.

## Our Solution for Silos
We ended up creating a build pipeline for our staging and production environments.
These two environments, staging and production, are set up on auto deployment.
In addition, we added development builds that mimic our staging
environment from our development branch in git. These development builds rename
the manifest file so we can specifically access the development static assets.
Our server then knows how to request the specific compiled assets via url
query parameters. Below is a screenshot of our Heroku build pipeline that auto
generates our assets.

![Heroku Build Pipeline]({{"/assets/heroku_spa_pipeline.png" | absolute_url}})

Here is a screenshot of the end result in our CDN.

![CDN Manifest]({{"/assets/aws_pipeline_result.png" | absolute_url}})
  
## One Size Does Not Fit All
While building this, we realized that everyone's purpose in this single page
build process is vastly different. Most open source tooling we found was built
around building and compiling assets and serving it up via nginx and express.
Given that, our solution may not be perfect for your situation but it has
certaintly helped us become more productive as engineers.
