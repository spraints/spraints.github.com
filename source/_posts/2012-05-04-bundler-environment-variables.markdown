---
layout: post
title: "bundler environment variables"
date: 2012-05-04 17:00
comments: true
categories: 
---
Here are some environment variables that [bundler](http://gembundler.com/) uses. This is only
the bundler-specific variables. I'm leaving out other environment
variables, like RUBYOPT or PATH, that bundler uses, but that are used
by other things on the system, too. There is a partial list in
[the bundle-config man page](http://gembundler.com/man/bundle-config.1.html).

Some of the values are accessed through `Bundler::Settings`. Any values accessed there can
be set in a local (`./.bundle/config`) or global (`~/.bundle/config`) settings file. The order
of precedence is: local, environment, global.

Incidentally, while looking through bundler 1.1.1's code, I learned that you can
trigger whether to load a particular gem with environment variables. So, for example,
if you say `gem 'debugger', :env => 'USE_DEBUGGER'`, it only loads the debugger
gem if `USE_DEBUGGER` is set; or, if you say `gem 'debugger', :env => { 'USE_DEBUGGER' => 'yes' }`,
then it only loads the debugger gem if `USE_DEBUGGER` is set to `yes`.

Variable | Description  | Source reference
---------|--------------|-----------------
`BUNDLER_EDITOR`    | editor to use for commands like `bundle open`, defaults to `$VISUAL` or `$EDITOR`.) | [bundler/cli.rb][cli]
`BUNDLE_APP_CONFIG` | path to directory where bundler stores local configuration, defaults to `./bundle/config` | [bundler.rb][base]
`BUNDLE_BIN_PATH`   | | [bundler/rubygems\_integration.rb][gemint], [bundler/runtime.rb][runtime]
`BUNDLE_BIN` (`settings[:bin]`) | | [bundler.rb][base], [bundler/installer.rb][installer]
`BUNDLE_CLEAN` (`settings[:clean]`) | Run `bundle clean` during `bundle install` and `bundle update` | [bundler/cli.rb][cli]
`BUNDLE_CONFIG`     | global bundler settings file, defaults to `~/.bundle/config` | [bundler/settings.rb][settings]
`BUNDLE_DISABLE_SHARED_GEMS` (`settings[:disable_shared_gems]`) | if set, bundle won't use shared (system) gems | [bundler.rb][base]
`BUNDLE_FROZEN` (`settings[:frozen]`) | used internally | [bundler/cli.rb][cli], [bundler/definition.rb][definition], [bundler/installer.rb][installer]
`BUNDLE_GEMFILE`    | path to the gemfile | [bundler/cli.rb][cli], [bundler/runtime.rb][runtime], [bundler/shared\_helpers.rb][shared]
`BUNDLE_NO_PRUNE` (`settings[:no_prune]`) | Don't remove stale gems from the cache | [bundler/cli.rb][cli], [bundler/runtime.rb][runtime]
`BUNDLE_PATH` (`settings.path` or `settings[:path]`) | where bundler should install your gems, defaults to the shared (system) gem path | [bundler/settings.rb][settings], [bundler/cli.rb][cli], [bundler/installer.rb][installer]
`BUNDLE_SPEC_RUN`   | used in bundler's tests | [bundler/shared\_helpers.rb][shared]
`BUNDLE_SYSTEM_BINDIR` (`settings[:system_bindir]`) | overridden gem binstub directory, similar to `-n` in `.gemrc` | [bundler.rb][base]
`BUNDLE_WITHOUT` (`settings.without`) | list of groups to exclude, :-separated | [bundler/cli.rb][cli], [bundler/definition.rb][definition]
`DEBUG`, `DEBUG_RESOLVER` | outputs debug logs | [bundler/resolver.rb][resolver], [bundler/setup.rb][setup], [bundler/ui.rb][ui]
`HTTP_PROXY`, `HTTP_PROXY_USER`, `HTTP_PROXY_PASS` | configures an HTTP proxy

[base]:       https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler.rb
[cli]:        https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/cli.rb
[definition]: https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/definition.rb
[gemint]:     https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/rubygems_integration.rb
[installer]:  https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/installer.rb
[resolver]:   https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/resolver.rb
[runtime]:    https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/runtime.rb
[settings]:   https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/settings.rb
[setup]:      https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/setup.rb
[shared]:     https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/shared_helpers.rb
[ui]:         https://github.com/carlhuda/bundler/blob/v1.1.1/lib/bundler/ui.rb
