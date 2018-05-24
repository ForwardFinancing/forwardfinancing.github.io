---
layout: post
title:  "Working with object oriented and functional programming without losing your mind."
date:   2018-04-26 12:00:00 -0400
categories: elixir coding ruby oo functional
author: Tania Paiva
---

## Working with object oriented and functional programming without losing your mind.

At Forward Financing we are always going forward, that's why our technology stack is pretty mixed.
We work with Rails, Phoenix, React, Redux among others, that means as developers we are changing the
development environment sometimes more than once at day.

Sometimes in the morning you are developing a new feature in the elixir-phoenix environment, and at the
afternoon we are improving existing code made on Ruby on Rails.

The main thing with this "switching" is the programming paradigm that each of this technologies uses is totally
different.

### Object Oriented Programming (OOP) vs Functional Programming (FP)

A program has two main components, the data and the behavior.
The main difference between OOP and FP is how the manage these components.

OOP puts together data and its associated behavior in a single place called object. For FP, the data and behavior are  different things and should be separated.

FP tries to avoid sharing state and mutation on data. The objective of FP is building pure functions with no side effects.

OOP provides properties and methods for objects, they live inside the class structure, an object has a class and we can
have an instance of it.

Let's see a practical example, we need to implement a program that shuffle cards on a deck.

Object Oriented approach 
```
class Card
  def new(value)
    @value = value
  end

  def value
    return @value
  end

end

class Deck
  def new(values)
    @cards = []
    values.each |v| do
      @cards << Card.new(v)
    end
  end

  def shuffle
    
  end

  def contains?
  
  end

end
```

Functional approach
```
Module Cards
  def create_deck
  end

  def shuffle_deck
  end

  def contains?
  end
end
```
