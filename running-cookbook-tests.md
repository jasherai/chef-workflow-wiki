Running Cookbook Tests
----------------------

chef-workflow's task library (**not** the test library, which is for
integration testing) supports running cookbook tests with
[minitest-chef-handler](https://github.com/btm/minitest-chef-handler). This
seems to be what most cookbooks use, fits well with chef-workflow-testlib's
minitest dependence and most importantly is simple to orchestrate.

**Before you do anything**, ensure that the above cookbook is in your chef
repository, by way of a cookbook resolver or a direct download. None of this
will work without it.

To run your tests, `bundle exec rake test:recipes`. This is a part of `bundle
exec rake test:full`, so presuming you have some recipes configured for testing
(see below), they will get run as a part of this task.

In `lib/chef-workflow-config.rb`, there is a knife configuration variable
called `test_recipes` that takes an array of strings with named recipes that
are to be tested.

For example:

```ruby
configure_knife do
  test_recipes [ 'chef-client::service' ]
end
```

This is what `bundle exec rake test:recipes` uses to determine what to test.

Each recipe is tested as such:

* A machine is provisioned
* It is bootstrapped with a chef client. The run list will contain the recipe,
  and `recipe[minitest-chef-handler]`. The node will be named a unique name
  based on the name of the recipe, prefixed with `recipe-`. For example:
  `recipe-chef-client-service`.
* After bootstrapping and converge, the result of the converge (by way of exit
  code) is evaluated. As with most things unix, zero is success, anything else
  is failure. The latter aborts the testing process.
* Machines are left around until the next run or until `bundle exec rake
  test:recipes:cleanup` is run, so you can investigate the cause of any
  failures.
