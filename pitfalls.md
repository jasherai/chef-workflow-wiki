Pitfalls
--------

Chef-Workflow does have some expectations of your chef repository. Most of this
is unavoidable, and some of it is a balance of difficulty and pragmaticism.
Whenever possible, we're more interested in letting you decide how things will
work instead of telling you how things will work, so please let us know if
there are good solutions to resolve some of these things for you.

In all cases we've attempted to provide alternatives that should be quick to
implement and provide equivalent functionality.

Hardcoded addresses and server names in cookbooks and roles
-----------------------------------------------------------

Generally this is an anti-pattern, but sometimes it's necessary. However, since
especially the testlib will bring machines up and down as it pleases, having
"hard" locations that are communicated with or relied upon is going to cause
unnecessary problems.

Two solutions:

* Put these in an environment your stack uses. The testing system uses an
  isolated environment and will not obtain these values.
* Use [Chef Search](http://docs.opscode.com/chef/essentials_search.html) to
  locate your servers. You may want a library like
  [this](https://gist.github.com/51abe87002c04bc07c97) if you mix and match the
  way you do provisions as the host only interface we tell vagrant to use will
  not be presented as `node['ipaddress']`, and there are additional quirks on
  systems with split interfaces such as the machines on EC2. Alternatively,
  you could write an ohai plugin, but this is a little more flexible.

Interdependent Roles
--------------------

This is a little trickier, but many stacks have a situation where a server
depends on things that depend on it. For example, a DNS server and a syslog
server -- the syslog server needs DNS and the DNS server needs to write to the
syslog server. This almost never comes up in production because how frequently
do you build all the machines that comprise both roles at the same time?
Chef-Workflow builds your stack from scratch every full test run, so this is
one of those places the difference is apparent.

The solution is not much different from what you'd have to do in production if
this happened; you'd build and converge one role, then build and converge the
other role, then reconverge both machines. Presuming you managed to
successfully configure the critical component of each role, the second
converges would align the dependencies.

The difference is that the test library is not going to tolerate a failing
converge, and we think it's a feature that it doesn't. So, what you'll need to
do is make your cookbooks tolerate the inability to configure the
co-dependency, and then use the test helper `ssh_role_command` to double
converge after both succeed. We think this actually results in more robust
cookbooks.

The code would look something like this:

```ruby
codependents = %w[syslog_server dns_server]
codependents.each { |x| provision(x) }
codependents.each { |x| wait_for(x) }
codependents.each { |x| ssh_role_command(x, 'chef-client') }
```

If any of this code fails, the test would fail; you still get a thorough
exercising of the cookbooks involved.
