---
layout: post
title: "Using Postgres Views with Rails"
date: 2014-09-17 22:17:00 -0400
comments: true
categories: ["Tech", "Ruby", "Rails", "Postgres", "PostgreSQL", "Database", "SQL", "View"]
---

*I'm a recent graduate of [Metis], a 12-week Ruby on Rails course taught by some great folks from [thoughtbot]. This post is part of a series sharing my experience and some of the things I've learned.*

Most relational databases support the concept of a [view][view wikipedia], a pre-defined relation that's essentially a stored query. You can query a view just like any other database table, but the fields and values in a view are defined based on the query used to create the view.

Views can be used for a number of reasons, most of them related to abstraction. For example, a view could be defined to allow easier querying of a complex join or to filter a relation to include only a subset of data.

In this post, we'll take a look at using a Postgres view with Rails to encapsulate data in a repeated aggregate query.

<!-- More -->

## The Project and the Problem

As my capstone project for [Metis], I created [BdwyCritic] ([source on GitHub]), a review aggregator for Broadway shows, similar to [Metacritic]. Every show (I called them "events") has many user reviews and many media (or critic) reviews.

When building the site, I found that I was frequently displaying the average user and media review score for events. In my original implementation, I used a class method to gather these statistics:

```ruby
# app/models/event.rb
class Event < ActiveRecord::Base
  has_many :user_reviews, dependent: :destroy
  has_many :media_reviews, dependent: :destroy

  def self.with_review_statistics
    select("events.*").
    select("AVG(media_reviews.score) AS average_media_review").
    select("AVG(user_reviews.score) AS average_user_review").
    joins("LEFT JOIN media_reviews ON events.id = media_reviews.event_id").
    joins("LEFT JOIN user_reviews ON events.id = user_reviews.event_id").
    group("events.id")
  end
end
```

There were at least two pain points I encountered with this code.

1. That's a lot of ugly SQL to have in the model, and it made it difficult to read and understand the method.
2. Every `Event` logically has the concept of an "average user review" or an "average media review." With this implementation, those values were only available for a given `Event` if I had explicitly queried the model using this class method.

Both of these [code smells] prompted me to explore alternative implementations.

## Choosing a Database View as the Solution

I considered a couple of solutions before settling on using a database view:

1. Change this class method into the **[default scope]** for the `Event` model. That would resolve the second issue above, because every instance of `Event` would be automatically scoped with fields representing the average review values. The difficult-to-read SQL code would still remain in the model, however, and default scopes can sometimes lead to other unexpected complications with queries.

2. **De-normalize the database** and keep a cache of the average review scores for each event. This would potentially come with a performance increase for database reads (which, in this application, are more frequent than writes). It would also likely involve using [ActiveRecord callbacks] to manually maintain the cached values, something I prefer to avoid when at all possible.

3. A **database view** solves both of the challenges of my previous implementation. It removes the SQL statements from the model and allows every `Event` to have methods for average user and media reviews. (I achieved the latter through [delegation], as we'll see.)

(There *is* one caveat to call out here: because ActiveRecord doesn't natively support views, using a view removes the flexibility that ActiveRecord provides to interface with different database backends. There could be a cost later if we want to switch over to, say, MySQL. Given what I know about this project and the production environment, that's a trade-off I'm currently comfortable making.)

## Creating the Database View

Like any other database change in Rails, creating the view involves a migration.

Because the ActiveRecord [DSL] doesn't provide methods for creating views, we actually have to [write the SQL ourselves][Postgres Create View], but it's not too complicated. The `SELECT` is basically a translation of our earlier class method into raw SQL.

```ruby
# db/migrate/TIMESTAMP_create_review_statistics_summary_view.rb
class CreateReviewStatisticsSummaryView < ActiveRecord::Migration
  def up
    execute <<-SQL
      CREATE VIEW review_statistics_summary AS
        SELECT
          events.id as event_id,
          AVG(user_reviews.score) AS average_user_review,
          AVG(media_reviews.score) AS average_media_review
        FROM
          events
          LEFT JOIN user_reviews ON events.id = user_reviews.event_id
          LEFT JOIN media_reviews ON events.id = media_reviews.event_id
        GROUP BY
          events.id
    SQL
  end

  def down
    execute "DROP VIEW review_statistics_summary"
  end
end
```

Then, just run `rake db:migrate`.

## Setting up the Models

### Defining a Model for the Database View

Despite ActiveRecord's lack of support for views, it does treat the view as any other relation, so we can easily hook it up to a model.

```ruby
# app/models/review_statistics_summary.rb
class ReviewStatisticsSummary < ActiveRecord::Base
  self.primary_key = "event_id"

  belongs_to :event
end
```

This view doesn't have it's own `id` column because `event_id` is the value that uniquely identifies records in the relation. ActiveRecord expects the primary key column to be called `id`, though, so we explicitly define the primary key column in the model. 

*Technically*, because this model will only be used in relation to an `Event` object, we don't even need to specify the primary key. I'm inclined to include it here anyway, since there's a chance we might run into unexpected ActiveRecord errors in the future without it (if we try to call `ReviewStatisticsSummmary.find`, for example).

### Updating the Event Model

The second step is to relate the `Event` model to the `ReviewStatisticsSummary` model. This one is easy:

```ruby
# app/models/event.rb
has_one :review_statistics_summary
```

### Delegating the Average Methods

Finally, we'd like to avoid chained method calls (like `event.review_statistics_summary.average_user_review`), which are difficult to read and violate the [Law of Demeter].

The simple solution here is to [delegate] the `average_user_review` and `average_media_review` methods to the `ReviewStatisticsSummary` class:

```ruby
# app/models/event.rb
delegate :average_user_review, :average_media_review, to: :review_statistics_summary
```

Now we can call `event.average_user_review` or `event.average_media_review` as expected, and those method calls are *delegated* to that event's instance of `ReviewStatisticsSummary`.

This syntax is a [Rails shorthand][delegate] equivalent to defining these two methods on `Event`:

```ruby
# app/models/event.rb
def average_user_review
  review_statistics_summary.average_user_review
end

def average_media_review
  review_statistics_summary.average_media_review
end
```

We can also delete the entire `self.with_review_statistics` class method from earlier. Hooray!

## Postgres Views and the Rails Database Schema

One final caveat to note here: because of the way we've defined the view with raw SQL, ActiveRecord has no way of adding that information to Rails's `db/schema.rb` file.

However, if you're deploying an app, you often don't want to run all of the migrations to set up the database. Instead, [the preferred way to deploy the database][Rails Migrations Guide 1] is to just load the schema itself. Since the schema doesn't contain a definition for the view we've just created, we'll run into problems.

This is of particular concern when using RSpec with Rails 4.1+, because RSpec loads the test environment database from the schema file by default. Since we've manually created the view and it's not in `db/schema.rb`, Postgres will complain that the view does not exist when running the tests.

The workaround is to tell Rails to use raw SQL for its schema in a file called `db/structure.sql`. This file replaces `db/schema.rb` and dumps the structure directly from the database instead of using the ActiveRecord [DSL]. The [Rails Guide on Migrations][Rails Migrations Guide 2] provides detailed instructions for making that change. It's pretty painless.

## Resources

* [Overview of Database Views][view wikipedia]
* [Postgres Documentation for Views][Postgres Create View]
* [Rails Schema Formats][Rails Migrations Guide 1]
* [Delegation Pattern][delegation]
* [Method Delegation in Rails][delegate]
* [Law of Demeter]

[ActiveRecord callbacks]: http://guides.rubyonrails.org/active_record_callbacks.html
[BdwyCritic]: http://bdwycritic.com
[code smells]: http://en.wikipedia.org/wiki/Code_smell
[delegate]: http://guides.rubyonrails.org/active_support_core_extensions.html#method-delegation
[delegation]: http://en.wikipedia.org/wiki/Delegation_pattern
[default scope]: http://guides.rubyonrails.org/active_record_querying.html#applying-a-default-scope
[DSL]: http://en.wikipedia.org/wiki/Domain-specific_language
[Law of Demeter]: http://c2.com/cgi/wiki?LawOfDemeter
[Metacritic]: http://www.metacritic.com
[Metis]: http://www.thisismetis.com
[Postgres Create View]: http://www.postgresql.org/docs/9.3/static/sql-createview.html
[Rails Migrations Guide 1]: http://guides.rubyonrails.org/migrations.html#schema-dumping-and-you
[Rails Migrations Guide 2]: http://guides.rubyonrails.org/migrations.html#types-of-schema-dumps
[source on GitHub]: https://github.com/reshleman/bdwycritic
[thoughtbot]: http://www.thoughtbot.com
[view wikipedia]: http://en.wikipedia.org/wiki/View_%28SQL%29