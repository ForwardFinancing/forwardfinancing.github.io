---
layout: post
title:  "Automating Processes with Gen Servers: A Beginner's Approach"
date:   2018-03-16 10:00:00 -0400
categories: elixir automation microservices
author: Kelvin Ma
---
As a financial services company, Forward Financing invests a signficant amount of time and resources in timebound reporting -- think daily, weekly, monthly, etc., report generation. As opportunities for automation are concerned, it doesn't get more obvious than this.

Getting out of my comfort zone has been a key part of my engineering growth, using new and unfamiliar tools to build potentially useful tools. After doing a little research on potential solutions for our automation opportunity, I settled on Elixir's GenServer, or Generic Server, pattern. Coming from a primarily Object Oriented Programming background with Rails, this pattern was a bit foreign to me, to say the least.

### Getting Started

The basic premise for this project was to spin up a simple Phoenix/Elixir microservice application to perform two main tasks -- make an API call to one of our monolith apps for portfolio data and then generate an email with the results once a month. After building out the API call in Rails, it was time to get started on the GenServer.

From the [GenServer](https://hexdocs.pm/elixir/GenServer.html) HexDocs:

> A GenServer is a process like any other Elixir process and it can be used to keep state, execute code asynchronously and so on.

Seems pretty straightforward. By writing a GenServer and isolating this single task into its own process, I am effectively creating a microservice that can be managed by the application's own [supervision tree](https://hexdocs.pm/elixir/Supervisor.html).

Here's what I ended up with, using the default callbacks for GenServer:

```elixir
  # Default implementation of GenServer
  def start_link(opts) do
    GenServer.start_link(__MODULE__, :ok, opts)
  end

  # Initializes the GenServer on application startup
  def init(state) do
    schedule_work()
    {:ok, state}
  end

  # Generic catch-all message/event handler
  def handle_info(:work, state) do
    do_work() # The actual business logic, defined below
    schedule_work() # Process scheduler
    {:noreply, state}
  end

  # The work!
  def do_work() do
    # Business logic goes here!
  end

  # Scheduler to set interval for do_work()
  defp schedule_work() do
    Process.send_after(self(), :work, INTERVAL_IN_MILLISECONDS)
  end
```

Back to the docs to explain what's going on here:
> The goal of a GenServer is to abstract the “receive” loop for developers, automatically handling system messages, support code change, synchronous calls and more.

Once this GenServer gets integrated into the supervision tree,  we've essentially created a black box that will receive periodic messages to "do work".

No controllers. No models. No database.

Doesn't get much simpler than that!

### Future Implications

At this point, you know that I can now read and follow basic documentation. What a relief!

What I find most fascinating about this particular pattern is its ability to effectively manage multiple microservices within a single supervision tree. While in this particular example I'm only implementing a single GenServer, adding another two, three, 10, or more would be a trivial process. They could even communicate with each other.

As an engineering organization, Forward Financing is committed to the microservice philosophy. Our ecosystem of applications has grown significantly in a very short period of time, so being able to effectively manage processes across all of our core business functions is and will continue to be a priority.

What kinds of options do we have with this pattern?
* A monolith of microservices, operating within a single application's supervision tree
* Multiple business-related microservices organized by business function
* Individual applications for each microservice
* Something else entirely?

There are pros and cons for all of these, I'm sure, but figuring it out on our own is going to be a lot of fun.
