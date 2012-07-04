---
layout: post
title: "Cache with method replacement in ruby"
date: 2012-07-04 09:49
comments: true
categories: 
- ruby
---
After my last post, about
[caching method results in javascript](/blog/2012/07/02/clever-caching-on-javascript-objects),
I wondered if I could do the same thing in ruby.

Here's the basic class:

```ruby
class NoCaching
  attr_accessor :a, :b
  def sum
    a + b + a + b + a + b + a + b + a + b + a + b
  end
end
```

As expected, sum is evaluated every time you call it.

For comparison, I created a cached version using ActiveSupport::Memoizable.

```ruby
require 'active_support/memoizable'
class CachedWithActiveSupport < NoCaching
  extend ActiveSupport::Memoizable
  memoize :sum
end
```

As expected, calling sum on `CachedWithActiveSupport` returns the same thing,
regardless of changes to `a` or `b`.

Finally, I created a class with a new memoizer.

```ruby
class CachedWithRedefinedMethod < NoCaching
  extend RedefinedMethodCaching
  memoize :sum
end
```

This new memoizer will, on the first call to the memoized method, create a singleton
method on the instance that just returns the memoized value.

```ruby
module RedefinedMethodCaching
  def memoize method
    original_method = instance_method(method)
    define_method method do
      value = original_method.bind(self).call
      class_eval do
        define_method method do
          value
        end
      end
      send method
    end
  end
end
```

`CachedWithRedefinedMethod` has the same basic behavior as `CachedWithActiveSupport`.

## Performance

At this point, I got curious about performance. One of the reasons to cache
a method's result is to avoid some performance hit, so we don't want to
introduce another performance hit. Here's the basic benchmark:

```ruby
require 'benchmark'
Benchmark.bm(35) do |x|
  [1, 10, 100, 1000, 10000].each do |usage|
    iterations = 500000 / usage
    [NoCaching, CachedWithActiveSupport, CachedWithRedefinedMethod].each do |k|
      x.report("#{usage}x#{k.name}") { iterations.times { o = k.new ; o.a = 10 ; o.b = 20 ; usage.times { o.sum }
    end
  end
end
```

Here's my ruby:

    ruby 1.9.3p125 (2012-02-16 revision 34643) [x86_64-darwin11.2.0]

Here are the results:

```
                                          user     system      total        real
1xNoCaching                           0.740000   0.010000   0.750000 (  0.739076)
1xCachedWithActiveSupport             1.080000   0.000000   1.080000 (  1.140270)
1xCachedWithRedefinedMethod           5.100000   0.070000   5.170000 (  5.187266)

10xNoCaching                          0.460000   0.000000   0.460000 (  0.461452)
10xCachedWithActiveSupport            0.250000   0.000000   0.250000 (  0.253478)
10xCachedWithRedefinedMethod          0.560000   0.000000   0.560000 (  0.567997)

100xNoCaching                         0.420000   0.000000   0.420000 (  0.421145)
100xCachedWithActiveSupport           0.230000   0.000000   0.230000 (  0.224381)
100xCachedWithRedefinedMethod         0.120000   0.000000   0.120000 (  0.127860)

1000xNoCaching                        0.420000   0.000000   0.420000 (  0.419429)
1000xCachedWithActiveSupport          0.190000   0.000000   0.190000 (  0.187462)
1000xCachedWithRedefinedMethod        0.080000   0.010000   0.090000 (  0.083700)

10000xNoCaching                       0.380000   0.000000   0.380000 (  0.383493)
10000xCachedWithActiveSupport         0.200000   0.000000   0.200000 (  0.196237)
10000xCachedWithRedefinedMethod       0.100000   0.000000   0.100000 (  0.106058)
```

Notice that, uncached, the times are pretty consistent. The number of calls to sum is
the same in each run (500,000), so this is expected.

For ActiveSupport's memoize, you can see the overhead on the 1x run, and you can see
the performance gain that even a small number of reps gives.

For the new, redefined method style of memoization, you can see there's a pretty big
overhead, but the performance gain is significant. I compared it to the simple style
of memoization (`@sum ||= ...`), and it's about the same.

## Conclusions

This was an interesting experiment, but I probably won't use it much.

While the performance gain is pretty nice, it's also not a general purpose solution.
In my applications, most memoized methods are only called a handful of times. For
that type of use, memoization comes with a penalty, rather than a payoff.

Also, comparing to ActiveSupport's memoization is unfair, as ActiveSupport provides
a couple other nice features. One is cache-busting, via an optional `force` argument.
Another is support for arguments (i.e. `cached_method(1)` is different from `cached_method(2)`).
