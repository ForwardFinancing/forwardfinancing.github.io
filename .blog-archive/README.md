# tech.forwardfinancing.com

This is our company's tech blog.

## Development

To start a local server run the following:

```sh
bundle exec jekyll serve --future
```

To add a new post simply create a new file in the `_posts`
directory with a filename starting with the current `date
--iso-8601`, then the title. For example
`2019-04-03-creating-a-new-post.md`. It should have the following YAML Front
Matter at the beginning of the file.

```yaml
---
layout: post
title:  ""
date:   1970-01-01
categories: one two another
author: Bob Smith
---
```
