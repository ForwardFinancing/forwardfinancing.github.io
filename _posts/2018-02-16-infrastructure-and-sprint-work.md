---
layout: post
title:  "Infrastructure and Sprint Work: The Great Balancing Act"
date:   2018-02-16 12:00:00 -0400
categories: automation infrastructure
author: Brad Leyden
---
# Wearing Different Hats

Being an automation engineer can involve a number of things, and on a team like
ours that still has a relatively new and growing automation team, it does involve
doing a lot of different things -- many times, a lot of different things at once.

One thing I often think about with this job is the balance between doing in-sprint
work with features as they come through the pipeline and doing more "big picture"
infrastructure work.

I think there is a natural inclination to spend as much time as possible working
on QA for features in the sprint because it's a more visible and immediate way to
help their fellow engineers. No one wants to feel like they're neglecting the team,
so when a QA request comes in there is an understandable urge to drop what you're
doing to help get that feature over the finish line.

But improving infrastructure is inherently the type of work that demands
attention over long periods of time, and it is important not to neglect it.

# Breaking The Divide

Our team's sprint cycles involve a week-long slack time between each sprint. This
time allows engineers to "pick up the slack" by addressing things that wouldn't
fall under a standard sprint, such as reducing tech debt or learning a new
technology to hopefully use in future sprints. Slack time is a great time to
tackle these long-term infrastructure tasks without being pulled out of context
to do work tied to a sprint.

Unfortunately, as we've come to realize, a single slack time is not always enough
to complete the lengthy cycle of researching, experimenting, building, failing,
and repeating until the system you are working on is functional.

As an example, in our previous slack time we began the process of integrating
our automated test suites with Jenkins and Docker, technologies we didn't
currently have in our team's pipeline. Naturally, we didn't complete this process
in a week, but we carried our momentum into the next sprint and the result was
that for the first time, infrastructure work was becoming sprint work for us.
And that's where the balancing act comes in.

# What Comes Next

As both our feature and automation teams grow, one thing that will become critical
is our ability to meet the demand of the QA queue as we increase our overall
output, while still devoting periods of undivided attention to tricky infrastructure
tasks that will improve our processes and output in the long run.

With a growing automation team, my hope is that we will be able to break up into
fluidly defined subteams that can alternate between "sprint" and "non-sprint"
work.

These roles would need to alternate infrequently enough that each subteam
has plenty of time to make meaningful progress on their task, but frequently
enough that every automation engineer remains active and engaged on both fronts
throughout the sprint.

So, it all comes back to a balancing act, and it will likely be a process that
continues to change and evolve as we discover what works and what doesn't.

# TL;DR

As much as I love working with the features team to QA the awesome work they do,
I've found that I enjoy working on infrastructure just as much. It's not something
I expected coming into this role, and it's definitely been one of my most
exciting career developments since joining the Forward Financing team.

Balancing these two types of work can be a challenge, but I'm eager to continue
working to strike that balance so that we can simultaneously help to improve
individual app features and the pipeline as a whole.

Thanks for reading!
