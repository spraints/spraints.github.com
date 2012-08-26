---
layout: post
title: "clever caching on javascript objects"
date: 2012-07-02 10:39
comments: true
categories: 
- javascript
---
Javascript sports a prototype inheritance model. This morning, I was working on
a Javascript object, and I wanted to lazily evaluate one of its properties.
Normally, I'd write it like this:

{% highlight javascript %}
var Thing = function(a,b) { this.a = a; this.b = b; }
Thing.prototype.add = function() {
  if(!this._sum) this._sum = this.a + this.b;
  return this._sum;
};
{% endhighlight %}

Today, I thought, "Maybe I can make a singleton method (or whatever that's called
in Javascript) that caches the result?"

{% highlight javascript %}
var Thing = function(a,b) { this.a = a; this.b = b; }
Thing.prototype.add = function() {
  var sum = this.a + this.b;
  this.add = function() { return sum; }
  return this.add();
};
{% endhighlight %}

The last line could return `sum`, but it adds to the cleverness to return
what looks like a recursive call.

I tried this out in Chrome's JS console in order to verify a couple of things.

First, does it return the right answer?

    > var x = new Thing(1,2);
    > x.add()
    3

Did I do the singleton thing right? i.e. does each instance have an independent cached value?

    > var y = new Thing(2,3);
    > y.add()
    5

Does the function change, as expected?

    > var x = new Thing(1,2);
    > x.add
    function () {
      var sum = this.a + this.b;
      this.add = function() { return sum; }
      return this.add();
    }
    > x.add()
    3
    > x.add
    function () { return sum; }

And, finally, can I invalidate the cache?

    > x.a = 4
    4
    > x.add() // returns the old value
    3
    > delete x.add // invalidate the cache
    true
    > x.add() // returns the new value
    6

I don't know if I like this enough to keep using it, but it seems simple and pliable
enough to be worth trying. Thoughts? [Drop me a line on twitter](http://twitter.com/spraints).
