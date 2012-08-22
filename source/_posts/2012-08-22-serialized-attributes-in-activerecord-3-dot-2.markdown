---
layout: post
title: "Serialized Attributes in ActiveRecord 3.2"
date: 2012-08-22 08:45
comments: true
categories: 
- rails
---
Rails has had serialized attributes for a while, and every
time I try to do something fancy with them, I have to rediscover
how they work.

I have used [`composed_of`](http://apidock.com/rails/ActiveRecord/Aggregations/ClassMethods/composed_of)
to do some of these things in the past, so take a look at it if
[`serialize`](http://apidock.com/rails/ActiveRecord/AttributeMethods/Serialization/ClassMethods/serialize)
isn't quite what you're looking for.

## The basics

To get started with serialize, add a `text` column to your table,
and add the serialize directive to your model.

```ruby
class MyModel < ActiveRecord::Base
  # This serializes whatever you assign.
  serialize :my_field
  # This serializes a hash. If the DB value is nil, the AR value will be {}
  serialize :my_field, Hash
  # This serializes some class that you've written.
  serialize :my_field, MyField
end
```

If you choose the last option, what aspects of serialization
can you control?

## Default serialization with YAML

By default, your attribute will be serialized as YAML. Let's say
you define MyField like this:

```ruby
class MyField
end
```

It will be serialized like this:

    --- !ruby/object:MyField {}

It will deserialize back to a `MyField` instance.

## Custom serialization

You can customize the serialization in a couple of different ways.
One way is to implement the instance method `to_yaml`. I wouldn't
do this, because you'll need to do it in a way that `Yaml.load` can
use to create an instance of your class.

The other way to do it is to add `dump` and `load` class methods to
your class. For example:

```ruby
class MyField
  def self.dump(obj)
    # ...
  end
  def self.load(string)
    # ...
  end
end
```

Dump converts your object to a string, and load converts a string into
an instance of your class. Here's an example of a custom formatting of
attributes:

```ruby
class MyField
  def self.dump(obj)
    coder.dump(:a => obj.a, :b => obj.b)
  end
  def self.load(string)
    new coder.load(string)
  end
  def self.coder
    ActiveRecord::Coders::YAMLColumn.new(Hash)
  end
  def initialize(options = {})
    @a = options[:a]
    @b = options[:b]
  end
  attr_accessor :a, :b
end
```

Now, let's say you want to be able to assign a hash to the serialized
object, maybe from a form. You can do that:

```ruby
class MyField
  def self.dump(obj)
    case obj
    when Hash then coder.dump(obj)
    else           coder.dump(:a => obj.a, :b => obj.b)
    end
  end
  def self.load(string)
    new coder.load(string)
  end
  def self.coder
    ActiveRecord::Coders::YAMLColumn.new(Hash)
  end
  def initialize(options = {})
    @a = options[:a]
    @b = options[:b]
  end
  attr_accessor :a, :b
end
```

It's worth noting that you'll be without the instance of MyField at first.

```
>>> rec = MyModel.new
>>> rec.my_field.class
 => MyField
>>> rec.my_field = {}
>>> rec.my_field.class
 => Hash
>>> rec.save
 => true
>>> rec.my_field.class
 => MyField
```

So, another option is to override the setter on MyModel:

```
class MyModel < ActiveRecord::Base
  serialize :my_field, MyField
  attr_accessible :my_field
  def my_field= value
    case value
    when Hash
      super MyField.new(value)
    else
      super
    end
  end
end
```

```
>>> rec = MyModel.new
>>> rec.my_field.class
 => MyField
>>> rec.my_field = {}
>>> rec.my_field.class
 => MyField
>>> rec.save
 => true
>>> rec.my_field.class
 => MyField
>>> rec.attributes = {:my_field => {}}
>>> rec.my_field.class
 => MyField
```
