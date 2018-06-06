---
layout: post
title:  "Working With Object Oriented and Functional Programming Without Losing Your Mind"
date:   2018-04-26 12:00:00 -0400
categories: elixir coding ruby oo functional
author: Tania Paiva
---

At Forward Financing we work with several technologies, our stack is pretty mixed.
Rails, Phoenix, React, Redux just to name a few - which means we have to switch from one development tool or environment to the next several times a day on a regular basis.

In practice, this means you could find yourself developing a new feature in the Elixir-Phoenix environment in the morning, and improving existing Ruby on Rails code in the afternoon.
The main thing with this switching is the programming paradigm that each of this technologies uses is totally
different.

### Object Oriented Programming (OOP) and Functional Programming (FP)

A program has two main components: the data and the behavior.
The main difference between OOP and FP is how we manage these components.

OOP puts together data and its associated behavior in a single place called an `Object`. For FP, the data and behavior are different things and should be separated.

FP tries to avoid sharing state and mutation on data. The objective of FP is building pure functions with no side effects.

OOP provides properties and methods for objects, they live inside the class structure, an object has a class and we can
have an instance of it.

A function is a piece of code that is called by name. It can be passed data on which to operate (parameters) and can optionally return data (the most common practice is to return something). All data that is passed to a function is explicitly passed.

A method is a piece of code that is called by a name that is associated with the Object.

Let's see a practical example, *and imagine* we need to implement a program that shuffles cards on a deck.

### Object Oriented Approach
``` ruby
class Card
  def initialize(value)
    @value = value
  end

  def value
    @value
  end

end

class Deck
  def initialize(values)
    @cards = []
    values.each do |v|
      @cards << Card.new(v)
    end
  end

  def shuffle_deck
    @cards = @cards.shuffle
  end

  def contains?(card_value)
    @cards.any? do |card|
      card.value == card_value
    end
  end

  def show_cards
    @cards.map(&:value).join(', ')
  end

end

> deck = Deck.new(['King of Spades', 'Ace of Heart', 'Two of Diamonds'])
<Deck:0x00007fb03e082208 @cards=[#<Card:0x00007fb03e082190 @value="King of Spades">, #<Card:0x00007fb03e082168 @value="Ace of Heart">, #<Card:0x00007fb03e0820c8 @value="Two of Diamonds">]>
> deck.show_cards
King of Spades, Ace of Heart, Two of Diamonds
> deck.shuffle_deck
> deck.show_cards
Ace of Heart, Two of Diamonds, King of Spades
> deck.contains?('Two of Clubs')
false
```

### Functional Approach
``` elixir
Module Cards
  def create_deck(list_of_values)
    list_of_values
  end

  def shuffle_deck(deck)
    Enum.shuffle(deck)
  end

  def contains?(deck, card)
    Enum.member?(deck, card)
  end
end

> deck = Cards.create_deck(["King of Spades", "Ace of Heart", "Two of Diamonds"])
["King of Spades", "Ace of Heart", "Two of Diamonds"]
> Cards.contains?(deck, "Two of Clubs")
false
> deck = Cards.shuffle_deck(deck)
["Ace of Heart", "King of Spades", "Two of Diamonds"]

```

As we can see the OOP takes us to separate the elements in objects (Card and Deck) as much as we can, we can compare an object with an atom, as the smallest structure on the ecosystem.

In the FP we can see the approach is more direct.

Of course both FP and OOP have their pros and cons, but we are not putting the focus on that in this post.

In OOP the value of the object mutates when the methods are being executed, for example shuffle_deck it saves
the new deck shuffled.
Also in OOP the object is the one that calls the methods, on FP the functions are executed by the module, and the
original value is not changed unless we assign that value.
Functions and methods are pretty similar, both do "stuff" with the data and define the behavior, except in the following examples:

- A method is implicitly passed the object on which it was called.

- A method is able to operate on data that is contained within the class (remembering that an object is an instance of a class - the class is the definition, the object is an instance of that data).

This is a pretty simple example of the differences between the structure of a program using these two paradigms, and sometimes switching the state from OOP to FP takes a little time. That's why it is necessary to know the basics of each
one.