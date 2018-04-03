---
layout: post
title: "Learning Rails via Error-Driven Development"
date: 2014-07-09 13:10:52 -0400
comments: true
redirect_from: /blog/2014/07/09/learning-rails-via-error-driven-development/
categories: ["Rails", "Ruby", "Tech", "Metis", "TDD", "EDD"]
---

*This summer, I'm learning Ruby on Rails at [Metis](http://www.thisismetis.com), a 12-week class taught by some great folks from [thoughtbot](http://www.thoughtbot.com). This post is part of a series sharing my experience and some of the things I'm learning.*

As we've started learning Rails the past two weeks, one of our instructors, [Goose](https://twitter.com/halogenandtoast), has been encouraging us to code using an approach he calls **[Error-Driven Development](http://www.halogenandtoast.com/error-driven-development/)**.

Much like its older cousin, **[Test-Driven Development](http://en.wikipedia.org/wiki/Test-driven_development)**, an error-driven approach follows a short, incremental, feedback-driven cycle when coding. As a new Rails developer, the habits built through Error-Driven Development will help prepare me to learn Test-Driven Development in the future.

<!-- More -->

## What is Error-Driven Development?

To develop a feature using Error-Driven Development, we first cause Rails to generate an error. Then, we make the simplest possible change to the code in order to resolve the error. This, in turn, will raise another error, and we can iterate by solving these incremental errors until our feature is complete.

## An Example

At Metis, we've been building a sample photo gallery app, so we'll walk through what Error-Driven Development might look like for getting our first page working with some boilerplate text.

(These instructions are working from a vanilla Rails app generated with `rails new your-app-name-here`.)

To start, we need to raise an error, so we'll navigate to the `/galleries` path (which will eventually be our first page) in the browser. Rails helpfully tells us:

> Routing Error: No route matches [GET] "/galleries"

Of course, this seems silly, because we knew there was no route defined. But this message also tells us that we've created the Rails project correctly and that the server is working.

Now that we have an error, our next step is to make the smallest change to the code that fixes the error. We can do this by adding a single line to `config/routes.rb`:

```ruby
resources :galleries, only: [:index]
```

That's it. One line of code to fix this error. When we refresh the page, Rails says:

> Routing Error: uninitialized constant GalleriesController

Great. We're making progress. To solve this error, we create the `app/controllers/galleries_controller.rb` file and add this code to define the controller:

```ruby
class GalleriesController < ApplicationController
end
```

Notice that we haven't yet defined any actions for this controller. After refreshing, Rails cheerfully greets us with:

> Unknown action: The action 'index' could not be found for GalleriesController

We correct this by defining the `index` action in the galleries controller:

```ruby
def index
end
```

Refreshing again, we expect Rails to complain about a missing view. And we see (shortened for clarity):

> Template is missing: Missing template galleries/index

So we create `app/views/galleries/index.html.erb` and add some boilerplate “Hello, world!” text to see that it's working. In the browser, we finally see:

> Hello, world!

## Advantages and Reflections

One advantage of using this technique to learn Rails is that it makes apparent how Rails processes a request. For me, as a programmer who is new to Rails, this helps clarify a lot of the framework's conventions that aren't necessarily transparent (like which class names are pluralized, capitalized, etc.).

Additionally, as I've begun to get more comfortable with Rails, I can begin to anticipate what error I'll receive as I make each change to my code. If a different error appears, it's usually a sign that I made the wrong change or mistyped something. These mistakes are much easier to find when I've only changed a few lines of code; had I made multiple changes at the same time, finding that error could take longer than coding did.

In just a couple of weeks, using Error-Driven Development has completely changed the way I code. I'm programming in a more logical, incremental way, and I'm paying closer attention to my error messages. I can already see how learning Rails with an error-driven approach will provide a natural transition to using Test-Driven Development as I continue to grow as a developer.
