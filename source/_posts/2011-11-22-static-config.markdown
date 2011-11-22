---
layout: post
title: "Configuring your Ruby/Rails application with static_config"
date: 2011-11-22
categories:
- rails
---

I've written a ruby gem called [static_config][gh] that handles a common
(for me) configuration task. The pattern is: I'm writing a rails
application, and I have some extra bits of configuration that doesn't fit
into the normal set of configuration files. And, usually I want to vary
the behavior by environment.

For example, one application does not yet allow open signups in production.
I want to be able to turn that on &amp; off in development so that
(a) I can test it and
(b) I can not be bothered by it at other times. Also, we expect to allow
open signups in production eventually, just not yet, so I would like to
be able to flip the switch without necessarily needing to deploy new code.

In another application, we've been trying out a couple different
background task runners, and it was uncertain enough as to which we'd use
that I abstracted the particulars out so that I could focus on getting
the jobs running. So now, I can set the queue type in the config file.
And, for easing my development setup, I have dev set to 'immediate'. And
test is set to 'none'. And, of course, production is set to use the real
queue, 'resque' (for now). Very flexible.

To do this, I wrote [static_config][gh].

## Installation

To install, you'll need the static_config gem. You can
`gem install static_config` or use Bundler:

    gem 'static_config'

## An ad-hoc config file

The simplest case is just a config file that you can put anything into.
This would be the case where you just want to collect some configuration
into one place.

First, create a file called config.yml, maybe with something like this:

``` yaml config.yml
thing: abc
nested:
  thing: def
```

In your app, load up the config and use it.

``` ruby app.rb
require 'static_config'
MyConfig = StaticConfig.build do
  file File.expand_path('config.yml', File.dirname(__FILE__))
end
puts MyConfig.thing        # => abc
puts MyConfig.nested.thing # => def
```

## An ad-hoc, per-environment config file

So now, you want to change nested.thing to 'ghi' in production. It's
not much different.

``` yaml config.yml
development:
  thing: abc
  nested:
    thing: def
production:
  thing: abc
  nested:
    thing: ghi
```

``` ruby app.rb
#...
MyConfig = StaticConfig.build do
  file File.expand_path('config.yml', File.dirname(__FILE__)),
    :section => ENV['MY_APP_ENV'] || 'development'
end
#...
```

## An ad-hoc, per-environment config file that you can override with environment variables

Now, let's override some of the configuration with environment variables.

``` ruby app.rb
require 'static_config'
MyConfig = StaticConfig.build do
  file File.expand_path('config.yml', File.dirname(__FILE__)),
    :section => ENV['MY_APP_ENV'] || 'development'
  env 'MY_APP'
end
puts MyConfig.nested.thing
```

Now, you get this output:

``` sh
$ ruby app.rb
def
$ MY_APP_ENV=production ruby app.rb
ghi
$ MY_APP_NESTED_THING=blah ruby app.rb
blah
```

## All that, and reloading!

Finally, let's say you're doing this in a rails app.
You're going to want to have the configuration loaded
automatically, and, in development, reloaded, too.

``` yaml config/my_app.yml
development:
  thing: abc
  nested:
    thing: def
test:
  thing: test
  nested:
    thing: test
production:
  thing: abc
  nested:
    thing: ghi
```

``` ruby config/initializers/config.rb
MyConfig = StaticConfig.build do
  file Rails.root.join('config/my_app.yml'), :section => Rails.env
  env 'MY_APP'
end

Rails.application.config.to_prepare do
  MyConfig.reload!
end
```

[gh]: http://github.com/spraints/static_config
