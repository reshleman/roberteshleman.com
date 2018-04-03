---
layout: post
title: "Using Rails Namespaces for Admin Actions"
date: 2014-08-14 17:17:56 -0400
comments: true
redirect_from: /blog/2014/08/14/using-rails-namespaces-for-admin-actions/
categories: ["Rails", "Ruby", "Tech", "How-to"]
---

*This summer, I'm learning Ruby on Rails at [Metis], a 12-week class taught by some great folks from [thoughtbot]. This post is part of a series sharing my experience and some of the things I'm learning.*

[Metis]: http://www.thisismetis.com
[thoughtbot]: http://www.thoughtbot.com

Rails has several strategies to help us separate concerns in our applications. We use separate files and directories for models, views and controllers, for example, and we can nest routes based on how the resources are related to each other.

In addition, Rails permits the use of [namespaces] to organize our resources and prevent naming conflicts. In this post, we'll take a look at why this feature is useful by implementing a namespace for admin actions in a sample [Craigslist] clone Rails app.

[namespaces]: http://en.wikipedia.org/wiki/Namespace
[Craigslist]: http://craigslist.org

<!--More-->

## Routing

In our version of Craigslist, our app has many categories (i.e., "bikes," "boats," "missed connections," etc.), each with many posts. To start out, our routes look something like this:

```ruby
resources :categories, only: [:index, :show] do
  resources :posts, only: [:new, :create, :show]
end
```

We want to give admin users the ability to add categories, so we add the following to our `routes.rb`:

```ruby
namespace :admin do
  resources :categories, only: [:new, :create]
end
```

This gives us routes under the path `/admin/categories` which are directed to `Admin::CategoriesController`.

## The Controller

There are a few considerations we have for our namespaced controller.

First, Rails expects this controller file to exist in an `admin/` folder within `app/controllers/` (i.e., at `app/controllers/admin/categories_controller.rb`).

Second, because the controller is named within the `admin` namespace, we use the [scope resolution operator] to define the controller class:

```ruby
class Admin::CategoriesController < ApplicationController
  # Methods omitted
end
```

From there, we can define our `new` and `create` actions as we normally would.

Note, also, that we still have our non-namespaced `categories` routes (for `show` and `index`) that direct requests to the standard `CategoriesController` in `app/controllers/`.

[scope resolution operator]: http://en.wikipedia.org/wiki/Scope_resolution_operator

## Views

Views that correspond to our `Admin::CategoriesController` actions are namespaced just like the controller is. When Rails needs these view files, it will look in the `app/views/admin/` subfolder.

Additionally, we'll need to point our path helpers to the right place for namespaced routes. For example, we'll use paths like `new_admin_category_path` and `form_for [:admin, @category]` in our views. 

## Restricting Access to Admin Actions

So far, we have our admin actions working within the namespace, but we're not actually restricting access to these actions.

There are a couple of approaches here. We'll start by adding a simple `before_action` to the `Admin::CategoriesController`:

```ruby
class Admin::CategoriesController < ApplicationController
  before_action :require_admin

  # Methods omitted

  def require_admin
    unless current_user.admin?
      redirect_to root_path
    end
  end
end
```

We have a `current_user` method already defined by our authentication system. (I've been quite happy using [Monban] recently.) Additionally, our `User` model has a boolean `admin` field, so we can simply redirect if the user isn't an admin.

## Refactoring

This approach works, but it's not very flexible. If we add other admin actions (say, for filtering posts flagged as spam), we have to redefine this `require_admin` method in each namespaced controller.

One solution might be to define the `require_admin` method in our `ApplicationController`. That's a step in the right direction, but we would have to remember to add the `before_action` to each admin controller; these controllers wouldn't be secure by default.

The solution I prefer is to define a separate class (in `app/controllers/`) called `AdminController` that inherits from `ApplicationController`:

```ruby
class AdminController < ApplicationController
  before_action :require_admin

  def require_admin
    unless current_user.admin?
      redirect_to root_path
    end
  end
end
```

Now, each namespaced admin controller can inherit directly (and quite appropriately) from `AdminController`. Since we've defined the `before_action` in `AdminController`, actions in any sub-classed controller will be restricted to admin users by default.

We no longer need to list the `before_action` or define a `require_admin` method in each of our namespaced controllers; they just inherit from `AdminController`:

```ruby
class Admin::CategoriesController < AdminController
  # Methods omitted
end
```

Simple, effective, and scalable. By extracting the `before_action` and `require_admin` logic into a separate parent controller, we've made it startlingly easy to add new admin actions in the future.

We'll thank ourselves later.

[Monban]: https://github.com/halogenandtoast/monban

## Resources

* [Rails Guide on Routing](http://guides.rubyonrails.org/routing.html#controller-namespaces-and-routing)
* [Monban User Authentication](https://github.com/halogenandtoast/monban)
