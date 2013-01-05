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
* `cookbooks:upload` - upload your cookbooks to the chef server.
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
