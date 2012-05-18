---
layout: post
title: "Killing complexity with TDD"
date: 2012-05-18 06:30
comments: true
categories: 
- tdd
---
_Sometimes it's the small things..._

# In short

I'm working on an API that can take one of its arguments in a couple of different forms.
TDD helped me quickly see where I was going to end up with some combinatorial complexity, so I quickly changed my design to avoid the complexity.

# My Rails controller

It's a common thing: add a user to a group.
The old UI allows adding 1 user at a time.
The new UI allows adding any number of users at a time.
I'm planning to support both.

Here's what the original controller, supporting the old UI, might look like:

``` ruby group_members_controller.rb
class GroupMembersController < ApplicationController
  # POST /groups/:group_id/members
  def create
    group = Group.find(params[:group_id])
    if group.users.where(:id => params[:user_id]).count.zero?
      group.users << User.find(params[:user_id])
    redirect_to root_path
  end
end
```

I got the new UI working, and needed to update the controller.
Being that it was so simple of a controller, I had previously skipped unit tests for it.
So, I spun up a new unit test suite with tests to catch any regressions...

``` ruby group_members_controller_test.rb
class GroupMembersControllerTest < ActionController::TestCase
  setup do
    @group = Group.create!
  end

  test 'adds a group member' do
    @user = User.create!
    post :create, :group_id => @group.id, :user_id => @user.id
    assert_equal [@user], @group.reload.users
  end

  test 'does not re-add a member' do
    @user = User.create!
    @group.users << @user
    post :create, :group_id => @group.id, :user_id => @user.id
    assert_equal [@user], @group.reload.users
  end
end
```

Now to add the multi-user test.
(In this case, the UI is building a single input with a ','-delimited list of user ids.)
Here's the first test:

``` ruby group_members_controller_test.rb
  test 'adds two members' do
    @user1 = User.create!
    @user2 = User.create!
    post :create, :group_id => @group.id, :user_ids => "#{@user1.id},#{@user2.id}"
    assert_equal [@user1, @user2], @group.reload.users
  end
```

In the back of my mind, I start thinking about other tests I'll need to write.
One set of tests that comes to mind is the case where a :user_id and :user_ids are both passed in.

Hm. How can I avoid writing all these combination-testing tests?
Well, seeing as 2! is bigger than 1!,
if I can figure out how to drop the number of handled parameters from 2 to 1,
I'll avoid the extra complexity.

In this case, `:user_id => '1'` and `:user_ids => '1'` should behave the same,
so there's really no need for two different parameters. A quick UI & test change ...

``` ruby group_members_controller_test.rb
  test 'adds two members' do
    @user1 = User.create!
    @user2 = User.create!
    post :create, :group_id => @group.id, :user_id => "#{@user1.id},#{@user2.id}"
    assert_equal [@user1, @user2], @group.reload.users
  end
```

... and I've crossed off several tests in my mental checklist.

The API isn't necessarily ideal (I'm mixing up singular and plural concepts).
But, I get a simpler API that still supports the old UI.

# Wrap up

In this case, the added complexity was relatively small (only two choices),
so the savings in test effort was minimal. Also, I don't know that I would have
actually tested the extra cases, because they're not that likely or interesting.

However, I think the principal applies: TDD helps you spot complexity as it's
forming, and trim it back to a reasonable level.

Also, I just didn't have to think any extra "what-ifs".
