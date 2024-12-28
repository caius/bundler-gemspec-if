# Bundler / gemspec `if` issue

The git history shows the issues, but that's hard to read 🙃

The basic issue seems to be having a `gemspec` specifying dependencies inside an `if` block, so on some versions of ruby there are more dependencies than older versions of ruby.

In the [oas_agent.gemspec][] we currently specify more dependencies if you're on Ruby 3.3+ or 3.4+, due to ruby extracting those gems from the default gems into bundled gems. Without specifying them they break on the next minor version of ruby from the one that extracted them.

[oas_agent.gemspec]: https://github.com/Own-and-Ship/oas_agent/blob/577b629d2167c0d04088bb0f4ec723c9739aa5a1/oas_agent.gemspec#L37-L44

Given we start on Ruby 3.1.6 and have a `Gemfile` depending on `oas_agent` from GitHub, running `bundle install` gets us a lockfile in [1cef81][] that depends on `oas_agent` and has one dependency, `msgpack`.

[1cef81]: https://github.com/caius/bundler-gemspec-if/commit/1cef810fe62ae8a0cd75ee330c3d40753674bbfc

We then upgrade to Ruby 3.3.6 and run `bundle install`. Bundler changes **nothing** in the lockfile, although we now expect `oas_agent` to have a second dependency on `base64`.

We have to run `bundle update oas_agent` to cause bundler to evaluate the dependencies from Ruby 3.3 and update the lockfile with the second dependency on `base64` as expected.

The same behaviour is evaluated on Ruby 3.4 with the `logger` and `ostruct` dependencies.

If we try and add them as top level dependencies then it fails on Ruby 2.2 and older, because those gems don't function there.
