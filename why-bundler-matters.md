Why Bundler Matters
-------------------

You may see many examples in here that start with `bundle exec` or `bundle exec
rake`. Bundler enforces our dependencies local to our repository. This actually
has a few benefits in the consistency department, but it's actually a
requirement for our project, because we use a newer version of `rake` than ruby
provides with it.

You may see from time to time `Rakefile`s that start with this:

```ruby
require 'bundler/setup'
```

This is ok for many situations but it's actually broken in a sense. See, the
dependency on `rake` is actually resolved long before `bundler` runs in this
case. If `rake` were to be a dependency in your Gemfile and conflicted with
your system installed rake, bundler and rubygems would both complain and abort.

This is actually what happens for us, and is largely unavoidable. `rake` also
depends on other libraries like `json` which chef also depends on, but
unsurprisingly, these are different, conflicting versions.

So, we're boned. Sorry. If you want to save your fingers, try adding this alias
to your shell configuration:

```sh
alias be="bundle exec"
```

Which makes it a little more tolerable.
