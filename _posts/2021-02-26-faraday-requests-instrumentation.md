---
layout: post
title: faraday requests instrumentation
date: 2021-02-26
tags:
- ruby
- rails
- faraday
- instrumentation
---

[Faraday][1] is a flexible HTTP client library for Ruby. It supports multiple adapters
and is built using a rack-inspired [middleware][2] stack. Various middlewares can be enabled
to modify or log information during the request/response cycle.

Quite often, it is useful to log detailed information or metrics about the API requests your backend performs,
while serving an HTTP request. There is a [logger][3] middleware, which provides plenty configuration
options such as logging request headers, the request body, filtering sensitive information, or
customizing the log format. However, one of the most important metrics you would usually need, is
the duration of the request. This is not supported by the logger middleware. Enter, the [instrumentation][4]
middleware.

The instrumentation middleware allows us to use the excellent [Active Support Instrumentation][5]
to instrument our requests. Active Support includes an instrumentation API, which allows us
to hook into various parts of our Rails application to take measurements.
It is built using a pub/sub mechanism, we define events to be broadcasted, and also define subscribers
who listen for these events. Rails, provides itself a wide range of predefined events that we
can subscribe to, but also allows us to create custom events.

For this article, we have a minimal sample Rails application, which stores some primitive information
about movies. Let's assume that it stores the movie name, release year and a rating. We also have access
to an external API, which we can use to query using the same movie id, to retrieve the movie's cast. The idea
is to hit the `/movies/:id` endpoint, retrieve all the information we have about this movie from our database,
then query the external API for the extra information (cast), and pass this info to our view. We will show how we can use
faraday for querying the external API and use the instrumentation middleware to log the API call duration.

First, we need to install faraday.

```
bundle add faraday
```

Our Rails application has the following parts:

```ruby
# db/migrate20210220175430_create_movies.rb
class CreateMovies < ActiveRecord::Migration[6.1]
  def change
    create_table :movies do |t|
      t.string :title
      t.string :year
      t.float :rating

      t.timestamps
    end
  end
end

# config/routes.rb
resources :movies

# app/controllers/movies_controller.rb
class MoviesController < ApplicationController
  def show
    @movie = Movie.find(params[:id])
  end
end

# app/models/movie.rb
class Movie < ApplicationRecord
end
```

Pretty basic. The db table could definitely be better, but for our use case is fine.
We have also defined `app/views/movies/show.html.erb` to just render our controller
instance variables (omitted for brevity).

Now we will add a new model which will use our external API to retrieve the extra
movie information. This won't be an ActiveRecord model, we will use a simple PORO.

```ruby
# app/models/movie_info.rb
class MovieInfo
  HOST = 'bf74dc7a-7b56-47f6-9fcb-0881f7a36ff9.mock.pstmn.io'.freeze

  class << self
    def find(id)
      conn = Faraday.new(url: "https://#{HOST}/movies/#{id}")
      res = conn.get
      JSON.parse(res.body)
    end
  end
end
```

This class defines a `find` class method which accepts an id, then
constructs a new faraday connection object, and performs an API call to
the configured endpoint. Finally, it returns the response body as a ruby hash.
Faraday uses [Net::HTTP][6] by default, for this example we will not be configuring another adapter.

Defining a class method named `find` is inspired by the Rails ActiveRecord API,
so updating our controller to use this model will look like this:

```ruby
def show
  @movie = Movie.find(params[:id])
  @extra_movie_info = MovieInfo.find(params[:id])
end
```

With the current setup, if we access our application through `http://localhost:3000/movies/1`
we will get the following log output:

```
Started GET "/movies/1" for 127.0.0.1 at 2021-02-25 13:18:21 +0200
Processing by MoviesController#show as HTML
  Parameters: {"id"=>"1"}
  Movie Load (0.2ms)  SELECT "movies".* FROM "movies" WHERE "movies"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  ↳ app/controllers/movies_controller.rb:3:in `show'
  Rendering layout layouts/application.html.erb
  Rendering movies/show.html.erb within layouts/application
  Rendered movies/show.html.erb within layouts/application (Duration: 0.4ms | Allocations: 139)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 9.8ms | Allocations: 3583)
Completed 200 OK in 1109ms (Views: 10.6ms | ActiveRecord: 0.2ms | Allocations: 5164)
```

Notice that by default we don't get any info about the performed API request whatsoever. If we enable the
logging middleware that we briefly mentioned at the beginning of the article, we will get some basic output
on what is happening. Let's enable the logging middleware in our faraday configuration:

```ruby
def find(id)
  conn = Faraday.new(url: "https://#{HOST}/movies/#{id}") do |faraday|
    faraday.response :logger, nil, { headers: false, bodies: false }
  end
  res = conn.get
  JSON.parse(res.body)
end
```

Then, the output will look like this:

```
Started GET "/movies/1" for 127.0.0.1 at 2021-02-20 20:25:07 +0200
Processing by MoviesController#show as HTML
  Parameters: {"id"=>"1"}
   (0.1ms)  SELECT sqlite_version(*)
  ↳ app/controllers/movies_controller.rb:11:in `show'
  Movie Load (0.1ms)  SELECT "movies".* FROM "movies" WHERE "movies"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  ↳ app/controllers/movies_controller.rb:11:in `show'
I, [2021-02-20T20:25:07.573589 #39486]  INFO -- request: GET https://bf74dc7a-7b56-47f6-9fcb-0881f7a36ff9.mock.pstmn.io/movies/1
I, [2021-02-20T20:25:08.647338 #39486]  INFO -- response: Status 200
  Rendering layout layouts/application.html.erb
  Rendering movies/show.html.erb within layouts/application
  Rendered movies/show.html.erb within layouts/application (Duration: 1.2ms | Allocations: 139)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 47.3ms | Allocations: 3549)
Completed 200 OK in 1140ms (Views: 52.1ms | ActiveRecord: 1.9ms | Allocations: 8848)
```

Notice the two INFO log lines starting with `I`. Definitely better than before, but what we are aiming for here,
is to get a line which includes the call duration. Much like the lines which show the view rendering duration,
or the time ActiveRecord took to retrieve the data from our database.

Let's swap the logger with the instrumentation middleware:

```ruby
def find(id)
  conn = Faraday.new(url: "https://#{HOST}/movies/#{id}") do |faraday|
    faraday.request :instrumentation, name: "movies.faraday"
  end
  res = conn.get
  JSON.parse(res.body)
end
```

The `name` parameter defines the name of the event that will be broadcasted. The Rails
convention for defining event names is `event.library` and faraday sets this to `request.faraday`
by default.

The next step is to subscribe to listening to those events. This is done by
defining an `ActiveSupport::Notifications.subscribe` block, usually in an initializer or under `/lib`.

```ruby
# /lib/faraday_subscriber.rb
ActiveSupport::Notifications.subscribe("movies.faraday") do |name, starts, ends, _, env|
  url = env[:url]
  http_method = env[:method].to_s.upcase
  duration = ((ends - starts) * 1000.0).round(1)
  log_prefix = name.split(".").last.camelize
  output = "[%s] %s %s %s (Duration: %sms)" % [log_prefix, url.host, http_method, url.request_uri, duration]

  Rails.logger.info(output)
end
```

Quite a few things happening here. The `name` argument is set to the broadcasted event name, in our case
`movies.faraday`. The `starts` and `ends` are [Time][7] objects representing the time our event started
and ended respectively. The `env` argument is a `Faraday::Env` object which holds various information
about our request. In the Rails instrumentation [documentation][8], we can see more details about the `subscribe`
block arguments.

The `env` argument which holds the faraday request information can be used to extract the API request host and
HTTP method, as well as the headers, and the response body. In our example we use the host, HTTP method and request
uri to construct our log line.

The `starts` and `ends` are used to calculate the duration. According to the `Time` [documentation][9], by subtracting
two `Time` objects we get a `Float` representing the difference in **seconds**. So we go one step further to calculate
the time in milliseconds.

After restarting our application, the log output looks like this:

```
Started GET "/movies/1" for 127.0.0.1 at 2021-02-25 20:20:00 +0200
   (0.9ms)  SELECT sqlite_version(*)
   (0.1ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Processing by MoviesController#show as HTML
  Parameters: {"id"=>"1"}
  Movie Load (0.2ms)  SELECT "movies".* FROM "movies" WHERE "movies"."id" = ? LIMIT ?  [["id", 1], ["LIMIT", 1]]
  ↳ app/controllers/movies_controller.rb:3:in `show'
[Faraday] bf74dc7a-7b56-47f6-9fcb-0881f7a36ff9.mock.pstmn.io GET /movies/1 (Duration: 1262.7ms)
  Rendering layout layouts/application.html.erb
  Rendering movies/show.html.erb within layouts/application
  Rendered movies/show.html.erb within layouts/application (Duration: 1.2ms | Allocations: 411)
[Webpacker] Everything's up-to-date. Nothing to do
  Rendered layout layouts/application.html.erb (Duration: 12.9ms | Allocations: 5527)
Completed 200 OK in 1299ms (Views: 16.5ms | ActiveRecord: 1.1ms | Allocations: 13035)
```

Our log is prepended with `[Faraday]` and includes the host, the HTTP method, the API endpoint and the total request
duration. Success!

The subscribe block can be simplified a bit more. We can construct an [ActiveSupport::Notifications::Event][10]
object from the arguments. This will give us an object-oriented interface to the event data.

```ruby
ActiveSupport::Notifications.subscribe("movies.faraday") do |*args|
  event = ActiveSupport::Notifications::Event.new(*args)
  url = event.payload[:url]
  http_method = event.payload[:method].to_s.upcase
  log_prefix = event.name.split(".").last.camelize
  output = "[%s] %s %s %s (Duration: %sms)" % [log_prefix, url.host, http_method, url.request_uri, event.duration]
  Rails.logger.info(output)
end
```

We even get the duration calculation for free (in ms), neat!


[1]: https://github.com/lostisland/faraday
[2]: https://lostisland.github.io/faraday/middleware/
[3]: https://lostisland.github.io/faraday/middleware/logger
[4]: https://lostisland.github.io/faraday/middleware/instrumentation
[5]: https://edgeguides.rubyonrails.org/active_support_instrumentation.html
[6]: https://rubyapi.org/3.0/o/net/http
[7]: https://rubyapi.org/3.0/o/time
[8]: https://edgeguides.rubyonrails.org/active_support_instrumentation.html#subscribing-to-an-event
[9]: https://rubyapi.org/3.0/o/time#method-i-2D
[10]: https://edgeapi.rubyonrails.org/classes/ActiveSupport/Notifications/Event.html
