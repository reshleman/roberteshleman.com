---
layout: post
title: "Private Class Methods in Ruby"
date: 2014-08-22 11:17:04 -0400
comments: true
categories: ["Ruby", "OO", "Object-Oriented", "Tips"]
---

*This summer, I'm learning Ruby on Rails at [Metis], a 12-week class taught by some great folks from [thoughtbot]. This post is part of a series sharing my experience and some of the things I'm learning.*

[Metis]: http://www.thisismetis.com
[thoughtbot]: http://www.thoughtbot.com

File this one under, "fool me once, shame on you; fool me twice, write a blog post."

In Ruby we use the [`private` method](http://www.ruby-doc.org/core-2.1.2/Module.html#method-i-private) to define instance methods that are only accessible from within a class:

<!-- More -->

```ruby
class HammerTime
  # Arbitrary public methods omitted

  private
  
  def stop
    puts "Can't touch this"
  end
end
```

Calling `stop` on an instance of the class `HammerTime` from somewhere other than within the class itself will result in an error:

```ruby
HammerTime.new.stop
# => NoMethodError: private method `stop' called for #<HammerTime:0x00>
```

Given this, we might expect the same syntax to work for *class methods*:

```ruby
class I
  # Arbitrary public methods omitted

  private

  def self.wanna
    puts "...dance with somebody"
    puts "...feel the heat with somebody"
  end
end
```

Surely we'll get the same `NoMethodError`, right? Wrong:

```ruby
I.wanna
# => ...dance with somebody
# => ...feel the heat with somebody
```

Instead, we'll need to explicitly define this method as a [`private_class_method`](http://www.ruby-doc.org/core-2.1.2/Module.html#method-i-private_class_method), like this:

```ruby
class Domo
  # Arbitrary public methods omitted

  def self.arigato
    puts "Secret secret, I've got a secret"
  end
  private_class_method :arigato
end
```

And we'll get the expected error, indicating this method is only accessible from within the class:

```ruby
Domo.arigato
# => NoMethodError: private method `arigato' called for Domo:Class
```

*Caveat emptor.*