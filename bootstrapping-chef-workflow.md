Bootstrapping Chef Workflow
---------------------------

Please see [[Hassle Free Trial]] for notes on requirements. Not all may be
applicable if you wish to host your VMs on EC2, or don't want to use the test
library.

* If you don't already have a `Gemfile` in your chef-repository, create one and put these lines in it:

```ruby
source :rubygems
gem 'chef-workflow-tasklib'
gem 'chef-workflow-testlib'
```
* Note that it's ok to add a chef gem requirement in here; most anything in the 10.x series should work.
  * If you use an version of chef older than 10.16.4, you may be required to add this to fix a chef gem dependency bug:

```ruby
gem 'moneta', '~> 0.6.0'
```

* Run `bundle install`
* Run `bundle exec chef-workflow-bootstrap
  * This creates a `Rakefile` and a `lib/chef-workflow-config.rb` if they do not already exist.
* If you already had a `Rakefile`, add these lines:

```ruby
require 'chef-workflow-tasklib'
chef_workflow_task 'default'
```

* Add `.chef-workflow` to your `.gitignore`.
* Don't forget to add `Rakefile`, `lib/chef-workflow-config.rb`, `Gemfile` and `Gemfile.lock` to your repository!
