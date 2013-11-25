---
layout: post
title: "factory_girl and a tree of related objects"
date: 2012-01-13 10:48
comments: true
categories:
- rails
- factory_girl
---

My fixture replacement of choice is [factory_girl](https://github.com/thoughtbot/factory_girl/).
In general, it's been very good. But, as with any tool, there are edge cases that just don't work
as well. This is a common use case for me. It seems common enough that there should be a good solution
for it, but I haven't googled it up yet.

## The problem

The pattern is that I have a root model, with a couple of collections living inside it.

{% highlight ruby %}
class Site < ActiveRecord::Base
  has_many :members
  has_many :posts
end
{% endhighlight %}

One of the collections is made of objects that are just nested...

{% highlight ruby %}
class Member < ActiveRecord::Base
  belongs_to :site
  has_many :posts
end
{% endhighlight %}

... and the other is associated with both.

{% highlight ruby %}
class Post < ActiveRecord::Base
  belongs_to :author, :class_name => 'Member'
  belongs_to :site
end
{% endhighlight %}

(An obvious solution to this problem is to drop the
`belongs_to :site` from Post, but that's not always
desirable. Sometimes it just doesn't work, if Post and
Member had a habtm association, and neither depended on
the other's existence.)

The straightforward factories for the models above is this:

{% highlight ruby %}
Factory.define :site do |x|
end

Factory.define :member do |x|
  x.name 'example member'
  x.association :site
end

Factory.define :post do |x|
  x.title 'example post'
  x.association :author, :factory => :member
  x.association :site
end
{% endhighlight %}

The problem I run into is this: If I do `Factory(:post)`, then
the post\[id:1] has an author\[id:1] linked to site\[id:1]. But
post\[id:1] is linked to site\[id:2]!


## Solution: Use an after_build callback

The best solution I've come up with is to define these associations
that depend on each other in an `after_build` block. The simple way
to do it is just to try to hook up common associations.

{% highlight ruby %}
Factory.define :site do |x|
end

Factory.define :member do |x|
  x.name 'example member'
  x.after_build do |member|
    member.site ||= Factory.build(:site)
  end
end

Factory.define :post do |x|
  x.title 'example post'
  x.after_build do |post|
    post.site ||= post.author.try(site) || Factory.build(:site)
    post.author ||= Factory.build(:member, :site => post.site)
  end
end
{% endhighlight %}

## Solution improved: after_build and a test-global context.

Another method that I like is to create a context class. Because
most of my tests deal with data that is well-formed (e.g. it is
unexpected that the application would contain a post that links
to a member from a different site), the context class can store
a global-to-test site instance.

{% highlight ruby %}
class TestContext
  class << self
    def reset!
      @instance = nil
    end

    def instance
      @instance ||= new
    end

    def method_missing(*args)
      instance.send(*args)
    end
  end

  attr_accessor :site

  def site!
    self.site = Factory :site
  end
end

Factory.define :site do |x|
end

Factory.define :member do |x|
  x.name 'example member'
  x.after_build do |member|
    member.site ||= TestContext.site! || Factory.build(:site)
  end
end

Factory.define :post do |x|
  x.title 'example post'
  x.after_build do |post|
    post.author ||= Factory.build(:member)
    post.site ||= post.author.site || TestContext.site! || Factory.build(:site)
  end
end

class MyTestCase << Test::Unit::TestCase
  def setup
    TestContext.site!
  end
  def teardown
    TestContext.reset!
  end

  def test_stuff
    post = Factory :post
    # ...
  end
end
{% endhighlight %}

This works pretty well for me, but it seems like there should be a better way.
I think what I want is the concept of a fixed `Site` instance for the duration
of a given test, so, a replacement for the TestContext.

_The exact code in this article has not been tried, so it probably doesn't run.
Hopefully the intent is clear enough._
