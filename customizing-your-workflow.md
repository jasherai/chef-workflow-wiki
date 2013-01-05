Customizing Your Workflow
-------------------------

One of the key features of chef-workflow is that nothing is sacred. In
[[Bootstrapping Chef-Workflow]] we have you create a `Rakefile` that looks like
this:

```ruby
require 'chef-workflow-tasklib'
chef_workflow_task 'default'
```

`chef_workflow_task` is just a call to `require` that honors the [[Filesystem
Layout]]. `lib/chef-workflow/tasks/default.rb` just contains a ton of other
requires to other tasks.

All tasks have been deliberately written to only require the dependencies they
need to function. This means that you can tweak it as you please to suit what
you actually want to do.

Let's say you *just* want to upload cookbooks. No chef server, no other meta,
no testing system, etc. Your rakefile would look something like this instead:

```ruby
require 'chef-workflow-tasklib'
chef_workflow_task 'chef/cookbooks/upload'
```

This will pull in a number of tasks, but only what it depends on. You will
still have to configure `knife_config_path` to point at your working
`knife.rb`, but it will all just work and do exactly what you expect.

Task List
---------

Here's a list of tasks that are included in chef-workflow-tasklib as of this
writing. You may wish to peruse the [chef-workflow-tasklib
repository](https://github.com/chef-workflow/chef-workflow-tasklib) to get an
exhaustive list.

It may help to compare these to the [[Standard Tasks]] documentation.

* `chef_server` - tasks for creating and destroying chef servers.
* `test` - tasks for running integration and recipe tests.
* `test/build` - tasks for streamlining testing, e.g. uploading data before running tests.
* `chef/build` - provides `chef:build`
* `chef/clean` - provides `chef:clean` and `chef:clean:machines`
* `chef/converge` - provides `chef:converge`
* `chef/data_bags` - provides knife support for data bags
* `chef/environments` - provides knife support for environments
* `chef/info` - provides tasks to inspect configuration and state
* `chef/roles` - provides knife support for roles
* `chef/upload` - links all knife upload tasks.
* `chef/cookbooks/upload` - provides `chef:cookbooks:upload`
* `chef/cookbooks/foodcritic` - provides foodcritic support
* `chef/cookbooks/resolve/berkshelf` - provides berkshelf support
* `chef/cookbooks/resolve/librarian` - provides librarian support

If you want to add tasks, see [[Writing Rake Tasks]] or if you're a gem author,
see [[Supporting Chef-Workflow in your Gem]]
