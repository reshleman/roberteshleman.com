---
layout: post
title: "The Value of Learning Object-Oriented Design in Ruby... Starting from Day 1"
date: 2014-06-23 12:56:03 -0400
comments: true
redirect_from: /blog/2014/06/23/the-value-of-learning-object-oriented-design-in-ruby-dot-dot-dot-starting-from-day-1/
categories: ['Ruby', 'Tech', 'Metis']
---

*This summer, I'm learning Ruby on Rails at [Metis](http://www.thisismetis.com), a 12-week class taught by some great folks from [thoughtbot](http://www.thoughtbot.com). This post is the first in a series sharing my experience and some of the things I'm learning.*

Using object-oriented design in programming is analogous to building scaffolding around a building. Once we have a basic class definition that models a real-world object, adding additional layers of abstraction on top becomes much simpler. This allows us to easily build on more complexity — or levels on the scaffold — without sacrificing readability. A procedural approach is like using a steep rickety ladder to scale the same tall building: you may make it to the top, but it's a much more difficult climb and the ladder is liable to collapse, dropping you right back to the ground to start over.

<!-- More -->

## The Problem

During the first week at Metis, we began to work as a group on a basic Ruby program that would choose a random integer between 1 and 10, ask the user for a guess, and output a string indicating whether or not the guess was correct.

As a class, we pseudo-coded our program like this:

1. Choose the correct answer
2. Ask the user to make a guess
3. Get the guess
4. Compare the guess and the answer
5. Provide feedback

## The Procedural Solution

Despite having been away from programming for a few years, I had a pretty good idea of how I would approach this problem. Had I put together a solution before we worked through this exercise as a group, I probably would have come up with something like this:

```ruby
answer = rand(1..10)
print "Please guess a number: "
guess = gets.to_i
if answer == guess
  puts "That’s correct!"
else
  puts "Incorrect. The answer was " + answer.to_s + "."
end
```

## The Object-Oriented Solution

With that solution in mind, I was somewhat surprised when our instructor guided us through building an object-oriented program for this scenario. It seemed strange to me that we would jump into object-oriented code so quickly; most people in the class did not yet have much exposure to Ruby syntax, let alone to more complex topics like object instantiation and the difference between instance and class methods.

The object-oriented solution we arrived at as a class looked something like this:

```ruby
class GuessingGame
  def initialize
    @answer = rand(1..10)
    @guess = nil
  end

  def play
    make_guess
    print_result
  end

  private

  attr_accessor :answer, :guess

  def make_guess
    print "Please guess a number: "
    @guess = gets.to_i
  end

  def print_result
    if answer == guess
      puts "That's correct."
    else
      puts "Incorrect. The answer was #{answer}."
    end
  end
end

GuessingGame.new.play
```

At first glance, this code seems to add additional complexity (and 22 additional lines!) to a problem with a somewhat trivial implementation. But once I began to see the patterns in how objects are defined in Ruby, it became apparent that the object-oriented code was much more elegant, even for such a simple program.

This explanation of what a class is really helped to make this clear: *a class is a __definition__ of behavior and state for a real-world entity.* It doesn't actually *do* anything until an object is instantiated.

In this case, defining the Guessing Game logic in its own class allows the body of the program to consist of a single line: `GuessingGame.new.play`. Abstracting `make_guess` and `print_result` out of the `play` method and into separate instance methods makes this code much easier to understand on first read — because we're relying less on Ruby syntax and more on words from the real-world domain to explain what's happening.

## Extending the Program

The most significant benefit of an object-oriented approach comes when we extend the Guessing Game program with additional functionality: allow the user to play multiple rounds of the guessing game and print the total number of rounds won.

In this example, it would be possible to extend the procedural program to add these new features, but it could quickly become unwieldy and difficult to read, requiring several extra variables and a loop all mashed together. (This becomes even more true as we continue to add additional functionality by, say, allowing multiple guesses in each round and printing the average number of guesses taken by the user each round.)

Extending the object-oriented solution still requires additional variables and a loop, but I'd argue that it's easier to write, and we can again abstract out some of the syntactic complexity to maintain readability. Here's the final program:

```ruby
# Note that we've renamed the GuessingGame class from the previous example to be GuessingGameRound.
class GuessingGameRound
  def initialize(round_number)
    @answer = rand(1..10)
    @guess = nil
    @round_number = round_number
    @won = false
  end

  def play
    print_round_number
    make_guess
    print_result
  end

  def won?
    won
  end

  private

  attr_accessor :answer, :guess, :round_number, :won

  def print_round_number
    puts "===Round ##{round_number}==="
  end

  def make_guess
    print "Please guess a number: "
    @guess = gets.to_i
  end

  def print_result
    if answer == guess
      @won = true
      puts "That's correct."
    else
      puts "Incorrect. The answer was #{answer}."
    end
  end
end

class GuessingGame
  def initialize(round_count)
    @round_count = round_count
    @rounds_won = 0
  end

  def play
    introduction
    play_rounds
    print_statistics
  end

  private

  attr_accessor :round_count, :rounds_won

  def introduction
    puts "Welcome to the Guessing Game. There are #{round_count} rounds."
  end

  def play_rounds
    1.upto(round_count) do |round_number|
      round = GuessingGameRound.new(round_number)
      round.play
      if round.won?
        @rounds_won += 1
      end
      puts
    end
  end

  def print_statistics
    puts "You won #{rounds_won} out of #{round_count} rounds."
  end
end

GuessingGame.new(5).play
```
