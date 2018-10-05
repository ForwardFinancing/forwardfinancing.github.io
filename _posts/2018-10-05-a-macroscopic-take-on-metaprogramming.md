---
layout: post
title:  "A macroscopic take on metaprogramming"
date:   2018-10-05 17:00:00 +0100
categories: elixir ruby javascript
author: Natacha Segala
---
If you are as keen on DIY as I am and equally as helpless, the first time you're presented with some sort of power tool you may think 'This is scary, how could I possibly use this?'.

In some respects, metaprogramming can feel just like this.

A quick look at [wikipedia's definition](https://en.wikipedia.org/wiki/Metaprogramming){:target= "_blank"} gives us:
"Metaprogramming is a programming technique in which computer programs have the ability to treat programs as their data."

Another common definition is "writing code that writes code".

One of the main unknowns when coding is user data.
Everyday code does not necessarily solve unique problems.
As developers, we use well defined conventions and set of functions we are familiar with to process this data and return whatever is defined in the product specifications.

There should be room for abstraction and generalisation and yet, the consensus is to try and avoid metaprogramming as much as possible.

The prefix "meta" is self-referential which tends to indicate linguistically that metaprogramming encompasses all of programming including "normal" programming.
We could think of "normal" programming as our ability to solve unique problems.
All of the other kinds of general problems could – in theory – be handled by metaprogramming.

Getting to know the reflective methods and tools every language we use offers is a way of refining one's craft.
It also offers a glimpse into and a better understanding of the internals of the language.

## Ruby

When working with Ruby, chances are you will come across the DRY (don't repeat yourself) paradigm.

The idea of avoiding duplication seems like a sound one in a more general context too.
It will make any refactoring (or change) easier.

Not a cure-all but definitely a tool to keep handy in one's dev belt.

### Example

We have a test suite responsible for checking the index page of our website includes the content we expect:

```
class IndexTest < TestCase
  test_homepage_has_about_content do
    visit '/'
    assert page.has_content?(about)
  end

  test_homepage_has_news_content do
    visit '/'
    assert page.has_content?(news)
  end

  ...
end
```

A quick online search or pairing could lead to reducing the code to:

```
class IndexTest < TestCase
  HOMEPAGE_CONTENT = ['About', 'Meetups', 'Tweets', 'News'].freeze

  HOMEPAGE_CONTENT.each do |content|
    define_method("test_homepage_has_#{content}_content") do
      visit '/'
      assert page.has_content?(content)
    end
  end
end
```

What does define_method do exactly and where does it come from?

Running `IndexTest.methods` returns a lot of methods we have not defined ourselves.
This is where the concept of inheritance starts to be visible.

This call returns a list which includes all the methods accessible in the object's ancestors.

If we want to know how many ancestors the IndexTest class has, we can run:
```
IndexTest.ancestors
=> [IndexTest, Object, Kernel, BasicObject]
```

If you check each ancestor individually, `define_method` or even `ancestors` are nowhere to be found.

This is because inheritance is not limited to direct "vertical" precedence.
`BasicObject`, `Object`, and `IndexText` are all classes and they have Class as their class.
All of them are instances of Class. Class ancestors are:

```
Class.ancestors
=> [Class, Module, Object, Kernel, BasicObject]
```

Module is where `define_methods` and `ancestors` are defined.
Understanding inheritance in Ruby is key to using the language itself better.

A lot of Ruby's so-called "magic" exists in fact due to metaprogramming.

## Javascript

es6 has introduced a new feature called Proxy – which lo and behold – is an application of the [proxy pattern](https://en.wikipedia.org/wiki/Proxy_pattern){:target= "_blank"}

In short, it acts as an interface to either just forward to the original recipient or apply additional logic.

### Example

We're writing a library to use with our frontend applications.
It will offer functions to report a set of events to a couple of external apis.
These APIs are in turn responsible for dispatching these to our internal messaging systems.

We have an object responsible for dispatching the event to the first external API.

```
import externalApi from 'external-api';
let isInitialized;

const FirstExternalApi = {
  _getDispatcher: () => {
  return (method, ...args) => {
  return externalApi.push([ method, ...args]);
};
},

dispatch: function(event) {
  if (isInitialized) {
    const { dispatchMethodName } = event;

    this[dispatchMethodName](event);
  }
},

_dispatchEvent: function({ method, eventAttributes={ } }) {
  this._getDispatcher()(method, eventAttributes);
},

_dispatchException: function({eventAttributes}) {
  this._dispatchEvent({ method: 'SHOUT', eventAttributes });
},

[...]
}
```

In a development or staging environment, we want to be able to view the output of _getDispatcher in real time.
We think it would avoid context switching and help speed up things a bit.

We implement a mock object to output the event to the console.
We deep clone the FirstExternalApi object and set the '_getDispatcher' to ouput to the console.

```
function clone() {
 // deep clone logic
}

const FirstExternalApiMock = clone(FirstExternalApi);

FirstExternalApiMock._getDispatcher = function() {
  return (method, ...args) => {
    const argsString = args.map(arg => {
      return JSON.stringify(arg, null, 4);
  }).join(', ');

    console.log(`externalApi('${method}', ${argsString})`);
  };
};
```

Based on the node environment, we have an object responsible for trickling our events down to:
- either the real FirstExternalApi object responsible for communicating with the external api
- or the mock object

Another way to look at this problem would be to wonder if we could hijack this `_getDispatcher` function and mock it directly at the "source".

es6 proxies allow just that.
They offer a way to catch just what you need of the behaviour or add to it.

So in our example, we replace the mock with the below:
```
const FirstExternalApiMock = new Proxy(FirstExternalApi, {
  get(target, propKey) {
    if (propKey === '_getDispatcher') {
      return () => {
        return (method, ...args)=> {
          const argsString = args.map(arg => {
            return JSON.stringify(arg, null, 4);
        }).join(', ');

          console.log(`externalApi: ('${argsString}')`);
        }
      };
    } else {
      return target[propKey];
    }
  }
});
```

We pass the object we are interested in.
'get' is responsible for 'listening' to whenever a property of this object is called.
Now we leave all properties but one alone and focus entirely on 'trapping' `_getDispatcher`.
When it's called, we output the arguments to the console.

It's a different, slightly more concise more way of implementing our solution.

## Elixir

In Elixir, most of the metaprogramming boils down to understanding the functions "quote" and "unquote" (by no means a measure of simplicity).
Quote effectively gives us access to Elixir's AST (abstract syntax tree) by showing us how an expression is parsed and transformed into a tree that Elixir can handle.
What is returned typically contains three-element tuples (operation/data type, metadata, arguments/data).

### Example

Again, we would like to make our test suite a bit more concise.
Our application offers two endpoints with a very similar set of functionality.
This should be a good case for abstracting most of their testing and sharing the tests between the two endpoints (endpoint_a and endpoint_b).
Not dissimilar to RSpec shared example feature, making use of a domain specific language would work for our purposes.

We have a series of tests like the one below:

```
defmodule Api.V1.AControllerTest do
	test "#create http returns a 403 status with json error" do
	  "/v1/endpoint_a"
	  |> create_record(403, ssl_error())
	end
end

defmodule Api.V1.BControllerTest do
	test "#create http returns a 403 status with json error" do
	  "/v1/endpoint_b"
	  |> create_record(403, ssl_error())
	end
end

```

The only difference being the endpoint, we can abstract this to:

```
defmodule SharedTestCase do
  use ExUnit.Case

  @moduletag endpoint: "endpoint_a"

  test "#create http returns a 403 status with json error" do
  		"/v1/#{endpoint}"
  		|> create_record(403, ssl_error())
	end
end
```

@moduletag is a module attribute provided by ExUnit: one of its uses being to act as a temporary module storage to be used during compilation.

A macro is going to help Api.V1.AControllerTest and Api.V1.BControllerTest modules make use of the SharedTestCase module.

```
defmodule SharedTestCase do
  defmacro __using__(options) do
    quote do
      use ExUnit.Case

      @moduletag unquote(options)

      test "#create http returns a 403 status with json error", %{endpoint: endpoint} do
        "/v1/#{endpoint}"
        |> create_record(403, ssl_error())
      end

      ...
    end
  end
end
```

It's possible to go even further by extracting all the tests to another macro to define the tests separately.


Metaprogramming is not a bag of tricks, it's the path to a better deeper understanding of any language and potentially more expressive code.

Languages like Elixir give the ability to the programmer of making extensions tailored to their needs which are first-class features.

It's too good an opportunity to pass up and not use for the greater good of generalisation, DRY-ing your codebase and making the code more expressive.

If this seems daunting at first, chances are you're like me.
This means one can start from a long redundant looking piece of code they have just written, feel an urge to make it better, and little by little, abstract away as much as possible.

If this doesn't seem daunting, it should simply lead to think more and code less.


