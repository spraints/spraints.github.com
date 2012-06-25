---
layout: post
title: "rails 3 exception handling"
date: 2012-06-25 11:00
comments: true
categories: 
- rails
---
Rails exception handling is pretty flexible. In development, it shows detailed
error information. In production, it shows a user-friendly page, possibly
of your choice. In any environment, it provides hooks that you can use to
customize the process.

In this blog post, I'd like to show how to do some neat exception handling
tricks with rails 3.

## Trick #1 - Use a custom exception

Using a custom exception is simple. Just define the exception class

```ruby
class MyCustomException < StandardError ; end
```

and raise an exception.

```ruby
raise MyCustomException
```

In development, you'll see the detailed exception page, including the
exception's type, message, and stack trace. In production, rails renders
the default, generic exception page, normally stored in public/500.html.

## Trick #2 - Custom exception status code

If you want the exception to produce a different HTTP result status, you need
to tell rails about it. The default is 500.

The way to customize the result code has changed slightly in different releases of
rails. Here are some ways to tell rails how to handle the exception.

In all versions of rails, you can refer to HTTP status codes numerically or with
[symbols as defined by Rack](https://github.com/rack/rack/blob/edc8b923fa4ea4e3c1ce8778f5ddbc89688bc01b/lib/rack/utils.rb#L473).
For example, you can refer to '404 Not Found' as `:not_found` or `404`.

In **Rails 3.0 and 3.1**, you can change the status code of an exception's response
with `rescue_responses`:

```ruby
ActionDispatch::ShowExceptions.rescue_responses['MyCustomException'] = :unauthorized
```

There are a few [default exception handlers](https://github.com/rails/rails/blob/3-0-stable/actionpack/lib/action_dispatch/middleware/show_exceptions.rb).

In **Rails 3.2**, this hash moved, to make it easier to update via a railtie.
You can change the status code of an exception's response in your
application configuration (e.g. `config/application.rb` or
`config/environments/development.rb`):

```ruby
config.action_dispatch.rescue_responses.merge!('MyCustomException' => :unauthorized)
```

In public (i.e. production), rails will render the exception from a
file in `public/`, with the status code as its name. For example, if
the result status is 404, rails will read and return `public/404.html`.
In **Rails 3.2**, you can override this behavior by setting `config.exceptions_app`.
See [ActionDispatch::PublicExceptions](https://github.com/rails/rails/blob/v3.2.6/actionpack/lib/action_dispatch/middleware/public_exceptions.rb)
for an example of what this should look like.

```ruby
class MyPublicExceptions
  def call(env)
    [404, {'Content-Type' => 'text/plain'}, ['not found']]
  end
end

# in config/application.rb
config.exceptions_app = MyPublicExceptions.new
```

## Trick #3 - Custom exception handler on controller

If you want to do something else with your exceptions, you can define a
custom exception handler on a controller. For example, in `ApplicationController`,
you can add this:

```ruby
rescue_from 'MyCustomException', :with => :my_exception_handler

def my_exception_handler exception # This method can take 0 or 1 args
  render :template => 'blah/blah'
end
```

The nice thing about this is that you can completely control how the exception
is rendered. Unfortunately, using this form of rescue short-circuits other
'normal' exception handling. So, it will always call the handler, even in
development. It will not bubble up to the airbrake notification middleware.

## Trick #4 - Production exception handling in development

As you build up more intricate exceptions & handlers, you'll want to
try out your custom handlers in development.

In development mode, you'll need to do a couple of things to view the
production-style exception pages. First, you need to turn off the config
flag that forces all requests to behave as if they're local:

```ruby
# in config/environments/development.rb
config.consider_all_requests_local       = false # true by default
```

In **Rails 3.2**, that should be all you need to do. You should now see
production-style exception pages.

In **Rails 3.0 and 3.1**, you'll also need to convice rails that the
request isn't local. I'm not sure what the best way to do this is, but
one way that worked for me was to add this to the bottom of
`app/controllers/application_controller.rb`:

```ruby
class ActionDispatch::Request
  def local?
    false
  end
end
```
