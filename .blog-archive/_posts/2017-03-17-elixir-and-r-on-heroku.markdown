---
layout: post
title:  "Getting Elixir and R working on Heroku"
date:   2017-03-15 12:00:00 -0400
published: true
categories: elixir r heroku
author: Patrick Hereford
---

## Framing the Problem
Our business executives needed some in depth analysis on payment performance in
the form of something called a [vintage curve](http://capitalsvcs.com/newsroom/whitepapers_case-studies/articles/Vintage%20Analysis-What%20is%20it%20and%20what%20does%20it%20do.pdf).
For those that are entrenched in the analysis world, this looks pretty familiar,
right? It looks eerily similar to [cohort analysis](https://en.wikipedia.org/wiki/Cohort_analysis).
It probably looks that way because it is a form of cohort analysis. What the
execs wanted to measure is payment performance of their portfolio for each
"vintage" (i.e. a month that we funded a small business) and seeing how that
group performs in comparison to other groups.  

## Formulating a Solution
Our solution, which will be detailed in another blog post, had to involve a 
handful of technologies. We need to host this software on a server (Heroku).
We needed SQL, as all of our data resides in a RDBMS. It has to involve a
highly computational language, we chose R, to handle summarization and data
reshaping. Lastly, we needed a layer to handle API requests to get the data
out of the system, of which we chose Elixir.  

## Implementing the Solution
**Assumption**: You have a heroku account and Heroku Toolbelt
installed on your system as I go through a variety of commands and concepts.
For our purposes, our heroku app will be called `tool-analysis-example`.

### Getting R on Heroku
The first thing we need to do here is get a R buildpack on associated to our
heroku app. This [one](https://github.com/virtualstaticvoid/heroku-buildpack-r/tree/cedar-14-chroot)
looks pretty good. To add it, we run the following command:
`heroku buildpacks:add --index 1 https://github.com/virtualstaticvoid/heroku-buildpack-r.git#cedar-14-chroot`

From reading the README, we need to add an `init.R` file to the main directory
of our repo. This `init.R` is used during deployment to install necessary
dependencies. Here are the dependencies we used:
```
install.packages('devtools', repos='http://cran.us.r-project.org')
devtools::install_github("RcppCore/Rcpp")
devtools::install_github("rstats-db/DBI")
devtools::install_github("rstats-db/RPostgres")
install.packages('jsonlite', repos='http://cran.us.r-project.org')
install.packages('httr', repos='http://cran.us.r-project.org')
install.packages('reshape2', repos='http://cran.us.r-project.org')
```

While installing R for the first time, all of these dependencies installed
correctly with the exception of RPostgres. The error that was raised was due
to not having libpq-dev on the instance. Bummer! Time to fix that now! We now
need to install libpg-dev on our instance using Apt. To do this, we add another
buildpack at the beginning of our pipeline:
```
heroku buildpacks:add --index 1 https://github.com/heroku/heroku-buildpack-apt
```

This takes an `Aptfile` in the root directory which will contain a single line
of `libpq-dev`.

Our next deploy will _NOW_ be successful and R will be able to communicate with
our RDMBS.

### Getting Elixir on Heroku
This is pretty straightforward and just takes a [buildpack](https://github.com/HashNuke/heroku-buildpack-elixir).
`heroku buildpacks:add --index 3 https://github.com/HashNuke/heroku-buildpack-elixir`
Per the readme, this one needs a `elixir_buildpack.config` file in the root
directory of your application's repository. Your next deploy will now install
erlang and elixir without issue.

### Getting Elixir to Communicate with R
Lastly, we need to get Elixir to invoke an R script we created. For this
example, our R script is located at `./r_scripts/vintage.R` in our repository.

Using Elixir's `System` module, we can invoke commands on the system.
```
System.cmd("Rscript", ["./app/r_scripts/vintage.R"])
```

The output of this command will be a tuple where the first element is a string
of the output from your R script.

Our API endpoint that returns the R results from this sytem averages about 3
seconds. Not too shabby given the amount of data it is returning! We could
reduce that probably by 60% by altering our JSON payload to be streamlined.

## TLDR
It is possible to get R and a web framework of choice on Heroku. Installing apt
on Heroku was definitely not expected, but got us over the finish line.

Thanks for reading!
