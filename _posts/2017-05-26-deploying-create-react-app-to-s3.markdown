---
layout: post
title:  "Integrating create-react-app with Elixir + Phoenix via Amazon S3"
date:   2017-06-01 00:00:00 -0000
categories: react elixir devops
author: Zach Cotter
---

At Forward Financing, we've started using
[create-react-app][create-react-app]
as a starter kit for all of our new
[React][react]
projects. create-react-app is awesome
because it comes with a one-size-fits-all webpack configuration via the
`react-scripts` package. All you have to do is `npm run build` and you have a
ready-to-ship `/build` folder with minified production asset files.

For our first project using create-react-app, we built an interface to manage our users' roles and access across our tech ecosystem. The backend server was our standalone single sign-on service,
written in Elixir+Phoenix. An
API for the new React frontend was built into that backend service.

We had a few requirements for the deployment process of the React app:

- The compiled JavaScript+CSS assets needed to be hosted somewhere cheaply and
efficiently.
- The React app need to live at a subresource of our existing single sign-on
application, specifically `login.forwardfinancing.com/panel/user_roles`.
- The React app needed to have a separate deploy process from the backend app

### Hosting the React build folder

One popular option to quickly deploy React apps is to spin up an
Express server on a shared
virtual server provider like Heroku or DigitalOcean. This doesn't make a lot of
sense because you are paying ~$5/month just to serve up a few static
files from a dedicated cloud server. I also am not a fan of using JavaScript as
a backend web server because it is single threaded. Instead, we decided to serve our assets from
AWS S3+CloudFront. This has a few big benefits:

- Amazon has already optimized those services to do one thing, and one thing
only - serve up static files.
- Instead of paying for a whole server, you are only paying fractions of a cent
to serve the files.
- If someone wants to launch a DDoS attack against the service providing the
assets, they will have to attack Amazon instead of you. S3 offers configuration
options to help prevent this.

In order to host the assets cheaply and effectively on S3, we more or less
followed [this helpful article.][helpful-article].

1. Create a bucket on S3 and make that bucket public

2. Install aws-cli using your favorite package manager (brew install aws-cli)

3. Run aws configure with your credentials

4. Add the following to your `package.json` scripts:
{% highlight json %}
{
  "scripts": {
    "deploy": "aws s3 sync build/ s3://your-bucket --region your-region"
  }
}
{% endhighlight %}

When you run `npm run deploy` after having run `npm run build`, the contents of
the build folder will be synced to your S3 bucket.

Now your React app should be visible at
`https://<your-bucket>.s3.amazonaws.com/index.html`

### Using the hosted assets in our backend app

But we don't want to mount the React app at
`https://<your-bucket>.s3.amazonaws.com/index.html`.

We want to mount it within
our backend service at `login.forwardfinancing.com/panel/user_roles`

In order to mount the React app in our Elixir backend we created a view layout
that matched the `index.html` file in the S3 bucket. Next we replaced the
relative asset URLs with the full URL to the file on S3:

{% highlight html %}
<!-- In `web/templates/layout/react.html.eex` -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- ...more head elements here... -->
    <link
      href="https://<your-bucket>.s3.amazonaws.com/static/css/main.123abc45.css"
      rel="stylesheet">
  </head>
  <body>
    <!-- This is the mount point for the react app -->
    <div id="react">
    </div>
    <script
      type="text/javascript"
      src="https://<your-bucket>.s3.amazonaws.com/static/js/main.123abc45.js">
    </script>
  </body>
</html>

{% endhighlight %}

Next, we modified the router file in our Elixir app to direct any requests to
paths starting with `/panel/user_roles` to the new layout which loads the React
app:

{% highlight elixir %}
defmodule YourApp.Router do
  use YourApp.Web, :router

  pipeline :react do
    plug :put_layout, {YourApp.LayoutView, :react}
  end

  scope "/panel", YourApp do
    pipe_through [:react]
      get "/user_roles/*path", YourController, :index
    end
  end
end
{% endhighlight %}

Now, when a user navigates to any path starting with `/panel/user_roles`, the
React layout renders a page with your React app. At that point, React router
takes over, so no more requests will be made to the backend other than those
for the API until the user refreshes the page or navigates out of the React
part of your frontend.  


### Handling the asset manifest fingerprints

You'll notice that the compiled asset files produced by `create-react-app`
include an 8-digit fingerprint, like `main.123abc45.css`. This fingerprint
changes every time you make a change to your app and recompile it. The
goal of this is to indicate to caching systems when the assets change that their
cache needs to be invalidated. So, we need our backend to make a request to
S3 for the asset manifest (which has a constant URL) in order to make sure we
are pointing to the correct version of our assets.

We created a small service module in our Elixir app to contain this logic:

{% highlight elixir %}
defmodule YourApp.AssetManifestFetcher do
  @moduledoc """
    Goes to S3 and downloads the asset manifest file, which we use to find
    the fingerprints for the frontend asset.
  """
  use HTTPoison.Base

  @doc """
    The url for the S3 asset manifest file
    Takes the domain of the S3 bucket
  """
  def process_url(url) do
    "#{url}/asset-manifest.json"
  end

  @doc """
    Turns the response body from a JSON string to a Map
  """
  def process_response_body(body) do
    body |> Poison.decode!
  end

  @doc """
    Makes the request for the asset manifest and returns it as a hash
  """
  def get_manifest(domain) do
    get!(domain).body
  end
end
{% endhighlight %}


We then used this service module in our controller to get the asset urls and pass them to the view:

{% highlight elixir %}

defmodule YourApp.YourController do
  use YourApp.Web, :controller

  # We define these variables in config so they can quickly be swapped out in
  #  production and so that its easy to stub over them in test
  #  (Great article about that test concept here: http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/)
  # Returns a String with your S3 url
  @domain Application.get_env(:your_app, :frontend_asset_domain)

  # Returns the service module (YourApp.AssetManifestFetcher)
  @asset_manifest_fetcher Application.get_env(:your_app, :asset_manifest_fetcher)

  @doc """
  Get the asset urls from the asset manifest and pass them to the layout.
  """
  def index(conn, _params) do
    manifest = @asset_manifest_fetcher.get_manifest(@domain)
    render(
      conn,
      :index,
      %{
        css_asset: asset_url(manifest, "main.css"),
        js_asset:  asset_url(manifest, "main.js")
      }
    )
  end

  # Given the downloaded asset-manifest, return the full URL to the
  # fingerprinted asset.
  # For example, "main.css" becomes:
  # "http://something.s3.us-east-2.amazonaws.com/static/css/main.441e57a7.css"
  defp asset_url(manifest, asset_name) do
    "#{@domain}/#{manifest[asset_name]}"
  end
end

{% endhighlight %}

Next, we modified the layout to reference the new variables instead of hard coded
asset paths:

{% highlight erb %}
<!-- In `web/templates/layout/react.html.eex` -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- ...more head elements here... -->
    <link href="<%= @css_asset %>" rel="stylesheet">
  </head>
  <body>
    <!-- This is the mount point for the react app -->
    <div id="react">
    </div>
    <script type="text/javascript" src="<%= @js_asset %>">
    </script>
  </body>
</html>

{% endhighlight %}

This setup adds a few milliseconds to the initial React load time as the round
trip request is made to S3 for the asset manifest. That request should probably
be cached in higher volume production environments.

### Future Steps

One feature that would be cool to add to the React build process would be
the ability to deploy compiled assets to different environments. This would
probably involve having the `npm run deploy` command take an argument with the
environment name (ie staging, production), and post the assets to the S3 bucket
in a folder with that name. Then the backend staging and production environments
could point to the correct folder, and the assets for each would be separate.

A further spin on that would be to mimic Heroku's "Review Apps" feature by
deploying assets to a folder within the bucket with the same name as the
current branch. This would be as simple as:

`aws s3 sync build/$(git symbolic-ref --short HEAD) s3://your-bucket --region your-region`

Then, reviewers could preview your changes just by going to:
`https://<your-bucket>.s3.amazonaws.com/your-branch/index.html`

The staging and production environments could point to the assets in the folders
for develop and master.

[react]: https://facebook.github.io/react/
[create-react-app]: https://github.com/facebookincubator/create-react-app
[helpful-article]: https://medium.com/@omgwtfmarc/deploying-create-react-app-to-s3-or-cloudfront-48dae4ce0af
