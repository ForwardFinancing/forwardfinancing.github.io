---
layout: post
title:  "Implementing Fuzzy Search in Elixir Using Ecto and Postgres"
date:   2017-12-20 12:00:00 -0400
categories: elixir ecto postgres fuzzy-search
author: Demi Filippou
---

## Background
  Here at Forward Financing, our Elixir single sign-on app has always stored only a small number of users. We developed only internal tools and therefore only kept track of internal users. However, now that we are expanding and developing external facing tools, the number of users is skyrocketing. This caused a problem for admins who were creating, editing, and checking for users. To check if a user exists, an admin would have to manually click through the growing number of pages of users. Yuck. This sprint, our project managers decided it was time to implement search functionality for users. We decided to go down the route of fuzzy search, because why vanilla search when you can fuzzy search?

### Prerequisites
  An Elixir app that uses Postgres and Ecto

### Where do we start?
  If we are going to write fuzzy search, we are going to need a way to calculate which strings are close enough to our query to show up in the results. After some quick investigating, we decided the best solution was to use the Levenshtein function from the [fuzzystrmatch Postgres module](https://www.postgresql.org/docs/current/static/fuzzystrmatch.html#idm46428636025488)

### Levenshtein? What is that? Why are we using it?
  Levenshtein is often referred to as "edit distance," a name that better reflects its purpose. The Levenshtein distance between two strings is essentially the number of character changes to transform string A to string B. Valid character changes are
  * Adding a character
  * Deleting a character
  * Changing a character

  A low Levenshtein distance means two strings are very similar. This makes it the perfect tool for fuzzy search because we can use it to find results similar to the search query. This will make our search tolerant to typos and small misspellings.

## The Solution
### Migration
  The first thing we want to do is add our fuzzystrmatch extension. We'll need a migration for this.

  `mix ecto.gen.migration add_fuzzystrmatch_extension`

  Make sure to define `up` and `down` rather than defining the default `change` Elixir provides, or Ecto will be unable to rollback this migration.
  ```elixir
  defmodule MyApp.Migrations.AddFuzzystrmatchExtension do
    use Ecto.Migration

    def up do
      execute "CREATE extension if not exists fuzzystrmatch;"
    end

    def down do
      execute "DROP extension if exists fuzzystrmatch;"
    end
  end
  ```
  Now we're ready for the fun part.

### Using Levenshtein
  I started off simple. Just a SQL query that we will execute using [Ecto's SQL adapter](https://hexdocs.pm/ecto/Ecto.Adapters.SQL.html#query/4). For now we will only search on email for simplicity. Let's add it to our model.

  ```elixir
  defmodule MyApp.User do

    .
    .
    .

    def fuzzy_search(query_string, threshold) do
      query = "SELECT * FROM users WHERE levenshtein(email, $1) <= $2"
      MyApp.Repo.query(query, [query_string, threshold])
    end
  end
  ```

  Test that by running something like this:
  ```elixir
  iex> MyApp.User.fuzzy_search("dfilippo@forwardfinancing.com", 5)
  {
    :ok,
    %Postgrex.Result{
    columns: ["name", "email"...],
    command: :select, connection_id: 45434, num_rows: 1,
    rows: [["Demi Filippou", "dfilippou@forwardfinancing.com"...]]}
  }
  ```

  This returns a tuple with a `Postgrex.Result` map containing the results of the query. It works! Even though my email is spelled slightly wrong, my user shows up. Yay! But we are still far from perfect.

### The Problem with Using Repo.Query
  A major flaw in this execution for me is that `Postgrex.Result` returns columns and rows, which are lists of strings, but not actual `User` structures. This makes it very inconsistent with our other user API endpoints and a nuisance to work with. This is when a fellow engineer recommended I use Ecto [fragments](https://hexdocs.pm/ecto/Ecto.Query.API.html#fragment/1).

### Fragments to the Rescue!
  Fragments are amazing. We can transform that nasty SQL query into something beautiful. Let's rewrite the bit from earlier.
  ```elixir

  defmodule MyApp.User do
    import Ecto.Query

    .
    .
    .

    def fuzzy_search(query_string, threshold) do
      query = from u in MyApp.User,
        where:
          fragment(
            "levenshtein(?, ?)",
            u.email,
            ^query_string
          ) <= ^threshold

      MyApp.Repo.all(query)
    end
  end
  ```
  Run the same test command as before
  ```elixir
  iex> MyApp.User.fuzzy_search("dfilipoop@forwardfinancing.com", 5)
  [%MyApp.User{name: "Demi Filippou", email: "dfilippou@forwardfinancing.com"...}]
  ```

  Now you should see the same results, but instead of receiving a tuple with a `Postgrex.Result`, you will get an actual user! Sweet. Plus, this code is way easier to build on and keep DRY. Let's add the ability to search by name as well, it will be super simple with fragments!
  ```elixir
  defmodule MyApp.User do
    import Ecto.Query

    .
    .
    .

    def fuzzy_search(query_string, threshold) do
      query = from u in MyApp.User,
        where:
          fragment(
            "levenshtein(?, ?)",
            u.email,
            ^query_string
          ) <= ^threshold or
          fragment(
            "levenshtein(?, ?)",
            u.name,
            ^query_string
          ) <= ^threshold
      MyApp.Repo.all(query)
    end
  end
  ```
  Now we can search on name and email, and it was so simple!
  ```elixir
  iex> MyApp.User.fuzzy_search("Demy Filippou", 5)
  [%MyApp.User{name: "Demi Filippou"...}]

  iex> MyApp.User.fuzzy_search("Foo", 5)
  [%MyApp.User{name: "Mr. Foo"...}, %MyApp.User{email: "z@foo.co"...}, %MyApp.User{name: "Foo"...},]
  ```

### Ordering Our Results
  Now, if this solution is to be of any use, we need to order our results by relevance. You can see in the previous example the user whose name was exactly "Foo" showed up last. With the way we have it written so far, there is no guarantee the ordering of our results will make sense, and the higher we set our Levenshtein threshold the more our results will be polluted, with the best results potentially being hidden at the end. So let's go ahead and order based on the smallest Levenshtein distance across email and name. We can use the Postgres function [LEAST](https://www.postgresql.org/docs/9.5/static/functions-conditional.html#FUNCTIONS-GREATEST-LEAST) for this, which selects the smallest value from a list. Let's modify the code from the last section.
  ```elixir
  defmodule MyApp.User do
    import Ecto.Query

    .
    .
    .

    def fuzzy_search(query_string, threshold) do
      query = from u in MyApp.User,
        where:
          fragment(
            "levenshtein(?, ?)",
            u.email,
            ^query_string
          ) <= ^threshold or
          fragment(
            "levenshtein(?, ?)",
            u.name,
            ^query_string
          ) <= ^threshold
        order_by:
          fragment(
            "LEAST (levenshtein(?, ?), levenshtein(?, ?)",
              u.email, ^query_string, u.name, ^query_string)
            )
      MyApp.Repo.all(query)
    end
  end
  ```
  Now your fuzzy search results should be ordered by smallest Levenshtein distance of both email and name.
  ```elixir
  iex> MyApp.User.fuzzy_search("Joe Johnson", 10)
  [%MyApp.User{name: "Joe Johnson"...}, %MyApp.User{name: "Joe Shmo"...}, %MyApp.User{email: "Pete@johnson.com"...}]
  ```

### Case Insensitivity
  Bad news for us! The Levenshtein function is case sensitive. This means using any caps casing at all, either in our query or in our user's name or email fields, will be driving up the Levenshtein distance. So if we had a user whose name was "demi filippou", and we did this...
  ```elixir
  iex> MyApp.User.fuzzy_search("DEMI FILIPPOU", 5)
  ```
  ...we'd get no results! The Levenshtein distance here would actually be the length of my name, because according to Levenshtein, D and d are different characters, as are E and e, and so on. I can't imagine how case sensitivity in fuzzy search would be helpful, so let's fix that. We can use Postgres [LOWER](https://www.postgresql.org/docs/9.5/static/functions-string.html#FUNCTIONS-STRING-SQL) for this. We can downcase our query string, and downcase the name and email of the users we are searching to ensure the case sensitivity of Levenshtein doesn't get in our way.
  ```elixir
  defmodule MyApp.User do
    import Ecto.Query

    .
    .
    .

    def fuzzy_search(query_string, threshold) do
      query_string = query_string |> String.downcase
      query = from u in MyApp.User,
        where:
          fragment(
            "levenshtein(LOWER(?), LOWER(?))",
            u.email,
            ^query_string
          ) <= ^threshold or
          fragment(
            "levenshtein(LOWER(?), LOWER(?))",
            u.name,
            ^query_string
          ) <= ^threshold
        order_by:
          fragment(
            "LEAST (levenshtein(LOWER(?), LOWER(?)), levenshtein(LOWER(?), LOWER(?))",
              u.email, ^query_string, u.name, ^query_string)
            )
      MyApp.Repo.all(query)
    end
  end
  ```
  ```elixir
  iex> MyApp.User.fuzzy_search("DEMI FILIPPOU", 5)
  [%MyApp.User{name: "Demi Filippou", email: "dfilippou@forwardfinancing.com"...}]
  ```
### Leveraging Macros to DRY it up
  It's looking good so far! But once you see how awesome fuzzy search is, you might want to search on more fields. We decided to search on first name, last name, full name, email address, and the domain name in the email. It worked great, but our original implementation with the fragments got very repetitive, and we ended up having something like this code block five times:
  ```elixir
   fragment(
            "levenshtein(LOWER(?), LOWER(?))",
            u.email,
            ^query_string
          ) <= ^threshold 
  ```
  Not to mention repeating ourselves again and again in the `order_by`. Ugh. I do *not* want to see any more LOWERs. So let's make it more dynamic! I took the advice of the Hex Docs and wrote some macros to [expose the Levenshtein function](https://hexdocs.pm/ecto/Ecto.Query.API.html#fragment/1-defining-custom-functions-using-macros-and-fragment). This made it neater, more readable, and more dynamic. Win.

  So go ahead and throw these macros in a helper module or in your model:
  ```elixir
    # Macro that takes two strings and determines if the levenshtein distance
    # between them is less than the given threshold
    defmacro levenshtein(str1, str2, threshold) do
      quote do
        levenshtein(unquote(str1), unquote(str2)) <= unquote(threshold)
      end
    end

    # Wrapper for SQL levenshtein function, which gets the levenshtein distance
    # between str1 and str2
    # SQL levenshtein is case-sensitive, so we downcase everything to make it
    # case-insensitive.
    defmacro levenshtein(str1, str2) do
      quote do
        fragment(
          "levenshtein(LOWER(?), LOWER(?))",
          (unquote(str1)),
          (unquote(str2))
        )
      end
    end
  ```

  Now, we can really clean up the fuzzy search function. Make sure to import your module with your macros if you decided to separate them.
  ```elixir
  defmodule MyApp.User do
    import Ecto.Query

    .
    .
    .

    def fuzzy_search(query_string, threshold) do
      query_string = String.downcase(query_string)
      query = from u in User,
        where:
          levenshtein(u.email, ^query_string, ^threshold) or
          levenshtein(u.name, ^query_string, ^threshold)
      order_by:
        fragment(
          "LEAST(?, ?)",
          levenshtein(u.email, ^query_string),
          levenshtein(u.name, ^query_string)
        )
    MyApp.Repo.all(query)
  end
end
```
The functionality is identical but now it's super easy to search on more fields. And that's it - you now have a fully functional `fuzzy_search` function that returns search results under a given Levenshtein threshold!
