---
layout: post
title: "accurate and lazy config errors"
date: 2012-02-15 16:21
comments: true
categories: 
---
What is better than being exactly right?
What about being lazy?
How about both?

This was a thought experiment that has migrated through code and into this post.

Here's the thought:
I write a ruby library that you have to configure, but I don't want to
pre-check all the config. I also want the exception that I through to come from
the bad config value, not from some weird-o spot in your and my code where we
both have a hard time figuring out what the problem is.

Can we pull off being lazy and accurate?

Here's one way to do it:

{% highlight ruby %}
class Config
  def driver=(d)
    if((exc = callcc { |cont| @driver_ptr = cont }) && exc.is_a?(Exception))
      raise exc
    end
    @driver = d
  end

  def try
    @driver_ptr.call ArgumentError.new("Driver cannot be :fail") if @driver == :fail
    puts @driver.inspect
  end
end

config = Config.new

config.driver = :ok
config.try

config.driver = :yes
config.try

config.driver = :fail
puts "Note that the error happens the line before this one (this line = #{__LINE__})"
config.try

config.driver = :not_reached
config.try
{% endhighlight %}

And, the output:

    :ok
    :yes
    Note that the error happens the line before this one (this line = 24)
    wacky.rb:4:in `driver=': Driver cannot be :fail (ArgumentError)
            from wacky.rb:23

I don't think I'd use this on a general purpose library, because of the ease of creating an infinite loop...

{% highlight ruby %}
begin
  config.driver = :fail
rescue => e
  puts e
end
config.try
{% endhighlight %}

... but it was fun to try it out.
