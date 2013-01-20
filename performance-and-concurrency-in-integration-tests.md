Performance and Concurrency in Integration Tests
------------------------------------------------

The chef-workflow provision scheduler is fully concurrent and can provision
many machines at the same time. This can be exploited for suites where larger
numbers of machines need to be provision to decrease the time required to
provision a network. This document will cover how you can do that.

Almost all of this document will assume you have read [[Provisioning
Machines]]. If you have not done that, you should before continuing.

**Note**: Vagrant provisioning does not support parallel testing, due to
limitations on how Vagrant and VirtualBox work together. Please see
[[Understanding the Scheduler]] for more information on the differences between
parallel and serial provisioning. If you want to get the most out of this
document, use the EC2 scheduler (see [[Stock Provisioners]] for how).

Getting Information on the Provisioning Status
----------------------------------------------

`CHEF_WORKFLOW_DEBUG` (see [[Environment Variables]]) can assist with making
things a little more obvious, but especially for lots of provisions, it may
still not be very obvious what's happening as you're waiting for output. You
can send `SIGUSR2` or `SIGINFO` on platforms that support it (OS X and FreeBSD,
which can be sent with CTRL+T to the foreground process) to the process that's
performing the provisions to get information on what the scheduler is doing,
namely what it's currently provisioning, the status of those provisions, and
what it thinks is already provisioned.

The Dependency System
--------------------------------

The `provision` command takes a final argument, which is a list of dependencies
that block this provision. This is a basic, linear dependency graph, the
`provision` command will raise an exception if you try to add a dependency for
something that hasn't already had a provision scheduled for it.

For example, let's say these machine relationships exist:

* A has no dependencies
* B depends on A
* C depends on B
* D depends on A
* E depends on C and D
* F has no dependencies

These commands might be expressed in a `self.before_suite` definition as such:

```ruby
provision('a')
provision('b', 1, %w[a])
provision('c', 1, %w[b])
provision('d', 1, %w[a])
provision('e', 1, %w[c d])
provision('f')
```

This is what the scheduler will do, in order:

* Provision A and F immediately
* When A finishes, provision B and D
* When B finishes, provision C
* When C and D finish, provision E 

Note that these transitions may happen at any time that the prerequisites are
satisfied, for example D may finish long before C does, but E will not start
until B, C, and D are finished. Likewise A and F start at roughly the same time
because they have no dependencies to satisfy.

Exploiting the dependency system
--------------------------------

What this is most useful for is writing test suites that test a system which
needs to interact with many kinds of machines, but not necessarily at the same
time -- for example, a monitoring system.

This also introduces `wait_for` as a flow control statement. `wait_for` merely
blocks further execution of the test until the scheduler determines that the
list of server groups are provisioned. Especially in parallel provisioning,
using `wait_for` to control when your tests run is critical.

```ruby
class TestMonitor < MiniTest::Unit::ProvisionedTestCase
  def self.before_suite
    provision('monitor')
    # the rest of these machines don't depend on each other, but depend on
    # monitor. They will all be provisioned in parallel.
    provision('web-server', 1, %w[monitor])
    provision('mail-server', 1, %w[monitor])
    provision('syslog-server', 1, %w[monitor])
    provision('database-server', 1, %w[monitor])
  end

  # we don't want to run the web server tests until the web server exists.
  # while we're waiting, the other machines will still be going through their
  # provisions, so in many cases there will be less waiting as the tests
  # progress than with a serial provision.
  #
  # You don't have to wait for 'monitor' here because it's a dependency of
  # 'web-server'.
  def test_monitors_web_server
    wait_for('web-server')
    # test web-server stuff
  end

  def test_monitors_mail_server
    wait_for('mail-server')
    # test mail-server
  end

  def test_monitors_syslog_and_database
    # wait_for takes a list of server groups. It will not continue until all
    # groups are provisioned.
    wait_for('syslog-server', 'database-server')
    # test this stuff
  end

  # ... you get the idea. by the time we get to the second or third test,
  # there's a good chance all of the machines are ready to go.
end
```
