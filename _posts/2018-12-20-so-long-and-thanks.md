---
layout: post
title:  "So Long, And Thanks for All the Coffee"
date:   2018-12-18 4:00:00 -0400
categories: co-op learning
author: Hobin Yang
---
Up until July when I joined Forward Financing for my first ever co-op, I didn't
really have a clear picture of what kind of work I was going to be doing. I had taken a good number of
courses in university which taught everything from functional programming to design
patterns and AI algorithms. I had done a few small-scale projects in
those classes and during my spare time, and even worked a little bit
with React and a few APIs to create a web app. Good enough, right? Well, not exactly.
I would soon find out that those projects bore little resemblance to
what day-to-day software engineering entailed and were nothing like the constantly force-pushed
one-branch repo with commit messages such as 'help' or 'possible fix'.

The first thing I noticed when I started was the sheer number of repositories
that I had to clone and oh wow look so many directories and branches and jeez
that's a lot of dependencies and what's a Sidekiq? I thought only superheroes had
those. It took a few weeks to get adjusted into a daily routine after setting everything
up and attending meetings to absorb as much knowledge about the different apps and how they
worked together.

To be honest, I felt quite unprepared for my role here as a full stack engineer
co-op. I didn't have a baseline at all as to what I could handle and working with
an unfamiliar and huge codebase was daunting to say the least. Not to mention that I had
little to no experience working with the tech stack here (Ruby, Elixir, React).
What did help was taking on small tasks like improving test coverage and addressing
minor tech debts to get exposed to the many different applications the company uses.
In addition, everyone on my fantastic team was friendly and eager to help, encouraging
that I ask any question I had. It seemed all dandy and fine besides the fact that I was still doubtful of my ability to perform well jumping off the deep end into the world of web development.

# Overcoming Impostor Syndrome
Ever since I was first accepted into Northeastern, I felt as if I didn't really deserve to be in
such a good school - all I did in high school was coast by and play lots of video games. And I felt
the same way everytime I was praised for being "smart" on the sole basis that I was majoring in Computer Science, when I got a tutoring job for a class that I really struggled in, and most recently being hired as a co-op here. I wasn't the kid that asked dozens of questions to the professor out of pure curiosity nor did I have a particularly easy time in my CS classes which didn't help. I had a lot of doubts about myself being competent as a co-op as I was unfamiliar with all of the tech stack as well as not possessing any prior experience. But as my first co-op comes to a close, I now feel that I've learnt a substantial amount compared to what I thought I was capable of both in terms of technical and communication skills. And sometimes you do get lucky, but it's what you make of those opportunities - and I think that I've done a satisfactory job.

# Really, You're Not Bothering Anyone
If there's one thing that I would say I could change from my time working here, it would be that I would ask a lot more questions. I thought that my questions were usually basic and could be easily found
searching for it online, and so I would always do that before asking any. The few questions I did
ask ended up with very simple answers so having a habit of doing things by myself, I felt that I hadn't tried enough to answer the question myself. However, I would end up not being able to find the answers I was looking for and kept feeling like a burden asking simple questions like what a specific query did or what the stack trace from the error meant. 
Furthermore, I would have asked so many questions about how the apps work from a business
perspective by shadowing the stakeholders (underwriters, prequal analysts) and seeing the overall workflow
and what issues there were with it. I only got a good feel for _one_ piece of the underwriting app - our internal risk analysis tool for funding merchants - 
after struggling on a card (described in detail later) and asking lots of questions directly to an
underwriter. As weeks passed by and I got more comfortable at the job, 
I got better about leaving aside that feeling and asking the question anyways - 
but I do wish that I had done it sooner.

If I had to do it all over again, I would've asked about five times more questions than I did
regardless of how basic or obvious the answers seemed.
Funny thing is, I've heard this advice over and over again
and yet it fell on deaf ears for me until I went through the experience myself. But I know now
for the future that I might end up a bit annoying sometimes but at least I'll learn a lot!

# Always in Flux
A rather unique experience I had during my time at Forward Financing was quickly adjusting
to the constant change. From when I joined in July to a couple months later in
December, the company went through a few big changes. We had (and still have) a shortage
of product managers which taught me how important they are in keeping the
software development lifecycle (SDLC) running smoothly especially with scoping out tasks and minimizing
the friction between stakeholders and engineers. 

I would like to start of by talking about a rather significant change in our engineering process where our
sprint of three weeks + a few days reserved for what we call slack time - a short period where the team
works on tech debt and projects that are not of immediate business concern -
was changed to two weeks with no slack time. However, I feel that this did not pan
out too well due to meetings happening too often causing a lack of productivity. Additionally, tech debt piled up due to immediate business needs being prioritized over it, thus there was no time to work on it with the removal of slack time.

These are just some of the observations I made during this slightly chaotic time; others
might have completely different interpretations and thoughts of the matter.
As of the time of writing this post, we have reverted back to a three-week sprint with a week's
worth of slack time. From this, I learned that processes can and will change, but they don't necessarily
need to be permanent and should be critically assessed to see if they work out or not. The same process that happened to not fit into our workflow may as well be the best one for another engineering team.

# Bite off a Bit More than You Can Chew
I had absolutely no idea how much I could take on in terms of workload and deliver in a timely manner.
What if I didn't finish my cards for the sprint? Or what if I break other features with my new one?
But with time I've come to realize that it wasn't about delivering everything on time or having a perfect
feature, but rather to extend myself a bit more every time and fail often. During one sprint I had just one card and was hesitant on asking for more (even though they weren't big tasks), but I took another and it went just fine. And although it might not work out everytime, failure in software engineering is not such a huge issue. Just as with tests, things just might not work the first time around and that's okay. 

# `Hash#Dig` then Dig a Bit More... Even If You Find a `nil`
So during my last few weeks here, I decided to pick up an seemingly straightforward
3 point task that involved pulling date info from LexisNexis and Experian to compare with
data from a merchant's application. All I was actually doing was writing a small service
to grab the data and load it in the ERB template for the view. Little did I know how awful
APIs could be as well as that it would take me a month to complete this tiny feature.
I would say that I spent maybe a couple days actually writing the service and helper to create
the view, but pulling the data was a nightmare. Having never worked with the Underwriting App
before, diving deep into binary payload and digging out 10-times deep nested hashes for one
Date object was not the most fun thing I had in mind.

In the beginning I thought it was a one hour task, as the documents had methods that pulled dates from a background check vendor and a credit bureau which I believed to correspond to what I needed.
However, I soon found out after that those were the dates of when the data was pulled, not the actual
dates that I needed. Figuring out the specific document type that contained the data was a hassle, not to
mention that I probably must have seen over a thousand nil values helplessly trying to dig for the proper
object. After wrestling with the task for a week, I figured out the exact method to pull the data,
but it was not pulling the correct data from some applications. Then another week of struggle passed until I realized that the backend logic was already done in a separate part of the app - until that point,I didn't know I could have felt this demoralized and plain dumb for not seeing it sooner.

However, I found a silver lining after going through this process: I had dug a lot deeper into the app than I would have if I were to find the pre-existing function initially, getting quite familiar with the app's object relations, MVC design, as well as discovering the nuances of Ruby. I would do it
better the second time around, but that's exactly what experience is for.


# The `end`... Or Not
And with less than a few days left of my first co-op, I would like to thank the
entire engineering team for making my time here wonderful and memorable.
We have a great team here not only in Boston, but all over the world
in the UK, India, Dominican Republic, Brazil, and Paraguay - all of whom flew in just last week
to make for a memorable holiday season and end of the year.
The clichéd response to "what's your favorite part of the job" is "the people" in every
interview I've had, but for this co-op it had substance and proved itself well.
I have had a lot of exciting and unique experiences here that I would be hard pressed to
find at another company and am going to take the lessons learned here forward (pun absolutely intended)
into the future. 


# Acknowledgements
I would like to thank everyone here at Forward Financing and shoutout a few people for making my time here so very pleasant:

**Kelvin** - For sound mentorship/guidance and establishing #BestRow

**Zach** - For solid leadership and nuanced humor (and being the only other person I know to use [Micro](https://micro-editor.github.io/))

**Scott** - For bringing so much energy into the office everyday

**Natacha** - For helping me find footing by pair programming extensively when I first started

**Mike** - For diving deep into seemingly infinitely-nested Hash structs with me

**Nate** - For the thorough code reviews and red-hot enthusiasm for programming languages

# **Thanks for reading!**