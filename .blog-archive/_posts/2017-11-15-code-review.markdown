---
layout: post
title:  "How To Do Code Review, So It Rocks!"
date:   2017-11-15 12:00:00 -0400
categories: culture code-review
author: Janice Smith
---

 We do a lot of code review at Forward Financing. It's helped our team commit to delivering quality software and it has
 turned out pretty great. It can be hard to know how to dive in, so I'm going to shed some light on how it can be approached.

In general, the practice of code review helps peers collaborate on features. That collaboration brings fresh perspectives, exposing the coder and reviewer to new ways of approaching a problem they may not have thought of, while distributing that knowledge across the team. In order for this process to be successful, the review needs to be done well.

Initially the coder should consider adding some context to a given Pull Request. I like to put a description of the changes and the feature, but it can also be nice to comment on specific lines that might be complex.  Also a link to the card or ticket can be a big help. This is a good time for the coder to think about the PR as a whole.
- Should this be more than one PR?
- Should the code I write have any comments to clarify or document any of these changes?

Once the PR is ready to be reviewed, asking for the opinion of specific engineers is good along with keeping it open to the group for more points of view. Getting the eyes of someone who has spent a lot of time in the codebase is great, but just keep in mind as the coder, you will get different kinds of feedback based on who is reviewing. I like to try to get a few different perspectives if I can.

As a reviewer, the first pass is a good time to look for areas of the code that jump out at you.
- Are there any clearly unaddressed edge cases?
- Are there any deviations from the agreed upon style guide?

Be sure to add notes to the PR in a clear and good-natured way.

Beyond that, a reviewer can have more specific questions that may be better served by having a quick conversation, rather than a back and forth on the PR. Learning how each person likes to communicate and having everyone get to know each other's style is great. Consider asking questions about different approaches to solving an issue, i.e. “Why do you prefer a case statement here?”. These kinds of questions can touch on performance, readability, and extensibility. If it is a large feature or a big bug fix, it is great in code review to ask questions about where else this code is used or will be used. What may happen if the new code is used in ways maybe not intended by the coder? A conversation is one of the great things to come out of a code review.

Questions can be a big help, because the coder may have an answer they can add for more context, or it could be that those cases are covered by another feature. This helps the coder and reviewer find any possible issues, but also actively lets both of them get more insight into the feature.

Code review helps teams distribute the understanding and cognitive load of a codebase across the team, making it much easier to step in when needed. When a team approaches review positively with a mind to exchange ideas it can be a great thing.

In summary, along with a good outlook, the keys to code review are context, questions, and conversation.
