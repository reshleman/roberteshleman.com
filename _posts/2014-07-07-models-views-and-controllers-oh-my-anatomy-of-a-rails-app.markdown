---
layout: post
title: "Models, Views, and Controllers, Oh My!: Anatomy of a Rails App"
date: 2014-07-07 14:14:17 -0400
comments: true
redirect_from: /blog/2014/07/07/models-views-and-controllers-oh-my-anatomy-of-a-rails-app/
categories: ["Rails", "Tech", "MVC", "Metis", "Ruby"]
---

*This summer, I'm learning Ruby on Rails at [Metis](http://www.thisismetis.com), a 12-week class taught by some great folks from [thoughtbot](http://www.thoughtbot.com). This post is part of a series sharing my experience and some of the things I'm learning.*

With [apologies to the late Judy Garland](http://www.youtube.com/watch?v=NecK4MwOfeI), let's start out by acknowledging that the [Model-View-Controller architecture](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) isn't nearly as frightening as lions, tigers, or bears. After all, we *are* wearing our Ruby slippers.

\**crickets*\*

Anyway, bad jokes aside, this way of organizing an application *can* seem intimidating to the uninitiated. In this post, we’ll take a look at what MVC means, and what this architecture looks like in a Rails application.

<!-- More -->

## Overview

At a high level, the Model-View-Controller architecture allows us to isolate logic into different categories (and files), based on that code’s role in a program. This standard structure also makes it easier to read through existing code, because you know where to find specific logic based on what it does.

We’ll start with some basic definitions:

* **Models** contain the *business logic* — this is the code that maps the real-world problem space to its representation in the computer.
* **Views** contain *presentation logic* — or the code that determines how data will be displayed to the user.
* **Controllers** contain *control logic* — the connective tissue that makes models and views work together.

Let's explore these components in more detail.

## Models

A model contains all of the code that matches the behavior and state of a real-world entity. This is what we call the "business logic" — code that is motivated by the the actual "things" we're modeling.

When matching our business logic to the problem space, we follow a convention called “Fat Models, Skinny Controllers.” This means that controllers should not contain any logic related to modeling the real world — that’s solely the job of the models. Instead, controllers should only contain the basic code required to construct and send a response to the user.

Often, but not always, models will correspond to an entity that we want to persist in the database. For these models, we use an [object-relational mapping](http://en.wikipedia.org/wiki/Object-relational_mapping) (or ORM), which matches database fields to data stored in Ruby objects. This type of mapping provides methods that allow us to easily persist our objects in the database.

In Rails, models are stored in the `app/models/` directory, and named in the singular form, like `gallery.rb`. ORM model classes inherit functionality from `ActiveRecord::Base`. 

## Views

Views contain all of the code that is required to display information to the user. In Rails, these files often take the form of an HTML file with embedded Ruby:

```erb
<h1><%= @user.name %></h1>
```

These `.html.erb` files are stored in a `app/views/` inside a subdirectory that matches the pluralized model name. So the “index” view for an “image” model would be located at `app/views/images/index.html.erb`.

### Layouts

Rails also uses layouts that can contain the boilerplate HTML code used for every page. This default file is located at `app/views/layouts/application.html.erb`, and all other views are rendered where the `<%= yield %>` call appears in the layout.

### Guidelines for Views

The Ruby logic in views should be somewhat simple and restricted as much as possible to presenting information. There are a few guidelines here:

* **Don’t** assign variables
* **Don’t** define classes
* **Don’t** define methods
* **Don’t** define constants
* **Avoid** complex conditionals (`if`, `case`, etc.)
* **Avoid** complex method calls

## Controllers

The best analogy I’ve heard for how controllers work is a comparison to an old-school (read: before my time) [telephone switchboard](http://en.wikipedia.org/wiki/Telephone_switchboard) operator. When a caller needs to reach another person, they place their request with the switchboard operator, who connects the right cables so the call can be completed.

Controllers in Rails work similarly. When a user submits a request to a Rails application, that request it is **routed** to a specific **action** method within a controller. (More on these in a moment.) This method contains logic that collects information (often — but not necessarily — using a model) and passes that data to a view. Finally, the controller sends the rendered view to the user.

Each request sent to the server will only be handled by a single action within a single controller.

### Routes

Routes are not technically part of the MVC architecture, but they play a key role in how controllers work. A route defines a path that the user can use to interact with an application. Rails defines routes in the `config/routes.rb` file, using the following syntax:

```ruby
verb "/route" => "controller#action"
```

In this syntax:

* `verb` represents one of the [HTTP verbs](http://www.restapitutorial.com/lessons/httpmethods.html)
* `"/route"` is the URL path, relative to the application root, like `/gallery/1/image/45`
* `controller` is the name of a specific controller
* and `action` is the name of a specific action method within that controller

### Actions

An action is a method in a controller that a route points to. When called, an action gathers the data necessary to render a view, and then returns a rendered view to the user.

In Rails, the convention is to use only these [seven actions](http://guides.rubyonrails.org/routing.html#crud-verbs-and-actions), which represent the standard ways of interacting with a resource:

* Index — display some or all resources of that type
* Show — display a single resource
* New — display a form for creating a new resource
* Create — actually create that resource
* Edit — display a form for changing a resource
* Update — actually change that resource
* Destroy — delete a resource

Rails even provides an easy, standard way to define routes to all of these resources in `config/routes.rb`. For a sample photo gallery application, this might look like:

```ruby
resources :galleries
```

This provides the following routes (table adapted from the Rails [Routing Guide](http://guides.rubyonrails.org/routing.html#crud-verbs-and-actions)):

**HTTP Verb** | **Path** | **Controller#Action** | **Used for**
--- | --- | --- | ---
GET | /galleries | galleries#index | display a list of all galleries
GET | /galleries/new | galleries#new | display a form for creating a new gallery
POST | /galleries | galleries#create | create a new gallery
GET | /galleries/:id | galleries#show | display a specific gallery
GET | /galleries/:id/edit | galleries#edit | display a form for editing a gallery
PATCH/PUT | /galleries/:id | galleries#update | update a specific gallery
DELETE | /galleries/:id | galleries#destroy | delete a specific gallery

You can also use the `resources` method to quickly define a subset of these routes:

```ruby
# Only
resources :galleries, only: [:index, :show]
```

```ruby
# Except
resources :galleries, except: [:edit, :update, :destroy]
```

## Further Reading

The [Rails Guides](http://guides.rubyonrails.org/) contain a ton of additional information about how MVC is implemented in Rails, including API references, usage, and code examples. There are specific pages covering [models](http://guides.rubyonrails.org/active_record_basics.html), [views](http://guides.rubyonrails.org/layouts_and_rendering.html), [controllers](http://guides.rubyonrails.org/action_controller_overview.html), and [routing](http://guides.rubyonrails.org/routing.html). The [Rails API](http://api.rubyonrails.org/) is also helpful for more precise information about available methods.
