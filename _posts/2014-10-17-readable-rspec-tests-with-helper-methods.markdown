---
layout: post
title: "Readable RSpec Tests with Helper Methods"
date: 2014-10-17 15:55:34 -0400
comments: true
categories: ["Ruby", "TDD", "Rails", "Testing", "RSpec", "Capybara", "Refactoring"]
---

*I'm a recent graduate of [Metis], a 12-week Ruby on Rails course taught by some great folks from [thoughtbot]. This post is part of a series sharing my experience and some of the things I've learned.*

I've been spending a lot of time recently becoming familiar with [Rspec], [Capybara], and [TDD]. Because tests frequently have similar or repeated steps, this has also been a great opportunity to practice [DRY]-ing up my code.

One particularly effective technique I've been using often in my tests is extracting helper methods with descriptive names to make my code easier to read.

<!--More-->

## Setup and Exercise

In the setup and exercise [phase]s, code across tests may be very similar and ripe for extraction into a method. For example, here's a helper method in one of my feature specs for a user signing in:

```ruby
def visit_sign_in_page
  visit root_path
  click_link "Sign in"
end
```

And another simple one that takes some parameters:

```ruby
def user_signs_in_as(email, password)
  fill_in "Email", with: email
  fill_in "Password", with: password
  click_button "Sign in"
end
```

## Verify

I've found that the verification [phase] of my tests tend to see the most readability benefit from extracting methods.

For example, this was some code that I used in a test verification recently:

```ruby
expect(page).to have_css(".event", text: event.name)
```

This isn't too bad, but it becomes clearer when extracted into a helper method:

```ruby
def have_event_summary(event)
  have_css(".event", text: event.name)
end
```

The intent of this verification is obvious when I now write `expect(page).to have_event_summary(event)` or `expect(page).not_to have_event_summary(event)`.

For more complex verifications, the expectation can also be included in the helper method, like so:

```ruby
def expect_page_to_have_user_review_summary_for(event)
  within(".review-score", text: "Users") do
    expect(page).to have_text(event.average_user_review_score)
    expect(page).to have_text("#{event.num_user_reviews} Reviews")
  end
end
```

The verification phase for most of my tests using this method simply reads `expct_page_to_have_user_review_summary_for(event)`. This is much nicer than multiple `have_css` or `have_text` expectations.

As an additional bonus, RSpec will even provide clear failure messages if these assertions fail:

```
Failure/Error: expect_page_to_have_user_review_summary_for(event)
expected to find text "9.0" in "No Review Here"
```

## Reuse Across Files

Finally, RSpec allows these methods to be easily reused across spec files, if we so desire.

All we have to do is wrap them up in a module like `EventSummaryHelpers` in `spec/support/`. Then, in `spec/rails_helper.rb`, simply tell RSpec to load that module:

```ruby
RSpec.configure do |config|
  config.include EventSummaryHelpers, type: :feature
end
```

In this case, to save overhead, RSpec will only load these helpers when running feature specs.

## Resources

* [Code Reuse in RSpec]
* [Four-Phase Tests][phase]
* [Writing Better Tests]
* [RSpec]
* [Capybara]
* [TDD]

[Metis]: http://www.thisismetis.com
[thoughtbot]: http://www.thoughtbot.com
[RSpec]: http://rspec.info
[Capybara]: http://jnicklas.github.io/capybara/
[TDD]: http://en.wikipedia.org/wiki/Test-driven_development
[DRY]: http://c2.com/cgi/wiki?DontRepeatYourself
[phase]: http://robots.thoughtbot.com/four-phase-test
[Code Reuse in RSpec]: http://testdrivenwebsites.com/2011/08/17/different-ways-of-code-reuse-in-rspec/
[Writing Better Tests]: http://betterspecs.org