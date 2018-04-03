---
layout: post
title: "AJAX: It's Better with Rails"
date: 2014-07-31 15:59:11 -0400
comments: true
redirect_from: /blog/2014/07/31/ajax-its-better-with-rails/
categories: ["AJAX", "Rails", "JavaScript", "Metis", "Tech", "jQuery"]
---

*This summer, I'm learning Ruby on Rails at [Metis](http://www.thisismetis.com), a 12-week class taught by some great folks from [thoughtbot](http://www.thoughtbot.com). This post is part of a series sharing my experience and some of the things I'm learning.*

Most people love the responsive experience of using [AJAX](http://en.wikipedia.org/wiki/Ajax_%28programming%29)-y web pages. *Creating* that functionality, on the other hand, is not always so enjoyable: asynchronous requests can be difficult to code and debug.

It may not be a surprise that Rails helps us abstract some of the more challenging AJAX code. In this post, we'll take a look at a basic AJAX request constructed with vanilla jQuery; then, we'll check out the tools Rails gives us to create and respond to asynchronous requests.

<!--More-->

## The Hard Way

The [jQuery library](http://jquery.com/) allows us to construct AJAX requests ourselves. If we were to do this for a form submission to create a new task in a todo-list app, it might look something like this:

```js
$(function() {
  $("#form_id").submit(function() {
    $.post(
      "/tasks",
      { task: 
        { title: 
          $("#task_title").val()
        }
      },
      function(data) {
        $("#tasks").prepend(data)
      }
    );
    return false;
  });
});
```

The outer function call (`$(function() { ... });`) tells jQuery that the code inside is to be executed once the [document is ready](http://api.jquery.com/ready/) (i.e., after the entire DOM loads).

Then, we tell the browser to [bind a function to the submit event](http://api.jquery.com/submit/) for our form, by calling `$("#form_id").submit(function() { ... });`. This tells the browser that the enclosed JavaScript function should be executed when the form is submitted.

Within that function we have two statements: the first one (`$.post( ... );`) [creates a POST request via AJAX](http://api.jquery.com/jQuery.post/); the second (`return false;`) tells the browser not to execute the default action for the submitted form, which, in this case, would cause the page to reload.

Our actual AJAX request has three parameters: the route "/tasks", which we send the request to, the data we're sending (an object), and a callback function that runs if the request is successful. Our callback function simply [prepends](http://api.jquery.com/prepend/) the `data` (in this case, HTML) returned by the server to the DOM element with the id of `tasks`.

Phew.

## The Rails Way

Fortunately, Rails can take care of a lot of this code for us. If we're using `form_for` or `link_to` in our view, we can specify the argument `remote: true` to cause Rails to [automatically submit the form](http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html#built-in-helpers) (or load the link) via an AJAX request. We don't have to write any of the JavaScript to send the request!

We will have to write the JavaScript we want the browser to execute when we receive the response, but Rails sends that JavaScript as the response to the AJAX request. We'll need to take a look at our Controller for that.

## XHR in Controllers

One neat thing about Rails is that we can use the same controller and action for a standard HTTP request and for an asynchronous request, simply by telling Rails what kind of requests to respond to.

In our controller, we use a [`respond_to`](http://api.rubyonrails.org/classes/ActionController/MimeResponds.html#method-i-respond_to) block to list the different formats: 

```rb
def create
  # Do some stuff to create a task
  respond_to do |format|
    format.html { redirect_to tasks_path }
    format.js   { }
   end
end
```

This code tells Rails to respond to either a request for an HTML response or a request for a JavaScript response. If the browser is asking for an HTML response, we just redirect to the given path; if the browser is expecting a JavaScript response (it does when we use `remote: true` on our form), then we render the default view for this controller, action, and response format. In this case, that would be `app/views/tasks/create.js.erb`

## .js.erb Views

In that view, we can include any JavaScript we want, **and the browser will automatically execute that JavaScript** when it receives the response. This basically becomes our callback function, and, because it's a view, we can render partials in it.

Our `.js.erb` file for this example might look like:

```js
$("#tasks").prepend("<%=j render @task %>");
$("#task_title").val("");
```

This JavaScript prepends the rendered task to the DOM element with the id `tasks`, and clears our form field. Note that we use special ERB tags (`<%=j ... %>`) to escape our rendered data so that it can be safely included in a JavaScript string.

## Resources

That's a quick overview of some of the most basic things you can do with Rails and JavaScript. There are plenty of resources on the web with more information about [JavaScript](http://it-ebooks.info/book/274/), [jQuery](http://api.jquery.com/), [CoffeeScript](http://coffeescript.org/) (a language that compiles to JavaScript), and [how Rails works with these](http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html). Here are a few:

* [Rails Guide on AJAX](http://edgeguides.rubyonrails.org/working_with_javascript_in_rails.html)
* [JavaScript: The Good Parts (ebook)](http://it-ebooks.info/book/274/)
* [jQuery API](http://api.jquery.com/)
* [CoffeeScript](http://coffeescript.org/)
