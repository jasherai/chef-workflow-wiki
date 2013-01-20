Provisioning Machines
---------------------

To do integration testing properly, you're going to need to be able to create
machines. Chef-Workflow allows you do this on the fly, and depending on your
Provisioner (see [[Stock Provisioners]] and possibly [[Writing A Provisioner]])
it may be able to do parts of your provisions in the background, to allow you
build many machines to improve test-run performance.

Integration Test Provisioning API
---------------------------------

There are 3 important calls that you will use in your tests to deal with
machines: `provision`, `deprovision`, and `wait_for`. You can use these
statements anywhere in any class that inherits from
`MiniTest::Unit::ProvisionedTestCase`. These calls are exposed from the VM
scheduler which all of chef-workflow uses to provision machines. If you want to
learn more about how it works, see [[Understanding The Scheduler]].

`provision` takes at least one argument, and as many as three. The first
argument is always the server group, which is a mapping to a role. Many
machines can exist in the server group. The second argument is a number of
servers, and the third argument is a list of other server groups that must be
fully provisioned before this one can be provisioned.

Provisioned machines are always created as new virtual machines, and
bootstrapped with knife against a chef server you have created. Machines are
not considered fully provisioned until both of these steps have completed, and
the chef bootstrap will hold up the process until a successful search completes
for all machines. This ensures that all machines are fully known by the chef
server before testing begins.

Machines will have the node name `$role-$num`, where `$num` indicates the order
in which they were provisioned. This order is preserved throughout the testing
system. For example, if you provisioned two machines in the server group
`syslog-client`, the node names would be `syslog-client-1` and
`syslog-client-2` respectively.

Note that for dependent provisions that the server groups depended on **must
already be scheduled**, even if they are not provisioned yet. `provision` will
raise an error unless this is the case.

For example (you may wish to refer to [[Using MiniTest]] for a more practical
example):

```ruby
provision('syslog-server') # one machine with the syslog-server role
# two machines with the role 'syslog-client', and 'syslog-server' must be
# satified before this provision will start.
provision('syslog-client', 2, %w[syslog-server])
```

Deprovisioning works against server groups and removes all machines in the
server group. Note that after you have provisioned a group that depends on
another group, you can deprovision that group without issue. This may have
unexpected side effects as provisions that depend on it may not have been
started by the time this happens, if not careful.

```ruby
deprovision('syslog-client')
```

The `wait_for` command blocks the test runner until the list of server groups
provided has successfully provisioned. This is important for controlling flow
around provisions and absolutely critical for provisioners that exploit the
parallel scheduler, like EC2.

Example:

```ruby
provision('syslog-server')
provision('syslog-client', 1, %w[syslog-server])
wait_for('syslog-client')
# in a parallel scenario, the next test would run immediately, long before the
# provision finishes, if it were not for the wait_for above, which will force
# the test suite to wait until syslog-client is finished before running.
assert_search(:node, "roles:syslog-server", %w[syslog-server-1])
```

When your test run finishes
---------------------------

When your test runs finish, either by way of getting through the whole run or
failing prematurely, your machines do not go away **unless you've told them to
do so**. The easiest way to make this happen is in `self.after_suite`, which
should always execute:

```ruby
class TestThing < MiniTest::Unit::ProvisionedTestCase
  def self.before_suite
    provision('thing')
  end

  def self.after_suite
    deprovision('thing')
  end
end
```

However if you're having trouble learning why a machine is failing, you may
wish to disable this and inspect the machine by hand. Alternatively, you may
wish to keep certain kinds of inconsequential machines around between test runs
to improve test performance.

These machines are still tracked in the state database. If you do want to clean
them up, `bundle exec rake chef:clean:machines` will do that for you, excepting
the chef server.
