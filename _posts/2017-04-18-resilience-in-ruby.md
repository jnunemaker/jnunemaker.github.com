---
title: 'Resilience in Ruby: Handling Failure'
layout: post
tags: recommended
---

For the second half of 2014 and 2015, most of my time at GitHub was spent working on moving the notifications feature off the primary MySQL cluster on to a new MySQL cluster and making the feature resilient to failure of that cluster.

When all your data is stored in one database, making your application resilient to failures of that database is kind of pointless. If the database is unavailable, your entire application is likely unusable.

Once you outgrow a single database and begin to partition data, resilience to failure becomes important. You do not want your entire application unavailable to users solely because you are unable to get a count of unread notifications to show in the top bar of the website.

## Error Handling Basics

The first step in resilience is error handling. There are a lot of different approaches to handling errors, as outlined by Joe Duffy in his marvelous post [The Error Model](http://joeduffyblog.com/2016/02/07/the-error-model/). The approach Ruby takes to errors is exceptions. When an error is encountered, it is pretty typical that an exception will be raised. For example, let's make an HTTP request to a service that is not available.

```ruby
require "net/http"
Net::HTTP.new("localhost", 9999).request(Net::HTTP::Get.new("/"))
```

If you drop the snippet above in IRB, you will end up with an error like this (assuming you do not have a web server running on port 9999):

```
Errno::ECONNREFUSED: Connection refused - connect(2) for "localhost" port 9999
```

The error causes your program to exit with a failure. If you drop the snippet above in a file, execute it with ruby and check the exit status, it will be 1 (failure).

```
$ ruby resilience.rb
  ... [snip several lines of error and backtrace] ...
$ echo $?
1
```

In order to handle the error, Ruby provides `begin`/`rescue`.

```ruby
require "net/http"
begin
  Net::HTTP.new("localhost", 9999).request(Net::HTTP::Get.new("/"))
  puts "Succeeded"
rescue
  puts "Failed"
end

# Outputs: Failed
```

The snippet above, if dropped in a file and executed, would return an exit status of 0 (success).

```
$ ruby resilience.rb
Failed
$ echo $?
0
```

The problem with a rescue statement like this is that all exceptions will be rescued, even those related to programmer error, such as an `ArgumentError`. For example, in this snippet note that I forgot to provide a request.

```ruby
require "net/http"
begin
  Net::HTTP.new("localhost", 9999).request()
  puts "Succeeded"
rescue
  puts "Failed"
end

# Outputs: Failed
```

If you execute this, your script indeed says that it failed and exits with success.

```
$ ruby resilience.rb
Failed
$ echo $?
0
```

The `rescue` happily swallows the `ArgumentError` and you never get feedback that the request is failing due to programmer error. You likely would assume that the backend service was down and lose a lot of time debugging it, instead of quickly noticing that you were missing an argument.

In order to allow for programmer feedback during development (like an `ArgumentError`), but resilience in production, you can be more specific about which exceptions you would like to handle. For example, you can handle the `Errno::ECONNREFUSED` error like so:

```ruby
require "net/http"
begin
  Net::HTTP.new("localhost", 9999).request(Net::HTTP::Get.new("/"))
  puts "Succeeded"
rescue Errno::ECONNREFUSED
  puts "Failed"
end

# Outputs: Failed
```

If you run this, you do indeed get `Failed` and a successful exit status.

```
$ ruby resilience.rb
Failed
$ echo $?
0
```

Additionally, if you forget the argument, you get an `ArgumentError` and an unsuccessful exit code. Running this snippet:

```ruby
require "net/http"
begin
  Net::HTTP.new("localhost", 9999).request()
  puts "Succeeded"
rescue Errno::ECONNREFUSED
  puts "Failed"
end
```

results in:

```
$ ruby resilience.rb
/Users/jnunemaker/.rbenv/versions/2.2.5/lib/ruby/2.2.0/net/http.rb:1373:in `request': wrong number of arguments (0 for 1..2) (ArgumentError)
	from resilience.rb:3:in `<main>'
$ echo $?
1
```

### Verbose and Brittle

Though `begin` and `rescue` give you the tools you need to handle errors in Ruby, they definitely left me wanting. For starters, needing to wrap every call felt verbose. A call that use to be one line and relatively clear, is now a bit muddled with at least 3 extra lines. In plain old ruby code, the verbosity is not that bad, but once you move into Rails views or something similar, you really feel you need to begin being rescued (see what I did there).

Additionally, if you do not provide some extra layer on top, you will end up with a lot of call sites rescuing a lot of exceptions. Each new exception will require updating all of the call sites and, trust me, this is tedious and brittle (easy to miss a call site).

## A Layer On Top

Let's continue with our HTTP request example and add a layer on top to chokepoint error handling to one place. Let's also pretend our HTTP request is reaching out to a service that provides a JSON response.

```ruby
require "json"
require "net/http"

class Client
  # Returns Hash of notifications data for successful response.
  # Returns nil if notification data cannot be retrieved.
  def notifications
    begin
      request = Net::HTTP::Get.new("/")
      http = Net::HTTP.new("localhost", 9999)
      response = http.request(request)
      JSON.parse(response.body)
    rescue Errno::ECONNREFUSED
      # what should we return here???
    end
  end
end

client = Client.new
p client.notifications
```

You now have the chokepoint for error handling in the notifications method, which is cool, but nil is being used as a way of communicating that something went wrong, which is not cool. You could return something other than nil, but you end up with a fork in the road. Success returns one object and failure returns something different.

## Response Objects

What I have been reaching for lately is something along these lines:

```ruby
require "json"
require "net/http"

class Client
  class NotificationsResponse
    attr_reader :notifications, :error

    def initialize(&block)
      @error = false
      @notifications = begin
        yield
      rescue Errno::ECONNREFUSED => error
        @error = error
        {} # sensible default
      end
    end

    def ok?
      @error == false
    end
  end

  def notifications
    NotificationsResponse.new do
      request = Net::HTTP::Get.new("/")
      http = Net::HTTP.new("localhost", 9999)
      http_response = http.request(request)
      JSON.parse(http_response.body)
    end
  end
end

client = Client.new
response = client.notifications

if response.ok?
  # Do something with notifications like show them as a list...
else
  # Communicate that things went wrong to the caller or user.
end
```

By putting a response object in between the caller and the call to get the data:

* we always return the same object, avoiding nil and retaining duck typing.
* we now have a place to add more context about the failure if necessary, which we did not have with `nil`.
* we have a single place to update rescued exceptions if a new one pops up.
* we have a nice place for instrumentation and circuit breakers in the future.
* we avoid needing `begin` and `rescue` all over and instead can use conditionals or whatever makes sense.
* we give the caller the ability to handle different failures differently (Conn refused vs Timeout vs Rate limited, etc.).

If we so desire, we can even go a step further and make sure that `Response#ok?` is checked by doing something like this:

```ruby
require "json"
require "net/http"

class Client
  class NotificationsResponse
    attr_reader :error

    def initialize(&block)
      @error = false
      @notifications = begin
        yield
      rescue Errno::ECONNREFUSED => error
        @error = error
        {} # sensible default
      end
    end

    def ok?
      @ok_predicate_checked = true
      @error == false
    end

    def notifications
      unless @ok_predicate_checked
        raise "ok? must be checked prior to accessing response data"
      end

      @notifications
    end
  end

  def notifications
    NotificationsResponse.new do
      request = Net::HTTP::Get.new("/")
      http = Net::HTTP.new("localhost", 9999)
      response = http.request(request)
      JSON.parse(response.body)
    end
  end
end

client = Client.new
response = client.notifications
# response.notifications would raise error because ok? was not checked
```

## GitHub::Result

Along these lines, I recently open sourced [github-ds](https://github.com/github/github-ds), which contains a `GitHub::Result` class. We use `GitHub::Result` at GitHub more and more of late to bake resiliency in at the beginning of projects. `GitHub::KV` is a [good example](https://github.com/github/github-ds/blob/a7ec2ac641ca0f30ff7f268b31efe13ab470cf34/lib/github/kv.rb) of `GitHub::Result` in the real world, but here is a little taste:

```ruby
def do_something
  1
end

def do_something_that_errors
  raise "noooooppppeeeee"
end

result = GitHub::Result.new { do_something }
p result.ok? # => true
p result.value! # => 1

result = GitHub::Result.new { do_something_that_errors }
p result.ok? # => false
p result.value { "default when error happens" } # => "default when error happens"
begin
  result.value! # raises exception because error happened
rescue => error
  p result.error
  p error
end

# Outputs Step 1, 2, 3
result = GitHub::Result.new {
  GitHub::Result.new { puts "Step 1: success!" }
}.then { |value|
  GitHub::Result.new { puts "Step 2: success!" }
}.then { |value|
  GitHub::Result.new { puts "Step 3: success!" }
}
p result.ok? # => true

# Outputs Step 1, 2 and stops.
result = GitHub::Result.new {
  GitHub::Result.new { puts "Step 1: success!" }
}.then { |value|
  GitHub::Result.new {
    puts "Step 2: failed!"
    raise
  }
}.then { |value|
  GitHub::Result.new {
    puts "Step 3: should not get here because previous step failed!"
  }
}
p result.ok? # => false
```

## Conclusion

Welp, this article has rambled on long enough. It is by no means a comprehensive article on resiliency, but it covers step 1, which is handling failure. The key to me including **a layer on top that bakes in the resiliency**, making it easy for callers to do the right thing in the face of failure. Using response objects and/or `GitHub::Result` can definitely help with that.

I am not going to be wreckless and promise that I will write up step 2, but if I do, there is a good chance it will be titled "Limit Everything". :)
