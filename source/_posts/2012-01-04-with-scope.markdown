---
layout: post
title: "with_scope"
date: 2012-01-04 13:14
comments: true
categories:
- rails
- activerecord
---
I was writing a custom 'find_or_create' method on a model today, that I
was planning to use from an association collection. I wanted it to pick
up the association's scope (the foreign key value). It turns out, it
happens automatically.

Here's an example:

``` ruby
class Room
  has_many :members
end
```

``` ruby
class Member
  belongs_to :room
  class << self
    def find_or_create_for_invitation(id_or_email)
      where(:id => id).first || where(:email => email).first
      # ... or create
    end
  end
end
```

What I really want, is for my member find-or-create to be scoped to the room
it's on when I do `room.members.find_or_create_for_invitation(params[:invitee])`.

I figured it was possible, so I set a debugger breakpoint in `find_or_create_for_invitation`
and checked out the available information. What I found is that the static method
is called inside of a `with_scope` block, so `Member.scoped_methods` has some
useful information. In this case, it's an array with one element, the scoped
association relation (i.e. the thing returned by `room.members`). It's possible for
the array to contain hashes, too, and several elements, if there are several
nested scopes. So, any find-like or create-like calls that you make inside the
class method will automatically get the right scope.
