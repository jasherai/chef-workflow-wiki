Testing Multi-Machine Behavior
------------------------------

One of the more powerful aspects of having proper integration testing for chef
is that you can test entire networks of machines, and how they interoperate.
Chef-Workflow provides a number of API calls you can use in your integration
tests to orchestrate many of these things.

Each machine is presumed to have:

* A working copy of ssh on port 22. The user and credentials are controlled by
  the configuration you set in [[Configuring Chef Workflow]]. While not
  explicitly required, it is probably in your best interest to ensure this user
  has unfettered access to passwordless `sudo`.
* An externally accessible IP address. This should be configured by your
  provisioner.

**Note**: for most of these calls, the term 'role' is use when 'server group'
is meant. We're hoping to decouple the notion of server groups and roles in the
future, but these calls were written with this association in mind. Just be
aware of this as you read, and expect synonyms that reflect the server group
nature as chef-workflow matures.

It may help to read the example in [[Using MiniTest]] while reading this
document as it uses much of this API.

Mining the IP database
----------------------

`get_role_ips` is used to get a list of IP addresses associated with a given
server group. The IP addresses returned preserve the order of the
provisioning, e.g., the first machine provisioned will be the first IP address
returned.

Example:

```ruby
provision('syslog-server', 2)
wait_for('syslog-server')
# syslog-server-1 is the first address, syslog-server-2 the second.
get_role_ips('syslog-server') # => ['10.10.10.2', '10.10.10.3']
```

Testing over the network in general
-----------------------------------

Note that, despite the API that will be presented to you in the next section,
there is no limitation to what you can do as long as you can contact the
service. For example, you may wish to use the ruby
[Resolv](http://ruby-doc.org/stdlib-1.9.2/libdoc/resolv/rdoc/Resolv.html)
library to do DNS queries against a machine running a DNS server, or maybe you
want to drive additional Chef interaction or use something like
[rest-client](https://github.com/archiloque/rest-client) to perform HTTP
requests. There's no limitations here, and you're encouraged to abuse the
network how you would expect it to (or not to) behave.

Orchestrating SSH commands
--------------------------

Here are a few commands you can use to perform commands with `ssh`. Note if
`use_sudo` is on in `configure_knife`, every command will be prefixed with it
(a future version of chef-workflow may change this).

* `ssh_role_command(role, command)`: takes a server group name and a command to
  run. Runs it in parallel on all machines in the group.
* `ssh_command(ip, command)`: execute a command on a single machine, accessing
  by way of the IP address. Returns the exit status of the command.
* `ssh_capture(ip, command)`: exactly like `ssh_command`, but returns a capture
  of the session (merged stdout/stderr) after it executes.
