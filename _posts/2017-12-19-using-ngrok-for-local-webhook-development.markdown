---
layout: post
title: Using ngrok for Local Webhook Development
date: 2017-12-26 12:00:00 -0400
categories: tools
author: Zach Cotter
---
Many API's use webhooks to subscribe consumers to realtime information changes. When a record is updated or an event occurs in the providers system, they notify consumers by sending that data via HTTP request to each consumer's web server. This strategy can be preferable to modern technologies like WebSockets in some cases, because the consumer isn't required to maintain a long running web connection with the provider.

The server-initiated nature of webhooks makes developing a webhook consumer on your local machine challenging.
With traditional API development, I would usually start by using a HTTP request client like Postman or Curl to hit the service I'm integrating with. Once I have a good idea of what the requests and responses look like, I'd start up a debugging console in my local app and start building the code interactively. But with webhooks, my app receives the HTTP requests instead of sending them. How do I point the webhook provider to my local app running on `localhost`?

A colleague recommended an awesome (free) tool called [`ngrok`](https://ngrok.com/). Ngrok provides a service which records web requests on a publically accessible domain. These recorded requests are then forwarded to your local machine at a port of your choice. The responses your local machine sends are forwarded all the way back to the original requester.

#### Installation

In order to run ngrok, you'll need to register for an account and [download+install](https://ngrok.com/download) the tool on your  machine.

#### Getting it started
First I spun up my local web server with a debugger in place. I was using Elixir/Phoenix for this project, so my endpoint looked something like this:

```elixir
defmodule MyProject.Api.V1.MyController do
  use MyProject.Web, :controller

  @spec create(Plug.Conn.t, Map.t) :: Plug.Conn.t
  def create(conn, params) do
    require IEx
    IEx.pry
  end

```

`ngrok` takes a network protocol and the port your server is running on. So if my app were running locally on port 3000, I would run:

`ngrok http 3000`

Once started, the ngrok console looks something like this:

```
Session Status                online
Account                       Zach Cotter (Plan: Free)
Version                       2.2.8
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://5e7e5b85.ngrok.io -> localhost:3000
Forwarding                    https://5e7e5b85.ngrok.io -> localhost:3000

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

The "Forwarding" attribute lists the publically accessible URL which is forwarding to our web server on port 3000, in this case `http://5e7e5b85.ngrok.io`

You can configure your webhook service to point to your debugging endpoint at that domain. For the endpoint above, the full url would be `http://5e7e5b85.ngrok.io/api/v1/my_controller`

You could also just test hitting that URL from Postman or Curl locally to double check that your web server is running.

#### The web interface

Ngrok offers a very handy local web interface which shows each request and response with URL, body and headers.
This console can be found at `http://localhost:4040`

![CDN Manifest]({{"/assets/ngrok_web_console.png" | absolute_url}})

#### Request Replay

One of the best features with ngrok is the "Replay" button in the upper right. Clicking this sends the request from the webhook server to your local server again. This made a number of things much easier for me. I found that whenever my local web server received a second request from the webhook server, it would break the debugging session I was currently in, causing me to lose some of my work. This was solved by disconnecting the webhook server after it had sent one request, and then just replaying that request from `ngrok` whenever I needed one to work with. The "Replay" button was also helpful because I didn't have to wait for a new webhook to come through to test my code everytime I changed something.

#### HTTPS

We use HTTPS locally for all of our web services in development for security and to more closely mimic the environment found in production. To get `ngrok` pointing to `https` locally, all I had to do was change the command its run with:

`ngrok tcp 3000`

This command gives you a forwarding URL that looks like: `tcp://0.tcp.ngrok.io:10034`. You can just change the `tcp://` to `https://` and give that URL to your webhook provider


#### More about ngrok

Ngrok has tons of other features, and seems really well documented. There is a lot more to see on the [documentation page](https://ngrok.com/docs)
