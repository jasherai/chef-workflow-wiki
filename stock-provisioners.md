Stock Provisioners
------------------

There are two stock, or bundled provisioners with chef-workflow: Vagrant and
EC2. Here's a highlight of the differences.

You should also read [[Configuration Sections]] to ensure that if you intend to
switch provisioners, that you're configuring them the way you wish them to be
configured.

Vagrant Provisioner
-------------------

This provisioner is currently the default. To use this provisioner, ensure this
is in your `lib/chef-workflow-config.rb`:

```ruby
configure_general do
  machine_provisioner :vagrant
end
```


* Runs on your local machine, with VirtualBox and Vagrant.
* Machines run on a host-only subnet defined by you.
  * Machines have a NAT connection through your primary interface and full
    internet access.
  * IP addresses are allocated incrementally and a secondary host-only
    interface is created for them.
  * No attempt is made to firewall, well, anything
  * Enhanced features like folder sharing is not supported.
* Requires a serial scheduler to run. See [[Understanding the Scheduler]] and
  [[Performance and Concurrency in Integration Tests]] to understand the
  implications.
* "Base Boxes" are expressed as URLs, so new contributors can get started
  immediately; vagrant will download the box on the first provision.
* For certain users, this method will be more appropriate for proxied or
  offline operation.

EC2 Provisioner
---------------

To use this provisioner, ensure this is in your `lib/chef-workflow-default.rb`:

```ruby
configure_general do
  machine_provisioner :ec2
end
```

* Runs on Amazon EC2, a for-pay service that is not your local machine. Network
  and Hosting charges will apply to most operations, including running tests.
* Machines run on a dual-interface system.
  * All chef-workflow interaction with the provisioned machines will be
    performed over the external interface. If you wish to drive internal
    interaction, use the ssh helpers or write additional helpers.
  * By default, a security group will be defined that all machines will run
    under with a conservative firewall.
     * You can change the ports that are open, which apply to all machines.
     * You can also create pre-seeded security groups with more elaborate rules.
      They still apply to all the machines, but this offers a little more
      flexibility.
     * auto-created security groups are destroyed when the `chef:clean` task is run.
* Instance provisioning is done by default from keys in the user's shell
  environment. **This is overridable and it is strongly recommended you do so.**
* The parallel scheduler is used by default. See [[Performance and Concurrency in Integration Tests]].
* If desired, one can employ a number of techniques such as using EBS-root AMIs
  or larger instances to improve provisioning performance as well, at the cost
  of, well, additional cost to do so.
* EC2-aware chef cookbooks can operate fully in this environment, doing things
  like manipulating EC2 volumes that live on the host.
* Additionally, testing of integration with other EC2 services like RDS is
  possible.
* Making machines that have test failures available to other team members is
  much simpler.
