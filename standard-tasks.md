Standard Tasks
--------------

In [[Bootstrapping Chef Workflow]] a `Rakefile` that looks like this was mentioned:

```ruby
require 'chef-workflow-tasklib'
chef_workflow_task 'default'
```

This is the default `Rakefile` that's provided by the bootstrapping tool. It imparts the
"default" task set. The tasks have descriptions which you can get with `bundle
exec rake -T` or `bundle exec rake -D`, but here's a short overview of what
each task does in the event you want more information.

Core Tasks
----------

These tasks provide primitive functionality that let you control in a fine
grained fashion how certain things happen. See below for composite tasks which
combine these core tasks to automate rote tasks.

* `chef_server:create` - initialize a chef server and create an appropriate knife
  configuration and certificates. This must be done before any tasks will work,
  or the configuration must be pointed at another knife configuration.
* `chef_server:destroy` - tears down any created chef server.
* `chef:cookbooks:upload` - upload your cookbooks to the chef server.
* `test` - run the integration test suite.
* `test:recipes` - run the recipes test suite.
* `test:recipes:cleanup` - clean up recipe machines.
* `chef:clean:machines` - clean up any machines that are not the chef server.
* `chef:roles:upload` - upload your roles to the chef server
* `chef:environments:upload` - upload your environments to the chef server
* `chef:data_bags:upload` - upload your data bags to the chef server
* `chef:build[role_name, machine_count]` - build `machine_count` machines of
  role `role_name`. These machines will be used for testing if your suites
  depend on machines with that role.
* `chef:converge[role_name]` - run `chef-client` on all machines created for
  role `role_name`.

**Note**: the `chef:info` namespace has a number of tasks to interrogate
information about the running environment, but they do not carry out any
actions themselves.

Composite Tasks
---------------

Many tasks are compositions of smaller tasks, intended to streamline rote tasks.

* `chef:clean` - clean everything up, remove `.chef-workflow` directory. 
* `chef:upload` - upload all chef information to the chef server.
* `test:build` - upload all the chef information to the chef server and run the
  integration tests.
* `test:rebuild` - same as `test:build`, only cleans up non-chef-server VMs first.
* `test:full` - everything. build a chef server, upload everything to it, run
  the integration tests, run the recipe tests, tear everything down and clean
  up. Great for CI, as it cleans up everything before it starts and when it
  finishes.

Optional Tasks
--------------

We include a few other tasks which are not a part of the "default" set yet are
commonly used by chef developers. To use these, include them into your
`Rakefile` with `chef_workflow_task`. For example to use 'berkshelf' support:

```ruby
chef_workflow_task 'chef/cookbooks/resolve/berkshelf'
```

Cookbook Resolvers
------------------

Cookbook resolvers work a bit like Bundler, but for cookbooks. Chef-Workflow
supports a common execution strategy for them, by exposing two tasks which are
found to be common amongst resolvers.

* `chef:cookbooks:resolve` - resolve and install the cookbooks. Will execute before
  `chef:cookbooks:upload` if it exists.
* `chef:cookbooks:update` - update the locked dependencies.

We provide support for:

* [librarian](https://github.com/applicationsonline/librarian):
  `chef_workflow_task 'chef/cookbooks/resolve/librarian'`
* [berkshelf](https://github.com/RiotGames/berkshelf): `chef_workflow_task
  'chef/cookbooks/resolve/berkshelf'`

**Important Notes**

For both of these you must `gem install` these packages directly, e.g., `gem
install berkshelf`. This is because they have conflicting dependencies we
unfortunately have no way of resolving.

Both of these will inject their results into the `cookbooks_path` in
`KnifeSupport`. It is *strongly advised* that you put nothing in there that is
not managed by these tools, and add that directory to your `.gitignore` if you
use these tools.

Foodcritic Support
------------------

We also optionally support [foodcritic](https://github.com/acrmp/foodcritic):
`chef_workflow_task 'chef/cookbooks/foodcritic'`.

This will add a `chef:cookbooks:foodcritic` task that, if a resolver is configured,
will resolve your cookbooks with `chef:cookbooks:resolve` beforehand.

The default is to check the `cookbooks_path`, but you can alter this behavior
with `fc_cookbooks_path` on KnifeSupport, which will become available if you
include the task. You can also adjust the options provided to foodcritic with
`fc_options`.
