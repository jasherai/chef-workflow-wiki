Using MiniTest
--------------

If you're new to MiniTest, or SUnit derivatives in general, you may want to may
read the [documentation](http://docs.seattlerb.org/minitest/) before you get
started.

MiniTest is extremely malleable and exposes both SUnit and "Spec" style
testing, which is why it was chosen to provide our functionality. We expose
some additional functionality to support [[Provisioning Machines]] and
[[Testing Multi-Machine Behavior]].

A unit test looks like this:

```ruby
require 'chef-workflow/helper'

class TestSyslog < MiniTest::Unit::ProvisionedTestCase
  def self.before_suite
    super
    provision('syslog-server')
  end

  def self.after_suite
    super
    deprovision('syslog-server')
  end

  def test_client_write
    provision('syslog-client', 1, %w[syslog-server])
    wait_for('syslog-client')

    client_ip = get_role_ips('syslog-client').first
    server_ip = get_role_ips('syslog-server').first

    ssh_command(client_ip, 'logger test_client_write speaking')

    assert_match(
      /test_client_write speaking/,
      ssh_capture(server_ip, "sudo cat /var/log/remote/#{client_ip}/syslog")
    )
  ensure
    deprovision('syslog-client')
  end
end
```

MiniTest::Unit tests are methods prefixed with `test_`. Integration tests that
use chef-workflow's extensions inherit from
`MiniTest::Unit::ProvisionedTestCase` and class names that will be treated as
suites start with `Test`.

There are a few extensions here worth discussing; all are imported when
inheriting from `MiniTest::Unit::ProvisionedTestCase`. Everything here is a
part of `chef-workflow-testlib`.

`chef-workflow/helper` is the require that brings in most of this functionality.

`self.before_suite` and `self.after_suite` are additional hooks similar to
minitest's standard `setup` and `teardown` that run when the suite starts, and
the suite finishes, where `setup` and `teardown` run around each test.

`provision`, `deprovision`, and `wait_for` are provisioning calls that are
discussed further in [[Provisioning Machines]].

`get_role_ips`, and the various `ssh` calls are additional helpers to talk to
different machines. You can talk to single machines or all machines of a server
group (machines provisioned with a common role), see [[Testing Multi-Machine
Behavior]] for more information.

MiniTest::Unit assertions start with `assert` or `refute` and come from the
[MiniTest::Assertions](http://docs.seattlerb.org/minitest/MiniTest/Assertions.html)
module. We have also provided some additional assertions in
`MiniTest::Assertions::RemoteChef`:

* `assert_search`: Given a type of search, query, and a list of objects, asserts the
  query for the type results in the list of objects provided.
* `refute_search`: Negative form of `assert_search`.
* `assert_search_count`: Counting form of `assert_search`. Instead of checking
  each object, just asserts that the expected number of results was returned.
* `refute_search_count`: Negative form of `assert_search_count`.
