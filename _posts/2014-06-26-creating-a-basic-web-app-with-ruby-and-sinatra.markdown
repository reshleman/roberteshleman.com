---
layout: post
title: "Creating a Basic Web App with Ruby and Sinatra"
date: 2014-06-26 12:45:25 -0400
comments: true
redirect_from: /blog/2014/06/26/creating-a-basic-web-app-with-ruby-and-sinatra/
categories: ["Metis", "Tech", "Ruby", "Sinatra", "How-to"]
---

*This summer, I'm learning Ruby on Rails at [Metis](http://www.thisismetis.com), a 12-week class taught by some great folks from [thoughtbot](http://www.thoughtbot.com). This post is part of a series sharing my experience and some of the things I'm learning.*

This week at Metis, we’re using Ruby and [Sinatra](http://www.sinatrarb.com) to get started with some basic dynamic web programming as a stepping stone to Rails. At its core, Sinatra is a Ruby gem that allows you to specify how an app will respond to different [http requests](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods) (`GET`, `POST`, etc.) and routes (the part of the URL after the domain, like `/` or `/blog` or `/blog/post/123`).

In this post, we’ll walk through creating a basic "hello world"-style app that offers a greeting to the user, customized based on the url.

<!-- MORE -->

## Setting up Sinatra

Since Sinatra is a gem, we’ll start by installing it with `gem install sinatra` in the terminal. We’ll also create a separate directory for this project (`mkdir hello_sinatra`).

In a new Ruby file (I’m using `hello_sinatra.rb`), add the line `require "sinatra"` so we can begin using the gem. We’re ready to go!

## Hello, World!

To start, we just want the app to print out "Hello, world!" when we navigate to the domain’s base directory in the browser. That’s the "root route" (say that 10 times fast), or the domain name without any path at the end (like `http://your_domain.com/`).

In Sinatra, we do this by calling [a method](http://www.sinatrarb.com/intro.html#Routes) that corresponds to the HTTP request type we want to recognize (in this case, `get`), and we pass it the route as an argument (in this case, `"/"`).

This method also takes a block, and the return value of the block is sent as the response to the web browser. Usually this will be a string containing HTML code, but we’ll start with simple text for now.

Here’s what it looks like:

``` ruby
require "sinatra"

get "/" do
  "Hello, world!"
end
```

### Test It Out

We can test it out in our browser by running `ruby hello_sinatra.rb`. This will launch a local server using port 4567 so we can load the app in a browser. Navigate to `localhost:4567` in your browser to check it out. If everything’s working, you’ll see "Hello, World!" in plain text.

Congratulations! You’ve built a simple web app!

## Adding URL Parameters

This is a nice proof-of-concept, but it doesn’t really *do* anything yet. Let’s change it up by printing different text based on the URL route that the user types in.

Sinatra allows us to specify parameters that take a value typed within the URL by the user. Parameters are specified with a `:symbol` in the route. So we could define a new route to dynamically change the greeting based on the URL:

``` ruby
get "/hello/:name" do
  @greeting_name = params[:name]
  "Hello, #{@greeting_name.capitalize}!"
end
```

We’ve added a couple of things here. Notice the symbol `:name` in the route — this is like a variable. Whatever is typed in the URL after `/hello/` will automatically be added by Sinatra to a hash called `params`. On the second line, we’re assigning that string from the `params` hash to an instance variable called `@greeting_name`. Finally, we just include that name in a string as the block’s return value, which is sent to the browser.

### Test it Out

Let’s try it out. Note that **every time we change the Ruby file, we’ll need to restart the server.** Use `CTL-c` in the terminal to quit the server, and run `ruby hello_sinatra.rb` again to relaunch the server with the updated Ruby file. In the browser, type `http://localhost:4567/hello/weasel` to load the page. If everything’s working correctly, you’ll see "Hello, Weasel!" in plain text.

Way to go! We’re dynamically generating content based on the URL! Try changing up the URL after `hello/` to see what happens.

## Multiple Parameters

From here, it’s trivial to add additional parameters to your routes by adding additional symbols. Let’s update the program to output the name *and* the greeting based on the URL:

``` ruby
get "/:greeting/:name" do
  @greeting_name = params[:name]
  @greeting_text = params[:greeting]
  "#{@greeting_text.capitalize}, #{@greeting_name.capitalize}!"
end
```

Reload the server and test it out in your browser with `http://localhost:4567/goodbye/squirrel`.

### Aside: Reloading the Server

Reloading the server every time we change our Ruby file is a pain, and it can cause headaches if you’re trying to troubleshoot your code but forget to restart the server. (It happens, I promise.) Luckily, there’s a gem that makes our life easier; it’s called Shotgun.

To get Shotgun, install it like any other gem: `gem install shotgun`. Then, instead of running your program with `ruby hello_sinatra.rb`, you can use `shotgun hello_sinatra.rb`. This will automatically reload your application each time a request is sent by the server, so you don’t have to kill the server and restart it each time. Awesome.

Also note that Shotgun uses a different default port, so you’ll want to point your browser to `http://localhost:9393` instead.

## Multiple Routes

Sinatra reads the routes in the order they’re written in your Ruby file; as soon as it finds a route that matches the request sent to the server, it executes that route. This means that routes should be listed in order from most specific to least specific. For example:

``` ruby
get "/:everything" do
  "I match everything typed in the URL after '/'!"
end

get "/partytime" do
  "This is a super exciting block, but it will never execute, because the first 'get' route matches '/partytime'."
end
```

Switching these around will produce the expected result.

## ERB

Now that we’re serving up some dynamic text, we’ll probably want to format it. But we also don’t want to start mixing a bunch of HTML code into our Ruby logic. Sinatra has a solution: it works with [embedded ruby](http://en.wikipedia.org/wiki/ERuby) files.

ERB is, in this case, simply HTML code with two additional pieces of syntax: `<% … %>` and `<%= … %>`. This allows us to embed Ruby code into an HTML page by inserting it between the tags.

There is one simple difference between these tags:

* The `<% … %>` tags execute any Ruby inside of them, but do not output the return value to the page.
*  The `<%= … %>` tags execute the Ruby code inside of them, and also perform string interpolation on the return value of the ruby code. This is printed to the page.

### Using ERB

To use an ERB template, we just add it to the block and use a symbol to reference the ERB file:

``` ruby
get "/:greeting/:name" do
  @greeting_name = params[:name]
  @greeting_text = params[:greeting]
  erb :greeting
end
```

The line `erb :greeting` tells Sinatra to look in the `views/` folder — the default for all template and layout files — for a file named `greeting.erb`. After processing the embedded Ruby, the server sends the output of that file to the browser.

Let’s create a our ERB file for this route: `views/greeting.erb`.

``` erb
<html>
  <head>
    <title>Greeting Page</title>
  </head>
  <body>
    <p>
      Hello, world!
    </p>
  </body>
</html>
```

Save everything and load `http://localhost:9393/howdy/partner` in your browser. You should see "Hello, world!" printed out.

### Instance Variables in ERB

But wait! ERB templates aren’t any fun if we print the same static page for every route. Fortunately, we can access our instance variables from within the ERB file:

``` erb
<html>
  <head>
    <title>Greeting Page</title>
  </head>
  <body>
    <p>
      <%= @greeting_text.capitalize %>, <%= @greeting_name.capitalize %>!
    </p>
  </body>
</html>
```

When we reload the page now, we should see "Howdy, Partner!", as expected.

Try updating the `<title>` element to include the instance variables as well, instead of a generic greeting.

### File Conventions and Directory Structure

In addition to using the `views/` directory for templates, Sinatra will also look in the `public/` directory (in the project root) for any *static resources* referenced in your views. That includes any images, stylesheets, javascripts, etc.

For example, this tag in an ERB file will load the specified image from `public/`:

``` html
<!-- note the leading slash -->
<img src="/grumpy_cat.png">
```

## Taking it Further

Sinatra has an additional templating feature called "layouts," which allows us to separate out some of the boilerplate HTML from our template files, to avoid repetition.

And we’ve only just scratched the surface here, but Sinatra also supports responding to other HTTP requests such as `POST` which allows the user to send information to the server via a web form.

I may explore these features (and/or others) in a later post.

## Additional Resources
* Sinatra’s Fantastic [Getting Started Guide](http://www.sinatrarb.com/intro.html)
* [Sinatra Documentation](http://www.sinatrarb.com/documentation.html)
* [Shotgun on GitHub](https://github.com/rtomayko/shotgun)
