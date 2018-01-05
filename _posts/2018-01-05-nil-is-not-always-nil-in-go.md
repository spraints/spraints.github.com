---
layout: post
title: Nil isn't always nil in Go
---

I ran across some interesting behavior in [Go](https://golang.org/) today.
Given [this program](https://gist.github.com/spraints/7fa7f93366b6cdfffe30e4e5232adb02), what would you expect to happen?

<script src="https://gist.github.com/spraints/7fa7f93366b6cdfffe30e4e5232adb02.js"></script>

My first guess is that it would output this:

<pre>
Trying &main.Owner{org:(*main.Organization)(nil), user:(*main.User)(nil)}
 => "NIL"
Trying &main.Owner{org:(*main.Organization)(0xc42000e200), user:(*main.User)(nil)}
 => "organization:group"
Trying &main.Owner{org:(*main.Organization)(nil), user:(*main.User)(0xc42000e250)}
 => "user:person"
</pre>

However, that's not what actually happens. Instead, you'd see this:

<pre>
Trying &main.Owner{org:(*main.Organization)(nil), user:(*main.User)(nil)}
 => panic!! "invalid memory address or nil pointer dereference"
Trying &main.Owner{org:(*main.Organization)(0xc42000e200), user:(*main.User)(nil)}
 => "organization:group"
Trying &main.Owner{org:(*main.Organization)(nil), user:(*main.User)(0xc42000e250)}
 => panic!! "invalid memory address or nil pointer dereference"
</pre>

To see why, it helps to see the output from the `fmt.Printf` statements that are in `Owner.DatabaseString`:

<pre>
Trying &main.Owner{org:(*main.Organization)(nil), user:(*main.User)(nil)}
  [0] <nil>
  [1] (*main.Organization)(nil)
 => panic!! "invalid memory address or nil pointer dereference"
Trying &main.Owner{org:(*main.Organization)(0xc42000e200), user:(*main.User)(nil)}
  [0] <nil>
  [1] &main.Organization{name:"group"}
 => "organization:group"
Trying &main.Owner{org:(*main.Organization)(nil), user:(*main.User)(0xc42000e250)}
  [0] <nil>
  [1] (*main.Organization)(nil)
 => panic!! "invalid memory address or nil pointer dereference"
</pre>

I think this is happening because Go interface types are really a [type plus a value](https://golang.org/doc/faq#nil_error). The call to `GetOrganization()` returns a nil `Organization`, but not a nil `DatabaseStringer`.
